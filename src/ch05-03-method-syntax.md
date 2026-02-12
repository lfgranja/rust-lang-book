## Métodos

Métodos são semelhantes a funções: Nós os declaramos com a palavra-chave `fn` e um
nome, eles podem ter parâmetros e um valor de retorno, e contêm algum código
que é executado quando o método é chamado de outro lugar. Diferente de funções,
métodos são definidos dentro do contexto de uma struct (ou um enum ou um objeto de trait,
que cobrimos no [Capítulo 6][enums]<!-- ignore --> e [Capítulo
18][trait-objects]<!-- ignore -->, respectivamente), e seu primeiro parâmetro é
sempre `self`, que representa a instância da struct na qual o método está sendo
chamado.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### Sintaxe de Método

Vamos mudar a função `area` que tem uma instância `Rectangle` como parâmetro
e, em vez disso, fazer um método `area` definido na struct `Rectangle`, como mostrado
na Listagem 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="Definindo um método `area` na struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Para definir a função dentro do contexto de `Rectangle`, iniciamos um bloco `impl`
(implementação) para `Rectangle`. Tudo dentro deste bloco `impl`
será associado ao tipo `Rectangle`. Então, movemos a função `area`
para dentro das chaves do `impl` e mudamos o primeiro (e neste caso, único)
parâmetro para ser `self` na assinatura e em todos os lugares dentro do corpo. Em
`main`, onde chamamos a função `area` e passamos `rect1` como argumento,
podemos usar a _sintaxe de método_ para chamar o método `area` em nossa instância
`Rectangle`. A sintaxe de método vai depois de uma instância: Adicionamos um ponto seguido
pelo nome do método, parênteses e quaisquer argumentos.

Na assinatura de `area`, usamos `&self` em vez de `rectangle: &Rectangle`.
O `&self` é na verdade uma abreviação para `self: &Self`. Dentro de um bloco `impl`, o
tipo `Self` é um alias para o tipo que o bloco `impl` é para. Métodos devem
ter um parâmetro chamado `self` do tipo `Self` para seu primeiro parâmetro, então Rust
permite abreviar isso apenas com o nome `self` na primeira posição de parâmetro.
Note que ainda precisamos usar o `&` na frente da abreviação `self` para
indicar que este método empresta a instância `Self`, assim como fizemos em
`rectangle: &Rectangle`. Métodos podem tomar posse de `self`, emprestar `self`
imutavelmente, como fizemos aqui, ou emprestar `self` mutavelmente, assim como podem com qualquer
outro parâmetro.

Escolhemos `&self` aqui pela mesma razão que usamos `&Rectangle` na versão da função:
Não queremos tomar posse, e queremos apenas ler os dados na
struct, não escrever nela. Se quiséssemos mudar a instância na qual
chamamos o método como parte do que o método faz, usaríamos `&mut self` como
o primeiro parâmetro. Ter um método que toma posse da instância
usando apenas `self` como o primeiro parâmetro é raro; essa técnica é geralmente
usada quando o método transforma `self` em outra coisa e você quer
impedir o chamador de usar a instância original após a transformação.

A principal razão para usar métodos em vez de funções, além de
fornecer sintaxe de método e não ter que repetir o tipo de `self` em cada
assinatura de método, é para organização. Colocamos todas as coisas que podemos fazer
com uma instância de um tipo em um bloco `impl` em vez de fazer futuros usuários
do nosso código procurarem por capacidades de `Rectangle` em vários lugares na
biblioteca que fornecemos.

