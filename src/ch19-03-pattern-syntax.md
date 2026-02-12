## Sintaxe de Padrões

Nesta seção, reunimos toda a sintaxe que é válida em padrões e discutimos
por que e quando você pode querer usar cada uma.

### Casando Literais

Como você viu no Capítulo 6, você pode casar padrões contra literais diretamente. O
código a seguir dá alguns exemplos:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Este código imprime `um` porque o valor em `x` é `1`. Esta sintaxe é útil
quando você quer que seu código tome uma ação se ele obtiver um valor concreto
particular.

### Casando Variáveis Nomeadas

Variáveis nomeadas são padrões irrefutáveis que casam com qualquer valor, e nós as usamos
muitas vezes neste livro. No entanto, há uma complicação quando você usa
variáveis nomeadas em expressões `match`, `if let` ou `while let`. Como cada
um desses tipos de expressões inicia um novo escopo, variáveis declaradas como parte de
um padrão dentro dessas expressões sombrearão aquelas com o mesmo nome fora
das construções, como é o caso com todas as variáveis. Na Listagem 19-11, declaramos
uma variável chamada `x` com o valor `Some(5)` e uma variável `y` com o valor
`10`. Então criamos uma expressão `match` no valor `x`. Olhe para os
padrões nos braços do match e `println!` no final, e tente descobrir
o que o código imprimirá antes de executar este código ou ler mais.

<Listing number="19-11" file-name="src/main.rs" caption="Uma expressão `match` com um braço que introduz uma nova variável que sombreia uma variável existente `y`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

Vamos percorrer o que acontece quando a expressão `match` é executada. O padrão
no primeiro braço do match não casa com o valor definido de `x`, então o código
continua.

O padrão no segundo braço do match introduz uma nova variável chamada `y` que
casará com qualquer valor dentro de um valor `Some`. Como estamos em um novo escopo dentro
da expressão `match`, este é um novo `y`, não o `y` que declaramos no
início com o valor `10`. Esta nova vinculação `y` casará com qualquer valor
dentro de um `Some`, que é o que temos em `x`. Portanto, este novo `y` se vincula ao
valor interno do `Some` em `x`. Esse valor é `5`, então a expressão para
aquele braço executa e imprime `Casou, y = 5`.

Se `x` tivesse sido um valor `None` em vez de `Some(5)`, os padrões nos primeiros
dois braços não teriam casado, então o valor teria casado com o
sublinhado. Não introduzimos a variável `x` no padrão do
braço do sublinhado, então o `x` na expressão ainda é o `x` externo que não foi
sombreado. Neste caso hipotético, o `match` imprimiria `Caso padrão,
x = None`.

Quando a expressão `match` termina, seu escopo termina, e o mesmo acontece com o escopo do
`y` interno. O último `println!` produz `no final: x = Some(5), y = 10`.

