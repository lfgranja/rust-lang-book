## Futures e a Sintaxe Async

Os elementos-chave da programação assíncrona em Rust são _futures_ e as palavras-chave `async` e `await` do Rust.

Um _future_ (futuro) é um valor que pode não estar pronto agora, mas ficará pronto em algum momento no futuro. (Esse mesmo conceito aparece em muitas linguagens, às vezes sob outros nomes, como _task_ ou _promise_.) Rust fornece uma trait `Future` como um bloco de construção para que diferentes operações assíncronas possam ser implementadas com diferentes estruturas de dados, mas com uma interface comum. Em Rust, futures são tipos que implementam a trait `Future`. Cada future mantém suas próprias informações sobre o progresso que foi feito e o que significa estar “pronto”.

Você pode aplicar a palavra-chave `async` a blocos e funções para especificar que eles podem ser interrompidos e retomados. Dentro de um bloco async ou função async, você pode usar a palavra-chave `await` para _aguardar um future_ (isto é, esperar que ele fique pronto). Qualquer ponto onde você aguarda um future dentro de um bloco ou função async é um local potencial para esse bloco ou função pausar e retomar. O processo de verificar com um future para ver se seu valor já está disponível é chamado de _polling_ (sondagem).

Algumas outras linguagens, como C# e JavaScript, também usam palavras-chave `async` e `await` para programação assíncrona. Se você está familiarizado com essas linguagens, pode notar algumas diferenças significativas em como Rust lida com a sintaxe. Isso é por um bom motivo, como veremos!

Ao escrever Rust assíncrono, usamos as palavras-chave `async` e `await` na maior parte do tempo. Rust as compila em código equivalente usando a trait `Future`, muito parecido com a forma como compila loops `for` em código equivalente usando a trait `Iterator`. Como Rust fornece a trait `Future`, no entanto, você também pode implementá-la para seus próprios tipos de dados quando precisar. Muitas das funções que veremos ao longo deste capítulo retornam tipos com suas próprias implementações de `Future`. Voltaremos à definição da trait no final do capítulo e nos aprofundaremos mais em como ela funciona, mas isso é detalhe suficiente para continuarmos avançando.

Tudo isso pode parecer um pouco abstrato, então vamos escrever nosso primeiro programa assíncrono: um pequeno web scraper. Passaremos duas URLs pela linha de comando, buscaremos ambas concorrentemente e retornaremos o resultado de qualquer uma que terminar primeiro. Este exemplo terá uma boa quantidade de sintaxe nova, mas não se preocupe — explicaremos tudo o que você precisa saber conforme avançamos.

## Nosso Primeiro Programa Async

Para manter o foco deste capítulo em aprender async em vez de lidar com partes do ecossistema, criamos o crate `trpl` (`trpl` é uma abreviação de “The Rust Programming Language”). Ele reexporta todos os tipos, traits e funções que você precisará, principalmente dos crates [`futures`][futures-crate]<!-- ignore --> e [`tokio`][tokio]<!-- ignore -->. O crate `futures` é um lar oficial para experimentação em Rust para código async, e é na verdade onde a trait `Future` foi originalmente projetada. Tokio é o runtime async mais amplamente usado em Rust hoje, especialmente para aplicações web. Existem outros ótimos runtimes por aí, e eles podem ser mais adequados para seus propósitos. Usamos o crate `tokio` por baixo dos panos para o `trpl` porque ele é bem testado e amplamente utilizado.

Em alguns casos, `trpl` também renomeia ou envolve as APIs originais para manter você focado nos detalhes relevantes para este capítulo. Se você quiser entender o que o crate faz, encorajamos você a conferir [seu código-fonte][crate-source]. Você poderá ver de qual crate cada reexportação vem, e deixamos extensos comentários explicando o que o crate faz.

Crie um novo projeto binário chamado `hello-async` e adicione o crate `trpl` como uma dependência:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Agora podemos usar as várias peças fornecidas pelo `trpl` para escrever nosso primeiro programa assíncrono. Construiremos uma pequena ferramenta de linha de comando que busca duas páginas da web, extrai o elemento `<title>` de cada uma e imprime o título de qualquer página que terminar todo esse processo primeiro.

### Definindo a Função page_title

Vamos começar escrevendo uma função que recebe uma URL de página como parâmetro, faz uma requisição para ela e retorna o texto do elemento `<title>` (veja a Listagem 17-1).

