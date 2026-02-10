# Apêndice D: Ferramentas Úteis de Desenvolvimento

Neste apêndice, falamos sobre algumas ferramentas de desenvolvimento úteis que o projeto Rust fornece além do compilador, Rustfmt e Clippy. Já abordamos os métodos para usar essas ferramentas no Capítulo 1.

## Formatação Automática com `rustfmt`

A ferramenta `rustfmt` reformata seu código de acordo com o estilo de código da comunidade. Muitos projetos colaborativos usam `rustfmt` para evitar discussões sobre qual estilo usar ao escrever Rust: todos formatam seu código usando a ferramenta.

Para instalar `rustfmt`, digite o seguinte:

```console
$ rustup component add rustfmt
```

Este comando fornece `rustfmt` e `cargo-fmt`, semelhantes a como Rust fornece `rustc` e `cargo`. Para formatar qualquer projeto Cargo, digite o seguinte:

```console
$ cargo fmt
```

Executar este comando formata todo o código Rust na crate atual. Isso deve apenas alterar o estilo do código, não a semântica do código. Para mais informações sobre `rustfmt`, consulte [sua documentação](https://github.com/rust-lang/rustfmt).

## Corrigir Seu Código com `rustfix`

A ferramenta `rustfix` está incluída nas instalações do Rust e pode corrigir automaticamente avisos do compilador que têm uma maneira clara de corrigir o problema que é provável que seja o que você deseja. É provável que você já tenha visto avisos do compilador antes. Por exemplo, considere este código:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for i in 0..100 {
        do_something();
    }
}
```

Aqui, estamos chamando a função `do_something` 100 vezes, mas nunca usamos a variável `i` no corpo do loop `for`. Rust nos adverte sobre isso:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: unused variable: `i`
 --> src/main.rs:4:9
  |
4 |     for i in 0..100 {
  |         ^ help: consider using `_i` instead
  |
  = note: #[warn(unused_variables)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
```

O aviso sugere que usemos `_i` como um nome em vez disso: o sublinhado indica que pretendemos que esta variável não seja usada. Podemos aplicar essa sugestão automaticamente usando a ferramenta `rustfix` executando o comando `cargo fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Quando olhamos para *src/main.rs* novamente, veremos que `cargo fix` mudou o código:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for _i in 0..100 {
        do_something();
    }
}
```

A variável do loop `for` agora é chamada de `_i`, e o aviso desapareceu.

Você também pode usar o comando `cargo fix` para fazer a transição do seu código entre diferentes edições do Rust. As edições são abordadas no Apêndice E.

## Mais Lints com `clippy`

A ferramenta `clippy` é uma coleção de lints para analisar seu código, para que você possa capturar erros comuns e melhorar seu código Rust.

Para instalar `clippy`, digite o seguinte:

```console
$ rustup component add clippy
```

Para executar lints do `clippy` em qualquer projeto Cargo, digite o seguinte:

```console
$ cargo clippy
```

Por exemplo, digamos que você escreva um programa que usa uma aproximação de uma constante matemática, como pi, como este:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("a área do círculo é {}", x * r * r);
}
```

Executar `cargo clippy` neste projeto resulta neste erro:

```console
error: approximate value of `f{32,64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: #[deny(clippy::approx_constant)] on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Este erro informa que Rust já tem uma constante `PI` mais precisa definida e que seu programa estaria mais correto se você usasse a constante em vez disso. Você alteraria então seu código para usar a constante `PI`. O código a seguir não resulta em erros ou avisos do `clippy`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("a área do círculo é {}", x * r * r);
}
```

Para mais informações sobre `clippy`, consulte [sua documentação](https://github.com/rust-lang/rust-clippy).

## Integração com IDE usando `rust-analyzer`

Para ajudar na integração com IDE, a comunidade Rust recomenda usar [`rust-analyzer`](https://rust-analyzer.github.io). Esta ferramenta é um conjunto de utilitários centrados em compiladores que fala o [Language Server Protocol](https://microsoft.github.io/language-server-protocol/), que é uma especificação para IDEs e editores de código se comunicarem com compiladores. Diferentes clientes podem usar `rust-analyzer`, como [o plug-in Rust para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer).

Visite a [página inicial](https://rust-analyzer.github.io) do projeto `rust-analyzer` para obter instruções de instalação e consulte o suporte ao Language Server Protocol do seu IDE específico para instalar o suporte para Rust.
