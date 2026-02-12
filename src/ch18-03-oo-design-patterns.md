## Implementando um Padrão de Projeto Orientado a Objetos

O _padrão de estado_ (state pattern) é um padrão de projeto orientado a objetos. O cerne do
padrão é que definimos um conjunto de estados que um valor pode ter internamente. Os
estados são representados por um conjunto de _objetos de estado_, e o comportamento do valor
muda baseado em seu estado. Vamos trabalhar através de um exemplo de uma struct de postagem de blog
que tem um campo para segurar seu estado, que será um objeto de estado
do conjunto “rascunho” (draft), “revisão” (review), ou “publicado” (published).

Os objetos de estado compartilham funcionalidade: Em Rust, é claro, usamos structs e
traits em vez de objetos e herança. Cada objeto de estado é responsável
por seu próprio comportamento e por governar quando deve mudar para outro
estado. O valor que segura um objeto de estado não sabe nada sobre o comportamento
diferente dos estados ou quando transicionar entre estados.

A vantagem de usar o padrão de estado é que, quando os requisitos de negócio
do programa mudam, não precisaremos mudar o código do
valor segurando o estado ou o código que usa o valor. Só precisaremos
atualizar o código dentro de um dos objetos de estado para mudar suas regras ou talvez
adicionar mais objetos de estado.

Primeiro, vamos implementar o padrão de estado de uma maneira mais tradicionalmente
orientada a objetos. Então, usaremos uma abordagem que é um pouco mais natural em
Rust. Vamos mergulhar para implementar incrementalmente um fluxo de trabalho de postagem de blog usando o
padrão de estado.

A funcionalidade final se parecerá com isso:

1. Uma postagem de blog começa como um rascunho vazio.
2. Quando o rascunho está pronto, uma revisão da postagem é solicitada.
3. Quando a postagem é aprovada, ela é publicada.
4. Apenas postagens de blog publicadas retornam conteúdo para imprimir, para que postagens não aprovadas
   não possam ser acidentalmente publicadas.

Quaisquer outras mudanças tentadas em uma postagem não devem ter efeito. Por exemplo, se
tentarmos aprovar um rascunho de postagem de blog antes de termos solicitado uma revisão, a postagem
deve permanecer um rascunho não publicado.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### Tentando Estilo Orientado a Objetos Tradicional

Existem infinitas maneiras de estruturar código para resolver o mesmo problema, cada uma com
diferentes compensações. A implementação desta seção é mais de um estilo tradicional
orientado a objetos, que é possível escrever em Rust, mas não tira
vantagem de alguns dos pontos fortes do Rust. Mais tarde, demonstraremos uma solução diferente
que ainda usa o padrão de projeto orientado a objetos, mas é estruturada
de uma maneira que pode parecer menos familiar para programadores com experiência
orientada a objetos. Compararemos as duas soluções para experimentar as compensações de
projetar código Rust diferentemente de código em outras linguagens.

A Listagem 18-11 mostra este fluxo de trabalho em forma de código: Este é um exemplo de uso da
API que implementaremos em um crate de biblioteca chamado `blog`. Isso não compilará ainda
porque não implementamos o crate `blog`.

<Listing number="18-11" file-name="src/main.rs" caption="Código que demonstra o comportamento desejado que queremos que nosso crate `blog` tenha">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Queremos permitir que o usuário crie uma nova postagem de blog de rascunho com `Post::new`. Nós
queremos permitir que texto seja adicionado à postagem de blog. Se tentarmos obter o conteúdo da postagem
imediatamente, antes da aprovação, não devemos obter nenhum texto porque a
postagem ainda é um rascunho. Adicionamos `assert_eq!` no código para fins de demonstração.
Um excelente teste unitário para isso seria afirmar que um rascunho de postagem de blog
retorna uma string vazia do método `content`, mas não vamos
escrever testes para este exemplo.

Em seguida, queremos permitir uma solicitação de revisão da postagem, e queremos que
`content` retorne uma string vazia enquanto espera pela revisão. Quando a postagem
recebe aprovação, ela deve ser publicada, significando que o texto da postagem
será retornado quando `content` for chamado.

