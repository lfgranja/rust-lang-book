## Armazenando Listas de Valores com Vetores

A primeira coleção que veremos é `Vec<T>`, também conhecida como *vetor*.
Vetores permitem armazenar mais de um valor em uma única estrutura de dados que
coloca todos os valores um ao lado do outro na memória. Vetores podem armazenar
apenas valores do mesmo tipo. Eles são úteis quando você tem uma lista de
itens, como as linhas de texto em um arquivo ou os preços dos itens em um
carrinho de compras.

### Criando um Novo Vetor

Para criar um novo vetor vazio, chamamos a função `Vec::new`, como mostrado na
Listagem 8-1.

<Listing number="8-1" caption="Criando um novo vetor vazio para conter valores do tipo `i32`">

```rust
let v: Vec<i32> = Vec::new();
```

</Listing>

Observe que adicionamos uma anotação de tipo aqui. Como não estamos inserindo
nenhum valor neste vetor, o Rust não sabe que tipo de elementos pretendemos
armazenar. Este é um ponto importante. Vetores são implementados usando
genéricos; cobriremos como usar genéricos com seus próprios tipos no Capítulo
10. Por enquanto, saiba que o tipo `Vec<T>` fornecido pela biblioteca padrão
pode conter qualquer tipo. Quando criamos um vetor para conter um tipo
específico, especificamos o tipo dentro de colchetes angulares. Na Listagem
8-1, dissemos ao Rust que o `Vec<T>` em `v` conterá elementos do tipo `i32`.

Frequentemente, você criará um `Vec<T>` com valores iniciais e o Rust inferirá
o tipo de valor que você deseja armazenar, então você raramente precisa fazer
essa anotação de tipo. O Rust fornece convenientemente o macro `vec!`, que cria
um novo vetor que contém os valores que você der a ele. A Listagem 8-2 cria um
novo `Vec<i32>` que contém os valores `1`, `2` e `3`. O tipo inteiro é `i32`
porque esse é o tipo inteiro padrão, como discutimos na seção ["Tipos de
Dados"][data-types]<!-- ignore --> do Capítulo 3.

<Listing number="8-2" caption="Criando um novo vetor contendo valores">

```rust
let v = vec![1, 2, 3];
```

</Listing>

Como fornecemos valores `i32` iniciais, o Rust pode inferir que o tipo de `v` é
`Vec<i32>`, e a anotação de tipo não é necessária. A seguir, veremos como
modificar um vetor.

### Atualizando um Vetor

Para criar um vetor e depois adicionar elementos a ele, podemos usar o método
`push`, como mostrado na Listagem 8-3.

<Listing number="8-3" caption="Usando o método `push` para adicionar valores a um vetor">

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

</Listing>

Como com qualquer variável, se quisermos ser capazes de mudar seu valor,
precisamos torná-la mutável usando a palavra-chave `mut`, como discutido no
Capítulo 3. Os números que colocamos dentro são todos do tipo `i32`, e o Rust
infere isso dos dados, então não precisamos da anotação `Vec<i32>`.

### Lendo Elementos de Vetores

Há duas maneiras de referenciar um valor armazenado em um vetor: via indexação
ou usando o método `get`. Nos exemplos a seguir, anotamos os tipos dos valores
retornados dessas funções para maior clareza.

A Listagem 8-4 mostra ambos os métodos de acessar um valor em um vetor, com
sintaxe de indexação e o método `get`.

<Listing number="8-4" caption="Usando sintaxe de indexação ou o método `get` para acessar um item em um vetor">

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("O terceiro elemento é {third}");

let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("O terceiro elemento é {third}"),
    None => println!("Não há terceiro elemento."),
}
```

</Listing>

Note alguns detalhes aqui. Usamos o valor de índice `2` para obter o terceiro
elemento porque vetores são indexados a partir de zero. Usar `&` e `[]` nos dá
uma referência ao elemento no valor do índice. Quando usamos o método `get` com
o índice passado como argumento, obtemos um `Option<&T>` que podemos usar com
`match`.

A razão pela qual o Rust fornece essas duas maneiras de referenciar um elemento
é para que você possa escolher como o programa se comporta quando você tenta
usar um valor de índice fora do intervalo de elementos existentes. Como exemplo,
vamos ver o que acontece quando temos um vetor de cinco elementos e então
tentamos acessar um elemento no índice 100 com cada técnica, como mostrado na
Listagem 8-5.

<Listing number="8-5" caption="Tentando acessar o elemento no índice 100 em um vetor contendo 5 elementos">

```rust,should_panic,panics
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