Note que podemos escolher dar a um método o mesmo nome de um dos campos da struct.
Por exemplo, podemos definir um método em `Rectangle` que também é chamado
`width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Aqui, estamos escolhendo fazer o método `width` retornar `true` se o valor no
campo `width` da instância for maior que `0` e `false` se o valor for
`0`: Podemos usar um campo dentro de um método de mesmo nome para qualquer propósito. Em
`main`, quando seguimos `rect1.width` com parênteses, Rust sabe que queremos dizer o
método `width`. Quando não usamos parênteses, Rust sabe que queremos dizer o campo
`width`.

Muitas vezes, mas nem sempre, quando damos a um método o mesmo nome de um campo, queremos
que ele retorne apenas o valor no campo e não faça mais nada. Métodos como este
são chamados de _getters_, e Rust não os implementa automaticamente para campos de struct
como algumas outras linguagens fazem. Getters são úteis porque você pode tornar o
campo privado, mas o método público, e assim habilitar acesso somente leitura a esse
campo como parte da API pública do tipo. Discutiremos o que é público e privado
e como designar um campo ou método como público ou privado no [Capítulo
7][public]<!-- ignore -->.

> ### Onde está o operador `->`?
>
> Em C e C++, dois operadores diferentes são usados para chamar métodos: Você usa
> `.` se estiver chamando um método no objeto diretamente e `->` se estiver
> chamando o método em um ponteiro para o objeto e precisar desreferenciar o
> ponteiro primeiro. Em outras palavras, se `object` é um ponteiro,
> `object->something()` é semelhante a `(*object).something()`.
>
> Rust não tem um equivalente ao operador `->`; em vez disso, Rust tem um
> recurso chamado _referenciamento e desreferenciamento automático_. Chamar métodos é
> um dos poucos lugares em Rust com este comportamento.
>
> Funciona assim: Quando você chama um método com `object.something()`, Rust
> adiciona automaticamente `&`, `&mut` ou `*` para que `object` corresponda à
> assinatura do método. Em outras palavras, o seguinte é o mesmo:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> O primeiro parece muito mais limpo. Esse comportamento de referenciamento automático funciona
> porque os métodos têm um receptor claro — o tipo de `self`. Dado o receptor
> e o nome de um método, Rust pode descobrir definitivamente se o método está
> lendo (`&self`), mutando (`&mut self`) ou consumindo (`self`). O fato
> de que Rust torna o empréstimo implícito para receptores de métodos é uma grande parte de
> tornar a propriedade ergonômica na prática.

### Métodos com Mais Parâmetros

Vamos praticar o uso de métodos implementando um segundo método na struct `Rectangle`.
Desta vez, queremos que uma instância de `Rectangle` receba outra instância
de `Rectangle` e retorne `true` se o segundo `Rectangle` puder caber completamente
dentro de `self` (o primeiro `Rectangle`); caso contrário, deve retornar `false`.
Ou seja, uma vez que tenhamos definido o método `can_hold`, queremos poder escrever
o programa mostrado na Listagem 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="Usando o método `can_hold` ainda não escrito">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

A saída esperada seria a seguinte, porque ambas as dimensões de
`rect2` são menores que as dimensões de `rect1`, mas `rect3` é mais largo que
`rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Sabemos que queremos definir um método, então ele estará dentro do bloco `impl Rectangle`.
O nome do método será `can_hold`, e ele receberá um empréstimo imutável
de outro `Rectangle` como parâmetro. Podemos dizer qual será o tipo do
parâmetro olhando para o código que chama o método:
`rect1.can_hold(&rect2)` passa `&rect2`, que é um empréstimo imutável para
`rect2`, uma instância de `Rectangle`. Isso faz sentido porque só precisamos
ler `rect2` (em vez de escrever, o que significaria que precisaríamos de um empréstimo mutável),
e queremos que `main` mantenha a posse de `rect2` para que possamos usá-lo novamente
após chamar o método `can_hold`. O valor de retorno de `can_hold` será um
Booleano, e a implementação verificará se a largura e a altura de
`self` são maiores que a largura e a altura do outro `Rectangle`,
respectivamente. Vamos adicionar o novo método `can_hold` ao bloco `impl` da
Listagem 5-13, mostrado na Listagem 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="Implementando o método `can_hold` em `Rectangle` que recebe outra instância `Rectangle` como parâmetro">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Quando executarmos este código com a função `main` na Listagem 5-14, obteremos nossa
saída desejada. Métodos podem receber múltiplos parâmetros que adicionamos à
assinatura após o parâmetro `self`, e esses parâmetros funcionam exatamente como
parâmetros em funções.

### Funções Associadas

Todas as funções definidas dentro de um bloco `impl` são chamadas _funções associadas_
porque estão associadas ao tipo nomeado após o `impl`. Podemos definir
funções associadas que não têm `self` como seu primeiro parâmetro (e assim
não são métodos) porque não precisam de uma instância do tipo para trabalhar.
Já usamos uma função assim: a função `String::from` que está
definida no tipo `String`.

Funções associadas que não são métodos são frequentemente usadas para construtores que
retornarão uma nova instância da struct. Estes são frequentemente chamados de `new`, mas
`new` não é um nome especial e não está embutido na linguagem. Por exemplo, poderíamos
escolher fornecer uma função associada chamada `square` que teria
um parâmetro de dimensão e usaria isso tanto como largura quanto altura, tornando assim
mais fácil criar um `Rectangle` quadrado em vez de ter que especificar o mesmo
valor duas vezes:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

As palavras-chave `Self` no tipo de retorno e no corpo da função são
aliases para o tipo que aparece após a palavra-chave `impl`, que neste caso
é `Rectangle`.

Para chamar esta função associada, usamos a sintaxe `::` com o nome da struct;
`let sq = Rectangle::square(3);` é um exemplo. Esta função é nomeada pela
struct: A sintaxe `::` é usada tanto para funções associadas quanto para
namespaces criados por módulos. Discutiremos módulos no [Capítulo
7][modules]<!-- ignore -->.

### Múltiplos Blocos `impl`

Cada struct pode ter múltiplos blocos `impl`. Por exemplo, a Listagem
5-15 é equivalente ao código mostrado na Listagem 5-16, que tem cada método em
seu próprio bloco `impl`.

<Listing number="5-16" caption="Reescrevendo a Listagem 5-15 usando múltiplos blocos `impl`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Não há razão para separar esses métodos em múltiplos blocos `impl` aqui,
mas esta é uma sintaxe válida. Veremos um caso em que múltiplos blocos `impl` são
úteis no Capítulo 10, onde discutimos tipos genéricos e traits.

## Resumo

Structs permitem criar tipos personalizados que são significativos para o seu domínio. Ao
usar structs, você pode manter pedaços de dados associados conectados uns aos outros
e nomear cada pedaço para tornar seu código claro. Em blocos `impl`, você pode definir
funções que estão associadas ao seu tipo, e métodos são um tipo de
função associada que permite especificar o comportamento que instâncias de suas
structs têm.

Mas structs não são a única maneira de criar tipos personalizados: Vamos nos voltar para
o recurso de enum do Rust para adicionar outra ferramenta à sua caixa de ferramentas.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
