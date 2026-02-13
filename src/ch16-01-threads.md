## Usando Threads para Executar Código Simultaneamente

Na maioria dos sistemas operacionais atuais, o código de um programa executado é rodado em um *processo*, e o sistema operacional gerenciará múltiplos processos de uma vez. Dentro de um programa, você também pode ter partes independentes que rodam simultaneamente. As funcionalidades que rodam essas partes independentes são chamadas *threads*. Por exemplo, um servidor web poderia ter múltiplas threads para que pudesse responder a mais de uma requisição ao mesmo tempo.

Dividir a computação em seu programa em múltiplas threads para rodar múltiplas tarefas ao mesmo tempo pode melhorar a performance, mas também adiciona complexidade. Como threads podem rodar simultaneamente, não há garantia inerente sobre a ordem em que partes do seu código em diferentes threads rodarão. Isso pode levar a problemas, como:

- Corridas de dados (Race conditions), onde threads estão acessando dados ou recursos em uma ordem inconsistente
- Deadlocks, onde duas threads estão esperando uma pela outra, impedindo ambas as threads de continuar
- Bugs que só acontecem em certas situações e são difíceis de reproduzir e corrigir de forma confiável

Rust tenta mitigar os efeitos negativos do uso de threads, mas programar em um contexto multithreaded ainda requer reflexão cuidadosa e exige uma estrutura de código que é diferente daquela em programas rodando em uma única thread.

Linguagens de programação implementam threads de algumas maneiras diferentes, e muitos sistemas operacionais fornecem uma API que a linguagem de programação pode chamar para criar novas threads. A biblioteca padrão de Rust usa um modelo *1:1* de implementação de thread, pelo qual um programa usa uma thread do sistema operacional para cada thread da linguagem. Existem crates que implementam outros modelos de threading que fazem compensações diferentes para o modelo 1:1. (O sistema async de Rust, que veremos no próximo capítulo, fornece outra abordagem para concorrência também.)

### Criando uma Nova Thread com `spawn`

Para criar uma nova thread, chamamos a função `thread::spawn` e passamos a ela uma closure (falamos sobre closures no Capítulo 13) contendo o código que queremos rodar na nova thread. O exemplo na Listagem 16-1 imprime algum texto de uma thread principal e outro texto de uma nova thread.

<Listing number="16-1" file-name="src/main.rs" caption="Criando uma nova thread para imprimir uma coisa enquanto a thread principal imprime outra coisa">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Note que quando a thread principal de um programa Rust termina, todas as threads criadas são encerradas, tenham elas terminado de rodar ou não. A saída deste programa pode ser um pouco diferente a cada vez, mas se parecerá com o seguinte:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

As chamadas para `thread::sleep` forçam uma thread a parar sua execução por uma curta duração, permitindo que uma thread diferente rode. As threads provavelmente se revezarão, mas isso não é garantido: depende de como seu sistema operacional agenda as threads. Nesta execução, a thread principal imprimiu primeiro, embora a instrução de impressão da thread criada apareça primeiro no código. E mesmo que tenhamos dito à thread criada para imprimir até que `i` seja `9`, ela só chegou a `5` antes que a thread principal fosse encerrada.

Se você rodar este código e vir apenas saída da thread principal, ou não vir nenhuma sobreposição, tente aumentar os números nos intervalos para criar mais oportunidades para o sistema operacional alternar entre as threads.

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### Esperando Todas as Threads Terminarem

O código na Listagem 16-1 não apenas para a thread criada prematuramente na maioria das vezes devido ao término da thread principal, mas como não há garantia na ordem em que as threads rodam, também não podemos garantir que a thread criada chegará a rodar!

Podemos corrigir o problema da thread criada não rodar ou terminar prematuramente salvando o valor de retorno de `thread::spawn` em uma variável. O tipo de retorno de `thread::spawn` é `JoinHandle<T>`. Um `JoinHandle<T>` é um valor possuído que, quando chamamos o método `join` nele, esperará que sua thread termine. A Listagem 16-2 mostra como usar o `JoinHandle<T>` da thread que criamos na Listagem 16-1 e como chamar `join` para garantir que a thread criada termine antes que `main` saia.

<Listing number="16-2" file-name="src/main.rs" caption="Salvando um `JoinHandle<T>` de `thread::spawn` para garantir que a thread seja executada até a conclusão">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Chamar `join` no handle bloqueia a thread atualmente em execução até que a thread representada pelo handle termine. *Bloquear* uma thread significa que essa thread é impedida de realizar trabalho ou sair. Como colocamos a chamada para `join` após o loop `for` da thread principal, rodar a Listagem 16-2 deve produzir saída semelhante a esta:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

As duas threads continuam alternando, mas a thread principal espera por causa da chamada para `handle.join()` e não termina até que a thread criada tenha terminado.

Mas vamos ver o que acontece quando movemos `handle.join()` para antes do loop `for` em `main`, assim:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

A thread principal esperará a thread criada terminar e então executará seu loop `for`, então a saída não será mais intercalada, como mostrado aqui:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Pequenos detalhes, como onde `join` é chamado, podem afetar se suas threads rodam ao mesmo tempo ou não.

### Usando Closures `move` com Threads

Frequentemente usaremos a palavra-chave `move` com closures passadas para `thread::spawn` porque a closure então tomará posse dos valores que usa do ambiente, transferindo assim a posse desses valores de uma thread para outra. Em ["Capturando Referências ou Movendo Posse"][capture] no Capítulo 13, discutimos `move` no contexto de closures. Agora vamos nos concentrar mais na interação entre `move` e `thread::spawn`.

Note na Listagem 16-1 que a closure que passamos para `thread::spawn` não recebe argumentos: não estamos usando nenhum dado da thread principal no código da thread criada. Para usar dados da thread principal na thread criada, a closure da thread criada deve capturar os valores de que precisa. A Listagem 16-3 mostra uma tentativa de criar um vetor na thread principal e usá-lo na thread criada. No entanto, isso não funcionará ainda, como você verá em um momento.

<Listing number="16-3" file-name="src/main.rs" caption="Tentando usar um vetor criado pela thread principal em outra thread">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

A closure usa `v`, então capturará `v` e o tornará parte do ambiente da closure. Como `thread::spawn` roda esta closure em uma nova thread, deveríamos ser capazes de acessar `v` dentro dessa nova thread. Mas quando compilamos este exemplo, recebemos o seguinte erro:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust *infere* como capturar `v`, e como `println!` precisa apenas de uma referência a `v`, a closure tenta pegar `v` emprestado. No entanto, há um problema: Rust não pode dizer quanto tempo a thread criada rodará, então não sabe se a referência a `v` será sempre válida.

A Listagem 16-4 fornece um cenário que é mais provável de ter uma referência a `v` que não será válida.

<Listing number="16-4" file-name="src/main.rs" caption="Uma thread com uma closure que tenta capturar uma referência a `v` de uma thread principal que descarta `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Se Rust nos permitisse rodar este código, haveria uma possibilidade de que a thread criada fosse imediatamente colocada em segundo plano sem rodar nada. A thread criada tem uma referência a `v` dentro, mas a thread principal descarta `v` imediatamente, usando a função `drop` que discutimos no Capítulo 15. Então, quando a thread criada começa a executar, `v` não é mais válido, então uma referência a ele também é inválida. Oh não!

Para corrigir o erro de compilador na Listagem 16-3, podemos usar o conselho da mensagem de erro:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Ao adicionar a palavra-chave `move` antes da closure, forçamos a closure a tomar posse dos valores que está usando em vez de permitir que Rust infira que deve pegar os valores emprestados. A modificação na Listagem 16-3 mostrada na Listagem 16-5 compilará e rodará como pretendemos.

<Listing number="16-5" file-name="src/main.rs" caption="Usando a palavra-chave `move` para forçar uma closure a tomar posse dos valores que usa">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Poderíamos ser tentados a tentar a mesma coisa para corrigir o código na Listagem 16-4 onde a thread principal chamou `drop` usando uma closure `move`. No entanto, essa correção não funcionará porque o que a Listagem 16-4 está tentando fazer é proibido por um motivo diferente. Se adicionássemos `move` à closure, moveríamos `v` para o ambiente da closure, e não poderíamos mais chamar `drop` nele na thread principal. Receberíamos este erro de compilador em vez disso:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

As regras de posse de Rust nos salvaram novamente! Recebemos um erro do código na Listagem 16-3 porque Rust estava sendo conservador e apenas pegando `v` emprestado para a thread, o que significava que a thread principal poderia teoricamente invalidar a referência da thread criada. Ao dizer a Rust para mover a posse de `v` para a thread criada, estamos garantindo a Rust que a thread principal não usará mais `v`. Se mudarmos a Listagem 16-4 da mesma maneira, estaremos então violando as regras de posse quando tentarmos usar `v` na thread principal. A palavra-chave `move` sobrescreve o padrão conservador de empréstimo de Rust; ela não nos deixa violar as regras de posse.

Agora que cobrimos o que são threads e os métodos fornecidos pela API de thread, vamos olhar para algumas situações em que podemos usar threads.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
