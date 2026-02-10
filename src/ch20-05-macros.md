# Macros

Usamos macros como `println!` ao longo deste livro, mas não exploramos totalmente o que é uma macro e como ela funciona. O termo *macro* refere-se a uma família de recursos em Rust: macros *declarativas* com `macro_rules!` e três tipos de macros *procedurais*:

- Macros `#[derive]` personalizadas que especificam o código adicionado com o atributo `derive` usado em structs e enums
- Macros do tipo atributo (attribute-like) que definem atributos personalizados utilizáveis em qualquer item
- Macros do tipo função (function-like) que parecem chamadas de função, mas operam nos tokens especificados como seu argumento

Falaremos sobre cada uma delas, mas primeiro, vamos ver por que precisamos de macros quando já temos funções.

## A Diferença Entre Macros e Funções

Fundamentalmente, macros são uma maneira de escrever código que escreve outro código, o que é conhecido como *metaprogramação*. No Apêndice C, discutimos o atributo `derive`, que gera uma implementação de vários traits para você. Também usamos as macros `println!` e `vec!` ao longo do livro. Todas essas macros *expandem* para produzir mais código do que o código que você escreveu manualmente.

A metaprogramação é útil para reduzir a quantidade de código que você precisa escrever e manter, o que também é uma das funções das funções. No entanto, macros têm alguns poderes adicionais que as funções não têm.

Uma assinatura de função deve declarar o número e o tipo de parâmetros que a função possui. Macros, por outro lado, podem receber um número variável de parâmetros: podemos chamar `println!("ola")` com um argumento ou `println!("ola {}", nome)` com dois argumentos. Além disso, as macros são expandidas antes que o compilador interprete o significado do código, então uma macro pode, por exemplo, implementar um trait em um determinado tipo. Uma função não pode, porque é chamada em tempo de execução e um trait precisa ser implementado em tempo de compilação.

A desvantagem de implementar uma macro em vez de uma função é que as definições de macro são mais complexas do que as definições de função porque você está escrevendo código Rust que escreve código Rust. Devido a essa indireção, as definições de macro são geralmente mais difíceis de ler, entender e manter do que as definições de função.

Outra diferença importante entre macros e funções é que você deve definir macros ou trazê-las para o escopo *antes* de chamá-las em um arquivo, ao contrário das funções que você pode definir em qualquer lugar e chamar em qualquer lugar.

## Macros Declarativas com `macro_rules!` para Metaprogramação Geral

A forma mais amplamente usada de macros em Rust é a *macro declarativa*. Elas também são às vezes referidas como "macros por exemplo", "macros `macro_rules!`" ou apenas "macros". Em sua essência, as macros declarativas permitem que você escreva algo semelhante a uma expressão `match` do Rust. Como discutido no Capítulo 6, as expressões `match` são estruturas de controle que recebem uma expressão, comparam o valor resultante da expressão com padrões e, em seguida, executam o código associado ao padrão correspondente. Macros também comparam um valor a padrões que estão associados a um código específico: nesta situação, o valor é o código-fonte literal Rust passado para a macro; os padrões são comparados com a estrutura desse código-fonte; e o código associado a cada padrão, quando correspondido, substitui o código passado para a macro. Tudo isso acontece durante a compilação.

Para definir uma macro, você usa a construção `macro_rules!`. Vamos explorar como usar `macro_rules!` observando como a macro `vec!` é definida. O Capítulo 8 cobriu como podemos usar a macro `vec!` para criar um novo vetor com valores específicos. Por exemplo, a seguinte macro cria um novo vetor contendo três inteiros:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Também poderíamos usar a macro `vec!` para fazer um vetor de dois inteiros ou um vetor de cinco fatias de string. Não seríamos capazes de usar uma função para fazer o mesmo porque não saberíamos o número ou tipo de valores antecipadamente.

O Listagem 19-28 mostra uma definição ligeiramente simplificada da macro `vec!`.

Listagem 19-28: Uma versão simplificada da definição da macro `vec!`

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

> Nota: A definição real da macro `vec!` na biblioteca padrão inclui código para pré-alocar a quantidade correta de memória antecipadamente. Esse código é uma otimização que não incluímos aqui, para tornar o exemplo mais simples.

A anotação `#[macro_export]` indica que esta macro deve ser disponibilizada sempre que a crate na qual a macro é definida for trazida para o escopo. Sem essa anotação, a macro não pode ser trazida para o escopo.

Em seguida, iniciamos a definição da macro com `macro_rules!` e o nome da macro que estamos definindo *sem* o ponto de exclamação. O nome, neste caso `vec`, é seguido por chaves denotando o corpo da definição da macro.

