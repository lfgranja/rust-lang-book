## Erros Recuperáveis com `Result`

A maioria dos erros não é grave o suficiente para exigir que o programa pare
totalmente. Às vezes, quando uma função falha, é por um motivo que você pode
interpretar e responder facilmente. Por exemplo, se você tentar abrir um arquivo
e essa operação falhar porque o arquivo não existe, você pode querer criar o
arquivo em vez de encerrar o processo.

Lembre-se de [“Lidando com Falhas Potenciais com `Result`”][handle_failure]<!--
ignore --> no Capítulo 2 que a enumeração `Result` é definida como tendo duas
variantes, `Ok` e `Err`, da seguinte forma:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

O `T` e o `E` são parâmetros de tipo genérico: discutiremos genéricos com mais
detalhes no Capítulo 10. O que você precisa saber agora é que `T` representa o
tipo do valor que será retornado em um caso de sucesso dentro da variante `Ok`,
e `E` representa o tipo do erro que será retornado em um caso de falha dentro da
variante `Err`. Como `Result` tem esses parâmetros de tipo genérico, podemos
usar o tipo `Result` e as funções definidas nele em muitas situações diferentes
onde o valor de sucesso e o valor de erro que queremos retornar podem diferir.

Vamos chamar uma função que retorna um valor `Result` porque a função pode
falhar. Na Listagem 9-3, tentamos abrir um arquivo.

<Listing number="9-3" file-name="src/main.rs" caption="Abrindo um arquivo">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

O tipo de retorno de `File::open` é um `Result<T, E>`. O parâmetro genérico `T`
foi preenchido pela implementação de `File::open` com o tipo do valor de
sucesso, `std::fs::File`, que é um manipulador de arquivo. O tipo de `E` usado
no valor de erro é `std::io::Error`. Este tipo de retorno significa que a
chamada para `File::open` pode ter sucesso e retornar um manipulador de arquivo
que podemos ler ou escrever. A chamada de função também pode falhar: Por
exemplo, o arquivo pode não existir ou podemos não ter permissão para acessar o
arquivo. A função `File::open` precisa ter uma maneira de nos dizer se teve
sucesso ou falhou e, ao mesmo tempo, nos dar o manipulador de arquivo ou
informações de erro. Esta informação é exatamente o que a enumeração `Result`
transmite.

No caso em que `File::open` tiver sucesso, o valor na variável
`greeting_file_result` será uma instância de `Ok` que contém um manipulador de
arquivo. No caso de falha, o valor em `greeting_file_result` será uma
instância de `Err` que contém mais informações sobre o tipo de erro que ocorreu.

Precisamos adicionar ao código na Listagem 9-3 para tomar ações diferentes
dependendo do valor que `File::open` retorna. A Listagem 9-4 mostra uma maneira
de lidar com o `Result` usando uma ferramenta básica, a expressão `match` que
discutimos no Capítulo 6.

<Listing number="9-4" file-name="src/main.rs" caption="Usando uma expressão `match` para lidar com as variantes de `Result` que podem ser retornadas">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Observe que, como a enumeração `Option`, a enumeração `Result` e suas variantes
foram trazidas para o escopo pelo prelúdio, então não precisamos especificar
`Result::` antes das variantes `Ok` e `Err` nos braços do `match`.

Quando o resultado é `Ok`, este código retornará o valor interno `file` da
variante `Ok`, e então atribuímos esse valor de manipulador de arquivo à
variável `greeting_file`. Após o `match`, podemos usar o manipulador de arquivo
para leitura ou escrita.

