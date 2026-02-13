<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## Um Olhar Mais Atento às Traits para Async

Ao longo do capítulo, usamos as traits `Future`, `Stream` e `StreamExt`
de várias maneiras. Até agora, porém, evitamos entrar muito nos
detalhes de como elas funcionam ou como se encaixam, o que é bom na maior
parte do tempo para o seu trabalho diário em Rust. Às vezes, no entanto, você encontrará
situações onde precisará entender um pouco mais sobre os detalhes dessas traits,
juntamente com o tipo `Pin` e a trait `Unpin`. Nesta seção, vamos nos aprofundar
apenas o suficiente para ajudar nesses cenários, deixando ainda o mergulho _realmente_ profundo
para outra documentação.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### A Trait `Future`

Vamos começar dando uma olhada mais de perto em como a trait `Future` funciona. Aqui está como
Rust a define:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Essa definição de trait inclui um monte de novos tipos e também alguma sintaxe que
não vimos antes, então vamos percorrer a definição peça por peça.

Primeiro, o tipo associado `Output` da `Future` diz no que o future resolve.
Isso é análogo ao tipo associado `Item` para a trait `Iterator`.
Segundo, `Future` tem o método `poll`, que recebe uma referência especial `Pin`
para seu parâmetro `self` e uma referência mutável para um tipo `Context`, e
retorna um `Poll<Self::Output>`. Falaremos mais sobre `Pin` e `Context` em um
momento. Por enquanto, vamos focar no que o método retorna, o tipo `Poll`:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

Este tipo `Poll` é similar a um `Option`. Ele tem uma variante que tem um valor,
`Ready(T)`, e uma que não tem, `Pending`. `Poll` significa algo bem
diferente de `Option`, no entanto! A variante `Pending` indica que o future
ainda tem trabalho a fazer, então o chamador precisará verificar novamente mais tarde. A variante `Ready`
indica que a `Future` terminou seu trabalho e o valor `T` está
disponível.

> Nota: É raro precisar chamar `poll` diretamente, mas se você precisar, tenha
> em mente que com a maioria dos futures, o chamador não deve chamar `poll` novamente depois
> que o future retornou `Ready`. Muitos futures entrarão em pânico se consultados (polled) novamente após
> ficarem prontos. Futures que são seguros para consultar novamente dirão isso explicitamente em
> sua documentação. Isso é similar a como `Iterator::next` se comporta.

Quando você vê código que usa `await`, Rust o compila "por baixo dos panos" para código
que chama `poll`. Se você olhar para trás na Listagem 17-4, onde imprimimos o
título da página para uma única URL assim que ela resolveu, Rust a compila em algo
meio (embora não exatamente) parecido com isso:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // Mas o que vai aqui?
    }
}
```

O que devemos fazer quando o future ainda está `Pending`? Precisamos de alguma maneira de tentar
de novo, e de novo, e de novo, até que o future esteja finalmente pronto. Em outras palavras,
precisamos de um loop:

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continue
        }
    }
}
```

Se o Rust compilasse para exatamente esse código, no entanto, cada `await` seria
bloqueante—exatamente o oposto do que estávamos buscando! Em vez disso, Rust garante
que o loop possa passar o controle para algo que possa pausar o trabalho neste
future para trabalhar em outros futures e então verificar este novamente mais tarde. Como
vimos, esse algo é um runtime async, e esse trabalho de agendamento e coordenação
é um de seus principais trabalhos.

Na seção [“Enviando Dados Entre Duas Tarefas Usando Passagem de Mensagem”][message-passing]<!-- ignore -->, descrevemos esperar em
`rx.recv`. A chamada `recv` retorna um future, e aguardar o future o consulta (poll).
Notamos que um runtime pausará o future até que ele esteja pronto com
`Some(message)` ou `None` quando o canal fechar. Com nosso entendimento mais profundo
da trait `Future`, e especificamente `Future::poll`, podemos
ver como isso funciona. O runtime sabe que o future não está pronto quando ele retorna
`Poll::Pending`. Inversamente, o runtime sabe que o future _está_ pronto e
o avança quando `poll` retorna `Poll::Ready(Some(message))` ou
`Poll::Ready(None)`.

Os detalhes exatos de como um runtime faz isso estão além do escopo deste livro,
mas a chave é ver a mecânica básica de futures: um runtime _consulta_ (polls) cada
future pelo qual é responsável, colocando o future de volta para dormir quando ele ainda não está
pronto.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### O Tipo `Pin` e a Trait `Unpin`

