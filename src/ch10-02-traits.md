## Definindo Comportamento Compartilhado com Traits

Uma _trait_ (característica) define a funcionalidade que um tipo particular tem
e pode compartilhar com outros tipos. Podemos usar traits para definir
comportamento compartilhado de forma abstrata. Podemos usar _trait bounds_
(limites de trait) para especificar que um tipo genérico pode ser qualquer tipo
que tenha certo comportamento.

> Nota: Traits são semelhantes a um recurso frequentemente chamado de
> _interfaces_ em outras linguagens, embora com algumas diferenças.

### Definindo uma Trait

O comportamento de um tipo consiste nos métodos que podemos chamar nesse tipo.
Diferentes tipos compartilham o mesmo comportamento se pudermos chamar os
mesmos métodos em todos esses tipos. As definições de trait são uma maneira de
agrupar assinaturas de método para definir um conjunto de comportamentos
necessários para realizar algum propósito.

Por exemplo, digamos que temos várias structs que contêm vários tipos e
quantidades de texto: uma struct `NewsArticle` que contém uma notícia arquivada
em um local específico e uma `SocialPost` que pode ter, no máximo, 280
caracteres junto com metadados que indicam se foi uma nova postagem, uma
repostagem ou uma resposta a outra postagem.

Queremos criar uma biblioteca de agregador de mídia chamada `aggregator` que
possa exibir resumos de dados que podem ser armazenados em uma instância
`NewsArticle` ou `SocialPost`. Para fazer isso, precisamos de um resumo de cada
tipo, e solicitaremos esse resumo chamando um método `summarize` em uma
instância. A Listagem 10-12 mostra a definição de uma trait pública `Summary`
que expressa esse comportamento.

<Listing number="10-12" file-name="src/lib.rs" caption="Uma trait `Summary` que consiste no comportamento fornecido por um método `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Aqui, declaramos uma trait usando a palavra-chave `trait` e, em seguida, o nome
da trait, que é `Summary` neste caso. Também declaramos a trait como `pub` para
que as crates que dependem desta crate também possam usar essa trait, como
veremos em alguns exemplos. Dentro das chaves, declaramos as assinaturas de
método que descrevem os comportamentos dos tipos que implementam essa trait, que
neste caso é `fn summarize(&self) -> String`.

Após a assinatura do método, em vez de fornecer uma implementação dentro de
chaves, usamos um ponto e vírgula. Cada tipo que implementa essa trait deve
fornecer seu próprio comportamento personalizado para o corpo do método. O
compilador garantirá que qualquer tipo que tenha a trait `Summary` terá o
método `summarize` definido com esta assinatura exatamente.

Uma trait pode ter vários métodos em seu corpo: As assinaturas de método são
listadas uma por linha, e cada linha termina em ponto e vírgula.

### Implementando uma Trait em um Tipo

Agora que definimos as assinaturas desejadas dos métodos da trait `Summary`,
podemos implementá-la nos tipos em nosso agregador de mídia. A Listagem 10-13
mostra uma implementação da trait `Summary` na struct `NewsArticle` que usa a
manchete, o autor e a localização para criar o valor de retorno de `summarize`.
Para a struct `SocialPost`, definimos `summarize` como o nome de usuário
seguido por todo o texto da postagem, assumindo que o conteúdo da postagem já
esteja limitado a 280 caracteres.

<Listing number="10-13" file-name="src/lib.rs" caption="Implementando a trait `Summary` nos tipos `NewsArticle` e `SocialPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Implementar uma trait em um tipo é semelhante a implementar métodos regulares.
A diferença é que, após `impl`, colocamos o nome da trait que queremos
implementar, depois usamos a palavra-chave `for` e, em seguida, especificamos o
nome do tipo para o qual queremos implementar a trait. Dentro do bloco `impl`,
colocamos as assinaturas de método que a definição da trait definiu. Em vez de
adicionar um ponto e vírgula após cada assinatura, usamos chaves e preenchemos
o corpo do método com o comportamento específico que queremos que os métodos da
trait tenham para o tipo específico.