Note que o único tipo com o qual estamos interagindo do crate é o tipo `Post`.
Este tipo usará o padrão de estado e segurará um valor que será
um de três objetos de estado representando os vários estados que uma postagem pode
estar—rascunho, revisão, ou publicado. Mudar de um estado para outro será
gerenciado internamente dentro do tipo `Post`. Os estados mudam em resposta aos
métodos chamados pelos usuários da nossa biblioteca na instância de `Post`, mas eles não
têm que gerenciar as mudanças de estado diretamente. Além disso, usuários não podem cometer um erro
com os estados, tal como publicar uma postagem antes de ser revisada.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### Definindo `Post` e Criando uma Nova Instância

Vamos começar a implementação da biblioteca! Sabemos que precisamos de uma
struct pública `Post` que segura algum conteúdo, então começaremos com a
definição da struct e uma função pública associada `new` para criar uma
instância de `Post`, como mostrado na Listagem 18-12. Também faremos uma trait privada
`State` que definirá o comportamento que todos os objetos de estado para um `Post`
devem ter.

Então, `Post` segurará um objeto de trait de `Box<dyn State>` dentro de um `Option<T>`
em um campo privado chamado `state` para segurar o objeto de estado. Você verá por que o
`Option<T>` é necessário em um instante.

<Listing number="18-12" file-name="src/lib.rs" caption="Definição de uma struct `Post` e uma função `new` que cria uma nova instância de `Post`, uma trait `State`, e uma struct `Draft`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

A trait `State` define o comportamento compartilhado por diferentes estados de postagem. Os
objetos de estado são `Draft`, `PendingReview`, e `Published`, e todos eles
implementarão a trait `State`. Por enquanto, a trait não tem nenhum método, e
começaremos definindo apenas o estado `Draft` porque esse é o estado em que
queremos que uma postagem comece.

Quando criamos um novo `Post`, definimos seu campo `state` para um valor `Some` que
segura um `Box`. Este `Box` aponta para uma nova instância da struct `Draft`. Isso
garante que sempre que criamos uma nova instância de `Post`, ela começará
como um rascunho. Porque o campo `state` de `Post` é privado, não há maneira de
criar um `Post` em qualquer outro estado! Na função `Post::new`, definimos o
campo `content` para uma nova `String` vazia.

#### Armazenando o Texto do Conteúdo da Postagem

Vimos na Listagem 18-11 que queremos ser capazes de chamar um método chamado
`add_text` e passar a ele um `&str` que é então adicionado como o conteúdo de texto da
postagem de blog. Implementamos isso como um método, em vez de expor o campo `content`
como `pub`, para que mais tarde possamos implementar um método que controlará como
os dados do campo `content` são lidos. O método `add_text` é bastante
direto, então vamos adicionar a implementação na Listagem 18-13 ao bloco `impl Post`.

<Listing number="18-13" file-name="src/lib.rs" caption="Implementando o método `add_text` para adicionar texto ao `content` de uma postagem">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

O método `add_text` recebe uma referência mutável para `self` porque estamos
mudando a instância de `Post` na qual estamos chamando `add_text`. Então chamamos
`push_str` na `String` em `content` e passamos o argumento `text` para adicionar ao
`content` salvo. Esse comportamento não depende do estado em que a postagem está,
então não é parte do padrão de estado. O método `add_text` não interage
com o campo `state` de forma alguma, mas é parte do comportamento que queremos
suportar.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### Garantindo que o Conteúdo de um Rascunho de Postagem Esteja Vazio

Mesmo depois de termos chamado `add_text` e adicionado algum conteúdo à nossa postagem, ainda
queremos que o método `content` retorne uma fatia de string vazia porque a postagem
ainda está no estado de rascunho, como mostrado pelo primeiro `assert_eq!` na Listagem 18-11.
Por enquanto, vamos implementar o método `content` com a coisa mais simples que
cumprirá este requisito: sempre retornando uma fatia de string vazia. Mudaremos
isso mais tarde, uma vez que implementarmos a capacidade de mudar o estado de uma postagem para que
ela possa ser publicada. Até agora, postagens podem estar apenas no estado de rascunho, então o conteúdo da postagem
deve ser sempre vazio. A Listagem 18-14 mostra esta implementação de espaço reservado.

<Listing number="18-14" file-name="src/lib.rs" caption="Adicionando uma implementação de espaço reservado para o método `content` em `Post` que sempre retorna uma fatia de string vazia">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Com este método `content` adicionado, tudo na Listagem 18-11 até o primeiro
`assert_eq!` funciona como pretendido.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### Solicitando uma Revisão, Que Muda o Estado da Postagem

Em seguida, precisamos adicionar funcionalidade para solicitar uma revisão de uma postagem, o que deve
mudar seu estado de `Draft` para `PendingReview`. A Listagem 18-15 mostra este código.

