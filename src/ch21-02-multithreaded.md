# Transformando Nosso Servidor Single-Threaded em um Servidor Multithreaded

No momento, o servidor processará cada requisição por vez, o que significa que não processará uma segunda conexão até que a primeira conexão termine de ser processada. Se o servidor recebesse mais e mais requisições, essa execução serial seria cada vez menos ideal. Se o servidor receber uma requisição que leva muito tempo para processar, as requisições subsequentes terão que esperar até que a requisição longa termine, mesmo que as novas requisições possam ser processadas rapidamente. Precisaremos corrigir isso, mas primeiro vamos ver o problema em ação.

### Simulando uma Requisição Lenta

Vamos ver como uma requisição de processamento lento pode afetar outras requisições feitas à nossa implementação de servidor atual. O Listagem 21-10 implementa o tratamento de uma requisição para */sleep* com uma resposta lenta simulada que fará com que o servidor durma por cinco segundos antes de responder.

Listagem 21-10: Simulando uma requisição lenta dormindo por cinco segundos

```rust,no_run
use std::{fs, thread};
use std::io::{BufRead, BufReader, Write};
use std::net::{TcpListener, TcpStream};
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(5));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
}
```

Mudamos de `if` para `match` agora que temos três casos. Precisamos corresponder explicitamente em uma fatia de `request_line` para corresponder ao padrão em relação aos valores literais de string; `match` não faz referência e desreferência automáticas, como o método de igualdade faz.

O primeiro braço é o mesmo que o bloco `if` do Listagem 21-9. O segundo braço corresponde a uma requisição para */sleep*. Quando essa requisição é recebida, o servidor dormirá por cinco segundos antes de renderizar a página HTML de sucesso. O terceiro braço é o mesmo que o bloco `else` do Listagem 21-9.

Você pode ver quão primitivo é nosso servidor: bibliotecas reais lidariam com o reconhecimento de múltiplas requisições de uma maneira muito menos verbosa!

Inicie o servidor usando `cargo run`. Em seguida, abra duas janelas do navegador: uma para *http://127.0.0.1:7878* e outra para *http://127.0.0.1:7878/sleep*. Se você inserir o URI */* algumas vezes, como antes, verá que ele responde rapidamente. Mas se você inserir */sleep* e depois carregar */*, verá que */* espera até que `sleep` tenha dormido por seus cinco segundos completos antes de carregar.

Existem várias técnicas que poderíamos usar para evitar que as requisições fiquem presas atrás de uma requisição lenta, incluindo o uso de async como fizemos no Capítulo 17; a que implementaremos é um pool de threads.

### Melhorando a Taxa de Transferência com um Pool de Threads

Um *thread pool* (pool de threads) é um grupo de threads geradas que estão prontas e esperando para lidar com uma tarefa. Quando o programa recebe uma nova tarefa, ele atribui uma das threads no pool à tarefa, e essa thread processará a tarefa. As threads restantes no pool estão disponíveis para lidar com quaisquer outras tarefas que cheguem enquanto a primeira thread está processando. Quando a primeira thread termina de processar sua tarefa, ela é devolvida ao pool de threads ociosas, pronta para lidar com uma nova tarefa. Um pool de threads permite processar conexões simultaneamente, aumentando a taxa de transferência do seu servidor.

Limitaremos o número de threads no pool a um número pequeno para nos proteger de ataques DoS; se tivéssemos nosso programa criando uma nova thread para cada requisição à medida que ela chegasse, alguém fazendo 10 milhões de requisições ao nosso servidor poderia causar estragos usando todos os recursos do nosso servidor e paralisando o processamento de requisições.

Em vez de gerar threads ilimitadas, então, teremos um número fixo de threads esperando no pool. As requisições que chegam são enviadas para o pool para processamento. O pool manterá uma fila de requisições recebidas. Cada uma das threads no pool retirará uma requisição desta fila, lidará com a requisição e, em seguida, pedirá à fila outra requisição. Com este design, podemos processar até *N* requisições simultaneamente, onde *N* é o número de threads. Se cada thread estiver respondendo a uma requisição de longa duração, as requisições subsequentes ainda podem ficar na fila, mas aumentamos o número de requisições de longa duração que podemos lidar antes de atingir esse ponto.