Agora que a biblioteca implementou a trait `Summary` em `NewsArticle` e
`SocialPost`, os usuários da crate podem chamar os métodos da trait em
instâncias de `NewsArticle` e `SocialPost` da mesma maneira que chamamos
métodos regulares. A única diferença é que o usuário deve trazer a trait para o
escopo, bem como os tipos. Aqui está um exemplo de como uma crate binária
poderia usar nossa crate de biblioteca `aggregator`:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Este código imprime `1 nova postagem: horse_ebooks: claro, como você
provavelmente já sabe, pessoas`.

Outras crates que dependem da crate `aggregator` também podem trazer a trait
`Summary` para o escopo para implementar `Summary` em seus próprios tipos. Uma
restrição a ser observada é que podemos implementar uma trait em um tipo apenas
se a trait ou o tipo, ou ambos, forem locais à nossa crate. Por exemplo,
podemos implementar traits da biblioteca padrão como `Display` em um tipo
customizado como `SocialPost` como parte da funcionalidade da nossa crate
`aggregator` porque o tipo `SocialPost` é local à nossa crate `aggregator`.
Também podemos implementar `Summary` em `Vec<T>` em nossa crate `aggregator`
porque a trait `Summary` é local à nossa crate `aggregator`.

Mas não podemos implementar traits externas em tipos externos. Por exemplo, não
podemos implementar a trait `Display` em `Vec<T>` dentro de nossa crate
`aggregator`, porque `Display` e `Vec<T>` são definidos na biblioteca padrão e
não são locais à nossa crate `aggregator`. Essa restrição faz parte de uma
propriedade chamada _coerência_, e mais especificamente a _regra do órfão_,
assim chamada porque o tipo pai não está presente. Essa regra garante que o
código de outras pessoas não possa quebrar seu código e vice-versa. Sem a
regra, duas crates poderiam implementar a mesma trait para o mesmo tipo, e o
Rust não saberia qual implementação usar.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### Usando Implementações Padrão

Às vezes, é útil ter um comportamento padrão para alguns ou todos os métodos em
uma trait, em vez de exigir implementações para todos os métodos em cada tipo.
Então, ao implementarmos a trait em um tipo específico, podemos manter ou
substituir o comportamento padrão de cada método.

Na Listagem 10-14, especificamos uma string padrão para o método `summarize` da
trait `Summary` em vez de apenas definir a assinatura do método, como fizemos
na Listagem 10-12.

<Listing number="10-14" file-name="src/lib.rs" caption="Definindo uma trait `Summary` com uma implementação padrão do método `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

Para usar uma implementação padrão para resumir instâncias de `NewsArticle`,
especificamos um bloco `impl` vazio com `impl Summary for NewsArticle {}`.

Mesmo que não estejamos mais definindo o método `summarize` em `NewsArticle`
diretamente, fornecemos uma implementação padrão e especificamos que
`NewsArticle` implementa a trait `Summary`. Como resultado, ainda podemos
chamar o método `summarize` em uma instância de `NewsArticle`, assim:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Este código imprime `(Leia mais...)`.

Criar uma implementação padrão não exige que mudemos nada sobre a implementação
de `Summary` em `SocialPost` na Listagem 10-13. A razão é que a sintaxe para
substituir uma implementação padrão é a mesma que a sintaxe para implementar um
método de trait que não tem uma implementação padrão.

