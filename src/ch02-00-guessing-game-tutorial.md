# Programando um Jogo de Adivinhação

Vamos mergulhar no Rust trabalhando juntos em um projeto prático! Este
capítulo apresenta a você alguns conceitos comuns do Rust mostrando como usá-los
em um programa real. Você aprenderá sobre `let`, `match`, métodos, funções
associadas, crates externos e muito mais! Nos capítulos seguintes, exploraremos
essas ideias em mais detalhes. Neste capítulo, você apenas praticará os
fundamentos.

Implementaremos um problema clássico de programação para iniciantes: um jogo de
adivinhação. Veja como funciona: O programa gerará um número inteiro aleatório
entre 1 e 100. Em seguida, ele solicitará que o jogador insira um palpite.
Depois que um palpite for inserido, o programa indicará se o palpite é muito
baixo ou muito alto. Se o palpite estiver correto, o jogo imprimirá uma mensagem
de parabenização e sairá.

## Configurando um Novo Projeto

Para configurar um novo projeto, vá para o diretório _projects_ que você criou
no [[ch01-00-getting-started.md|Capítulo 1]]<!-- ignore --> e crie um novo
projeto usando o Cargo, assim:

```console
$ cargo new guessing_game
$ cd guessing_game
```

O primeiro comando, `cargo new`, recebe o nome do projeto (`guessing_game`)
como o primeiro argumento. O segundo comando muda para o diretório do novo
projeto.

Olhe para o arquivo _Cargo.toml_ gerado:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Como você viu no [[ch01-00-getting-started.md|Capítulo 1]]<!-- ignore -->,
`cargo new` gera um programa “Hello, world!” para você. Confira o arquivo
_src/main.rs_:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Agora vamos compilar este programa “Hello, world!” e executá-lo no mesmo passo
usando o comando `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

O comando `run` é útil quando você precisa iterar rapidamente em um projeto,
como faremos neste jogo, testando rapidamente cada iteração antes de passar para
a próxima.

Reabra o arquivo _src/main.rs_. Você escreverá todo o código neste arquivo.

## Processando um Palpite

A primeira parte do programa do jogo de adivinhação pedirá a entrada do usuário,
processará essa entrada e verificará se a entrada está no formato esperado.
Para começar, permitiremos que o jogador insira um palpite. Digite o código da
Listagem 2-1 em _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Código que obtém um palpite do usuário e o imprime">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Este código contém muita informação, então vamos analisá-lo linha por linha.
Para obter a entrada do usuário e depois imprimir o resultado como saída,
precisamos trazer a biblioteca de entrada/saída `io` para o escopo. A
biblioteca `io` vem da biblioteca padrão, conhecida como `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Por padrão, o Rust tem um conjunto de itens definidos na biblioteca padrão que
ele traz para o escopo de cada programa. Esse conjunto é chamado de _prelúdio_,
e você pode ver tudo nele [na documentação da biblioteca padrão][prelude].

Se um tipo que você deseja usar não estiver no prelúdio, você deve trazer esse
tipo para o escopo explicitamente com uma instrução `use`. Usar a biblioteca
`std::io` fornece a você uma série de recursos úteis, incluindo a capacidade de
aceitar entrada do usuário.

Como você viu no [[ch01-00-getting-started.md|Capítulo 1]]<!-- ignore -->, a
função `main` é o ponto de entrada no programa:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

A sintaxe `fn` declara uma nova função; os parênteses, `()`, indicam que não há
parâmetros; e a chave, `{`, inicia o corpo da função.

Como você também aprendeu no [[ch01-00-getting-started.md|Capítulo 1]]<!--
ignore -->, `println!` é uma macro que imprime uma string na tela:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Este código está imprimindo um prompt informando o que é o jogo e solicitando
entrada do usuário.

### Armazenando Valores com Variáveis

Em seguida, criaremos uma _variável_ para armazenar a entrada do usuário, assim:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Agora o programa está ficando interessante! Há muita coisa acontecendo nesta
pequena linha. Usamos a instrução `let` para criar a variável. Aqui está outro
exemplo:

```rust,ignore
let apples = 5;
```

