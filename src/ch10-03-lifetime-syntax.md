## Validando Referências com Lifetimes

Lifetimes (tempos de vida) são outro tipo de genérico que já usamos. Em vez de
garantir que um tipo tenha o comportamento que queremos, lifetimes garantem que
as referências sejam válidas pelo tempo que precisarmos delas.

Um detalhe que não discutimos na seção [“Referências e Empréstimos”][references-and-borrowing]<!-- ignore -->
no Capítulo 4 é que toda referência em Rust tem um lifetime, que é o escopo
para o qual essa referência é válida. Na maioria das vezes, os lifetimes são
implícitos e inferidos, assim como na maioria das vezes, os tipos são
inferidos. Só somos obrigados a anotar tipos quando vários tipos são possíveis.
De maneira semelhante, devemos anotar lifetimes quando os lifetimes das
referências podem estar relacionados de algumas maneiras diferentes. Rust exige
que anotemos os relacionamentos usando parâmetros de lifetime genéricos para
garantir que as referências reais usadas em tempo de execução sejam
definitivamente válidas.

Anotar lifetimes não é nem mesmo um conceito que a maioria das outras
linguagens de programação tem, então isso vai parecer estranho. Embora não
cubramos lifetimes em sua totalidade neste capítulo, discutiremos maneiras
comuns de encontrar a sintaxe de lifetime para que você possa se familiarizar
com o conceito.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### Referências Pendentes (Dangling References)

O principal objetivo dos lifetimes é evitar referências pendentes, que, se
fossem permitidas, fariam com que um programa referenciasse dados diferentes
dos dados que pretende referenciar. Considere o programa na Listagem 10-16, que
tem um escopo externo e um escopo interno.

<Listing number="10-16" caption="Uma tentativa de usar uma referência cujo valor saiu de escopo">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> Nota: Os exemplos nas Listagens 10-16, 10-17 e 10-23 declaram variáveis sem
> dar a elas um valor inicial, então o nome da variável existe no escopo
> externo. À primeira vista, isso pode parecer estar em conflito com Rust não
> ter valores nulos. No entanto, se tentarmos usar uma variável antes de dar a
> ela um valor, obteremos um erro em tempo de compilação, o que mostra que, de
> fato, Rust não permite valores nulos.

O escopo externo declara uma variável chamada `r` sem valor inicial, e o escopo
interno declara uma variável chamada `x` com o valor inicial de `5`. Dentro do
escopo interno, tentamos definir o valor de `r` como uma referência a `x`. Em
seguida, o escopo interno termina e tentamos imprimir o valor em `r`. Este
código não compilará, porque o valor a que `r` se refere saiu de escopo antes
de tentarmos usá-lo. Aqui está a mensagem de erro:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

A mensagem de erro diz que a variável `x` “does not live long enough” (não vive
o suficiente). O motivo é que `x` estará fora de escopo quando o escopo interno
terminar na linha 7. Mas `r` ainda é válido para o escopo externo; porque seu
escopo é maior, dizemos que ele “vive mais”. Se Rust permitisse que esse código
funcionasse, `r` estaria referenciando a memória que foi desalocada quando `x`
saiu de escopo, e qualquer coisa que tentássemos fazer com `r` não funcionaria
corretamente. Então, como Rust determina que esse código é inválido? Ele usa um
verificador de empréstimo (borrow checker).

### O Verificador de Empréstimo (Borrow Checker)

O compilador Rust tem um _verificador de empréstimo_ que compara escopos para
determinar se todos os empréstimos são válidos. A Listagem 10-17 mostra o mesmo
código da Listagem 10-16, mas com anotações mostrando os lifetimes das
variáveis.

<Listing number="10-17" caption="Anotações dos lifetimes de `r` e `x`, nomeados `'a` e `'b`, respectivamente">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Aqui, anotamos o lifetime de `r` com `'a` e o lifetime de `x` com `'b`. Como
você pode ver, o bloco interno `'b` é muito menor que o bloco de lifetime
externo `'a`. Em tempo de compilação, Rust compara o tamanho dos dois lifetimes
e vê que `r` tem um lifetime de `'a`, mas que se refere à memória com um
lifetime de `'b`. O programa é rejeitado porque `'b` é menor que `'a`: O
assunto da referência não vive tanto quanto a referência.