O outro braço do `match` lida com o caso em que recebemos um valor `Err` de
`File::open`. Neste exemplo, escolhemos chamar a macro `panic!`. Se não houver
nenhum arquivo chamado _hello.txt_ em nosso diretório atual e executarmos este
código, veremos a seguinte saída da macro `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Como de costume, esta saída nos diz exatamente o que deu errado.

### Correspondendo a Diferentes Erros

O código na Listagem 9-4 entrará em `panic!` não importa por que `File::open`
falhou. No entanto, queremos tomar ações diferentes para diferentes razões de
falha. Se `File::open` falhou porque o arquivo não existe, queremos criar o
arquivo e retornar o manipulador para o novo arquivo. Se `File::open` falhou
por qualquer outro motivo — por exemplo, porque não tínhamos permissão para
abrir o arquivo — ainda queremos que o código entre em `panic!` da mesma forma
que fez na Listagem 9-4. Para isso, adicionamos uma expressão `match` interna,
mostrada na Listagem 9-5.

<Listing number="9-5" file-name="src/main.rs" caption="Lidando com diferentes tipos de erros de maneiras diferentes">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

O tipo do valor que `File::open` retorna dentro da variante `Err` é
`io::Error`, que é uma struct fornecida pela biblioteca padrão. Esta struct tem
um método, `kind`, que podemos chamar para obter um valor `io::ErrorKind`. A
enumeração `io::ErrorKind` é fornecida pela biblioteca padrão e tem variantes
representando os diferentes tipos de erros que podem resultar de uma operação
`io`. A variante que queremos usar é `ErrorKind::NotFound`, que indica que o
arquivo que estamos tentando abrir ainda não existe. Então, fazemos a
correspondência em `greeting_file_result`, mas também temos uma correspondência
interna em `error.kind()`.

A condição que queremos verificar na correspondência interna é se o valor
retornado por `error.kind()` é a variante `NotFound` da enumeração `ErrorKind`.
Se for, tentamos criar o arquivo com `File::create`. No entanto, como
`File::create` também pode falhar, precisamos de um segundo braço na expressão
`match` interna. Quando o arquivo não pode ser criado, uma mensagem de erro
diferente é impressa. O segundo braço do `match` externo permanece o mesmo,
então o programa entra em pânico em qualquer erro além do erro de arquivo
ausente.

> #### Alternativas ao Uso de `match` com `Result<T, E>`
>
> Isso é muito `match`! A expressão `match` é muito útil, mas também é muito
> primitiva. No Capítulo 13, você aprenderá sobre closures, que são usadas com
> muitos dos métodos definidos em `Result<T, E>`. Esses métodos podem ser mais
> concisos do que usar `match` ao lidar com valores `Result<T, E>` em seu
> código.
>
> Por exemplo, aqui está outra maneira de escrever a mesma lógica mostrada na
> Listagem 9-5, desta vez usando closures e o método `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problema ao criar o arquivo: {error:?}");
>             })
>         } else {
>             panic!("Problema ao abrir o arquivo: {error:?}");
>         }
>     });
> }
> ```
>
> Embora este código tenha o mesmo comportamento que a Listagem 9-5, ele não
> contém nenhuma expressão `match` e é mais limpo de ler. Volte a este exemplo
> depois de ler o Capítulo 13 e procure o método `unwrap_or_else` na
> documentação da biblioteca padrão. Muitos outros métodos podem limpar
> enormes expressões `match` aninhadas quando você está lidando com erros.

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### Atalhos para Pânico em Erro: `unwrap` e `expect`

Usar `match` funciona bem o suficiente, mas pode ser um pouco verboso e nem
sempre comunica bem a intenção. O tipo `Result<T, E>` tem muitos métodos
auxiliares definidos nele para fazer várias tarefas mais específicas. O método
`unwrap` é um método de atalho implementado exatamente como a expressão `match`
que escrevemos na Listagem 9-4. Se o valor `Result` for a variante `Ok`,
`unwrap` retornará o valor dentro do `Ok`. Se o `Result` for a variante `Err`,
`unwrap` chamará a macro `panic!` para nós. Aqui está um exemplo de `unwrap` em
ação:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Se executarmos este código sem um arquivo _hello.txt_, veremos uma mensagem de
erro da chamada `panic!` que o método `unwrap` faz:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Da mesma forma, o método `expect` nos permite também escolher a mensagem de erro
`panic!`. Usar `expect` em vez de `unwrap` e fornecer boas mensagens de erro
pode transmitir sua intenção e tornar o rastreamento da origem de um pânico
mais fácil. A sintaxe de `expect` se parece com isto:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

Usamos `expect` da mesma maneira que `unwrap`: para retornar o manipulador de
arquivo ou chamar a macro `panic!`. A mensagem de erro usada por `expect` em
sua chamada para `panic!` será o parâmetro que passamos para `expect`, em vez
da mensagem padrão de `panic!` que `unwrap` usa. Eis como fica:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Em código de qualidade de produção, a maioria dos Rustaceans escolhe `expect`
em vez de `unwrap` e fornece mais contexto sobre por que a operação deve sempre
ter sucesso. Dessa forma, se suas suposições forem provadas erradas, você terá
mais informações para usar na depuração.

### Propagando Erros

Quando a implementação de uma função chama algo que pode falhar, em vez de
lidar com o erro dentro da própria função, você pode retornar o erro para o
código chamador para que ele possa decidir o que fazer. Isso é conhecido como
_propagar_ o erro e dá mais controle ao código chamador, onde pode haver mais
informações ou lógica que dita como o erro deve ser tratado do que o que você
tem disponível no contexto do seu código.

Por exemplo, a Listagem 9-6 mostra uma função que lê um nome de usuário de um
arquivo. Se o arquivo não existir ou não puder ser lido, esta função retornará
esses erros para o código que chamou a função.

<Listing number="9-6" file-name="src/main.rs" caption="Uma função que retorna erros para o código chamador usando `match`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Esta função pode ser escrita de uma forma muito mais curta, mas vamos começar
fazendo muito disso manualmente para explorar o tratamento de erros; no final,
mostraremos o caminho mais curto. Vamos olhar para o tipo de retorno da função
primeiro: `Result<String, io::Error>`. Isso significa que a função está
retornando um valor do tipo `Result<T, E>`, onde o parâmetro genérico `T` foi
preenchido com o tipo concreto `String` e o tipo genérico `E` foi preenchido
com o tipo concreto `io::Error`.

Se esta função tiver sucesso sem problemas, o código que chama esta função
receberá um valor `Ok` que contém uma `String` — o `username` que esta função
leu do arquivo. Se esta função encontrar algum problema, o código chamador
receberá um valor `Err` que contém uma instância de `io::Error` que contém mais
informações sobre quais foram os problemas. Escolhemos `io::Error` como o tipo
de retorno desta função porque acontece de ser o tipo do valor de erro retornado
de ambas as operações que estamos chamando no corpo desta função que podem
falhar: a função `File::open` e o método `read_to_string`.

O corpo da função começa chamando a função `File::open`. Então, lidamos com o
valor `Result` com um `match` semelhante ao `match` na Listagem 9-4. Se
`File::open` tiver sucesso, o manipulador de arquivo na variável de padrão
`file` torna-se o valor na variável mutável `username_file` e a função
continua. No caso `Err`, em vez de chamar `panic!`, usamos a palavra-chave
`return` para retornar cedo da função inteiramente e passar o valor de erro de
`File::open`, agora na variável de padrão `e`, de volta para o código chamador
como o valor de erro desta função.

Então, se tivermos um manipulador de arquivo em `username_file`, a função cria
uma nova `String` na variável `username` e chama o método `read_to_string` no
manipulador de arquivo em `username_file` para ler o conteúdo do arquivo para
`username`. O método `read_to_string` também retorna um `Result` porque pode
falhar, mesmo que `File::open` tenha tido sucesso. Então, precisamos de outro
`match` para lidar com esse `Result`: Se `read_to_string` tiver sucesso, então
nossa função teve sucesso, e retornamos o nome de usuário do arquivo que agora
está em `username` envolvido em um `Ok`. Se `read_to_string` falhar, retornamos
o valor de erro da mesma maneira que retornamos o valor de erro no `match` que
lidou com o valor de retorno de `File::open`. No entanto, não precisamos dizer
explicitamente `return`, porque esta é a última expressão na função.

O código que chama este código lidará então com a obtenção de um valor `Ok` que
contém um nome de usuário ou um valor `Err` que contém um `io::Error`. Cabe ao
código chamador decidir o que fazer com esses valores. Se o código chamador
receber um valor `Err`, ele pode chamar `panic!` e travar o programa, usar um
nome de usuário padrão ou procurar o nome de usuário de outro lugar que não seja
um arquivo, por exemplo. Não temos informações suficientes sobre o que o código
chamador está realmente tentando fazer, então propagamos todas as informações de
sucesso ou erro para cima para que ele trate adequadamente.

Esse padrão de propagação de erros é tão comum em Rust que Rust fornece o
operador de ponto de interrogação `?` para tornar isso mais fácil.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### Um Atalho para Propagar Erros: o Operador `?`

A Listagem 9-7 mostra uma implementação de `read_username_from_file` que tem a
mesma funcionalidade que na Listagem 9-6, mas esta implementação usa o operador
`?`.

<Listing number="9-7" file-name="src/main.rs" caption="Uma função que retorna erros para o código chamador usando o operador `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