Implementações padrão podem chamar outros métodos na mesma trait, mesmo que
esses outros métodos não tenham uma implementação padrão. Dessa forma, uma
trait pode fornecer muita funcionalidade útil e exigir apenas que os
implementadores especifiquem uma pequena parte dela. Por exemplo, poderíamos
definir a trait `Summary` para ter um método `summarize_author` cuja
implementação é necessária, e então definir um método `summarize` que tenha uma
implementação padrão que chame o método `summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Para usar esta versão de `Summary`, precisamos apenas definir `summarize_author`
quando implementamos a trait em um tipo:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

Depois de definirmos `summarize_author`, podemos chamar `summarize` em
instâncias da struct `SocialPost`, e a implementação padrão de `summarize`
chamará a definição de `summarize_author` que fornecemos. Como implementamos
`summarize_author`, a trait `Summary` nos deu o comportamento do método
`summarize` sem exigir que escrevêssemos mais código. Aqui está como isso se
parece:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Este código imprime `1 nova postagem: (Leia mais de @horse_ebooks...)`.

Observe que não é possível chamar a implementação padrão de uma implementação
de substituição desse mesmo método.

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### Usando Traits como Parâmetros

Agora que você sabe como definir e implementar traits, podemos explorar como
usar traits para definir funções que aceitam muitos tipos diferentes. Usaremos a
trait `Summary` que implementamos nos tipos `NewsArticle` e `SocialPost` na
Listagem 10-13 para definir uma função `notify` que chama o método `summarize`
em seu parâmetro `item`, que é de algum tipo que implementa a trait `Summary`.
Para fazer isso, usamos a sintaxe `impl Trait`, assim:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Em vez de um tipo concreto para o parâmetro `item`, especificamos a
palavra-chave `impl` e o nome da trait. Este parâmetro aceita qualquer tipo que
implemente a trait especificada. No corpo de `notify`, podemos chamar quaisquer
métodos em `item` que vêm da trait `Summary`, como `summarize`. Podemos chamar
`notify` e passar qualquer instância de `NewsArticle` ou `SocialPost`. Código
que chama a função com qualquer outro tipo, como `String` ou `i32`, não
compilará, porque esses tipos não implementam `Summary`.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Sintaxe de Trait Bound

A sintaxe `impl Trait` funciona para casos simples, mas na verdade é um açúcar
sintático para uma forma mais longa conhecida como _trait bound_; parece com
isso:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Notícias de última hora! {}", item.summarize());
}
```

Esta forma mais longa é equivalente ao exemplo na seção anterior, mas é mais
verbosa. Colocamos trait bounds com a declaração do parâmetro de tipo genérico
após dois pontos e dentro de colchetes angulares.

A sintaxe `impl Trait` é conveniente e torna o código mais conciso em casos
simples, enquanto a sintaxe de trait bound mais completa pode expressar mais
complexidade em outros casos. Por exemplo, podemos ter dois parâmetros que
implementam `Summary`. Fazer isso com a sintaxe `impl Trait` se parece com
isso:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Usar `impl Trait` é apropriado se quisermos que essa função permita que `item1`
e `item2` tenham tipos diferentes (desde que ambos os tipos implementem
`Summary`). Se quisermos forçar ambos os parâmetros a ter o mesmo tipo, no
entanto, devemos usar um trait bound, assim:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

O tipo genérico `T` especificado como o tipo dos parâmetros `item1` e `item2`
restringe a função de tal forma que o tipo concreto do valor passado como
argumento para `item1` e `item2` deve ser o mesmo.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### Múltiplos Trait Bounds com a Sintaxe `+`

