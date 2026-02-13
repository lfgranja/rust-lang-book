## Ciclos de Referência Podem Vazar Memória

As garantias de segurança de memória de Rust tornam difícil, mas não impossível, criar acidentalmente memória que nunca é limpa (conhecido como *vazamento de memória* ou *memory leak*). Prevenir vazamentos de memória completamente não é uma das garantias de Rust, o que significa que vazamentos de memória são seguros quanto à memória em Rust. Podemos ver que Rust permite vazamentos de memória usando `Rc<T>` e `RefCell<T>`: é possível criar referências onde itens se referem uns aos outros em um ciclo. Isso cria vazamentos de memória porque a contagem de referências de cada item no ciclo nunca chegará a 0, e os valores nunca serão descartados.

### Criando um Ciclo de Referência

Vamos ver como um ciclo de referência pode acontecer e como preveni-lo, começando com a definição do enum `List` e um método `tail` na Listagem 15-25.

<Listing number="15-25" file-name="src/main.rs" caption="Uma definição de cons list que contém um `RefCell<T>` para que possamos modificar o que uma variante `Cons` está referenciando">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

Estamos usando outra variação da definição de `List` da Listagem 15-5. O segundo elemento na variante `Cons` agora é `RefCell<Rc<List>>`, significando que, em vez de ter a capacidade de modificar o valor `i32` como fizemos na Listagem 15-24, queremos modificar o valor `List` para o qual uma variante `Cons` está apontando. Também estamos adicionando um método `tail` para tornar conveniente para nós acessar o segundo item se tivermos uma variante `Cons`.

Na Listagem 15-26, estamos adicionando uma função `main` que usa as definições na Listagem 15-25. Este código cria uma lista em `a` e uma lista em `b` que aponta para a lista em `a`. Então, modifica a lista em `a` para apontar para `b`, criando um ciclo de referência. Existem instruções `println!` ao longo do caminho para mostrar quais são as contagens de referência em vários pontos neste processo.

<Listing number="15-26" file-name="src/main.rs" caption="Criando um ciclo de referência de dois valores `List` apontando um para o outro">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

Criamos uma instância `Rc<List>` contendo um valor `List` na variável `a` com uma lista inicial de `5, Nil`. Então criamos uma instância `Rc<List>` contendo outro valor `List` na variável `b` que contém o valor `10` e aponta para a lista em `a`.

Modificamos `a` para que aponte para `b` em vez de `Nil`, criando um ciclo. Fazemos isso usando o método `tail` para obter uma referência ao `RefCell<Rc<List>>` em `a`, que colocamos na variável `link`. Então, usamos o método `borrow_mut` no `RefCell<Rc<List>>` para mudar o valor dentro de um `Rc<List>` que contém um valor `Nil` para o `Rc<List>` em `b`.

Quando executamos este código, mantendo o último `println!` comentado por enquanto, obteremos esta saída:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

A contagem de referências das instâncias `Rc<List>` tanto em `a` quanto em `b` é 2 depois que mudamos a lista em `a` para apontar para `b`. No final de `main`, Rust descarta a variável `b`, o que diminui a contagem de referências da instância `Rc<List>` de `b` de 2 para 1. A memória que `Rc<List>` tem na heap não será descartada neste ponto porque sua contagem de referências é 1, não 0. Então, Rust descarta `a`, o que diminui a contagem de referências da instância `Rc<List>` de `a` de 2 para 1 também. A memória desta instância também não pode ser descartada, porque a outra instância `Rc<List>` ainda se refere a ela. A memória alocada para a lista permanecerá não coletada para sempre. Para visualizar este ciclo de referência, criamos o diagrama na Figura 15-4.

<img alt="Um retângulo rotulado 'a' que aponta para um retângulo contendo o inteiro 5. Um retângulo rotulado 'b' que aponta para um retângulo contendo o inteiro 10. O retângulo contendo 5 aponta para o retângulo contendo 10, e o retângulo contendo 10 aponta de volta para o retângulo contendo 5, criando um ciclo." src="img/trpl15-04.svg" class="center" />

