<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## Usando Objetos de Trait para Abstrair Comportamentos Comuns

No Capítulo 8, mencionamos que uma limitação dos vetores é que eles podem
armazenar elementos de apenas um tipo. Criamos uma solução alternativa na Listagem 8-9 onde
definimos um enum `SpreadsheetCell` que tinha variantes para conter inteiros, floats,
e texto. Isso significava que podíamos armazenar diferentes tipos de dados em cada célula e
ainda ter um vetor que representava uma linha de células. Esta é uma solução perfeitamente boa
quando nossos itens intercambiáveis são um conjunto fixo de tipos que conhecemos
quando nosso código é compilado.

No entanto, às vezes queremos que nosso usuário da biblioteca seja capaz de estender o conjunto de
tipos que são válidos em uma situação particular. Para mostrar como podemos alcançar
isso, criaremos um exemplo de ferramenta de interface gráfica do usuário (GUI) que itera
através de uma lista de itens, chamando um método `draw` em cada um para desenhá-lo na
tela—uma técnica comum para ferramentas GUI. Criaremos um crate de biblioteca chamado
`gui` que contém a estrutura de uma biblioteca GUI. Este crate pode incluir
alguns tipos para as pessoas usarem, como `Button` ou `TextField`. Além disso,
usuários do `gui` vão querer criar seus próprios tipos que podem ser desenhados: Por
exemplo, um programador pode adicionar uma `Image`, e outro pode adicionar uma
`SelectBox`.

No momento de escrever a biblioteca, não podemos saber e definir todos os tipos que
outros programadores podem querer criar. Mas sabemos que `gui` precisa manter
o controle de muitos valores de tipos diferentes, e precisa chamar um método `draw`
em cada um desses valores de tipos diferentes. Não precisa saber exatamente o que
acontecerá quando chamarmos o método `draw`, apenas que o valor terá esse
método disponível para chamarmos.

Para fazer isso em uma linguagem com herança, poderíamos definir uma classe chamada
`Component` que tem um método chamado `draw`. As outras classes, tais como
`Button`, `Image`, e `SelectBox`, herdariam de `Component` e assim
herdariam o método `draw`. Elas poderiam cada uma sobrescrever o método `draw` para definir
seu comportamento customizado, mas o framework poderia tratar todos os tipos como se
eles fossem instâncias de `Component` e chamar `draw` neles. Mas como Rust
não tem herança, precisamos de outra maneira de estruturar a biblioteca `gui` para
permitir que usuários criem novos tipos compatíveis com a biblioteca.

### Definindo uma Trait para Comportamento Comum

Para implementar o comportamento que queremos que `gui` tenha, definiremos uma trait
chamada `Draw` que terá um método chamado `draw`. Então, podemos definir um
vetor que recebe um objeto de trait (trait object). Um _objeto de trait_ aponta tanto para uma instância
de um tipo implementando nossa trait especificada quanto para uma tabela usada para procurar métodos
de trait nesse tipo em tempo de execução. Criamos um objeto de trait especificando algum
tipo de ponteiro, tal como uma referência ou um ponteiro inteligente `Box<T>`, então a
palavra-chave `dyn`, e então especificando a trait relevante. (Falaremos sobre a
razão pela qual objetos de trait devem usar um ponteiro em [“Tipos Dinamicamente Dimensionados e a
Trait `Sized`”][dynamically-sized]<!-- ignore --> no Capítulo 20.) Podemos usar
objetos de trait no lugar de um tipo genérico ou concreto. Onde quer que usemos um objeto
de trait, o sistema de tipos do Rust garantirá em tempo de compilação que qualquer valor usado
nesse contexto implementará a trait do objeto de trait. Consequentemente, não
precisamos saber todos os tipos possíveis em tempo de compilação.

Mencionamos que, em Rust, nos abstemos de chamar structs e enums de
“objetos” para distingui-los dos objetos de outras linguagens. Em uma struct ou
enum, os dados nos campos da struct e o comportamento nos blocos `impl` são
separados, enquanto em outras linguagens, os dados e comportamento combinados em um
conceito é frequentemente rotulado como um objeto. Objetos de trait diferem de objetos em outras
linguagens no sentido de que não podemos adicionar dados a um objeto de trait. Objetos de trait não são tão
geralmente úteis quanto objetos em outras linguagens: Seu propósito específico é
permitir abstração através de comportamento comum.