<Listing number="18-15" file-name="src/lib.rs" caption="Implementando métodos `request_review` em `Post` e na trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Damos a `Post` um método público chamado `request_review` que receberá uma referência mutável
para `self`. Então, chamamos um método interno `request_review` no
estado atual de `Post`, e este segundo método `request_review` consome o
estado atual e retorna um novo estado.

Adicionamos o método `request_review` à trait `State`; todos os tipos que
implementam a trait agora precisarão implementar o método `request_review`.
Note que em vez de ter `self`, `&self`, ou `&mut self` como o primeiro
parâmetro do método, temos `self: Box<Self>`. Essa sintaxe significa que o
método só é válido quando chamado em um `Box` segurando o tipo. Essa sintaxe toma
posse de `Box<Self>`, invalidando o estado antigo para que o valor de estado do
`Post` possa se transformar em um novo estado.

Para consumir o estado antigo, o método `request_review` precisa tomar posse
do valor de estado. É aqui que o `Option` no campo `state` de `Post`
entra: Chamamos o método `take` para tirar o valor `Some` do campo `state`
e deixar um `None` em seu lugar porque Rust não nos deixa ter
campos não preenchidos em structs. Isso nos permite mover o valor `state` para fora de
`Post` em vez de emprestá-lo. Então, definiremos o valor de `state` da postagem para
o resultado desta operação.

Precisamos definir `state` como `None` temporariamente em vez de defini-lo diretamente
com código como `self.state = self.state.request_review();` para obter posse do
valor `state`. Isso garante que `Post` não possa usar o valor antigo de `state`
depois de termos transformado-o em um novo estado.

O método `request_review` em `Draft` retorna uma nova instância empacotada (boxed) de uma nova
struct `PendingReview`, que representa o estado quando uma postagem está esperando por uma
revisão. A struct `PendingReview` também implementa o método `request_review`
mas não faz nenhuma transformação. Em vez disso, ela retorna a si mesma porque quando nós
solicitamos uma revisão em uma postagem já no estado `PendingReview`, ela deve permanecer
no estado `PendingReview`.

Agora podemos começar a ver as vantagens do padrão de estado: O método
`request_review` em `Post` é o mesmo não importa seu valor de `state`. Cada
estado é responsável por suas próprias regras.

Deixaremos o método `content` em `Post` como está, retornando uma fatia de string
vazia. Agora podemos ter um `Post` no estado `PendingReview` bem como no
estado `Draft`, mas queremos o mesmo comportamento no estado `PendingReview`.
A Listagem 18-11 agora funciona até a segunda chamada `assert_eq!`!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### Adicionando `approve` para Mudar o Comportamento de `content`

O método `approve` será similar ao método `request_review`: Ele irá
definir `state` para o valor que o estado atual diz que ele deve ter quando aquele
estado é aprovado, como mostrado na Listagem 18-16.

<Listing number="18-16" file-name="src/lib.rs" caption="Implementando o método `approve` em `Post` e na trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Adicionamos o método `approve` à trait `State` e adicionamos uma nova struct que
implementa `State`, o estado `Published`.

Similar à maneira como `request_review` em `PendingReview` funciona, se chamarmos o
método `approve` em um `Draft`, ele não terá efeito porque `approve`
retornará `self`. Quando chamamos `approve` em `PendingReview`, ele retorna uma nova
instância empacotada da struct `Published`. A struct `Published` implementa a
trait `State`, e tanto para o método `request_review` quanto para o método `approve`,
ela retorna a si mesma porque a postagem deve permanecer no estado `Published`
nesses casos.

Agora precisamos atualizar o método `content` em `Post`. Queremos que o valor
retornado de `content` dependa do estado atual do `Post`, então nós vamos
fazer o `Post` delegar para um método `content` definido em seu `state`,
como mostrado na Listagem 18-17.

<Listing number="18-17" file-name="src/lib.rs" caption="Atualizando o método `content` em `Post` para delegar para um método `content` em `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Porque o objetivo é manter todas essas regras dentro das structs que
implementam `State`, chamamos um método `content` no valor em `state` e passamos
a instância da postagem (isto é, `self`) como um argumento. Então, retornamos o valor
que é retornado do uso do método `content` no valor `state`.

