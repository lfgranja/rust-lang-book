## Erros Irrecuperáveis com `panic!`

Às vezes, coisas ruins acontecem em seu código e não há nada que você possa
fazer a respeito. Nesses casos, Rust tem a macro `panic!`. Existem duas maneiras
de causar um pânico na prática: tomando uma ação que faz com que nosso código
entre em pânico (como acessar um array além do final) ou chamando explicitamente
a macro `panic!`. Em ambos os casos, causamos um pânico em nosso programa. Por
padrão, esses pânicos imprimirão uma mensagem de falha, desenrolarão a pilha
(unwind), limparão a pilha e encerrarão. Através de uma variável de ambiente,
você também pode fazer com que Rust exiba a pilha de chamadas (call stack)
quando ocorrer um pânico para facilitar o rastreamento da origem do pânico.

> ### Desenrolando a Pilha ou Abortando em Resposta a um Pânico
>
> Por padrão, quando ocorre um pânico, o programa começa a _desenrolar_, o que
> significa que Rust volta a subir na pilha e limpa os dados de cada função que
> encontra. No entanto, voltar e limpar é muito trabalhoso. Rust, portanto,
> permite que você escolha a alternativa de _abortar_ imediatamente, o que
> encerra o programa sem limpar.
>
> A memória que o programa estava usando precisará então ser limpa pelo sistema
> operacional. Se em seu projeto você precisa tornar o binário resultante o
> menor possível, você pode mudar de desenrolar para abortar em um pânico
> adicionando `panic = 'abort'` às seções `[profile]` apropriadas em seu arquivo
> _Cargo.toml_. Por exemplo, se você quiser abortar em caso de pânico no modo
> release, adicione isto:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Vamos tentar chamar `panic!` em um programa simples:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Quando você executar o programa, verá algo como isto:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

A chamada para `panic!` causa a mensagem de erro contida nas duas últimas
linhas. A primeira linha mostra nossa mensagem de pânico e o local em nosso
código-fonte onde o pânico ocorreu: _src/main.rs:2:5_ indica que é a segunda
linha, quinto caractere do nosso arquivo _src/main.rs_.

Neste caso, a linha indicada faz parte do nosso código, e se formos para essa
linha, vemos a chamada da macro `panic!`. Em outros casos, a chamada `panic!`
pode estar no código que nosso código chama, e o nome do arquivo e o número da
linha relatados pela mensagem de erro serão o código de outra pessoa onde a
macro `panic!` é chamada, não a linha do nosso código que acabou levando à
chamada `panic!`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Podemos usar o backtrace das funções de onde veio a chamada `panic!` para
descobrir a parte do nosso código que está causando o problema. Para entender
como usar um backtrace de `panic!`, vamos ver outro exemplo e ver como é quando
uma chamada de `panic!` vem de uma biblioteca por causa de um bug em nosso
código em vez de nosso código chamar a macro diretamente. A Listagem 9-1 tem
algum código que tenta acessar um índice em um vetor além do intervalo de
índices válidos.

<Listing number="9-1" file-name="src/main.rs" caption="Tentando acessar um elemento além do final de um vetor, o que causará uma chamada para `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Aqui, estamos tentando acessar o 100º elemento do nosso vetor (que está no
índice 99 porque a indexação começa em zero), mas o vetor tem apenas três
elementos. Nesta situação, Rust entrará em pânico. O uso de `[]` deve retornar
um elemento, mas se você passar um índice inválido, não há elemento que Rust
possa retornar aqui que seria correto.

Em C, tentar ler além do final de uma estrutura de dados é um comportamento
indefinido. Você pode obter o que estiver no local da memória que corresponderia
a esse elemento na estrutura de dados, mesmo que a memória não pertença a essa
estrutura. Isso é chamado de _buffer overread_ e pode levar a vulnerabilidades
de segurança se um invasor for capaz de manipular o índice de tal forma a ler
dados que não deveriam ser permitidos e que estão armazenados após a estrutura
de dados.

Para proteger seu programa desse tipo de vulnerabilidade, se você tentar ler um
elemento em um índice que não existe, Rust interromperá a execução e se recusará
a continuar. Vamos tentar e ver:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Este erro aponta para a linha 4 do nosso _main.rs_ onde tentamos acessar o
índice 99 do vetor em `v`.

A linha `note:` nos diz que podemos definir a variável de ambiente
`RUST_BACKTRACE` para obter um backtrace de exatamente o que aconteceu para
causar o erro. Um _backtrace_ é uma lista de todas as funções que foram
chamadas para chegar a este ponto. Backtraces em Rust funcionam como em outras
linguagens: A chave para ler o backtrace é começar do topo e ler até ver
arquivos que você escreveu. Esse é o ponto onde o problema se originou. As
linhas acima desse ponto são códigos que seu código chamou; as linhas abaixo
são códigos que chamaram seu código. Essas linhas antes e depois podem incluir
código Rust principal, código da biblioteca padrão ou crates que você está
usando. Vamos tentar obter um backtrace definindo a variável de ambiente
`RUST_BACKTRACE` para qualquer valor exceto `0`. A Listagem 9-2 mostra uma
saída semelhante à que você verá.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="O backtrace gerado por uma chamada para `panic!` exibido quando a variável de ambiente `RUST_BACKTRACE` está definida">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

Isso é muita saída! A saída exata que você vê pode ser diferente dependendo do
seu sistema operacional e versão do Rust. Para obter backtraces com essas
informações, os símbolos de depuração devem estar habilitados. Símbolos de
depuração são habilitados por padrão ao usar `cargo build` ou `cargo run` sem a
flag `--release`, como temos aqui.

Na saída da Listagem 9-2, a linha 6 do backtrace aponta para a linha em nosso
projeto que está causando o problema: linha 4 de _src/main.rs_. Se não queremos
que nosso programa entre em pânico, devemos começar nossa investigação no local
apontado pela primeira linha mencionando um arquivo que escrevemos. Na Listagem
9-1, onde escrevemos deliberadamente um código que entraria em pânico, a
maneira de corrigir o pânico é não solicitar um elemento além do intervalo dos
índices do vetor. Quando seu código entrar em pânico no futuro, você precisará
descobrir qual ação o código está tomando com quais valores para causar o
pânico e o que o código deve fazer em vez disso.

Voltaremos ao `panic!` e quando devemos e não devemos usar `panic!` para lidar
com condições de erro na seção [“Entrar em `panic!` ou Não Entrar em
`panic!`”][to-panic-or-not-to-panic]<!-- ignore --> mais adiante neste
capítulo. A seguir, veremos como se recuperar de um erro usando `Result`.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
