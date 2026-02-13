# Transformando Nosso Servidor Single-Threaded em um Servidor Multithreaded

No momento, o servidor processa cada requisição sequencialmente, o que significa que ele não processará uma segunda conexão até terminar de processar a primeira. Se o servidor receber mais e mais requisições, essa execução serial se tornará menos e menos ideal. Se o servidor receber uma requisição que leva muito tempo para processar, as requisições subsequentes terão que esperar até que a requisição longa termine, mesmo que as novas requisições possam ser processadas rapidamente. Vamos ver isso em ação.

## Simulando uma Requisição Lenta

Vamos ver como uma requisição de processamento lento pode afetar outras requisições feitas ao nosso servidor atual. A Listagem 20-10 implementa o tratamento de uma requisição para `/sleep` que simula uma resposta lenta que fará com que o servidor durma por 5 segundos antes de responder.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
use std::fs;
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::thread;
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();

    let response = format!(
        "{}\r\nContent-Length: {}\r\n\r\n{}",
        status_line,
        contents.len(),
        contents
    );

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

<span class="caption">Listagem 20-10: Simulando uma requisição lenta dormindo por 5 segundos</span>

Mudamos `if` para `match`? Não, mudamos o `if` para verificar várias condições.
Agora temos um `if` para verificar a requisição `/`, um `else if` para verificar a requisição `/sleep` e um bloco `else` para qualquer outra coisa.

Quando recebemos uma requisição para `/sleep`, usamos `thread::sleep` para fazer a thread atual dormir por 5 segundos. Em seguida, retornamos o status e o nome do arquivo para `hello.html`.

Inicie o servidor usando `cargo run`. Em seguida, abra duas janelas do navegador: uma para `http://127.0.0.1:7878/` e outra para `http://127.0.0.1:7878/sleep`. Se você inserir a URI `/` algumas vezes, como antes, verá que ela responde rapidamente. Mas se você inserir `/sleep` e, em seguida, carregar imediatamente `/`, verá que `/` espera até que `sleep` tenha dormido por seus 5 segundos completos antes de carregar.

Existem várias técnicas que poderíamos usar para evitar que as requisições fiquem bloqueadas atrás de uma requisição lenta; a que implementaremos é um pool de threads.

## Melhorando a Taxa de Transferência com um Pool de Threads

Um _pool de threads_ é um grupo de threads geradas que estão esperando e prontas para lidar com uma tarefa. Quando o programa recebe uma nova tarefa, ele atribui uma das threads do pool à tarefa, e essa thread processará a tarefa. As threads restantes no pool estão disponíveis para lidar com quaisquer outras tarefas que chegarem enquanto a primeira thread está processando. Quando a primeira thread termina de processar sua tarefa, ela é devolvida ao pool de threads ociosas, pronta para lidar com uma nova tarefa. Um pool de threads nos permite processar conexões simultaneamente, aumentando a taxa de transferência do nosso servidor.

Limitaremos o número de threads no pool a um número pequeno para nos proteger contra ataques de negação de serviço (DoS); se tivéssemos nosso programa criando uma nova thread para cada requisição conforme ela chega, alguém fazendo 10 milhões de requisições ao nosso servidor poderia criar threads suficientes para esgotar todos os recursos do nosso servidor e interromper o processamento de requisições.

Em vez de gerar threads ilimitadas, teremos um número fixo de threads esperando no pool. À medida que as requisições chegam, elas são enviadas para o pool para processamento. O pool manterá uma fila de requisições recebidas. Cada uma das threads no pool pegará uma requisição dessa fila, lidará com ela e, em seguida, pedirá outra requisição da fila. Com esse design, podemos processar até `N` requisições simultaneamente, onde `N` é o número de threads. Se cada thread estiver ocupada processando uma requisição longa, as requisições subsequentes ainda farão backup na fila, mas aumentamos o número de requisições longas que podemos lidar antes de atingirmos esse ponto.

Essa técnica é apenas uma das muitas maneiras de melhorar a taxa de transferência de um servidor web. Outras opções que você pode explorar são o modelo _fork/join_, o modelo de _thread única assíncrona_ ou o modelo de _multi-thread assíncrona_. Se você estiver interessado neste tópico, leia mais sobre outras soluções e tente implementá-las; com uma linguagem de baixo nível como Rust, todas essas opções são possíveis.