</Listing>

Quando rodamos este código, o primeiro método `[]` fará com que o programa
entre em pânico porque referencia um elemento inexistente. Este método é melhor
usado quando você quer que seu programa falhe se houver uma tentativa de
acessar um elemento além do final do vetor.

Quando o método `get` recebe um índice que está fora do vetor, ele retorna
`None` sem entrar em pânico. Você usaria este método se acessar um elemento
além do alcance do vetor pudesse acontecer ocasionalmente sob circunstâncias
normais. Seu código terá então lógica para lidar com ter `Some(&element)` ou
`None`, como discutido no Capítulo 6. Por exemplo, o índice poderia vir de uma
pessoa inserindo um número. Se eles inserirem acidentalmente um número que é
muito grande e o programa obtiver um valor `None`, você poderia dizer ao
usuário quantos itens estão no vetor atual e dar a ele outra chance de inserir
um valor válido. Isso seria mais amigável do que travar o programa devido a um
erro de digitação!

Quando o programa tem uma referência válida, o borrow checker impõe as regras de
posse e empréstimo (cobertas no Capítulo 4) para garantir que esta referência e
quaisquer outras referências ao conteúdo do vetor permaneçam válidas. Lembre-se
da regra que afirma que você não pode ter referências mutáveis e imutáveis no
mesmo escopo. Essa regra se aplica na Listagem 8-6, onde mantemos uma
referência imutável ao primeiro elemento em um vetor e tentamos adicionar um
elemento ao final. Este programa não funcionará se também tentarmos nos referir
a esse elemento mais tarde na função:

<Listing number="8-6" caption="Tentando adicionar um elemento a um vetor enquanto mantém uma referência a um item">

```rust,ignore,does_not_compile
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);

println!("O primeiro elemento é: {first}");
```

</Listing>

Compilar este código resultará neste erro:

```text
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 |
8 |     println!("The first element is: {first}");
  |                                      ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to 1 previous error
```

O código na Listagem 8-6 pode parecer que deveria funcionar: por que uma
referência ao primeiro elemento deveria se importar com o que muda no final do
vetor? Este erro é devido à maneira como os vetores funcionam: porque os
vetores colocam os valores um ao lado do outro na memória, adicionar um novo
elemento ao final do vetor pode exigir alocar nova memória e copiar os
elementos antigos para o novo espaço, se não houver espaço suficiente onde o
vetor está atualmente armazenado. Nesse caso, a referência ao primeiro elemento
estaria apontando para memória desalocada. As regras de empréstimo impedem que
programas acabem nessa situação.

> Nota: Para mais informações sobre os detalhes de implementação do tipo `Vec<T>`,
> veja ["O Rustonomicon"][nomicon]<!-- ignore -->.

### Iterando Sobre os Valores em um Vetor

Para acessar cada elemento em um vetor por vez, iteraríamos através de todos os
elementos em vez de usar índices para acessar um de cada vez. A Listagem 8-7
mostra como usar um loop `for` para obter referências imutáveis a cada elemento
em um vetor de valores `i32` e imprimi-los.

