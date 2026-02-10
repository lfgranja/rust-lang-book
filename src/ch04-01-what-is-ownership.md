## O Que É Ownership?

*Ownership* é um conjunto de regras que governa como um programa Rust gerencia a memória. Todos os programas têm que gerenciar a forma como usam a memória de um computador enquanto estão rodando. Algumas linguagens têm *garbage collection* que procura regularmente por memória não mais utilizada enquanto o programa roda; em outras linguagens, o programador deve alocar e liberar a memória explicitamente. O Rust usa uma terceira abordagem: a memória é gerenciada através de um sistema de *ownership* com um conjunto de regras que o compilador verifica. Se qualquer uma das regras for violada, o programa não compilará. Nenhuma das características do *ownership* deixará seu programa mais lento enquanto ele estiver rodando.

Como o *ownership* é um conceito novo para muitos programadores, leva um tempo para se acostumar. A boa notícia é que quanto mais experiente você se tornar com o Rust e as regras do sistema de *ownership*, mais fácil você achará desenvolver naturalmente código que seja seguro e eficiente. Continue firme!

Quando você entender *ownership*, você terá uma base sólida para entender as características que tornam o Rust único. Neste capítulo, você aprenderá *ownership* trabalhando através de alguns exemplos que focam em uma estrutura de dados muito comum: strings.

> ### A Stack e a Heap
>
> Muitas linguagens de programação não exigem que você pense sobre a *stack* (pilha) e a *heap* (monte) com muita frequência. Mas em uma linguagem de programação de sistemas como o Rust, se um valor está na *stack* ou na *heap* afeta como a linguagem se comporta e por que você tem que tomar certas decisões. Partes do *ownership* serão descritas em relação à *stack* e à *heap* mais tarde neste capítulo, então aqui vai uma breve explicação como preparação.
>
> Tanto a *stack* quanto a *heap* são partes da memória disponíveis para seu código usar em tempo de execução, mas elas são estruturadas de formas diferentes. A *stack* armazena valores na ordem em que os recebe e remove os valores na ordem oposta. Isso é referido como *last in, first out* (último a entrar, primeiro a sair - LIFO). Pense em uma pilha de pratos: quando você adiciona mais pratos, você os coloca no topo da pilha, e quando você precisa de um prato, você tira um do topo. Adicionar ou remover pratos do meio ou do fundo não funcionaria tão bem! Adicionar dados é chamado de *pushing onto the stack* (empurrar para a *stack*), e remover dados é chamado de *popping off the stack* (retirar da *stack*). Todos os dados armazenados na *stack* devem ter um tamanho conhecido e fixo. Dados com um tamanho desconhecido em tempo de compilação ou um tamanho que pode mudar devem ser armazenados na *heap*.
>
> A *heap* é menos organizada: quando você coloca dados na *heap*, você requisita uma certa quantidade de espaço. O alocador de memória encontra um espaço vazio na *heap* que seja grande o suficiente, marca-o como estando em uso, e retorna um *ponteiro*, que é o endereço daquela localização. Esse processo é chamado de *allocating on the heap* (alocar na *heap*) e às vezes é abreviado apenas como *alocação* (empurrar valores para a *stack* não é considerado alocação). Como o ponteiro para a *heap* é de um tamanho conhecido e fixo, você pode armazenar o ponteiro na *stack*, mas quando você quer os dados reais, você deve seguir o ponteiro. Pense em sentar-se em um restaurante. Quando você entra, você informa o número de pessoas no seu grupo, e o anfitrião encontra uma mesa vazia que caiba todos e leva vocês até lá. Se alguém no seu grupo chegar atrasado, eles podem perguntar onde vocês foram sentados para encontrá-los.
>
> Empurrar para a *stack* é mais rápido do que alocar na *heap* porque o alocador nunca tem que procurar um lugar para armazenar novos dados; essa localização está sempre no topo da *stack*. Comparativamente, alocar espaço na *heap* requer mais trabalho porque o alocador deve primeiro encontrar um espaço grande o suficiente para conter os dados e então realizar a contabilidade para se preparar para a próxima alocação.
>
> Acessar dados na *heap* é geralmente mais lento do que acessar dados na *stack* porque você tem que seguir um ponteiro para chegar lá. Processadores contemporâneos são mais rápidos se pularem menos na memória. Continuando a analogia, considere um garçom em um restaurante anotando pedidos de muitas mesas. É mais eficiente pegar todos os pedidos de uma mesa antes de ir para a próxima mesa. Pegar um pedido da mesa A, depois um pedido da mesa B, depois um da A de novo, e depois um da B de novo seria um processo muito mais lento. Pelo mesmo motivo, um processador pode geralmente fazer seu trabalho melhor se ele trabalhar em dados que estão próximos de outros dados (como estão na *stack*) em vez de mais distantes (como podem estar na *heap*).
>
> Quando seu código chama uma função, os valores passados para a função (incluindo, potencialmente, ponteiros para dados na *heap*) e as variáveis locais da função são empurrados para a *stack*. Quando a função acaba, esses valores são retirados da *stack*.
>
> Manter o controle de quais partes do código estão usando quais dados na *heap*, minimizar a quantidade de dados duplicados na *heap*, e limpar dados não utilizados na *heap* para que você não fique sem espaço são todos problemas que o *ownership* aborda. Uma vez que você entenda *ownership*, você não precisará pensar sobre a *stack* e a *heap* com muita frequência. Mas saber que o objetivo principal do *ownership* é gerenciar dados da *heap* pode ajudar a explicar por que ele funciona da maneira que funciona.