Antes de começarmos a implementar um pool de threads, vamos falar sobre como deve ser o uso do pool. Quando você está tentando projetar um código, escrever a interface do cliente primeiro pode ajudar a orientar seu design. Escreva a API do código de forma que seja estruturada da maneira que você deseja chamá-la; em seguida, implemente a funcionalidade dentro dessa estrutura, em vez de implementar a funcionalidade e depois projetar a API pública.

Semelhante a como usamos o Desenvolvimento Guiado por Testes no projeto de E/S no Capítulo 12, usaremos o Desenvolvimento Guiado pelo Compilador aqui. Escreveremos o código que chama as funções que desejamos e, em seguida, olharemos para os erros do compilador para determinar o que devemos alterar a seguir para fazer o código funcionar.

### Estrutura de Código se Pudéssemos Gerar uma Thread para Cada Requisição

Primeiro, vamos explorar como nosso código poderia parecer se criássemos uma nova thread para cada conexão. Como mencionado anteriormente, essa não é nossa solução final devido aos problemas com a criação potencial de um número ilimitado de threads, mas é um ponto de partida para obter um servidor multithreaded funcionando primeiro. Em seguida, adicionaremos o pool de threads como uma melhoria, e refatorar o código será mais fácil do que adicionar tudo de uma vez. A Listagem 20-11 mostra as alterações a serem feitas em `main` para gerar uma nova thread para lidar com cada fluxo dentro do loop `for`.

<span class="filename">Nome do arquivo: src/main.rs</span>

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

<span class="caption">Listagem 20-11: Gerando uma nova thread para cada fluxo</span>

Como aprendemos no Capítulo 16, `thread::spawn` criará uma nova thread e executará o código na closure nela. Se você executar este código, verá que carregar `/sleep` no navegador, seguido imediatamente por carregar `/` em duas outras guias do navegador, fará com que as requisições para `/` terminem imediatamente, enquanto `/sleep` é executado em segundo plano.

No entanto, como mencionamos, isso acabará sobrecarregando o sistema se você receber muitas requisições. Então, vamos mudar nosso código para usar um pool de threads.

### Criando um Número Finito de Threads

Queremos que nosso pool de threads funcione de maneira familiar e semelhante, para que mudar de threads para um pool de threads não exija grandes alterações no código que usa nosso pool. A Listagem 20-12 mostra a interface hipotética para uma estrutura `ThreadPool` que queremos usar em vez de `thread::spawn`.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
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

<span class="caption">Listagem 20-12: Nossa interface hipotética `ThreadPool`</span>

Usamos `ThreadPool::new` para criar um novo pool de threads com um número configurável de threads, neste caso quatro. Em seguida, no loop `for`, usamos `pool.execute` com uma interface semelhante a `thread::spawn` que pega uma closure que o pool deve executar para cada fluxo. Precisamos implementar `ThreadPool` e `ThreadPool::execute` para que eles funcionem assim.

### Construindo `ThreadPool` Usando Desenvolvimento Guiado pelo Compilador

Vamos continuar e tentar compilar a Listagem 20-12 para obter nosso primeiro erro.

```console
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:11:16
   |
11 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type `ThreadPool`
```

O compilador reclama que estamos usando um tipo `ThreadPool` que não definimos. O que fazemos a seguir? Definimos o tipo! O próximo passo é criar uma struct `ThreadPool`. Vamos colocar a struct `ThreadPool` em um módulo de biblioteca, para que possamos testá-la separadamente e possivelmente reutilizá-la em outros projetos. Crie um arquivo `src/lib.rs` que contém a definição da struct `ThreadPool`, que é a coisa mais simples que podemos fazer para corrigir o erro atual.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
pub struct ThreadPool;
```

Em seguida, precisamos trazer `ThreadPool` para o escopo em `src/main.rs`.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
use hello::ThreadPool;
```

Tente executar `cargo check` novamente.

```console
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:12:28
   |
12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`
```

Este erro indica que precisamos criar uma função associada chamada `new` para `ThreadPool`. Também sabemos que `new` precisa ter um parâmetro que pode aceitar `4` como argumento e deve retornar uma instância `ThreadPool`. Vamos implementar a função `new` mais simples que terá essas características:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

Escolhemos `usize` como o tipo do parâmetro `size`, porque sabemos que um número negativo de threads não faz sentido. Também sabemos que usaremos esse 4 como o número de elementos em uma coleção de threads, que é para o que `usize` serve, como discutido na seção "Tipos de Dados Inteiros" do Capítulo 3.

Vamos verificar o código novamente:

```console
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:17:14
   |
