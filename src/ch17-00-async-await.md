# Fundamentos da Programação Assíncrona: Async, Await, Futures e Streams

Muitas operações que pedimos ao computador para fazer podem demorar um pouco para terminar. Seria bom se pudéssemos fazer outra coisa enquanto esperamos que esses processos de longa duração sejam concluídos. Computadores modernos oferecem duas técnicas para trabalhar em mais de uma operação ao mesmo tempo: paralelismo e concorrência. A lógica dos nossos programas, no entanto, é escrita de uma maneira principalmente linear. Gostaríamos de poder especificar as operações que um programa deve realizar e pontos em que uma função poderia pausar e alguma outra parte do programa poderia rodar, sem precisar especificar antecipadamente exatamente a ordem e a maneira como cada pedaço de código deve rodar. _Programação assíncrona_ é uma abstração que nos permite expressar nosso código em termos de possíveis pontos de pausa e resultados eventuais, cuidando dos detalhes de coordenação para nós.

Este capítulo baseia-se no uso de threads para paralelismo e concorrência do Capítulo 16, introduzindo uma abordagem alternativa para escrever código: futures, streams e a sintaxe `async` e `await` do Rust, que nos permitem expressar como as operações podem ser assíncronas, e os crates de terceiros que implementam runtimes assíncronos: código que gerencia e coordena a execução de operações assíncronas.

Vamos considerar um exemplo. Digamos que você está exportando um vídeo que criou de uma celebração familiar, uma operação que pode levar de minutos a horas. A exportação do vídeo usará o máximo de poder de CPU e GPU que puder. Se você tivesse apenas um núcleo de CPU e seu sistema operacional não pausasse essa exportação até que ela fosse concluída — isto é, se ele executasse a exportação _sicronamente_ — você não poderia fazer mais nada no seu computador enquanto essa tarefa estivesse rodando. Essa seria uma experiência bastante frustrante. Felizmente, o sistema operacional do seu computador pode, e faz, interromper invisivelmente a exportação com frequência suficiente para permitir que você faça outro trabalho simultaneamente.

Agora digamos que você está baixando um vídeo compartilhado por outra pessoa, o que também pode demorar um pouco, mas não consome tanto tempo de CPU. Neste caso, a CPU tem que esperar que os dados cheguem da rede. Embora você possa começar a ler os dados assim que eles começarem a chegar, pode levar algum tempo para que todos apareçam. Mesmo quando todos os dados estiverem presentes, se o vídeo for muito grande, pode levar pelo menos um ou dois segundos para carregá-lo todo. Isso pode não parecer muito, mas é muito tempo para um processador moderno, que pode realizar bilhões de operações a cada segundo. Novamente, seu sistema operacional interromperá invisivelmente seu programa para permitir que a CPU realize outro trabalho enquanto espera a chamada de rede terminar.

A exportação de vídeo é um exemplo de uma operação _limitada pela CPU_ (CPU-bound) ou _limitada por computação_ (compute-bound). É limitada pela velocidade potencial de processamento de dados do computador dentro da CPU ou GPU, e quanto dessa velocidade ele pode dedicar à operação. O download de vídeo é um exemplo de uma operação _limitada por E/S_ (I/O-bound), porque é limitada pela velocidade de _entrada e saída_ do computador; ela só pode ir tão rápido quanto os dados podem ser enviados pela rede.

Em ambos os exemplos, as interrupções invisíveis do sistema operacional fornecem uma forma de concorrência. Essa concorrência acontece apenas no nível de todo o programa, no entanto: o sistema operacional interrompe um programa para deixar outros programas realizarem trabalho. Em muitos casos, porque entendemos nossos programas em um nível muito mais granular do que o sistema operacional, podemos identificar oportunidades para concorrência que o sistema operacional não consegue ver.

Por exemplo, se estamos construindo uma ferramenta para gerenciar downloads de arquivos, devemos ser capazes de escrever nosso programa de modo que iniciar um download não trave a interface do usuário (UI), e os usuários devem ser capazes de iniciar múltiplos downloads ao mesmo tempo. Muitas APIs do sistema operacional para interagir com a rede são _bloqueantes_, no entanto; isto é, elas bloqueiam o progresso do programa até que os dados que estão processando estejam completamente prontos.

> Nota: É assim que a _maioria_ das chamadas de função funciona, se você pensar bem. No entanto, o termo _bloqueante_ é geralmente reservado para chamadas de função que interagem com arquivos, a rede ou outros recursos no computador, porque esses são os casos em que um programa individual se beneficiaria se a operação fosse _não_-bloqueante.

Poderíamos evitar bloquear nossa thread principal criando uma thread dedicada para baixar cada arquivo. No entanto, a sobrecarga dos recursos do sistema usados por essas threads acabaria se tornando um problema. Seria preferível se a chamada não bloqueasse em primeiro lugar e, em vez disso, pudéssemos definir um número de tarefas que gostaríamos que nosso programa completasse e permitir que o runtime escolhesse a melhor ordem e maneira de executá-las.

Isso é exatamente o que a abstração _async_ (abreviação de _asynchronous_, ou assíncrono) do Rust nos dá. Neste capítulo, você aprenderá tudo sobre async à medida que cobrimos os seguintes tópicos:

