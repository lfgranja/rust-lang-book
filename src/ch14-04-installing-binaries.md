<!-- Old headings. Do not remove or links may break. -->
<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Instalando Binários com `cargo install`

O comando `cargo install` permite que você instale e use crates binários localmente. Isso não se destina a substituir pacotes do sistema; destina-se a ser uma maneira conveniente para desenvolvedores Rust instalarem ferramentas que outros compartilharam em [crates.io](https://crates.io/)<!-- ignore -->. Observe que você só pode instalar pacotes que tenham alvos binários. Um *alvo binário* é o programa executável que é criado se o crate tiver um arquivo *src/main.rs* ou outro arquivo especificado como binário, em oposição a um alvo de biblioteca que não é executável por si só, mas é adequado para inclusão em outros programas. Normalmente, os crates têm informações no arquivo README sobre se um crate é uma biblioteca, tem um alvo binário ou ambos.

Todos os binários instalados com `cargo install` são armazenados na pasta *bin* da raiz de instalação. Se você instalou o Rust usando *rustup.rs* e não tem nenhuma configuração personalizada, este diretório será *$HOME/.cargo/bin*. Certifique-se de que este diretório esteja em seu `$PATH` para poder executar programas que você instalou com `cargo install`.

Por exemplo, no Capítulo 12 mencionamos que há uma implementação Rust da ferramenta `grep` chamada `ripgrep` para pesquisar arquivos. Para instalar o `ripgrep`, podemos executar o seguinte:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

A penúltima linha da saída mostra a localização e o nome do binário instalado, que no caso do `ripgrep` é `rg`. Contanto que o diretório de instalação esteja em seu `$PATH`, conforme mencionado anteriormente, você pode então executar `rg --help` e começar a usar uma ferramenta mais rápida e "Rustica" para pesquisar arquivos!
