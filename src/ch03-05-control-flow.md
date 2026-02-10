# Controle de Fluxo

A habilidade de executar algum código dependendo se uma condição é `true` e a
habilidade de executar algum código repetidamente enquanto uma condição é `true` são blocos de construção
básicos na maioria das linguagens de programação. As construções mais comuns que
permitem controlar o fluxo de execução do código Rust são expressões `if` e
loops.

### Expressões `if`

Uma expressão `if` permite ramificar seu código dependendo de condições. Você
fornece uma condição e então declara, “Se esta condição for atendida, execute este bloco
de código. Se a condição não for atendida, não execute este bloco de código.”

Crie um novo projeto chamado _branches_ no seu diretório _projects_ para explorar
a expressão `if`. No arquivo _src/main.rs_, insira o seguinte:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Todas as expressões `if` começam com a palavra-chave `if`, seguida por uma condição. Neste
caso, a condição verifica se a variável `number` tem ou não um
valor menor que 5. Colocamos o bloco de código para executar se a condição for
`true` imediatamente após a condição dentro de chaves. Blocos de código
associados com as condições em expressões `if` são às vezes chamados de _braços_ (arms),
assim como os braços em expressões `match` que discutimos na seção [[ch02-00-guessing-game-tutorial.md#comparing-the-guess-to-the-secret-number|“Comparando o Palpite com o Número Secreto”]]<!--
ignore --> do Capítulo 2.

Opcionalmente, também podemos incluir uma expressão `else`, o que escolhemos fazer
aqui, para dar ao programa um bloco de código alternativo para executar caso a
condição avalie para `false`. Se você não fornecer uma expressão `else` e
a condição for `false`, o programa apenas pulará o bloco `if` e seguirá
para o próximo trecho de código.

Tente executar este código; você deve ver a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Vamos tentar mudar o valor de `number` para um valor que torne a condição
`false` para ver o que acontece:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Execute o programa novamente e olhe para a saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Também vale a pena notar que a condição neste código _deve_ ser um `bool`. Se
a condição não for um `bool`, teremos um erro. Por exemplo, tente executar o
seguinte código:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

A condição `if` avalia para um valor de `3` desta vez, e o Rust lança um
erro:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

O erro indica que o Rust esperava um `bool` mas obteve um inteiro. Ao contrário de
linguagens como Ruby e JavaScript, o Rust não tentará automaticamente
converter tipos não-Booleanos para um Booleano. Você deve ser explícito e sempre fornecer
ao `if` um Booleano como sua condição. Se quisermos que o bloco de código `if` execute
apenas quando um número não for igual a `0`, por exemplo, podemos mudar a expressão `if`
para o seguinte:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Executar este código imprimirá `number was something other than zero`.

#### Lidando com Múltiplas Condições com `else if`

Você pode usar múltiplas condições combinando `if` e `else` em uma expressão `else if`.
Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Este programa tem quatro caminhos possíveis que pode tomar. Após executá-lo, você deve
ver a seguinte saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Quando este programa executa, ele verifica cada expressão `if` por vez e executa
o primeiro corpo para o qual a condição avalia para `true`. Note que mesmo
que 6 seja divisível por 2, não vemos a saída `number is divisible by 2`,
nem vemos o texto `number is not divisible by 4, 3, or 2` do bloco `else`.
Isso é porque o Rust só executa o bloco para a primeira condição `true`,
e uma vez que ele encontra uma, ele nem mesmo verifica o resto.

Usar muitas expressões `else if` pode desorganizar seu código, então se você tiver mais
de uma, você pode querer refatorar seu código. O Capítulo 6 descreve uma poderosa
construção de ramificação do Rust chamada `match` para esses casos.

#### Usando `if` em uma Declaração `let`

Como `if` é uma expressão, podemos usá-lo no lado direito de uma declaração `let`
para atribuir o resultado a uma variável, como na Listagem 3-2.

<Listing number="3-2" file-name="src/main.rs" caption="Atribuindo o resultado de uma expressão `if` a uma variável">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

A variável `number` será vinculada a um valor baseado no resultado da expressão `if`.
Execute este código para ver o que acontece:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Lembre-se de que blocos de código avaliam para a última expressão neles, e
números por si mesmos também são expressões. Neste caso, o valor de
toda a expressão `if` depende de qual bloco de código executa. Isso significa que os
valores que têm o potencial de ser resultados de cada braço do `if` devem ser
do mesmo tipo; na Listagem 3-2, os resultados tanto do braço `if` quanto do braço `else`
foram inteiros `i32`. Se os tipos forem incompatíveis, como no exemplo a seguir,
teremos um erro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Quando tentamos compilar este código, teremos um erro. Os braços `if` e `else`
têm tipos de valor que são incompatíveis, e o Rust indica exatamente onde
encontrar o problema no programa:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

A expressão no bloco `if` avalia para um inteiro, e a expressão no
bloco `else` avalia para uma string. Isso não funcionará, porque variáveis devem
ter um único tipo, e o Rust precisa saber definitivamente em tempo de compilação qual
tipo a variável `number` é. Saber o tipo de `number` permite que o compilador
verifique se o tipo é válido em todos os lugares em que usamos `number`. O Rust não seria capaz de
fazer isso se o tipo de `number` fosse determinado apenas em tempo de execução; o compilador
seria mais complexo e faria menos garantias sobre o código se tivesse
que acompanhar múltiplos tipos hipotéticos para qualquer variável.

### Repetição com Loops

É frequentemente útil executar um bloco de código mais de uma vez. Para esta tarefa,
o Rust fornece vários _loops_, que executarão o código dentro do corpo do loop
até o fim e então começarão imediatamente de volta no início. Para experimentar
com loops, vamos fazer um novo projeto chamado _loops_.

O Rust tem três tipos de loops: `loop`, `while` e `for`. Vamos tentar cada um.

#### Repetindo Código com `loop`

A palavra-chave `loop` diz ao Rust para executar um bloco de código repetidamente para sempre
ou até que você diga explicitamente para parar.

Como um exemplo, mude o arquivo _src/main.rs_ no seu diretório _loops_ para parecer
com isto:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Quando executamos este programa, veremos `again!` impresso repetidamente continuamente
até pararmos o programa manualmente. A maioria dos terminais suporta o atalho de teclado
<kbd>ctrl</kbd>-<kbd>C</kbd> para interromper um programa que está preso em um loop
contínuo. Tente:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

O símbolo `^C` representa onde você pressionou <kbd>ctrl</kbd>-<kbd>C</kbd>.

Você pode ou não ver a palavra `again!` impressa após o `^C`, dependendo de
onde o código estava no loop quando recebeu o sinal de interrupção.

Felizmente, o Rust também fornece uma maneira de sair de um loop usando código. Você
pode colocar a palavra-chave `break` dentro do loop para dizer ao programa quando parar
de executar o loop. Lembre-se que fizemos isso no jogo de adivinhação na seção
[“Saindo Após um Palpite Correto”][quitting-after-a-correct-guess]<!-- ignore
--> do Capítulo 2 para sair do programa quando o usuário ganhou o jogo
adivinhando o número correto.

Também usamos `continue` no jogo de adivinhação, que em um loop diz ao programa
para pular qualquer código restante nesta iteração do loop e ir para a
próxima iteração.

#### Retornando Valores de Loops

Um dos usos de um `loop` é tentar novamente uma operação que você sabe que pode falhar, como
verificar se uma thread completou seu trabalho. Você também pode precisar passar
o resultado dessa operação para fora do loop para o resto do seu código. Para fazer
isso, você pode adicionar o valor que você quer retornado após a expressão `break` que você
usa para parar o loop; esse valor será retornado para fora do loop para que você
possa usá-lo, como mostrado aqui:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Antes do loop, declaramos uma variável chamada `counter` e a inicializamos com
`0`. Então, declaramos uma variável chamada `result` para segurar o valor retornado do
loop. Em cada iteração do loop, adicionamos `1` à variável `counter`,
e então verificamos se o `counter` é igual a `10`. Quando é, usamos a
palavra-chave `break` com o valor `counter * 2`. Após o loop, usamos um
ponto e vírgula para terminar a declaração que atribui o valor a `result`. Finalmente,
imprimimos o valor em `result`, que neste caso é `20`.

Você também pode usar `return` de dentro de um loop. Enquanto `break` apenas sai do loop
atual, `return` sempre sai da função atual.

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### Desambiguando com Rótulos de Loop (Loop Labels)

Se você tem loops dentro de loops, `break` e `continue` se aplicam ao loop
mais interno naquele ponto. Você pode opcionalmente especificar um _rótulo de loop_ em um loop
que você pode então usar com `break` ou `continue` para especificar que essas palavras-chave
se aplicam ao loop rotulado em vez do loop mais interno. Rótulos de loop devem começar
com uma aspa simples. Aqui está um exemplo com dois loops aninhados:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

O loop externo tem o rótulo `'counting_up`, e ele contará de 0 a 2.
O loop interno sem rótulo conta de 10 a 9. O primeiro `break` que
não especifica um rótulo sairá apenas do loop interno. A declaração `break
'counting_up;` sairá do loop externo. Este código imprime:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### Simplificando Loops Condicionais com while

Um programa frequentemente precisará avaliar uma condição dentro de um loop. Enquanto a
condição for `true`, o loop executa. Quando a condição deixar de ser `true`, o
programa chama `break`, parando o loop. É possível implementar comportamento
como este usando uma combinação de `loop`, `if`, `else`, e `break`; você poderia
tentar isso agora em um programa, se quisesse. No entanto, esse padrão é tão comum
que o Rust tem uma construção de linguagem embutida para isso, chamada de loop `while`. Na
Listagem 3-3, usamos `while` para repetir o programa três vezes, contando para baixo a cada
vez, e então, após o loop, imprimir uma mensagem e sair.

<Listing number="3-3" file-name="src/main.rs" caption="Usando um loop `while` para executar código enquanto uma condição avalia para `true`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Essa construção elimina muito aninhamento que seria necessário se você usasse
`loop`, `if`, `else`, e `break`, e é mais clara. Enquanto uma condição
avalia para `true`, o código executa; caso contrário, ele sai do loop.

#### Loop Através de uma Coleção com `for`

Você pode escolher usar a construção `while` para fazer loop sobre os elementos de uma
coleção, como um array. Por exemplo, o loop na Listagem 3-4 imprime cada
elemento no array `a`.

<Listing number="3-4" file-name="src/main.rs" caption="Fazendo loop através de cada elemento de uma coleção usando um loop `while`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Aqui, o código conta através dos elementos no array. Ele começa no índice
`0` e então faz o loop até atingir o índice final no array (isto é,
quando `index < 5` não for mais `true`). Executar este código imprimirá cada
elemento no array:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Todos os cinco valores do array aparecem no terminal, como esperado. Mesmo que `index`
atinja um valor de `5` em algum momento, o loop para de executar antes de tentar
buscar um sexto valor do array.

No entanto, essa abordagem é propensa a erros; poderíamos fazer o programa entrar em pânico se
o valor do índice ou a condição de teste estiver incorreta. Por exemplo, se você mudasse a
definição do array `a` para ter quatro elementos mas esquecesse de atualizar a
condição para `while index < 4`, o código entraria em pânico. Também é lento, porque
o compilador adiciona código em tempo de execução para realizar a verificação condicional de se o
índice está dentro dos limites do array em cada iteração através do loop.

Como uma alternativa mais concisa, você pode usar um loop `for` e executar algum código
para cada item em uma coleção. Um loop `for` se parece com o código na Listagem 3-5.

<Listing number="3-5" file-name="src/main.rs" caption="Fazendo loop através de cada elemento de uma coleção usando um loop `for`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Quando executamos este código, veremos a mesma saída que na Listagem 3-4. Mais
importante, agora aumentamos a segurança do código e eliminamos a
chance de bugs que podem resultar de ir além do final do array ou não
ir longe o suficiente e perder alguns itens. Código de máquina gerado de loops `for`
pode ser mais eficiente também porque o índice não precisa ser
comparado ao comprimento do array em cada iteração.

Usando o loop `for`, você não precisaria lembrar de mudar qualquer outro código se
você mudasse o número de valores no array, como faria com o método
usado na Listagem 3-4.

A segurança e concisão dos loops `for` os tornam a construção de loop mais comumente usada
em Rust. Mesmo em situações em que você quer executar algum código um
certo número de vezes, como no exemplo de contagem regressiva que usou um loop `while`
na Listagem 3-3, a maioria dos Rustaceans usaria um loop `for`. A maneira de fazer isso
seria usar um `Range`, fornecido pela biblioteca padrão, que gera
todos os números em sequência começando de um número e terminando antes de outro
número.

Aqui está como a contagem regressiva ficaria usando um loop `for` e outro método
que ainda não falamos, `rev`, para reverter o intervalo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Este código é um pouco melhor, não é?

## Resumo

Você conseguiu! Este foi um capítulo considerável: Você aprendeu sobre variáveis, tipos de dados escalares
e compostos, funções, comentários, expressões `if`, e loops! Para
praticar com os conceitos discutidos neste capítulo, tente construir programas para
fazer o seguinte:

- Converter temperaturas entre Fahrenheit e Celsius.
- Gerar o *n*-ésimo número de Fibonacci.
- Imprimir a letra da canção de Natal “The Twelve Days of Christmas,”
  aproveitando a repetição na música.

Quando você estiver pronto para seguir em frente, falaremos sobre um conceito em Rust que _não_
existe comumente em outras linguagens de programação: ownership (propriedade).

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.md#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.md#quitting-after-a-correct-guess
