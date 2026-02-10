# Funções

Funções são predominantes no código Rust. Você já viu uma das funções mais
importantes na linguagem: a função `main`, que é o ponto de entrada
de muitos programas. Você também viu a palavra-chave `fn`, que permite que você
declare novas funções.

O código Rust usa _snake case_ como o estilo convencional para nomes de funções e variáveis,
no qual todas as letras são minúsculas e sublinhados separam palavras.
Aqui está um programa que contém uma definição de função de exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Definimos uma função em Rust digitando `fn` seguido por um nome de função e um
conjunto de parênteses. As chaves dizem ao compilador onde o corpo da função
começa e termina.

Podemos chamar qualquer função que definimos digitando seu nome seguido por um conjunto
de parênteses. Como `another_function` é definida no programa, ela pode ser
chamada de dentro da função `main`. Note que definimos `another_function`
_depois_ da função `main` no código fonte; poderíamos tê-la definido antes
também. O Rust não se importa onde você define suas funções, apenas que elas estejam
definidas em algum lugar em um escopo que possa ser visto pelo chamador.

Vamos começar um novo projeto binário chamado _functions_ para explorar funções
mais a fundo. Coloque o exemplo `another_function` em _src/main.rs_ e execute-o. Você
deve ver a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

As linhas executam na ordem em que aparecem na função `main`.
Primeiro a mensagem “Hello, world!” é impressa, e então `another_function` é chamada
e sua mensagem é impressa.

### Parâmetros

Podemos definir funções para ter _parâmetros_, que são variáveis especiais que
fazem parte da assinatura de uma função. Quando uma função tem parâmetros, você pode
fornecer a ela valores concretos para esses parâmetros. Tecnicamente, os valores
concretos são chamados de _argumentos_, mas em conversas informais, as pessoas tendem a usar
as palavras _parâmetro_ e _argumento_ de forma intercambiável para as variáveis
na definição de uma função ou os valores concretos passados quando você chama uma
função.

Nesta versão de `another_function` adicionamos um parâmetro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Tente executar este programa; você deve obter a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

A declaração de `another_function` tem um parâmetro chamado `x`. O tipo de
`x` é especificado como `i32`. Quando passamos `5` para `another_function`, a
macro `println!` coloca `5` onde o par de chaves contendo `x` estava
na string de formatação.

Em assinaturas de função, você _deve_ declarar o tipo de cada parâmetro. Esta é
uma decisão deliberada no design do Rust: Exigir anotações de tipo em definições de
função significa que o compilador quase nunca precisa que você as use em outro lugar no
código para descobrir qual tipo você quer dizer. O compilador também é capaz de dar
mensagens de erro mais úteis se souber quais tipos a função espera.

Ao definir múltiplos parâmetros, separe as declarações de parâmetros com
vírgulas, como este:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Este exemplo cria uma função chamada `print_labeled_measurement` com dois
parâmetros. O primeiro parâmetro é chamado `value` e é um `i32`. O segundo é
chamado `unit_label` e é do tipo `char`. A função então imprime texto contendo
tanto o `value` quanto o `unit_label`.

