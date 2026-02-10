# Desligamento Gracioso e Limpeza

O código no Listagem 21-20 responde a requisições assincronamente através do uso de um pool de threads, como pretendíamos. Recebemos alguns avisos sobre os campos `workers`, `id` e `thread` que não estamos usando de forma direta que nos lembrem que não estamos limpando nada. Quando usamos o método menos elegante <kbd>ctrl</kbd>-<kbd>C</kbd> para parar a thread principal, todas as outras threads são interrompidas imediatamente também, mesmo que estejam no meio do atendimento de uma requisição.

A seguir, implementaremos o trait `Drop` para chamar `join` em cada uma das threads no pool para que elas possam terminar as requisições em que estão trabalhando antes de fechar. Então implementaremos uma maneira de dizer às threads que elas devem parar de aceitar novas requisições e desligar. Para ver este código em ação, modificaremos nosso servidor para aceitar apenas duas requisições antes de desligar graciosamente seu loop.

### Implementando o Trait `Drop` em `ThreadPool`

Vamos começar implementando `Drop` em nosso pool de threads. Quando o pool for descartado, nossas threads devem todas se juntar (join) para garantir que terminem seu trabalho. O Listagem 21-22 mostra uma primeira tentativa de uma implementação de `Drop`; este código ainda não funcionará completamente.

Listagem 21-22: Juntando cada thread quando o pool de threads sai do escopo

```rust,ignore,does_not_compile
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Desligando o trabalhador {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

Primeiro, percorremos cada um dos `workers` no pool de threads. Usamos `&mut` para isso porque `self` é uma referência mutável, e também precisamos ser capazes de mutar `worker`. Para cada trabalhador, imprimimos uma mensagem dizendo que este trabalhador em particular está desligando, e então chamamos `join` na thread desse trabalhador. Se a chamada para `join` falhar, usamos `unwrap` para fazer o Rust entrar em pânico e ir para um desligamento não gracioso.

Aqui está o erro que obtemos quando compilamos este código:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0507]: cannot move out of `worker.thread` which is behind a mutable reference
  --> src/lib.rs:52:13
   |
52 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ move occurs because `worker.thread` has type `JoinHandle<()>`, which does not implement the `Copy` trait
```

O erro nos diz que não podemos chamar `join` porque só temos uma referência mutável para cada `worker` e `join` toma posse de seu argumento. Para resolver este problema, precisamos mover a thread para fora da instância `Worker` que possui `thread` para que `join` possa consumi-la. Fizemos isso no Listagem 17-15: se `Worker` contivesse um `Option<thread::JoinHandle<()>>` em vez disso, poderíamos chamar o método `take` no `Option` para mover o valor para fora do campo `Some` e deixar um `None` em seu lugar. Dessa forma, a função `join` poderia consumir o identificador da thread.

Vamos atualizar nossa definição de `Worker` assim:

```rust
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
```

Agora vamos inclinar-nos sobre o compilador para encontrar os outros lugares que precisam mudar. Verificando este código, obtemos dois erros:

```console
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `join` found for enum `Option` in the current scope
  --> src/lib.rs:52:27
   |
52 |             worker.thread.join().unwrap();
   |                           ^^^^ method not found in `Option<JoinHandle<()>>`

error[E0308]: mismatched types
  --> src/lib.rs:89:13
   |
89 |         Worker { id, thread }
   |                      ^^^^^^ expected enum `Option`, found struct `JoinHandle`
   |
   = note: expected enum `Option<JoinHandle<()>>`
            found struct `JoinHandle<_>`
help: try wrapping the expression in `Some`
   |
89 |         Worker { id, thread: Some(thread) }
   |                      ~~~~~~~~~~~~~~      ~
```

Vamos abordar o segundo erro, que aponta para o código no final de `Worker::new`; precisamos envolver o valor `thread` em `Some` quando criamos um novo `Worker`. Faça as seguintes alterações para corrigir este erro:

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --trecho omitido--

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