<Listing number="17-1" file-name="src/main.rs" caption="Definindo uma função async para obter o elemento título de uma página HTML">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Primeiro, definimos uma função chamada `page_title` e a marcamos com a palavra-chave `async`. Então usamos a função `trpl::get` para buscar qualquer URL que for passada e adicionamos a palavra-chave `await` para aguardar a resposta. Para obter o texto da `response` (resposta), chamamos seu método `text` e mais uma vez o aguardamos com a palavra-chave `await`. Ambos os passos são assíncronos. Para a função `get`, temos que esperar o servidor enviar de volta a primeira parte de sua resposta, que incluirá cabeçalhos HTTP, cookies e assim por diante, e pode ser entregue separadamente do corpo da resposta. Especialmente se o corpo for muito grande, pode levar algum tempo para que tudo chegue. Como temos que esperar pela _totalidade_ da resposta chegar, o método `text` também é async.

Temos que aguardar explicitamente ambos esses futures, porque futures em Rust são _preguiçosos_ (lazy): eles não fazem nada até que você peça a eles com a palavra-chave `await`. (Na verdade, Rust mostrará um aviso do compilador se você não usar um future.) Isso pode lembrá-lo da discussão sobre iteradores na seção [“Processando uma Série de Itens com Iteradores”][iterators-lazy]<!-- ignore --> no Capítulo 13. Iteradores não fazem nada a menos que você chame seu método `next` — seja diretamente ou usando loops `for` ou métodos como `map` que usam `next` por baixo dos panos. Da mesma forma, futures não fazem nada a menos que você explicitamente peça a eles. Essa preguiça permite que Rust evite rodar código async até que ele seja realmente necessário.

> Nota: Isso é diferente do comportamento que vimos ao usar `thread::spawn` na seção [“Criando uma Nova Thread com spawn”][thread-spawn]<!-- ignore --> no Capítulo 16, onde a closure que passamos para outra thread começava a rodar imediatamente. Também é diferente de como muitas outras linguagens abordam async. Mas é importante para Rust ser capaz de fornecer suas garantias de performance, assim como é com iteradores.

Uma vez que temos `response_text`, podemos analisá-lo em uma instância do tipo `Html` usando `Html::parse`. Em vez de uma string bruta, agora temos um tipo de dados que podemos usar para trabalhar com o HTML como uma estrutura de dados mais rica. Em particular, podemos usar o método `select_first` para encontrar a primeira instância de um dado seletor CSS. Passando a string `"title"`, obteremos o primeiro elemento `<title>` no documento, se houver um. Como pode não haver nenhum elemento correspondente, `select_first` retorna um `Option<ElementRef>`. Finalmente, usamos o método `Option::map`, que nos permite trabalhar com o item no `Option` se ele estiver presente, e não fazer nada se não estiver. (Poderíamos também usar uma expressão `match` aqui, mas `map` é mais idiomático.) No corpo da função que fornecemos ao `map`, chamamos `inner_html` no `title` para obter seu conteúdo, que é uma `String`. Quando tudo estiver dito e feito, temos um `Option<String>`.

Note que a palavra-chave `await` do Rust vai _depois_ da expressão que você está aguardando, não antes dela. Isto é, é uma palavra-chave _pós-fixada_. Isso pode diferir do que você está acostumado se usou `async` em outras linguagens, mas em Rust isso torna cadeias de métodos muito mais agradáveis de trabalhar. Como resultado, poderíamos mudar o corpo de `page_title` para encadear as chamadas de função `trpl::get` e `text` juntas com `await` entre elas, como mostrado na Listagem 17-2.

<Listing number="17-2" file-name="src/main.rs" caption="Encadeando com a palavra-chave `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Com isso, escrevemos com sucesso nossa primeira função async! Antes de adicionarmos algum código em `main` para chamá-la, vamos falar um pouco mais sobre o que escrevemos e o que isso significa.

Quando Rust vê um _bloco_ marcado com a palavra-chave `async`, ele o compila em um tipo de dados único e anônimo que implementa a trait `Future`. Quando Rust vê uma _função_ marcada com `async`, ele a compila em uma função não-async cujo corpo é um bloco async. O tipo de retorno de uma função async é o tipo do dado anônimo que o compilador cria para aquele bloco async.

Assim, escrever `async fn` é equivalente a escrever uma função que retorna um _future_ do tipo de retorno. Para o compilador, uma definição de função como a `async fn page_title` na Listagem 17-1 é aproximadamente equivalente a uma função não-async definida assim:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Vamos percorrer cada parte da versão transformada:

