# Desligamento Gracioso e Limpeza

O código na Listagem 20-20 responde às solicitações de forma assíncrona usando um pool de threads, como pretendíamos. Recebemos alguns avisos sobre os campos `workers`, `id` e `thread` que não estamos usando de maneira direta, o que nos lembra que não estamos limpando nada. Quando usamos <span class="keystroke">ctrl-c</span> para parar a thread principal, todas as outras threads são interrompidas imediatamente, mesmo que estejam no meio do processamento de uma solicitação.

Vamos implementar a trait `Drop` para chamar `join` em cada uma das threads no pool para que elas possam terminar as solicitações em que estão trabalhando antes de fechar. Em seguida, implementaremos uma maneira de dizer às threads que elas devem parar de aceitar novas solicitações e desligar. Para ver esse código em ação, modificaremos nosso servidor para aceitar apenas duas solicitações antes de desligar graciosamente seu pool de threads.

## Implementando a Trait `Drop` no `ThreadPool`

Vamos começar implementando `Drop` no nosso pool de threads. Quando o pool for descartado, nossas threads devem todas se juntar (join) para garantir que terminem seu trabalho. A Listagem 20-22 mostra uma primeira tentativa de uma implementação `Drop`; este código ainda não funcionará.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Desligando trabalhador {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

<span class="caption">Listagem 20-22: Juntando cada thread quando o pool de threads sai do escopo</span>

Primeiro, percorremos cada um dos trabalhadores (`workers`) no pool de threads. Usamos `&mut` para isso porque `self` é uma referência mutável, e também precisamos ser capazes de modificar `worker`. Para cada trabalhador, imprimimos uma mensagem dizendo que esse trabalhador específico está desligando e, em seguida, chamamos `join` na thread desse trabalhador. Se a chamada para `join` falhar, usamos `unwrap` para fazer o Rust entrar em pânico e ir para um desligamento não gracioso.

Aqui está o erro quando compilamos este código:

```console
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0507]: cannot move out of `worker.thread` which is behind a mutable reference
  --> src/lib.rs:52:13
   |
52 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ move occurs because `worker.thread` has type `std::thread::JoinHandle<()>`, which does not implement the `Copy` trait
```

O erro nos diz que não podemos chamar `join` porque só temos uma referência mutável para cada `worker` e `join` assume a propriedade de seu argumento. Para resolver esse problema, precisamos mover a thread para fora da instância `Worker` que possui `thread` para que `join` possa consumi-la. Fizemos isso na Listagem 17-15: se o `Worker` contiver uma `Option<thread::JoinHandle<()>>` em vez disso, podemos chamar o método `take` na `Option` para mover o valor para fora da variante `Some` e deixar uma variante `None` em seu lugar. Desta forma, quando `Drop` for executado em um `Worker`, `thread` já terá sido movido, então não haverá nada para limpar.

Então, primeiro, precisamos mudar a definição de `Worker` desta forma:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
```

Agora precisamos que o compilador nos ajude a encontrar os outros lugares que precisam mudar. Verificando este código, obtemos dois erros:

```console
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `join` found for type `std::option::Option<std::thread::JoinHandle<()>>` in the current scope
  --> src/lib.rs:52:27
   |
52 |             worker.thread.join().unwrap();
   |                           ^^^^ method not found in `std::option::Option<std::thread::JoinHandle<()>>`

error[E0308]: mismatched types
  --> src/lib.rs:74:22
   |
74 |         Worker { id, thread }
   |                      ^^^^^^ expected enum `std::option::Option`, found struct `std::thread::JoinHandle`
   |
   = note: expected type `std::option::Option<std::thread::JoinHandle<()>>`
              found type `std::thread::JoinHandle<()>`
```

Vamos abordar o segundo erro, que aponta para o código no final de `Worker::new`; precisamos embrulhar o valor `thread` em `Some` quando criamos um novo `Worker`. Faça as seguintes alterações para corrigir esse erro:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

O primeiro erro está na nossa implementação `Drop`. Mencionamos que pretendíamos chamar `take` no valor `Option` para mover `thread` para fora de `worker`. As seguintes alterações farão isso:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Desligando trabalhador {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

Como discutido no Capítulo 17, o método `take` em `Option` tira a variante `Some` e deixa `None` em seu lugar. Estamos usando `if let` para desestruturar o `Some` e obter a thread; então chamamos `join` na thread. Se a thread de um trabalhador já for `None`, sabemos que o trabalhador já teve sua thread limpa, então nada acontece nesse caso.

## Sinalizando para as Threads Pararem de Escutar

Com todas as alterações que fizemos, nosso código compila sem avisos. Mas a má notícia é que este código ainda não funciona da maneira que queremos. A lógica `Drop` itera sobre cada `worker` e chama `join` na thread. No entanto, as threads `worker` estão em um loop infinito que aguarda eternamente por trabalhos. O `join` irá bloquear a thread atual (a thread principal, neste caso) até que a thread do trabalhador termine. Como a thread do trabalhador nunca terminará, a thread principal bloqueará para sempre esperando que a primeira thread do trabalhador termine, e o programa nunca será encerrado.

Para corrigir isso, precisamos de uma maneira de dizer às threads para parar de escutar e sair do loop infinito. Modificaremos `ThreadPool` para enviar um valor que sinalize que o trabalho deve ser interrompido. Mostramos como fazer isso na Listagem 20-23.

Nossa struct `Job` atualmente é um alias de tipo para `Box<dyn FnOnce() + Send + 'static>`. Vamos torná-lo um enum para que possamos enviar dois tipos diferentes de valores: o trabalho real que queremos fazer e o sinal de encerramento.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
enum Message {
    NewJob(Job),
    Terminate,
}
```

Agora precisamos mudar o tipo de itens que enviamos pelo canal. `sender` precisa ser do tipo `Sender<Message>` e `receiver` precisa ser do tipo `Receiver<Message>`. O `Worker` precisará receber mensagens do tipo `Message` em vez de `Job`. Quando um trabalhador recebe uma mensagem `NewJob`, ele processará o trabalho. Quando recebe uma mensagem `Terminate`, ele sairá do loop e parará.

Faça as seguintes alterações em `ThreadPool`:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

// --snip--

impl ThreadPool {
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

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(Message::NewJob(job)).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Enviando mensagem de encerramento para todos os trabalhadores.");

        for _ in &mut self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }

        println!("Desligando todos os trabalhadores.");

        for worker in &mut self.workers {
            println!("Desligando trabalhador {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

<span class="caption">Listagem 20-23: Enviando mensagens `Terminate` para os trabalhadores quando o `ThreadPool` é descartado</span>

Precisamos atualizar `Worker::new` para aceitar o novo tipo de receptor. E precisamos atualizar o loop `loop` dentro de `Worker::new` para lidar com as mensagens recebidas. Se a mensagem for `NewJob`, executamos o trabalho. Se for `Terminate`, saímos do loop.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Message>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv().unwrap();

            match message {
                Message::NewJob(job) => {
                    println!("Trabalhador {} conseguiu um trabalho; executando.", id);

                    job();
                }
                Message::Terminate => {
                    println!("Trabalhador {} foi instruído a terminar.", id);

                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

<span class="caption">Listagem 20-24: Lidando com mensagens `Terminate` no loop do trabalhador</span>

Com essas alterações, nosso código deve compilar e funcionar corretamente! Quando `ThreadPool` sair do escopo, `Drop` será executado. Ele enviará mensagens `Terminate` para o canal para cada trabalhador. Como o mutex garante que apenas um trabalhador processe uma mensagem de cada vez, cada trabalhador receberá uma mensagem `Terminate`, imprimirá uma mensagem dizendo que está terminando e, em seguida, sairá do loop infinito. Depois de sair do loop, a thread terminará.

Em seguida, o método `drop` chamará `join` em cada thread do trabalhador. Como as threads estão terminando, `join` retornará rapidamente, e a limpeza será concluída.

Para ver isso em ação, podemos modificar `main` para aceitar apenas duas requisições e depois sair do loop.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Desligando.");
}
```

<span class="caption">Listagem 20-25: Desligando o servidor após 2 requisições</span>

Você não gostaria de um servidor web real que desligasse após duas requisições, mas isso demonstra que o desligamento gracioso e a limpeza estão funcionando.

Inicie o servidor com `cargo run` e faça três requisições. As duas primeiras devem carregar a página, e o programa deve imprimir "Desligando" e sair. A terceira requisição deve falhar.

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Trabalhador 0 conseguiu um trabalho; executando.
Trabalhador 3 conseguiu um trabalho; executando.
Desligando.
Enviando mensagem de encerramento para todos os trabalhadores.
Desligando todos os trabalhadores.
Desligando trabalhador 0
Trabalhador 1 foi instruído a terminar.
Trabalhador 2 foi instruído a terminar.
Trabalhador 3 foi instruído a terminar.
Trabalhador 0 foi instruído a terminar.
Desligando trabalhador 1
Desligando trabalhador 2
Desligando trabalhador 3
```

Parabéns! Você concluiu o projeto final e agora tem um servidor web multithreaded básico.

## Resumo

Neste capítulo, usamos um pouco de tudo o que aprendemos no livro. Usamos threads e canais para concorrência, traits e trait objects para despacho dinâmico, tipos inteligentes e padrões de propriedade, e tratamento de erros. Esperamos que este projeto tenha mostrado como os conceitos de Rust se encaixam e como você pode usá-los em seus próprios projetos.

Obrigado por ler "A Linguagem de Programação Rust"! Agora vá e construa algo incrível!
