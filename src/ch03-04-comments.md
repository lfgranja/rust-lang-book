# Comentários

Todos os programadores se esforçam para tornar seu código fácil de entender, mas às vezes
uma explicação extra é justificada. Nesses casos, os programadores deixam _comentários_ em
seu código fonte que o compilador ignorará, mas que as pessoas lendo o
código fonte podem achar úteis.

Aqui está um comentário simples:

```rust
// hello, world
```

Em Rust, o estilo de comentário idiomático inicia um comentário com duas barras, e o
comentário continua até o final da linha. Para comentários que se estendem além de uma
única linha, você precisará incluir `//` em cada linha, como este:

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

Comentários também podem ser colocados no final de linhas contendo código:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Mas você os verá mais frequentemente usados neste formato, com o comentário em uma
linha separada acima do código que ele está anotando:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

O Rust também tem outro tipo de comentário, comentários de documentação, que
discutiremos na seção de [“Publicando um Crate no Crates.io”][publishing]<!-- ignore -->
do Capítulo 14.

[publishing]: ch14-02-publishing-to-crates-io.html
