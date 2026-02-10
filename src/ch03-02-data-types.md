# Tipos de Dados

Cada valor em Rust é de um certo _tipo de dados_, que diz ao Rust que tipo de
dado está sendo especificado para que ele saiba como trabalhar com esse dado. Vamos olhar
para dois subconjuntos de tipos de dados: escalares e compostos.

Tenha em mente que Rust é uma linguagem _estaticamente tipada_, o que significa que ele
deve saber os tipos de todas as variáveis em tempo de compilação. O compilador geralmente pode
inferir qual tipo queremos usar com base no valor e como o usamos. Em casos
quando muitos tipos são possíveis, como quando convertemos uma `String` para um tipo numérico
usando `parse` na seção [[ch02-00-guessing-game-tutorial.md#comparing-the-guess-to-the-secret-number|“Comparando o Palpite com o Número Secreto”]]<!-- ignore -->
no Capítulo 2, devemos adicionar uma anotação de tipo, como esta:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Se não adicionarmos a anotação de tipo `: u32` mostrada no código anterior, o Rust
exibirá o seguinte erro, o que significa que o compilador precisa de mais
informações de nós para saber qual tipo queremos usar:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Você verá diferentes anotações de tipo para outros tipos de dados.

### Tipos Escalares

Um tipo _escalar_ representa um valor único. O Rust tem quatro tipos escalares primários:
inteiros, números de ponto flutuante, Booleanos e caracteres. Você pode reconhecer
estes de outras linguagens de programação. Vamos ver como eles funcionam em Rust.

#### Tipos Inteiros

Um _inteiro_ é um número sem um componente fracionário. Usamos um tipo inteiro
no Capítulo 2, o tipo `u32`. Esta declaração de tipo indica que o
valor associado a ele deve ser um inteiro sem sinal (tipos inteiros com sinal
começam com `i` em vez de `u`) que ocupa 32 bits de espaço. A Tabela 3-1 mostra
os tipos inteiros embutidos no Rust. Podemos usar qualquer uma dessas variantes para declarar
o tipo de um valor inteiro.

<span class="caption">Tabela 3-1: Tipos Inteiros em Rust</span>

| Tamanho | Com Sinal | Sem Sinal |
| ------- | --------- | --------- |
| 8-bit   | `i8`      | `u8`      |
| 16-bit  | `i16`     | `u16`     |
| 32-bit  | `i32`     | `u32`     |
| 64-bit  | `i64`     | `u64`     |
| 128-bit | `i128`    | `u128`    |
| Dependente da arquitetura | `isize` | `usize` |

Cada variante pode ser com sinal ou sem sinal e tem um tamanho explícito.
_Com sinal_ e _sem sinal_ referem-se a se é possível para o número ser
negativo—em outras palavras, se o número precisa ter um sinal com ele
(com sinal) ou se ele sempre será apenas positivo e pode, portanto, ser
representado sem um sinal (sem sinal). É como escrever números no papel: Quando
o sinal importa, um número é mostrado com um sinal de mais ou um sinal de menos; no entanto,
quando é seguro assumir que o número é positivo, ele é mostrado sem sinal.
Números com sinal são armazenados usando a representação de [complemento de dois](https://pt.wikipedia.org/wiki/Complemento_de_dois)<!-- ignore
-->.

Cada variante com sinal pode armazenar números de -(2<sup>n - 1</sup>) a 2<sup>n -
1</sup> - 1 inclusive, onde _n_ é o número de bits que a variante usa. Então, um
`i8` pode armazenar números de -(2<sup>7</sup>) a 2<sup>7</sup> - 1, que é igual a
-128 a 127. Variantes sem sinal podem armazenar números de 0 a 2<sup>n</sup> - 1,
então um `u8` pode armazenar números de 0 a 2<sup>8</sup> - 1, que é igual a 0 a 255.

Além disso, os tipos `isize` e `usize` dependem da arquitetura do
computador em que seu programa está rodando: 64 bits se você estiver em uma arquitetura de 64 bits
e 32 bits se você estiver em uma arquitetura de 32 bits.

Você pode escrever literais inteiros em qualquer uma das formas mostradas na Tabela 3-2. Note
que literais numéricos que podem ser múltiplos tipos numéricos permitem um sufixo de tipo,
como `57u8`, para designar o tipo. Literais numéricos também podem usar `_` como um
separador visual para tornar o número mais fácil de ler, como `1_000`, que terá
o mesmo valor como se você tivesse especificado `1000`.

<span class="caption">Tabela 3-2: Literais Inteiros em Rust</span>

| Literais numéricos | Exemplo       |
| ------------------ | ------------- |
| Decimal            | `98_222`      |
| Hex                | `0xff`        |
| Octal              | `0o77`        |
| Binário            | `0b1111_0000` |
| Byte (`u8` apenas) | `b'A'`        |

Então, como você sabe qual tipo de inteiro usar? Se você não tiver certeza, os
padrões do Rust são geralmente bons lugares para começar: tipos inteiros padrão para `i32`.
A situação primária na qual você usaria `isize` ou `usize` é ao indexar
algum tipo de coleção.

> ##### Overflow de Inteiro
>
> Digamos que você tenha uma variável do tipo `u8` que pode conter valores entre 0 e
> 255. Se você tentar mudar a variável para um valor fora desse intervalo, como
> 256, ocorrerá _overflow de inteiro_ (estouro de inteiro), o que pode resultar em um de dois comportamentos.
> Quando você está compilando em modo de debug, o Rust inclui verificações para overflow de inteiro
> que fazem seu programa entrar em _pânico_ (panic) em tempo de execução se esse comportamento ocorrer. Rust
> usa o termo _panicking_ quando um programa sai com um erro; discutiremos
> pânicos em mais profundidade na seção [[ch09-01-unrecoverable-errors-with-panic.md|“Erros Irrecuperáveis com panic!”]]<!-- ignore -->
> no Capítulo 9.
>
> Quando você está compilando em modo de release com a flag `--release`, o Rust _não_
> inclui verificações para overflow de inteiro que causam pânicos. Em vez disso, se
> ocorrer overflow, o Rust realiza _empacotamento de complemento de dois_ (two’s complement wrapping). Em resumo, valores
> maiores que o valor máximo que o tipo pode conter “dão a volta” para o mínimo
> dos valores que o tipo pode conter. No caso de um `u8`, o valor 256 torna-se
> 0, o valor 257 torna-se 1, e assim por diante. O programa não entrará em pânico, mas a
> variável terá um valor que provavelmente não é o que você estava esperando que ela
> tivesse. Confiar no comportamento de empacotamento do overflow de inteiro é considerado um erro.
>
> Para lidar explicitamente com a possibilidade de overflow, você pode usar essas famílias
> de métodos fornecidos pela biblioteca padrão para tipos numéricos primitivos:
>
> - Empacotar em todos os modos com os métodos `wrapping_*`, como `wrapping_add`.
> - Retornar o valor `None` se houver overflow com os métodos `checked_*`.
> - Retornar o valor e um Booleano indicando se houve overflow com os
>   métodos `overflowing_*`.
> - Saturar no valor mínimo ou máximo do valor com os métodos `saturating_*`.

#### Tipos de Ponto Flutuante

Rust também tem dois tipos primitivos para _números de ponto flutuante_, que são
números com casas decimais. Os tipos de ponto flutuante do Rust são `f32` e `f64`,
que têm 32 bits e 64 bits de tamanho, respectivamente. O tipo padrão é `f64`
porque em CPUs modernas, é aproximadamente a mesma velocidade que `f32`, mas é capaz de
mais precisão. Todos os tipos de ponto flutuante são com sinal.

Aqui está um exemplo que mostra números de ponto flutuante em ação:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Números de ponto flutuante são representados de acordo com o padrão IEEE-754.

#### Operações Numéricas

O Rust suporta as operações matemáticas básicas que você esperaria para todos os tipos
numéricos: adição, subtração, multiplicação, divisão e resto. A divisão de inteiros
trunca em direção a zero para o inteiro mais próximo. O código a seguir mostra
como você usaria cada operação numérica em uma instrução `let`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Cada expressão nessas instruções usa um operador matemático e avalia
para um único valor, que é então vinculado a uma variável. O [[appendix-02-operators.md|Apêndice B]]<!-- ignore -->
contém uma lista de todos os operadores que o Rust fornece.

#### O Tipo Booleano

Como na maioria das outras linguagens de programação, um tipo Booleano em Rust tem dois valores
possíveis: `true` e `false`. Booleanos têm um byte de tamanho. O tipo Booleano em
Rust é especificado usando `bool`. Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

A principal maneira de usar valores Booleanos é através de condicionais, como uma expressão `if`.
Cobraremos como as expressões `if` funcionam em Rust na seção [[ch03-05-control-flow.md#control-flow|“Controle de Fluxo”]]<!-- ignore -->.

#### O Tipo Caractere

O tipo `char` do Rust é o tipo alfabético mais primitivo da linguagem. Aqui estão
alguns exemplos de declaração de valores `char`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Note que especificamos literais `char` com aspas simples, ao contrário de
literais de string, que usam aspas duplas. O tipo `char` do Rust tem 4
bytes de tamanho e representa um valor escalar Unicode, o que significa que ele pode
representar muito mais do que apenas ASCII. Letras acentuadas; caracteres chineses, japoneses e
coreanos; emojis; e espaços de largura zero são todos valores `char` válidos em
Rust. Valores escalares Unicode variam de `U+0000` a `U+D7FF` e `U+E000` a
`U+10FFFF` inclusive. No entanto, um “caractere” não é realmente um conceito em Unicode,
então sua intuição humana para o que é um “caractere” pode não corresponder ao que é um
`char` em Rust. Discutiremos este tópico em detalhes em [[ch08-02-strings.md#storing-utf-8-encoded-text-with-strings|“Armazenando Texto Codificado em UTF-8 com Strings”]]<!-- ignore --> no Capítulo 8.

### Tipos Compostos

_Tipos compostos_ podem agrupar múltiplos valores em um tipo. Rust tem dois
tipos compostos primitivos: tuplas e arrays.

#### O Tipo Tupla

Uma _tupla_ é uma maneira geral de agrupar um número de valores com uma
variedade de tipos em um tipo composto. Tuplas têm um comprimento fixo: Uma vez
declaradas, elas não podem crescer ou diminuir de tamanho.

Criamos uma tupla escrevendo uma lista de valores separados por vírgula dentro de
parênteses. Cada posição na tupla tem um tipo, e os tipos dos
diferentes valores na tupla não precisam ser os mesmos. Adicionamos anotações de tipo
opcionais neste exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

A variável `tup` vincula-se a toda a tupla porque uma tupla é considerada um
único elemento composto. Para obter os valores individuais de uma tupla, podemos
usar correspondência de padrão para desestruturar um valor de tupla, como este:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Este programa primeiro cria uma tupla e a vincula à variável `tup`. Ele então
usa um padrão com `let` para pegar `tup` e transformá-la em três variáveis
separadas, `x`, `y` e `z`. Isso é chamado de _desestruturação_ porque quebra
a única tupla em três partes. Finalmente, o programa imprime o valor de
`y`, que é `6.4`.

Também podemos acessar um elemento de tupla diretamente usando um ponto (`.`) seguido pelo
índice do valor que queremos acessar. Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Este programa cria a tupla `x` e então acessa cada elemento da tupla
usando seus respectivos índices. Como na maioria das linguagens de programação, o primeiro
índice em uma tupla é 0.

A tupla sem nenhum valor tem um nome especial, _unidade_ (unit). Este valor e seu
tipo correspondente são ambos escritos `()` e representam um valor vazio ou um
tipo de retorno vazio. Expressões retornam implicitamente o valor unitário se não
retornarem nenhum outro valor.

#### O Tipo Array

Outra maneira de ter uma coleção de múltiplos valores é com um _array_. Ao contrário de
uma tupla, cada elemento de um array deve ter o mesmo tipo. Ao contrário de arrays em
algumas outras linguagens, arrays em Rust têm um comprimento fixo.

Escrevemos os valores em um array como uma lista separada por vírgula dentro de colchetes:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Arrays são úteis quando você quer seus dados alocados na pilha (stack), o mesmo que
os outros tipos que vimos até agora, em vez de no heap (discutiremos a
pilha e o heap mais no [[ch04-01-what-is-ownership.md#the-stack-and-the-heap|Capítulo 4]]<!-- ignore -->) ou quando
você quer garantir que sempre tenha um número fixo de elementos. Um array
não é tão flexível quanto o tipo vetor, no entanto. Um vetor é um tipo de coleção similar
fornecido pela biblioteca padrão que _tem_ permissão para crescer ou diminuir de
tamanho porque seu conteúdo vive no heap. Se você não tiver certeza se deve usar um
array ou um vetor, é provável que você deva usar um vetor. O [[ch08-01-vectors.md|Capítulo 8]]<!-- ignore --> discute vetores em mais detalhes.

No entanto, arrays são mais úteis quando você sabe que o número de elementos não
precisará mudar. Por exemplo, se você estivesse usando os nomes do mês em um
programa, você provavelmente usaria um array em vez de um vetor porque você sabe
que ele sempre conterá 12 elementos:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Você escreve o tipo de um array usando colchetes com o tipo de cada elemento,
um ponto e vírgula, e então o número de elementos no array, assim:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Aqui, `i32` é o tipo de cada elemento. Após o ponto e vírgula, o número `5`
indica que o array contém cinco elementos.

Você também pode inicializar um array para conter o mesmo valor para cada elemento
especificando o valor inicial, seguido por um ponto e vírgula, e então o comprimento do
array em colchetes, como mostrado aqui:

```rust
let a = [3; 5];
```

O array chamado `a` conterá `5` elementos que serão todos definidos para o valor
`3` inicialmente. Isso é o mesmo que escrever `let a = [3, 3, 3, 3, 3];` mas de uma
maneira mais concisa.

<!-- Old headings. Do not remove or links may break. -->
<a id="accessing-array-elements"></a>

#### Acesso a Elementos de Array

Um array é um único pedaço de memória de um tamanho conhecido e fixo que pode ser
alocado na pilha. Você pode acessar elementos de um array usando indexação,
como este:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Neste exemplo, a variável chamada `first` obterá o valor `1` porque esse
é o valor no índice `[0]` no array. A variável chamada `second` obterá o
valor `2` do índice `[1]` no array.

#### Acesso Inválido a Elemento de Array

Vamos ver o que acontece se você tentar acessar um elemento de um array que está além
do final do array. Digamos que você execute este código, similar ao jogo de adivinhação no
Capítulo 2, para obter um índice de array do usuário:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Este código compila com sucesso. Se você executar este código usando `cargo run` e
digitar `0`, `1`, `2`, `3` ou `4`, o programa imprimirá o valor correspondente
naquele índice no array. Se você, em vez disso, digitar um número além do final do
array, como `10`, você verá uma saída como esta:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

O programa resultou em um erro de tempo de execução no ponto de usar um valor inválido
na operação de indexação. O programa saiu com uma mensagem de erro e
não executou a instrução `println!` final. Quando você tenta acessar um
elemento usando indexação, o Rust verificará se o índice que você especificou é menor
que o comprimento do array. Se o índice for maior ou igual ao comprimento,
o Rust entrará em pânico. Essa verificação tem que acontecer em tempo de execução, especialmente neste caso,
porque o compilador não pode possivelmente saber qual valor um usuário digitará quando eles
executarem o código mais tarde.

Este é um exemplo dos princípios de segurança de memória do Rust em ação. Em muitas
linguagens de baixo nível, esse tipo de verificação não é feito, e quando você fornece um
índice incorreto, memória inválida pode ser acessada. O Rust protege você contra esse
tipo de erro saindo imediatamente em vez de permitir o acesso à memória e
continuar. O Capítulo 9 discute mais sobre o tratamento de erros do Rust e como você pode
escrever código legível e seguro que não entra em pânico nem permite acesso inválido à memória.
