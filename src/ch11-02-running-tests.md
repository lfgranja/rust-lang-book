## Controlando Como os Testes São Executados

Assim como `cargo run` compila seu código e depois executa o binário resultante,
`cargo test` compila seu código em modo de teste e executa o binário de teste
resultante. O comportamento padrão do binário produzido por `cargo test` é
executar todos os testes em paralelo e capturar a saída gerada durante as
execuções de teste, impedindo que a saída seja exibida e facilitando a leitura
da saída relacionada aos resultados do teste. Você pode, no entanto,
especificar opções de linha de comando para alterar esse comportamento padrão.

Algumas opções de linha de comando vão para `cargo test`, e algumas vão para o
binário de teste resultante. Para separar esses dois tipos de argumentos, você
lista os argumentos que vão para `cargo test` seguidos pelo separador `--` e
depois os que vão para o binário de teste. Executar `cargo test --help` exibe
as opções que você pode usar com `cargo test`, e executar
`cargo test -- --help` exibe as opções que você pode usar após o separador.
Essas opções também estão documentadas na [seção “Testes” do _The `rustc`
Book_][tests].

[tests]: https://doc.rust-lang.org/rustc/tests/index.html

### Executando Testes em Paralelo ou Consecutivamente

Quando você executa vários testes, por padrão eles são executados em paralelo
usando threads, o que significa que eles terminam de ser executados mais
rapidamente e você obtém feedback mais cedo. Como os testes estão sendo
executados ao mesmo tempo, você deve garantir que seus testes não dependam uns
dos outros ou de qualquer estado compartilhado, incluindo um ambiente
compartilhado, como o diretório de trabalho atual ou variáveis de ambiente.

Por exemplo, digamos que cada um dos seus testes execute algum código que cria
um arquivo no disco chamado _test-output.txt_ e grava alguns dados nesse
arquivo. Então, cada teste lê os dados nesse arquivo e afirma que o arquivo
contém um valor específico, que é diferente em cada teste. Como os testes são
executados ao mesmo tempo, um teste pode sobrescrever o arquivo no tempo entre
quando outro teste está gravando e lendo o arquivo. O segundo teste falhará
então, não porque o código está incorreto, mas porque os testes interferiram
uns nos outros durante a execução em paralelo. Uma solução é garantir que cada
teste grave em um arquivo diferente; outra solução é executar os testes um de
cada vez.

Se você não quiser executar os testes em paralelo ou se quiser um controle mais
refinado sobre o número de threads usadas, pode enviar a flag `--test-threads`
e o número de threads que deseja usar para o binário de teste. Dê uma olhada no
seguinte exemplo:

```console
$ cargo test -- --test-threads=1
```

Definimos o número de threads de teste como `1`, dizendo ao programa para não
usar nenhum paralelismo. Executar os testes usando uma thread levará mais tempo
do que executá-los em paralelo, mas os testes não interferirão uns nos outros
se compartilharem estado.

### Mostrando a Saída da Função

Por padrão, se um teste passar, a biblioteca de teste do Rust captura qualquer
coisa impressa na saída padrão. Por exemplo, se chamarmos `println!` em um
teste e o teste passar, não veremos a saída de `println!` no terminal; veremos
apenas a linha que indica que o teste passou. Se um teste falhar, veremos o que
foi impresso na saída padrão com o restante da mensagem de falha.

Como exemplo, a Listagem 11-10 tem uma função boba que imprime o valor de seu
parâmetro e retorna 10, bem como um teste que passa e um teste que falha.

<Listing number="11-10" file-name="src/lib.rs" caption="Testes para uma função que chama `println!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Quando executarmos esses testes com `cargo test`, veremos a seguinte saída:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Observe que em nenhum lugar desta saída vemos `I got the value 4`, que é
impresso quando o teste que passa é executado. Essa saída foi capturada. A
saída do teste que falhou, `I got the value 8`, aparece na seção do resumo do
teste, que também mostra a causa da falha do teste.

Se quisermos ver os valores impressos para testes aprovados também, podemos
dizer ao Rust para também mostrar a saída de testes bem-sucedidos com
`--show-output`:

```console
$ cargo test -- --show-output
```

Quando executamos os testes na Listagem 11-10 novamente com a flag
`--show-output`, vemos a seguinte saída:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Executando um Subconjunto de Testes por Nome

Executar um conjunto completo de testes às vezes pode levar muito tempo. Se você
estiver trabalhando em código em uma área específica, pode querer executar
apenas os testes pertencentes a esse código. Você pode escolher quais testes
executar passando para `cargo test` o nome ou nomes do(s) teste(s) que deseja
executar como argumento.

Para demonstrar como executar um subconjunto de testes, primeiro criaremos três
testes para nossa função `add_two`, conforme mostrado na Listagem 11-11, e
escolheremos quais executar.

<Listing number="11-11" file-name="src/lib.rs" caption="Três testes com três nomes diferentes">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Se executarmos os testes sem passar nenhum argumento, como vimos anteriormente,
todos os testes serão executados em paralelo:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Executando Testes Únicos

Podemos passar o nome de qualquer função de teste para `cargo test` para
executar apenas esse teste:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Apenas o teste com o nome `one_hundred` foi executado; os outros dois testes
não corresponderam a esse nome. A saída do teste nos informa que tivemos mais
testes que não foram executados exibindo `2 filtered out` no final.

Não podemos especificar os nomes de vários testes dessa maneira; apenas o
primeiro valor dado a `cargo test` será usado. Mas há uma maneira de executar
vários testes.

#### Filtrando para Executar Vários Testes

Podemos especificar parte de um nome de teste, e qualquer teste cujo nome
corresponda a esse valor será executado. Por exemplo, como dois dos nomes dos
nossos testes contêm `add`, podemos executar esses dois executando
`cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Este comando executou todos os testes com `add` no nome e filtrou o teste
chamado `one_hundred`. Observe também que o módulo em que um teste aparece se
torna parte do nome do teste, portanto, podemos executar todos os testes em um
módulo filtrando pelo nome do módulo.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-some-tests-unless-specifically-requested"></a>

### Ignorando Testes A Menos Que Solicitados Especificamente

Às vezes, alguns testes específicos podem demorar muito para serem executados,
então você pode querer excluí-los durante a maioria das execuções de
`cargo test`. Em vez de listar como argumentos todos os testes que você deseja
executar, você pode anotar os testes demorados usando o atributo `ignore` para
excluí-los, como mostrado aqui:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

Após `#[test]`, adicionamos a linha `#[ignore]` ao teste que queremos excluir.
Agora, quando executamos nossos testes, `it_works` é executado, mas
`expensive_test` não:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

A função `expensive_test` é listada como `ignored`. Se quisermos executar
apenas os testes ignorados, podemos usar `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Ao controlar quais testes são executados, você pode garantir que seus
resultados de `cargo test` sejam retornados rapidamente. Quando você estiver em
um ponto em que faz sentido verificar os resultados dos testes `ignored` e tiver
tempo para esperar pelos resultados, poderá executar `cargo test -- --ignored`
em vez disso. Se você quiser executar todos os testes, ignorados ou não, pode
executar `cargo test -- --include-ignored`.
