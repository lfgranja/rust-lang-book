## Características de Linguagens Orientadas a Objetos

Não há consenso na comunidade de programação sobre quais funcionalidades uma
linguagem deve ter para ser considerada orientada a objetos. Rust é influenciada por muitos
paradigmas de programação, incluindo POO; por exemplo, exploramos as funcionalidades
que vieram da programação funcional no Capítulo 13. Indiscutivelmente, linguagens POO
compartilham certas características comuns—nomeadamente, objetos, encapsulamento e
herança. Vamos olhar para o que cada uma dessas características significa e se
o Rust a suporta.

### Objetos Contêm Dados e Comportamento

O livro _Design Patterns: Elements of Reusable Object-Oriented Software_ por
Erich Gamma, Richard Helm, Ralph Johnson e John Vlissides (Addison-Wesley,
1994), coloquialmente referido como o livro da _Gangue dos Quatro_ (Gang of Four), é um catálogo de
padrões de projeto orientados a objetos. Ele define POO desta maneira:

> Programas orientados a objetos são feitos de objetos. Um **objeto** empacota tanto
> dados quanto os procedimentos que operam nesses dados. Os procedimentos são
> tipicamente chamados de **métodos** ou **operações**.

Usando essa definição, Rust é orientada a objetos: Structs e enums têm dados,
e blocos `impl` fornecem métodos em structs e enums. Embora structs e
enums com métodos não sejam _chamados_ de objetos, eles fornecem a mesma
funcionalidade, de acordo com a definição de objetos da Gangue dos Quatro.

### Encapsulamento que Esconde Detalhes de Implementação

Outro aspecto comumente associado com POO é a ideia de _encapsulamento_,
que significa que os detalhes de implementação de um objeto não são acessíveis ao
código usando esse objeto. Portanto, a única maneira de interagir com um objeto é
através de sua API pública; código usando o objeto não deve ser capaz de alcançar
os internos do objeto e mudar dados ou comportamento diretamente. Isso permite ao
programador mudar e refatorar os internos de um objeto sem precisar
mudar o código que usa o objeto.

Discutimos como controlar o encapsulamento no Capítulo 7: Podemos usar a palavra-chave `pub`
para decidir quais módulos, tipos, funções e métodos em nosso código
devem ser públicos, e por padrão todo o resto é privado. Por exemplo, nós
podemos definir uma struct `AveragedCollection` que tem um campo contendo um vetor
de valores `i32`. A struct também pode ter um campo que contém a média dos
valores no vetor, significando que a média não precisa ser calculada sob
demanda sempre que alguém precisar dela. Em outras palavras, `AveragedCollection` irá
fazer cache da média calculada para nós. A Listagem 18-1 tem a definição da
struct `AveragedCollection`.

<Listing number="18-1" file-name="src/lib.rs" caption="Uma struct `AveragedCollection` que mantém uma lista de inteiros e a média dos itens na coleção">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

A struct é marcada como `pub` para que outro código possa usá-la, mas os campos dentro
da struct permanecem privados. Isso é importante neste caso porque queremos
garantir que sempre que um valor é adicionado ou removido da lista, a média também
seja atualizada. Fazemos isso implementando métodos `add`, `remove` e `average`
na struct, como mostrado na Listagem 18-2.

<Listing number="18-2" file-name="src/lib.rs" caption="Implementações dos métodos públicos `add`, `remove` e `average` em `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Os métodos públicos `add`, `remove` e `average` são as únicas maneiras de acessar
ou modificar dados em uma instância de `AveragedCollection`. Quando um item é adicionado à
`list` usando o método `add` ou removido usando o método `remove`, as
implementações de cada um chamam o método privado `update_average` que lida com
a atualização do campo `average` também.

Deixamos os campos `list` e `average` privados para que não haja maneira de
código externo adicionar ou remover itens do campo `list` diretamente;
caso contrário, o campo `average` poderia ficar dessincronizado quando a `list`
mudasse. O método `average` retorna o valor no campo `average`,
permitindo que código externo leia a `average` mas não a modifique.