O `?` colocado após um valor `Result` é definido para funcionar quase da mesma
maneira que as expressões `match` que definimos para lidar com os valores
`Result` na Listagem 9-6. Se o valor do `Result` for um `Ok`, o valor dentro do
`Ok` será retornado desta expressão, e o programa continuará. Se o valor for um
`Err`, o `Err` será retornado de toda a função como se tivéssemos usado a
palavra-chave `return` para que o valor de erro seja propagado para o código
chamador.

Há uma diferença entre o que a expressão `match` da Listagem 9-6 faz e o que o
operador `?` faz: Valores de erro que têm o operador `?` chamado neles passam
pela função `from`, definida na trait `From` na biblioteca padrão, que é usada
para converter valores de um tipo em outro. Quando o operador `?` chama a
função `from`, o tipo de erro recebido é convertido no tipo de erro definido no
tipo de retorno da função atual. Isso é útil quando uma função retorna um tipo
de erro para representar todas as maneiras como uma função pode falhar, mesmo
que partes possam falhar por muitos motivos diferentes.

Por exemplo, poderíamos alterar a função `read_username_from_file` na Listagem
9-7 para retornar um tipo de erro personalizado chamado `OurError` que
definimos. Se também definirmos `impl From<io::Error> for OurError` para
construir uma instância de `OurError` a partir de um `io::Error`, então as
chamadas do operador `?` no corpo de `read_username_from_file` chamarão `from`
e converterão os tipos de erro sem a necessidade de adicionar mais código à
função.