Esta técnica é apenas uma das muitas maneiras de melhorar a taxa de transferência de um servidor web. Outras opções que você pode explorar são o modelo fork/join, o modelo de E/S assíncrona single-threaded e o modelo de E/S assíncrona multithreaded. Se você estiver interessado neste tópico, pode ler mais sobre outras soluções e tentar implementá-las; com uma linguagem de baixo nível como Rust, todas essas opções são possíveis.

Antes de começarmos a implementar um pool de threads, vamos falar sobre como o uso do pool deve ser. Quando você está tentando projetar código, escrever a interface do cliente primeiro pode ajudar a guiar seu design. Escreva a API do código de modo que ele seja estruturado da maneira que você deseja chamá-lo; então, implemente a funcionalidade dentro dessa estrutura em vez de implementar a funcionalidade e depois projetar a API pública.

Semelhante a como usamos o desenvolvimento orientado a testes no projeto no Capítulo 12, usaremos o desenvolvimento orientado pelo compilador aqui. Escreveremos o código que chama as funções que queremos e, em seguida, olharemos para os erros do compilador para determinar o que devemos mudar a seguir para fazer o código funcionar. Antes de fazermos isso, no entanto, exploraremos a técnica que não vamos usar como ponto de partida.

#### Gerando uma Thread para Cada Requisição

Primeiro, vamos explorar como nosso código poderia parecer se criasse uma nova thread para cada conexão. Como mencionado anteriormente, este não é nosso plano final devido aos problemas com a possibilidade de gerar um número ilimitado de threads, mas é um ponto de partida para obter um servidor multithreaded funcionando primeiro. Em seguida, adicionaremos o pool de threads como uma melhoria, e contrastar as duas soluções será mais fácil.

O Listagem 21-11 mostra as alterações a serem feitas em `main` para gerar uma nova thread para lidar com cada fluxo dentro do loop `for`.

Listagem 21-11: Gerando uma nova thread para cada fluxo

```rust,no_run
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
```

Como você aprendeu no Capítulo 16, `thread::spawn` criará uma nova thread e, em seguida, executará o código na closure na nova thread. Se você executar este código e carregar */sleep* no seu navegador, depois */* em mais duas abas do navegador, você verá de fato que as requisições para */* não precisam esperar que */sleep* termine. No entanto, como mencionamos, isso acabará sobrecarregando o sistema porque você estaria fazendo novas threads sem qualquer limite.

Você também pode se lembrar do Capítulo 17 que este é exatamente o tipo de situação em que async e await realmente brilham! Tenha isso em mente enquanto construímos o pool de threads e pense em como as coisas pareceriam diferentes ou iguais com async.

#### Criando um Número Finito de Threads

Queremos que nosso pool de threads funcione de maneira semelhante e familiar para que a mudança de threads para um pool de threads não exija grandes alterações no código que usa nossa API. O Listagem 21-12 mostra a interface hipotética para uma struct `ThreadPool` que queremos usar em vez de `thread::spawn`.

Listagem 21-12: Nossa interface ideal de `ThreadPool`

```rust,ignore,does_not_compile
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

Usamos `ThreadPool::new` para criar um novo pool de threads com um número configurável de threads, neste caso quatro. Então, no loop `for`, `pool.execute` tem uma interface semelhante a `thread::spawn` no sentido de que recebe uma closure que o pool deve executar para cada fluxo. Precisamos implementar `pool.execute` para que ele receba a closure e a dê a uma thread no pool para executar. Este código ainda não compilará, mas tentaremos para que o compilador possa nos guiar em como corrigi-lo.

#### Construindo a Struct `ThreadPool` Usando Desenvolvimento Orientado pelo Compilador

Faça as alterações no Listagem 21-12 em *src/main.rs* e, em seguida, vamos usar os erros do compilador de `cargo check` para conduzir nosso desenvolvimento. Aqui está o primeiro erro que obtemos:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:10:16
   |
10 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type `ThreadPool`
```