### Regras de Ownership

Primeiro, vamos dar uma olhada nas regras de *ownership*. Mantenha essas regras em mente enquanto trabalhamos nos exemplos que as ilustram:

- Cada valor em Rust tem um *owner* (dono).
- Só pode haver um *owner* por vez.
- Quando o *owner* sai de escopo, o valor será descartado (*dropped*).

### Escopo de Variável

Agora que passamos da sintaxe básica do Rust, não incluiremos todo o código `fn main() {` nos exemplos, então se você estiver acompanhando, certifique-se de colocar os exemplos a seguir dentro de uma função `main` manualmente. Como resultado, nossos exemplos serão um pouco mais concisos, permitindo-nos focar nos detalhes reais em vez de código repetitivo.

Como um primeiro exemplo de *ownership*, olharemos para o escopo de algumas variáveis. Um *escopo* é o intervalo dentro de um programa para o qual um item é válido. Pegue a seguinte variável:

```rust
let s = "olá";
```

A variável `s` refere-se a uma string literal, onde o valor da string é codificado diretamente no texto do nosso programa. A variável é válida do ponto em que é declarada até o fim do escopo atual. A Listagem 4-1 mostra um programa com comentários anotando onde a variável `s` seria válida.

<Listing number="4-1" caption="Uma variável e o escopo no qual ela é válida">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</Listing>

Em outras palavras, existem dois pontos no tempo importantes aqui:

- Quando `s` entra *no* escopo, ela é válida.
- Ela permanece válida até que saia *do* escopo.

Neste ponto, a relação entre escopos e quando variáveis são válidas é similar à de outras linguagens de programação. Agora vamos construir sobre esse entendimento introduzindo o tipo `String`.

### O Tipo `String`

Para ilustrar as regras de *ownership*, precisamos de um tipo de dados que seja mais complexo do que aqueles que cobrimos na seção [“Tipos de Dados”][data-types]<!-- ignore --> do Capítulo 3. Os tipos cobertos anteriormente são de um tamanho conhecido, podem ser armazenados na *stack* e retirados da *stack* quando seu escopo termina, e podem ser copiados rápida e trivialmente para fazer uma nova instância independente se outra parte do código precisar usar o mesmo valor em um escopo diferente. Mas queremos olhar para dados que são armazenados na *heap* e explorar como o Rust sabe quando limpar esses dados, e o tipo `String` é um ótimo exemplo.

Vamos nos concentrar nas partes de `String` que se relacionam com *ownership*. Esses aspectos também se aplicam a outros tipos de dados complexos, sejam eles fornecidos pela biblioteca padrão ou criados por você. Discutiremos aspectos de não-*ownership* de `String` no [Capítulo 8][ch8]<!-- ignore -->.