No contexto da Listagem 9-7, o `?` no final da chamada `File::open` retornará o
valor dentro de um `Ok` para a variável `username_file`. Se ocorrer um erro, o
operador `?` retornará cedo de toda a função e dará qualquer valor `Err` ao
código chamador. A mesma coisa se aplica ao `?` no final da chamada
`read_to_string`.

O operador `?` elimina muito código repetitivo e torna a implementação desta
função mais simples. Poderíamos até encurtar este código ainda mais
encadeando chamadas de método imediatamente após o `?`, como mostrado na
Listagem 9-8.

<Listing number="9-8" file-name="src/main.rs" caption="Encadeando chamadas de método após o operador `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

Movemos a criação da nova `String` em `username` para o início da função; essa
parte não mudou. Em vez de criar uma variável `username_file`, encadeamos a
chamada para `read_to_string` diretamente no resultado de
`File::open("hello.txt")?`. Ainda temos um `?` no final da chamada
`read_to_string`, e ainda retornamos um valor `Ok` contendo `username` quando
tanto `File::open` quanto `read_to_string` têm sucesso em vez de retornar
erros. A funcionalidade é novamente a mesma que na Listagem 9-6 e na Listagem
9-7; esta é apenas uma maneira diferente e mais ergonômica de escrevê-la.