Ótimo! Este erro nos diz que precisamos de um tipo ou módulo `ThreadPool`, então construiremos um agora. Nossa implementação de `ThreadPool` será independente do tipo de trabalho que nosso servidor web está fazendo. Então, vamos mudar a crate `hello` de uma crate binária para uma crate de biblioteca para manter nossa implementação de `ThreadPool`. Depois de mudar para uma crate de biblioteca, também poderíamos usar a biblioteca de pool de threads separada para qualquer trabalho que queiramos fazer usando um pool de threads, não apenas para servir requisições web.

Crie um arquivo *src/lib.rs* que contenha o seguinte, que é a definição mais simples de uma struct `ThreadPool` que podemos ter por enquanto:

```rust
pub struct ThreadPool;
```

Em seguida, edite o arquivo *main.rs* para trazer `ThreadPool` para o escopo da crate da biblioteca adicionando o seguinte código ao topo de *src/main.rs*:

```rust,ignore
use hello::ThreadPool;
```

Este código ainda não funcionará, mas vamos verificá-lo novamente para obter o próximo erro que precisamos resolver:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:11:28
   |
11 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`
```

Este erro indica que a seguir precisamos criar uma função associada chamada `new` para `ThreadPool`. Também sabemos que `new` precisa ter um parâmetro que possa aceitar `4` como argumento e deve retornar uma instância de `ThreadPool`. Vamos implementar a função `new` mais simples que terá essas características:

```rust
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

Escolhemos `usize` como o tipo do parâmetro `size` porque sabemos que um número negativo de threads não faz sentido. Também sabemos que usaremos este `4` como o número de elementos em uma coleção de threads, que é para o que o tipo `usize` serve, conforme discutido na seção "Tipos Inteiros" no Capítulo 3.

Vamos verificar o código novamente:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:16:14
   |
16 |         pool.execute(|| {
   |              ^^^^^^^ method not found in `ThreadPool`
```

Agora o erro ocorre porque não temos um método `execute` em `ThreadPool`. Lembre-se da seção "Criando um Número Finito de Threads" que decidimos que nosso pool de threads deve ter uma interface semelhante a `thread::spawn`. Além disso, implementaremos a função `execute` para que ela receba a closure que lhe é dada e a dê a uma thread ociosa no pool para executar.

Definiremos o método `execute` em `ThreadPool` para receber uma closure como parâmetro. Lembre-se da seção "Movendo Valores Capturados Para Fora das Closures" no Capítulo 13 que podemos receber closures como parâmetros com três traits diferentes: `Fn`, `FnMut` e `FnOnce`. Precisamos decidir que tipo de closure usar aqui. Sabemos que acabaremos fazendo algo semelhante à implementação da biblioteca padrão `thread::spawn`, então podemos olhar para quais limites a assinatura de `thread::spawn` tem em seu parâmetro. A documentação nos mostra o seguinte:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

O parâmetro de tipo `F` é o que nos preocupa aqui; o parâmetro de tipo `T` está relacionado ao valor de retorno, e não estamos preocupados com isso. Podemos ver que `spawn` usa `FnOnce` como o limite de trait em `F`. Provavelmente é isso que queremos também, porque eventualmente passaremos o argumento que obtemos em `execute` para `spawn`. Podemos ter mais certeza de que `FnOnce` é o trait que queremos usar porque a thread para executar uma requisição executará a closure dessa requisição apenas uma vez, o que corresponde ao `Once` em `FnOnce`.

O parâmetro de tipo `F` também tem o limite de trait `Send` e o limite de tempo de vida `'static`, que são úteis em nossa situação: precisamos de `Send` para transferir a closure de uma thread para outra e `'static` porque não sabemos quanto tempo a thread levará para executar. Vamos criar um método `execute` em `ThreadPool` que receberá um parâmetro genérico do tipo `F` com esses limites:

