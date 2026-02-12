# Summary

[A Linguagem de Programação Rust](title-page.md)

# Introdução
- [Prefácio](foreword.md)
- [Introdução](ch00-00-introduction.md)

# Iniciando
- [Iniciando](ch01-00-getting-started.md)
    - [Instalação](ch01-01-installation.md)
    - [Olá, Mundo!](ch01-02-hello-world.md)
    - [Olá, Cargo!](ch01-03-hello-cargo.md)

# Tutoriais
- [Programando um Jogo de Adivinhação](ch02-00-guessing-game-tutorial.md)

# Conceitos Básicos
- [Conceitos Comuns de Programação](ch03-00-common-programming-concepts.md)
    - [Variáveis e Mutabilidade](ch03-01-variables-and-mutability.md)
    - [Tipos de Dados](ch03-02-data-types.md)
    - [Funções](ch03-03-how-functions-work.md)
    - [Comentários](ch03-04-comments.md)
    - [Controle de Fluxo](ch03-05-control-flow.md)

# Posse
- [Entendendo a Posse (Ownership)](ch04-00-understanding-ownership.md)
    - [O que é Posse?](ch04-01-what-is-ownership.md)
    - [Referências e Empréstimo (Borrowing)](ch04-02-references-and-borrowing.md)
    - [O Tipo Slice](ch04-03-slices.md)

# Structs
- [Usando Structs para Estruturar Dados Relacionados](ch05-00-structs.md)
    - [Definindo e Instanciando Structs](ch05-01-defining-structs.md)
    - [Um Programa Exemplo Usando Structs](ch05-02-example-structs.md)
    - [Sintaxe de Métodos](ch05-03-method-syntax.md)

# Enums
- [Enums e Casamento de Padrões](ch06-00-enums.md)
    - [Definindo um Enum](ch06-01-defining-an-enum.md)
    - [O Operador de Controle de Fluxo `match`](ch06-02-match.md)
    - [Controle de Fluxo Conciso com `if let`](ch06-03-if-let.md)