17 |         pool.execute(|| {
   |              ^^^^^^^ method not found in `ThreadPool`
```

O erro ocorre porque não definimos o método `execute` em `ThreadPool`. Lembre-se da seção "Criando um Número Finito de Threads" que decidimos que nosso pool de threads deve ter uma interface semelhante a `thread::spawn`. Além disso, implementaremos a função `execute` para que ela receba a closure que é dada e a dê a uma thread ociosa no pool para execução.

Definiremos o método `execute` em `ThreadPool` para receber uma closure como parâmetro. Lembre-se do Capítulo 13 que podemos usar traits como limites para parâmetros genéricos, e as três traits que podemos usar para closures são `Fn`, `FnMut` e `FnOnce`. Precisamos decidir que tipo de closure usar aqui. Sabemos que acabaremos fazendo algo semelhante à implementação da biblioteca padrão `thread::spawn`, então podemos ver quais limites a assinatura de `thread::spawn` tem em seu parâmetro. A documentação nos mostra o seguinte:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

O parâmetro de tipo `F` é o que nos interessa aqui; o parâmetro de tipo `T` está relacionado ao valor de retorno, e não estamos preocupados com isso. Podemos ver que `spawn` usa `FnOnce` como a trait bound em `F`. Isso é o que queremos também, porque eventualmente passaremos o argumento que recebemos em `execute` para `spawn`. Podemos ter certeza ainda maior de que `FnOnce` é a trait que queremos usar porque a thread para executar uma requisição executará a closure dessa requisição apenas uma vez, o que corresponde ao `Once` em `FnOnce`.

O parâmetro de tipo `F` também tem a trait bound `Send` e o tempo de vida bound `'static`, que são úteis em nossa situação: precisamos de `Send` para transferir a closure de uma thread para outra e `'static` porque não sabemos quanto tempo a thread levará para executar. Vamos criar um método `execute` em `ThreadPool` que receberá um parâmetro genérico do tipo `F` com essas bounds:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
pub struct ThreadPool;

impl ThreadPool {
    // --snip--
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}
```

Ainda usamos `()` após `FnOnce` porque este `FnOnce` representa uma closure que não recebe parâmetros e retorna o tipo unitário `()`. Assim como definições de função, o tipo de retorno pode ser omitido da assinatura, mas mesmo que não tenhamos parâmetros, ainda precisamos dos parênteses.

Novamente, esta é a implementação mais simples do método `execute`: ele não faz nada, mas estamos apenas tentando fazer nosso código compilar. Vamos verificar novamente:

```console
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
```

Está compilando! Mas observe que se você tentar `cargo run` e fizer uma requisição no navegador, você verá os erros no navegador que vimos no início do capítulo. Nossa biblioteca ainda não está chamando a closure passada para `execute`!

> Observação: Uma implementação que você pode ter ouvido falar é `rayon`. `ThreadPool` é um nome comum para essa funcionalidade. Estamos implementando um pool de threads básico para fins educacionais.

### Validando o Número de Threads em `new`

Não estamos fazendo nada com os parâmetros para `new` e `execute`. Vamos implementar os corpos dessas funções com o comportamento que queremos. Para começar, vamos pensar em `new`. Anteriormente, escolhemos um tipo não assinado para o parâmetro `size`, porque um pool com um número negativo de threads não faz sentido. No entanto, um pool com zero threads também não faz sentido, porque não seríamos capazes de processar nenhuma requisição. Vamos adicionar código para verificar se `size` é maior que zero antes de retornarmos uma instância `ThreadPool` e fazer o programa entrar em pânico se receber um zero usando a macro `assert!`:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }

    // --snip--
}
```

Adicionamos doc comments para nossa struct `ThreadPool` e função `new` também.

### Criando Espaço para Armazenar as Threads

Agora que temos uma maneira de saber que temos um número válido de threads para armazenar no pool, podemos criar essas threads e armazená-las na struct `ThreadPool` antes de retorná-la. Mas como "armazenamos" uma thread? Vamos dar outra olhada na assinatura de `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

A função `spawn` retorna um `JoinHandle<T>`, onde `T` é o tipo que a closure retorna. Vamos tentar usar `JoinHandle` também e ver o que acontece. No nosso caso, as closures que estamos passando para o pool de threads lidarão com a conexão e não retornarão nada, então `T` será o tipo unitário `()`.