A Listagem 10-18 corrige o código para que ele não tenha uma referência
pendente e compile sem erros.

<Listing number="10-18" caption="Uma referência válida porque os dados têm um lifetime mais longo do que a referência">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Aqui, `x` tem o lifetime `'b`, que neste caso é maior que `'a`. Isso significa
que `r` pode referenciar `x` porque Rust sabe que a referência em `r` será
sempre válida enquanto `x` for válida.

Agora que você sabe onde estão os lifetimes das referências e como Rust analisa
lifetimes para garantir que as referências sejam sempre válidas, vamos explorar
lifetimes genéricos em parâmetros de função e valores de retorno.

### Lifetimes Genéricos em Funções

Escreveremos uma função que retorna a maior de duas fatias de string. Esta
função receberá duas fatias de string e retornará uma única fatia de string.
Depois de implementarmos a função `longest`, o código na Listagem 10-19 deve
imprimir `A string mais longa é abcd`.

<Listing number="10-19" file-name="src/main.rs" caption="Uma função `main` que chama a função `longest` para encontrar a maior de duas fatias de string">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

Observe que queremos que a função receba fatias de string, que são referências,
em vez de strings, porque não queremos que a função `longest` tome posse de
seus parâmetros. Consulte [“Fatias de String como Parâmetros”][string-slices-as-parameters]<!-- ignore -->
no Capítulo 4 para mais discussões sobre por que os parâmetros que usamos na
Listagem 10-19 são os que queremos.

Se tentarmos implementar a função `longest` como mostrado na Listagem 10-20, ela
não compilará.

<Listing number="10-20" file-name="src/main.rs" caption="Uma implementação da função `longest` que retorna a maior de duas fatias de string, mas ainda não compila">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

Em vez disso, recebemos o seguinte erro que fala sobre lifetimes:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

O texto de ajuda revela que o tipo de retorno precisa de um parâmetro de
lifetime genérico porque Rust não consegue dizer se a referência retornada se
refere a `x` ou `y`. Na verdade, também não sabemos, porque o bloco `if` no
corpo desta função retorna uma referência a `x` e o bloco `else` retorna uma
referência a `y`!

Ao definir essa função, não sabemos os valores concretos que serão passados para
ela, então não sabemos se o caso `if` ou o caso `else` será executado. Também
não sabemos os lifetimes concretos das referências que serão passadas, então
não podemos olhar para os escopos como fizemos nas Listagens 10-17 e 10-18 para
determinar se a referência que retornamos será sempre válida. O verificador de
empréstimo também não pode determinar isso, porque não sabe como os lifetimes
de `x` e `y` se relacionam com o lifetime do valor de retorno. Para corrigir
esse erro, adicionaremos parâmetros de lifetime genéricos que definem o
relacionamento entre as referências para que o verificador de empréstimo possa
realizar sua análise.

### Sintaxe de Anotação de Lifetime

Anotações de lifetime não mudam quanto tempo qualquer uma das referências vive.
Em vez disso, elas descrevem os relacionamentos dos lifetimes de várias
referências entre si sem afetar os lifetimes. Assim como as funções podem
aceitar qualquer tipo quando a assinatura especifica um parâmetro de tipo
genérico, as funções podem aceitar referências com qualquer lifetime
especificando um parâmetro de lifetime genérico.

Anotações de lifetime têm uma sintaxe um pouco incomum: Os nomes dos parâmetros
de lifetime devem começar com um apóstrofo (`'`) e geralmente são todos
minúsculos e muito curtos, como tipos genéricos. A maioria das pessoas usa o
nome `'a` para a primeira anotação de lifetime. Colocamos anotações de
parâmetro de lifetime após o `&` de uma referência, usando um espaço para
separar a anotação do tipo da referência.