Já vimos strings literais, onde um valor de string é codificado diretamente em nosso programa. Strings literais são convenientes, mas não são adequadas para toda situação em que podemos querer usar texto. Uma razão é que elas são imutáveis. Outra é que nem todo valor de string pode ser conhecido quando escrevemos nosso código: por exemplo, e se quisermos receber entrada do usuário e armazená-la? É para essas situações que o Rust tem o tipo `String`. Esse tipo gerencia dados alocados na *heap* e, como tal, é capaz de armazenar uma quantidade de texto que é desconhecida para nós em tempo de compilação. Você pode criar uma `String` a partir de uma string literal usando a função `from`, assim:

```rust
let s = String::from("olá");
```

O operador de dois pontos duplos `::` nos permite criar um *namespace* para essa função `from` particular sob o tipo `String` em vez de usar algum tipo de nome como `string_from`. Discutiremos essa sintaxe mais na seção [“Métodos”][methods]<!-- ignore --> do Capítulo 5, e quando falarmos sobre *namespacing* com módulos em [“Caminhos para Referenciar um Item na Árvore de Módulos”][paths-module-tree]<!-- ignore --> no Capítulo 7.

Esse tipo de string *pode* ser mutado:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

Então, qual é a diferença aqui? Por que `String` pode ser mutada mas literais não podem? A diferença está em como esses dois tipos lidam com a memória.

### Memória e Alocação

No caso de uma string literal, sabemos o conteúdo em tempo de compilação, então o texto é codificado diretamente no executável final. É por isso que strings literais são rápidas e eficientes. Mas essas propriedades vêm apenas da imutabilidade da string literal. Infelizmente, não podemos colocar um pedaço de memória no binário para cada pedaço de texto cujo tamanho é desconhecido em tempo de compilação e cujo tamanho pode mudar enquanto o programa roda.

Com o tipo `String`, para suportar um pedaço de texto mutável e que pode crescer, precisamos alocar uma quantidade de memória na *heap*, desconhecida em tempo de compilação, para conter o conteúdo. Isso significa:

- A memória deve ser requisitada ao alocador de memória em tempo de execução.
- Precisamos de uma maneira de retornar essa memória ao alocador quando terminarmos com nossa `String`.

A primeira parte é feita por nós: quando chamamos `String::from`, sua implementação requisita a memória de que precisa. Isso é praticamente universal em linguagens de programação.

No entanto, a segunda parte é diferente. Em linguagens com um *garbage collector (GC)*, o GC mantém o controle e limpa a memória que não está sendo mais usada, e nós não precisamos pensar sobre isso. Na maioria das linguagens sem um GC, é nossa responsabilidade identificar quando a memória não está mais sendo usada e chamar código para liberá-la explicitamente, assim como fizemos para requisitá-la. Fazer isso corretamente tem sido historicamente um problema difícil de programação. Se esquecermos, desperdiçaremos memória. Se fizermos muito cedo, teremos uma variável inválida. Se fizermos duas vezes, isso também é um bug. Precisamos emparelhar exatamente um `allocate` (alocar) com exatamente um `free` (liberar).

O Rust toma um caminho diferente: a memória é retornada automaticamente uma vez que a variável que a possui sai de escopo. Aqui está uma versão do nosso exemplo de escopo da Listagem 4-1 usando uma `String` em vez de uma string literal:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

Existe um ponto natural em que podemos retornar a memória que nossa `String` precisa ao alocador: quando `s` sai de escopo. Quando uma variável sai de escopo, o Rust chama uma função especial para nós. Essa função é chamada `drop`, e é onde o autor de `String` pode colocar o código para retornar a memória. O Rust chama `drop` automaticamente no fechamento da chave.

> Nota: Em C++, esse padrão de desalocar recursos no final do tempo de vida de um item é às vezes chamado de *Resource Acquisition Is Initialization (RAII)*. A função `drop` no Rust será familiar para você se você já usou padrões RAII.

Esse padrão tem um impacto profundo na maneira como o código Rust é escrito. Pode parecer simples agora, mas o comportamento do código pode ser inesperado em situações mais complicadas quando queremos ter múltiplas variáveis usando os dados que alocamos na *heap*. Vamos explorar algumas dessas situações agora.

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-move"></a>

