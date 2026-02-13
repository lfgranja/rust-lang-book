<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## Aplicando Concorrência com Async

Nesta seção, aplicaremos async a alguns dos mesmos desafios de concorrência que abordamos com threads no Capítulo 16. Como já falamos sobre muitas das ideias principais lá, nesta seção focaremos no que é diferente entre threads e futures.

Em muitos casos, as APIs para trabalhar com concorrência usando async são muito semelhantes às de uso de threads. Em outros casos, elas acabam sendo bastante diferentes. Mesmo quando as APIs _parecem_ semelhantes entre threads e async, elas frequentemente têm comportamento diferente — e quase sempre têm características de desempenho diferentes.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Criando uma Nova Tarefa com `spawn_task`

A primeira operação que abordamos na seção [“Criando uma Nova Thread com `spawn`”][thread-spawn]<!-- ignore --> no Capítulo 16 foi contar em duas threads separadas. Vamos fazer o mesmo usando async. O crate `trpl` fornece uma função `spawn_task` que se parece muito com a API `thread::spawn`, e uma função `sleep` que é uma versão async da API `thread::sleep`. Podemos usá-las juntas para implementar o exemplo de contagem, como mostrado na Listagem 17-6.

<Listing number="17-6" caption="Criando uma nova tarefa para imprimir uma coisa enquanto a tarefa principal imprime outra coisa" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Como nosso ponto de partida, configuramos nossa função `main` com `trpl::block_on` para que nossa função de nível superior possa ser async.

> Nota: Deste ponto em diante no capítulo, todo exemplo incluirá este exatamente mesmo código de envolvimento com `trpl::block_on` em `main`, então frequentemente o pularemos, assim como fazemos com `main`. Lembre-se de incluí-lo em seu código!

Então escrevemos dois loops dentro desse bloco, cada um contendo uma chamada `trpl::sleep`, que espera por meio segundo (500 milissegundos) antes de enviar a próxima mensagem. Colocamos um loop no corpo de um `trpl::spawn_task` e o outro em um loop `for` de nível superior. Também adicionamos um `await` após as chamadas `sleep`.

Este código se comporta de forma semelhante à implementação baseada em threads — incluindo o fato de que você pode ver as mensagens aparecerem em uma ordem diferente no seu próprio terminal quando rodá-lo:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Esta versão para assim que o loop `for` no corpo do bloco async principal termina, porque a tarefa criada por `spawn_task` é encerrada quando a função `main` termina. Se você quiser que ela rode até a conclusão da tarefa, precisará usar um join handle para esperar a primeira tarefa completar. Com threads, usamos o método `join` para “bloquear” até que a thread tivesse terminado de rodar. Na Listagem 17-7, podemos usar `await` para fazer a mesma coisa, porque o próprio identificador da tarefa é um future. Seu tipo `Output` é um `Result`, então também o desembrulhamos (unwrap) após aguardá-lo.

<Listing number="17-7" caption="Usando `await` com um join handle para rodar uma tarefa até a conclusão" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Esta versão atualizada roda até que _ambos_ os loops terminem:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Até agora, parece que async e threads nos dão resultados semelhantes, apenas com sintaxe diferente: usando `await` em vez de chamar `join` no join handle, e aguardando as chamadas `sleep`.

A maior diferença é que não precisamos criar outra thread do sistema operacional para fazer isso. Na verdade, nem precisamos criar uma tarefa aqui. Como blocos async compilam para futures anônimos, podemos colocar cada loop em um bloco async e fazer o runtime rodar ambos até a conclusão usando a função `trpl::join`.

Na seção [“Esperando Todas as Threads Terminarem”][join-handles]<!-- ignore --> no Capítulo 16, mostramos como usar o método `join` no tipo `JoinHandle` retornado quando você chama `std::thread::spawn`. A função `trpl::join` é semelhante, mas para futures. Quando você dá a ela dois futures, ela produz um único novo future cuja saída é uma tupla contendo a saída de cada future que você passou, uma vez que _ambos_ completem. Assim, na Listagem 17-8, usamos `trpl::join` para esperar que tanto `fut1` quanto `fut2` terminem. Nós _não_ aguardamos `fut1` e `fut2`, mas sim o novo future produzido por `trpl::join`. Ignoramos a saída, porque é apenas uma tupla contendo dois valores unitários.

<Listing number="17-8" caption="Usando `trpl::join` para aguardar dois futures anônimos" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Quando rodamos isso, vemos ambos os futures rodarem até a conclusão:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Agora, você verá a exata mesma ordem toda vez, o que é muito diferente do que vimos com threads e com `trpl::spawn_task` na Listagem 17-7. Isso é porque a função `trpl::join` é _justa_ (fair), significando que ela verifica cada future com igual frequência, alternando entre eles, e nunca deixa um correr à frente se o outro estiver pronto. Com threads, o sistema operacional decide qual thread verificar e por quanto tempo deixá-la rodar. Com Rust async, o runtime decide qual tarefa verificar. (Na prática, os detalhes ficam complicados porque um runtime async pode usar threads do sistema operacional por baixo dos panos como parte de como gerencia a concorrência, então garantir justiça pode ser mais trabalho para um runtime — mas ainda é possível!) Runtimes não precisam garantir justiça para qualquer operação dada, e eles frequentemente oferecem APIs diferentes para deixar você escolher se quer ou não justiça.

Tente algumas dessas variações ao aguardar os futures e veja o que elas fazem:

- Remova o bloco async de volta de um ou ambos os loops.
- Aguarde cada bloco async imediatamente após defini-lo.
- Envolva apenas o primeiro loop em um bloco async, e aguarde o future resultante após o corpo do segundo loop.

Para um desafio extra, veja se consegue descobrir qual será a saída em cada caso _antes_ de rodar o código!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### Enviando Dados Entre Duas Tarefas Usando Passagem de Mensagem

Compartilhar dados entre futures também será familiar: usaremos passagem de mensagem novamente, mas desta vez com versões async dos tipos e funções. Tomaremos um caminho ligeiramente diferente do que fizemos na seção [“Transferir Dados Entre Threads com Passagem de Mensagem”][message-passing-threads]<!-- ignore --> no Capítulo 16 para ilustrar algumas das principais diferenças entre concorrência baseada em threads e baseada em futures. Na Listagem 17-9, começaremos com apenas um único bloco async — _não_ criando uma tarefa separada como criamos uma thread separada.

<Listing number="17-9" caption="Criando um canal async e atribuindo as duas metades a `tx` e `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Aqui, usamos `trpl::channel`, uma versão async da API de canal de múltiplos produtores e único consumidor que usamos com threads no Capítulo 16. A versão async da API é apenas um pouco diferente da versão baseada em threads: ela usa um receiver `rx` mutável em vez de imutável, e seu método `recv` produz um future que precisamos aguardar em vez de produzir o valor diretamente. Agora podemos enviar mensagens do remetente para o receptor. Note que não precisamos criar uma thread separada ou mesmo uma tarefa; precisamos apenas aguardar a chamada `rx.recv`.

O método síncrono `Receiver::recv` em `std::mpsc::channel` bloqueia até que receba uma mensagem. O método `trpl::Receiver::recv` não bloqueia, porque é async. Em vez de bloquear, ele devolve o controle ao runtime até que uma mensagem seja recebida ou o lado de envio do canal feche. Em contraste, não aguardamos a chamada `send`, porque ela não bloqueia. Ela não precisa, porque o canal para o qual estamos enviando é ilimitado (unbounded).

> Nota: Como todo esse código async roda em um bloco async em uma chamada `trpl::block_on`, tudo dentro dele pode evitar bloquear. No entanto, o código _fora_ dele bloqueará no retorno da função `block_on`. Esse é todo o ponto da função `trpl::block_on`: ela permite que você _escolha_ onde bloquear em algum conjunto de código async, e assim onde transitar entre código sync e async.

Note duas coisas sobre este exemplo. Primeiro, a mensagem chegará imediatamente. Segundo, embora usemos um future aqui, não há concorrência ainda. Tudo na listagem acontece em sequência, exatamente como aconteceria se não houvesse futures envolvidos.

Vamos abordar a primeira parte enviando uma série de mensagens e dormindo entre elas, como mostrado na Listagem 17-10.

<!-- We cannot test this one porque ele nunca para! -->

<Listing number="17-10" caption="Enviando e recebendo múltiplas mensagens pelo canal async e dormindo com um `await` entre cada mensagem" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Além de enviar as mensagens, precisamos recebê-las. Neste caso, como sabemos quantas mensagens estão chegando, poderíamos fazer isso manualmente chamando `rx.recv().await` quatro vezes. No mundo real, porém, geralmente estaremos esperando por algum número _desconhecido_ de mensagens, então precisamos continuar esperando até determinarmos que não há mais mensagens.

Na Listagem 16-10, usamos um loop `for` para processar todos os itens recebidos de um canal síncrono. Rust ainda não tem uma maneira de usar um loop `for` com uma série de itens _produzidos assincronamente_, no entanto, então precisamos usar um loop que não vimos antes: o loop condicional `while let`. Esta é a versão em loop do construto `if let` que vimos na seção [“Controle de Fluxo Conciso com `if let` e `let...else`”][if-let]<!-- ignore --> no Capítulo 6. O loop continuará executando enquanto o padrão que ele especifica continuar correspondendo ao valor.

A chamada `rx.recv` produz um future, que aguardamos. O runtime pausará o future até que ele esteja pronto. Uma vez que uma mensagem chegue, o future resolverá para `Some(mensagem)` tantas vezes quanto uma mensagem chegar. Quando o canal fechar, independentemente de _alguma_ mensagem ter chegado, o future resolverá para `None` para indicar que não há mais valores e, portanto, devemos parar de sondar (polling) — isto é, parar de aguardar.

O loop `while let` junta tudo isso. Se o resultado de chamar `rx.recv().await` for `Some(mensagem)`, obtemos acesso à mensagem e podemos usá-la no corpo do loop, assim como poderíamos com `if let`. Se o resultado for `None`, o loop termina. Toda vez que o loop completa, ele atinge o ponto de await novamente, então o runtime o pausa novamente até que outra mensagem chegue.

O código agora envia e recebe com sucesso todas as mensagens. Infelizmente, ainda existem alguns problemas. Por um lado, as mensagens não chegam em intervalos de meio segundo. Elas chegam todas de uma vez, 2 segundos (2.000 milissegundos) depois de iniciarmos o programa. Por outro lado, este programa também nunca termina! Em vez disso, ele espera para sempre por novas mensagens. Você precisará encerrá-lo usando <kbd>ctrl</kbd>-<kbd>C</kbd>.

#### Código Dentro de Um Bloco Async Executa Linearmente

Vamos começar examinando por que as mensagens chegam todas de uma vez após o atraso total, em vez de chegarem com atrasos entre cada uma. Dentro de um determinado bloco async, a ordem em que as palavras-chave `await` aparecem no código é também a ordem em que são executadas quando o programa roda.

Há apenas um bloco async na Listagem 17-10, então tudo nele roda linearmente. Ainda não há concorrência. Todas as chamadas `tx.send` acontecem, intercaladas com todas as chamadas `trpl::sleep` e seus pontos de await associados. Só então o loop `while let` consegue passar por qualquer um dos pontos de `await` nas chamadas `recv`.

Para obter o comportamento que queremos, onde o atraso de sono acontece entre cada mensagem, precisamos colocar as operações `tx` e `rx` em seus próprios blocos async, como mostrado na Listagem 17-11. Então o runtime pode executar cada um deles separadamente usando `trpl::join`, exatamente como na Listagem 17-8. Mais uma vez, aguardamos o resultado de chamar `trpl::join`, não os futures individuais. Se aguardássemos os futures individuais em sequência, acabaríamos de volta em um fluxo sequencial — exatamente o que estamos tentando _não_ fazer.

<!-- We cannot test this one porque ele nunca para! -->

<Listing number="17-11" caption="Separando `send` e `recv` em seus próprios blocos `async` e aguardando os futures para esses blocos" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Com o código atualizado na Listagem 17-11, as mensagens são impressas em intervalos de 500 milissegundos, em vez de todas de uma vez após 2 segundos.