<span class="caption">Figura 15-4: Um ciclo de referência das listas `a` e `b` apontando uma para a outra</span>

Se você descomentar o último `println!` e executar o programa, Rust tentará imprimir este ciclo com `a` apontando para `b` apontando para `a` e assim por diante até estourar a pilha.

Comparado a um programa do mundo real, as consequências de criar um ciclo de referência neste exemplo não são muito terríveis: logo após criarmos o ciclo de referência, o programa termina. No entanto, se um programa mais complexo alocasse muita memória em um ciclo e a mantivesse por um longo tempo, o programa usaria mais memória do que precisava e poderia sobrecarregar o sistema, fazendo-o ficar sem memória disponível.

Criar ciclos de referência não é feito facilmente, mas também não é impossível. Se você tiver valores `RefCell<T>` que contêm valores `Rc<T>` ou combinações aninhadas semelhantes de tipos com mutabilidade interior e contagem de referências, você deve garantir que não cria ciclos; você não pode confiar em Rust para capturá-los. Criar um ciclo de referência seria um bug de lógica em seu programa que você deve usar testes automatizados, revisões de código e outras práticas de desenvolvimento de software para minimizar.

Outra solução para evitar ciclos de referência é reorganizar suas estruturas de dados para que algumas referências expressem posse e algumas referências não. Como resultado, você pode ter ciclos compostos por alguns relacionamentos de posse e alguns relacionamentos de não-posse, e apenas os relacionamentos de posse afetam se um valor pode ou não ser descartado. Na Listagem 15-25, sempre queremos que as variantes `Cons` possuam sua lista, então reorganizar a estrutura de dados não é possível. Vamos ver um exemplo usando grafos compostos por nós pais e nós filhos para ver quando relacionamentos de não-posse são uma maneira apropriada de prevenir ciclos de referência.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### Prevenindo Ciclos de Referência Usando `Weak<T>`

Até agora, demonstramos que chamar `Rc::clone` aumenta a `strong_count` (contagem forte) de uma instância `Rc<T>`, e uma instância `Rc<T>` só é limpa se sua `strong_count` for 0. Você também pode criar uma *referência fraca* (weak reference) para o valor dentro de uma instância `Rc<T>` chamando `Rc::downgrade` e passando uma referência ao `Rc<T>`. *Referências fortes* são como você pode compartilhar a posse de uma instância `Rc<T>`. *Referências fracas* não expressam um relacionamento de posse, e sua contagem não afeta quando uma instância `Rc<T>` é limpa. Elas não causarão um ciclo de referência, porque qualquer ciclo envolvendo algumas referências fracas será quebrado assim que a contagem de referências fortes dos valores envolvidos for 0.

Quando você chama `Rc::downgrade`, você obtém um ponteiro inteligente do tipo `Weak<T>`. Em vez de aumentar a `strong_count` na instância `Rc<T>` em 1, chamar `Rc::downgrade` aumenta a `weak_count` em 1. O tipo `Rc<T>` usa `weak_count` para manter o controle de quantas referências `Weak<T>` existem, semelhante a `strong_count`. A diferença é que a `weak_count` não precisa ser 0 para que a instância `Rc<T>` seja limpa.

Como o valor que `Weak<T>` referencia pode ter sido descartado, para fazer qualquer coisa com o valor para o qual um `Weak<T>` aponta, você deve se certificar de que o valor ainda existe. Faça isso chamando o método `upgrade` em uma instância `Weak<T>`, que retornará um `Option<Rc<T>>`. Você receberá um resultado de `Some` se o valor `Rc<T>` ainda não tiver sido descartado e um resultado de `None` se o valor `Rc<T>` tiver sido descartado. Como `upgrade` retorna um `Option<Rc<T>>`, Rust garantirá que o caso `Some` e o caso `None` sejam tratados, e não haverá um ponteiro inválido.