O código na Listagem 20-13 compilará, mas ainda não cria nenhuma thread. Mudamos a definição de `ThreadPool` para conter um vetor de instâncias `thread::JoinHandle<()>`, inicializamos o vetor com uma capacidade de `size`, configuramos um loop `for` que executará algum código para criar as threads e retornamos uma instância `ThreadPool` contendo as threads.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // create some threads and store them in the vector
        }

        ThreadPool { threads }
    }
    // --snip--
}
```

<span class="caption">Listagem 20-13: Criando um vetor para armazenar as threads em `ThreadPool`</span>

Trouxemos `std::thread` para o escopo na crate da biblioteca, porque estamos usando `thread::JoinHandle` como o tipo dos itens no vetor em `ThreadPool`.

Depois que um tamanho válido é recebido, nosso `ThreadPool` cria um novo vetor que pode conter `size` itens. Usamos `with_capacity` em vez de `new`, porque sabemos que precisaremos de espaço para exatamente `size` elementos no vetor. Fazer essa pré-alocação é ligeiramente mais eficiente do que usar `Vec::new`, que redimensiona o vetor à medida que elementos são inseridos.

Quando você executa `cargo check` novamente, ele deve ter sucesso.

### Uma Struct `Worker` Responsável por Enviar Código do `ThreadPool` para uma Thread

Deixamos um comentário no loop `for` na Listagem 20-13 sobre a criação de threads. Aqui, veremos como realmente criamos threads. A biblioteca padrão fornece `thread::spawn` como uma maneira de criar threads, e `thread::spawn` espera receber algum código que a thread deve executar assim que a thread for criada. No entanto, em nosso caso, queremos criar as threads e fazê-las _esperar_ por código que enviaremos mais tarde. A implementação de threads da biblioteca padrão não inclui nenhuma maneira de fazer isso; temos que implementá-la manualmente.

Implementaremos esse comportamento introduzindo uma nova estrutura de dados entre o `ThreadPool` e as threads que gerenciará o novo comportamento. Chamaremos essa estrutura de dados de `Worker`, que é um termo comum em implementações de pool. Imagine pessoas trabalhando em uma cozinha de restaurante: os trabalhadores esperam até que os pedidos cheguem de clientes e, em seguida, são responsáveis por atender a esses pedidos e cumpri-los.

Em vez de armazenar um vetor de instâncias `JoinHandle<()>`, armazenaremos instâncias da nossa struct `Worker`. Cada `Worker` armazenará uma única instância `JoinHandle<()>`. Em seguida, implementaremos um método em `Worker` que receberá uma closure de código para executar e a enviará para a thread já em execução para execução. Também daremos a cada trabalhador um `id` para que possamos distinguir entre os diferentes trabalhadores no pool ao registrar ou depurar.

O processo de criação de um `Worker` e envio de código para ele será:

1. Defina uma struct `Worker` que contenha um `id` e um `JoinHandle<()>`.
2. Mude `ThreadPool` para conter um vetor de instâncias `Worker`.
3. Defina uma função `Worker::new` que receba um número `id` e retorne uma instância `Worker` que contém o `id` e uma thread gerada com uma closure vazia.
4. Em `ThreadPool::new`, use o contador de loop `for` para gerar um `id`, crie um novo `Worker` com esse `id` e armazene o trabalhador no vetor.

Se você está pronto para um desafio, tente implementar essas alterações por conta própria antes de olhar o código na Listagem 20-14.

Pronto? Aqui está a Listagem 20-14 com uma maneira de fazer as modificações anteriores.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers }
    }
    // --snip--
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

<span class="caption">Listagem 20-14: Modificando `ThreadPool` para conter instâncias `Worker` em vez de segurar threads diretamente</span>

Mudamos o nome do campo em `ThreadPool` de `threads` para `workers` porque ele agora contém instâncias `Worker` em vez de instâncias `JoinHandle<()>`. Usamos o contador no loop `for` como um argumento para `Worker::new`, e armazenamos cada novo `Worker` no vetor chamado `workers`.

O código externo (como nosso servidor em `src/main.rs`) não precisa saber os detalhes de implementação sobre o uso de uma struct `Worker` dentro de `ThreadPool`, então tornamos a struct `Worker` e sua função `new` privadas. A função `Worker::new` usa o `id` que damos e armazena uma instância `JoinHandle<()>` que é criada gerando uma nova thread usando uma closure vazia.

Este código compilará e armazenará o número de instâncias `Worker` que especificamos como um argumento para `ThreadPool::new`. Mas _ainda_ não estamos processando a closure que recebemos em `execute`. Vamos ver como fazer isso a seguir.

### Enviando Requisições para Threads via Canais

Para enviar as closures para as threads, usaremos _canais_, como aprendemos no Capítulo 16. Vamos usar um canal para funcionar como a fila de tarefas, e `execute` enviará tarefas para o pool, e as threads do pool estarão escutando no canal por tarefas.

Aqui está o plano:

1. O `ThreadPool` criará um canal e manterá o lado de envio (sender) do canal.
2. Cada `Worker` manterá o lado de recepção (receiver) do canal.
3. Criaremos uma nova struct `Job` que conterá as closures que queremos enviar pelo canal.
4. O método `execute` enviará o trabalho que deseja executar pelo lado de envio do canal.
5. Em sua thread, o `Worker` fará um loop sobre seu lado de recepção do canal e executará as closures de qualquer trabalho que receber.

Vamos começar criando um canal em `ThreadPool::new` e mantendo o lado de envio na instância `ThreadPool`, como mostrado na Listagem 20-15. A struct `Job` não contém nada por enquanto, mas será o tipo de item que enviaremos pelo canal.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
use std::sync::mpsc;
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers, sender }
    }
    // --snip--
}
```

<span class="caption">Listagem 20-15: Modificando `ThreadPool` para armazenar o remetente de um canal que envia instâncias `Job`</span>

Em `ThreadPool::new`, criamos nosso novo canal e fazemos com que o pool mantenha o remetente. Isso compilará com sucesso.

Vamos tentar passar um receptor do canal para cada trabalhador à medida que o pool de threads cria o canal. Sabemos que queremos usar o receptor na thread que os trabalhadores geram, então referenciamos o parâmetro `receiver` na closure. O código na Listagem 20-16 ainda não compilará.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }

        ThreadPool { workers, sender }
    }
    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
```

<span class="caption">Listagem 20-16: Passando o receptor para os trabalhadores</span>

Fizemos algumas mudanças pequenas e triviais: passamos o receptor para `Worker::new` e, em seguida, o usamos dentro da closure.

Quando tentamos verificar este código, obtemos este erro:

```console
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: `receiver`
  --> src/lib.rs:27:42
   |
22 |         let (sender, receiver) = mpsc::channel();
   |                      -------- move occurs because `receiver` has type `std::sync::mpsc::Receiver<Job>`, which does not implement the `Copy` trait
...
27 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here, in previous iteration of loop
```

O código tenta passar `receiver` para vários trabalhadores `Worker`. Isso não funcionará, como você deve se lembrar do Capítulo 16: a implementação do canal que o Rust fornece é _multiple producer, single consumer_. Isso significa que não podemos apenas clonar a extremidade consumidora do canal para consertar esse código. Não queremos enviar várias mensagens para vários consumidores; queremos uma lista de mensagens com vários trabalhadores (consumers) de modo que cada mensagem seja processada uma vez.

Além disso, tirar um trabalho da fila do canal envolve mudar o `receiver`, então as threads precisam de uma maneira segura de compartilhar e modificar `receiver`; caso contrário, poderíamos ter condições de corrida (como abordado no Capítulo 16).

Lembre-se dos ponteiros inteligentes thread-safe discutidos no Capítulo 16: para compartilhar a propriedade entre várias threads e permitir que as threads mutem o valor, precisamos usar `Arc<Mutex<T>>`. O `Arc` permitirá que vários trabalhadores possuam o receptor, e o `Mutex` garantirá que apenas um trabalhador obtenha um trabalho do receptor por vez. A Listagem 20-17 mostra as alterações que precisamos fazer.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
use std::sync::mpsc;
use std::sync::Arc;
use std::sync::Mutex;
use std::thread;

// --snip--

impl ThreadPool {
    // --snip--
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

    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
    }
}
```

<span class="caption">Listagem 20-17: Compartilhando o receptor entre os trabalhadores usando `Arc` e `Mutex`</span>

Em `ThreadPool::new`, colocamos o receptor em um `Arc` e um `Mutex`. Para cada novo trabalhador, clonamos o `Arc` para aumentar a contagem de referências para que os trabalhadores possam compartilhar a propriedade do receptor.

Com essas mudanças, o código compila! Estamos chegando lá!

### Implementando o Método `execute`

