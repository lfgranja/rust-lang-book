## Um Exemplo de Programa Usando Structs

Para entender quando podemos querer usar structs, vamos escrever um programa que
calcula a área de um retângulo. Começaremos usando variáveis simples e
então refatoraremos o programa até estarmos usando structs.

Vamos fazer um novo projeto binário com Cargo chamado _rectangles_ que receberá
a largura e a altura de um retângulo especificadas em pixels e calculará a área
do retângulo. A Listagem 5-8 mostra um programa curto com uma maneira de fazer
exatamente isso em nosso _src/main.rs_ do projeto.

<Listing number="5-8" file-name="src/main.rs" caption="Calculando a área de um retângulo especificada por variáveis separadas de largura e altura">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Agora, execute este programa usando `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Este código consegue descobrir a área do retângulo chamando a
função `area` com cada dimensão, mas podemos fazer mais para tornar este código claro
e legível.

O problema com este código é evidente na assinatura de `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

A função `area` deve calcular a área de um retângulo, mas a
função que escrevemos tem dois parâmetros, e não está claro em lugar nenhum em nosso
programa que os parâmetros estão relacionados. Seria mais legível e mais
gerenciável agrupar largura e altura. Já discutimos uma maneira
de fazermos isso na seção [“O Tipo Tupla”][the-tuple-type]<!-- ignore -->
do Capítulo 3: usando tuplas.

### Refatorando com Tuplas

A Listagem 5-9 mostra outra versão do nosso programa que usa tuplas.

<Listing number="5-9" file-name="src/main.rs" caption="Especificando a largura e altura do retângulo com uma tupla">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

De uma maneira, este programa é melhor. Tuplas nos permitem adicionar um pouco de estrutura, e
agora estamos passando apenas um argumento. Mas de outra maneira, esta versão é menos
clara: Tuplas não nomeiam seus elementos, então temos que indexar nas partes da
tupla, tornando nosso cálculo menos óbvio.

Confundir a largura e a altura não importaria para o cálculo da área, mas se
quisermos desenhar o retângulo na tela, importaria! Teríamos que
ter em mente que `width` é o índice de tupla `0` e `height` é o índice de tupla
`1`. Isso seria ainda mais difícil para outra pessoa descobrir e manter em
mente se ela fosse usar nosso código. Porque não transmitimos o significado dos
nossos dados em nosso código, agora é mais fácil introduzir erros.

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### Refatorando com Structs

Usamos structs para adicionar significado rotulando os dados. Podemos transformar a tupla
que estamos usando em uma struct com um nome para o todo, bem como nomes para as
partes, como mostrado na Listagem 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Definindo uma struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Aqui, definimos uma struct e a nomeamos `Rectangle`. Dentro das chaves,
definimos os campos como `width` e `height`, ambos do tipo `u32`. Então, em `main`, criamos uma instância específica de `Rectangle`
que tem uma largura de `30` e uma altura de `50`.

Nossa função `area` agora é definida com um parâmetro, que nomeamos
`rectangle`, cujo tipo é um empréstimo imutável de uma instância da struct `Rectangle`.
Como mencionado no Capítulo 4, queremos pegar a struct emprestada em vez de
tomar posse dela. Desta forma, `main` mantém sua posse e pode continuar
usando `rect1`, que é a razão pela qual usamos o `&` na assinatura da função e
onde chamamos a função.

A função `area` acessa os campos `width` e `height` da instância `Rectangle`
(note que acessar campos de uma instância de struct emprestada não
move os valores dos campos, e é por isso que você frequentemente vê empréstimos de structs). Nossa
assinatura de função para `area` agora diz exatamente o que queremos dizer: Calcule a área
de `Rectangle`, usando seus campos `width` e `height`. Isso transmite que a
largura e a altura estão relacionadas entre si, e dá nomes descritivos aos
valores em vez de usar os valores de índice de tupla de `0` e `1`. Isso é uma
vitória para a clareza.

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### Adicionando Funcionalidade Útil com Traits Derivadas

Seria útil poder imprimir uma instância de `Rectangle` enquanto estamos
depurando nosso programa e ver os valores de todos os seus campos. A Listagem 5-11 tenta
usar a macro [`println!`][println]<!-- ignore --> como usamos em
capítulos anteriores. Isso não funcionará, no entanto.

<Listing number="5-11" file-name="src/main.rs" caption="Tentando imprimir uma instância de `Rectangle`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Quando compilamos este código, recebemos um erro com esta mensagem principal:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