A estrutura no corpo de `vec!` é semelhante à estrutura de uma expressão `match`. Aqui temos um braço com o padrão `( $( $x:expr ),* )`, seguido por `=>` e o bloco de código associado a este padrão. Se o padrão corresponder, o bloco de código associado será emitido. Dado que este é o único padrão nesta macro, há apenas uma maneira válida de corresponder; qualquer outro padrão resultará em um erro. Macros mais complexas terão mais de um braço.

A sintaxe de padrão válida em definições de macro é diferente da sintaxe de padrão coberta no Capítulo 18 porque os padrões de macro são comparados com a estrutura do código Rust em vez de valores. Vamos percorrer o que as peças do padrão no Listagem 19-28 significam; para a sintaxe completa de padrão de macro, consulte a Referência do Rust.

Primeiro, usamos um conjunto de parênteses para abranger todo o padrão. Usamos um cifrão (`$`) para declarar uma variável no sistema de macro que conterá o código Rust correspondente ao padrão. O cifrão deixa claro que esta é uma variável de macro, em oposição a uma variável Rust regular. Em seguida, vem um conjunto de parênteses que captura valores que correspondem ao padrão dentro dos parênteses para uso no código de substituição. Dentro de `$()` está `$x:expr`, que corresponde a qualquer expressão Rust e dá à expressão o nome `$x`.

A vírgula seguindo `$()` indica que um caractere separador de vírgula literal pode aparecer opcionalmente após o código que corresponde ao código em `$()`. O `*` especifica que o padrão corresponde a zero ou mais de qualquer coisa que precede o `*`.

Quando chamamos esta macro com `vec![1, 2, 3];`, o padrão `$x` corresponde três vezes com as três expressões `1`, `2` e `3`.

Agora vamos olhar para o padrão no corpo do código associado a este braço: `temp_vec.push()` dentro de `$()*` é gerado para cada parte que corresponde a `$()` no padrão zero ou mais vezes, dependendo de quantas vezes o padrão corresponde. O `$x` é substituído por cada expressão correspondida. Quando chamamos esta macro com `vec![1, 2, 3];`, o código gerado que substitui esta chamada de macro será o seguinte:

```rust
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Definimos uma macro que pode receber qualquer número de argumentos de qualquer tipo e pode gerar código para criar um vetor contendo os elementos especificados.

Para saber mais sobre como escrever macros, consulte a documentação online ou outros recursos, como "The Little Book of Rust Macros" iniciado por Daniel Keep e continuado por Lukas Wirth.

## Macros Procedurais para Gerar Código a partir de Atributos

A segunda forma de macros é a *macro procedural*, que age mais como uma função (e é um tipo de procedimento). Macros procedurais aceitam algum código como entrada, operam nesse código e produzem algum código como saída, em vez de corresponder a padrões e substituir o código por outro código como as macros declarativas fazem. Os três tipos de macros procedurais são derive personalizada, tipo atributo e tipo função, e todas funcionam de maneira semelhante.

Ao criar macros procedurais, as definições devem residir em sua própria crate com um tipo de crate especial. Isso ocorre por razões técnicas complexas que esperamos eliminar no futuro. No Listagem 19-29, mostramos como definir uma macro procedural, onde `algum_atributo` é um espaço reservado para usar uma variedade de macro específica.

Listagem 19-29: Um exemplo de definição de uma macro procedural

```rust
use proc_macro::TokenStream;

#[algum_atributo]
pub fn algum_nome(input: TokenStream) -> TokenStream {
}
```

A função que define uma macro procedural recebe um `TokenStream` como entrada e produz um `TokenStream` como saída. O tipo `TokenStream` é definido pela crate `proc_macro` que está incluída no Rust e representa uma sequência de tokens. Este é o núcleo da macro: o código-fonte no qual a macro está operando compõe o `TokenStream` de entrada, e o código que a macro produz é o `TokenStream` de saída. A função também possui um atributo anexado a ela que especifica qual tipo de macro procedural estamos criando. Podemos ter vários tipos de macros procedurais na mesma crate.

Vamos ver os diferentes tipos de macros procedurais. Começaremos com uma macro derive personalizada e, em seguida, explicaremos as pequenas diferenças que tornam as outras formas diferentes.

### Macros `derive` Personalizadas

Vamos criar uma crate chamada `hello_macro` que define um trait chamado `HelloMacro` com uma função associada chamada `hello_macro`. Em vez de fazer com que nossos usuários implementem o trait `HelloMacro` para cada um de seus tipos, forneceremos uma macro procedural para que os usuários possam anotar seu tipo com `#[derive(HelloMacro)]` para obter uma implementação padrão da função `hello_macro`. A implementação padrão imprimirá `Hello, Macro! Meu nome é TypeName!` onde `TypeName` é o nome do tipo no qual este trait foi definido. Em outras palavras, escreveremos uma crate que permite a outro programador escrever código como o Listagem 19-30 usando nossa crate.