#### Variáveis e Dados Interagindo com Move

Múltiplas variáveis podem interagir com os mesmos dados de maneiras diferentes em Rust. A Listagem 4-2 mostra um exemplo usando um inteiro.

<Listing number="4-2" caption="Atribuindo o valor inteiro da variável `x` a `y`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</Listing>

Podemos provavelmente adivinhar o que isso está fazendo: "Vincule o valor `5` a `x`; então, faça uma cópia do valor em `x` e vincule-o a `y`." Agora temos duas variáveis, `x` e `y`, e ambas são iguais a `5`. Isso é de fato o que está acontecendo, porque inteiros são valores simples com um tamanho conhecido e fixo, e esses dois valores `5` são empurrados para a *stack*.

Agora vamos olhar para a versão com `String`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

Isso parece muito similar, então poderíamos assumir que a maneira como funciona seria a mesma: isto é, a segunda linha faria uma cópia do valor em `s1` e o vincularia a `s2`. Mas não é bem isso que acontece.

Dê uma olhada na Figura 4-1 para ver o que está acontecendo com a `String` por baixo dos panos. Uma `String` é feita de três partes, mostradas à esquerda: um ponteiro para a memória que contém o conteúdo da string, um comprimento e uma capacidade. Esse grupo de dados é armazenado na *stack*. À direita está a memória na *heap* que contém o conteúdo.

<img alt="Two tables: the first table contains the representation of s1 on the
stack, consisting of its length (5), capacity (5), and a pointer to the first
value in the second table. The second table contains the representation of the
string data on the heap, byte by byte." src="img/trpl04-01.svg" class="center"
style="width: 50%;" />

<span class="caption">Figura 4-1: A representação em memória de uma `String` contendo o valor `"hello"` vinculado a `s1`</span>

O comprimento é quanta memória, em bytes, o conteúdo da `String` está usando atualmente. A capacidade é a quantidade total de memória, em bytes, que a `String` recebeu do alocador. A diferença entre comprimento e capacidade importa, mas não neste contexto, então por enquanto, tudo bem ignorar a capacidade.

Quando atribuímos `s1` a `s2`, os dados da `String` são copiados, o que significa que copiamos o ponteiro, o comprimento e a capacidade que estão na *stack*. Não copiamos os dados na *heap* aos quais o ponteiro se refere. Em outras palavras, a representação de dados na memória se parece com a Figura 4-2.

<img alt="Three tables: tables s1 and s2 representing those strings on the
stack, respectively, and both pointing to the same string data on the heap."
src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-2: A representação em memória da variável `s2` que tem uma cópia do ponteiro, comprimento e capacidade de `s1`</span>

A representação *não* se parece com a Figura 4-3, que é como a memória se pareceria se o Rust copiasse os dados da *heap* também. Se o Rust fizesse isso, a operação `s2 = s1` poderia ser muito cara em termos de desempenho em tempo de execução se os dados na *heap* fossem grandes.

<img alt="Four tables: two tables representing the stack data for s1 and s2,
and each points to its own copy of string data on the heap."
src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-3: Outra possibilidade para o que `s2 = s1` poderia fazer se o Rust copiasse os dados da *heap* também</span>

Anteriormente, dissemos que quando uma variável sai de escopo, o Rust chama automaticamente a função `drop` e limpa a memória da *heap* para aquela variável. Mas a Figura 4-2 mostra ambos os ponteiros de dados apontando para a mesma localização. Isso é um problema: quando `s2` e `s1` saem de escopo, ambos tentarão liberar a mesma memória. Isso é conhecido como um erro de *double free* (liberação dupla) e é um dos bugs de segurança de memória que mencionamos anteriormente. Liberar memória duas vezes pode levar à corrupção de memória, o que pode potencialmente levar a vulnerabilidades de segurança.

Para garantir a segurança de memória, depois da linha `let s2 = s1;`, o Rust considera `s1` como não sendo mais válido. Portanto, o Rust não precisa liberar nada quando `s1` sai de escopo. Confira o que acontece quando você tenta usar `s1` depois que `s2` é criada; não funcionará:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Você receberá um erro como este porque o Rust impede você de usar a referência invalidada:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

