## Definindo e Instanciando Structs

Structs são semelhantes a [[Tuplas]], discutidas na seção [“O Tipo Tupla”][tuples]<!--
ignore -->, pois ambas contêm múltiplos valores relacionados. Como as tuplas, as
partes de uma struct podem ser de diferentes tipos. Diferente das tuplas, em uma struct
você nomeia cada pedaço de dado para que fique claro o que os valores significam. Adicionar esses
nomes significa que structs são mais flexíveis que tuplas: Você não precisa depender
da ordem dos dados para especificar ou acessar os valores de uma instância.

Para definir uma struct, inserimos a palavra-chave `struct` e nomeamos a struct inteira. O
nome de uma struct deve descrever o significado dos pedaços de dados sendo
agrupados. Então, dentro de chaves, definimos os nomes e tipos dos
pedaços de dados, que chamamos de _campos_. Por exemplo, a Listagem 5-1 mostra uma
struct que armazena informações sobre uma conta de usuário.

<Listing number="5-1" file-name="src/main.rs" caption="Uma definição de struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Para usar uma struct depois de tê-la definido, criamos uma _instância_ dessa struct
especificando valores concretos para cada um dos campos. Criamos uma instância
declarando o nome da struct e depois adicionando chaves contendo pares _`chave:
valor`_, onde as chaves são os nomes dos campos e os valores são os
dados que queremos armazenar nesses campos. Não precisamos especificar os campos na
mesma ordem em que os declaramos na struct. Em outras palavras, a
definição da struct é como um modelo geral para o tipo, e as instâncias preenchem
esse modelo com dados específicos para criar valores do tipo. Por
exemplo, podemos declarar um usuário específico como mostrado na Listagem 5-2.

<Listing number="5-2" file-name="src/main.rs" caption="Criando uma instância da struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Para obter um valor específico de uma struct, usamos a notação de ponto. Por exemplo, para
acessar o endereço de email deste usuário, usamos `user1.email`. Se a instância for
mutável, podemos mudar um valor usando a notação de ponto e atribuindo em um
campo específico. A Listagem 5-3 mostra como mudar o valor no campo `email`
de uma instância `User` mutável.

<Listing number="5-3" file-name="src/main.rs" caption="Mudando o valor no campo `email` de uma instância `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Note que a instância inteira deve ser mutável; Rust não nos permite marcar
apenas certos campos como mutáveis. Como com qualquer expressão, podemos construir uma nova
instância da struct como a última expressão no corpo da função para
retornar implicitamente essa nova instância.

A Listagem 5-4 mostra uma função `build_user` que retorna uma instância `User` com
o email e nome de usuário fornecidos. O campo `active` recebe o valor `true`, e o
`sign_in_count` recebe o valor de `1`.

<Listing number="5-4" file-name="src/main.rs" caption="Uma função `build_user` que recebe um email e nome de usuário e retorna uma instância `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Faz sentido nomear os parâmetros da função com o mesmo nome dos campos da struct,
mas ter que repetir os nomes dos campos e variáveis `email` e `username` é um pouco entediante.
Se a struct tivesse mais campos, repetir cada nome ficaria ainda mais irritante.
Felizmente, há uma abreviação conveniente!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Usando a Abreviação de Inicialização de Campo (Field Init Shorthand)

Porque os nomes dos parâmetros e os nomes dos campos da struct são exatamente os mesmos na
Listagem 5-4, podemos usar a sintaxe de _field init shorthand_ para reescrever
`build_user` de modo que ela se comporte exatamente da mesma maneira, mas sem a
repetição de `username` e `email`, como mostrado na Listagem 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="Uma função `build_user` que usa field init shorthand porque os parâmetros `username` e `email` têm o mesmo nome que os campos da struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Aqui, estamos criando uma nova instância da struct `User`, que tem um campo
chamado `email`. Queremos definir o valor do campo `email` para o valor no
parâmetro `email` da função `build_user`. Porque o campo `email` e o
parâmetro `email` têm o mesmo nome, só precisamos escrever `email` em vez
de `email: email`.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### Criando Instâncias a partir de Outras Instâncias com Sintaxe de Atualização de Struct (Struct Update Syntax)

Muitas vezes é útil criar uma nova instância de uma struct que inclua a maioria dos
valores de outra instância do mesmo tipo, mas altere alguns deles.
Você pode fazer isso usando a [[Struct Update Syntax|sintaxe de atualização de struct]].

Primeiro, na Listagem 5-6 mostramos como criar uma nova instância `User` em `user2` da
maneira regular, sem a sintaxe de atualização. Definimos um novo valor para `email`, mas
de resto usamos os mesmos valores de `user1` que criamos na Listagem 5-2.

<Listing number="5-6" file-name="src/main.rs" caption="Criando uma nova instância `User` usando todos, exceto um, dos valores de `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Usando a sintaxe de atualização de struct, podemos obter o mesmo efeito com menos código,
como mostrado na Listagem 5-7. A sintaxe `..` especifica que os campos restantes não
explicitamente definidos devem ter o mesmo valor que os campos na instância fornecida.

<Listing number="5-7" file-name="src/main.rs" caption="Usando a sintaxe de atualização de struct para definir um novo valor `email` para uma instância `User`, mas usar o restante dos valores de `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