```rust
pub struct ThreadPool;

impl ThreadPool {
    // --trecho omitido--
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}
```

Ainda usamos o `()` após `FnOnce` porque este `FnOnce` representa uma closure que não recebe parâmetros e retorna o tipo de unidade `()`. Assim como definições de função, o tipo de retorno pode ser omitido da assinatura, mas mesmo se não tivermos parâmetros, ainda precisamos dos parênteses.

Novamente, esta é a implementação mais simples do método `execute`: ele não faz nada, mas estamos apenas tentando fazer nosso código compilar. Vamos verificar novamente:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.24s
```

Ele compila! Mas observe que se você tentar `cargo run` e fizer uma requisição no navegador, verá os erros no navegador que vimos no início do capítulo. Nossa biblioteca ainda não está chamando a closure passada para `execute`!

> Nota: Um ditado que você pode ouvir sobre linguagens com compiladores estritos, como Haskell e Rust, é "Se o código compila, ele funciona". Mas este ditado não é universalmente verdadeiro. Nosso projeto compila, mas não faz absolutamente nada! Se estivéssemos construindo um projeto real e completo, este seria um bom momento para começar a escrever testes unitários para verificar se o código compila *e* tem o comportamento que queremos.

Considere: O que seria diferente aqui se fôssemos executar um futuro em vez de uma closure?

#### Validando o Número de Threads em `new`

Não estamos fazendo nada com os parâmetros para `new` e `execute`. Vamos implementar os corpos dessas funções com o comportamento que queremos. Para começar, vamos pensar sobre `new`. Anteriormente escolhemos um tipo sem sinal para o parâmetro `size` porque um pool com um número negativo de threads não faz sentido. No entanto, um pool com zero threads também não faz sentido, mas zero é um `usize` perfeitamente válido. Adicionaremos código para verificar se `size` é maior que zero antes de retornarmos uma instância de `ThreadPool`, e faremos o programa entrar em pânico se receber um zero usando a macro `assert!`, como mostrado no Listagem 21-13.

Listagem 21-13: Implementando `ThreadPool::new` para entrar em pânico se `size` for zero

```rust
impl ThreadPool {
    /// Cria um novo ThreadPool.
    ///
    /// O tamanho é o número de threads no pool.
    ///
    /// # Pânicos
    ///
    /// A função `new` entrará em pânico se o tamanho for zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }

    // --trecho omitido--
}
```

Também adicionamos alguma documentação para nosso `ThreadPool` com comentários de documentação. Note que seguimos as boas práticas de documentação adicionando uma seção que chama a atenção para as situações em que nossa função pode entrar em pânico, conforme discutido no Capítulo 14. Tente executar `cargo doc --open` e clicar na struct `ThreadPool` para ver como a documentação gerada para `new` se parece!

Em vez de adicionar a macro `assert!` como fizemos aqui, poderíamos mudar `new` para `build` e retornar um `Result` como fizemos com `Config::build` no projeto de E/S no Listagem 12-9. Mas decidimos neste caso que tentar criar um pool de threads sem threads deve ser um erro irrecuperável. Se você estiver se sentindo ambicioso, tente escrever uma função chamada `build` com a seguinte assinatura para comparar com a função `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Criando Espaço para Armazenar as Threads