# Módulos
- [Gerenciando Projetos Crescentes](ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
    - [Pacotes e Crates](ch07-01-packages-and-crates.md)
    - [Definindo Módulos](ch07-02-defining-modules-to-control-scope-and-privacy.md)
    - [Caminhos na Árvore de Módulos](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
    - [Trazendo Caminhos com `use`](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
    - [Separando Módulos](ch07-05-separating-modules-into-different-files.md)

# Coleções
- [Coleções Comuns](ch08-00-common-collections.md)
    - [Vetores](ch08-01-vectors.md)
    - [Strings](ch08-02-strings.md)
    - [Hash Maps](ch08-03-hash-maps.md)

# Erros
- [Tratamento de Erros](ch09-00-error-handling.md)
    - [panic!](ch09-01-unrecoverable-errors-with-panic.md)
    - [Result](ch09-02-recoverable-errors-with-result.md)
    - [panic! ou Result?](ch09-03-to-panic-or-not-to-panic.md)

# Genéricos e Traits
- [Tipos Genéricos, Traits e Lifetimes](ch10-00-generics.md)
    - [Genéricos](ch10-01-syntax.md)
    - [Traits](ch10-02-traits.md)
    - [Lifetimes](ch10-03-lifetime-syntax.md)

# Testes
- [Escrevendo Testes Automatizados](ch11-00-testing.md)
    - [Como Escrever Testes](ch11-01-writing-tests.md)
    - [Executando Testes](ch11-02-running-tests.md)
    - [Organização dos Testes](ch11-03-test-organization.md)

# Projeto I/O
- [Um Projeto de I/O](ch12-00-an-io-project.md)
    - [Argumentos de Linha de Comando](ch12-01-accepting-command-line-arguments.md)
    - [Lendo um Arquivo](ch12-02-reading-a-file.md)
    - [Refatorando](ch12-03-improving-error-handling-and-modularity.md)
    - [Test-Driven Development](ch12-04-testing-the-librarys-functionality.md)
    - [Variáveis de Ambiente](ch12-05-working-with-environment-variables.md)
    - [Saída de Erro Padrão](ch12-06-writing-to-stderr-instead-of-stdout.md)

# Funcional
- [Recursos de Linguagem Funcional](ch13-00-functional-features.md)
    - [Closures](ch13-01-closures.md)
    - [Iteradores](ch13-02-iterators.md)
    - [Melhorando I/O](ch13-03-improving-our-io-project.md)
    - [Performance](ch13-04-performance.md)

# Cargo Avançado
- [Mais Sobre Cargo](ch14-00-more-about-cargo.md)
    - [Perfis de Release](ch14-01-release-profiles.md)
    - [Publicando no Crates.io](ch14-02-publishing-to-crates-io.md)
    - [Workspaces](ch14-03-cargo-workspaces.md)
    - [Instalando Binários](ch14-04-installing-binaries.md)
    - [Comandos Personalizados](ch14-05-extending-cargo.md)

# Ponteiros Inteligentes
- [Ponteiros Inteligentes](ch15-00-smart-pointers.md)
    - [Box<T>](ch15-01-box.md)
    - [Deref](ch15-02-deref.md)
    - [Drop](ch15-03-drop.md)
    - [Rc<T>](ch15-04-rc.md)
    - [RefCell<T>](ch15-05-interior-mutability.md)
    - [Ciclos de Referência](ch15-06-reference-cycles.md)

# Concorrência
- [Concorrência Sem Medo](ch16-00-concurrency.md)
    - [Threads](ch16-01-threads.md)
    - [Passagem de Mensagem](ch16-02-message-passing.md)
    - [Estado Compartilhado](ch16-03-shared-state.md)
    - [Sync e Send](ch16-04-extensible-concurrency-sync-and-send.md)

# Async
- [Programação Assíncrona](ch17-00-async-await.md)
    - [Futures e Syntax](ch17-01-futures-and-syntax.md)
    - [Concorrência Async](ch17-02-concurrency-with-async.md)
    - [Mais Futures](ch17-03-more-futures.md)
    - [Streams](ch17-04-streams.md)
    - [Traits Async](ch17-05-traits-for-async.md)
    - [Futures, Tarefas e Threads](ch17-06-futures-tasks-threads.md)

# OOP
- [Orientação a Objetos](ch18-00-oop.md)
    - [O que é OO?](ch18-01-what-is-oo.md)
    - [Objetos de Trait](ch18-02-trait-objects.md)
    - [Padrão de Design OO](ch18-03-oo-design-patterns.md)

# Padrões
- [Padrões](ch19-00-patterns.md)
    - [Todos os Lugares](ch19-01-all-the-places-for-patterns.md)
    - [Refutabilidade](ch19-02-refutability.md)
    - [Sintaxe](ch19-03-pattern-syntax.md)

# Avançado
- [Recursos Avançados](ch20-00-advanced-features.md)
    - [Unsafe Rust](ch20-01-unsafe-rust.md)
    - [Traits Avançadas](ch20-02-advanced-traits.md)
    - [Tipos Avançados](ch20-03-advanced-types.md)
    - [Funções Avançadas](ch20-04-advanced-functions-and-closures.md)
    - [Macros](ch20-05-macros.md)

# Projeto Final
- [Servidor Web](ch21-00-final-project-a-web-server.md)
    - [Single-Threaded](ch21-01-single-threaded.md)
    - [Multithreaded](ch21-02-multithreaded.md)
    - [Shutdown](ch21-03-graceful-shutdown-and-cleanup.md)

# Apêndices
- [Apêndices](appendix-00.md)
    - [A - Palavras-chave](appendix-01-keywords.md)
    - [B - Operadores](appendix-02-operators.md)
    - [C - Traits Deriváveis](appendix-03-derivable-traits.md)
    - [D - Ferramentas](appendix-04-useful-development-tools.md)
    - [E - Edições](appendix-05-editions.md)
    - [F - Traduções](appendix-06-translation.md)
    - [G - Nightly Rust](appendix-07-nightly-rust.md)