Voltando à Listagem 17-13, usamos a macro `trpl::join!` para aguardar três
futures. No entanto, é comum ter uma coleção como um vetor contendo
algum número de futures que não será conhecido até o tempo de execução. Vamos mudar a Listagem
17-13 para o código na Listagem 17-23 que coloca os três futures em um vetor
e chama a função `trpl::join_all` em vez disso, o que não compilará ainda.

<Listing number="17-23" caption="Aguardando futures em uma coleção"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

Colocamos cada future dentro de um `Box` para torná-los em _trait objects_ (objetos de trait), assim como
fizemos na seção “Retornando Erros de `run`” no Capítulo 12. (Cobriremos
trait objects em detalhes no Capítulo 18.) Usar trait objects nos permite tratar cada
um dos futures anônimos produzidos por esses tipos como o mesmo tipo, porque todos
eles implementam a trait `Future`.

Isso pode ser surpreendente. Afinal, nenhum dos blocos async retorna nada,
então cada um produz um `Future<Output = ()>`. Lembre-se que `Future` é uma
trait, no entanto, e que o compilador cria um enum único para cada bloco async,
mesmo quando eles têm tipos de saída idênticos. Assim como você não pode colocar duas
structs escritas à mão diferentes em um `Vec`, você não pode misturar enums
gerados pelo compilador.

Então passamos a coleção de futures para a função `trpl::join_all` e
aguardamos o resultado. No entanto, isso não compila; aqui está a parte relevante
das mensagens de erro.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

A nota nesta mensagem de erro nos diz que devemos usar a macro `pin!` para
_fixar_ (pin) os valores, o que significa colocá-los dentro do tipo `Pin` que
garante que os valores não serão movidos na memória. A mensagem de erro diz que pinning
é necessário porque `dyn Future<Output = ()>` precisa implementar a trait `Unpin`
e ela atualmente não implementa.

A função `trpl::join_all` retorna uma struct chamada `JoinAll`. Essa struct é
genérica sobre um tipo `F`, que é restrito a implementar a trait `Future`.
Aguardar diretamente um future com `await` fixa (pins) o future implicitamente. É por isso que
não precisamos usar `pin!` em todos os lugares que queremos aguardar futures.

No entanto, não estamos aguardando diretamente um future aqui. Em vez disso, construímos um novo
future, JoinAll, passando uma coleção de futures para a função `join_all`.
A assinatura para `join_all` requer que os tipos dos itens na
coleção implementem a trait `Future`, e `Box<T>` implementa `Future`
apenas se o `T` que ele envolve for um future que implementa a trait `Unpin`.