Chamamos o método `as_ref` no `Option` porque queremos uma referência para o
valor dentro do `Option` em vez de posse do valor. Porque `state` é
um `Option<Box<dyn State>>`, quando chamamos `as_ref`, um `Option<&Box<dyn
State>>` é retornado. Se não chamássemos `as_ref`, obteríamos um erro porque
não podemos mover `state` para fora do `&self` emprestado do parâmetro da função.

Então chamamos o método `unwrap`, que sabemos que nunca entrará em pânico porque
sabemos que os métodos em `Post` garantem que `state` sempre conterá um valor
`Some` quando esses métodos terminarem. Este é um dos casos que falamos na
seção [“Quando Você Tem Mais Informação Que o
Compilador”][more-info-than-rustc]<!-- ignore --> do Capítulo 9 quando nós
sabemos que um valor `None` nunca é possível, mesmo que o compilador não seja capaz
de entender isso.

Neste ponto, quando chamamos `content` no `&Box<dyn State>`, a coerção deref
entrará em vigor no `&` e no `Box` para que o método `content` seja
finalmente chamado no tipo que implementa a trait `State`. Isso significa
que precisamos adicionar `content` à definição da trait `State`, e é aí que
colocaremos a lógica para qual conteúdo retornar dependendo de qual estado
temos, como mostrado na Listagem 18-18.

<Listing number="18-18" file-name="src/lib.rs" caption="Adicionando o método `content` à trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Adicionamos uma implementação padrão para o método `content` que retorna uma
fatia de string vazia. Isso significa que não precisamos implementar `content` nas structs
`Draft` e `PendingReview`. A struct `Published` sobrescreverá o método `content`
e retornará o valor em `post.content`. Embora conveniente, ter o
método `content` em `State` determinando o conteúdo do `Post` está borrando
as linhas entre a responsabilidade de `State` e a responsabilidade de
`Post`.

Note que precisamos de anotações de tempo de vida (lifetime annotations) neste método, como discutimos no
Capítulo 10. Estamos tomando uma referência para um `post` como um argumento e retornando uma
referência para parte desse `post`, então o tempo de vida da referência retornada é
relacionado ao tempo de vida do argumento `post`.

E terminamos—tudo na Listagem 18-11 agora funciona! Implementamos o padrão
de estado com as regras do fluxo de trabalho de postagem de blog. A lógica relacionada às
regras vive nos objetos de estado em vez de estar espalhada por todo `Post`.

> ### Por Que Não Um Enum?
>
> Você pode ter se perguntado por que não usamos um enum com os diferentes
> estados possíveis da postagem como variantes. Essa é certamente uma solução possível; tente
> e compare os resultados finais para ver qual você prefere! Uma desvantagem de usar
> um enum é que todo lugar que verifica o valor do enum precisará de uma
> expressão `match` ou similar para lidar com cada variante possível. Isso poderia ficar
> mais repetitivo do que esta solução de objeto de trait.

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### Avaliando o Padrão de Estado

Mostramos que Rust é capaz de implementar o padrão de estado orientado a objetos
para encapsular os diferentes tipos de comportamento que uma postagem deve ter em
cada estado. Os métodos em `Post` não sabem nada sobre os vários comportamentos.
Por causa da maneira como organizamos o código, temos que olhar em apenas um lugar para
saber as diferentes maneiras que uma postagem publicada pode se comportar: a implementação da
trait `State` na struct `Published`.

Se fôssemos criar uma implementação alternativa que não usasse o padrão de estado,
poderíamos em vez disso usar expressões `match` nos métodos em `Post` ou
mesmo no código `main` que verifica o estado da postagem e muda o comportamento
nesses lugares. Isso significaria que teríamos que olhar em vários lugares para
entender todas as implicações de uma postagem estar no estado publicado.

Com o padrão de estado, os métodos `Post` e os lugares que usamos `Post` não
precisam de expressões `match`, e para adicionar um novo estado, precisaríamos apenas adicionar uma
nova struct e implementar os métodos de trait nessa struct em um local.

A implementação usando o padrão de estado é fácil de estender para adicionar mais
funcionalidade. Para ver a simplicidade de manter código que usa o padrão
de estado, tente algumas destas sugestões:

- Adicione um método `reject` que muda o estado da postagem de `PendingReview` de volta
  para `Draft`.
- Exija duas chamadas para `approve` antes que o estado possa ser mudado para `Published`.
- Permita que usuários adicionem conteúdo de texto apenas quando uma postagem está no estado `Draft`.
  Dica: tenha o objeto de estado responsável pelo que pode mudar sobre o
  conteúdo, mas não responsável por modificar o `Post`.