Se você já ouviu os termos *shallow copy* (cópia rasa) e *deep copy* (cópia profunda) enquanto trabalhava com outras linguagens, o conceito de copiar o ponteiro, comprimento e capacidade sem copiar os dados provavelmente soa como fazer uma *shallow copy*. Mas como o Rust também invalida a primeira variável, em vez de ser chamado de *shallow copy*, isso é conhecido como um *move* (movimento). Neste exemplo, diríamos que `s1` foi *movida* para `s2`. Então, o que realmente acontece é mostrado na Figura 4-4.

<img alt="Three tables: tables s1 and s2 representing those strings on the
stack, respectively, and both pointing to the same string data on the heap.
Table s1 is grayed out because s1 is no longer valid; only s2 can be used to
access the heap data." src="img/trpl04-04.svg" class="center" style="width:
50%;" />

<span class="caption">Figura 4-4: A representação em memória depois que `s1` foi invalidada</span>

Isso resolve nosso problema! Com apenas `s2` válida, quando ela sair de escopo, ela sozinha liberará a memória, e pronto.

Além disso, há uma escolha de design que está implícita nisso: o Rust nunca criará automaticamente cópias "profundas" dos seus dados. Portanto, qualquer cópia *automática* pode ser assumida como sendo barata em termos de desempenho em tempo de execução.

#### Escopo e Atribuição

O inverso disso é verdadeiro para a relação entre escopo, *ownership* e memória sendo liberada via a função `drop` também. Quando você atribui um valor completamente novo a uma variável existente, o Rust chamará `drop` e liberará a memória do valor original imediatamente. Considere este código, por exemplo:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

Nós inicialmente declaramos uma variável `s` e a vinculamos a uma `String` com o valor `"hello"`. Então, imediatamente criamos uma nova `String` com o valor `"ahoy"` e a atribuímos a `s`. Neste ponto, nada está se referindo ao valor original na *heap*. A Figura 4-5 ilustra os dados da *stack* e da *heap* agora:

<img alt="One table representing the string value on the stack, pointing to
the second piece of string data (ahoy) on the heap, with the original string
data (hello) grayed out because it cannot be accessed anymore."
src="img/trpl04-05.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-5: A representação em memória depois que o valor inicial foi substituído em sua totalidade</span>

A string original, assim, sai imediatamente de escopo. O Rust rodará a função `drop` nela e sua memória será liberada imediatamente. Quando imprimimos o valor no final, será `"ahoy, world!"`.

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-clone"></a>

#### Variáveis e Dados Interagindo com Clone

Se nós *quisermos* copiar profundamente os dados da *heap* da `String`, não apenas os dados da *stack*, podemos usar um método comum chamado `clone`. Discutiremos sintaxe de método no Capítulo 5, mas como métodos são uma característica comum em muitas linguagens de programação, você provavelmente já os viu antes.

Aqui está um exemplo do método `clone` em ação:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

Isso funciona muito bem e produz explicitamente o comportamento mostrado na Figura 4-3, onde os dados da *heap* *são* copiados.

Quando você vê uma chamada para `clone`, você sabe que algum código arbitrário está sendo executado e que esse código pode ser caro. É um indicador visual de que algo diferente está acontecendo.

#### Dados Somente da Stack: Copy

Há outra ruga sobre a qual não falamos ainda. Este código usando inteiros — parte do qual foi mostrado na Listagem 4-2 — funciona e é válido:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

Mas esse código parece contradizer o que acabamos de aprender: não temos uma chamada para `clone`, mas `x` ainda é válido e não foi movido para `y`.

A razão é que tipos como inteiros que têm um tamanho conhecido em tempo de compilação são armazenados inteiramente na *stack*, então cópias dos valores reais são rápidas de fazer. Isso significa que não há razão para querermos impedir `x` de ser válido depois de criarmos a variável `y`. Em outras palavras, não há diferença entre cópia profunda e rasa aqui, então chamar `clone` não faria nada diferente da cópia rasa usual, e podemos deixá-lo de fora.

