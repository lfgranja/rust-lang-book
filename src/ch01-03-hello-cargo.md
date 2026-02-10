# Hello, Cargo!

Cargo é o sistema de build e gerenciador de pacotes do Rust. A maioria dos Rustaceans usa esta ferramenta
para gerenciar seus projetos Rust porque o Cargo lida com muitas tarefas para você,
como compilar seu código, baixar as bibliotecas das quais seu código depende, e
compilar essas bibliotecas. (Chamamos as bibliotecas que seu código precisa de
_dependências_.)

Os programas Rust mais simples, como o que escrevemos até agora, não têm nenhuma
dependência. Se tivéssemos construído o projeto “Hello, world!” com Cargo, ele
apenas usaria a parte do Cargo que lida com a compilação do seu código. À medida que você escreve
programas Rust mais complexos, você adicionará dependências, e se você começar um projeto
usando Cargo, adicionar dependências será muito mais fácil de fazer.

Como a vasta maioria dos projetos Rust usa Cargo, o restante deste livro
assume que você está usando Cargo também. O Cargo vem instalado com o Rust se você
usou os instaladores oficiais discutidos na seção de
[[ch01-01-installation.md#installation|“Instalação”]]<!-- ignore -->. Se você instalou o Rust
através de algum outro meio, verifique se o Cargo está instalado digitando o
seguinte no seu terminal:

```console
$ cargo --version
```

Se você vir um número de versão, você o tem! Se você vir um erro, como `command
not found`, olhe a documentação para seu método de instalação para
determinar como instalar o Cargo separadamente.

### Criando um Projeto com Cargo

Vamos criar um novo projeto usando Cargo e ver como ele difere do nosso
projeto “Hello, world!” original. Navegue de volta para o seu diretório _projects_
(ou onde quer que você tenha decidido armazenar seu código). Então, em qualquer sistema operacional,
execute o seguinte:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

O primeiro comando cria um novo diretório e projeto chamado _hello_cargo_.
Nós nomeamos nosso projeto _hello_cargo_, e o Cargo cria seus arquivos em um
diretório de mesmo nome.

Entre no diretório _hello_cargo_ e liste os arquivos. Você verá que o Cargo
gerou dois arquivos e um diretório para nós: um arquivo _Cargo.toml_ e um
diretório _src_ com um arquivo _main.rs_ dentro.

Ele também inicializou um novo repositório Git junto com um arquivo _.gitignore_.
Arquivos Git não serão gerados se você executar `cargo new` dentro de um repositório Git
existente; você pode sobrescrever esse comportamento usando `cargo new --vcs=git`.

> Nota: Git é um sistema de controle de versão comum. Você pode mudar o `cargo new` para
> usar um sistema de controle de versão diferente ou nenhum sistema de controle de versão usando
> a flag `--vcs`. Execute `cargo new --help` para ver as opções disponíveis.

Abra _Cargo.toml_ no seu editor de texto de escolha. Ele deve parecer similar ao
código na Listagem 1-2.

<Listing number="1-2" file-name="Cargo.toml" caption="Conteúdo do *Cargo.toml* gerado pelo `cargo new`">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Este arquivo está no formato [_TOML_](https://toml.io)<!-- ignore --> (_Tom’s Obvious, Minimal
Language_), que é o formato de configuração do Cargo.

A primeira linha, `[package]`, é um cabeçalho de seção que indica que as
declarações a seguir estão configurando um pacote. À medida que adicionamos mais informações a
este arquivo, adicionaremos outras seções.

As próximas três linhas definem as informações de configuração que o Cargo precisa para compilar
seu programa: o nome, a versão e a edição do Rust a ser usada. Falaremos
sobre a chave `edition` no [[appendix-05-editions.md|Apêndice E]]<!-- ignore -->.

A última linha, `[dependencies]`, é o início de uma seção para você listar qualquer
uma das dependências do seu projeto. Em Rust, pacotes de código são referidos como
_crates_. Não precisaremos de nenhum outro crate para este projeto, mas precisaremos no
primeiro projeto no Capítulo 2, então usaremos esta seção de dependências lá.

Agora abra _src/main.rs_ e dê uma olhada:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

O Cargo gerou um programa “Hello, world!” para você, exatamente como o que
escrevemos na Listagem 1-1! Até agora, as diferenças entre nosso projeto e o
projeto que o Cargo gerou são que o Cargo colocou o código no diretório _src_,
e temos um arquivo de configuração _Cargo.toml_ no diretório superior.

O Cargo espera que seus arquivos fonte vivam dentro do diretório _src_. O
diretório de nível superior do projeto é apenas para arquivos README, informações de licença,
arquivos de configuração e qualquer outra coisa não relacionada ao seu código. Usar o Cargo
ajuda você a organizar seus projetos. Há um lugar para tudo, e
tudo está em seu lugar.

Se você começou um projeto que não usa Cargo, como fizemos com o projeto “Hello,
world!”, você pode convertê-lo para um projeto que usa Cargo. Mova o
código do projeto para o diretório _src_ e crie um arquivo _Cargo.toml_
apropriado. Uma maneira fácil de obter esse arquivo _Cargo.toml_ é executar `cargo init`, que
o criará para você automaticamente.

### Compilando e Executando um Projeto Cargo

Agora vamos ver o que é diferente quando compilamos e executamos o programa “Hello, world!”
com Cargo! Do seu diretório _hello_cargo_, compile seu projeto
digitando o seguinte comando:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Este comando cria um arquivo executável em _target/debug/hello_cargo_ (ou
_target\debug\hello_cargo.exe_ no Windows) em vez de no seu diretório atual.
Como o build padrão é um build de debug (depuração), o Cargo coloca o binário em
um diretório chamado _debug_. Você pode executar o executável com este comando:

```console
$ ./target/debug/hello_cargo # ou .\target\debug\hello_cargo.exe no Windows
Hello, world!
```

Se tudo correr bem, `Hello, world!` deve ser impresso no terminal. Executar `cargo
build` pela primeira vez também faz com que o Cargo crie um novo arquivo no nível
superior: _Cargo.lock_. Este arquivo mantém o registro das versões exatas das
dependências no seu projeto. Este projeto não tem dependências, então o
arquivo é um pouco vazio. Você nunca precisará alterar este arquivo manualmente; o Cargo
gerencia seu conteúdo para você.

Nós acabamos de compilar um projeto com `cargo build` e o executamos com
`./target/debug/hello_cargo`, mas também podemos usar `cargo run` para compilar o
código e então executar o executável resultante tudo em um comando:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Usar `cargo run` é mais conveniente do que ter que lembrar de executar `cargo
build` e então usar o caminho completo para o binário, então a maioria dos desenvolvedores usa `cargo
run`.

Note que desta vez não vimos saída indicando que o Cargo estava compilando
`hello_cargo`. O Cargo descobriu que os arquivos não tinham mudado, então ele não
recompilou, apenas executou o binário. Se você tivesse modificado seu código fonte, o Cargo
teria recompilado o projeto antes de executá-lo, e você teria visto esta
saída:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

O Cargo também fornece um comando chamado `cargo check`. Este comando verifica rapidamente
seu código para garantir que ele compila, mas não produz um executável:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Por que você não iria querer um executável? Frequentemente, `cargo check` é muito mais rápido que
`cargo build` porque ele pula o passo de produzir um executável. Se você está
continuamente verificando seu trabalho enquanto escreve o código, usar `cargo check` vai
acelerar o processo de deixar você saber se seu projeto ainda está compilando! Como
tal, muitos Rustaceans executam `cargo check` periodicamente enquanto escrevem seu
programa para garantir que ele compila. Então, eles executam `cargo build` quando estão
prontos para usar o executável.

Vamos recapitular o que aprendemos até agora sobre o Cargo:

- Podemos criar um projeto usando `cargo new`.
- Podemos compilar um projeto usando `cargo build`.
- Podemos compilar e executar um projeto em um passo usando `cargo run`.
- Podemos compilar um projeto sem produzir um binário para verificar erros usando
  `cargo check`.
- Em vez de salvar o resultado do build no mesmo diretório que nosso código,
  o Cargo o armazena no diretório _target/debug_.

Uma vantagem adicional de usar o Cargo é que os comandos são os mesmos não
importa em qual sistema operacional você está trabalhando. Então, neste ponto, não
forneceremos mais instruções específicas para Linux e macOS versus Windows.

### Compilando para Release

Quando seu projeto estiver finalmente pronto para lançamento, você pode usar `cargo build
--release` para compilá-lo com otimizações. Este comando criará um
executável em _target/release_ em vez de _target/debug_. As otimizações
fazem seu código Rust rodar mais rápido, mas ligá-las aumenta o tempo que leva
para seu programa compilar. É por isso que existem dois perfis diferentes: um
para desenvolvimento, quando você quer recompilar rapidamente e com frequência, e outro para
compilar o programa final que você dará a um usuário, que não será recompilado
repetidamente e que rodará o mais rápido possível. Se você está fazendo benchmarking do
tempo de execução do seu código, certifique-se de executar `cargo build --release` e fazer o benchmark com
o executável em _target/release_.

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### Aproveitando as Convenções do Cargo

Com projetos simples, o Cargo não fornece muito valor além de apenas usar
`rustc`, mas ele provará seu valor à medida que seus programas se tornarem mais complexos.
Uma vez que os programas cresçam para múltiplos arquivos ou precisem de uma dependência, é muito mais fácil
deixar o Cargo coordenar o build.

Mesmo que o projeto `hello_cargo` seja simples, ele agora usa muito do ferramental
real que você usará no resto da sua carreira Rust. De fato, para trabalhar em qualquer
projeto existente, você pode usar os seguintes comandos para baixar o código
usando Git, mudar para o diretório desse projeto e compilar:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Para mais informações sobre o Cargo, confira [sua documentação](https://doc.rust-lang.org/cargo/).

## Resumo

Você já está com um ótimo começo na sua jornada Rust! Neste capítulo, você
aprendeu como:

- Instalar a versão estável mais recente do Rust usando `rustup`.
- Atualizar para uma versão mais nova do Rust.
- Abrir a documentação instalada localmente.
- Escrever e executar um programa “Hello, world!” usando `rustc` diretamente.
- Criar e executar um novo projeto usando as convenções do Cargo.

Este é um ótimo momento para construir um programa mais substancial para se acostumar a ler
e escrever código Rust. Então, no Capítulo 2, construiremos um programa de jogo de adivinhação.
Se você preferir começar aprendendo como conceitos comuns de programação funcionam em
Rust, veja o Capítulo 3 e depois retorne ao Capítulo 2.
