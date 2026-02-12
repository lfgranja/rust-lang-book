## Pacotes e Crates

As primeiras partes do sistema de módulos que cobriremos são pacotes e crates.

Um *crate* é a menor quantidade de código que o compilador Rust considera de uma
vez. Mesmo se você executar `rustc` em vez de `cargo` e passar um único arquivo de código fonte
(como fizemos lá atrás em ["Noções Básicas de Programação em Rust"][basics]<!-- ignore
--> no Capítulo 1), o compilador considera esse arquivo como um crate. Crates podem
conter módulos, e os módulos podem ser definidos em outros arquivos que são
compilados com o crate, como veremos nas próximas seções.

Um crate pode vir em duas formas: um crate binário ou um crate de biblioteca.
*Crates binários* são programas que você pode compilar em um executável que você pode executar,
como um programa de linha de comando ou um servidor. Cada um deve ter uma função chamada
`main` que define o que acontece quando o executável roda. Todos os crates que
criamos até agora foram crates binários.

*Crates de biblioteca* não têm uma função `main`, e eles não compilam para um
executável. Em vez disso, eles definem funcionalidades destinadas a serem compartilhadas com
vários projetos. Por exemplo, o crate `rand` que usamos no [Capítulo
2][rand]<!-- ignore --> fornece funcionalidade que gera números aleatórios.
Na maioria das vezes quando Rustaceans dizem "crate", eles querem dizer crate de biblioteca, e eles
usam "crate" de forma intercambiável com o conceito geral de programação de uma "biblioteca".

A *raiz do crate* é um arquivo fonte a partir do qual o compilador Rust começa e compõe
o módulo raiz do seu crate (explicaremos módulos em profundidade em ["Controlando
Escopo e Privacidade com Módulos"][modules]<!-- ignore -->).

Um *pacote* é um conjunto de um ou mais crates que fornece um conjunto de
funcionalidades. Um pacote contém um arquivo *Cargo.toml* que descreve como
construir esses crates. Cargo é na verdade um pacote que contém o crate binário
para a ferramenta de linha de comando que você tem usado para construir seu código. O pacote Cargo
também contém um crate de biblioteca do qual o crate binário depende. Outros
projetos podem depender do crate de biblioteca Cargo para usar a mesma lógica que a ferramenta de
linha de comando Cargo usa.

Um pacote pode conter quantos crates binários você quiser, mas no máximo apenas um
crate de biblioteca. Um pacote deve conter pelo menos um crate, seja ele um
crate de biblioteca ou binário.

Vamos ver o que acontece quando criamos um pacote. Primeiro, digitamos o
comando `cargo new meu-projeto`:

```console
$ cargo new meu-projeto
     Created binary (application) `meu-projeto` package
$ ls meu-projeto
Cargo.toml
src
$ ls meu-projeto/src
main.rs
```

Depois de executarmos `cargo new meu-projeto`, usamos `ls` para ver o que o Cargo cria. No
diretório *meu-projeto*, há um arquivo *Cargo.toml*, nos dando um pacote.
Há também um diretório *src* que contém *main.rs*. Abra *Cargo.toml* no
seu editor de texto e note que não há menção a *src/main.rs*. O Cargo
segue uma convenção de que *src/main.rs* é a raiz do crate de um crate binário
com o mesmo nome do pacote. Da mesma forma, o Cargo sabe que se o diretório do pacote
contiver *src/lib.rs*, o pacote contém um crate de biblioteca com o
mesmo nome do pacote, e *src/lib.rs* é sua raiz do crate. O Cargo passa os
arquivos raiz do crate para `rustc` para construir a biblioteca ou binário.

Aqui, temos um pacote que contém apenas *src/main.rs*, significando que ele apenas
contém um crate binário chamado `meu-projeto`. Se um pacote contiver *src/main.rs*
e *src/lib.rs*, ele tem dois crates: um binário e uma biblioteca, ambos com o mesmo
nome do pacote. Um pacote pode ter múltiplos crates binários colocando arquivos no
diretório *src/bin*: Cada arquivo será um crate binário separado.

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
