## Workspaces do Cargo

No Capítulo 12, construímos um pacote que incluía um crate binário e um crate de biblioteca. À medida que seu projeto se desenvolve, você pode descobrir que o crate de biblioteca continua a crescer e você deseja dividir seu pacote ainda mais em vários crates de biblioteca. O Cargo oferece um recurso chamado *workspaces* (espaços de trabalho) que pode ajudar a gerenciar vários pacotes relacionados que são desenvolvidos em conjunto.

### Criando um Workspace

Um *workspace* é um conjunto de pacotes que compartilham o mesmo *Cargo.lock* e diretório de saída. Vamos fazer um projeto usando um workspace—usaremos código trivial para que possamos nos concentrar na estrutura do workspace. Existem várias maneiras de estruturar um workspace, então mostraremos apenas uma maneira comum. Teremos um workspace contendo um binário e duas bibliotecas. O binário, que fornecerá a funcionalidade principal, dependerá das duas bibliotecas. Uma biblioteca fornecerá uma função `add_one` e a outra biblioteca uma função `add_two`. Esses três crates farão parte do mesmo workspace. Começaremos criando um novo diretório para o workspace:

```console
$ mkdir add
$ cd add
```

Em seguida, no diretório *add*, criamos o arquivo *Cargo.toml* que configurará todo o workspace. Este arquivo não terá uma seção `[package]`. Em vez disso, ele começará com uma seção `[workspace]` que nos permitirá adicionar membros ao workspace. Também fazemos questão de usar a melhor e mais recente versão do algoritmo resolvedor do Cargo em nosso workspace, definindo o valor `resolver` como `"3"`:

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Em seguida, criaremos o crate binário `adder` executando `cargo new` dentro do diretório *add*:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Executar `cargo new` dentro de um workspace também adiciona automaticamente o pacote recém-criado à chave `members` na definição `[workspace]` no *Cargo.toml* do workspace, assim:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Neste ponto, podemos construir o workspace executando `cargo build`. Os arquivos em seu diretório *add* devem ficar assim:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

O workspace tem um diretório *target* no nível superior onde os artefatos compilados serão colocados; o pacote `adder` não tem seu próprio diretório *target*. Mesmo se estivéssemos executando `cargo build` de dentro do diretório *adder*, os artefatos compilados ainda acabariam em *add/target* em vez de *add/adder/target*. O Cargo estrutura o diretório *target* em um workspace dessa maneira porque os crates em um workspace devem depender uns dos outros. Se cada crate tivesse seu próprio diretório *target*, cada crate teria que recompilar cada um dos outros crates no workspace para colocar os artefatos em seu próprio diretório *target*. Ao compartilhar um diretório *target*, os crates podem evitar a reconstrução desnecessária.

### Criando o Segundo Pacote no Workspace

Em seguida, vamos criar outro pacote membro no workspace e chamá-lo de `add_one`. Gere um novo crate de biblioteca chamado `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

O *Cargo.toml* de nível superior agora incluirá o caminho *add_one* na lista `members`:

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Seu diretório *add* agora deve ter esses diretórios e arquivos:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

No arquivo *add_one/src/lib.rs*, vamos adicionar uma função `add_one`:

<span class="filename">Nome do arquivo: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Agora podemos fazer com que o pacote `adder` com nosso binário dependa do pacote `add_one` que tem nossa biblioteca. Primeiro, precisaremos adicionar uma dependência de caminho em `add_one` ao *adder/Cargo.toml*.

<span class="filename">Nome do arquivo: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

O Cargo não assume que os crates em um workspace dependerão uns dos outros, então precisamos ser explícitos sobre os relacionamentos de dependência.

Em seguida, vamos usar a função `add_one` (do crate `add_one`) no crate `adder`. Abra o arquivo *adder/src/main.rs* e altere a função `main` para chamar a função `add_one`, como na Listagem 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Usando o crate de biblioteca `add_one` a partir do crate `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Vamos construir o workspace executando `cargo build` no diretório *add* de nível superior!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

