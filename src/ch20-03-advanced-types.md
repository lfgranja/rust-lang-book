## Tipos Avançados

O sistema de tipos do Rust tem algumas funcionalidades que mencionamos até agora, mas ainda não discutimos. Começaremos discutindo newtypes em geral enquanto examinamos por que eles são úteis como tipos. Em seguida, passaremos para aliases de tipo, uma funcionalidade semelhante a newtypes, mas com semântica ligeiramente diferente. Também discutiremos o tipo `!` e tipos de tamanho dinâmico.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### Segurança de Tipo e Abstração com o Padrão Newtype

Esta seção assume que você leu a seção anterior [“Implementando Traits Externas com o Padrão Newtype”][newtype]<!-- ignore -->. O padrão newtype também é útil para tarefas além das que discutimos até agora, incluindo impor estaticamente que os valores nunca sejam confundidos e indicar as unidades de um valor. Você viu um exemplo de uso de newtypes para indicar unidades na Listagem 20-16: lembre-se de que as structs `Millimeters` e `Meters` envolviam valores `u32` em um newtype. Se escrevêssemos uma função com um parâmetro do tipo `Millimeters`, não poderíamos compilar um programa que acidentalmente tentasse chamar essa função com um valor do tipo `Meters` ou um `u32` simples.

Também podemos usar o padrão newtype para abstrair alguns detalhes de implementação de um tipo: o novo tipo pode expor uma API pública que é diferente da API do tipo interno privado.

Newtypes também podem ocultar a implementação interna. Por exemplo, poderíamos fornecer um tipo `People` para envolver um `HashMap<i32, String>` que armazena o ID de uma pessoa associado ao seu nome. O código usando `People` interagiria apenas com a API pública que fornecemos, como um método para adicionar uma string de nome à coleção `People`; esse código não precisaria saber que atribuímos um ID `i32` aos nomes internamente. O padrão newtype é uma maneira leve de obter encapsulamento para ocultar detalhes de implementação, o que discutimos na seção [“Encapsulamento que Oculta Detalhes de Implementação”][encapsulation-that-hides-implementation-details]<!-- ignore --> no Capítulo 18.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### Sinônimos de Tipo e Aliases de Tipo

O Rust fornece a capacidade de declarar um _alias de tipo_ para dar a um tipo existente outro nome. Para isso, usamos a palavra-chave `type`. Por exemplo, podemos criar o alias `Kilometers` para `i32` assim:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Agora o alias `Kilometers` é um _sinônimo_ para `i32`; ao contrário dos tipos `Millimeters` e `Meters` que criamos na Listagem 20-16, `Kilometers` não é um tipo novo e separado. Valores que têm o tipo `Kilometers` serão tratados da mesma forma que valores do tipo `i32`:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Como `Kilometers` e `i32` são o mesmo tipo, podemos adicionar valores de ambos os tipos e podemos passar valores de `Kilometers` para funções que recebem parâmetros `i32`. No entanto, usando este método, não obtemos os benefícios de verificação de tipo que obtemos do padrão newtype discutido anteriormente. Em outras palavras, se misturarmos valores de `Kilometers` e `i32` em algum lugar, o compilador não nos dará um erro.

O principal caso de uso para sinônimos de tipo é reduzir a repetição. Por exemplo, podemos ter um tipo longo como este:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Escrever esse tipo longo em assinaturas de função e como anotações de tipo em todo o código pode ser cansativo e propenso a erros. Imagine ter um projeto cheio de código como o da Listagem 20-25.

<Listing number="20-25" caption="Usando um tipo longo em muitos lugares">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Um alias de tipo torna este código mais gerenciável reduzindo a repetição. Na Listagem 20-26, introduzimos um alias chamado `Thunk` para o tipo verboso e podemos substituir todos os usos do tipo pelo alias mais curto `Thunk`.

<Listing number="20-26" caption="Introduzindo um alias de tipo, `Thunk`, para reduzir a repetição">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

Este código é muito mais fácil de ler e escrever! Escolher um nome significativo para um alias de tipo também pode ajudar a comunicar sua intenção (_thunk_ é uma palavra para código a ser avaliado posteriormente, então é um nome apropriado para uma closure que é armazenada).