Como exemplo, em vez de usar uma lista cujos itens sabem apenas sobre o próximo item, criaremos uma árvore cujos itens sabem sobre seus itens filhos *e* seus itens pais.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### Criando uma Estrutura de Dados de Árvore

Para começar, construiremos uma árvore com nós que sabem sobre seus nós filhos. Criaremos uma struct chamada `Node` que contém seu próprio valor `i32` bem como referências aos seus valores `Node` filhos:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Queremos que um `Node` possua seus filhos, e queremos compartilhar essa posse com variáveis para que possamos acessar cada `Node` na árvore diretamente. Para fazer isso, definimos os itens `Vec<T>` para serem valores do tipo `Rc<Node>`. Também queremos modificar quais nós são filhos de outro nó, então temos um `RefCell<T>` em `children` ao redor do `Vec<Rc<Node>>`.

Em seguida, usaremos nossa definição de struct e criaremos uma instância de `Node` chamada `leaf` (folha) com o valor `3` e sem filhos, e outra instância chamada `branch` (galho) com o valor `5` e `leaf` como um de seus filhos, como mostrado na Listagem 15-27.

<Listing number="15-27" file-name="src/main.rs" caption="Criando um nó `leaf` sem filhos e um nó `branch` com `leaf` como um de seus filhos">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

Clonamos o `Rc<Node>` em `leaf` e armazenamos isso em `branch`, significando que o `Node` em `leaf` agora tem dois donos: `leaf` e `branch`. Podemos ir de `branch` para `leaf` através de `branch.children`, mas não há como ir de `leaf` para `branch`. A razão é que `leaf` não tem referência a `branch` e não sabe que eles estão relacionados. Queremos que `leaf` saiba que `branch` é seu pai. Faremos isso a seguir.

#### Adicionando uma Referência de um Filho para Seu Pai

Para tornar o nó filho ciente de seu pai, precisamos adicionar um campo `parent` à nossa definição de struct `Node`. O problema é decidir qual deve ser o tipo de `parent`. Sabemos que não pode conter um `Rc<T>`, porque isso criaria um ciclo de referência com `leaf.parent` apontando para `branch` e `branch.children` apontando para `leaf`, o que causaria que seus valores de `strong_count` nunca fossem 0.

Pensando nos relacionamentos de outra maneira, um nó pai deve possuir seus filhos: se um nó pai for descartado, seus nós filhos devem ser descartados também. No entanto, um filho não deve possuir seu pai: se descartarmos um nó filho, o pai ainda deve existir. Este é um caso para referências fracas!

Então, em vez de `Rc<T>`, faremos o tipo de `parent` usar `Weak<T>`, especificamente um `RefCell<Weak<Node>>`. Agora nossa definição de struct `Node` se parece com isso:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Um nó será capaz de se referir ao seu nó pai, mas não possui seu pai. Na Listagem 15-28, atualizamos `main` para usar esta nova definição para que o nó `leaf` tenha uma maneira de se referir ao seu pai, `branch`.

<Listing number="15-28" file-name="src/main.rs" caption="Um nó `leaf` com uma referência fraca ao seu nó pai, `branch`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

Criar o nó `leaf` parece semelhante à Listagem 15-27 com a exceção do campo `parent`: `leaf` começa sem um pai, então criamos uma nova instância de referência `Weak<Node>` vazia.

Neste ponto, quando tentamos obter uma referência ao pai de `leaf` usando o método `upgrade`, recebemos um valor `None`. Vemos isso na saída da primeira instrução `println!`:

```text
leaf parent = None
```

Quando criamos o nó `branch`, ele também terá uma nova referência `Weak<Node>` no campo `parent` porque `branch` não tem um nó pai. Ainda temos `leaf` como um dos filhos de `branch`. Uma vez que temos a instância `Node` em `branch`, podemos modificar `leaf` para dar a ele uma referência `Weak<Node>` ao seu pai. Usamos o método `borrow_mut` no `RefCell<Weak<Node>>` no campo `parent` de `leaf`, e então usamos a função `Rc::downgrade` para criar uma referência `Weak<Node>` para `branch` a partir do `Rc<Node>` em `branch`.