Listagem 19-30: O código que um usuário da nossa crate poderá escrever ao usar nossa macro procedural

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Panquecas;

fn main() {
    Panquecas::hello_macro();
}
```

Este código imprimirá `Hello, Macro! Meu nome é Panquecas!` quando terminarmos. O primeiro passo é fazer uma nova crate de biblioteca, assim:

```console
$ cargo new hello_macro --lib
```

Em seguida, definiremos o trait `HelloMacro` e sua função associada.

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

Temos um trait e sua função. Neste ponto, nosso usuário da crate poderia implementar o trait para alcançar a funcionalidade desejada, como no Listagem 19-31.

Listagem 19-31: Como ficaria se os usuários escrevessem uma implementação manual do trait `HelloMacro`

```rust
use hello_macro::HelloMacro;

struct Panquecas;

impl HelloMacro for Panquecas {
    fn hello_macro() {
        println!("Hello, Macro! Meu nome é Panquecas!");
    }
}

fn main() {
    Panquecas::hello_macro();
}
```

No entanto, eles precisariam escrever o bloco de implementação para cada tipo que quisessem usar com `hello_macro`; queremos poupá-los de ter que fazer esse trabalho.

Além disso, ainda não podemos fornecer a função `hello_macro` com implementação padrão que imprimirá o nome do tipo no qual o trait é implementado: Rust não tem capacidades de reflexão, então não pode procurar o nome do tipo em tempo de execução. Precisamos de uma macro para gerar código em tempo de compilação.

O próximo passo é definir a macro procedural. No momento da escrita deste texto, as macros procedurais precisam estar em sua própria crate. Eventualmente, essa restrição pode ser levantada. A convenção para estruturar crates e crates de macro é a seguinte: para uma crate chamada `foo`, uma crate de macro procedural derive personalizada é chamada `foo_derive`. Vamos começar uma nova crate chamada `hello_macro_derive` dentro do nosso projeto `hello_macro`:

```console
$ cargo new hello_macro_derive --lib
```

Nossas duas crates estão intimamente relacionadas, então criamos a crate de macro procedural dentro do diretório de nossa crate `hello_macro`. Se alterarmos a definição do trait em `hello_macro`, teremos que alterar a implementação da macro procedural em `hello_macro_derive` também. As duas crates precisarão ser publicadas separadamente, e os programadores que usam essas crates precisarão adicionar ambas como dependências e trazer ambas para o escopo. Poderíamos, em vez disso, fazer com que a crate `hello_macro` usasse `hello_macro_derive` como uma dependência e reexportasse o código da macro procedural. No entanto, a maneira como estruturamos o projeto torna possível para os programadores usarem `hello_macro` mesmo se não quiserem a funcionalidade `derive`.

Precisamos declarar a crate `hello_macro_derive` como uma crate de macro procedural. Também precisaremos de funcionalidade das crates `syn` e `quote`, como você verá em um momento, então precisamos adicioná-las como dependências. Adicione o seguinte ao arquivo *Cargo.toml* para `hello_macro_derive`:

```toml
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
```

Para começar a definir a macro procedural, coloque o código no Listagem 19-32 em seu arquivo *src/lib.rs* para a crate `hello_macro_derive`. Observe que este código não compilará até adicionarmos uma definição para a função `impl_hello_macro`.

Listagem 19-32: Código que a maioria das crates de macro procedural exigirá para processar código Rust

```rust
extern crate proc_macro;