Aqui estão alguns exemplos — uma referência a um `i32` sem um parâmetro de
lifetime, uma referência a um `i32` que tem um parâmetro de lifetime chamado
`'a` e uma referência mutável a um `i32` que também tem o lifetime `'a`:

```rust,ignore
&i32        // uma referência
&'a i32     // uma referência com um lifetime explícito
&'a mut i32 // uma referência mutável com um lifetime explícito
```

Uma anotação de lifetime por si só não tem muito significado, porque as
anotações destinam-se a dizer ao Rust como os parâmetros de lifetime genéricos
de várias referências se relacionam entre si. Vamos examinar como as anotações
de lifetime se relacionam entre si no contexto da função `longest`.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-function-signatures"></a>

### Em Assinaturas de Função

Para usar anotações de lifetime em assinaturas de função, precisamos declarar
os parâmetros de lifetime genéricos dentro de colchetes angulares entre o nome
da função e a lista de parâmetros, assim como fizemos com parâmetros de tipo
genérico.

Queremos que a assinatura expresse a seguinte restrição: A referência retornada
será válida enquanto ambos os parâmetros forem válidos. Este é o relacionamento
entre lifetimes dos parâmetros e o valor de retorno. Vamos nomear o lifetime
`'a` e adicioná-lo a cada referência, conforme mostrado na Listagem 10-21.

<Listing number="10-21" file-name="src/main.rs" caption="A definição da função `longest` especificando que todas as referências na assinatura devem ter o mesmo lifetime `'a`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

Este código deve compilar e produzir o resultado que queremos quando o usarmos
com a função `main` na Listagem 10-19.

A assinatura da função agora diz ao Rust que, para algum lifetime `'a`, a função
recebe dois parâmetros, ambos os quais são fatias de string que vivem pelo
menos tanto tempo quanto o lifetime `'a`. A assinatura da função também diz ao
Rust que a fatia de string retornada da função viverá pelo menos tanto tempo
quanto o lifetime `'a`. Na prática, isso significa que o lifetime da referência
retornada pela função `longest` é o mesmo que o menor dos lifetimes dos valores
referidos pelos argumentos da função. Esses relacionamentos são o que queremos
que o Rust use ao analisar este código.

Lembre-se, quando especificamos os parâmetros de lifetime nesta assinatura de
função, não estamos alterando os lifetimes de quaisquer valores passados ou
retornados. Em vez disso, estamos especificando que o verificador de empréstimo
deve rejeitar quaisquer valores que não sigam essas restrições. Observe que a
função `longest` não precisa saber exatamente quanto tempo `x` e `y` viverão,
apenas que algum escopo pode ser substituído por `'a` que satisfaça esta
assinatura.

Ao anotar lifetimes em funções, as anotações vão na assinatura da função, não
no corpo da função. As anotações de lifetime tornam-se parte do contrato da
função, muito parecido com os tipos na assinatura. Ter assinaturas de função
contendo o contrato de lifetime significa que a análise que o compilador Rust
faz pode ser mais simples. Se houver um problema com a maneira como uma função
é anotada ou a maneira como ela é chamada, os erros do compilador podem apontar
para a parte do nosso código e as restrições com mais precisão. Se, em vez
disso, o compilador Rust fizesse mais inferências sobre o que pretendíamos que
fossem os relacionamentos dos lifetimes, o compilador poderia apenas apontar
para um uso do nosso código muitos passos longe da causa do problema.

Quando passamos referências concretas para `longest`, o lifetime concreto que é
substituído por `'a` é a parte do escopo de `x` que se sobrepõe ao escopo de
`y`. Em outras palavras, o lifetime genérico `'a` obterá o lifetime concreto
que é igual ao menor dos lifetimes de `x` e `y`. Como anotamos a referência
retornada com o mesmo parâmetro de lifetime `'a`, a referência retornada também
será válida pelo comprimento do menor dos lifetimes de `x` e `y`.

Vamos ver como as anotações de lifetime restringem a função `longest` passando
referências que têm lifetimes concretos diferentes. A Listagem 10-22 é um
exemplo direto.

