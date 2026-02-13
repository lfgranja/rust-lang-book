## Usando `Box<T>` para Apontar para Dados na Heap

O ponteiro inteligente mais simples é uma *box* (caixa), cujo tipo é escrito como `Box<T>`. *Boxes* permitem que você armazene dados na heap em vez de na pilha. O que permanece na pilha é o ponteiro para os dados na heap. Consulte o Capítulo 4 para rever a diferença entre a pilha e a heap.

Boxes não têm custo de performance adicional, além de armazenar seus dados na heap em vez de na pilha. Mas elas também não têm muitas capacidades extras. Você as usará com mais frequência nestas situações:

- Quando você tem um tipo cujo tamanho não pode ser conhecido em tempo de compilação e você quer usar um valor desse tipo em um contexto que requer um tamanho exato
- Quando você tem uma grande quantidade de dados e quer transferir a posse, mas garantir que os dados não serão copiados quando você o fizer
- Quando você quer possuir um valor e se importa apenas que ele seja de um tipo que implementa uma trait específica, em vez de ser de um tipo específico

Demonstraremos a primeira situação em ["Habilitando Tipos Recursivos com Boxes"](#habilitando-tipos-recursivos-com-boxes). No segundo caso, transferir a posse de uma grande quantidade de dados pode levar muito tempo porque os dados são copiados na pilha. Para melhorar a performance nessa situação, podemos armazenar a grande quantidade de dados na heap em uma box. Então, apenas a pequena quantidade de dados do ponteiro é copiada na pilha, enquanto os dados que ele referencia permanecem em um lugar na heap. O terceiro caso é conhecido como *objeto de trait*, e o Capítulo 17 dedica uma seção inteira a esse tópico. Então, o que você aprender aqui você aplicará novamente no Capítulo 17!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-store-data-on-the-heap"></a>

### Armazenando Dados na Heap

Antes de discutirmos o caso de uso de armazenamento na heap para `Box<T>`, cobriremos a sintaxe e como interagir com valores armazenados dentro de uma `Box<T>`.

A Listagem 15-1 mostra como usar uma box para armazenar um valor `i32` na heap:

<Listing number="15-1" file-name="src/main.rs" caption="Armazenando um valor `i32` na heap usando uma box">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

Definimos a variável `b` para ter o valor de uma `Box` que aponta para o valor `5`, que é alocado na heap. Este programa imprimirá `b = 5`; neste caso, podemos acessar os dados na box de forma similar a como faríamos se esses dados estivessem na pilha. Assim como qualquer valor possuído, quando uma box sai de escopo, como `b` faz no final de `main`, ela será desalocada. A desalocação acontece tanto para a box (armazenada na pilha) quanto para os dados para os quais ela aponta (armazenados na heap).

Colocar um único valor na heap não é muito útil, então você não usará boxes sozinhas dessa maneira com muita frequência. Ter valores como um único `i32` na pilha, onde eles são armazenados por padrão, é mais apropriado na maioria das situações. Vamos ver um caso onde boxes nos permitem definir tipos que não poderíamos definir se não tivéssemos boxes.

### Habilitando Tipos Recursivos com Boxes

Um valor de um *tipo recursivo* pode ter outro valor do mesmo tipo como parte de si mesmo. Tipos recursivos impõem um problema porque Rust precisa saber em tempo de compilação quanto espaço um tipo ocupa. No entanto, o aninhamento de valores de tipos recursivos poderia teoricamente continuar infinitamente, então Rust não pode saber quanto espaço o valor precisa. Como boxes têm um tamanho conhecido, podemos habilitar tipos recursivos inserindo uma box na definição do tipo recursivo.

Como exemplo de um tipo recursivo, vamos explorar a *cons list* (lista construtora). Este é um tipo de dados comumente encontrado em linguagens de programação funcional. O tipo cons list que definiremos é simples, exceto pela recursão; portanto, os conceitos no exemplo com o qual trabalharemos serão úteis sempre que você se deparar com situações mais complexas envolvendo tipos recursivos.

<!-- Old headings. Do not remove or links may break. -->

<a id="more-information-about-the-cons-list"></a>

#### Entendendo a Cons List

Uma *cons list* é uma estrutura de dados que vem da linguagem de programação Lisp e seus dialetos e é feita de pares aninhados, e é a versão Lisp de uma lista encadeada. Seu nome vem da função `cons` (abreviação de "construct function", função de construção) em Lisp que constrói um novo par a partir de seus dois argumentos. Ao chamar `cons` em um par consistindo de um valor e outro par, podemos construir cons lists feitas de pares recursivos.

Por exemplo, aqui está uma representação em pseudocódigo de uma cons list contendo a lista `1, 2, 3` com cada par entre parênteses:

```text
(1, (2, (3, Nil)))
```

Cada item em uma cons list contém dois elementos: o valor do item atual e o próximo item. O último item na lista contém apenas um valor chamado `Nil` sem um próximo item. Uma cons list é produzida chamando recursivamente a função `cons`. O nome canônico para denotar o caso base da recursão é `Nil`. Note que isso não é o mesmo que o conceito de "null" ou "nil" discutido no Capítulo 6, que é um valor inválido ou ausente.

A cons list não é uma estrutura de dados comumente usada em Rust. Na maioria das vezes, quando você tem uma lista de itens em Rust, `Vec<T>` é uma escolha melhor. Outros tipos de dados recursivos mais complexos *são* úteis em várias situações, mas começando com a cons list neste capítulo, podemos explorar como boxes nos permitem definir um tipo de dados recursivo sem muita distração.

A Listagem 15-2 contém uma definição de enum para uma cons list. Note que este código não compilará ainda, porque o tipo `List` não tem um tamanho conhecido, o que demonstraremos.

<Listing number="15-2" file-name="src/main.rs" caption="A primeira tentativa de definir um enum para representar uma estrutura de dados cons list de valores `i32`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> Nota: Estamos implementando uma cons list que contém apenas valores `i32` para os propósitos deste exemplo. Poderíamos tê-la implementado usando genéricos, como discutimos no Capítulo 10, para definir um tipo cons list que poderia armazenar valores de qualquer tipo.

Usar o tipo `List` para armazenar a lista `1, 2, 3` se pareceria com o código na Listagem 15-3:

<Listing number="15-3" file-name="src/main.rs" caption="Usando o enum `List` para armazenar a lista `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

O primeiro valor `Cons` contém `1` e outro valor `List`. Este valor `List` é outro valor `Cons` que contém `2` e outro valor `List`. Este valor `List` é mais um valor `Cons` que contém `3` e um valor `List`, que é finalmente `Nil`, a variante não recursiva que sinaliza o fim da lista.

Se tentarmos compilar o código na Listagem 15-3, recebemos o erro mostrado na Listagem 15-4:

<Listing number="15-4" caption="O erro que recebemos ao tentar definir um enum recursivo">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

O erro mostra que este tipo "tem tamanho infinito" (has infinite size). A razão é que definimos `List` com uma variante que é recursiva: ela contém outro valor de si mesma diretamente. Como resultado, Rust não consegue descobrir quanto espaço precisa para armazenar um valor `List`. Vamos analisar por que recebemos este erro. Primeiro, veremos como Rust decide quanto espaço precisa para armazenar um valor de um tipo não recursivo.

#### Calculando o Tamanho de um Tipo Não Recursivo

Relembre o enum `Message` que definimos na Listagem 6-2 quando discutimos definições de enum no Capítulo 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Para determinar quanto espaço alocar para um valor `Message`, Rust percorre cada uma das variantes para ver qual variante precisa de mais espaço. Rust vê que `Message::Quit` não precisa de nenhum espaço, `Message::Move` precisa de espaço suficiente para armazenar dois valores `i32`, e assim por diante. Como apenas uma variante será usada, o maior espaço que um valor `Message` precisará é o espaço que levaria para armazenar a maior de suas variantes.

Compare isso com o que acontece quando Rust tenta determinar quanto espaço um tipo recursivo como o enum `List` na Listagem 15-2 precisa. O compilador começa olhando para a variante `Cons`, que contém um valor do tipo `i32` e um valor do tipo `List`. Portanto, `Cons` precisa de uma quantidade de espaço igual ao tamanho de um `i32` mais o tamanho de um `List`. Para descobrir quanta memória o tipo `List` precisa, o compilador olha para as variantes, começando com a variante `Cons`. A variante `Cons` contém um valor do tipo `i32` e um valor do tipo `List`, e este processo continua infinitamente, como mostrado na Figura 15-1.

<img alt="Uma cons list infinita: um retângulo rotulado 'Cons' dividido em dois retângulos menores. O primeiro retângulo menor contém o rótulo 'i32', e o segundo retângulo menor contém o rótulo 'Cons' e uma versão menor do retângulo 'Cons' externo. Os retângulos 'Cons' continuam a conter versões cada vez menores de si mesmos até que o menor retângulo de tamanho confortável contenha um símbolo de infinito, indicando que esta repetição continua para sempre." src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 15-1: Uma `List` infinita consistindo de variantes `Cons` infinitas</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-get-a-recursive-type-with-a-known-size"></a>

#### Obtendo um Tipo Recursivo com um Tamanho Conhecido

Como Rust não consegue descobrir quanto espaço alocar para tipos definidos recursivamente, o compilador dá um erro com esta sugestão útil:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

Nesta sugestão, "indireção" (indirection) significa que, em vez de armazenar um valor diretamente, devemos mudar a estrutura de dados para armazenar o valor indiretamente, armazenando um ponteiro para o valor.

Como uma `Box<T>` é um ponteiro, Rust sempre sabe quanto espaço uma `Box<T>` precisa: o tamanho de um ponteiro não muda com base na quantidade de dados para os quais ele aponta. Isso significa que podemos colocar uma `Box<T>` dentro da variante `Cons` em vez de outro valor `List` diretamente. A `Box<T>` apontará para o próximo valor `List` que estará na heap em vez de dentro da variante `Cons`. Conceitualmente, ainda temos uma lista, criada com listas contendo outras listas, mas esta implementação agora é mais como colocar os itens um ao lado do outro em vez de um dentro do outro.

Podemos mudar a definição do enum `List` na Listagem 15-2 e o uso de `List` na Listagem 15-3 para o código na Listagem 15-5, que compilará:

<Listing number="15-5" file-name="src/main.rs" caption="A definição de `List` que usa `Box<T>` para ter um tamanho conhecido">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

A variante `Cons` precisa do tamanho de um `i32` mais o espaço para armazenar os dados do ponteiro da box. A variante `Nil` não armazena valores, então precisa de menos espaço na pilha do que a variante `Cons`. Agora sabemos que qualquer valor `List` ocupará o tamanho de um `i32` mais o tamanho dos dados do ponteiro de uma box. Usando uma box, quebramos a cadeia infinita e recursiva, então o compilador pode descobrir o tamanho que precisa para armazenar um valor `List`. A Figura 15-2 mostra como a variante `Cons` se parece agora.

<img alt="Um retângulo rotulado 'Cons' dividido em dois retângulos menores. O primeiro retângulo menor contém o rótulo 'i32', e o segundo retângulo menor contém o rótulo 'Box' com um retângulo interno que contém o rótulo 'usize', representando o tamanho finito do ponteiro da box." src="img/trpl15-02.svg" class="center" />

<span class="caption">Figura 15-2: Uma `List` que não é infinitamente dimensionada, porque `Cons` contém uma `Box`</span>

Boxes fornecem apenas a indireção e a alocação na heap; elas não têm nenhuma outra capacidade especial, como aquelas que veremos com os outros tipos de ponteiros inteligentes. Elas também não têm o custo de performance que essas capacidades especiais incorrem, então podem ser úteis em casos como a cons list, onde a indireção é a única funcionalidade que precisamos. Veremos mais casos de uso para boxes no Capítulo 17.

O tipo `Box<T>` é um ponteiro inteligente porque implementa a trait `Deref`, que permite que valores `Box<T>` sejam tratados como referências. Quando um valor `Box<T>` sai de escopo, os dados da heap para os quais a box aponta também são limpos devido à implementação da trait `Drop`. Essas duas traits serão ainda mais importantes para a funcionalidade fornecida pelos outros tipos de ponteiros inteligentes que discutiremos no restante deste capítulo. Vamos explorar essas duas traits em mais detalhes.

[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