Agora que temos uma maneira de saber que temos um número válido de threads para armazenar no pool, podemos criar essas threads e armazená-las na struct `ThreadPool` antes de retornar a struct. Mas como "armazenamos" uma thread? Vamos dar outra olhada na assinatura de `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

A função `spawn` retorna um `JoinHandle<T>`, onde `T` é o tipo que a closure retorna. Vamos tentar usar `JoinHandle` também e ver o que acontece. No nosso caso, as closures que estamos passando para o pool de threads lidarão com a conexão e não retornarão nada, então `T` será o tipo de unidade `()`.

O código no Listagem 21-14 compilará, mas ainda não cria nenhuma thread. Mudamos a definição de `ThreadPool` para conter um vetor de instâncias `thread::JoinHandle<()>`, inicializamos o vetor com uma capacidade de `size`, configuramos um loop `for` que executará algum código para criar as threads e retornamos uma instância `ThreadPool` contendo-as.

Listagem 21-14: Criando um vetor para `ThreadPool` conter as threads

```rust
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --trecho omitido--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // criar algumas threads e armazená-las no vetor
        }

        ThreadPool { threads }
    }
    // --trecho omitido--
}
```

Trouxemos `std::thread` para o escopo na crate da biblioteca porque estamos usando `thread::JoinHandle` como o tipo dos itens no vetor em `ThreadPool`.

Uma vez que um tamanho válido é recebido, nosso `ThreadPool` cria um novo vetor que pode conter `size` itens. A função `with_capacity` realiza a mesma tarefa que `Vec::new`, mas com uma diferença importante: ela pré-aloca espaço no vetor. Como sabemos que precisamos armazenar `size` elementos no vetor, fazer essa alocação antecipadamente é ligeiramente mais eficiente do que usar `Vec::new`, que se redimensiona à medida que elementos são inseridos.

Quando você executar `cargo check` novamente, deve ter sucesso.

#### Uma Struct `Worker` Responsável por Enviar Código do `ThreadPool` para uma Thread

Deixamos um comentário no loop `for` no Listagem 21-14 em relação à criação de threads. Aqui, veremos como realmente criamos threads. A biblioteca padrão fornece `thread::spawn` como uma maneira de criar threads, e `thread::spawn` espera receber algum código que a thread deve executar assim que a thread for criada. No entanto, no nosso caso, queremos criar as threads e fazê-las *esperar* pelo código que enviaremos mais tarde. A implementação de threads da biblioteca padrão não inclui nenhuma maneira de fazer isso; temos que implementá-la manualmente.

Implementaremos esse comportamento introduzindo uma nova estrutura de dados entre o `ThreadPool` e as threads que gerenciará esse novo comportamento. Chamaremos essa estrutura de dados de *Worker* (Trabalhador), que é um termo comum em implementações de pooling. O `Worker` pega o código que precisa ser executado e executa o código em sua thread.

Pense nas pessoas trabalhando na cozinha de um restaurante: os trabalhadores esperam até que os pedidos cheguem dos clientes, e então são responsáveis por pegar esses pedidos e atendê-los.

Em vez de armazenar um vetor de instâncias `JoinHandle<()>` no pool de threads, armazenaremos instâncias da struct `Worker`. Cada `Worker` armazenará uma única instância `JoinHandle<()>`. Então, implementaremos um método em `Worker` que receberá uma closure de código para executar e a enviará para a thread já em execução para execução. Também daremos a cada `Worker` um `id` para que possamos distinguir entre as diferentes instâncias de `Worker` no pool ao registrar ou depurar.

Aqui está o novo processo que acontecerá quando criarmos um `ThreadPool`. Implementaremos o código que envia a closure para a thread depois de termos o `Worker` configurado desta forma:

1. Definir uma struct `Worker` que contém um `id` e um `JoinHandle<()>`.
2. Mudar `ThreadPool` para conter um vetor de instâncias `Worker`.
3. Definir uma função `Worker::new` que recebe um número `id` e retorna uma instância `Worker` que contém o `id` e uma thread gerada com uma closure vazia.
4. Em `ThreadPool::new`, use o contador do loop `for` para gerar um `id`, crie um novo `Worker` com esse `id` e armazene o `Worker` no vetor.

Se você estiver pronto para um desafio, tente implementar essas mudanças por conta própria antes de olhar para o código no Listagem 21-15.

Pronto? Aqui está o Listagem 21-15 com uma maneira de fazer as modificações anteriores.

Listagem 21-15: Modificando `ThreadPool` para conter instâncias `Worker` em vez de conter threads diretamente

```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --trecho omitido--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers }
    }
    // --trecho omitido--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker { id, thread }
    }
}
```

Mudamos o nome do campo em `ThreadPool` de `threads` para `workers` porque agora ele está contendo instâncias `Worker` em vez de instâncias `JoinHandle<()>`. Usamos o contador no loop `for` como um argumento para `Worker::new`, e armazenamos cada novo `Worker` no vetor chamado `workers`.

O código externo (como nosso servidor em *src/main.rs*) não precisa conhecer os detalhes de implementação sobre o uso de uma struct `Worker` dentro de `ThreadPool`, então tornamos a struct `Worker` e sua função `new` privadas. A função `Worker::new` usa o `id` que damos a ela e armazena uma instância `JoinHandle<()>` que é criada gerando uma nova thread usando uma closure vazia.

> Nota: Se o sistema operacional não puder criar uma thread porque não há recursos do sistema suficientes, `thread::spawn` entrará em pânico. Isso fará com que todo o nosso servidor entre em pânico, mesmo que a criação de algumas threads possa ter sucesso. Por uma questão de simplicidade, esse comportamento é bom, mas em uma implementação de pool de threads de produção, você provavelmente gostaria de usar `std::thread::Builder` e seu método `spawn` que retorna `Result`.

Este código compilará e armazenará o número de instâncias `Worker` que especificamos como argumento para `ThreadPool::new`. Mas *ainda* não estamos processando a closure que obtemos em `execute`. Vamos ver como fazer isso a seguir.

#### Enviando Requisições para Threads via Canais

O próximo problema que abordaremos é que as closures dadas a `thread::spawn` não fazem absolutamente nada. Atualmente, obtemos a closure que queremos executar no método `execute`. Mas precisamos dar a `thread::spawn` uma closure para executar quando criamos cada `Worker` durante a criação do `ThreadPool`.

Queremos que as structs `Worker` que acabamos de criar busquem o código para executar de uma fila mantida no `ThreadPool` e enviem esse código para sua thread para executar.

Os canais que aprendemos no Capítulo 16 — uma maneira simples de se comunicar entre duas threads — seriam perfeitos para este caso de uso. Usaremos um canal para funcionar como a fila de trabalhos, e `execute` enviará um trabalho do `ThreadPool` para as instâncias `Worker`, que enviarão o trabalho para sua thread. Aqui está o plano:

1. O `ThreadPool` criará um canal e manterá o remetente.
2. Cada `Worker` manterá o receptor.
3. Criaremos uma nova struct `Job` que conterá as closures que queremos enviar pelo canal.
4. O método `execute` enviará o trabalho que deseja executar através do remetente.
5. Em sua thread, o `Worker` fará um loop sobre seu receptor e executará as closures de quaisquer trabalhos que receber.

Vamos começar criando um canal em `ThreadPool::new` e mantendo o remetente na instância `ThreadPool`, conforme mostrado no Listagem 21-16. A struct `Job` não contém nada por enquanto, mas será o tipo de item que estamos enviando pelo canal.

Listagem 21-16: Modificando `ThreadPool` para armazenar o remetente de um canal que transmite instâncias `Job`

```rust
use std::sync::mpsc;
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --trecho omitido--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers, sender }
    }
    // --trecho omitido--
}
```

Em `ThreadPool::new`, criamos nosso novo canal e fazemos com que o pool mantenha o remetente. Isso compilará com sucesso.

Vamos tentar passar um receptor do canal para cada `Worker` à medida que o pool de threads cria o canal. Sabemos que queremos usar o receptor na thread que as instâncias `Worker` geram, então referenciaremos o parâmetro `receiver` na closure. O código no Listagem 21-17 ainda não compilará.

Listagem 21-17: Passando o receptor para cada `Worker`

```rust,ignore,does_not_compile
impl ThreadPool {
    // --trecho omitido--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }

        ThreadPool { workers, sender }
    }
    // --trecho omitido--
}

// --trecho omitido--

impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
```

Fizemos algumas pequenas e simples alterações: passamos o receptor para `Worker::new` e, em seguida, o usamos dentro da closure.

Quando tentamos verificar este código, obtemos este erro:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: `receiver`
  --> src/lib.rs:26:42
   |
21 |         let (sender, receiver) = mpsc::channel();
   |                      -------- move occurs because `receiver` has type `std::sync::mpsc::Receiver<Job>`, which does not implement the `Copy` trait
...
25 |         for id in 0..size {
   |         ----------------- inside of this loop
26 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here, in previous iteration of loop
```

O código está tentando passar `receiver` para várias instâncias de `Worker`. Isso não funcionará, como você deve se lembrar do Capítulo 16: a implementação do canal que Rust fornece é *múltiplos produtores, consumidor único*. Isso significa que não podemos simplesmente clonar o lado consumidor do canal para corrigir este código. Também não queremos enviar uma mensagem várias vezes para vários consumidores; queremos uma lista de mensagens com várias instâncias de `Worker` de modo que cada mensagem seja processada uma vez.

Além disso, retirar um trabalho da fila do canal envolve mutar o `receiver`, então as threads precisam de uma maneira segura de compartilhar e modificar `receiver`; caso contrário, poderíamos obter condições de corrida (como coberto no Capítulo 16).

