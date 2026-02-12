## Publicando um Crate no Crates.io

Usamos pacotes de [crates.io](https://crates.io/)<!-- ignore --> como dependências do nosso projeto, mas você também pode compartilhar seu código com outras pessoas publicando seus próprios pacotes. O registro de crates em [crates.io](https://crates.io/)<!-- ignore --> distribui o código-fonte de seus pacotes, portanto, ele hospeda principalmente código que é open source.

Rust e Cargo possuem recursos que tornam seu pacote publicado mais fácil para as pessoas encontrarem e usarem. Falaremos sobre alguns desses recursos a seguir e depois explicaremos como publicar um pacote.

### Fazendo Comentários de Documentação Úteis

Documentar seus pacotes com precisão ajudará outros usuários a saber como e quando usá-los, então vale a pena investir tempo para escrever documentação. No Capítulo 3, discutimos como comentar o código Rust usando duas barras, `//`. O Rust também tem um tipo específico de comentário para documentação, conhecido convenientemente como *comentário de documentação*, que gerará documentação HTML. O HTML exibe o conteúdo dos comentários de documentação para itens da API pública destinados a programadores interessados em saber como *usar* seu crate, em oposição a como seu crate é *implementado*.

Os comentários de documentação usam três barras, `///`, em vez de duas e suportam a notação Markdown para formatar o texto. Coloque comentários de documentação logo antes do item que eles estão documentando. A Listagem 14-1 mostra comentários de documentação para uma função `add_one` em um crate chamado `my_crate`.

<Listing number="14-1" file-name="src/lib.rs" caption="Um comentário de documentação para uma função">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Aqui, damos uma descrição do que a função `add_one` faz, iniciamos uma seção com o título `Examples` e, em seguida, fornecemos um código que demonstra como usar a função `add_one`. Podemos gerar a documentação HTML a partir deste comentário de documentação executando `cargo doc`. Este comando executa a ferramenta `rustdoc` distribuída com o Rust e coloca a documentação HTML gerada no diretório *target/doc*.

Por conveniência, executar `cargo doc --open` construirá o HTML para a documentação do seu crate atual (bem como a documentação para todas as dependências do seu crate) e abrirá o resultado em um navegador da web. Navegue até a função `add_one` e você verá como o texto nos comentários de documentação é renderizado, conforme mostrado na Figura 14-1.

<img alt="Documentação HTML renderizada para a função `add_one` de `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figura 14-1: A documentação HTML para a função `add_one`</span>

#### Seções Comumente Usadas

Usamos o cabeçalho Markdown `# Examples` na Listagem 14-1 para criar uma seção no HTML com o título “Examples”. Aqui estão algumas outras seções que os autores de crates comumente usam em sua documentação:

- **Panics**: Estes são os cenários nos quais a função sendo documentada pode entrar em pânico. Os chamadores da função que não querem que seus programas entrem em pânico devem garantir que não chamem a função nessas situações.
- **Errors**: Se a função retornar um `Result`, descrever os tipos de erros que podem ocorrer e quais condições podem fazer com que esses erros sejam retornados pode ser útil para os chamadores, para que eles possam escrever código para lidar com os diferentes tipos de erros de maneiras diferentes.
- **Safety**: Se a função for `unsafe` para chamar (discutimos `unsafe` no Capítulo 20), deve haver uma seção explicando por que a função é insegura e cobrindo os invariantes que a função espera que os chamadores mantenham.

A maioria dos comentários de documentação não precisa de todas essas seções, mas esta é uma boa lista de verificação para lembrá-lo dos aspectos do seu código que os usuários estarão interessados em saber.

#### Comentários de Documentação como Testes

Adicionar blocos de código de exemplo em seus comentários de documentação pode ajudar a demonstrar como usar sua biblioteca e tem um bônus adicional: Executar `cargo test` executará os exemplos de código em sua documentação como testes! Nada é melhor do que documentação com exemplos. Mas nada é pior do que exemplos que não funcionam porque o código mudou desde que a documentação foi escrita. Se executarmos `cargo test` com a documentação para a função `add_one` da Listagem 14-1, veremos uma seção nos resultados do teste parecida com esta:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Agora, se mudarmos a função ou o exemplo para que o `assert_eq!` no exemplo entre em pânico, e executarmos `cargo test` novamente, veremos que os testes de documentação detectam que o exemplo e o código estão fora de sincronia um com o outro!

<!-- Old headings. Do not remove or links may break. -->

<a id="commenting-contained-items"></a>

#### Comentários de Itens Contidos

O estilo de comentário de documentação `//!` adiciona documentação ao item que *contém* os comentários, em vez de aos itens *seguintes* aos comentários. Normalmente usamos esses comentários de documentação dentro do arquivo raiz do crate (*src/lib.rs* por convenção) ou dentro de um módulo para documentar o crate ou o módulo como um todo.

Por exemplo, para adicionar documentação que descreve o propósito do crate `my_crate` que contém a função `add_one`, adicionamos comentários de documentação que começam com `//!` no início do arquivo *src/lib.rs*, conforme mostrado na Listagem 14-2.

<Listing number="14-2" file-name="src/lib.rs" caption="A documentação para o crate `my_crate` como um todo">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

Observe que não há nenhum código após a última linha que começa com `//!`. Como começamos os comentários com `//!` em vez de `///`, estamos documentando o item que contém este comentário em vez de um item que segue este comentário. Neste caso, esse item é o arquivo *src/lib.rs*, que é a raiz do crate. Esses comentários descrevem o crate inteiro.

Quando executamos `cargo doc --open`, esses comentários serão exibidos na página inicial da documentação de `my_crate` acima da lista de itens públicos no crate, conforme mostrado na Figura 14-2.

Comentários de documentação dentro de itens são úteis para descrever crates e módulos especialmente. Use-os para explicar o propósito geral do contêiner para ajudar seus usuários a entender a organização do crate.

<img alt="Documentação HTML renderizada com um comentário para o crate como um todo" src="img/trpl14-02.png" class="center" />

<span class="caption">Figura 14-2: A documentação renderizada para `my_crate`, incluindo o comentário descrevendo o crate como um todo</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="exporting-a-convenient-public-api-with-pub-use"></a>

### Exportando uma API Pública Conveniente

A estrutura da sua API pública é uma consideração importante ao publicar um crate. As pessoas que usam seu crate estão menos familiarizadas com a estrutura do que você e podem ter dificuldade em encontrar as peças que desejam usar se seu crate tiver uma grande hierarquia de módulos.

No Capítulo 7, abordamos como tornar os itens públicos usando a palavra-chave `pub` e como trazer itens para um escopo com a palavra-chave `use`. No entanto, a estrutura que faz sentido para você enquanto você está desenvolvendo um crate pode não ser muito conveniente para seus usuários. Você pode querer organizar suas structs em uma hierarquia contendo vários níveis, mas então as pessoas que desejam usar um tipo que você definiu profundamente na hierarquia podem ter problemas para descobrir que esse tipo existe. Elas também podem se incomodar por ter que digitar `use my_crate::some_module::another_module::UsefulType;` em vez de `use my_crate::UsefulType;`.

A boa notícia é que se a estrutura *não* for conveniente para outros usarem de outra biblioteca, você não precisa reorganizar sua organização interna: Em vez disso, você pode reexportar itens para criar uma estrutura pública diferente da sua estrutura privada usando `pub use`. *Reexportar* pega um item público em um local e o torna público em outro local, como se tivesse sido definido no outro local.

Por exemplo, digamos que fizemos uma biblioteca chamada `art` para modelar conceitos artísticos. Dentro desta biblioteca estão dois módulos: um módulo `kinds` contendo dois enums chamados `PrimaryColor` e `SecondaryColor` e um módulo `utils` contendo uma função chamada `mix`, conforme mostrado na Listagem 14-3.

<Listing number="14-3" file-name="src/lib.rs" caption="Uma biblioteca `art` com itens organizados em módulos `kinds` e `utils`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

A Figura 14-3 mostra como seria a página inicial da documentação para este crate gerada por `cargo doc`.

<img alt="Documentação renderizada para o crate `art` que lista os módulos `kinds` e `utils`" src="img/trpl14-03.png" class="center" />

<span class="caption">Figura 14-3: A página inicial da documentação para `art` que lista os módulos `kinds` e `utils`</span>

Observe que os tipos `PrimaryColor` e `SecondaryColor` não estão listados na página inicial, nem a função `mix`. Temos que clicar em `kinds` e `utils` para vê-los.

Outro crate que depende desta biblioteca precisaria de instruções `use` que trazem os itens de `art` para o escopo, especificando a estrutura do módulo que está definida atualmente. A Listagem 14-4 mostra um exemplo de um crate que usa os itens `PrimaryColor` e `mix` do crate `art`.

<Listing number="14-4" file-name="src/main.rs" caption="Um crate usando os itens do crate `art` com sua estrutura interna exportada">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

O autor do código na Listagem 14-4, que usa o crate `art`, teve que descobrir que `PrimaryColor` está no módulo `kinds` e `mix` está no módulo `utils`. A estrutura do módulo do crate `art` é mais relevante para os desenvolvedores que trabalham no crate `art` do que para aqueles que o usam. A estrutura interna não contém nenhuma informação útil para alguém tentando entender como usar o crate `art`, mas causa confusão porque os desenvolvedores que o usam têm que descobrir onde procurar e devem especificar os nomes dos módulos nas instruções `use`.

Para remover a organização interna da API pública, podemos modificar o código do crate `art` na Listagem 14-3 para adicionar instruções `pub use` para reexportar os itens no nível superior, conforme mostrado na Listagem 14-5.

<Listing number="14-5" file-name="src/lib.rs" caption="Adicionando instruções `pub use` para reexportar itens">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

A documentação da API que `cargo doc` gera para este crate agora listará e vinculará as reexportações na página inicial, conforme mostrado na Figura 14-4, tornando os tipos `PrimaryColor` e `SecondaryColor` e a função `mix` mais fáceis de encontrar.

<img alt="Documentação renderizada para o crate `art` com as reexportações na página inicial" src="img/trpl14-04.png" class="center" />

<span class="caption">Figura 14-4: A página inicial da documentação para `art` que lista as reexportações</span>

Os usuários do crate `art` ainda podem ver e usar a estrutura interna da Listagem 14-3, conforme demonstrado na Listagem 14-4, ou podem usar a estrutura mais conveniente na Listagem 14-5, conforme mostrado na Listagem 14-6.

<Listing number="14-6" file-name="src/main.rs" caption="Um programa usando os itens reexportados do crate `art`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

Em casos onde há muitos módulos aninhados, reexportar os tipos no nível superior com `pub use` pode fazer uma diferença significativa na experiência das pessoas que usam o crate. Outro uso comum de `pub use` é reexportar definições de uma dependência no crate atual para tornar as definições desse crate parte da API pública do seu crate.

Criar uma estrutura de API pública útil é mais uma arte do que uma ciência, e você pode iterar para encontrar a API que funciona melhor para seus usuários. Escolher `pub use` lhe dá flexibilidade em como você estrutura seu crate internamente e dissocia essa estrutura interna do que você apresenta aos seus usuários. Olhe para alguns dos códigos de crates que você instalou para ver se a estrutura interna deles difere de sua API pública.

### Configurando uma Conta no Crates.io

Antes de poder publicar quaisquer crates, você precisa criar uma conta em [crates.io](https://crates.io/)<!-- ignore --> e obter um token de API. Para fazer isso, visite a página inicial em [crates.io](https://crates.io/)<!-- ignore --> e faça login através de uma conta GitHub. (A conta GitHub é atualmente um requisito, mas o site pode suportar outras formas de criar uma conta no futuro.) Depois de fazer login, visite as configurações da sua conta em [https://crates.io/me/](https://crates.io/me/)<!-- ignore --> e recupere sua chave de API. Em seguida, execute o comando `cargo login` e cole sua chave de API quando solicitado, assim:

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Este comando informará ao Cargo seu token de API e o armazenará localmente em *~/.cargo/credentials.toml*. Observe que este token é um segredo: Não o compartilhe com mais ninguém. Se você compartilhá-lo com alguém por qualquer motivo, deve revogá-lo e gerar um novo token em [crates.io](https://crates.io/)<!-- ignore -->.

### Adicionando Metadados a um Novo Crate

Digamos que você tenha um crate que deseja publicar. Antes de publicar, você precisará adicionar alguns metadados na seção `[package]` do arquivo *Cargo.toml* do crate.

Seu crate precisará de um nome exclusivo. Enquanto você está trabalhando em um crate localmente, você pode nomear um crate como quiser. No entanto, os nomes de crate em [crates.io](https://crates.io/)<!-- ignore --> são alocados por ordem de chegada. Uma vez que um nome de crate é tomado, ninguém mais pode publicar um crate com esse nome. Antes de tentar publicar um crate, procure o nome que você deseja usar. Se o nome tiver sido usado, você precisará encontrar outro nome e editar o campo `name` no arquivo *Cargo.toml* na seção `[package]` para usar o novo nome para publicação, assim:

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Mesmo que você tenha escolhido um nome exclusivo, quando executar `cargo publish` para publicar o crate neste ponto, receberá um aviso e depois um erro:

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

Isso resulta em um erro porque você está perdendo algumas informações cruciais: Uma descrição e uma licença são necessárias para que as pessoas saibam o que seu crate faz e sob quais termos podem usá-lo. No *Cargo.toml*, adicione uma descrição que seja apenas uma ou duas frases, porque ela aparecerá com seu crate nos resultados da pesquisa. Para o campo `license`, você precisa fornecer um *valor identificador de licença*. A [Software Package Data Exchange (SPDX) da Linux Foundation][spdx] lista os identificadores que você pode usar para este valor. Por exemplo, para especificar que você licenciou seu crate usando a Licença MIT, adicione o identificador `MIT`:

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Se você quiser usar uma licença que não aparece no SPDX, precisará colocar o texto dessa licença em um arquivo, incluir o arquivo em seu projeto e, em seguida, usar `license-file` para especificar o nome desse arquivo em vez de usar a chave `license`.

Orientação sobre qual licença é apropriada para seu projeto está além do escopo deste livro. Muitas pessoas na comunidade Rust licenciam seus projetos da mesma maneira que o Rust, usando uma licença dupla de `MIT OR Apache-2.0`. Essa prática demonstra que você também pode especificar vários identificadores de licença separados por `OR` para ter várias licenças para seu projeto.

Com um nome exclusivo, a versão, sua descrição e uma licença adicionados, o arquivo *Cargo.toml* para um projeto que está pronto para ser publicado pode ser assim:

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "Um jogo divertido onde você adivinha qual número o computador escolheu."
license = "MIT OR Apache-2.0"

[dependencies]
```

A [documentação do Cargo](https://doc.rust-lang.org/cargo/) descreve outros metadados que você pode especificar para garantir que outros possam descobrir e usar seu crate mais facilmente.

### Publicando no Crates.io

Agora que você criou uma conta, salvou seu token de API, escolheu um nome para seu crate e especificou os metadados necessários, você está pronto para publicar! Publicar um crate carrega uma versão específica para [crates.io](https://crates.io/)<!-- ignore --> para outros usarem.

Tenha cuidado, porque uma publicação é *permanente*. A versão nunca pode ser substituída e o código não pode ser excluído, exceto em certas circunstâncias. Um objetivo importante do Crates.io é atuar como um arquivo permanente de código para que as compilações de todos os projetos que dependem de crates de [crates.io](https://crates.io/)<!-- ignore --> continuem a funcionar. Permitir a exclusão de versões tornaria impossível cumprir esse objetivo. No entanto, não há limite para o número de versões de crate que você pode publicar.

Execute o comando `cargo publish` novamente. Deve ter sucesso agora:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

Parabéns! Você agora compartilhou seu código com a comunidade Rust, e qualquer pessoa pode adicionar facilmente seu crate como uma dependência de seu projeto.

### Publicando uma Nova Versão de um Crate Existente

Quando você fez alterações em seu crate e está pronto para lançar uma nova versão, você altera o valor `version` especificado em seu arquivo *Cargo.toml* e publica novamente. Use as [regras de Versionamento Semântico][semver] para decidir qual é o próximo número de versão apropriado, com base nos tipos de alterações que você fez. Em seguida, execute `cargo publish` para carregar a nova versão.

<!-- Old headings. Do not remove or links may break. -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>
<a id="deprecating-versions-from-cratesio-with-cargo-yank"></a>

### Depreciando Versões do Crates.io

Embora você não possa remover versões anteriores de um crate, você pode impedir que quaisquer projetos futuros as adicionem como uma nova dependência. Isso é útil quando uma versão de crate está quebrada por um motivo ou outro. Nessas situações, o Cargo suporta *yanking* (arrancar) uma versão de crate.

*Yanking* uma versão impede que novos projetos dependam dessa versão, permitindo que todos os projetos existentes que dependem dela continuem. Essencialmente, um yank significa que todos os projetos com um *Cargo.lock* não quebrarão, e quaisquer arquivos *Cargo.lock* futuros gerados não usarão a versão yanked.

Para fazer yank de uma versão de um crate, no diretório do crate que você publicou anteriormente, execute `cargo yank` e especifique qual versão você deseja fazer yank. Por exemplo, se publicamos um crate chamado `guessing_game` versão 1.0.1 e queremos fazer yank nele, executaríamos o seguinte no diretório do projeto para `guessing_game`:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Ao adicionar `--undo` ao comando, você também pode desfazer um yank e permitir que os projetos voltem a depender de uma versão:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Um yank *não* exclui nenhum código. Ele não pode, por exemplo, excluir segredos carregados acidentalmente. Se isso acontecer, você deve redefinir esses segredos imediatamente.

[spdx]: https://spdx.org/licenses/
[semver]: https://semver.org/