<Listing number="10-22" file-name="src/main.rs" caption="Usando a função `longest` com referências a valores `String` que têm lifetimes concretos diferentes">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

Neste exemplo, `string1` é válida até o final do escopo externo, `string2` é
válida até o final do escopo interno, e `result` referencia algo que é válido
até o final do escopo interno. Execute este código e você verá que o
verificador de empréstimo aprova; ele compilará e imprimirá `A string mais
longa é long string is long`.

Em seguida, vamos tentar um exemplo que mostra que o lifetime da referência em
`result` deve ser o menor lifetime dos dois argumentos. Vamos mover a
declaração da variável `result` para fora do escopo interno, mas deixar a
atribuição do valor à variável `result` dentro do escopo com `string2`. Em
seguida, moveremos o `println!` que usa `result` para fora do escopo interno,
após o término do escopo interno. O código na Listagem 10-23 não compilará.

<Listing number="10-23" file-name="src/main.rs" caption="Tentando usar `result` após `string2` ter saído de escopo">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

Quando tentamos compilar este código, obtemos este erro:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

O erro mostra que, para `result` ser válido para a instrução `println!`,
`string2` precisaria ser válido até o final do escopo externo. Rust sabe disso
porque anotamos os lifetimes dos parâmetros da função e valores de retorno
usando o mesmo parâmetro de lifetime `'a`.

Como humanos, podemos olhar para este código e ver que `string1` é mais longa
que `string2` e, portanto, `result` conterá uma referência a `string1`. Como
`string1` ainda não saiu de escopo, uma referência a `string1` ainda será
válida para a instrução `println!`. No entanto, o compilador não consegue ver
que a referência é válida neste caso. Dissemos ao Rust que o lifetime da
referência retornada pela função `longest` é o mesmo que o menor dos lifetimes
das referências passadas. Portanto, o verificador de empréstimo não permite o
código na Listagem 10-23 como possivelmente tendo uma referência inválida.

Tente projetar mais experimentos que variem os valores e lifetimes das
referências passadas para a função `longest` e como a referência retornada é
usada. Faça hipóteses sobre se seus experimentos passarão ou não pelo
verificador de empréstimo antes de compilar; em seguida, verifique se você
está certo!

<!-- Old headings. Do not remove or links may break. -->

<a id="thinking-in-terms-of-lifetimes"></a>

### Relacionamentos

A maneira como você precisa especificar parâmetros de lifetime depende do que sua
função está fazendo. Por exemplo, se alterássemos a implementação da função
`longest` para retornar sempre o primeiro parâmetro em vez da fatia de string
mais longa, não precisaríamos especificar um lifetime no parâmetro `y`. O
seguinte código compilará:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

Especificamos um parâmetro de lifetime `'a` para o parâmetro `x` e o tipo de
retorno, mas não para o parâmetro `y`, porque o lifetime de `y` não tem nenhuma
relação com o lifetime de `x` ou o valor de retorno.

Ao retornar uma referência de uma função, o parâmetro de lifetime para o tipo
de retorno precisa corresponder ao parâmetro de lifetime de um dos parâmetros.
Se a referência retornada _não_ se referir a um dos parâmetros, ela deve se
referir a um valor criado dentro desta função. No entanto, isso seria uma
referência pendente porque o valor sairá de escopo no final da função.
Considere esta tentativa de implementação da função `longest` que não
compilará:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

Aqui, mesmo que tenhamos especificado um parâmetro de lifetime `'a` para o tipo
de retorno, esta implementação falhará ao compilar porque o lifetime do valor
de retorno não está relacionado ao lifetime dos parâmetros de forma alguma.
Aqui está a mensagem de erro que recebemos:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

O problema é que `result` sai de escopo e é limpo no final da função `longest`.
Também estamos tentando retornar uma referência a `result` da função. Não há
como especificar parâmetros de lifetime que mudariam a referência pendente, e
Rust não nos deixará criar uma referência pendente. Nesse caso, a melhor
correção seria retornar um tipo de dados de propriedade em vez de uma
referência para que a função de chamada seja então responsável por limpar o
valor.