Uma desvantagem do padrão de estado é que, porque os estados implementam as
transições entre estados, alguns dos estados são acoplados uns aos outros. Se nós
adicionarmos outro estado entre `PendingReview` e `Published`, tal como `Scheduled`,
teríamos que mudar o código em `PendingReview` para transicionar para
`Scheduled` em vez disso. Seria menos trabalho se `PendingReview` não precisasse
mudar com a adição de um novo estado, mas isso significaria mudar para
outro padrão de projeto.

Outra desvantagem é que duplicamos alguma lógica. Para eliminar parte da
duplicação, poderíamos tentar fazer implementações padrão para os métodos
`request_review` e `approve` na trait `State` que retornam `self`.
No entanto, isso não funcionaria: Quando usamos `State` como um objeto de trait, a trait
não sabe o que o `self` concreto será exatamente, então o tipo de retorno não é
conhecido em tempo de compilação. (Esta é uma das regras de compatibilidade dyn mencionadas
anteriormente.)

Outra duplicação inclui as implementações similares dos métodos `request_review`
e `approve` em `Post`. Ambos os métodos usam `Option::take` com o
campo `state` de `Post`, e se `state` for `Some`, eles delegam para a implementação
do valor envolvido do mesmo método e definem o novo valor do campo `state`
para o resultado. Se tivéssemos muitos métodos em `Post` que seguissem esse
padrão, poderíamos considerar definir uma macro para eliminar a repetição (veja
a seção [“Macros”][macros]<!-- ignore --> no Capítulo 20).

Ao implementar o padrão de estado exatamente como é definido para linguagens
orientadas a objetos, não estamos tirando vantagem total dos pontos fortes do Rust como poderíamos.
Vamos olhar para algumas mudanças que podemos fazer no crate `blog` que podem tornar
estados e transições inválidos em erros de tempo de compilação.

### Codificando Estados e Comportamento como Tipos

Mostraremos como repensar o padrão de estado para obter um conjunto diferente de
compensações. Em vez de encapsular os estados e transições completamente para que
código externo não tenha conhecimento deles, codificaremos os estados em
tipos diferentes. Consequentemente, o sistema de verificação de tipos do Rust impedirá
tentativas de usar postagens de rascunho onde apenas postagens publicadas são permitidas emitindo um
erro de compilador.

Vamos considerar a primeira parte de `main` na Listagem 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Ainda habilitamos a criação de novas postagens no estado de rascunho usando `Post::new`
e a capacidade de adicionar texto ao conteúdo da postagem. Mas em vez de ter um
método `content` em um rascunho de postagem que retorna uma string vazia, faremos com que
rascunhos de postagens não tenham o método `content` de forma alguma. Dessa forma, se tentarmos
obter o conteúdo de um rascunho de postagem, obteremos um erro de compilador nos dizendo que o método
não existe. Como resultado, será impossível para nós acidentalmente
exibir conteúdo de rascunho de postagem em produção porque esse código nem mesmo compilará.
A Listagem 18-19 mostra a definição de uma struct `Post` e uma struct `DraftPost`,
bem como métodos em cada uma.

<Listing number="18-19" file-name="src/lib.rs" caption="Um `Post` com um método `content` e um `DraftPost` sem um método `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Ambas as structs `Post` e `DraftPost` têm um campo `content` privado que
armazena o texto da postagem de blog. As structs não têm mais o campo `state` porque
estamos movendo a codificação do estado para os tipos das structs. A struct `Post`
representará uma postagem publicada, e tem um método `content` que
retorna o `content`.

Ainda temos uma função `Post::new`, mas em vez de retornar uma instância de
`Post`, ela retorna uma instância de `DraftPost`. Porque `content` é privado e
não há funções que retornam `Post`, não é possível criar uma
instância de `Post` agora.

A struct `DraftPost` tem um método `add_text`, para que possamos adicionar texto ao
`content` como antes, mas note que `DraftPost` não tem um método `content`
definido! Então agora o programa garante que todas as postagens comecem como rascunhos de postagens, e
rascunhos de postagens não têm seu conteúdo disponível para exibição. Qualquer tentativa de contornar
essas restrições resultará em um erro de compilador.

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