Aliases de tipo também são comumente usados com o tipo `Result<T, E>` para reduzir a repetição. Considere o módulo `std::io` na biblioteca padrão. As operações de I/O frequentemente retornam um `Result<T, E>` para lidar com situações em que as operações falham ao funcionar. Esta biblioteca tem uma struct `std::io::Error` que representa todos os erros de I/O possíveis. Muitas das funções em `std::io` retornarão `Result<T, E>` onde o `E` é `std::io::Error`, como essas funções na trait `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

O `Result<..., Error>` é repetido muito. Como tal, `std::io` tem esta declaração de alias de tipo:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Como essa declaração está no módulo `std::io`, podemos usar o alias totalmente qualificado `std::io::Result<T>`; ou seja, um `Result<T, E>` com o `E` preenchido como `std::io::Error`. As assinaturas da função da trait `Write` acabam parecendo com isso:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

O alias de tipo ajuda de duas maneiras: torna o código mais fácil de escrever _e_ nos dá uma interface consistente em todo o `std::io`. Como é um alias, é apenas outro `Result<T, E>`, o que significa que podemos usar quaisquer métodos que funcionem em `Result<T, E>` com ele, bem como sintaxe especial como o operador `?`.

### O Tipo Never que Nunca Retorna

O Rust tem um tipo especial chamado `!` que é conhecido no jargão da teoria dos tipos como o _tipo vazio_ porque não tem valores. Preferimos chamá-lo de _tipo never_ porque ele substitui o tipo de retorno quando uma função nunca retornará. Aqui está um exemplo:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Este código é lido como "a função `bar` retorna never". Funções que retornam never são chamadas de _funções divergentes_. Não podemos criar valores do tipo `!`, então `bar` nunca poderá retornar.

Mas de que serve um tipo para o qual você nunca pode criar valores? Lembre-se do código da Listagem 2-5, parte do jogo de adivinhação; reproduzimos um pouco dele aqui na Listagem 20-27.

<Listing number="20-27" caption="Um `match` com um braço que termina em `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

Na época, pulamos alguns detalhes neste código. Na seção [“A Estrutura de Controle de Fluxo `match`”][the-match-control-flow-construct]<!-- ignore --> no Capítulo 6, discutimos que os braços de `match` devem retornar todos o mesmo tipo. Portanto, por exemplo, o seguinte código não funciona:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

O tipo de `guess` neste código teria que ser um inteiro _e_ uma string, e o Rust exige que `guess` tenha apenas um tipo. Então, o que `continue` retorna? Como fomos autorizados a retornar um `u32` de um braço e ter outro braço que termina com `continue` na Listagem 20-27?

Como você deve ter adivinhado, `continue` tem um valor `!`. Ou seja, quando o Rust calcula o tipo de `guess`, ele olha para ambos os braços de correspondência, o primeiro com um valor de `u32` e o último com um valor `!`. Como `!` nunca pode ter um valor, o Rust decide que o tipo de `guess` é `u32`.

A maneira formal de descrever esse comportamento é que expressões do tipo `!` podem ser coagidas em qualquer outro tipo. Temos permissão para terminar este braço de `match` com `continue` porque `continue` não retorna um valor; em vez disso, move o controle de volta para o topo do loop, então no caso `Err`, nunca atribuímos um valor a `guess`.

O tipo never é útil com a macro `panic!` também. Lembre-se da função `unwrap` que chamamos em valores `Option<T>` para produzir um valor ou entrar em pânico com esta definição:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Neste código, a mesma coisa acontece que no `match` na Listagem 20-27: o Rust vê que `val` tem o tipo `T` e `panic!` tem o tipo `!`, então o resultado da expressão `match` geral é `T`. Este código funciona porque `panic!` não produz um valor; ele termina o programa. No caso `None`, não retornaremos um valor de `unwrap`, então este código é válido.

Uma expressão final que tem o tipo `!` é um loop:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Aqui, o loop nunca termina, então `!` é o valor da expressão. No entanto, isso não seria verdade se incluíssemos um `break`, porque o loop terminaria quando chegasse ao `break`.

### Tipos de Tamanho Dinâmico e a Trait `Sized`