Esta linha cria uma nova variável chamada `apples` e a vincula ao valor `5`. Em
Rust, as variáveis são imutáveis por padrão, o que significa que, uma vez que
damos um valor à variável, o valor não mudará. Discutiremos esse conceito em
detalhes na seção
[[ch03-01-variables-and-mutability.md#variables-and-mutability|“Variáveis e Mutabilidade”]]<!-- ignore -->
no Capítulo 3. Para tornar uma variável mutável, adicionamos `mut` antes do
nome da variável:

```rust,ignore
let apples = 5; // imutável
let mut bananas = 5; // mutável
```

> Nota: A sintaxe `//` inicia um comentário que continua até o final da linha.
> O Rust ignora tudo nos comentários. Discutiremos comentários em mais detalhes
> no [[ch03-04-comments.md|Capítulo 3]]<!-- ignore -->.

Voltando ao programa do jogo de adivinhação, agora você sabe que `let mut guess`
introduzirá uma variável mutável chamada `guess`. O sinal de igual (`=`) diz
ao Rust que queremos vincular algo à variável agora. À direita do sinal de
igual está o valor ao qual `guess` está vinculado, que é o resultado de chamar
`String::new`, uma função que retorna uma nova instância de uma `String`.
[`String`][string]<!-- ignore --> é um tipo de string fornecido pela
biblioteca padrão que é um pedaço de texto expansível codificado em UTF-8.

A sintaxe `::` na linha `::new` indica que `new` é uma função associada do tipo
`String`. Uma _função associada_ é uma função implementada em um tipo, neste
caso `String`. Esta função `new` cria uma nova string vazia. Você encontrará
uma função `new` em muitos tipos, porque é um nome comum para uma função que
cria um novo valor de algum tipo.

No total, a linha `let mut guess = String::new();` criou uma variável mutável
que está atualmente vinculada a uma nova instância vazia de uma `String`. Ufa!

### Recebendo Entrada do Usuário

Lembre-se de que incluímos a funcionalidade de entrada/saída da biblioteca
padrão com `use std::io;` na primeira linha do programa. Agora chamaremos a
função `stdin` do módulo `io`, que nos permitirá lidar com a entrada do
usuário:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Se não tivéssemos importado o módulo `io` com `use std::io;` no início do
programa, ainda poderíamos usar a função escrevendo esta chamada de função como
`std::io::stdin`. A função `stdin` retorna uma instância de
[`std::io::Stdin`][iostdin]<!-- ignore -->, que é um tipo que representa um
identificador para a entrada padrão do seu terminal.

Em seguida, a linha `.read_line(&mut guess)` chama o método
[`read_line`][read_line]<!-- ignore --> no identificador de entrada padrão para
obter entrada do usuário. Também estamos passando `&mut guess` como argumento
para `read_line` para dizer a ele em qual string armazenar a entrada do
usuário. O trabalho completo de `read_line` é pegar o que o usuário digita na
entrada padrão e anexar isso em uma string (sem sobrescrever seu conteúdo),
então passamos essa string como argumento. O argumento string precisa ser
mutável para que o método possa alterar o conteúdo da string.

O `&` indica que este argumento é uma _referência_, que fornece uma maneira de
permitir que várias partes do seu código acessem um pedaço de dados sem
precisar copiar esses dados na memória várias vezes. Referências são um recurso
complexo, e uma das maiores vantagens do Rust é o quão seguro e fácil é usar
referências. Você não precisa saber muitos desses detalhes para terminar este
programa. Por enquanto, tudo o que você precisa saber é que, como as variáveis,
as referências são imutáveis por padrão. Portanto, você precisa escrever `&mut
guess` em vez de `&guess` para torná-la mutável. (O Capítulo 4 explicará as
referências mais detalhadamente.)

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Lidando com Potencial Falha com o Tipo `Result`

Ainda estamos trabalhando nesta linha de código. Estamos discutindo agora uma
terceira linha de texto, mas note que ainda faz parte de uma única linha lógica
de código. A próxima parte é este método:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Poderíamos ter escrito este código como:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Falha ao ler linha");
```

No entanto, uma linha longa é difícil de ler, então é melhor dividi-la. É
frequentemente sábio introduzir uma nova linha e outros espaços em branco para
ajudar a quebrar linhas longas quando você chama um método com a sintaxe
`.nome_do_metodo()`. Agora vamos discutir o que esta linha faz.

Como mencionado anteriormente, `read_line` coloca o que o usuário insere na
string que passamos para ele, mas também retorna um valor `Result`.
[`Result`][result]<!-- ignore --> é uma [_enumeração_][enums]<!-- ignore -->,
muitas vezes chamada de _enum_, que é um tipo que pode estar em um de vários
estados possíveis. Chamamos cada estado possível de _variante_.

O [[ch06-00-enums.md|Capítulo 6]]<!-- ignore --> cobrirá enums em mais
detalhes. O objetivo desses tipos `Result` é codificar informações de tratamento
de erros.

As variantes de `Result` são `Ok` e `Err`. A variante `Ok` indica que a
operação foi bem-sucedida e contém o valor gerado com sucesso. A variante `Err`
significa que a operação falhou e contém informações sobre como ou por que a
operação falhou.

Valores do tipo `Result`, como valores de qualquer tipo, têm métodos definidos
neles. Uma instância de `Result` tem um [método `expect`][expect]<!-- ignore
--> que você pode chamar. Se esta instância de `Result` for um valor `Err`,
`expect` fará com que o programa falhe e exiba a mensagem que você passou como
argumento para `expect`. Se o método `read_line` retornar um `Err`,
provavelmente seria o resultado de um erro vindo do sistema operacional
subjacente. Se esta instância de `Result` for um valor `Ok`, `expect` pegará o
valor de retorno que `Ok` está segurando e retornará apenas esse valor para
você para que possa usá-lo. Neste caso, esse valor é o número de bytes na
entrada do usuário.

Se você não chamar `expect`, o programa compilará, mas você receberá um aviso:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

O Rust avisa que você não usou o valor `Result` retornado por `read_line`,
indicando que o programa não tratou um possível erro.

A maneira correta de suprimir o aviso é realmente escrever código de tratamento
de erros, mas em nosso caso queremos apenas travar este programa quando ocorrer
um problema, então podemos usar `expect`. Você aprenderá sobre como se
recuperar de erros no [[ch09-02-recoverable-errors-with-result.md|Capítulo 9]]<!-- ignore -->.

### Imprimindo Valores com Espaços Reservados `println!`

Além da chave de fechamento, há apenas mais uma linha para discutir no código
até agora:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Esta linha imprime a string que agora contém a entrada do usuário. O conjunto de
chaves `{}` é um espaço reservado: Pense em `{}` como pequenas pinças de
caranguejo que seguram um valor no lugar. Ao imprimir o valor de uma variável,
o nome da variável pode ir dentro das chaves. Ao imprimir o resultado da
avaliação de uma expressão, coloque chaves vazias na string de formatação,
seguidas pela string de formatação com uma lista separada por vírgulas de
expressões para imprimir em cada espaço reservado de chave vazia na mesma ordem.
Imprimir uma variável e o resultado de uma expressão em uma chamada para
`println!` ficaria assim:

```rust
let x = 5;
let y = 10;

println!("x = {x} e y + 2 = {}", y + 2);
```

Este código imprimiria `x = 5 e y + 2 = 12`.

### Testando a Primeira Parte

Vamos testar a primeira parte do jogo de adivinhação. Execute-o usando
`cargo run`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Adivinhe o número!
Por favor, insira o seu palpite.
6
Você adivinhou: 6
```

Neste ponto, a primeira parte do jogo está pronta: estamos obtendo entrada do
teclado e depois imprimindo-a.

## Gerando um Número Secreto

Em seguida, precisamos gerar um número secreto que o usuário tentará adivinhar.
O número secreto deve ser diferente a cada vez para que o jogo seja divertido de
jogar mais de uma vez. Usaremos um número aleatório entre 1 e 100 para que o
jogo não seja muito difícil. O Rust ainda não inclui funcionalidade de números
aleatórios em sua biblioteca padrão. No entanto, a equipe Rust fornece um
[`rand` crate][randcrate] com tal funcionalidade.

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### Aumentando a Funcionalidade com um Crate

Lembre-se de que um crate é uma coleção de arquivos de código-fonte Rust. O
projeto que estamos construindo é um crate binário, que é um executável. O crate
`rand` é um crate de biblioteca, que contém código destinado a ser usado em
outros programas e não pode ser executado por conta própria.

A coordenação de crates externos pelo Cargo é onde o Cargo realmente brilha.
Antes de podermos escrever código que usa `rand`, precisamos modificar o arquivo
_Cargo.toml_ para incluir o crate `rand` como uma dependência. Abra esse
arquivo agora e adicione a seguinte linha na parte inferior, abaixo do
cabeçalho da seção `[dependencies]` que o Cargo criou para você. Certifique-se
de especificar `rand` exatamente como temos aqui, com este número de versão, ou
os exemplos de código neste tutorial podem não funcionar:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

No arquivo _Cargo.toml_, tudo o que segue um cabeçalho faz parte dessa seção
que continua até que outra seção comece. Em `[dependencies]`, você diz ao Cargo
de quais crates externos seu projeto depende e quais versões desses crates você
precisa. Neste caso, especificamos o crate `rand` com o especificador de versão
semântica `0.8.5`. O Cargo entende o [Versionamento Semântico][semver]<!--
ignore --> (às vezes chamado de _SemVer_), que é um padrão para escrever
números de versão. O especificador `0.8.5` é na verdade uma abreviação para
`^0.8.5`, o que significa qualquer versão que seja pelo menos 0.8.5, mas abaixo
de 0.9.0.

O Cargo considera essas versões como tendo APIs públicas compatíveis com a
versão 0.8.5, e essa especificação garante que você obterá a versão de patch
mais recente que ainda compilará com o código neste capítulo. Qualquer versão
0.9.0 ou superior não é garantida de ter a mesma API que os exemplos a seguir
usam.

Agora, sem alterar nenhum código, vamos construir o projeto, conforme mostrado
na Listagem 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="A saída da execução de `cargo build` após adicionar o crate `rand` como uma dependência">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

Você pode ver números de versão diferentes (mas todos serão compatíveis com o
código, graças ao SemVer!) e linhas diferentes (dependendo do sistema
operacional), e as linhas podem estar em uma ordem diferente.

Quando incluímos uma dependência externa, o Cargo busca as versões mais
recentes de tudo o que essa dependência precisa do _registro_, que é uma cópia
dos dados do [Crates.io][cratesio]. Crates.io é onde as pessoas no ecossistema
Rust postam seus projetos Rust de código aberto para que outros os usem.

Depois de atualizar o registro, o Cargo verifica a seção `[dependencies]` e
baixa quaisquer crates listados que ainda não foram baixados. Neste caso, embora
tenhamos listado apenas `rand` como dependência, o Cargo também pegou outros
crates dos quais `rand` depende para funcionar. Depois de baixar os crates, o
Rust os compila e, em seguida, compila o projeto com as dependências
disponíveis.

Se você executar imediatamente `cargo build` novamente sem fazer nenhuma
alteração, não receberá nenhuma saída além da linha `Finished`. O Cargo sabe
que já baixou e compilou as dependências e você não alterou nada sobre elas em
seu arquivo _Cargo.toml_. O Cargo também sabe que você não alterou nada sobre
seu código, então ele também não recompila isso. Sem nada para fazer, ele
simplesmente sai.

Se você abrir o arquivo _src/main.rs_, fizer uma alteração trivial e, em
seguida, salvar e compilar novamente, verá apenas duas linhas de saída:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Essas linhas mostram que o Cargo atualiza apenas o build com sua pequena
alteração no arquivo _src/main.rs_. Suas dependências não mudaram, então o
Cargo sabe que pode reutilizar o que já baixou e compilou para elas.

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### Garantindo Builds Reprodutíveis com o Arquivo _Cargo.lock_

O Cargo tem um mecanismo que garante que você possa reconstruir o mesmo artefato
toda vez que você ou qualquer outra pessoa construir seu código: o Cargo usará
apenas as versões das dependências que você especificou até que você indique o
contrário. Por exemplo, digamos que na próxima semana saia a versão 0.8.6 do
crate `rand` e que essa versão contenha uma correção de bug importante, mas
também contenha uma regressão que quebrará seu código. Para lidar com isso, o
Rust cria o arquivo _Cargo.lock_ na primeira vez que você executa `cargo
build`, então agora temos isso no diretório _guessing_game_.

Quando você constrói um projeto pela primeira vez, o Cargo descobre todas as
versões das dependências que se encaixam nos critérios e as grava no arquivo
_Cargo.lock_. Quando você construir seu projeto no futuro, o Cargo verá que o
arquivo _Cargo.lock_ existe e usará as versões especificadas lá, em vez de
fazer todo o trabalho de descobrir versões novamente. Isso permite que você
tenha um build reprodutível automaticamente. Em outras palavras, seu projeto
permanecerá em 0.8.5 até que você atualize explicitamente, graças ao arquivo
_Cargo.lock_. Como o arquivo _Cargo.lock_ é importante para builds
reprodutíveis, ele geralmente é verificado no controle de versão com o restante
do código em seu projeto.

#### Atualizando um Crate para Obter uma Nova Versão

Quando você _realmente_ quer atualizar um crate, o Cargo fornece o comando
`update`, que ignorará o arquivo _Cargo.lock_ e descobrirá todas as versões
mais recentes que se encaixam em suas especificações no _Cargo.toml_. O Cargo
então gravará essas versões no arquivo _Cargo.lock_. Caso contrário, por
padrão, o Cargo procurará apenas versões maiores que 0.8.5 e menores que 0.9.0.
Se o crate `rand` tiver lançado as duas novas versões 0.8.6 e 0.999.0, você
veria o seguinte se executasse `cargo update`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.999.0)
```

O Cargo ignora a versão 0.999.0. Neste ponto, você também notaria uma alteração
em seu arquivo _Cargo.lock_ observando que a versão do crate `rand` que você
está usando agora é 0.8.6. Para usar a versão `rand` 0.999.0 ou qualquer versão
na série 0.999._x_, você teria que atualizar o arquivo _Cargo.toml_ para ficar
assim (não faça essa alteração na verdade, porque os exemplos a seguir assumem
que você está usando `rand` 0.8):

```toml
[dependencies]
rand = "0.999.0"
```

Na próxima vez que você executar `cargo build`, o Cargo atualizará o registro de
crates disponíveis e reavaliará seus requisitos de `rand` de acordo com a nova
versão que você especificou.

Há muito mais a dizer sobre o [Cargo][doccargo]<!-- ignore --> e [seu
ecossistema][doccratesio]<!-- ignore -->, que discutiremos no Capítulo 14, mas
por enquanto, isso é tudo o que você precisa saber. O Cargo torna muito fácil
reutilizar bibliotecas, então os Rustaceans são capazes de escrever projetos
menores que são montados a partir de vários pacotes.

### Gerando um Número Aleatório

Vamos começar a usar `rand` para gerar um número para adivinhar. O próximo
passo é atualizar _src/main.rs_, conforme mostrado na Listagem 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Adicionando código para gerar um número aleatório">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Primeiro, adicionamos a linha `use rand::Rng;`. O trait `Rng` define métodos que
os geradores de números aleatórios implementam, e esse trait deve estar no
escopo para usarmos esses métodos. O Capítulo 10 cobrirá traits em detalhes.

Em seguida, estamos adicionando duas linhas no meio. Na primeira linha, chamamos
a função `rand::thread_rng` que nos dá o gerador de números aleatórios
particular que vamos usar: um que é local para a thread de execução atual e é
semeado pelo sistema operacional. Em seguida, chamamos o método `gen_range` no
gerador de números aleatórios. Este método é definido pelo trait `Rng` que
trouxemos para o escopo com a instrução `use rand::Rng;`. O método `gen_range`
recebe uma expressão de intervalo como argumento e gera um número aleatório no
intervalo. O tipo de expressão de intervalo que estamos usando aqui assume a
forma `start..=end` e é inclusivo nos limites inferior e superior, então
precisamos especificar `1..=100` para solicitar um número entre 1 e 100.

> Nota: Você não saberá apenas quais traits usar e quais métodos e funções
> chamar de um crate, então cada crate tem documentação com instruções para
> usá-lo. Outro recurso interessante do Cargo é que executar o comando `cargo
> doc --open` construirá a documentação fornecida por todas as suas
> dependências localmente e a abrirá em seu navegador. Se você estiver
> interessado em outra funcionalidade no crate `rand`, por exemplo, execute
> `cargo doc --open` e clique em `rand` na barra lateral à esquerda.

A segunda nova linha imprime o número secreto. Isso é útil enquanto estamos
desenvolvendo o programa para poder testá-lo, mas vamos excluí-lo da versão
final. Não é muito um jogo se o programa imprime a resposta assim que começa!

Tente executar o programa algumas vezes:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Você deve obter números aleatórios diferentes, e todos devem ser números entre 1
e 100. Bom trabalho!

## Comparando o Palpite com o Número Secreto

Agora que temos a entrada do usuário e um número aleatório, podemos compará-los.
Esse passo é mostrado na Listagem 2-4. Observe que este código não compilará
apenas ainda, como explicaremos.

<Listing number="2-4" file-name="src/main.rs" caption="Lidando com os possíveis valores de retorno da comparação de dois números">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Primeiro, adicionamos outra instrução `use`, trazendo um tipo chamado
`std::cmp::Ordering` para o escopo da biblioteca padrão. O tipo `Ordering` é
outro enum e tem as variantes `Less`, `Greater` e `Equal`. Esses são os três
resultados possíveis quando você compara dois valores.

Em seguida, adicionamos cinco novas linhas na parte inferior que usam o tipo
`Ordering`. O método `cmp` compara dois valores e pode ser chamado em qualquer
coisa que possa ser comparada. Ele recebe uma referência a qualquer coisa com a
qual você queira comparar: Aqui, está comparando `guess` com `secret_number`.
Em seguida, ele retorna uma variante do enum `Ordering` que trouxemos para o
escopo com a instrução `use`. Usamos uma expressão [`match`][match]<!-- ignore
--> para decidir o que fazer em seguida com base em qual variante de `Ordering`
foi retornada da chamada para `cmp` com os valores em `guess` e
`secret_number`.

Uma expressão `match` é composta por _braços_. Um braço consiste em um _padrão_
para corresponder e o código que deve ser executado se o valor dado a `match`
se encaixar no padrão desse braço. O Rust pega o valor dado a `match` e olha
através do padrão de cada braço por vez. Padrões e a construção `match` são
recursos poderosos do Rust: Eles permitem que você expresse uma variedade de
situações que seu código pode encontrar e garantem que você lide com todas
elas. Esses recursos serão cobertos em detalhes no Capítulo 6 e Capítulo 19,
respectivamente.

Vamos percorrer um exemplo com a expressão `match` que usamos aqui. Digamos que
o usuário adivinhou 50 e o número secreto gerado aleatoriamente desta vez é 38.

Quando o código compara 50 a 38, o método `cmp` retornará `Ordering::Greater`
porque 50 é maior que 38. A expressão `match` obtém o valor
`Ordering::Greater` e começa a verificar o padrão de cada braço. Ele olha para
o padrão do primeiro braço, `Ordering::Less`, e vê que o valor
`Ordering::Greater` não corresponde a `Ordering::Less`, então ignora o código
nesse braço e passa para o próximo braço. O padrão do próximo braço é
`Ordering::Greater`, que _corresponde_ a `Ordering::Greater`! O código
associado nesse braço será executado e imprimirá `Muito alto!` na tela. A
expressão `match` termina após a primeira correspondência bem-sucedida, então
não olhará para o último braço neste cenário.

No entanto, o código na Listagem 2-4 não compilará ainda. Vamos tentar:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

O núcleo do erro afirma que existem _tipos incompatíveis_. O Rust tem um
sistema de tipos forte e estático. No entanto, ele também tem inferência de
tipos. Quando escrevemos `let mut guess = String::new()`, o Rust foi capaz de
inferir que `guess` deveria ser uma `String` e não nos fez escrever o tipo. O
`secret_number`, por outro lado, é um tipo numérico. Alguns dos tipos
numéricos do Rust podem ter um valor entre 1 e 100: `i32`, um número de 32
bits; `u32`, um número de 32 bits sem sinal; `i64`, um número de 64 bits; bem
como outros. A menos que especificado de outra forma, o Rust assume como padrão
um `i32`, que é o tipo de `secret_number` a menos que você adicione informações
de tipo em outro lugar que façam o Rust inferir um tipo numérico diferente. A
razão para o erro é que o Rust não pode comparar uma string e um tipo numérico.

Por fim, queremos converter a `String` que o programa lê como entrada em um
tipo numérico para que possamos compará-lo numericamente com o número secreto.
Fazemos isso adicionando esta linha ao corpo da função `main`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

A linha é:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Por favor, digite um número!");
```

Criamos uma variável chamada `guess`. Mas espere, o programa já não tem uma
variável chamada `guess`? Tem, mas felizmente o Rust nos permite sombrear o
valor anterior de `guess` com um novo. _Shadowing_ (sombreamento) nos permite
reutilizar o nome da variável `guess` em vez de nos forçar a criar duas
variáveis únicas, como `guess_str` e `guess`, por exemplo. Cobriremos isso em
mais detalhes no
[[ch03-01-variables-and-mutability.md#shadowing|Capítulo 3]]<!-- ignore -->,
mas por enquanto, saiba que esse recurso é frequentemente usado quando você
deseja converter um valor de um tipo para outro tipo.

Vinculamos essa nova variável à expressão `guess.trim().parse()`. O `guess` na
expressão refere-se à variável `guess` original que continha a entrada como uma
string. O método `trim` em uma instância de `String` eliminará qualquer espaço
em branco no início e no fim, o que devemos fazer antes de podermos converter a
string para um `u32`, que só pode conter dados numéricos. O usuário deve
pressionar <kbd>enter</kbd> para satisfazer `read_line` e inserir seu palpite,
o que adiciona um caractere de nova linha à string. Por exemplo, se o usuário
digitar <kbd>5</kbd> e pressionar <kbd>enter</kbd>, `guess` se parece com
isso: `5\n`. O `\n` representa “nova linha”. (No Windows, pressionar
<kbd>enter</kbd> resulta em um retorno de carro e uma nova linha, `\r\n`.) O
método `trim` elimina `\n` ou `\r\n`, resultando em apenas `5`.

O [método `parse` em strings][parse]<!-- ignore --> converte uma string em
outro tipo. Aqui, nós o usamos para converter de uma string para um número.
Precisamos dizer ao Rust o tipo exato de número que queremos usando `let guess:
u32`. Os dois pontos (`:`) após `guess` dizem ao Rust que anotaremos o tipo da
variável. O Rust tem alguns tipos de números integrados; o `u32` visto aqui é
um inteiro sem sinal de 32 bits. É uma boa escolha padrão para um pequeno
número positivo. Você aprenderá sobre outros tipos de números no
[[ch03-02-data-types.md#integer-types|Capítulo 3]]<!-- ignore -->.

Além disso, a anotação `u32` neste programa de exemplo e a comparação com
`secret_number` significam que o Rust inferirá que `secret_number` deve ser um
`u32` também. Então, agora a comparação será entre dois valores do mesmo tipo!

O método `parse` funcionará apenas em caracteres que podem ser logicamente
convertidos em números e, portanto, pode facilmente causar erros. Se, por
exemplo, a string contivesse `A👍%`, não haveria maneira de converter isso em
um número. Como pode falhar, o método `parse` retorna um tipo `Result`, muito
parecido com o método `read_line` (discutido anteriormente em
[“Lidando com Potencial Falha com o Tipo `Result`”](#handling-potential-failure-with-the-result-type)<!-- ignore -->).
Trataremos esse `Result` da mesma maneira usando o método `expect` novamente.
Se `parse` retornar uma variante `Err` de `Result` porque não conseguiu criar
um número a partir da string, a chamada `expect` travará o jogo e imprimirá a
mensagem que fornecemos. Se `parse` puder converter com sucesso a string em um
número, ele retornará a variante `Ok` de `Result`, e `expect` retornará o
número que queremos do valor `Ok`.

Vamos executar o programa agora:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Legal! Mesmo que espaços tenham sido adicionados antes do palpite, o programa
ainda descobriu que o usuário adivinhou 76. Execute o programa algumas vezes
para verificar o comportamento diferente com diferentes tipos de entrada:
Adivinhe o número corretamente, adivinhe um número muito alto e adivinhe um
número muito baixo.

Temos a maior parte do jogo funcionando agora, mas o usuário pode fazer apenas
um palpite. Vamos mudar isso adicionando um loop!

## Permitindo Múltiplos Palpites com Looping

A palavra-chave `loop` cria um loop infinito. Adicionaremos um loop para dar aos
usuários mais chances de adivinhar o número:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Como você pode ver, movemos tudo do prompt de entrada de palpite em diante para
um loop. Certifique-se de recuar as linhas dentro do loop mais quatro espaços
cada e execute o programa novamente. O programa agora pedirá outro palpite para
sempre, o que na verdade introduz um novo problema. Não parece que o usuário
possa sair!

O usuário sempre pode interromper o programa usando o atalho de teclado
<kbd>ctrl</kbd>-<kbd>C</kbd>. Mas há outra maneira de escapar deste monstro
insaciável, como mencionado na discussão de `parse` em
[“Comparando o Palpite com o Número Secreto”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->:
Se o usuário inserir uma resposta não numérica, o programa travará. Podemos
aproveitar isso para permitir que o usuário saia, conforme mostrado aqui:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Digitar `quit` encerrará o jogo, mas como você notará, inserir qualquer outra
entrada não numérica também encerrará. Isso é subótimo, para dizer o mínimo;
queremos que o jogo também pare quando o número correto for adivinhado.

### Saindo Após um Palpite Correto

Vamos programar o jogo para sair quando o usuário ganhar adicionando uma
instrução `break`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Adicionar a linha `break` após `Você ganhou!` faz com que o programa saia do
loop quando o usuário adivinha o número secreto corretamente. Sair do loop
também significa sair do programa, porque o loop é a última parte de `main`.

### Lidando com Entrada Inválida

Para refinar ainda mais o comportamento do jogo, em vez de travar o programa
quando o usuário insere um não-número, vamos fazer o jogo ignorar um não-número
para que o usuário possa continuar adivinhando. Podemos fazer isso alterando a
linha onde `guess` é convertido de uma `String` para um `u32`, conforme
mostrado na Listagem 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Ignorando um palpite não numérico e pedindo outro palpite em vez de travar o programa">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Mudamos de uma chamada `expect` para uma expressão `match` para passar de travar
em um erro para lidar com o erro. Lembre-se de que `parse` retorna um tipo
`Result` e `Result` é um enum que tem as variantes `Ok` e `Err`. Estamos usando
uma expressão `match` aqui, como fizemos com o resultado `Ordering` do método
`cmp`.

Se `parse` conseguir transformar com sucesso a string em um número, ele
retornará um valor `Ok` contendo o número resultante. Esse valor `Ok`
corresponderá ao padrão do primeiro braço, e a expressão `match` retornará
apenas o valor `num` que `parse` produziu e colocou dentro do valor `Ok`. Esse
número acabará exatamente onde o queremos na nova variável `guess` que estamos
criando.

Se `parse` _não_ conseguir transformar a string em um número, ele retornará um
valor `Err` contendo mais informações sobre o erro. O valor `Err` não
corresponde ao padrão `Ok(num)` no primeiro braço do `match`, mas corresponde
ao padrão `Err(_)` no segundo braço. O sublinhado, `_`, é um valor catch-all
(pega-tudo); neste exemplo, estamos dizendo que queremos corresponder a todos
os valores `Err`, não importa quais informações eles tenham dentro deles.
Portanto, o programa executará o código do segundo braço, `continue`, que diz
ao programa para ir para a próxima iteração do `loop` e pedir outro palpite.
Assim, efetivamente, o programa ignora todos os erros que `parse` pode
encontrar!

Agora tudo no programa deve funcionar conforme o esperado. Vamos tentar:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Incrível! Com um pequeno ajuste final, terminaremos o jogo de adivinhação.
Lembre-se de que o programa ainda está imprimindo o número secreto. Isso
funcionou bem para testes, mas estraga o jogo. Vamos excluir o `println!` que
gera o número secreto. A Listagem 2-6 mostra o código final.

<Listing number="2-6" file-name="src/main.rs" caption="Código completo do jogo de adivinhação">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Neste ponto, você construiu com sucesso o jogo de adivinhação. Parabéns!

## Resumo

Este projeto foi uma maneira prática de apresentar a você muitos novos conceitos
do Rust: `let`, `match`, funções, o uso de crates externos e muito mais. Nos
próximos capítulos, você aprenderá sobre esses conceitos em mais detalhes. O
Capítulo 3 cobre conceitos que a maioria das linguagens de programação possui,
como variáveis, tipos de dados e funções, e mostra como usá-los no Rust. O
Capítulo 4 explora a propriedade (ownership), um recurso que torna o Rust
diferente de outras linguagens. O Capítulo 5 discute structs e sintaxe de
métodos, e o Capítulo 6 explica como os enums funcionam.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
