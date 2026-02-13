## Todos os Lugares Onde Padrões Podem Ser Usados

Padrões aparecem em vários lugares em Rust, e você tem usado muito eles
sem perceber! Esta seção discute todos os lugares onde padrões são válidos.

### Braços de `match`

Como discutido no Capítulo 6, usamos padrões nos braços de expressões `match`.
Formalmente, expressões `match` são definidas como a palavra-chave `match`, um valor para
casar, e um ou mais braços de match que consistem em um padrão e uma
expressão para executar se o valor casar com o padrão daquele braço, assim:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALOR</em> {
    <em>PADRÃO</em> => <em>EXPRESSÃO</em>,
    <em>PADRÃO</em> => <em>EXPRESSÃO</em>,
    <em>PADRÃO</em> => <em>EXPRESSÃO</em>,
}</code></pre>

Por exemplo, aqui está a expressão `match` da Listagem 6-5 que casa com um
valor `Option<i32>` na variável `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Os padrões nesta expressão `match` são o `None` e `Some(i)` à esquerda de cada seta.

Um requisito para expressões `match` é que elas precisam ser exaustivas no
sentido de que todas as possibilidades para o valor na expressão `match` devem
ser levadas em conta. Uma maneira de garantir que você cobriu todas as possibilidades é
ter um padrão pega-tudo para o último braço: Por exemplo, um nome de variável
casando com qualquer valor nunca pode falhar e, portanto, cobre todos os casos restantes.

O padrão específico `_` irá casar com qualquer coisa, mas nunca se vincula a uma
variável, então é frequentemente usado no último braço de match. O padrão `_` pode ser
útil quando você quer ignorar qualquer valor não especificado, por exemplo. Nós
cobriremos o padrão `_` em mais detalhes em ["Ignorando Valores em um
Padrão"][ignoring-values-in-a-pattern]<!-- ignore --> mais adiante neste capítulo.

### Instruções `let`

Antes deste capítulo, havíamos discutido explicitamente apenas o uso de padrões com
`match` e `if let`, mas, de fato, usamos padrões em outros lugares também,
incluindo em instruções `let`. Por exemplo, considere esta atribuição de variável
direta com `let`:

```rust
let x = 5;
```

Toda vez que você usou uma instrução `let` como essa, você estava usando padrões,
embora possa não ter percebido! Mais formalmente, uma instrução `let` se parece
com isso:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PADRÃO</em> = <em>EXPRESSÃO</em>;</code>
</pre>

Em instruções como `let x = 5;` com um nome de variável no espaço PADRÃO, o
nome da variável é apenas uma forma particularmente simples de um padrão. Rust compara
a expressão contra o padrão e atribui quaisquer nomes que encontrar. Então, no
exemplo `let x = 5;`, `x` é um padrão que significa "vincule o que casar aqui
à variável `x`." Como o nome `x` é o padrão inteiro, este padrão
efetivamente significa "vincule tudo à variável `x`, qualquer que seja o valor."

Para ver o aspecto de casamento de padrões do `let` mais claramente, considere a Listagem
19-1, que usa um padrão com `let` para desestruturar uma tupla.

<Listing number="19-1" caption="Usando um padrão para desestruturar uma tupla e criar três variáveis de uma vez">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

Aqui, casamos uma tupla contra um padrão. Rust compara o valor `(1, 2, 3)`
ao padrão `(x, y, z)` e vê que o valor casa com o padrão — isto é,
ele vê que o número de elementos é o mesmo em ambos — então Rust vincula `1` a
`x`, `2` a `y`, e `3` a `z`. Você pode pensar neste padrão de tupla como aninhando
três padrões de variáveis individuais dentro dele.

Se o número de elementos no padrão não corresponder ao número de elementos
na tupla, o tipo geral não corresponderá e teremos um erro de compilação. Por
exemplo, a Listagem 19-2 mostra uma tentativa de desestruturar uma tupla com três
elementos em duas variáveis, o que não funcionará.

<Listing number="19-2" caption="Construindo incorretamente um padrão cujas variáveis não correspondem ao número de elementos na tupla">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Tentar compilar este código resulta neste erro de tipo:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

Para corrigir o erro, poderíamos ignorar um ou mais dos valores na tupla usando
`_` ou `..`, como você verá na seção ["Ignorando Valores em um
Padrão"][ignoring-values-in-a-pattern]<!-- ignore -->. Se o problema
é que temos muitas variáveis no padrão, a solução é fazer os
tipos corresponderem removendo variáveis para que o número de variáveis seja igual ao
número de elementos na tupla.

### Expressões Condicionais `if let`

No Capítulo 6, discutimos como usar expressões `if let` principalmente como uma maneira mais curta
de escrever o equivalente a um `match` que apenas casa com um caso.
Opcionalmente, `if let` pode ter um `else` correspondente contendo código para executar se
o padrão no `if let` não casar.

A Listagem 19-3 mostra que também é possível misturar e combinar `if let`, `else
if` e expressões `else if let`. Fazer isso nos dá mais flexibilidade do que uma
expressão `match` na qual podemos expressar apenas um valor para comparar com os
padrões. Além disso, Rust não exige que as condições em uma série de braços `if
let`, `else if` e `else if let` se relacionem entre si.

O código na Listagem 19-3 determina qual cor usar como fundo com base em
uma série de verificações para várias condições. Para este exemplo, criamos
variáveis com valores fixos que um programa real poderia receber da entrada do usuário.

<Listing number="19-3" file-name="src/main.rs" caption="Misturando `if let`, `else if`, `else if let` e `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

