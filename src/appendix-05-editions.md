# Apêndice E: Edições

No Capítulo 1, você viu que `cargo new` adiciona um pouco de metadados ao seu arquivo *Cargo.toml* sobre uma *edição*. Este apêndice explica o que isso significa.

A linguagem Rust e o compilador têm um ciclo de lançamento de seis semanas. Isso significa que os usuários obtêm um fluxo constante de novas funcionalidades. Outras linguagens de programação lançam atualizações maiores com menos frequência; Rust lança atualizações menores com mais frequência. Depois de algum tempo, todas essas pequenas mudanças se acumulam. Mas de lançamento em lançamento, pode ser difícil olhar para trás e dizer: "Uau, entre Rust 1.10 e Rust 1.31, Rust mudou muito!"

A cada dois ou três anos, a equipe Rust produz uma nova *edição* do Rust. Cada edição reúne as funcionalidades que chegaram em um pacote claro e totalmente atualizado de documentação e ferramentas. Novas edições são enviadas como parte do processo normal de lançamento de seis semanas.

Edições servem a diferentes propósitos para diferentes pessoas:

* Para usuários de Rust, uma nova edição reúne funcionalidades que foram adicionadas incrementalmente em um pacote fácil de entender.
* Para não usuários de Rust, uma nova edição sinaliza que alguns grandes avanços foram feitos na linguagem, o que pode tornar Rust valioso de uma nova olhada.
* Para desenvolvedores de Rust, uma nova edição fornece um ponto de encontro para todo o projeto se unir e focar em um objetivo.

No momento da escrita, há três edições do Rust disponíveis:

* *Edição 2015*: Esta é a versão 1.0 do Rust com algumas pequenas melhorias incrementais adicionadas posteriormente. Se você não especificar uma edição em seu *Cargo.toml*, esta é a edição que você obtém por padrão para compatibilidade com versões anteriores.
* *Edição 2018*: Esta edição foi lançada com Rust 1.31.0! Esta edição introduziu uma variedade de novos recursos, incluindo uma nova sintaxe para caminhos de módulo, `async`/`await` e um sistema de macros processuais mais simples.
* *Edição 2021*: Esta edição foi lançada com Rust 1.56.0!

A parte mais importante das edições é que, se você não optar, verá apenas funcionalidades compatíveis com versões anteriores. Para ver coisas que podem não ser compatíveis com versões anteriores (como uma nova palavra-chave), você deve optar por aceitar as alterações.

Você pode optar por uma nova edição de uma crate adicionando a chave `edition` à seção `[package]` em seu arquivo *Cargo.toml*. O Listagem E-1 mostra um arquivo *Cargo.toml* que opta pela edição 2021.

Listagem E-1: Optando pela edição 2021

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"
```

A maioria dos projetos Cargo criados hoje são criados usando a edição 2021. O arquivo *Cargo.toml* que você viu no Capítulo 1 e criou em todos os projetos neste livro opta pela edição 2021 por padrão.

Uma grande vantagem do sistema de edição é que as crates em diferentes edições podem interoperar umas com as outras. Se você tiver uma crate escrita na edição 2015 e quiser usar uma dependência escrita na edição 2021, tudo funcionará. O oposto também é verdadeiro: crates de edição 2021 podem usar dependências de edição 2015.

Este sistema permite que o ecossistema migre para novas edições gradualmente, sem quebrar o código existente.

Para mais detalhes, o [Guia de Edição](https://doc.rust-lang.org/edition-guide/index.html) é um livro completo sobre o assunto.