use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construa uma representação de código Rust em nossa árvore de sintaxe
    // que possamos manipular
    let ast = syn::parse(input).unwrap();

    // Construa a implementação do trait
    impl_hello_macro(&ast)
}
```

Observe que dividimos o código na função `hello_macro_derive`, que é responsável por analisar o `TokenStream`, e na função `impl_hello_macro`, que é responsável por transformar a árvore de sintaxe: isso torna a escrita de uma macro procedural mais conveniente. O código na função externa (`hello_macro_derive` neste caso) será o mesmo para quase todas as crates de macro procedural que você vir ou criar. O código que você especifica no corpo da função interna (`impl_hello_macro` neste caso) será diferente dependendo do propósito da sua macro procedural.

Introduzimos três novas crates: `proc_macro`, `syn` e `quote`. A crate `proc_macro` vem com Rust, então não precisamos adicioná-la às dependências em *Cargo.toml*. A crate `proc_macro` é a API do compilador que nos permite ler e manipular código Rust a partir do nosso código.

A crate `syn` analisa o código Rust de uma string em uma estrutura de dados na qual podemos realizar operações. A crate `quote` transforma as estruturas de dados `syn` de volta em código Rust. Essas crates tornam muito mais simples analisar qualquer tipo de código Rust que possamos querer manipular: escrever um analisador completo para o código Rust não é uma tarefa simples.

A função `hello_macro_derive` será chamada quando um usuário de nossa biblioteca especificar `#[derive(HelloMacro)]` em um tipo. Isso é possível porque anotamos a função `hello_macro_derive` aqui com `proc_macro_derive` e especificamos o nome `HelloMacro`, que corresponde ao nome do nosso trait; esta é a convenção que a maioria das macros procedurais segue.

A função `hello_macro_derive` primeiro converte o `input` de um `TokenStream` em uma estrutura de dados que podemos interpretar e realizar operações. É aqui que `syn` entra em jogo. A função `parse` em `syn` recebe um `TokenStream` e retorna uma struct `DeriveInput` representando o código Rust analisado. O Listagem 19-33 mostra as partes relevantes da struct `DeriveInput` que obtemos ao analisar a string `struct Panquecas;`.

Listagem 19-33: A instância `DeriveInput` que obtemos ao analisar o código que tem o atributo da macro no Listagem 19-30