- Como usar a sintaxe `async` e `await` do Rust e executar funções assíncronas com um runtime
- Como usar o modelo async para resolver alguns dos mesmos desafios que vimos no Capítulo 16
- Como multithreading e async fornecem soluções complementares que você pode combinar em muitos casos

Antes de vermos como async funciona na prática, no entanto, precisamos fazer um pequeno desvio para discutir as diferenças entre paralelismo e concorrência.

## Paralelismo e Concorrência

Tratamos paralelismo e concorrência como majoritariamente intercambiáveis até agora. Agora precisamos distingui-los com mais precisão, porque as diferenças aparecerão quando começarmos a trabalhar.

Considere as diferentes maneiras como uma equipe poderia dividir o trabalho em um projeto de software. Você poderia atribuir a um único membro várias tarefas, atribuir a cada membro uma tarefa ou usar uma mistura das duas abordagens.

Quando um indivíduo trabalha em várias tarefas diferentes antes que qualquer uma delas esteja completa, isso é _concorrência_. Uma maneira de implementar concorrência é semelhante a ter dois projetos diferentes no seu computador e, quando você fica entediado ou travado em um projeto, muda para o outro. Você é apenas uma pessoa, então não pode fazer progresso em ambas as tarefas exatamente ao mesmo tempo, mas pode realizar multitarefa, fazendo progresso em uma de cada vez alternando entre elas (veja a Figura 17-1).

<figure>

<img src="img/trpl17-01.svg" class="center" alt="Um diagrama com caixas empilhadas rotuladas Tarefa A e Tarefa B, com diamantes nelas representando subtarefas. Setas apontam de A1 para B1, B1 para A2, A2 para B2, B2 para A3, A3 para A4 e A4 para B3. As setas entre as subtarefas cruzam as caixas entre Tarefa A e Tarefa B." />

<figcaption>Figura 17-1: Um fluxo de trabalho concorrente, alternando entre Tarefa A e Tarefa B</figcaption>

</figure>

Quando a equipe divide um grupo de tarefas fazendo com que cada membro pegue uma tarefa e trabalhe nela sozinho, isso é _paralelismo_. Cada pessoa na equipe pode fazer progresso exatamente ao mesmo tempo (veja a Figura 17-2).

<figure>

<img src="img/trpl17-02.svg" class="center" alt="Um diagrama com caixas empilhadas rotuladas Tarefa A e Tarefa B, com diamantes nelas representando subtarefas. Setas apontam de A1 para A2, A2 para A3, A3 para A4, B1 para B2 e B2 para B3. Nenhuma seta cruza entre as caixas para Tarefa A e Tarefa B." />

<figcaption>Figura 17-2: Um fluxo de trabalho paralelo, onde o trabalho acontece na Tarefa A e Tarefa B independentemente</figcaption>

</figure>

Em ambos os fluxos de trabalho, você pode ter que coordenar entre diferentes tarefas. Talvez você tenha pensado que a tarefa atribuída a uma pessoa era totalmente independente do trabalho de todos os outros, mas na verdade requer que outra pessoa na equipe termine sua tarefa primeiro. Parte do trabalho poderia ser feita em paralelo, mas parte dele era na verdade _serial_: só poderia acontecer em uma série, uma tarefa após a outra, como na Figura 17-3.

<figure>

<img src="img/trpl17-03.svg" class="center" alt="Um diagrama com caixas empilhadas rotuladas Tarefa A e Tarefa B, com diamantes nelas representando subtarefas. Na Tarefa A, setas apontam de A1 para A2, de A2 para um par de linhas verticais grossas como um símbolo de “pausa”, e desse símbolo para A3. Na tarefa B, setas apontam de B1 para B2, de B2 para B3, de B3 para A3 e de B3 para B4." />

<figcaption>Figura 17-3: Um fluxo de trabalho parcialmente paralelo, onde o trabalho acontece na Tarefa A e Tarefa B independentemente até que a Tarefa A3 seja bloqueada pelos resultados da Tarefa B3.</figcaption>

</figure>

Da mesma forma, você pode perceber que uma das suas próprias tarefas depende de outra das suas tarefas. Agora seu trabalho concorrente também se tornou serial.

Paralelismo e concorrência podem se cruzar também. Se você descobrir que um colega está travado até que você termine uma das suas tarefas, você provavelmente focará todos os seus esforços nessa tarefa para “desbloquear” seu colega. Você e seu colega de trabalho não são mais capazes de trabalhar em paralelo, e você também não é mais capaz de trabalhar concorrentemente em suas próprias tarefas.

A mesma dinâmica básica entra em jogo com software e hardware. Em uma máquina com um único núcleo de CPU, a CPU pode realizar apenas uma operação por vez, mas ainda pode trabalhar concorrentemente. Usando ferramentas como threads, processos e async, o computador pode pausar uma atividade e mudar para outras antes de eventualmente voltar para aquela primeira atividade novamente. Em uma máquina com múltiplos núcleos de CPU, ela também pode fazer trabalho em paralelo. Um núcleo pode estar realizando uma tarefa enquanto outro núcleo realiza uma completamente não relacionada, e essas operações realmente acontecem ao mesmo tempo.

Rodar código async em Rust geralmente acontece de forma concorrente. Dependendo do hardware, do sistema operacional e do runtime async que estamos usando (mais sobre runtimes async em breve), essa concorrência também pode usar paralelismo por baixo dos panos.

Agora, vamos mergulhar em como a programação async em Rust realmente funciona.