Por fim, a sintaxe de lifetime é sobre conectar os lifetimes de vários
parâmetros e valores de retorno de funções. Uma vez conectados, Rust tem
informações suficientes para permitir operações seguras de memória e não
permitir operações que criariam ponteiros pendentes ou violariam de outra forma
a segurança da memória.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-struct-definitions"></a>

### Em Definições de Struct

Até agora, as structs que definimos mantêm tipos de propriedade. Podemos
definir structs para manter referências, mas nesse caso, precisaríamos
adicionar uma anotação de lifetime em cada referência na definição da struct. A
Listagem 10-24 tem uma struct chamada `ImportantExcerpt` que contém uma fatia
de string.

<Listing number="10-24" file-name="src/main.rs" caption="Uma struct que contém uma referência, exigindo uma anotação de lifetime">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

Esta struct tem o único campo `part` que contém uma fatia de string, que é uma
referência. Assim como com tipos de dados genéricos, declaramos o nome do
parâmetro de lifetime genérico dentro de colchetes angulares após o nome da
struct para que possamos usar o parâmetro de lifetime no corpo da definição da
struct. Esta anotação significa que uma instância de `ImportantExcerpt` não
pode sobreviver à referência que ela contém em seu campo `part`.

A função `main` aqui cria uma instância da struct `ImportantExcerpt` que contém
uma referência à primeira frase da `String` de propriedade da variável `novel`.
Os dados em `novel` existem antes que a instância `ImportantExcerpt` seja
criada. Além disso, `novel` não sai de escopo até depois que `ImportantExcerpt`
sai de escopo, então a referência na instância `ImportantExcerpt` é válida.

### Elisão de Lifetime

Você aprendeu que toda referência tem um lifetime e que você precisa especificar
parâmetros de lifetime para funções ou structs que usam referências. No
entanto, tivemos uma função na Listagem 4-9, mostrada novamente na Listagem
10-25, que compilou sem anotações de lifetime.

<Listing number="10-25" file-name="src/lib.rs" caption="Uma função que definimos na Listagem 4-9 que compilou sem anotações de lifetime, embora o parâmetro e o tipo de retorno sejam referências">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

A razão pela qual esta função compila sem anotações de lifetime é histórica: Em
versões iniciais (pré-1.0) de Rust, este código não teria compilado, porque
toda referência precisava de um lifetime explícito. Naquela época, a assinatura
da função teria sido escrita assim:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Depois de escrever muito código Rust, a equipe Rust descobriu que os
programadores Rust estavam inserindo as mesmas anotações de lifetime repetidas
vezes em situações específicas. Essas situações eram previsíveis e seguiam
alguns padrões determinísticos. Os desenvolvedores programaram esses padrões no
código do compilador para que o verificador de empréstimo pudesse inferir os
lifetimes nessas situações e não precisasse de anotações explícitas.

Esta parte da história do Rust é relevante porque é possível que mais padrões
determinísticos surjam e sejam adicionados ao compilador. No futuro, ainda
menos anotações de lifetime podem ser necessárias.

Os padrões programados na análise de referências do Rust são chamados de
_regras de elisão de lifetime_. Estas não são regras para programadores
seguirem; elas são um conjunto de casos particulares que o compilador
considerará, e se o seu código se encaixar nesses casos, você não precisará
escrever os lifetimes explicitamente.

As regras de elisão não fornecem inferência completa. Se ainda houver
ambiguidade sobre quais lifetimes as referências têm depois que Rust aplica as
regras, o compilador não adivinhará qual deve ser o lifetime das referências
restantes. Em vez de adivinhar, o compilador lhe dará um erro que você pode
resolver adicionando as anotações de lifetime.

Lifetimes em parâmetros de função ou método são chamados de _lifetimes de
entrada_, e lifetimes em valores de retorno são chamados de _lifetimes de
saída_.