Também podemos especificar mais de um trait bound. Digamos que queríamos que
`notify` usasse formatação de exibição, bem como `summarize` em `item`:
Especificamos na definição de `notify` que `item` deve implementar tanto
`Display` quanto `Summary`. Podemos fazer isso usando a sintaxe `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

A sintaxe `+` também é válida com trait bounds em tipos genéricos:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

Com os dois trait bounds especificados, o corpo de `notify` pode chamar
`summarize` e usar `{}` para formatar `item`.

#### Trait Bounds Mais Claros com Cláusulas `where`

Usar muitos trait bounds tem suas desvantagens. Cada genérico tem seus próprios
trait bounds, então funções com vários parâmetros de tipo genérico podem conter
muitas informações de trait bound entre o nome da função e sua lista de
parâmetros, tornando a assinatura da função difícil de ler. Por esse motivo,
Rust tem uma sintaxe alternativa para especificar trait bounds dentro de uma
cláusula `where` após a assinatura da função. Então, em vez de escrever isso:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

podemos usar uma cláusula `where`, assim:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

A assinatura desta função é menos confusa: O nome da função, a lista de
parâmetros e o tipo de retorno estão próximos, semelhantes a uma função sem
muitos trait bounds.

### Retornando Tipos que Implementam Traits

Também podemos usar a sintaxe `impl Trait` na posição de retorno para retornar
um valor de algum tipo que implementa uma trait, como mostrado aqui:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Ao usar `impl Summary` para o tipo de retorno, especificamos que a função
`returns_summarizable` retorna algum tipo que implementa a trait `Summary` sem
nomear o tipo concreto. Neste caso, `returns_summarizable` retorna um
`SocialPost`, mas o código que chama essa função não precisa saber disso.

A capacidade de especificar um tipo de retorno apenas pela trait que ele
implementa é especialmente útil no contexto de closures e iteradores, que
cobrimos no Capítulo 13. Closures e iteradores criam tipos que apenas o
compilador conhece ou tipos que são muito longos para especificar. A sintaxe
`impl Trait` permite especificar concisamente que uma função retorna algum tipo
que implementa a trait `Iterator` sem precisar escrever um tipo muito longo.

No entanto, você só pode usar `impl Trait` se estiver retornando um único tipo.
Por exemplo, este código que retorna um `NewsArticle` ou um `SocialPost` com o
tipo de retorno especificado como `impl Summary` não funcionaria:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Retornar um `NewsArticle` ou um `SocialPost` não é permitido devido a
restrições em torno de como a sintaxe `impl Trait` é implementada no
compilador. Abordaremos como escrever uma função com esse comportamento na
seção [“Usando Objetos de Trait para Abstrair sobre Comportamento
Compartilhado”][trait-objects]<!-- ignore --> do Capítulo 18.

### Usando Trait Bounds para Implementar Métodos Condicionalmente

Ao usar um trait bound com um bloco `impl` que usa parâmetros de tipo genérico,
podemos implementar métodos condicionalmente para tipos que implementam as
traits especificadas. Por exemplo, o tipo `Pair<T>` na Listagem 10-15 sempre
implementa a função `new` para retornar uma nova instância de `Pair<T>`
(lembre-se da seção [“Sintaxe de Método”][methods]<!-- ignore --> do Capítulo 5
que `Self` é um alias de tipo para o tipo do bloco `impl`, que neste caso é
`Pair<T>`). Mas no próximo bloco `impl`, `Pair<T>` implementa apenas o método
`cmp_display` se seu tipo interno `T` implementar a trait `PartialOrd` que
permite comparação _e_ a trait `Display` que permite impressão.

<Listing number="10-15" file-name="src/lib.rs" caption="Implementando métodos condicionalmente em um tipo genérico dependendo de trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Também podemos implementar condicionalmente uma trait para qualquer tipo que
implemente outra trait. Implementações de uma trait em qualquer tipo que
satisfaça os trait bounds são chamadas de _implementações gerais_ (blanket
implementations) e são usadas extensivamente na biblioteca padrão do Rust. Por
exemplo, a biblioteca padrão implementa a trait `ToString` em qualquer tipo que
implemente a trait `Display`. O bloco `impl` na biblioteca padrão se parece com
este código:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Como a biblioteca padrão tem essa implementação geral, podemos chamar o método
`to_string` definido pela trait `ToString` em qualquer tipo que implemente a
trait `Display`. Por exemplo, podemos transformar inteiros em seus valores
`String` correspondentes assim, porque inteiros implementam `Display`:

```rust
let s = 3.to_string();
```

Implementações gerais aparecem na documentação da trait na seção
“Implementors”.

Traits e trait bounds nos permitem escrever código que usa parâmetros de tipo
genérico para reduzir a duplicação, mas também especificar ao compilador que
queremos que o tipo genérico tenha um comportamento específico. O compilador
pode então usar as informações de trait bound para verificar se todos os tipos
concretos usados com nosso código fornecem o comportamento correto. Em
linguagens tipadas dinamicamente, obteríamos um erro em tempo de execução se
chamássemos um método em um tipo que não definisse o método. Mas Rust move
esses erros para o tempo de compilação, de modo que somos forçados a corrigir
os problemas antes que nosso código possa ser executado. Além disso, não
precisamos escrever código que verifique o comportamento em tempo de execução,
porque já verificamos em tempo de compilação. Fazer isso melhora o desempenho
sem ter que abrir mão da flexibilidade dos genéricos.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax
