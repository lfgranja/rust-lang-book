<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## Transferir Dados Entre Threads com Passagem de Mensagem

Uma abordagem cada vez mais popular para garantir concorrência segura é a *passagem de mensagem*, onde threads ou atores se comunicam enviando mensagens uns aos outros contendo dados. Aqui está a ideia em um slogan da [documentação da linguagem Go](https://golang.org/doc/effective_go.html#concurrency): "Não se comunique compartilhando memória; em vez disso, compartilhe memória comunicando-se."

Para realizar a concorrência por envio de mensagens, a biblioteca padrão de Rust fornece uma implementação de *canais* (channels). Um canal é um conceito geral de programação pelo qual dados são enviados de uma thread para outra.

Você pode imaginar um canal em programação como sendo como um canal direcional de água, como um córrego ou um rio. Se você colocar algo como um pato de borracha em um rio, ele viajará rio abaixo até o fim do curso d'água.

Um canal tem duas metades: um transmissor e um receptor. A metade transmissora é o local a montante onde você coloca o pato de borracha no rio, e a metade receptora é onde o pato de borracha termina a jusante. Uma parte do seu código chama métodos no transmissor com os dados que você quer enviar, e outra parte verifica a extremidade receptora para mensagens chegando. Um canal é dito estar *fechado* se a metade transmissora ou receptora for descartada.

Aqui, trabalharemos em um programa que tem uma thread para gerar valores e enviá-los por um canal, e outra thread que receberá os valores e os imprimirá. Enviaremos valores simples entre threads usando um canal para ilustrar o recurso. Uma vez que você esteja familiarizado com a técnica, você poderia usar canais para quaisquer threads que precisem se comunicar entre si, como um sistema de chat ou um sistema onde muitas threads executam partes de um cálculo e enviam as partes para uma thread que agrega os resultados.

Primeiro, na Listagem 16-6, criaremos um canal, mas não faremos nada com ele. Note que isso não compilará ainda porque Rust não pode dizer que tipo de valores queremos enviar pelo canal.

<Listing number="16-6" file-name="src/main.rs" caption="Criando um canal e atribuindo as duas metades a `tx` e `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

Criamos um novo canal usando a função `mpsc::channel`; `mpsc` significa *multiple producer, single consumer* (múltiplos produtores, consumidor único). Resumindo, a maneira como a biblioteca padrão de Rust implementa canais significa que um canal pode ter múltiplas extremidades de *envio* que produzem valores, mas apenas uma extremidade de *recebimento* que consome esses valores. Imagine múltiplos córregos fluindo juntos em um grande rio: tudo o que for enviado por qualquer um dos córregos acabará em um rio no final. Começaremos com um único produtor por enquanto, mas adicionaremos múltiplos produtores quando fizermos este exemplo funcionar.

A função `mpsc::channel` retorna uma tupla, o primeiro elemento da qual é a extremidade de envio — o transmissor — e o segundo elemento da qual é a extremidade de recebimento — o receptor. As abreviações `tx` e `rx` são tradicionalmente usadas em muitos campos para *transmissor* e *receptor*, respectivamente, então nomeamos nossas variáveis como tal para indicar cada extremidade. Estamos usando uma instrução `let` com um padrão que desestrutura as tuplas; discutiremos o uso de padrões em instruções `let` e desestruturação no Capítulo 18. Por enquanto, saiba que usar uma instrução `let` dessa maneira é uma abordagem conveniente para extrair as partes da tupla retornada por `mpsc::channel`.

Vamos mover a extremidade transmissora para uma thread criada e fazê-la enviar uma string para que a thread criada esteja se comunicando com a thread principal, como mostrado na Listagem 16-7. Isso é como colocar um pato de borracha no rio a montante ou enviar uma mensagem de chat de uma thread para outra.

<Listing number="16-7" file-name="src/main.rs" caption='Movendo `tx` para uma thread criada e enviando `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Novamente, estamos usando `thread::spawn` para criar uma nova thread e depois usando `move` para mover `tx` para a closure para que a thread criada possua `tx`. A thread criada precisa possuir o transmissor para poder enviar mensagens através do canal.

O transmissor tem um método `send` que recebe o valor que queremos enviar. O método `send` retorna um tipo `Result<T, E>`, então se o receptor já tiver sido descartado e não houver para onde enviar um valor, a operação de envio retornará um erro. Neste exemplo, estamos chamando `unwrap` para entrar em pânico em caso de erro. Mas em uma aplicação real, nós o trataríamos adequadamente: retorne ao Capítulo 9 para rever estratégias para tratamento de erros adequado.

Na Listagem 16-8, obteremos o valor do receptor na thread principal. Isso é como recuperar o pato de borracha da água no final do rio ou receber uma mensagem de chat.

<Listing number="16-8" file-name="src/main.rs" caption='Recebendo o valor `"hi"` na thread principal e imprimindo-o'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

O receptor tem dois métodos úteis: `recv` e `try_recv`. Estamos usando `recv`, abreviação de *receive* (receber), que bloqueará a execução da thread principal e esperará até que um valor seja enviado pelo canal. Uma vez que um valor é enviado, `recv` o retornará em um `Result<T, E>`. Quando o transmissor fecha, `recv` retornará um erro para sinalizar que não virão mais valores.

O método `try_recv` não bloqueia, mas em vez disso retornará um `Result<T, E>` imediatamente: um valor `Ok` contendo uma mensagem se houver uma disponível e um valor `Err` se não houver mensagens desta vez. Usar `try_recv` é útil se esta thread tiver outro trabalho a fazer enquanto espera por mensagens: poderíamos escrever um loop que chama `try_recv` de vez em quando, trata uma mensagem se houver uma disponível, e caso contrário faz outro trabalho por um tempo até verificar novamente.

Usamos `recv` neste exemplo por simplicidade; não temos nenhum outro trabalho para a thread principal fazer além de esperar por mensagens, então bloquear a thread principal é apropriado.

Quando rodamos o código na Listagem 16-8, veremos o valor impresso da thread principal:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Perfeito!

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### Transferindo Posse Através de Canais

As regras de posse desempenham um papel vital no envio de mensagens porque ajudam você a escrever código seguro e concorrente. Prevenir erros em programação concorrente é a vantagem de pensar sobre posse em todos os seus programas Rust. Vamos fazer um experimento para mostrar como canais e posse trabalham juntos para prevenir problemas: tentaremos usar um valor `val` na thread criada *depois* de o enviarmos pelo canal. Tente compilar o código na Listagem 16-9 para ver por que este código não é permitido.

<Listing number="16-9" file-name="src/main.rs" caption="Tentando usar `val` depois de o enviarmos pelo canal">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Aqui, tentamos imprimir `val` depois de o enviarmos pelo canal via `tx.send`. Permitir isso seria uma má ideia: uma vez que o valor foi enviado para outra thread, essa thread poderia modificá-lo ou descartá-lo antes que tentássemos usar o valor novamente. Potencialmente, as modificações da outra thread poderiam causar erros ou resultados inesperados devido a dados inconsistentes ou inexistentes. No entanto, Rust nos dá um erro se tentarmos compilar o código na Listagem 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Nosso erro de concorrência causou um erro de tempo de compilação. A função `send` toma posse de seu parâmetro, e quando o valor é movido, o receptor toma posse dele. Isso nos impede de usar acidentalmente o valor novamente depois de enviá-lo; o sistema de posse verifica se tudo está bem.

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### Enviando Múltiplos Valores

O código na Listagem 16-8 compilou e rodou, mas não nos mostrou claramente que duas threads separadas estavam conversando entre si pelo canal.

Na Listagem 16-10, fizemos algumas modificações que provarão que o código na Listagem 16-8 está rodando concorrentemente: a thread criada agora enviará múltiplas mensagens e fará uma pausa de um segundo entre cada mensagem.

<Listing number="16-10" file-name="src/main.rs" caption="Enviando múltiplas mensagens e pausando entre cada uma">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Desta vez, a thread criada tem um vetor de strings que queremos enviar para a thread principal. Iteramos sobre elas, enviando cada uma individualmente, e pausamos entre cada uma chamando a função `thread::sleep` com um valor `Duration` de um segundo.

Na thread principal, não estamos mais chamando a função `recv` explicitamente: em vez disso, estamos tratando `rx` como um iterador. Para cada valor recebido, estamos imprimindo-o. Quando o canal é fechado, a iteração terminará.

Ao rodar o código na Listagem 16-10, você deve ver a seguinte saída com uma pausa de um segundo entre cada linha:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Como não temos nenhum código que pausa ou atrasa no loop `for` na thread principal, podemos dizer que a thread principal está esperando receber valores da thread criada.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### Criando Múltiplos Produtores

Mencionamos anteriormente que `mpsc` era um acrônimo para *multiple producer, single consumer*. Vamos colocar `mpsc` em uso e expandir o código na Listagem 16-10 para criar múltiplas threads que enviam valores para o mesmo receptor. Podemos fazer isso clonando o transmissor, como mostrado na Listagem 16-11.

<Listing number="16-11" file-name="src/main.rs" caption="Enviando múltiplas mensagens de múltiplos produtores">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Desta vez, antes de criarmos a primeira thread criada, chamamos `clone` no transmissor. Isso nos dará um novo transmissor que podemos passar para a primeira thread criada. Passamos o transmissor original para uma segunda thread criada. Isso nos dá duas threads, cada uma enviando mensagens diferentes para o único receptor.

Quando você rodar o código, sua saída deve se parecer com algo assim:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Você pode ver os valores em outra ordem, dependendo do seu sistema. Isso é o que torna a concorrência interessante, bem como difícil. Se você experimentar com `thread::sleep`, dando-lhe vários valores nas diferentes threads, cada execução será mais não determinística e criará saída diferente a cada vez.

Agora que vimos como os canais funcionam, vamos olhar para um método diferente de concorrência.