Para executar o crate binário do diretório *add*, podemos especificar qual pacote no workspace queremos executar usando o argumento `-p` e o nome do pacote com `cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Isso executa o código em *adder/src/main.rs*, que depende do crate `add_one`.

<!-- Old headings. Do not remove or links may break. -->

<a id="depending-on-an-external-package-in-a-workspace"></a>

### Dependendo de um Pacote Externo

Observe que o workspace tem apenas um arquivo *Cargo.lock* no nível superior, em vez de ter um *Cargo.lock* no diretório de cada crate. Isso garante que todos os crates estejam usando a mesma versão de todas as dependências. Se adicionarmos o pacote `rand` aos arquivos *adder/Cargo.toml* e *add_one/Cargo.toml*, o Cargo resolverá ambos para uma versão de `rand` e registrará isso no único *Cargo.lock*. Fazer com que todos os crates no workspace usem as mesmas dependências significa que os crates sempre serão compatíveis entre si. Vamos adicionar o crate `rand` à seção `[dependencies]` no arquivo *add_one/Cargo.toml* para que possamos usar o crate `rand` no crate `add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Nome do arquivo: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Agora podemos adicionar `use rand;` ao arquivo *add_one/src/lib.rs*, e construir todo o workspace executando `cargo build` no diretório *add* trará e compilará o crate `rand`. Receberemos um aviso porque não estamos nos referindo ao `rand` que trouxemos para o escopo:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

O *Cargo.lock* de nível superior agora contém informações sobre a dependência de `add_one` em `rand`. No entanto, mesmo que `rand` seja usado em algum lugar no workspace, não podemos usá-lo em outros crates no workspace, a menos que adicionemos `rand` aos seus arquivos *Cargo.toml* também. Por exemplo, se adicionarmos `use rand;` ao arquivo *adder/src/main.rs* para o pacote `adder`, obteremos um erro:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Para corrigir isso, edite o arquivo *Cargo.toml* para o pacote `adder` e indique que `rand` é uma dependência para ele também. Construir o pacote `adder` adicionará `rand` à lista de dependências para `adder` no *Cargo.lock*, mas nenhuma cópia adicional de `rand` será baixada. O Cargo garantirá que cada crate em cada pacote no workspace usando o pacote `rand` usará a mesma versão, desde que especifiquem versões compatíveis de `rand`, economizando espaço e garantindo que os crates no workspace sejam compatíveis entre si.

Se os crates no workspace especificarem versões incompatíveis da mesma dependência, o Cargo resolverá cada uma delas, mas ainda tentará resolver o menor número possível de versões.

### Adicionando um Teste a um Workspace

Para um aprimoramento adicional, vamos adicionar um teste da função `add_one::add_one` dentro do crate `add_one`:

<span class="filename">Nome do arquivo: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Agora execute `cargo test` no diretório *add* de nível superior. Executar `cargo test` em um workspace estruturado como este executará os testes para todos os crates no workspace:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

A primeira seção da saída mostra que o teste `it_works` no crate `add_one` passou. A próxima seção mostra que zero testes foram encontrados no crate `adder` e, em seguida, a última seção mostra que zero testes de documentação foram encontrados no crate `add_one`.

Também podemos executar testes para um crate específico em um workspace do diretório de nível superior usando o sinalizador `-p` e especificando o nome do crate que queremos testar:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Esta saída mostra que `cargo test` executou apenas os testes para o crate `add_one` e não executou os testes do crate `adder`.

Se você publicar os crates no workspace em [crates.io](https://crates.io/)<!-- ignore -->, cada crate no workspace precisará ser publicado separadamente. Assim como `cargo test`, podemos publicar um crate específico em nosso workspace usando o sinalizador `-p` e especificando o nome do crate que queremos publicar.

Para prática adicional, adicione um crate `add_two` a este workspace de maneira semelhante ao crate `add_one`!

À medida que seu projeto cresce, considere usar um workspace: ele permite que você trabalhe com componentes menores e mais fáceis de entender do que um grande bloco de código. Além disso, manter os crates em um workspace pode facilitar a coordenação entre os crates se eles forem alterados frequentemente ao mesmo tempo.