Lembre-se dos ponteiros inteligentes thread-safe discutidos no Capítulo 16: para compartilhar a propriedade entre várias threads e permitir que as threads mutem o valor, precisamos usar `Arc<Mutex<T>>`. O tipo `Arc` permitirá que várias instâncias de `Worker` possuam o receptor, e `Mutex` garantirá que apenas um `Worker` receba um trabalho do receptor por vez. O Listagem 21-18 mostra as alterações que precisamos fazer.

Listagem 21-18: Compartilhando o receptor entre as instâncias `Worker` usando `Arc` e `Mutex`

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --trecho omitido--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    // --trecho omitido--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --trecho omitido--
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
```

Em `ThreadPool::new`, colocamos o receptor em um `Arc` e um `Mutex`. Para cada novo `Worker`, clonamos o `Arc` para aumentar a contagem de referência para que as instâncias `Worker` possam compartilhar a propriedade do receptor.

Com essas alterações, o código compila! Estamos chegando lá!

#### Implementando o Método `execute`

Vamos finalmente implementar o método `execute` em `ThreadPool`. Também mudaremos `Job` de uma struct para um apelido de tipo para um trait object que contém o tipo de closure que `execute` recebe. Como discutido na seção "Criando Sinônimos de Tipo com Apelidos de Tipo" no Capítulo 19, apelidos de tipo nos permitem tornar tipos longos mais curtos para facilitar o uso. Veja o Listagem 21-19.

Listagem 21-19: Criando um apelido de tipo `Job` para um `Box` que contém cada closure e enviando o trabalho pelo canal

```rust
// --trecho omitido--

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    // --trecho omitido--

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