```rust
DeriveInput {
    // --trecho omitido--

    ident: Ident {
        ident: "Panquecas",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

Os campos desta struct mostram que o código Rust que analisamos é uma struct de unidade com o `ident` (*identificador*, significando o nome) de `Panquecas`. Existem mais campos nesta struct para descrever todos os tipos de código Rust; verifique a documentação de `syn` para `DeriveInput` para mais informações.

Em breve definiremos a função `impl_hello_macro`, que é onde construiremos o novo código Rust que queremos incluir. Mas antes de fazermos isso, observe que a saída para nossa macro `derive` também é um `TokenStream`. O `TokenStream` retornado é adicionado ao código que nossos usuários da crate escrevem, então, quando eles compilam sua crate, eles obterão a funcionalidade extra que fornecemos no `TokenStream` modificado.

Você pode ter notado que estamos chamando `unwrap` para fazer a função `hello_macro_derive` entrar em pânico se a chamada para a função `syn::parse` falhar aqui. É necessário que nossa macro procedural entre em pânico em erros porque as funções `proc_macro_derive` devem retornar `TokenStream` em vez de `Result` para estar em conformidade com a API de macro procedural. Simplificamos este exemplo usando `unwrap`; em código de produção, você deve fornecer mensagens de erro mais específicas sobre o que deu errado usando `panic!` ou `expect`.

Agora que temos o código para transformar o código Rust anotado de um `TokenStream` em uma instância `DeriveInput`, vamos gerar o código que implementa o trait `HelloMacro` no tipo anotado, como mostrado no Listagem 19-34.

Listagem 19-34: Implementando o trait `HelloMacro` usando o código Rust analisado

```rust
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! Meu nome é {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

Obtemos uma instância de struct `Ident` contendo o nome (identificador) do tipo anotado usando `ast.ident`. A struct no Listagem 19-33 mostra que quando executamos a função `impl_hello_macro` no código do Listagem 19-30, o `ident` que obtemos terá o campo `ident` com um valor de `"Panquecas"`. Assim, a variável `name` no Listagem 19-34 conterá uma instância de struct `Ident` que, quando impressa, será a string `"Panquecas"`, o nome da struct no Listagem 19-30.

A macro `quote!` nos permite definir o código Rust que queremos retornar. O compilador espera algo diferente do resultado direto da execução da macro `quote!`, então precisamos convertê-lo em um `TokenStream`. Fazemos isso chamando o método `into`, que consome essa representação intermediária e retorna um valor do tipo `TokenStream` necessário.

A macro `quote!` também fornece alguns mecanismos de modelagem muito legais: podemos inserir `#name`, e `quote!` o substituirá pelo valor na variável `name`. Você pode até fazer alguma repetição semelhante à maneira como as macros regulares funcionam. Confira a documentação da crate `quote` para uma introdução completa.

Queremos que nossa macro procedural gere uma implementação do nosso trait `HelloMacro` para o tipo que o usuário anotou, que podemos obter usando `#name`. A implementação do trait tem a única função `hello_macro`, cujo corpo contém a funcionalidade que queremos fornecer: imprimir `Hello, Macro! Meu nome é` e depois o nome do tipo anotado.

A macro `stringify!` usada aqui é incorporada ao Rust. Ela recebe uma expressão Rust, como `1 + 2`, e em tempo de compilação transforma a expressão em um literal de string, como `"1 + 2"`. Isso é diferente de `format!` ou `println!`, que são macros que avaliam a expressão e depois transformam o resultado em uma `String`. Existe a possibilidade de que a entrada `#name` possa ser uma expressão para imprimir literalmente, então usamos `stringify!`. Usar `stringify!` também economiza uma alocação convertendo `#name` em um literal de string em tempo de compilação.

Neste ponto, `cargo build` deve ser concluído com sucesso tanto em `hello_macro` quanto em `hello_macro_derive`. Vamos conectar essas crates ao código no Listagem 19-30 para ver a macro procedural em ação! Crie um novo projeto binário em seu diretório *projects* usando `cargo new panquecas`. Precisamos adicionar `hello_macro` e `hello_macro_derive` como dependências no *Cargo.toml* da crate `panquecas`. Se você estiver publicando suas versões de `hello_macro` e `hello_macro_derive` no crates.io, elas seriam dependências regulares; se não, você pode especificá-las como dependências de `path`.

Coloque o código do Listagem 19-30 em *src/main.rs* e execute `cargo run`: ele deve imprimir `Hello, Macro! Meu nome é Panquecas!`. A implementação do trait `HelloMacro` da macro procedural foi incluída sem que a crate `panquecas` precisasse implementá-la; o `#[derive(HelloMacro)]` adicionou a implementação do trait.

A seguir, vamos explorar como os outros tipos de macros procedurais diferem das macros `derive` personalizadas.

### Macros do Tipo Atributo

Macros do tipo atributo (attribute-like macros) são semelhantes às macros `derive` personalizadas, mas em vez de gerar código para o atributo `derive`, elas permitem que você crie novos atributos. Elas também são mais flexíveis: `derive` funciona apenas para structs e enums; atributos podem ser aplicados a outros itens também, como funções. Aqui está um exemplo de uso de uma macro do tipo atributo. Digamos que você tenha um atributo chamado `rota` que anota funções ao usar um framework de aplicação web:

```rust
#[rota(GET, "/")]
fn index() {
```

Este atributo `#[rota]` seria definido pelo framework como uma macro procedural. A assinatura da função de definição da macro seria assim:

```rust
#[proc_macro_attribute]
pub fn rota(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Aqui, temos dois parâmetros do tipo `TokenStream`. O primeiro é para o conteúdo do atributo: a parte `GET, "/"`. O segundo é o corpo do item ao qual o atributo está anexado: neste caso, `fn index() {}` e o resto do corpo da função.

Fora isso, as macros do tipo atributo funcionam da mesma maneira que as macros `derive` personalizadas: você cria uma crate com o tipo de crate `proc-macro` e implementa uma função que gera o código que você deseja!

### Macros do Tipo Função

Macros do tipo função (function-like macros) definem macros que parecem chamadas de função. Da mesma forma que as macros `macro_rules!`, elas são mais flexíveis do que funções; por exemplo, elas podem receber um número desconhecido de argumentos. No entanto, as macros `macro_rules!` só podem ser definidas usando a sintaxe de correspondência que discutimos na seção "Macros Declarativas com `macro_rules!` para Metaprogramação Geral" anteriormente. Macros do tipo função recebem um parâmetro `TokenStream`, e sua definição manipula esse `TokenStream` usando código Rust como os outros dois tipos de macros procedurais fazem. Um exemplo de uma macro do tipo função é uma macro `sql!` que pode ser chamada assim:

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Esta macro analisaria a instrução SQL dentro dela e verificaria se está sintaticamente correta, o que é um processamento muito mais complexo do que uma macro `macro_rules!` pode fazer. A macro `sql!` seria definida assim:

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Esta definição é semelhante à assinatura da macro `derive` personalizada: recebemos os tokens que estão dentro dos parênteses e retornamos o código que queríamos gerar.

## Resumo

Ufa! Agora você tem algumas funcionalidades de Rust em sua caixa de ferramentas que você provavelmente não usará com frequência, mas saberá que elas estão disponíveis em circunstâncias muito específicas. Introduzimos vários tópicos complexos para que, quando você os encontrar em sugestões de mensagens de erro ou no código de outras pessoas, você seja capaz de reconhecer esses conceitos e sintaxe. Use este capítulo como referência para guiá-lo às soluções.

A seguir, colocaremos tudo o que discutimos ao longo do livro em prática e faremos mais um projeto!