Então, como obtemos uma postagem publicada? Queremos impor a regra de que um rascunho
de postagem tem que ser revisado e aprovado antes que possa ser publicado. Uma postagem no
estado de revisão pendente ainda não deve exibir nenhum conteúdo. Vamos implementar
essas restrições adicionando outra struct, `PendingReviewPost`, definindo o
método `request_review` em `DraftPost` para retornar um `PendingReviewPost` e
definindo um método `approve` em `PendingReviewPost` para retornar um `Post`, como
mostrado na Listagem 18-20.

<Listing number="18-20" file-name="src/lib.rs" caption="Um `PendingReviewPost` que é criado chamando `request_review` em `DraftPost` e um método `approve` que transforma um `PendingReviewPost` em um `Post` publicado">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

Os métodos `request_review` e `approve` tomam posse de `self`, assim
consumindo as instâncias `DraftPost` e `PendingReviewPost` e transformando
elas em um `PendingReviewPost` e um `Post` publicado, respectivamente. Dessa forma,
não teremos nenhuma instância `DraftPost` persistente depois de termos chamado
`request_review` nelas, e assim por diante. A struct `PendingReviewPost` não
tem um método `content` definido nela, então tentar ler seu conteúdo
resulta em um erro de compilador, como com `DraftPost`. Porque a única maneira de obter uma
instância `Post` publicada que tem um método `content` definido é chamar
o método `approve` em um `PendingReviewPost`, e a única maneira de obter um
`PendingReviewPost` é chamar o método `request_review` em um `DraftPost`,
nós agora codificamos o fluxo de trabalho de postagem de blog no sistema de tipos.

Mas também temos que fazer algumas pequenas mudanças em `main`. Os métodos `request_review` e
`approve` retornam novas instâncias em vez de modificar a struct na qual são
chamados, então precisamos adicionar mais atribuições de sombreamento (shadowing) `let post =` para salvar
as instâncias retornadas. Também não podemos ter as afirmações sobre os conteúdos dos rascunhos e
postagens de revisão pendente serem strings vazias, nem precisamos delas: Não podemos
compilar código que tenta usar o conteúdo de postagens nesses estados mais.
O código atualizado em `main` é mostrado na Listagem 18-21.

<Listing number="18-21" file-name="src/main.rs" caption="Modificações em `main` para usar a nova implementação do fluxo de trabalho de postagem de blog">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

As mudanças que precisamos fazer em `main` para reatribuir `post` significam que esta
implementação não segue exatamente o padrão de estado orientado a objetos mais:
As transformações entre os estados não são mais encapsuladas inteiramente
dentro da implementação de `Post`. No entanto, nosso ganho é que estados inválidos são
agora impossíveis por causa do sistema de tipos e da verificação de tipos que acontece em
tempo de compilação! Isso garante que certos bugs, como exibição do conteúdo de
uma postagem não publicada, serão descobertos antes de chegarem à produção.

Tente as tarefas sugeridas no início desta seção no crate `blog` como ele
está após a Listagem 18-21 para ver o que você pensa sobre o design desta versão
do código. Note que algumas das tarefas podem já estar completas neste
design.

Vimos que, embora Rust seja capaz de implementar padrões de projeto orientados a
objetos, outros padrões, como codificar estado no sistema de tipos,
também estão disponíveis em Rust. Esses padrões têm compensações diferentes. Embora
você possa estar muito familiarizado com padrões orientados a objetos, repensar o
problema para tirar vantagem das funcionalidades do Rust pode fornecer benefícios, como
prevenir alguns bugs em tempo de compilação. Padrões orientados a objetos nem sempre serão
a melhor solução em Rust devido a certas funcionalidades, como posse (ownership), que
linguagens orientadas a objetos não têm.

## Resumo

Independentemente de se você pensa que Rust é uma linguagem orientada a objetos depois
de ler este capítulo, agora você sabe que pode usar objetos de trait para obter algumas
funcionalidades orientadas a objetos em Rust. Despacho dinâmico pode dar ao seu código alguma
flexibilidade em troca de um pouco de desempenho em tempo de execução. Você pode usar essa
flexibilidade para implementar padrões orientados a objetos que podem ajudar na manutenibilidade
do seu código. Rust também tem outras funcionalidades, como posse, que
linguagens orientadas a objetos não têm. Um padrão orientado a objetos nem sempre
será a melhor maneira de tirar vantagem dos pontos fortes do Rust, mas é uma opção
disponível.

A seguir, olharemos para padrões, que são outra das funcionalidades do Rust que permitem
muita flexibilidade. Olhamos para eles brevemente ao longo do livro mas
não vimos sua capacidade total ainda. Vamos lá!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