Quando imprimimos o pai de `leaf` novamente, desta vez obteremos uma variante `Some` contendo `branch`: agora `leaf` pode acessar seu pai! Quando imprimimos `leaf`, também evitamos o ciclo que eventualmente terminava em um estouro de pilha como tivemos na Listagem 15-26; as referências `Weak<Node>` são impressas como `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

A falta de saída infinita indica que este código não criou um ciclo de referência. Também podemos dizer isso olhando para os valores que obtemos chamando `Rc::strong_count` e `Rc::weak_count`.

#### Visualizando Mudanças em `strong_count` e `weak_count`

Vamos ver como os valores `strong_count` e `weak_count` das instâncias `Rc<Node>` mudam criando um novo escopo interno e movendo a criação de `branch` para esse escopo. Ao fazer isso, podemos ver o que acontece quando `branch` é criado e depois descartado quando sai de escopo. As modificações são mostradas na Listagem 15-29.

<Listing number="15-29" file-name="src/main.rs" caption="Criando `branch` em um escopo interno e examinando contagens de referência fortes e fracas">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

Depois que `leaf` é criado, seu `Rc<Node>` tem uma contagem forte de 1 e uma contagem fraca de 0. No escopo interno, criamos `branch` e o associamos a `leaf`, momento em que, quando imprimimos as contagens, o `Rc<Node>` em `branch` terá uma contagem forte de 1 e uma contagem fraca de 1 (para `leaf.parent` apontando para `branch` com um `Weak<Node>`). Quando imprimimos as contagens em `leaf`, veremos que ele terá uma contagem forte de 2 porque `branch` agora tem um clone do `Rc<Node>` de `leaf` armazenado em `branch.children`, mas ainda terá uma contagem fraca de 0.

Quando o escopo interno termina, `branch` sai de escopo e a contagem forte do `Rc<Node>` diminui para 0, então seu `Node` é descartado. A contagem fraca de 1 de `leaf.parent` não tem influência sobre se o `Node` é descartado ou não, então não temos vazamentos de memória!

Se tentarmos acessar o pai de `leaf` após o final do escopo, obteremos `None` novamente. No final do programa, o `Rc<Node>` em `leaf` tem uma contagem forte de 1 e uma contagem fraca de 0 porque a variável `leaf` agora é a única referência ao `Rc<Node>` novamente.

Toda a lógica que gerencia as contagens e o descarte de valores é construída em `Rc<T>` e `Weak<T>` e suas implementações da trait `Drop`. Ao especificar que o relacionamento de um filho para seu pai deve ser uma referência `Weak<T>` na definição de `Node`, você é capaz de ter nós pais apontando para nós filhos e vice-versa sem criar um ciclo de referência e vazamentos de memória.

## Resumo

Este capítulo cobriu como usar ponteiros inteligentes para fazer garantias e compensações diferentes daquelas que Rust faz por padrão com referências regulares. O tipo `Box<T>` tem um tamanho conhecido e aponta para dados alocados na heap. O tipo `Rc<T>` mantém o controle do número de referências a dados na heap para que os dados possam ter múltiplos donos. O tipo `RefCell<T>` com sua mutabilidade interior nos dá um tipo que podemos usar quando precisamos de um tipo imutável, mas precisamos mudar um valor interno desse tipo; ele também impõe as regras de empréstimo em tempo de execução em vez de em tempo de compilação.

Também foram discutidas as traits `Deref` e `Drop`, que habilitam muitas das funcionalidades dos ponteiros inteligentes. Exploramos ciclos de referência que podem causar vazamentos de memória e como preveni-los usando `Weak<T>`.

Se este capítulo despertou seu interesse e você quer implementar seus próprios ponteiros inteligentes, confira ["The Rustonomicon"][nomicon] para informações mais úteis.

A seguir, falaremos sobre concorrência em Rust. Você aprenderá até sobre alguns novos ponteiros inteligentes.

[nomicon]: ../nomicon/index.html