A Listagem 18-3 mostra como definir uma trait chamada `Draw` com um método chamado
`draw`.

<Listing number="18-3" file-name="src/lib.rs" caption="Definição da trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Essa sintaxe deve parecer familiar de nossas discussões sobre como definir traits
no Capítulo 10. A seguir vem alguma sintaxe nova: A Listagem 18-4 define uma struct chamada
`Screen` que segura um vetor chamado `components`. Este vetor é do tipo
`Box<dyn Draw>`, que é um objeto de trait; é um substituto para qualquer tipo dentro de um
`Box` que implementa a trait `Draw`.

<Listing number="18-4" file-name="src/lib.rs" caption="Definição da struct `Screen` com um campo `components` segurando um vetor de objetos de trait que implementam a trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

Na struct `Screen`, definiremos um método chamado `run` que chamará o
método `draw` em cada um de seus `components`, como mostrado na Listagem 18-5.

<Listing number="18-5" file-name="src/lib.rs" caption="Um método `run` em `Screen` que chama o método `draw` em cada componente">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Isso funciona diferentemente de definir uma struct que usa um parâmetro de tipo genérico
com limites de trait. Um parâmetro de tipo genérico pode ser substituído por
apenas um tipo concreto de cada vez, enquanto objetos de trait permitem múltiplos
tipos concretos preencherem o objeto de trait em tempo de execução. Por exemplo, nós
poderíamos ter definido a struct `Screen` usando um tipo genérico e um limite de trait,
como na Listagem 18-6.

<Listing number="18-6" file-name="src/lib.rs" caption="Uma implementação alternativa da struct `Screen` e seu método `run` usando genéricos e limites de trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Isso nos restringe a uma instância de `Screen` que tem uma lista de componentes todos do
tipo `Button` ou todos do tipo `TextField`. Se você só terá coleções homogêneas,
usar genéricos e limites de trait é preferível porque as
definições serão monomorfizadas em tempo de compilação para usar os tipos concretos.

Por outro lado, com o método usando objetos de trait, uma instância de `Screen`
pode segurar um `Vec<T>` que contém um `Box<Button>` bem como um
`Box<TextField>`. Vamos olhar para como isso funciona, e então falaremos sobre as
implicações de desempenho em tempo de execução.

### Implementando a Trait

Agora adicionaremos alguns tipos que implementam a trait `Draw`. Forneceremos o
tipo `Button`. Novamente, implementar uma biblioteca GUI real está além do escopo
deste livro, então o método `draw` não terá nenhuma implementação útil em seu
corpo. Para imaginar como a implementação poderia se parecer, uma struct `Button`
poderia ter campos para `width`, `height`, e `label`, como mostrado na Listagem 18-7.

<Listing number="18-7" file-name="src/lib.rs" caption="Uma struct `Button` que implementa a trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

Os campos `width`, `height`, e `label` em `Button` diferirão dos
campos em outros componentes; por exemplo, um tipo `TextField` poderia ter esses
mesmos campos mais um campo `placeholder`. Cada um dos tipos que queremos desenhar na
tela implementará a trait `Draw` mas usará código diferente no
método `draw` para definir como desenhar esse tipo particular, como `Button` tem aqui
(sem o código GUI real, como mencionado). O tipo `Button`, por exemplo,
poderia ter um bloco `impl` adicional contendo métodos relacionados ao que
acontece quando um usuário clica no botão. Esses tipos de métodos não se aplicarão a
tipos como `TextField`.

Se alguém usando nossa biblioteca decidir implementar uma struct `SelectBox` que tem
campos `width`, `height`, e `options`, eles implementariam a trait `Draw`
no tipo `SelectBox` também, como mostrado na Listagem 18-8.

<Listing number="18-8" file-name="src/main.rs" caption="Outro crate usando `gui` e implementando a trait `Draw` em uma struct `SelectBox`">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