Porque encapsulamos os detalhes de implementação da struct
`AveragedCollection`, podemos facilmente mudar aspectos, tais como a estrutura de dados,
no futuro. Por exemplo, poderíamos usar um `HashSet<i32>` em vez de um
`Vec<i32>` para o campo `list`. Desde que as assinaturas dos métodos públicos `add`,
`remove` e `average` permaneçam as mesmas, o código usando
`AveragedCollection` não precisaria mudar. Se tornássemos `list` pública em vez disso,
esse não seria necessariamente o caso: `HashSet<i32>` e `Vec<i32>` têm
métodos diferentes para adicionar e remover itens, então o código externo
provavelmente teria que mudar se estivesse modificando `list` diretamente.

Se encapsulamento é um aspecto necessário para uma linguagem ser considerada orientada a
objetos, então Rust atende a esse requisito. A opção de usar `pub` ou não para
diferentes partes do código permite o encapsulamento de detalhes de implementação.

### Herança como um Sistema de Tipos e como Compartilhamento de Código

_Herança_ é um mecanismo pelo qual um objeto pode herdar elementos da
definição de outro objeto, ganhando assim os dados e comportamento do objeto pai
sem que você tenha que defini-los novamente.

Se uma linguagem deve ter herança para ser orientada a objetos, então Rust não é
tal linguagem. Não há maneira de definir uma struct que herda os campos e
implementações de métodos da struct pai sem usar uma macro.

No entanto, se você está acostumado a ter herança em sua caixa de ferramentas de programação, você
pode usar outras soluções em Rust, dependendo da sua razão para recorrer à
herança em primeiro lugar.

Você escolheria herança por duas razões principais. Uma é para reuso de código:
Você pode implementar comportamento particular para um tipo, e a herança permite que você
reutilize essa implementação para um tipo diferente. Você pode fazer isso de uma maneira limitada
em código Rust usando implementações padrão de métodos de trait, que você viu na
Listagem 10-14 quando adicionamos uma implementação padrão do método `summarize`
na trait `Summary`. Qualquer tipo implementando a trait `Summary` teria
o método `summarize` disponível nele sem qualquer código adicional. Isso é
similar a uma classe pai tendo uma implementação de um método e uma
classe filha herdeira também tendo a implementação do método. Nós também podemos
sobrescrever a implementação padrão do método `summarize` quando nós
implementamos a trait `Summary`, o que é similar a uma classe filha sobrescrevendo a
implementação de um método herdado de uma classe pai.

A outra razão para usar herança relaciona-se ao sistema de tipos: para permitir que um
tipo filho seja usado nos mesmos lugares que o tipo pai. Isso também é
chamado de _polimorfismo_, que significa que você pode substituir múltiplos objetos uns pelos
outros em tempo de execução se eles compartilharem certas características.

> ### Polimorfismo
>
> Para muitas pessoas, polimorfismo é sinônimo de herança. Mas é
> na verdade um conceito mais geral que se refere a código que pode trabalhar com dados de
> múltiplos tipos. Para herança, esses tipos são geralmente subclasses.
>
> Rust usa genéricos para abstrair sobre diferentes tipos possíveis e
> limites de trait (trait bounds) para impor restrições sobre o que esses tipos devem fornecer. Isso é
> às vezes chamado de _polimorfismo paramétrico limitado_.

Rust escolheu um conjunto diferente de compensações (trade-offs) ao não oferecer herança.
Herança está frequentemente em risco de compartilhar mais código do que o necessário. Subclasses
não deveriam sempre compartilhar todas as características de sua classe pai, mas farão isso
com herança. Isso pode tornar o design de um programa menos flexível. Também
introduz a possibilidade de chamar métodos em subclasses que não fazem
sentido ou que causam erros porque os métodos não se aplicam à subclasse. Além disso,
algumas linguagens permitirão apenas _herança simples_ (significando que uma
subclasse pode herdar apenas de uma classe), restringindo ainda mais a flexibilidade
do design de um programa.

Por essas razões, Rust toma a abordagem diferente de usar objetos de trait (trait objects)
em vez de herança para alcançar polimorfismo em tempo de execução. Vamos olhar para como
objetos de trait funcionam.