Vamos tentar executar este código. Substitua o programa atualmente no arquivo _src/main.rs_ do seu
projeto _functions_ com o exemplo precedente e execute-o usando `cargo
run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Porque chamamos a função com `5` como o valor para `value` e `'h'` como
o valor para `unit_label`, a saída do programa contém esses valores.

### Declarações e Expressões

Corpos de função são compostos de uma série de declarações (statements) opcionalmente terminando em uma
expressão. Até agora, as funções que cobrimos não incluíram uma expressão
final, mas você viu uma expressão como parte de uma declaração. Como o
Rust é uma linguagem baseada em expressões, esta é uma distinção importante para
entender. Outras linguagens não têm as mesmas distinções, então vamos olhar para
o que são declarações e expressões e como suas diferenças afetam os corpos
das funções.

- _Declarações_ são instruções que realizam alguma ação e não retornam
  um valor.
- _Expressões_ avaliam para um valor resultante.

Vamos olhar para alguns exemplos.

Na verdade, já usamos declarações e expressões. Criar uma variável e
atribuir um valor a ela com a palavra-chave `let` é uma declaração. Na Listagem 3-1,
`let y = 6;` é uma declaração.

<Listing number="3-1" file-name="src/main.rs" caption="Uma declaração de função `main` contendo uma declaração">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Definições de função também são declarações; todo o exemplo precedente é uma
declaração em si. (Como veremos em breve, chamar uma função não é uma
declaração, no entanto.)

Declarações não retornam valores. Portanto, você não pode atribuir uma declaração `let`
a outra variável, como o código a seguir tenta fazer; você receberá um erro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Quando você executa este programa, o erro que você receberá se parece com este:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

A declaração `let y = 6` não retorna um valor, então não há nada para
`x` se vincular. Isso é diferente do que acontece em outras linguagens, como
C e Ruby, onde a atribuição retorna o valor da atribuição. Nessas
linguagens, você pode escrever `x = y = 6` e ter tanto `x` quanto `y` com o valor
`6`; esse não é o caso em Rust.

Expressões avaliam para um valor e compõem a maior parte do resto do código que
você escreverá em Rust. Considere uma operação matemática, como `5 + 6`, que é uma
expressão que avalia para o valor `11`. Expressões podem ser parte de
declarações: Na Listagem 3-1, o `6` na declaração `let y = 6;` é uma
expressão que avalia para o valor `6`. Chamar uma função é uma
expressão. Chamar uma macro é uma expressão. Um novo bloco de escopo criado com
chaves é uma expressão, por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Esta expressão:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

é um bloco que, neste caso, avalia para `4`. Esse valor é vinculado a `y`
como parte da declaração `let`. Note a linha `x + 1` sem um ponto e vírgula no
final, o que é diferente da maioria das linhas que você viu até agora. Expressões não
incluem pontos e vírgulas finais. Se você adicionar um ponto e vírgula ao final de uma
expressão, você a transforma em uma declaração, e ela então não retornará um valor.
Tenha isso em mente enquanto explora valores de retorno de função e expressões a seguir.

### Funções com Valores de Retorno

Funções podem retornar valores para o código que as chama. Não nomeamos valores de
retorno, mas devemos declarar seu tipo após uma seta (`->`). Em Rust, o
valor de retorno da função é sinônimo do valor da expressão final
no bloco do corpo de uma função. Você pode retornar cedo de uma
função usando a palavra-chave `return` e especificando um valor, mas a maioria das
funções retorna a última expressão implicitamente. Aqui está um exemplo de uma
função que retorna um valor:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Não há chamadas de função, macros, ou mesmo declarações `let` na função
`five`—apenas o número `5` por si só. Essa é uma função perfeitamente válida em
Rust. Note que o tipo de retorno da função é especificado também, como `-> i32`. Tente
executar este código; a saída deve se parecer com esta:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

O `5` em `five` é o valor de retorno da função, e é por isso que o tipo de retorno
é `i32`. Vamos examinar isso com mais detalhes. Há duas partes importantes:
Primeiro, a linha `let x = five();` mostra que estamos usando o valor de retorno de uma
função para inicializar uma variável. Porque a função `five` retorna um `5`,
essa linha é a mesma que a seguinte:

```rust
let x = 5;
```

Segundo, a função `five` não tem parâmetros e define o tipo do
valor de retorno, mas o corpo da função é um `5` solitário sem ponto e vírgula
porque é uma expressão cujo valor queremos retornar.

Vamos olhar para outro exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Executar este código imprimirá `The value of x is: 6`. Mas o que acontece se
colocarmos um ponto e vírgula no final da linha contendo `x + 1`, mudando-a de
uma expressão para uma declaração?

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Compilar este código produzirá um erro, como segue:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

A mensagem de erro principal, `mismatched types` (tipos incompatíveis), revela o problema central com este
código. A definição da função `plus_one` diz que ela retornará um
`i32`, mas declarações não avaliam para um valor, o que é expresso por `()`,
o tipo unitário. Portanto, nada é retornado, o que contradiz a definição da função
e resulta em um erro. Nesta saída, o Rust fornece uma mensagem para
possivelmente ajudar a retificar este problema: Ele sugere remover o ponto e vírgula, o que
corrigiria o erro.