Vamos finalmente implementar o método `execute` em `ThreadPool`. Também mudaremos `Job` de uma struct para um type alias para uma trait object que contém o tipo de closure que `execute` recebe. Como discutido na seção "Criando um Sinônimo de Tipo com Aliases de Tipo" do Capítulo 19, aliases de tipo nos permitem tornar tipos longos mais curtos para facilitar o uso. Veja a Listagem 20-18.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
// --snip--

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

// --snip--
```

<span class="caption">Listagem 20-18: Criando um alias de tipo `Job` para um `Box` que contém cada closure e, em seguida, enviando o job pelo canal</span>

Depois de criar uma nova instância `Job` usando a closure que recebemos em `execute`, enviamos esse job para a extremidade remetente do canal. Estamos chamando `unwrap` em `send` para o caso de o envio falhar. Isso pode acontecer se, por exemplo, pararmos todas as nossas threads de receber mensagens, o que significa que o canal está fechado; mas como nosso pool de threads deve estar em execução o tempo todo, podemos simplesmente usar `unwrap`.

Se tentarmos `cargo check`, compila! Mas ainda não estamos processando o trabalho no trabalhador. Vamos fazer isso.

### Implementando o Código no `Worker`

Consumiremos o trabalho no método `new` do trabalhador. Na closure que passamos para `thread::spawn`, ainda referenciamos o lado receptor do canal. Mas, em vez de `receiver`, a closure precisa bloquear e esperar por um trabalho, e então executá-lo. Precisamos bloquear no método `lock` do mutex para garantir que apenas um trabalhador receba um trabalho por vez.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!("Trabalhador {} conseguiu um trabalho; executando.", id);

            job();
        });

        Worker { id, thread }
    }
}
```

<span class="caption">Listagem 20-19: Recebendo e executando os trabalhos na thread do trabalhador</span>

Aqui, primeiro chamamos `lock` no `receiver` para adquirir o mutex e, em seguida, chamamos `unwrap` para entrar em pânico em qualquer erro. Adquirir um bloqueio pode falhar se o mutex estiver em um estado _envenenado_, o que pode acontecer se alguma outra thread entrou em pânico enquanto mantinha o bloqueio. Em vez de liberar o bloqueio, chamamos `recv` para receber um `Job` do canal. Uma chamada final para `unwrap` nos move além de quaisquer erros aqui também, o que pode ocorrer se a thread que segura o remetente tiver desligado, semelhante a como o método `send` retorna `Err` se o receptor desligar.

A chamada para `recv` bloqueia, então se não houver trabalho ainda, a thread atual esperará até que um trabalho fique disponível. O `Mutex<T>` garante que apenas uma thread `Worker` por vez esteja tentando solicitar um trabalho.

Nossa implementação de `ThreadPool` agora está completa! Se você executar seu servidor agora, ele deverá ser capaz de atender a outras solicitações enquanto processa uma solicitação `/sleep`.

Espero que você tenha notado um detalhe interessante no código acima: colocamos o `lock` e o `recv` na mesma linha e não atribuímos o `MutexGuard` retornado por `lock` a uma variável. Se tivéssemos escrito o código assim:

```rust
let job = receiver.lock().unwrap().recv().unwrap();
```

vs

```rust
let receiver_lock = receiver.lock().unwrap();
let job = receiver_lock.recv().unwrap();
```

O código funcionaria? Sim. Mas há uma diferença sutil. No código de uma linha, o tempo de vida do `MutexGuard` temporário termina assim que a instrução `let job` termina. Isso significa que o bloqueio é liberado *antes* de chamarmos `job()`. Se atribuirmos o bloqueio a uma variável `receiver_lock`, o bloqueio será mantido até que `receiver_lock` saia do escopo no final da iteração do loop. Isso significaria que outras threads não poderiam receber trabalhos enquanto a thread atual estivesse executando `job()`, o que derrotaria o propósito do pool de threads de executar tarefas em paralelo!

### Testando

Vamos verificar se tudo funciona. Execute `cargo run` e faça algumas requisições.

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

    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Trabalhador 0 conseguiu um trabalho; executando.
Trabalhador 2 conseguiu um trabalho; executando.
Trabalhador 1 conseguiu um trabalho; executando.
Trabalhador 3 conseguiu um trabalho; executando.
```

Temos alguns avisos porque não estamos usando os campos `workers`, `id` e `thread` de uma maneira que o Rust saiba que é útil (por exemplo, lendo-os). Vamos limpar esses avisos no próximo passo.

Mas funciona! Nosso pool de threads está respondendo a requisições de forma assíncrona.