<Listing number="8-7" caption="Imprimindo cada elemento em um vetor iterando sobre os elementos usando um loop `for`">

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{i}");
}
```

</Listing>

Também podemos iterar sobre referências mutáveis a cada elemento em um vetor
mutável para fazer alterações em todos os elementos. O loop `for` na Listagem
8-8 adicionará `50` a cada elemento.

<Listing number="8-8" caption="Iterando sobre referências mutáveis a elementos em um vetor">

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

</Listing>

Para alterar o valor ao qual a referência mutável se refere, temos que usar o
operador de desreferenciamento `*` para chegar ao valor em `i` antes de podermos
usar o operador `+=`. Falaremos mais sobre o operador de desreferenciamento no
Capítulo 15.

Iterar sobre um vetor, seja imutavelmente ou mutavelmente, é seguro por causa
das regras do borrow checker. Se tentássemos inserir ou remover itens nos corpos
do loop `for` na Listagem 8-7 e Listagem 8-8, obteríamos um erro de compilador
semelhante ao que obtivemos com o código na Listagem 8-6. A referência ao vetor
que o loop `for` segura impede a modificação simultânea de todo o vetor.

### Usando um Enum para Armazenar Múltiplos Tipos

Vetores podem armazenar apenas valores que são do mesmo tipo. Isso pode ser
inconveniente; há casos de uso para a necessidade de armazenar uma lista de
itens de tipos diferentes. Felizmente, as variantes de um enum são definidas sob
o mesmo tipo de enum, então quando precisamos de um tipo para representar
elementos de tipos diferentes, podemos definir e usar um enum!

Por exemplo, digamos que queremos obter valores de uma linha em uma planilha na
qual algumas das colunas na linha contêm inteiros, alguns números de ponto
flutuante e algumas strings. Podemos definir um enum cujas variantes conterão
os diferentes tipos de valor, e todas as variantes do enum serão consideradas o
mesmo tipo: o do enum. Então podemos criar um vetor para conter esse enum e
assim, em última análise, conter tipos diferentes. Demonstramos isso na Listagem
8-9.

<Listing number="8-9" caption="Definindo um `enum` para armazenar valores de tipos diferentes em um vetor">

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

</Listing>

O Rust precisa saber quais tipos estarão no vetor em tempo de compilação para
que saiba exatamente quanta memória na heap será necessária para armazenar cada
elemento. Também devemos ser explícitos sobre quais tipos são permitidos neste
vetor. Se o Rust permitisse que um vetor contivesse qualquer tipo, haveria uma
chance de que um ou mais tipos causassem erros com as operações realizadas nos
elementos do vetor. Usar um enum mais uma expressão `match` significa que o Rust
garantirá em tempo de compilação que todos os casos possíveis sejam tratados,
como discutido no Capítulo 6.

Se você não souber o conjunto exaustivo de tipos que um programa precisará
armazenar em um vetor em tempo de execução, a técnica de enum não funcionará.
Em vez disso, você pode usar um objeto trait, que cobriremos no Capítulo 17.

Agora que discutimos algumas das maneiras mais comuns de usar vetores, certifique-se
de revisar a [documentação da API][vec-api]<!-- ignore --> para todos os muitos
métodos úteis definidos em `Vec<T>` pela biblioteca padrão. Por exemplo, além
de `push`, um método `pop` remove e retorna o último elemento.

### Descartando um Vetor Descarta seus Elementos

Como qualquer outra `struct`, um vetor é liberado quando sai de escopo, como
anotado na Listagem 8-10.

<Listing number="8-10" caption="Mostrando onde o vetor e seus elementos são descartados">

```rust
{
    let v = vec![1, 2, 3, 4];

    // faz coisas com v
} // <- v sai de escopo e é liberado aqui
```

</Listing>

Quando o vetor é descartado, todo o seu conteúdo também é descartado,
significando que os inteiros que ele contém serão limpos. O borrow checker
garante que quaisquer referências ao conteúdo de um vetor sejam usadas apenas
enquanto o próprio vetor for válido.

Vamos passar para o próximo tipo de coleção: `String`!

[data-types]: ch03-02-data-types.html#tipos-de-dados
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