- Ela usa a sintaxe `impl Trait` que discutimos no Capítulo 10 na seção [“Traits como Parâmetros”][impl-trait]<!-- ignore -->.
- O valor retornado implementa a trait `Future` com um tipo associado de `Output`. Note que o tipo `Output` é `Option<String>`, que é o mesmo que o tipo de retorno original da versão `async fn` de `page_title`.
- Todo o código chamado no corpo da função original é envolvido em um bloco `async move`. Lembre-se que blocos são expressões. Todo este bloco é a expressão retornada da função.
- Este bloco async produz um valor com o tipo `Option<String>`, como descrito. Esse valor corresponde ao tipo `Output` no tipo de retorno. Isso é exatamente como outros blocos que você viu.
- O novo corpo da função é um bloco `async move` por causa de como ele usa o parâmetro `url`. (Falaremos muito mais sobre `async` versus `async move` mais tarde no capítulo.)

Agora podemos chamar `page_title` em `main`.

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### Executando uma Função Async com um Runtime

Para começar, vamos obter o título de uma única página, mostrado na Listagem 17-3. Infelizmente, este código ainda não compila.

<Listing number="17-3" file-name="src/main.rs" caption="Chamando a função `page_title` de `main` com um argumento fornecido pelo usuário">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Seguimos o mesmo padrão que usamos para obter argumentos de linha de comando na seção [“Aceitando Argumentos de Linha de Comando”][cli-args]<!-- ignore --> no Capítulo 12. Então passamos o argumento URL para `page_title` e aguardamos o resultado. Como o valor produzido pelo future é um `Option<String>`, usamos uma expressão `match` para imprimir mensagens diferentes para considerar se a página tinha um `<title>`.