Se o usuário especificar uma cor favorita, essa cor é usada como plano de fundo.
Se nenhuma cor favorita for especificada e hoje for terça-feira, a cor de fundo é
verde. Caso contrário, se o usuário especificar sua idade como uma string e pudermos analisar
isso como um número com sucesso, a cor é roxa ou laranja dependendo do
valor do número. Se nenhuma dessas condições se aplicar, a cor de fundo
é azul.

Esta estrutura condicional nos permite suportar requisitos complexos. Com os
valores fixos que temos aqui, este exemplo imprimirá `Usando roxo como a
cor de fundo`.

Você pode ver que `if let` também pode introduzir novas variáveis que sombreiam variáveis
existentes da mesma maneira que braços de `match` podem: A linha `if let Ok(age) = age`
introduz uma nova variável `age` que contém o valor dentro da variante `Ok`,
sombreando a variável `age` existente. Isso significa que precisamos colocar a condição `if age >
30` dentro desse bloco: Não podemos combinar essas duas condições em `if
let Ok(age) = age && age > 30`. O novo `age` que queremos comparar com 30 não é
válido até que o novo escopo comece com a chave.

A desvantagem de usar expressões `if let` é que o compilador não verifica
exaustividade, enquanto com expressões `match` ele verifica. Se omitíssemos o
último bloco `else` e, portanto, deixássemos de tratar alguns casos, o compilador não
nos alertaria sobre o possível bug lógico.

### Loops Condicionais `while let`

Semelhante em construção ao `if let`, o loop condicional `while let` permite que um
loop `while` execute enquanto um padrão continuar a casar. Na Listagem
19-4, mostramos um loop `while let` que espera por mensagens enviadas entre threads,
mas neste caso verificando um `Result` em vez de uma `Option`.

<Listing number="19-4" caption="Usando um loop `while let` para imprimir valores enquanto `rx.recv()` retornar `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Este exemplo imprime `1`, `2` e depois `3`. O método `recv` pega a primeira
mensagem do lado receptor do canal e retorna um `Ok(valor)`. Quando
vimos `recv` pela primeira vez no Capítulo 16, desembrulhamos o erro diretamente, ou
interagimos com ele como um iterador usando um loop `for`. Como a Listagem 19-4 mostra,
no entanto, também podemos usar `while let`, porque o método `recv` retorna um `Ok`
cada vez que uma mensagem chega, desde que o remetente exista, e então produz um
`Err` assim que o lado remetente se desconecta.

### Loops `for`

Em um loop `for`, o valor que segue diretamente a palavra-chave `for` é um
padrão. Por exemplo, em `for x in y`, o `x` é o padrão. A Listagem 19-5
demonstra como usar um padrão em um loop `for` para desestruturar, ou quebrar
em partes, uma tupla como parte do loop `for`.

<Listing number="19-5" caption="Usando um padrão em um loop `for` para desestruturar uma tupla">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

O código na Listagem 19-5 imprimirá o seguinte:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

Adaptamos um iterador usando o método `enumerate` para que ele produza um valor
e o índice para esse valor, colocados em uma tupla. O primeiro valor produzido é
a tupla `(0, 'a')`. Quando este valor é casado com o padrão `(index,
value)`, index será `0` e value será `'a'`, imprimindo a primeira linha da
saída.

### Parâmetros de Função

Parâmetros de função também podem ser padrões. O código na Listagem 19-6, que
declara uma função chamada `foo` que recebe um parâmetro chamado `x` do tipo
`i32`, deve parecer familiar agora.

<Listing number="19-6" caption="Uma assinatura de função usando padrões nos parâmetros">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

A parte `x` é um padrão! Como fizemos com `let`, poderíamos casar uma tupla nos
argumentos de uma função ao padrão. A Listagem 19-7 divide os valores em uma tupla
ao passá-la para uma função.

<Listing number="19-7" file-name="src/main.rs" caption="Uma função com parâmetros que desestruturam uma tupla">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Este código imprime `Localização atual: (3, 5)`. Os valores `&(3, 5)` casam com o
padrão `&(x, y)`, então `x` é o valor `3` e `y` é o valor `5`.

Também podemos usar padrões em listas de parâmetros de closure da mesma maneira que em
listas de parâmetros de função, porque closures são semelhantes a funções, como
discutido no Capítulo 13.

Neste ponto, você viu várias maneiras de usar padrões, mas padrões não
funcionam da mesma maneira em todos os lugares onde podemos usá-los. Em alguns lugares, os padrões devem
ser irrefutáveis; em outras circunstâncias, eles podem ser refutáveis. Discutiremos
esses dois conceitos a seguir.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