O usuário da nossa biblioteca agora pode escrever sua função `main` para criar uma instância de `Screen`.
Para a instância de `Screen`, eles podem adicionar um `SelectBox` e um `Button`
colocando cada um em um `Box<T>` para se tornar um objeto de trait. Eles podem então chamar o
método `run` na instância de `Screen`, que chamará `draw` em cada um dos
componentes. A Listagem 18-9 mostra esta implementação.

<Listing number="18-9" file-name="src/main.rs" caption="Usando objetos de trait para armazenar valores de tipos diferentes que implementam a mesma trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Quando escrevemos a biblioteca, não sabíamos que alguém poderia adicionar o
tipo `SelectBox`, mas nossa implementação de `Screen` foi capaz de operar no
novo tipo e desenhá-lo porque `SelectBox` implementa a trait `Draw`, o que
significa que implementa o método `draw`.

Esse conceito—de estar preocupado apenas com as mensagens que um valor responde
em vez do tipo concreto do valor—é similar ao conceito de _duck typing_
(tipagem pato) em linguagens dinamicamente tipadas: Se anda como um pato e grasna como
um pato, então deve ser um pato! Na implementação de `run` em `Screen` na
Listagem 18-5, `run` não precisa saber qual é o tipo concreto de cada
componente. Ele não verifica se um componente é uma instância de um `Button`
ou um `SelectBox`, ele apenas chama o método `draw` no componente. Ao
especificar `Box<dyn Draw>` como o tipo dos valores no vetor `components`,
definimos `Screen` para precisar de valores nos quais podemos chamar o método `draw`.

A vantagem de usar objetos de trait e o sistema de tipos do Rust para escrever código
similar a código usando duck typing é que nunca temos que verificar se um
valor implementa um método particular em tempo de execução ou nos preocupar em obter erros
se um valor não implementa um método mas o chamamos de qualquer maneira. Rust não compilará
nosso código se os valores não implementarem as traits que os objetos de trait precisam.

Por exemplo, a Listagem 18-10 mostra o que acontece se tentarmos criar uma `Screen`
com uma `String` como um componente.

<Listing number="18-10" file-name="src/main.rs" caption="Tentando usar um tipo que não implementa a trait do objeto de trait">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

Receberemos este erro porque `String` não implementa a trait `Draw`:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Este erro nos deixa saber que ou estamos passando algo para `Screen` que não
pretendíamos passar e então devemos passar um tipo diferente, ou devemos implementar
`Draw` em `String` para que `Screen` seja capaz de chamar `draw` nela.

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### Realizando Despacho Dinâmico (Dynamic Dispatch)

Lembre-se de [“Desempenho de Código Usando Genéricos”][performance-of-code-using-generics]<!-- ignore --> no Capítulo 10 nossa
discussão sobre o processo de monomorfização realizado em genéricos pelo
compilador: O compilador gera implementações não genéricas de funções e
métodos para cada tipo concreto que usamos no lugar de um parâmetro de tipo
genérico. O código que resulta da monomorfização está fazendo _despacho estático_ (static dispatch),
que é quando o compilador sabe qual método você está chamando em
tempo de compilação. Isso é oposto ao _despacho dinâmico_ (dynamic dispatch), que é quando o compilador
não consegue dizer em tempo de compilação qual método você está chamando. Em casos de despacho dinâmico,
o compilador emite código que em tempo de execução saberá qual método chamar.

Quando usamos objetos de trait, Rust deve usar despacho dinâmico. O compilador não
sabe todos os tipos que podem ser usados com o código que está usando objetos de trait,
então ele não sabe qual método implementado em qual tipo chamar. Em vez disso, em
tempo de execução, Rust usa os ponteiros dentro do objeto de trait para saber qual método
chamar. Essa busca incorre em um custo de tempo de execução que não ocorre com despacho estático.
Despacho dinâmico também impede o compilador de escolher fazer inline do código de um método,
o que por sua vez impede algumas otimizações, e Rust tem algumas regras sobre
onde você pode e não pode usar despacho dinâmico, chamadas _compatibilidade dyn_ (dyn compatibility). Essas
regras estão além do escopo desta discussão, mas você pode ler mais sobre elas
[na referência][dyn-compatibility]<!-- ignore -->. No entanto, ganhamos flexibilidade extra
no código que escrevemos na Listagem 18-5 e fomos capazes de suportar
na Listagem 18-9, então é uma compensação a considerar.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