O único lugar onde podemos usar a palavra-chave `await` é em funções ou blocos async, e Rust não nos deixará marcar a função especial `main` como `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

A razão pela qual `main` não pode ser marcada como `async` é que o código async precisa de um _runtime_: um crate Rust que gerencia os detalhes de execução de código assíncrono. A função `main` de um programa pode _inicializar_ um runtime, mas não é um runtime _ela mesma_. (Veremos mais sobre por que esse é o caso em breve.) Todo programa Rust que executa código async tem pelo menos um lugar onde configura um runtime que executa os futures.

A maioria das linguagens que suportam async inclui um runtime, mas Rust não. Em vez disso, existem muitos runtimes async diferentes disponíveis, cada um dos quais faz compensações diferentes adequadas ao caso de uso que visa. Por exemplo, um servidor web de alto rendimento com muitos núcleos de CPU e uma grande quantidade de RAM tem necessidades muito diferentes de um microcontrolador com um único núcleo, uma pequena quantidade de RAM e sem capacidade de alocação de heap. Os crates que fornecem esses runtimes também frequentemente fornecem versões async de funcionalidades comuns, como E/S de arquivo ou rede.

Aqui, e ao longo do resto deste capítulo, usaremos a função `block_on` do crate `trpl`, que recebe um future como argumento e bloqueia a thread atual até que este future rode até a conclusão. Por trás dos panos, chamar `block_on` configura um runtime usando o crate `tokio` que é usado para rodar o future passado (o comportamento de `block_on` do crate `trpl` é semelhante às funções `block_on` de outros crates de runtime). Uma vez que o future completa, `block_on` retorna qualquer valor que o future produziu.

Poderíamos passar o future retornado por `page_title` diretamente para `block_on` e, uma vez que ele completasse, poderíamos fazer match no `Option<String>` resultante como tentamos fazer na Listagem 17-3. No entanto, para a maioria dos exemplos no capítulo (e a maioria dos códigos async no mundo real), faremos mais do que apenas uma chamada de função async, então, em vez disso, passaremos um bloco `async` e aguardaremos explicitamente o resultado da chamada `page_title`, como na Listagem 17-4.

<Listing number="17-4" caption="Aguardando um bloco async com `trpl::block_on`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Quando rodamos este código, obtemos o comportamento que esperávamos inicialmente:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Ufa — finalmente temos algum código async funcionando! Mas antes de adicionarmos o código para fazer a corrida entre dois sites, vamos brevemente voltar nossa atenção para como futures funcionam.

Cada _ponto de await_ — isto é, cada lugar onde o código usa a palavra-chave `await` — representa um lugar onde o controle é devolvido ao runtime. Para fazer isso funcionar, Rust precisa acompanhar o estado envolvido no bloco async para que o runtime possa iniciar algum outro trabalho e depois voltar quando estiver pronto para tentar avançar o primeiro novamente. Esta é uma máquina de estado invisível, como se você tivesse escrito um enum como este para salvar o estado atual em cada ponto de await:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Escrever o código para transitar entre cada estado à mão seria tedioso e propenso a erros, no entanto, especialmente quando você precisa adicionar mais funcionalidade e mais estados ao código mais tarde. Felizmente, o compilador Rust cria e gerencia as estruturas de dados da máquina de estado para código async automaticamente. As regras normais de empréstimo e posse em torno de estruturas de dados ainda se aplicam, e felizmente, o compilador também lida com a verificação delas para nós e fornece mensagens de erro úteis. Trabalharemos em algumas delas mais tarde no capítulo.

Em última análise, algo tem que executar essa máquina de estado, e esse algo é um runtime. (É por isso que você pode encontrar menções a _executores_ ao pesquisar sobre runtimes: um executor é a parte de um runtime responsável por executar o código async.)

Agora você pode ver por que o compilador nos impediu de tornar `main` ela mesma uma função async na Listagem 17-3. Se `main` fosse uma função async, outra coisa precisaria gerenciar a máquina de estado para qualquer future que `main` retornasse, mas `main` é o ponto de partida para o programa! Em vez disso, chamamos a função `trpl::block_on` em `main` para configurar um runtime e rodar o future retornado pelo bloco `async` até que ele termine.

> Nota: Alguns runtimes fornecem macros para que você _possa_ escrever uma função `main` async. Essas macros reescrevem `async fn main() { ... }` para ser uma `fn main` normal, que faz a mesma coisa que fizemos à mão na Listagem 17-4: chamar uma função que roda um future até a conclusão da maneira que `trpl::block_on` faz.

Agora vamos juntar essas peças e ver como podemos escrever código concorrente.

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### Correndo Duas URLs Uma Contra a Outra Concorrentemente

Na Listagem 17-5, chamamos `page_title` com duas URLs diferentes passadas pela linha de comando e fazemos uma corrida entre elas selecionando qualquer future que terminar primeiro.

<Listing number="17-5" caption="Chamando `page_title` para duas URLs para ver qual retorna primeiro" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Começamos chamando `page_title` para cada uma das URLs fornecidas pelo usuário. Salvamos os futures resultantes como `title_fut_1` e `title_fut_2`. Lembre-se, estes não fazem nada ainda, porque futures são preguiçosos e ainda não os aguardamos. Então passamos os futures para `trpl::select`, que retorna um valor para indicar qual dos futures passados para ele termina primeiro.

> Nota: Por baixo dos panos, `trpl::select` é construído sobre uma função `select` mais geral definida no crate `futures`. A função `select` do crate `futures` pode fazer muitas coisas que a função `trpl::select` não pode, mas também tem alguma complexidade adicional que podemos pular por agora.

Qualquer future pode legitimamente “ganhar”, então não faz sentido retornar um `Result`. Em vez disso, `trpl::select` retorna um tipo que não vimos antes, `trpl::Either`. O tipo `Either` é um pouco semelhante a um `Result` no sentido de que tem dois casos. Diferente de `Result`, no entanto, não há noção de sucesso ou falha embutida em `Either`. Em vez disso, ele usa `Left` (Esquerda) e `Right` (Direita) para indicar “um ou o outro”:

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

A função `select` retorna `Left` com a saída daquele future se o primeiro argumento ganhar, e `Right` com a saída do segundo argumento future se _aquele_ ganhar. Isso corresponde à ordem em que os argumentos aparecem ao chamar a função: o primeiro argumento está à esquerda do segundo argumento.

Também atualizamos `page_title` para retornar a mesma URL passada. Dessa forma, se a página que retornar primeiro não tiver um `<title>` que possamos resolver, ainda podemos imprimir uma mensagem significativa. Com essa informação disponível, concluímos atualizando nossa saída `println!` para indicar tanto qual URL terminou primeiro quanto qual, se houver, é o `<title>` para a página da web naquela URL.

Você construiu um pequeno web scraper funcional agora! Escolha algumas URLs e rode a ferramenta de linha de comando. Você pode descobrir que alguns sites são consistentemente mais rápidos que outros, enquanto em outros casos o site mais rápido varia de execução para execução. Mais importante, você aprendeu o básico de trabalhar com futures, então agora podemos cavar mais fundo no que podemos fazer com async.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
