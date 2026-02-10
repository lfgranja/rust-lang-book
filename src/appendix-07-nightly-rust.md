# Apêndice G: Rust Nightly e Funcionalidades Instáveis

Este apêndice documenta como o modelo de desenvolvimento e lançamento do Rust funciona e como você pode acessar funcionalidades que ainda não estão prontas para serem estabilizadas.

## Estabilidade Sem Estagnação

O modelo de lançamento do Rust foi inspirado pelo "Modern C++" e pelo modelo de "Evergreen browser" usado pelo Chrome e Firefox. A ideia é fornecer atualizações frequentes e menores para que os usuários não tenham que esperar anos por novas funcionalidades, mas também manter a estabilidade para que as atualizações não quebrem o código existente.

Rust tem três canais de lançamento:

* **Nightly**
* **Beta**
* **Stable**

A cada dia, uma nova versão do compilador é produzida a partir do código-fonte mais recente no repositório `master`. Esta versão é chamada de **nightly** (noturna). Se a compilação noturna passar por todos os testes, ela é promovida a uma versão nightly oficial.

A cada seis semanas, a versão nightly atual é promovida para **beta**. A versão beta é testada por seis semanas. Se nenhum bug grave for encontrado, ela é promovida para **stable** (estável). A versão estável é a que a maioria dos usuários do Rust usa.

Isso significa que, se uma funcionalidade for adicionada ao Rust hoje, ela estará disponível na versão nightly amanhã. Em seis semanas, ela estará na versão beta. Em mais seis semanas, ela estará na versão estável. Portanto, uma funcionalidade leva pelo menos 12 semanas para passar do desenvolvimento para a versão estável.

## Funcionalidades Instáveis

Rust usa uma técnica chamada "feature flags" (sinalizadores de funcionalidade) para determinar quais funcionalidades estão habilitadas em uma determinada versão do compilador. Se uma nova funcionalidade estiver sendo trabalhada ativamente, ela deve ser adicionada atrás de um sinalizador de funcionalidade.

Se você estiver usando a versão estável ou beta do Rust, não poderá usar nenhum sinalizador de funcionalidade. Isso evita que funcionalidades instáveis sejam usadas acidentalmente em código de produção.

Se você quiser usar funcionalidades instáveis, deve usar a versão nightly do Rust. Você pode alternar para a versão nightly usando `rustup`:

```console
$ rustup default nightly
```

Com a versão nightly instalada, você pode usar sinalizadores de funcionalidade adicionando `#![feature(nome_da_funcionalidade)]` ao topo do seu arquivo `main.rs` ou `lib.rs`.

Por exemplo, se houver uma funcionalidade instável chamada `box_syntax`, você pode habilitá-la assim:

```rust
#![feature(box_syntax)]

fn main() {
    let b = box 5;
}
```

## O Manifesto de Funcionalidades do Compilador

O compilador Rust mantém uma lista de todas as funcionalidades instáveis permitidas. Se você tentar usar uma funcionalidade que não está na lista ou se tentar usar uma funcionalidade instável sem o sinalizador de funcionalidade apropriado, o compilador emitirá um erro.

## Rastreamento de Problemas

Cada funcionalidade instável tem um problema de rastreamento no repositório GitHub do Rust. Este problema é usado para discutir o design da funcionalidade, relatar bugs e acompanhar o progresso em direção à estabilização. Se você encontrar um bug em uma funcionalidade instável, deve relatá-lo no problema de rastreamento correspondente.

## Rustup e Toolchains

`rustup` é a ferramenta recomendada para gerenciar versões do Rust. Ele permite que você instale e alterne facilmente entre as toolchains estável, beta e nightly.

Você pode instalar uma toolchain específica usando `rustup install`:

```console
$ rustup install nightly
```

Você pode então executar um comando usando essa toolchain com `rustup run`:

```console
$ rustup run nightly cargo build
```

Ou você pode substituir a toolchain padrão para um diretório específico:

```console
$ rustup override set nightly
```

Isso fará com que todos os comandos `cargo` e `rustc` nesse diretório usem a versão nightly.

## Conclusão

O modelo de lançamento do Rust permite que a linguagem evolua rapidamente, mantendo a estabilidade para os usuários. Se você precisar de acesso às funcionalidades mais recentes, pode usar a versão nightly, mas esteja ciente de que as funcionalidades instáveis podem mudar ou ser removidas a qualquer momento. Para a maioria dos usuários, a versão estável é a melhor escolha.