A Listagem 9-9 mostra uma maneira de tornar isso ainda mais curto usando
`fs::read_to_string`.

<Listing number="9-9" file-name="src/main.rs" caption="Usando `fs::read_to_string` em vez de abrir e depois ler o arquivo">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Ler um arquivo em uma string é uma operação bastante comum, então a biblioteca
padrão fornece a função conveniente `fs::read_to_string` que abre o arquivo,
cria uma nova `String`, lê o conteúdo do arquivo, coloca o conteúdo nessa
`String` e o retorna. Claro, usar `fs::read_to_string` não nos dá a
oportunidade de explicar todo o tratamento de erros, então fizemos da maneira
mais longa primeiro.

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### Onde Usar o Operador `?`

O operador `?` só pode ser usado em funções cujo tipo de retorno seja compatível
com o valor em que o `?` é usado. Isso ocorre porque o operador `?` é definido
para realizar um retorno antecipado de um valor da função, da mesma maneira
que a expressão `match` que definimos na Listagem 9-6. Na Listagem 9-6, o
`match` estava usando um valor `Result`, e o braço de retorno antecipado
retornou um valor `Err(e)`. O tipo de retorno da função tem que ser um `Result`
para que seja compatível com este `return`.

Na Listagem 9-10, vamos ver o erro que obteremos se usarmos o operador `?` em
uma função `main` com um tipo de retorno incompatível com o tipo do valor em
que usamos `?`.

<Listing number="9-10" file-name="src/main.rs" caption="Tentar usar o `?` na função `main` que retorna `()` não compilará.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Este código abre um arquivo, o que pode falhar. O operador `?` segue o valor
`Result` retornado por `File::open`, mas esta função `main` tem o tipo de
retorno `()`, não `Result`. Quando compilamos este código, obtemos a seguinte
mensagem de erro:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Este erro aponta que só podemos usar o operador `?` em uma função que retorna
`Result`, `Option`, ou outro tipo que implementa `FromResidual`.

Para corrigir o erro, você tem duas escolhas. Uma escolha é alterar o tipo de
retorno da sua função para ser compatível com o valor em que você está usando o
operador `?`, desde que você não tenha restrições impedindo isso. A outra
escolha é usar um `match` ou um dos métodos de `Result<T, E>` para lidar com o
`Result<T, E>` de qualquer maneira que seja apropriada.

A mensagem de erro também mencionou que `?` pode ser usado com valores
`Option<T>` também. Assim como o uso de `?` em `Result`, você só pode usar `?`
em `Option` em uma função que retorna um `Option`. O comportamento do operador
`?` quando chamado em um `Option<T>` é semelhante ao seu comportamento quando
chamado em um `Result<T, E>`: Se o valor for `None`, o `None` será retornado
antecipadamente da função naquele ponto. Se o valor for `Some`, o valor dentro
do `Some` é o valor resultante da expressão, e a função continua. A Listagem
9-11 tem um exemplo de uma função que encontra o último caractere da primeira
linha no texto fornecido.

<Listing number="9-11" caption="Usando o operador `?` em um valor `Option<T>`">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Esta função retorna `Option<char>` porque é possível que haja um caractere lá,
mas também é possível que não haja. Este código recebe o argumento de fatia de
string `text` e chama o método `lines` nele, que retorna um iterador sobre as
linhas na string. Como esta função quer examinar a primeira linha, ela chama
`next` no iterador para obter o primeiro valor do iterador. Se `text` for a
string vazia, esta chamada para `next` retornará `None`, caso em que usamos `?`
para parar e retornar `None` de `last_char_of_first_line`. Se `text` não for a
string vazia, `next` retornará um valor `Some` contendo uma fatia de string da
primeira linha em `text`.