// --trecho omitido--
```

Depois de criarmos uma nova instância `Job` usando a closure que recebemos em `execute`, enviamos esse trabalho pelo lado remetente do canal. Estamos chamando `unwrap` em `send` para o caso de o envio falhar. Isso pode acontecer se, por exemplo, pararmos todas as nossas threads de receber mensagens, o que significaria que o canal fechou.

Em seguida, precisamos processar os trabalhos em `Worker::new`. Neste método, passamos o receptor para a closure. Dentro da closure, fazemos um loop infinito, pedimos um trabalho ao receptor e, em seguida, executamos o trabalho. O Listagem 21-20 mostra as alterações que precisamos fazer.

Listagem 21-20: Recebendo e executando os trabalhos na thread de um `Worker`

```rust
// --trecho omitido--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!("Trabalhador {} obteve um trabalho; executando.", id);

            job();
        });

        Worker { id, thread }
    }
}
```

Aqui, primeiro chamamos `lock` no `receiver` para adquirir o mutex, e então chamamos `unwrap` para entrar em pânico em quaisquer erros. Adquirir um bloqueio pode falhar se o mutex estiver em um estado *envenenado* (poisoned state), o que pode acontecer se alguma outra thread entrou em pânico enquanto segurava o bloqueio. Em vez de o compilador impor que não atribuamos o mutex a nada para que possamos chamar `recv` diretamente, devemos atribuir o mutex a uma variável ou usar a sintaxe que usamos aqui com o método `recv` na mesma linha; isso garante que o bloqueio seja mantido enquanto estamos esperando por um trabalho.

Se adquirirmos o bloqueio no mutex, chamamos `recv` para receber um `Job` do canal. Uma chamada final para `unwrap` passa por quaisquer erros aqui, o que pode ocorrer se a thread segurando o remetente tiver desligado, semelhante a como o método `send` retorna `Err` se o receptor desligar.

A chamada para `recv` bloqueia, então, se não houver trabalho ainda, a thread atual esperará até que um trabalho fique disponível. O `Mutex<T>` garante que apenas uma thread `Worker` por vez esteja tentando solicitar um trabalho.

Nosso pool de threads está em um estado funcional agora! Dê a ele um `cargo run` e faça algumas requisições:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Trabalhador 0 obteve um trabalho; executando.
Trabalhador 2 obteve um trabalho; executando.
Trabalhador 1 obteve um trabalho; executando.
Trabalhador 3 obteve um trabalho; executando.
Trabalhador 0 obteve um trabalho; executando.
Trabalhador 2 obteve um trabalho; executando.
Trabalhador 1 obteve um trabalho; executando.
Trabalhador 3 obteve um trabalho; executando.
Trabalhador 0 obteve um trabalho; executando.
Trabalhador 2 obteve um trabalho; executando.
```

Sucesso! Agora temos um pool de threads que executa conexões assincronamente. Nunca há mais de quatro threads criadas, então nosso sistema não ficará sobrecarregado se o servidor receber muitas requisições. Se fizermos uma requisição para */sleep*, o servidor poderá atender outras requisições fazendo com que outra thread as execute.

> Nota: se você abrir */sleep* em várias abas do navegador simultaneamente, elas podem carregar uma de cada vez em intervalos de 5 segundos. Alguns navegadores da web executam várias instâncias da mesma requisição sequencialmente para fins de cache e otimização.

Depois de aprender sobre o loop `while let` no Capítulo 18, você pode estar se perguntando por que não escrevemos o código do trabalhador como mostrado no Listagem 21-21.

Listagem 21-21: Uma implementação alternativa de `Worker::new` usando `while let`

```rust,ignore,not_desired_behavior
// --trecho omitido--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            while let Ok(job) = receiver.lock().unwrap().recv() {
                println!("Trabalhador {} obteve um trabalho; executando.", id);

                job();
            }
        });

        Worker { id, thread }
    }
}
```

Este código compila e executa, mas não resulta no comportamento de threading desejado: uma requisição lenta ainda fará com que outras requisições esperem para serem processadas. A razão é um pouco sutil: a estrutura `Mutex` não tem um método público `unlock` porque a propriedade do bloqueio é baseada no tempo de vida do `MutexGuard<T>` dentro do `LockResult<MutexGuard<T>>` que o método `lock` retorna. Em tempo de compilação, o verificador de empréstimo pode então impor a regra de que um recurso guardado por um `Mutex` não pode ser acessado a menos que tenhamos o bloqueio. Mas essa implementação também pode resultar no bloqueio sendo mantido por mais tempo do que o pretendido se não formos cuidadosos com o tempo de vida do `MutexGuard<T>`.

O código no Listagem 21-20 que usa `let job = receiver.lock().unwrap().recv().unwrap();` funciona porque com `let`, quaisquer valores temporários usados na expressão no lado direito do sinal de igual são descartados imediatamente quando a instrução `let` termina. No entanto, `while let` (e `if let` e `match`) não descarta valores temporários até o final do bloco associado. No Listagem 21-21, o bloqueio permanece mantido pela duração da chamada para `job()`, o que significa que outros trabalhadores não podem receber trabalhos.