O Rust tem uma anotação especial chamada a *trait* `Copy` que podemos colocar em tipos que são armazenados na *stack*, como inteiros são (falaremos mais sobre traits no [Capítulo 10][traits]<!-- ignore -->). Se um tipo implementa a trait `Copy`, variáveis que a usam não se movem, mas sim são trivialmente copiadas, tornando-as ainda válidas após atribuição a outra variável.

O Rust não nos deixará anotar um tipo com `Copy` se o tipo, ou qualquer uma de suas partes, tiver implementado a trait `Drop`. Se o tipo precisa que algo especial aconteça quando o valor sai de escopo e adicionamos a anotação `Copy` a esse tipo, teremos um erro em tempo de compilação. Para aprender sobre como adicionar a anotação `Copy` ao seu tipo para implementar a trait, veja [“Traits Deriváveis”][derivable-traits]<!-- ignore --> no Apêndice C.

Então, quais tipos implementam a trait `Copy`? Você pode checar a documentação para o tipo dado para ter certeza, mas como regra geral, qualquer grupo de valores escalares simples pode implementar `Copy`, e nada que exija alocação ou seja alguma forma de recurso pode implementar `Copy`. Aqui estão alguns dos tipos que implementam `Copy`:

- Todos os tipos inteiros, como `u32`.
- O tipo Booleano, `bool`, com valores `true` e `false`.
- Todos os tipos de ponto flutuante, como `f64`.
- O tipo caractere, `char`.
- Tuplas, se elas contiverem apenas tipos que também implementam `Copy`. Por exemplo, `(i32, i32)` implementa `Copy`, mas `(i32, String)` não.

### Ownership e Funções

A mecânica de passar um valor para uma função é similar à de atribuir um valor a uma variável. Passar uma variável para uma função irá mover ou copiar, assim como a atribuição faz. A Listagem 4-3 tem um exemplo com algumas anotações mostrando onde variáveis entram e saem de escopo.

<Listing number="4-3" file-name="src/main.rs" caption="Funções com ownership e escopo anotados">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</Listing>

Se tentássemos usar `s` após a chamada para `takes_ownership`, o Rust lançaria um erro em tempo de compilação. Essas verificações estáticas nos protegem de erros. Tente adicionar código a `main` que usa `s` e `x` para ver onde você pode usá-los e onde as regras de *ownership* impedem você de fazê-lo.

### Valores de Retorno e Escopo

Retornar valores também pode transferir *ownership*. A Listagem 4-4 mostra um exemplo de uma função que retorna algum valor, com anotações similares às da Listagem 4-3.

<Listing number="4-4" file-name="src/main.rs" caption="Transferindo ownership de valores de retorno">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</Listing>

O *ownership* de uma variável segue o mesmo padrão toda vez: atribuir um valor a outra variável o move. Quando uma variável que inclui dados na *heap* sai de escopo, o valor será limpo pelo `drop` a menos que o *ownership* dos dados tenha sido movido para outra variável.

Embora isso funcione, tomar *ownership* e então retornar *ownership* com cada função é um pouco tedioso. E se quisermos deixar uma função usar um valor mas não tomar *ownership*? É bastante irritante que qualquer coisa que passemos também precise ser passada de volta se quisermos usá-la novamente, além de quaisquer dados resultantes do corpo da função que possamos querer retornar também.

O Rust nos permite retornar múltiplos valores usando uma tupla, como mostrado na Listagem 4-5.

<Listing number="4-5" file-name="src/main.rs" caption="Retornando ownership de parâmetros">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</Listing>

Mas isso é muita cerimônia e muito trabalho para um conceito que deveria ser comum. Felizmente para nós, o Rust tem uma funcionalidade para usar um valor sem transferir *ownership*: referências.

[data-types]: [[ch03-02-data-types.md#data-types|Tipos de Dados]]
[ch8]: [[ch08-02-strings.md|Capítulo 8]]
[traits]: [[ch10-02-traits.md|Capítulo 10]]
[derivable-traits]: [[appendix-03-derivable-traits.md|Traits Deriváveis]]
[methods]: [[ch05-03-method-syntax.md#methods|Métodos]]
[paths-module-tree]: [[ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md|Caminhos para Referenciar um Item na Árvore de Módulos]]