O primeiro erro está em nossa implementação de `Drop`. Mencionamos que pretendíamos chamar `take` no valor `Option` para mover `thread` para fora de `worker`. As alterações a seguir farão isso:

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Desligando o trabalhador {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

Como discutido no Capítulo 17, o método `take` em `Option` tira a variante `Some` e deixa `None` em seu lugar. Estamos usando `if let` para desestruturar o `Some` e obter a thread; então chamamos `join` na thread. Se a thread de um trabalhador já for `None`, sabemos que o trabalhador já teve sua thread limpa, então nada acontece nesse caso.

### Sinalizando para as Threads Pararem de Ouvir por Trabalhos

Com todas as alterações que fizemos, nosso código compila sem avisos. Mas a má notícia é que este código ainda não funciona da maneira que queremos. A chave é a lógica nas closures executadas pelas threads das instâncias `Worker`: atualmente, chamamos `join`, mas isso não desligará as threads porque elas `loop` para sempre procurando por trabalhos. Se tentarmos descartar nosso `ThreadPool` com nossa implementação atual de `drop`, a thread principal bloqueará para sempre esperando que a primeira thread termine.

Para corrigir este problema, precisaremos de uma mudança na implementação `drop` do `ThreadPool` e depois uma mudança no loop `Worker`.

Primeiro, mudaremos a implementação `drop` do `ThreadPool` para explicitamente descartar o `sender` antes de esperar que as threads terminem. O Listagem 21-23 mostra as alterações em `ThreadPool` para descartar explicitamente `sender`. Precisamos usar um `Option` para `sender` dentro de `ThreadPool` para poder tirar o `sender` no `drop`.

Listagem 21-23: Descartando explicitamente o `sender` antes de juntar as threads do trabalhador

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
// --trecho omitido--
impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        // --trecho omitido--

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Desligando o trabalhador {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

Descartar `sender` fecha o canal, o que indica que não serão enviadas mais mensagens. Quando isso acontece, todas as chamadas para `recv` que os trabalhadores fazem no loop infinito retornarão um erro. No Listagem 21-24, mudamos o loop `Worker` para sair graciosamente do loop nesse caso, o que significa que as threads terminarão quando a implementação `drop` do `ThreadPool` chamar `join` nelas.

Listagem 21-24: Saindo explicitamente do loop quando `recv` retorna um erro

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!("Trabalhador {} obteve um trabalho; executando.", id);

                    job();
                }
                Err(_) => {
                    println!("Trabalhador {} desconectado; desligando.", id);
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

Para ver este código em ação, vamos modificar `main` para aceitar apenas duas requisições antes de desligar o servidor graciosamente, como mostrado no Listagem 21-25.

Listagem 21-25: Desligando o servidor após duas requisições usando `take`

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

Você não gostaria de um servidor web real desligando após duas requisições. Este código apenas demonstra que o desligamento gracioso e a limpeza estão funcionando.

O método `take` é definido no trait `Iterator` e limita a iteração aos primeiros dois itens no máximo. O `ThreadPool` sairá do escopo no final de `main`, e a implementação `drop` será executada.

Inicie o servidor com `cargo run` e faça três requisições. A terceira requisição deve falhar, e no seu terminal você deve ver uma saída semelhante a esta:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Trabalhador 0 obteve um trabalho; executando.
Desligando.
Desligando o trabalhador 0
Trabalhador 3 desconectado; desligando.
Trabalhador 1 desconectado; desligando.
Trabalhador 2 desconectado; desligando.
Trabalhador 0 desconectado; desligando.
Desligando o trabalhador 1
Desligando o trabalhador 2
Desligando o trabalhador 3
```

Você pode ver uma ordem diferente de trabalhadores e mensagens impressas. Podemos ver como este código funciona a partir das mensagens: os trabalhadores 0 e 3 obtiveram as duas primeiras requisições. O servidor parou de aceitar conexões após a segunda conexão, e a implementação `Drop` em `ThreadPool` começa a ser executada antes mesmo que o trabalhador 3 comece seu trabalho. Descartar o `sender` desconecta todos os trabalhadores e diz para eles desligarem. Os trabalhadores desconectam quando terminam seus trabalhos, e o trabalhador 0 desconecta também.

Conseguimos! Temos um servidor web assíncrono básico e um pool de threads em funcionamento.

## Resumo

Bom trabalho! Você chegou ao final do livro! Queremos agradecer por se juntar a nós nesta viagem por Rust. Agora você está pronto para implementar seus próprios projetos em Rust e ajudar com os projetos de outras pessoas. Lembre-se de que há uma comunidade acolhedora de outros Rustáceos que adorariam ajudá-lo com quaisquer desafios que você encontrar em sua jornada em Rust.