O Rust precisa saber certos detalhes sobre seus tipos, como quanto espaço alocar para um valor de um tipo específico. Isso deixa um canto de seu sistema de tipos um pouco confuso no início: o conceito de _tipos de tamanho dinâmico_. Às vezes referidos como _DSTs_ ou _tipos não dimensionados_, esses tipos nos permitem escrever código usando valores cujo tamanho podemos saber apenas em tempo de execução.

Vamos nos aprofundar nos detalhes de um tipo de tamanho dinâmico chamado `str`, que temos usado ao longo do livro. Isso mesmo, não `&str`, mas `str` por conta própria, é um DST. Em muitos casos, como ao armazenar texto inserido por um usuário, não podemos saber quão longa é a string até o tempo de execução. Isso significa que não podemos criar uma variável do tipo `str`, nem podemos receber um argumento do tipo `str`. Considere o seguinte código, que não funciona:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

O Rust precisa saber quanta memória alocar para qualquer valor de um tipo específico, e todos os valores de um tipo devem usar a mesma quantidade de memória. Se o Rust nos permitisse escrever este código, esses dois valores `str` precisariam ocupar a mesma quantidade de espaço. Mas eles têm comprimentos diferentes: `s1` precisa de 12 bytes de armazenamento e `s2` precisa de 15. É por isso que não é possível criar uma variável contendo um tipo de tamanho dinâmico.

Então, o que fazemos? Neste caso, você já sabe a resposta: tornamos o tipo de `s1` e `s2` uma fatia de string (`&str`) em vez de `str`. Lembre-se da seção [“Fatias de String”][string-slices]<!-- ignore --> no Capítulo 4 que a estrutura de dados da fatia armazena apenas a posição inicial e o comprimento da fatia. Portanto, embora `&T` seja um valor único que armazena o endereço de memória de onde o `T` está localizado, uma fatia de string é _dois_ valores: o endereço do `str` e seu comprimento. Como tal, podemos saber o tamanho de um valor de fatia de string em tempo de compilação: é o dobro do comprimento de um `usize`. Ou seja, sempre sabemos o tamanho de uma fatia de string, não importa quão longa seja a string a que ela se refere. Em geral, esta é a maneira pela qual os tipos de tamanho dinâmico são usados no Rust: eles têm um pouco extra de metadados que armazena o tamanho da informação dinâmica. A regra de ouro dos tipos de tamanho dinâmico é que devemos sempre colocar valores de tipos de tamanho dinâmico atrás de um ponteiro de algum tipo.

Podemos combinar `str` com todos os tipos de ponteiros: por exemplo, `Box<str>` ou `Rc<str>`. Na verdade, você já viu isso antes, mas com um tipo de tamanho dinâmico diferente: traits. Cada trait é um tipo de tamanho dinâmico ao qual podemos nos referir usando o nome da trait. Na seção [“Usando Objetos de Trait para Abstrair sobre Comportamento Compartilhado”][using-trait-objects-to-abstract-over-shared-behavior]<!-- ignore --> no Capítulo 18, mencionamos que para usar traits como objetos de trait, devemos colocá-los atrás de um ponteiro, como `&dyn Trait` ou `Box<dyn Trait>` (`Rc<dyn Trait>` também funcionaria).

Para trabalhar com DSTs, o Rust fornece a trait `Sized` para determinar se o tamanho de um tipo é conhecido ou não em tempo de compilação. Esta trait é implementada automaticamente para tudo cujo tamanho é conhecido em tempo de compilação. Além disso, o Rust adiciona implicitamente um limite em `Sized` para cada função genérica. Ou seja, uma definição de função genérica como esta:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

é realmente tratada como se tivéssemos escrito isto:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Por padrão, funções genéricas funcionarão apenas em tipos que têm um tamanho conhecido em tempo de compilação. No entanto, você pode usar a seguinte sintaxe especial para relaxar essa restrição:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Um limite de trait em `?Sized` significa "`T` pode ou não ser `Sized`", e essa notação substitui o padrão de que os tipos genéricos devem ter um tamanho conhecido em tempo de compilação. A sintaxe `?Trait` com este significado está disponível apenas para `Sized`, não para quaisquer outras traits.

Observe também que mudamos o tipo do parâmetro `t` de `T` para `&T`. Como o tipo pode não ser `Sized`, precisamos usá-lo atrás de algum tipo de ponteiro. Neste caso, escolhemos uma referência.

A seguir, falaremos sobre funções e closures!

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