A macro `println!` pode fazer muitos tipos de formatação, e por padrão, as chaves
dizem a `println!` para usar a formatação conhecida como `Display`: saída destinada
para consumo direto do usuário final. Os tipos primitivos que vimos até agora
implementam `Display` por padrão porque há apenas uma maneira de querer mostrar
um `1` ou qualquer outro tipo primitivo para um usuário. Mas com structs, a maneira como
`println!` deve formatar a saída é menos clara porque há mais
possibilidades de exibição: Você quer vírgulas ou não? Você quer imprimir as
chaves? Todos os campos devem ser mostrados? Devido a essa ambiguidade, Rust
não tenta adivinhar o que queremos, e structs não têm uma implementação fornecida
de `Display` para usar com `println!` e o placeholder `{}`.

Se continuarmos lendo os erros, encontraremos esta nota útil:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Vamos tentar! A chamada da macro `println!` agora ficará como `println!("rect1 is
{rect1:?}");`. Colocar o especificador `:?` dentro das chaves diz a
`println!` que queremos usar um formato de saída chamado `Debug`. A trait `Debug`
nos permite imprimir nossa struct de uma maneira que seja útil para desenvolvedores para que
possamos ver seu valor enquanto estamos depurando nosso código.

Compile o código com esta mudança. Droga! Ainda recebemos um erro:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Mas novamente, o compilador nos dá uma nota útil:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _inclui_ funcionalidade para imprimir informações de depuração, mas
temos que explicitamente optar por disponibilizar essa funcionalidade para nossa struct.
Para fazer isso, adicionamos o atributo externo `#[derive(Debug)]` logo antes da
definição da struct, como mostrado na Listagem 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Adicionando o atributo para derivar a trait `Debug` e imprimindo a instância `Rectangle` usando formatação de depuração">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Agora, quando executarmos o programa, não receberemos nenhum erro, e veremos a
seguinte saída:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Legal! Não é a saída mais bonita, mas mostra os valores de todos os campos
para esta instância, o que definitivamente ajudaria durante a depuração. Quando temos
structs maiores, é útil ter uma saída que seja um pouco mais fácil de ler; nesses
casos, podemos usar `{:#?}` em vez de `{:?}` na string `println!`. Neste
exemplo, usar o estilo `{:#?}` produzirá o seguinte:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Outra maneira de imprimir um valor usando o formato `Debug` é usar a macro [`dbg!`][dbg]<!-- ignore -->, que toma posse de uma expressão (ao contrário de
`println!`, que toma uma referência), imprime o arquivo e o número da linha de
onde essa chamada da macro `dbg!` ocorre em seu código junto com o valor resultante
dessa expressão, e retorna a posse do valor.

> Nota: Chamar a macro `dbg!` imprime para o fluxo do console de erro padrão
> (`stderr`), ao contrário de `println!`, que imprime para o fluxo do console de saída padrão
> (`stdout`). Falaremos mais sobre `stderr` e `stdout` na
> [seção “Redirecionando Erros para Erro Padrão” no Capítulo 12][err]<!-- ignore -->.

Aqui está um exemplo onde estamos interessados no valor que é atribuído ao
campo `width`, bem como o valor da struct inteira em `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Podemos colocar `dbg!` em torno da expressão `30 * scale` e, porque `dbg!`
retorna a posse do valor da expressão, o campo `width` receberá o
mesmo valor como se não tivéssemos a chamada `dbg!` lá. Não queremos que `dbg!`
tome posse de `rect1`, então usamos uma referência a `rect1` na próxima chamada.
Aqui está como a saída deste exemplo se parece:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Podemos ver que a primeira parte da saída veio de _src/main.rs_ linha 10 onde estamos
depurando a expressão `30 * scale`, e seu valor resultante é `60` (a
formatação `Debug` implementada para inteiros é imprimir apenas seu valor). A
chamada `dbg!` na linha 14 de _src/main.rs_ exibe o valor de `&rect1`, que é
a struct `Rectangle`. Esta saída usa a formatação `Debug` bonita do
tipo `Rectangle`. A macro `dbg!` pode ser realmente útil quando você está tentando
descobrir o que seu código está fazendo!

Além da trait `Debug`, Rust forneceu um número de traits para nós
usarmos com o atributo `derive` que podem adicionar comportamento útil aos nossos tipos
personalizados. Essas traits e seus comportamentos estão listados no [Apêndice C][app-c]<!--
ignore -->. Cobriremos como implementar essas traits com comportamento personalizado,
bem como como criar suas próprias traits no Capítulo 10. Também há muitos
atributos além de `derive`; para mais informações, veja [a seção “Atributos”
da Referência do Rust][attributes].

Nossa função `area` é muito específica: Ela calcula apenas a área de retângulos.
Seria útil amarrar esse comportamento mais intimamente à nossa struct `Rectangle`
porque ela não funcionará com nenhum outro tipo. Vamos ver como podemos continuar a
refatorar este código transformando a função `area` em um método `area`
definido em nosso tipo `Rectangle`.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