O `?` extrai a fatia de string, e podemos chamar `chars` nessa fatia de string
para obter um iterador de seus caracteres. Estamos interessados no último
caractere nesta primeira linha, então chamamos `last` para retornar o último
item no iterador. Este é um `Option` porque é possível que a primeira linha
seja a string vazia; por exemplo, se `text` começar com uma linha em branco,
mas tiver caracteres em outras linhas, como em `"\nhi"`. No entanto, se houver
um último caractere na primeira linha, ele será retornado na variante `Some`. O
operador `?` no meio nos dá uma maneira concisa de expressar essa lógica,
permitindo-nos implementar a função em uma linha. Se não pudéssemos usar o
operador `?` em `Option`, teríamos que implementar essa lógica usando mais
chamadas de método ou uma expressão `match`.

Observe que você pode usar o operador `?` em um `Result` em uma função que
retorna `Result`, e você pode usar o operador `?` em um `Option` em uma função
que retorna `Option`, mas você não pode misturar e combinar. O operador `?` não
converterá automaticamente um `Result` em um `Option` ou vice-versa; nesses
casos, você pode usar métodos como o método `ok` em `Result` ou o método
`ok_or` em `Option` para fazer a conversão explicitamente.

Até agora, todas as funções `main` que usamos retornam `()`. A função `main` é
especial porque é o ponto de entrada e saída de um programa executável, e há
restrições sobre qual deve ser seu tipo de retorno para que o programa se
comporte conforme o esperado.

Felizmente, `main` também pode retornar um `Result<(), E>`. A Listagem 9-12 tem
o código da Listagem 9-10, mas mudamos o tipo de retorno de `main` para ser
`Result<(), Box<dyn Error>>` e adicionamos um valor de retorno `Ok(())` ao
final. Este código agora compilará.

<Listing number="9-12" file-name="src/main.rs" caption="Mudar `main` para retornar `Result<(), E>` permite o uso do operador `?` em valores `Result`.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

O tipo `Box<dyn Error>` é um objeto de trait, sobre o qual falaremos em
[“Usando Objetos de Trait para Abstrair sobre Comportamento Compartilhado”][trait-objects]<!-- ignore -->
no Capítulo 18. Por enquanto, você pode ler `Box<dyn Error>` como significando
“qualquer tipo de erro”. Usar `?` em um valor `Result` em uma função `main` com
o tipo de erro `Box<dyn Error>` é permitido porque permite que qualquer valor
`Err` seja retornado antecipadamente. Mesmo que o corpo desta função `main`
retorne apenas erros do tipo `std::io::Error`, ao especificar `Box<dyn Error>`,
esta assinatura continuará correta mesmo se mais código que retorna outros erros
for adicionado ao corpo de `main`.

Quando uma função `main` retorna um `Result<(), E>`, o executável sairá com um
valor de `0` se `main` retornar `Ok(())` e sairá com um valor diferente de zero
se `main` retornar um valor `Err`. Executáveis escritos em C retornam inteiros
quando saem: Programas que saem com sucesso retornam o inteiro `0`, e programas
que erram retornam algum inteiro diferente de `0`. Rust também retorna inteiros
de executáveis para ser compatível com essa convenção.

A função `main` pode retornar quaisquer tipos que implementem [a trait
`std::process::Termination`][termination]<!-- ignore -->, que contém uma função
`report` que retorna um `ExitCode`. Consulte a documentação da biblioteca
padrão para obter mais informações sobre a implementação da trait `Termination`
para seus próprios tipos.

Agora que discutimos os detalhes de chamar `panic!` ou retornar `Result`, vamos
voltar ao tópico de como decidir qual é o apropriado para usar em quais casos.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html