O compilador usa três regras para descobrir os lifetimes das referências quando
não há anotações explícitas. A primeira regra se aplica aos lifetimes de
entrada, e a segunda e terceira regras se aplicam aos lifetimes de saída. Se o
compilador chegar ao final das três regras e ainda houver referências para as
quais ele não consegue descobrir os lifetimes, o compilador parará com um erro.
Essas regras se aplicam a definições `fn` bem como a blocos `impl`.

A primeira regra é que o compilador atribui um parâmetro de lifetime a cada
parâmetro que é uma referência. Em outras palavras, uma função com um parâmetro
obtém um parâmetro de lifetime: `fn foo<'a>(x: &'a i32)`; uma função com dois
parâmetros obtém dois parâmetros de lifetime separados: `fn foo<'a, 'b>(x: &'a
i32, y: &'b i32)`; e assim por diante.

A segunda regra é que, se houver exatamente um parâmetro de lifetime de
entrada, esse lifetime é atribuído a todos os parâmetros de lifetime de saída:
`fn foo<'a>(x: &'a i32) -> &'a i32`.

A terceira regra é que, se houver vários parâmetros de lifetime de entrada, mas
um deles for `&self` ou `&mut self` porque este é um método, o lifetime de
`self` é atribuído a todos os parâmetros de lifetime de saída. Esta terceira
regra torna os métodos muito mais agradáveis de ler e escrever porque menos
símbolos são necessários.

Vamos fingir que somos o compilador. Aplicaremos essas regras para descobrir os
lifetimes das referências na assinatura da função `first_word` na Listagem
10-25. A assinatura começa sem quaisquer lifetimes associados às referências:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Então, o compilador aplica a primeira regra, que especifica que cada parâmetro
obtém seu próprio lifetime. Vamos chamá-lo de `'a` como de costume, então agora
a assinatura é esta:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

A segunda regra se aplica porque há exatamente um lifetime de entrada. A
segunda regra especifica que o lifetime do único parâmetro de entrada é
atribuído ao lifetime de saída, então a assinatura agora é esta:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Agora todas as referências nesta assinatura de função têm lifetimes, e o
compilador pode continuar sua análise sem precisar que o programador anote os
lifetimes nesta assinatura de função.

Vamos olhar para outro exemplo, desta vez usando a função `longest` que não
tinha parâmetros de lifetime quando começamos a trabalhar com ela na Listagem
10-20:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Vamos aplicar a primeira regra: Cada parâmetro obtém seu próprio lifetime.
Desta vez temos dois parâmetros em vez de um, então temos dois lifetimes:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Você pode ver que a segunda regra não se aplica, porque há mais de um lifetime
de entrada. A terceira regra também não se aplica, porque `longest` é uma
função em vez de um método, então nenhum dos parâmetros é `self`. Depois de
trabalhar através de todas as três regras, ainda não descobrimos qual é o
lifetime do tipo de retorno. É por isso que obtivemos um erro ao tentar
compilar o código na Listagem 10-20: O compilador trabalhou através das regras
de elisão de lifetime, mas ainda não conseguiu descobrir todos os lifetimes das
referências na assinatura.

Como a terceira regra realmente só se aplica em assinaturas de método, veremos
lifetimes nesse contexto a seguir para ver por que a terceira regra significa
que não temos que anotar lifetimes em assinaturas de método com muita
frequência.

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-method-definitions"></a>

### Em Definições de Método

Quando implementamos métodos em uma struct com lifetimes, usamos a mesma
sintaxe que a de parâmetros de tipo genérico, conforme mostrado na Listagem
10-11. Onde declaramos e usamos os parâmetros de lifetime depende de se eles
estão relacionados aos campos da struct ou aos parâmetros do método e valores
de retorno.

Nomes de lifetime para campos de struct sempre precisam ser declarados após a
palavra-chave `impl` e então usados após o nome da struct porque esses
lifetimes fazem parte do tipo da struct.

Em assinaturas de método dentro do bloco `impl`, as referências podem estar
ligadas ao lifetime das referências nos campos da struct, ou podem ser
independentes. Além disso, as regras de elisão de lifetime muitas vezes fazem
com que as anotações de lifetime não sejam necessárias em assinaturas de
método. Vamos ver alguns exemplos usando a struct chamada `ImportantExcerpt` que
definimos na Listagem 10-24.