#### Movendo Posse Para Dentro de um Bloco Async

O programa ainda nunca termina, no entanto, por causa da maneira como o loop `while let` interage com `trpl::join`:

- O future retornado de `trpl::join` completa apenas quando _ambos_ os futures passados para ele completam.
- O future `tx_fut` completa uma vez que termina de dormir após enviar a última mensagem em `vals`.
- O future `rx_fut` não completará até que o loop `while let` termine.
- O loop `while let` não terminará até que aguardar `rx.recv` produza `None`.
- Aguardar `rx.recv` retornará `None` apenas uma vez que a outra ponta do canal for fechada.
- O canal fechará apenas se chamarmos `rx.close` ou quando o lado do remetente, `tx`, for descartado (dropped).
- Não chamamos `rx.close` em lugar nenhum, e `tx` não será descartado até que o bloco async mais externo passado para `trpl::block_on` termine.
- O bloco não pode terminar porque está bloqueado em `trpl::join` completando, o que nos leva de volta ao topo desta lista.

No momento, o bloco async onde enviamos as mensagens apenas _toma emprestado_ `tx` porque enviar uma mensagem não requer posse, mas se pudéssemos _mover_ `tx` para dentro desse bloco async, ele seria descartado uma vez que esse bloco terminasse. Na seção [“Capturando Referências ou Movendo Posse”][capture-or-move]<!-- ignore --> no Capítulo 13, você aprendeu como usar a palavra-chave `move` com closures e, como discutido na seção [“Usando Closures `move` com Threads”][move-threads]<!-- ignore --> no Capítulo 16, frequentemente precisamos mover dados para dentro de closures quando trabalhamos com threads. A mesma dinâmica básica se aplica a blocos async, então a palavra-chave `move` funciona com blocos async exatamente como faz com closures.

Na Listagem 17-12, mudamos o bloco usado para enviar mensagens de `async` para `async move`.

<Listing number="17-12" caption="Uma revisão do código da Listagem 17-11 que encerra corretamente quando completo" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Quando rodamos _esta_ versão do código, ela encerra graciosamente após a última mensagem ser enviada e recebida. A seguir, vamos ver o que precisaria mudar para enviar dados de mais de um future.

#### Juntando um Número de Futures com a Macro `join!`

Este canal async também é um canal de múltiplos produtores, então podemos chamar `clone` em `tx` se quisermos enviar mensagens de múltiplos futures, como mostrado na Listagem 17-13.

<Listing number="17-13" caption="Usando múltiplos produtores com blocos async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Primeiro, clonamos `tx`, criando `tx1` fora do primeiro bloco async. Movemos `tx1` para dentro desse bloco assim como fizemos antes com `tx`. Então, mais tarde, movemos o `tx` original para dentro de um _novo_ bloco async, onde enviamos mais mensagens com um atraso ligeiramente mais lento. Acontece que colocamos este novo bloco async após o bloco async para receber mensagens, mas ele poderia ir antes dele igualmente bem. A chave é a ordem em que os futures são aguardados, não em que são criados.

Ambos os blocos async para enviar mensagens precisam ser blocos `async move` para que tanto `tx` quanto `tx1` sejam descartados quando esses blocos terminarem. Caso contrário, acabaremos de volta no mesmo loop infinito em que começamos.

Finalmente, mudamos de `trpl::join` para `trpl::join!` para lidar com o future adicional: a macro `join!` aguarda um número arbitrário de futures onde sabemos o número de futures em tempo de compilação. Discutiremos aguardar uma coleção de um número desconhecido de futures mais tarde neste capítulo.

Agora vemos todas as mensagens de ambos os futures de envio, e como os futures de envio usam atrasos ligeiramente diferentes após enviar, as mensagens também são recebidas nesses intervalos diferentes:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Exploramos como usar passagem de mensagem para enviar dados entre futures, como código dentro de um bloco async roda sequencialmente, como mover posse para dentro de um bloco async e como juntar múltiplos futures. A seguir, vamos discutir como e por que dizer ao runtime que ele pode mudar para outra tarefa.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
