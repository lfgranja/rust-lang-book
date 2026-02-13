## Juntando Tudo: Futures, Tarefas e Threads

Como vimos no [Capítulo 16][ch16]<!-- ignore -->, threads fornecem uma abordagem para
concorrência. Vimos outra abordagem neste capítulo: usando async com
futures e streams. Se você está se perguntando quando escolher um método em vez do outro,
a resposta é: depende! E em muitos casos, a escolha não é threads _ou_
async, mas sim threads _e_ async.

Muitos sistemas operacionais têm fornecido modelos de concorrência baseados em threads por
décadas agora, e muitas linguagens de programação os suportam como resultado. No entanto,
esses modelos não são sem seus compromissos (tradeoffs). Em muitos sistemas operacionais, eles
usam uma quantidade razoável de memória para cada thread. Threads também são apenas uma opção quando
seu sistema operacional e hardware as suportam. Diferente de computadores desktop e
móveis convencionais, alguns sistemas embarcados não têm um SO de forma alguma, então eles também
não têm threads.

O modelo async fornece um conjunto diferente—e em última análise complementar—de
compromissos. No modelo async, operações concorrentes não requerem suas próprias
threads. Em vez disso, elas podem rodar em tarefas (tasks), como quando usamos `trpl::spawn_task` para
iniciar trabalho a partir de uma função síncrona na seção de streams. Uma tarefa é
similar a uma thread, mas em vez de ser gerenciada pelo sistema operacional, ela é
gerenciada por código em nível de biblioteca: o runtime.

Há uma razão para as APIs para criar (spawning) threads e tarefas serem tão
similares. Threads agem como um limite para conjuntos de operações síncronas;
concorrência é possível _entre_ threads. Tarefas agem como um limite para conjuntos de
operações _assíncronas_; concorrência é possível tanto _entre_ quanto _dentro_
de tarefas, porque uma tarefa pode alternar entre futures em seu corpo. Finalmente, futures
são a unidade mais granular de concorrência do Rust, e cada future pode representar uma
árvore de outros futures. O runtime—especificamente, seu executor—gerencia tarefas,
e tarefas gerenciam futures. Nesse aspecto, tarefas são similares a threads leves,
gerenciadas pelo runtime com capacidades adicionais que vêm de serem gerenciadas por
um runtime em vez de pelo sistema operacional.

Isso não significa que tarefas async são sempre melhores que threads (ou vice
versa). Concorrência com threads é de algumas maneiras um modelo de programação mais simples
do que concorrência com `async`. Isso pode ser uma força ou uma fraqueza. Threads são
um pouco “fogo e esqueça” (fire and forget); elas não têm equivalente nativo a um future, então elas
simplesmente rodam até a conclusão sem serem interrompidas exceto pelo próprio
sistema operacional.

E acontece que threads e tarefas frequentemente funcionam
muito bem juntas, porque tarefas podem (pelo menos em alguns runtimes) ser movidas
entre threads. Na verdade, "por baixo dos panos", o runtime que temos
usado—incluindo as funções `spawn_blocking` e `spawn_task`—é multithreaded
por padrão! Muitos runtimes usam uma abordagem chamada _roubo de trabalho_ (work stealing) para
transparentemente mover tarefas entre threads, baseado em como as threads estão
sendo utilizadas atualmente, para melhorar o desempenho geral do sistema. Essa
abordagem realmente requer threads _e_ tarefas, e portanto futures.

Ao pensar sobre qual método usar e quando, considere estas regras práticas:

- Se o trabalho é _muito paralelizável_ (isto é, limitado por CPU), tal como processar
  um monte de dados onde cada parte pode ser processada separadamente, threads são uma
  escolha melhor.
- Se o trabalho é _muito concorrente_ (isto é, limitado por E/S), tal como lidar com
  mensagens de um monte de fontes diferentes que podem chegar em intervalos
  diferentes ou taxas diferentes, async é uma escolha melhor.

E se você precisa tanto de paralelismo quanto de concorrência, você não tem que escolher
entre threads e async. Você pode usá-los juntos livremente, deixando cada um
fazer a parte em que é melhor. Por exemplo, a Listagem 17-25 mostra um exemplo bastante comum
desse tipo de mistura em código Rust do mundo real.

<Listing number="17-25" caption="Enviando mensagens com código bloqueante em uma thread e aguardando as mensagens em um bloco async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:all}}
```

</Listing>

Começamos criando um canal async, depois criando (spawning) uma thread que toma
posse do lado remetente do canal usando a palavra-chave `move`. Dentro
da thread, enviamos os números 1 a 10, dormindo por um segundo entre
cada um. Finalmente, rodamos um future criado com um bloco async passado para
`trpl::block_on` assim como fizemos ao longo do capítulo. Nesse future, nós
aguardamos essas mensagens, assim como nos outros exemplos de passagem de mensagem que
vimos.

Para retornar ao cenário com o qual abrimos o capítulo, imagine rodar um conjunto de
tarefas de codificação de vídeo usando uma thread dedicada (porque codificação de vídeo é
limitada por computação) mas notificando a UI que essas operações terminaram com um
canal async. Há incontáveis exemplos desses tipos de combinações em
casos de uso do mundo real.

## Resumo

Este não é o fim do que você verá sobre concorrência neste livro. O projeto no
[Capítulo 21][ch21]<!-- ignore --> aplicará esses conceitos em uma situação mais realista
do que os exemplos mais simples discutidos aqui e comparará a resolução de problemas
com threads versus tarefas e futures mais diretamente.

Não importa qual dessas abordagens você escolha, Rust lhe dá as ferramentas que você
precisa para escrever código seguro, rápido e concorrente—seja para um servidor web de alta taxa de transferência
ou um sistema operacional embarcado.

A seguir, falaremos sobre maneiras idiomáticas de modelar problemas e estruturar soluções
à medida que seus programas Rust ficam maiores. Além disso, discutiremos como os idiomas (idioms) do Rust
se relacionam com aqueles com os quais você pode estar familiarizado da programação orientada a objetos.

[ch16]: http://localhost:3000/ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