Primeiro, usaremos um método chamado `level` cujo único parâmetro é uma
referência a `self` e cujo valor de retorno é um `i32`, que não é uma
referência a nada:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

A declaração do parâmetro de lifetime após `impl` e seu uso após o nome do tipo
são necessários, mas por causa da primeira regra de elisão, não somos obrigados
a anotar o lifetime da referência a `self`.

Aqui está um exemplo onde a terceira regra de elisão de lifetime se aplica:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Existem dois lifetimes de entrada, então Rust aplica a primeira regra de elisão
de lifetime e dá a `&self` e `announcement` seus próprios lifetimes. Então,
porque um dos parâmetros é `&self`, o tipo de retorno obtém o lifetime de
`&self`, e todos os lifetimes foram contabilizados.

### O Lifetime Estático

Um lifetime especial que precisamos discutir é `'static`, que denota que a
referência afetada _pode_ viver por toda a duração do programa. Todos os
literais de string têm o lifetime `'static`, que podemos anotar da seguinte
forma:

```rust
let s: &'static str = "Eu tenho um lifetime estático.";
```

O texto desta string é armazenado diretamente no binário do programa, que é
sempre disponível. Portanto, o lifetime de todos os literais de string é
`'static`.

Você pode ver sugestões em mensagens de erro para usar o lifetime `'static`.
Mas antes de especificar `'static` como o lifetime para uma referência, pense
se a referência que você tem realmente vive por todo o lifetime do seu
programa, e se você quer isso. Na maioria das vezes, uma mensagem de erro
sugerindo o lifetime `'static` resulta da tentativa de criar uma referência
pendente ou uma incompatibilidade dos lifetimes disponíveis. Nesses casos, a
solução é corrigir esses problemas, não especificar o lifetime `'static`.

<!-- Old headings. Do not remove or links may break. -->

<a id="generic-type-parameters-trait-bounds-and-lifetimes-together"></a>

## Parâmetros de Tipo Genérico, Trait Bounds e Lifetimes Juntos

Vamos olhar brevemente para a sintaxe de especificar parâmetros de tipo
genérico, trait bounds e lifetimes todos em uma função!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Esta é a função `longest` da Listagem 10-21 que retorna a maior de duas fatias
de string. Mas agora ela tem um parâmetro extra chamado `ann` do tipo genérico
`T`, que pode ser preenchido por qualquer tipo que implemente a trait `Display`
conforme especificado pela cláusula `where`. Este parâmetro extra será impresso
usando `{}`, e é por isso que o trait bound `Display` é necessário. Como
lifetimes são um tipo de genérico, as declarações do parâmetro de lifetime `'a`
e do parâmetro de tipo genérico `T` vão na mesma lista dentro dos colchetes
angulares após o nome da função.

## Resumo

Cobrimos muito neste capítulo! Agora que você sabe sobre parâmetros de tipo
genérico, traits e trait bounds, e parâmetros de lifetime genéricos, você está
pronto para escrever código sem repetição que funciona em muitas situações
diferentes. Parâmetros de tipo genérico permitem que você aplique o código a
diferentes tipos. Traits e trait bounds garantem que, mesmo que os tipos sejam
genéricos, eles terão o comportamento que o código precisa. Você aprendeu a
usar anotações de lifetime para garantir que esse código flexível não tenha
nenhuma referência pendente. E toda essa análise acontece em tempo de
compilação, o que não afeta o desempenho em tempo de execução!

Acredite ou não, há muito mais para aprender sobre os tópicos que discutimos
neste capítulo: O Capítulo 18 discute objetos de trait, que são outra maneira
de usar traits. Também existem cenários mais complexos envolvendo anotações de
lifetime que você só precisará em cenários muito avançados; para esses, você
deve ler a [Referência do Rust][reference]. Mas a seguir, você aprenderá a
escrever testes em Rust para que possa garantir que seu código esteja
funcionando da maneira que deveria.

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/trait-bounds.html