Para criar uma expressão `match` que compara os valores do `x` externo e
`y`, em vez de introduzir uma nova variável que sombreia a variável `y`
existente, precisaríamos usar uma condicional match guard. Falaremos
sobre match guards mais tarde na seção ["Adicionando Condicionais com Match
Guards"](#adding-conditionals-with-match-guards)<!-- ignore -->.

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

### Casando Múltiplos Padrões

Em expressões `match`, você pode casar múltiplos padrões usando a sintaxe `|`,
que é o operador de padrão _ou_. Por exemplo, no código a seguir, casamos
o valor de `x` contra os braços de match, o primeiro dos quais tem uma opção _ou_,
significando que se o valor de `x` casar com qualquer um dos valores naquele braço, o
código daquele braço será executado:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Este código imprime `um ou dois`.

### Casando Faixas de Valores com `..=`

A sintaxe `..=` nos permite casar com uma faixa inclusiva de valores. No
código a seguir, quando um padrão casa com qualquer um dos valores dentro da faixa
dada, aquele braço será executado:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Se `x` for `1`, `2`, `3`, `4` ou `5`, o primeiro braço casará. Esta sintaxe é
mais conveniente para múltiplos valores de match do que usar o operador `|` para
expressar a mesma ideia; se fôssemos usar `|`, teríamos que especificar `1 | 2 |
3 | 4 | 5`. Especificar uma faixa é muito mais curto, especialmente se quisermos casar,
digamos, qualquer número entre 1 e 1.000!

O compilador verifica se a faixa não está vazia em tempo de compilação, e como os
únicos tipos para os quais Rust pode dizer se uma faixa está vazia ou não são `char` e
valores numéricos, faixas só são permitidas com valores numéricos ou `char`.

Aqui está um exemplo usando faixas de valores `char`:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust pode dizer que `'c'` está dentro da faixa do primeiro padrão e imprime `letra
ASCII inicial`.

### Desestruturando para Quebrar Valores

Também podemos usar padrões para desestruturar structs, enums e tuplas para usar
diferentes partes desses valores. Vamos percorrer cada valor.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

#### Structs

A Listagem 19-12 mostra uma struct `Point` com dois campos, `x` e `y`, que podemos
quebrar usando um padrão com uma instrução `let`.

<Listing number="19-12" file-name="src/main.rs" caption="Desestruturando os campos de uma struct em variáveis separadas">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Este código cria as variáveis `a` e `b` que casam com os valores dos campos `x`
e `y` da struct `p`. Este exemplo mostra que os nomes das
variáveis no padrão não precisam corresponder aos nomes dos campos da struct.
No entanto, é comum corresponder os nomes das variáveis aos nomes dos campos para tornar
mais fácil lembrar quais variáveis vieram de quais campos. Por causa deste
uso comum, e porque escrever `let Point { x: x, y: y } = p;` contém muita
duplicação, Rust tem uma abreviação para padrões que casam com campos de struct:
Você só precisa listar o nome do campo da struct, e as variáveis criadas
a partir do padrão terão os mesmos nomes. A Listagem 19-13 se comporta da mesma
maneira que o código na Listagem 19-12, mas as variáveis criadas no padrão
`let` são `x` e `y` em vez de `a` e `b`.

<Listing number="19-13" file-name="src/main.rs" caption="Desestruturando campos de struct usando abreviação de campo de struct">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Este código cria as variáveis `x` e `y` que casam com os campos `x` e `y`
da variável `p`. O resultado é que as variáveis `x` e `y` contêm os
valores da struct `p`.

Também podemos desestruturar com valores literais como parte do padrão de struct
em vez de criar variáveis para todos os campos. Fazer isso nos permite testar
alguns dos campos para valores particulares enquanto criamos variáveis para
desestruturar os outros campos.

Na Listagem 19-14, temos uma expressão `match` que separa valores `Point`
em três casos: pontos que ficam diretamente no eixo `x` (o que é verdade quando
`y = 0`), no eixo `y` (`x = 0`), ou em nenhum dos eixos.

<Listing number="19-14" file-name="src/main.rs" caption="Desestruturando e casando valores literais em um padrão">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

O primeiro braço casará com qualquer ponto que fique no eixo `x` especificando que
o campo `y` casa se seu valor casar com o literal `0`. O padrão ainda
cria uma variável `x` que podemos usar no código para este braço.

Da mesma forma, o segundo braço casa com qualquer ponto no eixo `y` especificando que
o campo `x` casa se seu valor for `0` e cria uma variável `y` para o
valor do campo `y`. O terceiro braço não especifica nenhum literal, então ele
casa com qualquer outro `Point` e cria variáveis para ambos os campos `x` e `y`.

Neste exemplo, o valor `p` casa com o segundo braço em virtude de `x`
conter um `0`, então este código imprimirá `No eixo y em 7`.

Lembre-se de que uma expressão `match` para de verificar braços assim que encontrar o
primeiro padrão correspondente, então, embora `Point { x: 0, y: 0 }` esteja no eixo `x`
e no eixo `y`, este código imprimiria apenas `No eixo x em 0`.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

#### Enums

Desestruturamos enums neste livro (por exemplo, Listagem 6-5 no Capítulo 6),
mas ainda não discutimos explicitamente que o padrão para desestruturar um enum
corresponde à maneira como os dados armazenados dentro do enum são definidos. Como um
exemplo, na Listagem 19-15, usamos o enum `Message` da Listagem 6-2 e escrevemos
um `match` com padrões que desestruturarão cada valor interno.

<Listing number="19-15" file-name="src/main.rs" caption="Desestruturando variantes de enum que contêm diferentes tipos de valores">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Este código imprimirá `Mudar cor para vermelho 0, verde 160, e azul 255`. Tente
mudar o valor de `msg` para ver o código dos outros braços rodar.

Para variantes de enum sem dados, como `Message::Quit`, não podemos desestruturar
o valor mais. Só podemos casar com o valor literal `Message::Quit`,
e nenhuma variável está nesse padrão.

Para variantes de enum semelhantes a struct, como `Message::Move`, podemos usar um padrão
semelhante ao padrão que especificamos para casar structs. Após o nome da variante,
colocamos chaves e então listamos os campos com variáveis para que quebremos
as partes para usar no código para este braço. Aqui usamos a forma abreviada
como fizemos na Listagem 19-13.

Para variantes de enum semelhantes a tupla, como `Message::Write` que contém uma tupla com um
elemento e `Message::ChangeColor` que contém uma tupla com três elementos, o
padrão é semelhante ao padrão que especificamos para casar tuplas. O número de
variáveis no padrão deve corresponder ao número de elementos na variante que estamos
casando.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

#### Structs e Enums Aninhados

Até agora, nossos exemplos foram todos casando structs ou enums com um nível de profundidade,
mas o casamento pode funcionar em itens aninhados também! Por exemplo, podemos refatorar o
código na Listagem 19-15 para suportar cores RGB e HSV na mensagem `ChangeColor`,
como mostrado na Listagem 19-16.

<Listing number="19-16" caption="Casando em enums aninhados">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

O padrão do primeiro braço na expressão `match` casa com uma
variante de enum `Message::ChangeColor` que contém uma variante `Color::Rgb`; então,
o padrão se vincula aos três valores `i32` internos. O padrão do segundo
braço também casa com uma variante de enum `Message::ChangeColor`, mas o enum interno
casa com `Color::Hsv`. Podemos especificar essas condições complexas em uma
expressão `match`, mesmo que dois enums estejam envolvidos.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

#### Structs e Tuplas

Podemos misturar, combinar e aninhar padrões de desestruturação de maneiras ainda mais complexas.
O exemplo a seguir mostra uma desestruturação complicada onde aninhamos structs e
tuplas dentro de uma tupla e desestruturamos todos os valores primitivos:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Este código nos permite quebrar tipos complexos em suas partes componentes para que possamos
usar os valores em que estamos interessados separadamente.

Desestruturar com padrões é uma maneira conveniente de usar pedaços de valores, como
o valor de cada campo em uma struct, separadamente um do outro.

### Ignorando Valores em um Padrão

Você viu que às vezes é útil ignorar valores em um padrão, como
no último braço de um `match`, para obter um pega-tudo que na verdade não faz
nada, mas leva em conta todos os valores possíveis restantes. Existem algumas
maneiras de ignorar valores inteiros ou partes de valores em um padrão: usando o padrão `_`
(que você viu), usando o padrão `_` dentro de outro padrão,
usando um nome que começa com um sublinhado, ou usando `..` para ignorar partes restantes
de um valor. Vamos explorar como e por que usar cada um desses padrões.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

#### Um Valor Inteiro com `_`

Usamos o sublinhado como um padrão curinga que casará com qualquer valor, mas
não se vinculará ao valor. Isso é especialmente útil como o último braço em uma expressão `match`,
mas também podemos usá-lo em qualquer padrão, incluindo parâmetros de função,
como mostrado na Listagem 19-17.

<Listing number="19-17" file-name="src/main.rs" caption="Usando `_` em uma assinatura de função">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Este código ignorará completamente o valor `3` passado como o primeiro argumento,
e imprimirá `Este código usa apenas o parâmetro y: 4`.

Na maioria dos casos, quando você não precisa mais de um parâmetro de função específico, você
mudaria a assinatura para que ela não incluísse o parâmetro não utilizado.
Ignorar um parâmetro de função pode ser especialmente útil em casos quando, por
exemplo, você está implementando uma trait quando precisa de uma certa assinatura de tipo, mas
o corpo da função em sua implementação não precisa de um dos parâmetros.
Você então evita receber um aviso do compilador sobre parâmetros de função não utilizados, como
você receberia se usasse um nome.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### Partes de um Valor com um `_` Aninhado

Também podemos usar `_` dentro de outro padrão para ignorar apenas parte de um valor, por
exemplo, quando queremos testar apenas parte de um valor, mas não temos uso para as
outras partes no código correspondente que queremos executar. A Listagem 19-18 mostra o código
responsável por gerenciar o valor de uma configuração. Os requisitos de negócios são que
o usuário não deve ter permissão para sobrescrever uma personalização existente de uma
configuração, mas pode desmarcar a configuração e dar a ela um valor se ela estiver atualmente desmarcada.

<Listing number="19-18" caption="Usando um sublinhado dentro de padrões que casam com variantes `Some` quando não precisamos usar o valor dentro do `Some`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Este código imprimirá `Não é possível sobrescrever um valor personalizado existente` e então
`configuração é Some(5)`. No primeiro braço do match, não precisamos casar ou usar
os valores dentro de qualquer variante `Some`, mas precisamos testar o caso
em que `setting_value` e `new_setting_value` são a variante `Some`. Nesse
caso, imprimimos o motivo para não mudar `setting_value`, e ele não é
alterado.

Em todos os outros casos (se `setting_value` ou `new_setting_value` for `None`)
expressos pelo padrão `_` no segundo braço, queremos permitir que
`new_setting_value` se torne `setting_value`.

Também podemos usar sublinhados em vários lugares dentro de um padrão para ignorar
valores particulares. A Listagem 19-19 mostra um exemplo de ignorar o segundo e
quarto valores em uma tupla de cinco itens.

<Listing number="19-19" caption="Ignorando múltiplas partes de uma tupla">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

Este código imprimirá `Alguns números: 2, 8, 32`, e os valores `4` e `16` serão
ignorados.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### Uma Variável Não Utilizada Começando Seu Nome com `_`

Se você criar uma variável, mas não a usar em lugar nenhum, Rust geralmente emitirá um
aviso porque uma variável não utilizada pode ser um bug. No entanto, às vezes é
útil poder criar uma variável que você não usará ainda, como quando você está
prototipando ou apenas começando um projeto. Nesta situação, você pode dizer a Rust
para não avisá-lo sobre a variável não utilizada começando o nome da variável
com um sublinhado. Na Listagem 19-20, criamos duas variáveis não utilizadas, mas quando
compilamos este código, devemos receber apenas um aviso sobre uma delas.

<Listing number="19-20" file-name="src/main.rs" caption="Começando um nome de variável com um sublinhado para evitar receber avisos de variáveis não utilizadas">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Aqui, recebemos um aviso sobre não usar a variável `y`, mas não recebemos um
aviso sobre não usar `_x`.

Note que há uma diferença sutil entre usar apenas `_` e usar um nome
que começa com um sublinhado. A sintaxe `_x` ainda vincula o valor à
variável, enquanto `_` não vincula de forma alguma. Para mostrar um caso onde esta
distinção importa, a Listagem 19-21 nos fornecerá um erro.

<Listing number="19-21" caption="Uma variável não utilizada começando com um sublinhado ainda vincula o valor, o que pode tomar posse do valor.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Receberemos um erro porque o valor `s` ainda será movido para `_s`,
o que nos impede de usar `s` novamente. No entanto, usar o sublinhado por si só
nunca se vincula ao valor. A Listagem 19-22 compilará sem erros
porque `s` não é movido para `_`.

<Listing number="19-22" caption="Usando um sublinhado não vincula o valor.">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

Este código funciona perfeitamente porque nunca vinculamos `s` a nada; ele não é movido.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### Partes Restantes de um Valor com `..`

Com valores que têm muitas partes, podemos usar a sintaxe `..` para usar partes específicas
e ignorar o resto, evitando a necessidade de listar sublinhados para cada
valor ignorado. O padrão `..` ignora quaisquer partes de um valor que não tenhamos
explicitamente casado no resto do padrão. Na Listagem 19-23, temos uma
struct `Point` que contém uma coordenada no espaço tridimensional. Na
expressão `match`, queremos operar apenas na coordenada `x` e ignorar
os valores nos campos `y` e `z`.

<Listing number="19-23" caption="Ignorando todos os campos de um `Point` exceto `x` usando `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

Listamos o valor `x` e depois apenas incluímos o padrão `..`. Isso é mais rápido
do que ter que listar `y: _` e `z: _`, particularmente quando estamos trabalhando com
structs que têm muitos campos em situações onde apenas um ou dois campos são
relevantes.

A sintaxe `..` se expandirá para quantos valores precisar ser. A Listagem 19-24
mostra como usar `..` com uma tupla.

<Listing number="19-24" file-name="src/main.rs" caption="Casando apenas o primeiro e o último valor em uma tupla e ignorando todos os outros valores">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

Neste código, o primeiro e o último valor são casados com `first` e `last`.
O `..` casará e ignorará tudo no meio.

No entanto, usar `..` deve ser inequívoco. Se não estiver claro quais valores são
destinados ao casamento e quais devem ser ignorados, Rust nos dará um erro.
A Listagem 19-25 mostra um exemplo de uso de `..` de forma ambígua, então ele não
compilará.

<Listing number="19-25" file-name="src/main.rs" caption="Uma tentativa de usar `..` de uma maneira ambígua">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Quando compilamos este exemplo, obtemos este erro:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

É impossível para Rust determinar quantos valores na tupla ignorar
antes de casar um valor com `second` e então quantos valores adicionais
ignorar depois disso. Este código poderia significar que queremos ignorar `2`, vincular
`second` a `4` e, em seguida, ignorar `8`, `16` e `32`; ou que queremos ignorar
`2` e `4`, vincular `second` a `8` e, em seguida, ignorar `16` e `32`; e assim por diante.
O nome da variável `second` não significa nada especial para Rust, então recebemos um
erro de compilador porque usar `..` em dois lugares como este é ambíguo.

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

### Adicionando Condicionais com Match Guards

Um _match guard_ é uma condição `if` adicional, especificada após o padrão em
um braço de `match`, que também deve casar para que aquele braço seja escolhido. Match guards são
úteis para expressar ideias mais complexas do que um padrão sozinho permite. Note,
no entanto, que eles estão disponíveis apenas em expressões `match`, não em expressões `if let` ou
`while let`.

A condição pode usar variáveis criadas no padrão. A Listagem 19-26 mostra um
`match` onde o primeiro braço tem o padrão `Some(x)` e também tem um match
guard de `if x % 2 == 0` (que será `true` se o número for par).

<Listing number="19-26" caption="Adicionando um match guard a um padrão">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Este exemplo imprimirá `O número 4 é par`. Quando `num` é comparado ao
padrão no primeiro braço, ele casa porque `Some(4)` casa com `Some(x)`. Então,
o match guard verifica se o resto da divisão de `x` por 2 é igual a
0, e como é, o primeiro braço é selecionado.

Se `num` tivesse sido `Some(5)`, o match guard no primeiro braço teria
sido `false` porque o resto de 5 dividido por 2 é 1, o que não é
igual a 0. Rust então iria para o segundo braço, que casaria porque o
segundo braço não tem um match guard e, portanto, casa com qualquer variante `Some`.

Não há como expressar a condição `if x % 2 == 0` dentro de um padrão, então
o match guard nos dá a capacidade de expressar essa lógica. A desvantagem desta
expressividade adicional é que o compilador não tenta verificar
exaustividade quando expressões de match guard estão envolvidas.

Ao discutir a Listagem 19-11, mencionamos que poderíamos usar match guards para
resolver nosso problema de sombreamento de padrão. Lembre-se de que criamos uma nova variável
dentro do padrão na expressão `match` em vez de usar a variável
fora do `match`. Essa nova variável significava que não podíamos testar contra o valor
da variável externa. A Listagem 19-27 mostra como podemos usar um match guard para consertar
esse problema.

<Listing number="19-27" file-name="src/main.rs" caption="Usando um match guard para testar a igualdade com uma variável externa">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Este código agora imprimirá `Caso padrão, x = Some(5)`. O padrão no segundo
braço de match não introduz uma nova variável `y` que sombrearia o `y` externo,
significando que podemos usar o `y` externo no match guard. Em vez de especificar o
padrão como `Some(y)`, o que teria sombreado o `y` externo, especificamos
`Some(n)`. Isso cria uma nova variável `n` que não sombreia nada porque
não há variável `n` fora do `match`.

O match guard `if n == y` não é um padrão e, portanto, não introduz novas
variáveis. Este `y` _é_ o `y` externo em vez de um novo `y` sombreando-o, e
podemos procurar um valor que tenha o mesmo valor que o `y` externo comparando
`n` a `y`.

Você também pode usar o operador _ou_ `|` em um match guard para especificar múltiplos
padrões; a condição do match guard se aplicará a todos os padrões. A Listagem
19-28 mostra a precedência ao combinar um padrão que usa `|` com um match
guard. A parte importante deste exemplo é que o match guard `if y` se
aplica a `4`, `5` _e_ `6`, mesmo que pareça que `if y` se aplica apenas
a `6`.

<Listing number="19-28" caption="Combinando múltiplos padrões com um match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

A condição de match afirma que o braço só casa se o valor de `x` for
igual a `4`, `5` ou `6` _e_ se `y` for `true`. Quando este código é executado, o
padrão do primeiro braço casa porque `x` é `4`, mas o match guard `if y`
é `false`, então o primeiro braço não é escolhido. O código passa para o segundo
braço, que casa, e este programa imprime `não`. A razão é que a
condição `if` se aplica a todo o padrão `4 | 5 | 6`, não apenas ao último
valor `6`. Em outras palavras, a precedência de um match guard em relação a um
padrão se comporta assim:

```text
(4 | 5 | 6) if y => ...
```

em vez de assim:

```text
4 | 5 | (6 if y) => ...
```

Após executar o código, o comportamento de precedência é evidente: Se o match guard
fosse aplicado apenas ao valor final na lista de valores especificados usando o
operador `|`, o braço teria casado, e o programa teria impresso
`sim`.

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

### Usando Vínculos `@`

O operador _at_ `@` nos permite criar uma variável que contém um valor ao mesmo
tempo em que estamos testando esse valor para um casamento de padrão. Na Listagem 19-29, queremos
testar se um campo `id` de `Message::Hello` está dentro da faixa `3..=7`. Também
queremos vincular o valor à variável `id` para que possamos usá-lo no código
associado ao braço.

<Listing number="19-29" caption="Usando `@` para vincular a um valor em um padrão enquanto também o testa">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Este exemplo imprimirá `Encontrou um id na faixa: 5`. Ao especificar `id @` antes
da faixa `3..=7`, estamos capturando qualquer valor que casou com a faixa em uma
variável chamada `id` enquanto também testamos se o valor casou com o padrão de faixa.

No segundo braço, onde temos apenas uma faixa especificada no padrão, o código
associado ao braço não tem uma variável que contenha o valor real
do campo `id`. O valor do campo `id` poderia ter sido 10, 11 ou 12, mas
o código que acompanha aquele padrão não sabe qual é. O código do padrão
não é capaz de usar o valor do campo `id` porque não salvamos o
valor `id` em uma variável.

No último braço, onde especificamos uma variável sem uma faixa, temos
o valor disponível para usar no código do braço em uma variável chamada `id`. A
razão é que usamos a sintaxe abreviada de campo de struct. Mas não
aplicamos nenhum teste ao valor no campo `id` neste braço, como fizemos com os
primeiros dois braços: Qualquer valor casaria com este padrão.

Usar `@` nos permite testar um valor e salvá-lo em uma variável dentro de um padrão.

## Resumo

Os padrões do Rust são muito úteis para distinguir entre diferentes tipos de
dados. Quando usados em expressões `match`, Rust garante que seus padrões cubram
todos os valores possíveis, ou seu programa não compilará. Padrões em instruções `let`
e parâmetros de função tornam essas construções mais úteis, permitindo
a desestruturação de valores em partes menores e atribuindo essas partes a
variáveis. Podemos criar padrões simples ou complexos para atender às nossas necessidades.

A seguir, para o penúltimo capítulo do livro, veremos alguns aspectos avançados
de uma variedade de recursos do Rust.