O código na Listagem 5-7 também cria uma instância em `user2` que tem um
valor diferente para `email`, mas tem os mesmos valores para os campos `username`,
`active` e `sign_in_count` de `user1`. O `..user1` deve vir por último
para especificar que quaisquer campos restantes devem obter seus valores dos
campos correspondentes em `user1`, mas podemos escolher especificar valores para quantos
campos quisermos em qualquer ordem, independentemente da ordem dos campos na
definição da struct.

Note que a sintaxe de atualização de struct usa `=` como uma atribuição; isso é porque
ela move os dados, assim como vimos na seção [“Variáveis e Dados Interagindo com
Move”][move]<!-- ignore -->. Neste exemplo, não podemos mais usar
`user1` depois de criar `user2` porque a `String` no campo `username` de
`user1` foi movida para `user2`. Se tivéssemos dado a `user2` novos valores `String` para
`email` e `username`, e assim usado apenas os valores de `active` e `sign_in_count`
de `user1`, então `user1` ainda seria válido após criar `user2`.
Tanto `active` quanto `sign_in_count` são tipos que implementam a trait `Copy`, então
o comportamento que discutimos na seção [“Dados Somente na Stack: Copy”][copy]<!-- ignore -->
se aplicaria. Também podemos usar `user1.email` neste exemplo,
porque seu valor não foi movido de `user1`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### Criando Tipos Diferentes com Tuple Structs (Structs de Tupla)

Rust também suporta structs que parecem semelhantes a tuplas, chamadas _tuple structs_.
Tuple structs têm o significado adicionado que o nome da struct fornece, mas não têm
nomes associados aos seus campos; em vez disso, elas apenas têm os tipos dos
campos. Tuple structs são úteis quando você quer dar à tupla inteira um nome
e tornar a tupla um tipo diferente de outras tuplas, e quando nomear cada
campo como em uma struct regular seria verborrágico ou redundante.

Para definir uma tuple struct, comece com a palavra-chave `struct` e o nome da struct
seguido pelos tipos na tupla. Por exemplo, aqui definimos e usamos duas
tuple structs chamadas `Color` e `Point`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

Note que os valores `black` e `origin` são tipos diferentes porque são
instâncias de diferentes tuple structs. Cada struct que você define é seu próprio tipo,
mesmo que os campos dentro da struct possam ter os mesmos tipos. Por
exemplo, uma função que aceita um parâmetro do tipo `Color` não pode aceitar um
`Point` como argumento, mesmo que ambos os tipos sejam compostos por três valores
`i32`. Caso contrário, instâncias de tuple struct são semelhantes a tuplas, pois você pode
desestruturá-las em suas partes individuais, e pode usar um `.` seguido
pelo índice para acessar um valor individual. Diferente das tuplas, tuple structs
exigem que você nomeie o tipo da struct quando as desestrutura. Por
exemplo, escreveríamos `let Point(x, y, z) = origin;` para desestruturar os
valores no ponto `origin` em variáveis chamadas `x`, `y` e `z`.

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### Definindo Unit-Like Structs (Structs Tipo Unit)

Você também pode definir structs que não têm campos! Estas são chamadas de
_unit-like structs_ porque se comportam de maneira semelhante a `()`, o tipo unit que
mencionamos na seção [“O Tipo Tupla”][tuples]<!-- ignore -->. Unit-like
structs podem ser úteis quando você precisa implementar uma trait em algum tipo, mas não
tem nenhum dado que queira armazenar no próprio tipo. Discutiremos traits
no Capítulo 10. Aqui está um exemplo de declaração e instanciação de uma unit struct
chamada `AlwaysEqual`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

Para definir `AlwaysEqual`, usamos a palavra-chave `struct`, o nome que queremos e
então um ponto e vírgula. Não há necessidade de chaves ou parênteses! Então, podemos obter
uma instância de `AlwaysEqual` na variável `subject` de maneira semelhante: usando
o nome que definimos, sem chaves ou parênteses. Imagine que
mais tarde implementaremos um comportamento para este tipo de modo que cada instância de
`AlwaysEqual` seja sempre igual a cada instância de qualquer outro tipo, talvez para
ter um resultado conhecido para fins de teste. Não precisaríamos de nenhum dado para
implementar esse comportamento! Você verá no Capítulo 10 como definir traits e
implementá-las em qualquer tipo, incluindo unit-like structs.

> ### Ownership de Dados da Struct
>
> Na definição da struct `User` na Listagem 5-1, usamos o tipo `String` proprietário (owned)
> em vez do tipo slice de string `&str`. Esta é uma escolha deliberada
> porque queremos que cada instância desta struct possua todos os seus dados e que
> esses dados sejam válidos por tanto tempo quanto a struct inteira for válida.
>
> Também é possível que structs armazenem referências a dados de propriedade de outra coisa,
> mas fazer isso requer o uso de _lifetimes_, um recurso do Rust que
> discutiremos no Capítulo 10. Lifetimes garantem que os dados referenciados por uma struct
> sejam válidos por tanto tempo quanto a struct for. Digamos que você tente armazenar uma referência
> em uma struct sem especificar lifetimes, como o seguinte em
> *src/main.rs*; isso não funcionará:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> O compilador reclamará que precisa de especificadores de lifetime:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> No Capítulo 10, discutiremos como corrigir esses erros para que você possa armazenar
> referências em structs, mas por enquanto, corrigiremos erros como esses usando tipos
> proprietários (owned) como `String` em vez de referências como `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