Isso é muita coisa para absorver! Para realmente entender, vamos mergulhar um pouco mais fundo
em como a trait `Future` realmente funciona, em particular em torno de pinning. Olhe
novamente para a definição da trait `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Método obrigatório
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

O parâmetro `cx` e seu tipo `Context` são a chave para como um runtime realmente
sabe quando verificar qualquer future dado enquanto ainda é preguiçoso (lazy). Novamente, os detalhes
de como isso funciona estão além do escopo deste capítulo, e você geralmente só
precisa pensar sobre isso ao escrever uma implementação customizada de `Future`. Vamos
focar em vez disso no tipo para `self`, pois esta é a primeira vez que vimos um
método onde `self` tem uma anotação de tipo. Uma anotação de tipo para `self` funciona
como anotações de tipo para outros parâmetros de função, mas com duas diferenças chave:

- Ela diz ao Rust qual tipo `self` deve ser para o método ser chamado.
- Ela não pode ser qualquer tipo. É restrita ao tipo no qual o método é
  implementado, uma referência ou ponteiro inteligente para esse tipo, ou um `Pin` envolvendo uma
  referência para esse tipo.

Veremos mais sobre essa sintaxe no [Capítulo 18][ch-18]<!-- ignore -->. Por enquanto,
é suficiente saber que se quisermos consultar (poll) um future para verificar se ele está
`Pending` ou `Ready(Output)`, precisamos de uma referência mutável envolvida em `Pin` para o
tipo.

`Pin` é um wrapper (envoltório) para tipos semelhantes a ponteiros como `&`, `&mut`, `Box`, e `Rc`.
(Tecnicamente, `Pin` funciona com tipos que implementam as traits `Deref` ou `DerefMut`,
mas isso é efetivamente equivalente a trabalhar apenas com referências e
ponteiros inteligentes.) `Pin` não é um ponteiro em si e não tem nenhum comportamento
próprio como `Rc` e `Arc` têm com contagem de referência; é puramente uma ferramenta que o
compilador pode usar para impor restrições no uso de ponteiros.

Lembrar que `await` é implementado em termos de chamadas para `poll` começa a
explicar a mensagem de erro que vimos antes, mas aquilo foi em termos de `Unpin`, não
`Pin`. Então, como exatamente `Pin` se relaciona com `Unpin`, e por que `Future` precisa que
`self` esteja em um tipo `Pin` para chamar `poll`?

Lembre-se de antes neste capítulo que uma série de pontos de await em um future
são compilados em uma máquina de estados, e o compilador garante que essa máquina de estados
siga todas as regras normais do Rust em torno de segurança, incluindo empréstimo (borrowing)
e posse (ownership). Para fazer isso funcionar, Rust olha para quais dados são necessários entre um
ponto de await e o próximo ponto de await ou o final do bloco async. Ele
então cria uma variante correspondente na máquina de estados compilada. Cada
variante obtém o acesso que precisa aos dados que serão usados nessa seção
do código fonte, seja tomando posse desses dados ou obtendo uma
referência mutável ou imutável para eles.

Até agora, tudo bem: se errarmos algo sobre a posse ou referências em
um determinado bloco async, o borrow checker nos dirá. Quando queremos mover
o future que corresponde a esse bloco—como movê-lo para um `Vec` para
passar para `join_all`—as coisas ficam mais complicadas.

Quando movemos um future—seja empurrando-o para uma estrutura de dados para usar como um
iterador com `join_all` ou retornando-o de uma função—isso na verdade significa
mover a máquina de estados que o Rust cria para nós. E diferente da maioria dos outros tipos em
Rust, os futures que o Rust cria para blocos async podem acabar com referências a
si mesmos nos campos de qualquer variante dada, como mostrado na ilustração simplificada na Figura 17-4.

<figure>

<img alt="Uma tabela de uma coluna e três linhas representando um future, fut1, que tem valores de dados 0 e 1 nas duas primeiras linhas e uma seta apontando da terceira linha de volta para a segunda linha, representando uma referência interna dentro do future." src="img/trpl17-04.svg" class="center" />

<figcaption>Figura 17-4: Um tipo de dado autorreferencial</figcaption>

</figure>

Por padrão, no entanto, qualquer objeto que tenha uma referência a si mesmo é inseguro de mover,
porque referências sempre apontam para o endereço de memória real de qualquer coisa a que
se referem (veja a Figura 17-5). Se você mover a própria estrutura de dados, essas
referências internas ficarão apontando para o local antigo. No entanto, aquele
local de memória agora é inválido. Por um lado, seu valor não será atualizado
quando você fizer alterações na estrutura de dados. Por outro—coisa mais importante—,
o computador agora está livre para reutilizar essa memória para outros propósitos! Você poderia acabar
lendo dados completamente não relacionados mais tarde.

<figure>

<img alt="Duas tabelas, descrevendo dois futures, fut1 e fut2, cada um dos quais tem uma coluna e três linhas, representando o resultado de ter movido um future para fora de fut1 para dentro de fut2. O primeiro, fut1, está acinzentado, com um ponto de interrogação em cada índice, representando memória desconhecida. O segundo, fut2, tem 0 e 1 na primeira e segunda linhas e uma seta apontando de sua terceira linha de volta para a segunda linha de fut1, representando um ponteiro que está referenciando a localização antiga na memória do future antes de ele ter sido movido." src="img/trpl17-05.svg" class="center" />

<figcaption>Figura 17-5: O resultado inseguro de mover um tipo de dado autorreferencial</figcaption>

</figure>

Teoricamente, o compilador Rust poderia tentar atualizar cada referência a um
objeto sempre que ele fosse movido, mas isso poderia adicionar muita sobrecarga de desempenho,
especialmente se toda uma teia de referências precisasse de atualização. Se pudéssemos, em vez disso, garantir
que a estrutura de dados em questão _não se mova na memória_, não teríamos
que atualizar nenhuma referência. É exatamente para isso que serve o borrow checker do Rust:
em código seguro (safe code), ele impede que você mova qualquer item com uma referência ativa para
ele.

`Pin` baseia-se nisso para nos dar a garantia exata de que precisamos. Quando nós _fixamos_ (pin) um
valor envolvendo um ponteiro para esse valor em `Pin`, ele não pode mais se mover. Assim,
se você tem `Pin<Box<SomeType>>`, você na verdade fixa o valor `SomeType`, _não_
o ponteiro `Box`. A Figura 17-6 ilustra este processo.

<figure>

<img alt="Três caixas dispostas lado a lado. A primeira é rotulada “Pin”, a segunda “b1”, e a terceira “pinned”. Dentro de “pinned” é uma tabela rotulada “fut”, com uma única coluna; ela representa um future com células para cada parte da estrutura de dados. Sua primeira célula tem o valor “0”, sua segunda célula tem uma seta saindo dela e apontando para a quarta e última célula, que tem o valor “1” nela, e a terceira célula tem linhas tracejadas e reticências para indicar que pode haver outras partes da estrutura de dados. No total, a tabela “fut” representa um future que é autorreferencial. Uma seta sai da caixa rotulada “Pin”, passa pela caixa rotulada “b1” e termina dentro da caixa “pinned” na tabela “fut”." src="img/trpl17-06.svg" class="center" />

<figcaption>Figura 17-6: Pinando um `Box` que aponta para um tipo de future autorreferencial</figcaption>

</figure>

Na verdade, o ponteiro `Box` ainda pode se mover livremente. Lembre-se: nós nos importamos em
garantir que os dados sendo referenciados em última análise permaneçam no lugar. Se um ponteiro
se move, _mas os dados para os quais ele aponta_ estão no mesmo lugar, como na Figura
17-7, não há problema potencial. (Como um exercício independente, olhe a documentação
para os tipos, bem como o módulo `std::pin` e tente descobrir como você faria
isso com um `Pin` envolvendo um `Box`.) A chave é que o tipo autorreferencial
em si não pode se mover, porque ele ainda está fixado (pinned).

<figure>

<img alt="Quatro caixas dispostas em três colunas grosseiras, idênticas ao diagrama anterior com uma mudança na segunda coluna. Agora há duas caixas na segunda coluna, rotuladas “b1” e “b2”, “b1” está acinzentada, e a seta de “Pin” passa por “b2” em vez de “b1”, indicando que o ponteiro se moveu de “b1” para “b2”, mas os dados em “pinned” não se moveram." src="img/trpl17-07.svg" class="center" />

<figcaption>Figura 17-7: Movendo um `Box` que aponta para um tipo de future autorreferencial</figcaption>

</figure>

No entanto, a maioria dos tipos são perfeitamente seguros de mover, mesmo se eles acontecerem de estar
atrás de um ponteiro `Pin`. Só precisamos pensar sobre pinning quando itens têm
referências internas. Valores primitivos como números e Booleanos são seguros
porque eles obviamente não têm nenhuma referência interna.
Nem a maioria dos tipos com os quais você normalmente trabalha em Rust. Você pode mover
um `Vec`, por exemplo, sem se preocupar. Dado o que vimos até agora, se
você tiver um `Pin<Vec<String>>`, você teria que fazer tudo através das APIs seguras, mas
restritivas fornecidas por `Pin`, mesmo que um `Vec<String>` seja sempre seguro
para mover se não houver outras referências a ele. Precisamos de uma maneira de dizer ao
compilador que tudo bem mover itens em casos como este—e é aí que
`Unpin` entra em jogo.

`Unpin` é uma trait de marcação (marker trait), similar às traits `Send` e `Sync` que vimos no
Capítulo 16, e portanto não tem funcionalidade própria. Traits de marcação existem apenas
para dizer ao compilador que é seguro usar o tipo implementando uma dada trait em um
contexto particular. `Unpin` informa ao compilador que um dado tipo _não_
precisa manter nenhuma garantia sobre se o valor em questão pode ser seguramente
movido.

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

Assim como com `Send` e `Sync`, o compilador implementa `Unpin` automaticamente
para todos os tipos onde ele pode provar que é seguro. Um caso especial, novamente similar a
`Send` e `Sync`, é onde `Unpin` _não_ é implementada para um tipo. A
notação para isso é <code>impl !Unpin for <em>SomeType</em></code>, onde
<code><em>SomeType</em></code> é o nome de um tipo que _sim_ precisa manter
essas garantias para ser seguro sempre que um ponteiro para esse tipo for usado em um `Pin`.

Em outras palavras, há duas coisas para manter em mente sobre a relação
entre `Pin` e `Unpin`. Primeiro, `Unpin` é o caso “normal”, e `!Unpin` é
o caso especial. Segundo, se um tipo implementa `Unpin` ou `!Unpin` _apenas_
importa quando você está usando um ponteiro fixado (pinned) para esse tipo como <code>Pin<&mut
<em>SomeType</em>></code>.

Para tornar isso concreto, pense sobre uma `String`: ela tem um comprimento e os caracteres
Unicode que a compõem. Podemos envolver uma `String` em `Pin`, como visto na Figura
17-8. No entanto, `String` implementa automaticamente `Unpin`, assim como a maioria dos outros tipos
em Rust.

<figure>

<img alt="Uma caixa rotulada “Pin” à esquerda com uma seta indo dela para uma caixa rotulada “String” à direita. A caixa “String” contém o dado 5usize, representando o comprimento da string, e as letras “h”, “e”, “l”, “l”, e “o” representando os caracteres da string “hello” armazenada nesta instância de String. Um retângulo pontilhado envolve a caixa “String” e seu rótulo, mas não a caixa “Pin”." src="img/trpl17-08.svg" class="center" />

<figcaption>Figura 17-8: Pinando uma `String`; a linha pontilhada indica que a `String` implementa a trait `Unpin` e portanto não está fixada</figcaption>

</figure>

Como resultado, podemos fazer coisas que seriam ilegais se `String` implementasse
`!Unpin` em vez disso, como substituir uma string por outra no exato mesmo
local na memória como na Figura 17-9. Isso não viola o contrato de `Pin`,
porque `String` não tem referências internas que a tornem insegura para mover.
É precisamente por isso que ela implementa `Unpin` em vez de `!Unpin`.

<figure>

<img alt="O mesmo dado de string “hello” do exemplo anterior, agora rotulado “s1” e acinzentado. A caixa “Pin” do exemplo anterior agora aponta para uma instância de String diferente, uma que é rotulada “s2”, é válida, tem um comprimento de 7usize, e contém os caracteres da string “goodbye”. s2 é cercada por um retângulo pontilhado porque ela, também, implementa a trait Unpin." src="img/trpl17-09.svg" class="center" />

<figcaption>Figura 17-9: Substituindo a `String` por uma `String` inteiramente diferente na memória</figcaption>

</figure>

Agora sabemos o suficiente para entender os erros relatados para aquela chamada `join_all`
da Listagem 17-23. Originalmente tentamos mover os futures produzidos por
blocos async para um `Vec<Box<dyn Future<Output = ()>>>`, mas como vimos,
esses futures podem ter referências internas, então eles não implementam automaticamente
`Unpin`. Uma vez que os fixamos, podemos passar o tipo `Pin` resultante para
o `Vec`, confiantes de que os dados subjacentes nos futures _não_ serão
movidos. A Listagem 17-24 mostra como corrigir o código chamando a macro `pin!`
onde cada um dos três futures são definidos e ajustando o tipo de trait object.

<Listing number="17-24" caption="Pinando os futures para permitir movê-los para o vetor">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

Este exemplo agora compila e roda, e poderíamos adicionar ou remover futures do
vetor em tempo de execução e juntar todos eles.

`Pin` e `Unpin` são principalmente importantes para construir bibliotecas de nível mais baixo, ou
quando você está construindo um runtime em si, em vez de para código Rust do dia a dia.
Quando você vir essas traits em mensagens de erro, no entanto, agora você terá uma melhor
ideia de como corrigir seu código!

> Nota: Essa combinação de `Pin` e `Unpin` torna possível implementar com segurança
> toda uma classe de tipos complexos em Rust que de outra forma se provariam
> desafiadores porque são autorreferenciais. Tipos que requerem `Pin` aparecem
> mais comumente em Rust async hoje, mas de vez em quando, você pode vê-los
> em outros contextos também.
>
> As especificidades de como `Pin` e `Unpin` funcionam, e as regras que eles são obrigados
> a manter, são cobertas extensivamente na documentação da API para `std::pin`, então
> se você estiver interessado em aprender mais, esse é um ótimo lugar para começar.
>
> Se você quiser entender como as coisas funcionam "por baixo dos panos" em ainda mais detalhes,
> veja os Capítulos [2][under-the-hood]<!-- ignore --> e
> [4][pinning]<!-- ignore --> do
> [_Asynchronous Programming in Rust_][async-book].

### A Trait `Stream`

Agora que você tem uma compreensão mais profunda das traits `Future`, `Pin`, e `Unpin`, nós
podemos voltar nossa atenção para a trait `Stream`. Como você aprendeu anteriormente no
capítulo, streams são similares a iteradores assíncronos. Diferente de `Iterator` e
`Future`, no entanto, `Stream` não tem definição na biblioteca padrão até
o momento desta escrita, mas _existe_ uma definição muito comum do crate `futures`
usada em todo o ecossistema.

Vamos revisar as definições das traits `Iterator` e `Future` antes de
olhar como uma trait `Stream` pode fundi-las. De `Iterator`, nós
temos a ideia de uma sequência: seu método `next` fornece um
`Option<Self::Item>`. De `Future`, temos a ideia de prontidão ao longo do tempo:
seu método `poll` fornece um `Poll<Self::Output>`. Para representar uma sequência de
itens que se tornam prontos ao longo do tempo, definimos uma trait `Stream` que coloca essas
funcionalidades juntas:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

A trait `Stream` define um tipo associado chamado `Item` para o tipo dos
itens produzidos pelo stream. Isso é similar a `Iterator`, onde pode haver
de zero a muitos itens, e diferente de `Future`, onde há sempre um único
`Output`, mesmo que seja o tipo unitário `()`.

`Stream` também define um método para obter esses itens. Nós o chamamos de `poll_next`, para
deixar claro que ele consulta (polls) da mesma maneira que `Future::poll` faz e produz uma
sequência de itens da mesma maneira que `Iterator::next` faz. Seu tipo de retorno
combina `Poll` com `Option`. O tipo externo é `Poll`, porque ele tem que ser
verificado quanto à prontidão, assim como um future faz. O tipo interno é `Option`,
porque ele precisa sinalizar se há mais mensagens, assim como um iterador
faz.

Algo muito similar a esta definição provavelmente acabará como parte da
biblioteca padrão do Rust. Enquanto isso, é parte do kit de ferramentas da maioria dos runtimes,
então você pode contar com isso, e tudo o que cobrimos a seguir deve se aplicar geralmente!

Nos exemplos que vimos na seção [“Streams: Futures em Sequência”][streams]<!--
ignore -->, no entanto, não usamos `poll_next` _ou_ `Stream`, mas
em vez disso usamos `next` e `StreamExt`. Nós _poderíamos_ trabalhar diretamente em termos da
API `poll_next` escrevendo manualmente nossas próprias máquinas de estado `Stream`, é claro,
assim como _poderíamos_ trabalhar com futures diretamente via seu método `poll`. Usar
`await` é muito mais agradável, no entanto, e a trait `StreamExt` fornece o método `next`
para que possamos fazer exatamente isso:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> Nota: A definição real que usamos anteriormente no capítulo parece ligeiramente
> diferente desta, porque ela suporta versões do Rust que ainda não
> suportavam usar funções async em traits. Como resultado, ela se parece com isso:
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> Aquele tipo `Next` é uma `struct` que implementa `Future` e nos permite nomear
> o lifetime da referência a `self` com `Next<'_, Self>`, para que `await`
> possa funcionar com este método.

A trait `StreamExt` é também o lar de todos os métodos interessantes disponíveis
para usar com streams. `StreamExt` é automaticamente implementada para todo tipo
que implementa `Stream`, mas essas traits são definidas separadamente para permitir que a
comunidade itere em APIs de conveniência sem afetar a trait
fundamental.

Na versão de `StreamExt` usada no crate `trpl`, a trait não apenas
define o método `next` mas também fornece uma implementação padrão de `next`
que lida corretamente com os detalhes de chamar `Stream::poll_next`. Isso significa
que mesmo quando você precisa escrever seu próprio tipo de dado de streaming, você _apenas_ tem
que implementar `Stream`, e então qualquer um que use seu tipo de dado pode usar
`StreamExt` e seus métodos com ele automaticamente.

Isso é tudo o que vamos cobrir para os detalhes de nível mais baixo sobre essas traits. Para
encerrar, vamos considerar como futures (incluindo streams), tarefas e threads se
encaixam!

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-between-two-tasks-using-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
[streams]: ch17-04-streams.html
