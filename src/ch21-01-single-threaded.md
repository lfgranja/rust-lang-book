# Construindo um Servidor Web Single-Threaded

Começaremos fazendo um servidor web single-threaded funcionar. Antes de começarmos, vamos dar uma olhada rápida nos protocolos envolvidos na construção de servidores web. Os detalhes desses protocolos estão além do escopo deste livro, mas uma breve visão geral fornecerá as informações necessárias.

Os dois principais protocolos envolvidos em servidores web são o _Hypertext Transfer Protocol_ (HTTP) e o _Transmission Control Protocol_ (TCP). Ambos os protocolos são protocolos de _requisição-resposta_, o que significa que um _cliente_ inicia requisições e um _servidor_ ouve as requisições e fornece uma resposta ao cliente. O conteúdo dessas requisições e respostas é definido pelos protocolos.

O TCP é o protocolo de nível inferior que descreve os detalhes de como as informações chegam de um servidor para outro, mas não especifica o que são essas informações. O HTTP se baseia no TCP definindo o conteúdo das requisições e respostas. É tecnicamente possível usar o HTTP com outros protocolos, mas na grande maioria dos casos, o HTTP envia seus dados por TCP. Vamos trabalhar com os bytes brutos das requisições e respostas TCP e HTTP.

## Ouvindo Conexões TCP

Nosso servidor web precisa ouvir conexões TCP, então essa é a primeira parte em que trabalharemos. A biblioteca padrão oferece um módulo `std::net` que nos permite fazer isso. Vamos criar um novo projeto:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Agora insira o código da Listagem 20-1 em `src/main.rs` para começar. Este código ouvirá no endereço local `127.0.0.1` na porta `7878` por conexões TCP de entrada. Quando receber uma conexão de entrada, imprimirá `Conexão estabelecida!`.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        println!("Conexão estabelecida!");
    }
}
```

<span class="caption">Listagem 20-1: Ouvindo conexões de entrada e imprimindo uma mensagem quando recebemos uma</span>

Usamos `TcpListener`, que pode ouvir conexões TCP no endereço `127.0.0.1` e porta `7878`. Escolhemos a porta 7878 porque `7878` é _rust_ digitado em um telefone. O método `bind` retorna um `Result<T, E>`, o que indica que é possível que a ligação falhe. Por exemplo, se tentarmos nos conectar à porta 80 e não formos administradores, a ligação falhará. Ou, se executarmos duas instâncias do nosso programa e ambas tentarem ouvir na mesma porta, a ligação falhará. Como estamos escrevendo um servidor básico para fins de aprendizado, não vamos nos preocupar em lidar com esses erros; apenas usamos `unwrap` para parar o programa se ocorrerem erros.

O método `incoming` em `TcpListener` retorna um iterador que nos dá uma sequência de fluxos (streams) (mais especificamente, fluxos do tipo `TcpStream`). Um único _stream_ representa uma conexão aberta entre o cliente e o servidor. Uma _conexão_ é o nome para o processo completo de requisição e resposta em que um cliente se conecta ao servidor, o servidor gera uma resposta e o servidor fecha a conexão. Como tal, leremos do `TcpStream` para ver o que o cliente enviou e, em seguida, escreveremos no `TcpStream` para enviar nossa resposta.

O motivo pelo qual iteramos sobre `incoming` é que o listener nunca para de verificar novas conexões. Quando detecta uma nova conexão, o iterador produz um novo `TcpStream`.

O iterador `incoming` retorna um `Result` contendo o `TcpStream` ou um erro. Um erro pode acontecer se a conexão falhar por algum motivo. Novamente, para simplificar, paramos o programa se encontrarmos um erro.

Tente executar este código! Invoque `cargo run` no terminal e, em seguida, carregue `127.0.0.1:7878` em um navegador da web. O navegador deve mostrar uma mensagem de erro como "Conexão redefinida", porque o servidor não está enviando dados de volta. Mas se você olhar para o seu terminal, deverá ver várias mensagens que foram impressas quando o navegador se conectou ao servidor!

```text
     Running `target/debug/hello`
Conexão estabelecida!
Conexão estabelecida!
Conexão estabelecida!
```

Às vezes, você verá várias mensagens impressas para uma requisição do navegador; o motivo pode ser que o navegador está fazendo uma requisição para a página, bem como uma requisição para outros recursos, como o ícone `favicon.ico` que aparece na guia do navegador.

Também pode ser que o navegador esteja tentando se conectar ao servidor várias vezes porque não estamos respondendo com dados. Quando `stream` sai do escopo e é descartado no final do loop, a conexão é fechada como parte da implementação de `Drop`. Os navegadores às vezes lidam com conexões fechadas tentando reconectar, porque o problema pode ser temporário. O importante é que lidamos com sucesso com uma conexão TCP!

Lembre-se de parar o programa pressionando <span class="keystroke">ctrl-c</span> quando terminar de executar uma versão específica do código. Em seguida, reinicie `cargo run` após fazer cada conjunto de alterações de código para garantir que você esteja executando o código mais recente.

## Lendo a Requisição

Vamos implementar a funcionalidade para ler a requisição do navegador! Para separar as preocupações de primeiro obter uma conexão e depois tomar alguma ação com a conexão, iniciaremos uma nova função para processar as conexões. Nesta nova função `handle_connection`, leremos os dados do fluxo TCP e os imprimiremos para que possamos ver os dados sendo enviados pelo navegador. Mude o código para ficar como a Listagem 20-2.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;

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

    println!("Requisição: {}", String::from_utf8_lossy(&buffer[..]));
}
```

<span class="caption">Listagem 20-2: Lendo do `TcpStream` e imprimindo os dados</span>

Trazemos `std::io::prelude` para o escopo para ter acesso a certas traits que nos permitem ler e escrever no stream. Na função `main` no loop `for`, em vez de imprimir uma mensagem que diz que fizemos uma conexão, agora chamamos a nova função `handle_connection` e passamos o `stream` para ela.

Na função `handle_connection`, tornamos o parâmetro `stream` mutável. A razão é que a instância `TcpStream` mantém o controle de quais dados ele retorna para nós internamente. Ele pode ler mais dados do que pedimos e salvar esses dados para a próxima vez que pedirmos dados. Portanto, ele precisa ser `mut` porque seu estado interno pode mudar; normalmente pensamos em "ler" como não sendo uma mutação, mas, neste caso, a palavra-chave `mut` é necessária.

Em seguida, declaramos um `buffer` na pilha para conter os dados que são lidos. Criamos um buffer de 1024 bytes de tamanho, o que é grande o suficiente para conter os dados de uma requisição básica e é suficiente para nossos propósitos neste capítulo. Se quiséssemos lidar com requisições de tamanho arbitrário, o gerenciamento de buffer precisaria ser mais complicado; vamos mantê-lo simples por enquanto. Passamos o buffer para `stream.read`, que lerá bytes do `TcpStream` e os colocará no buffer.

Finalmente, convertemos os bytes no buffer em uma string e imprimimos essa string. A função `String::from_utf8_lossy` pega um `&[u8]` e produz uma `String`. A parte "lossy" (com perdas) do nome indica o comportamento desta função quando vê uma sequência UTF-8 inválida: ela substituirá a sequência inválida por `?`, o caractere de substituição . Você pode ver caracteres de substituição para caracteres no buffer que não são preenchidos por dados da requisição.

Execute o código novamente e faça uma requisição no seu navegador. Você deve ver uma saída semelhante a esta:

```text
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Requisição: GET / HTTP/1.1
Host: 127.0.0.1:7878
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Cache-Control: max-age=0
...
```

Dependendo do seu navegador, você pode ver uma saída ligeiramente diferente. Agora que estamos imprimindo os dados da requisição, podemos ver por que recebemos várias conexões de uma requisição do navegador olhando para o caminho após `GET` na primeira linha da requisição. Se as conexões repetidas estiverem todas solicitando `/`, sabemos que o navegador está tentando buscar `/` repetidamente porque não está obtendo uma resposta.

Vamos analisar esses dados de requisição para entender o que o navegador está pedindo ao nosso servidor.

## Um Olhar Mais Atento em uma Requisição HTTP

HTTP é um protocolo baseado em texto, e uma requisição tem este formato:

```text
Método Request-URI Versão-HTTP CRLF
headers CRLF
message-body
```

A primeira linha é a _linha de requisição_ que contém informações sobre o que o cliente está solicitando. A primeira parte da linha de requisição indica o _método_ sendo usado, como `GET` ou `POST`, que descreve como o cliente está fazendo essa requisição. Nosso cliente usou uma requisição `GET`, o que significa que está pedindo informações. A próxima parte da linha de requisição é `/`, que indica a _Uniform Resource Identifier_ (URI) que o cliente está solicitando: uma URI é quase, mas não exatamente, a mesma coisa que uma URL (Uniform Resource Locator). A diferença entre URIs e URLs não é importante para nossos propósitos neste capítulo, mas a especificação HTTP usa o termo URI, então podemos apenas substituir mentalmente URL por URI aqui. A última parte é a versão HTTP que o cliente usa e, em seguida, a linha de requisição termina em uma sequência _CRLF_. (CRLF significa carriage return e line feed, que são termos da época da máquina de escrever!) A sequência CRLF também pode ser escrita como `\r\n`, onde `\r` é um retorno de carro e `\n` é uma alimentação de linha. A especificação CRLF separa a linha de requisição do restante dos dados da requisição. Observe que quando o CRLF é impresso, vemos uma nova linha começar em vez de `\r\n`.

Olhando para a linha de requisição nos dados que nosso programa imprimiu até agora, vemos que `GET` é o método, `/` é a URI da requisição e `HTTP/1.1` é a versão.

Após a linha de requisição, as linhas restantes, começando de `Host:` em diante, são cabeçalhos (headers). As requisições `GET` não têm corpo (body).

Tente fazer uma requisição de um navegador diferente ou pedir um endereço diferente, como `127.0.0.1:7878/test`, para ver como os dados da requisição mudam.

Agora que sabemos o que o navegador está pedindo, vamos enviar alguns dados de volta!

## Escrevendo uma Resposta

Vamos implementar o envio de dados em resposta a uma requisição do cliente. As respostas têm o seguinte formato:

```text
Versão-HTTP Código-de-Status Frase-de-Razão CRLF
headers CRLF
message-body
```

A primeira linha é uma _linha de status_ que contém a versão HTTP usada na resposta, um código de status numérico que resume o resultado da requisição e uma frase de razão que fornece uma descrição de texto do código de status. Após a sequência CRLF estão quaisquer cabeçalhos, outra sequência CRLF e o corpo da resposta.

Aqui está um exemplo de resposta que usa a versão HTTP 1.1, tem um código de status 200, uma frase de razão OK, sem cabeçalhos e sem corpo:

```text
HTTP/1.1 200 OK\r\n\r\n
```

O código de status 200 é a resposta de sucesso padrão. O texto é uma pequena resposta HTTP de sucesso. Vamos escrever isso no fluxo como nossa resposta a uma requisição bem-sucedida! Da função `handle_connection`, remova o `println!` que estava imprimindo os dados da requisição e substitua-o pelo código na Listagem 20-3.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];

    stream.read(&mut buffer).unwrap();

    let response = "HTTP/1.1 200 OK\r\n\r\n";

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

<span class="caption">Listagem 20-3: Escrevendo uma pequena resposta HTTP bem-sucedida no fluxo</span>

O primeiro novo código define a variável `response` que contém os dados da mensagem de sucesso. Em seguida, chamamos `as_bytes` em nossa `response` para converter os dados da string em bytes. O método `write` em `stream` recebe um `&[u8]` e envia esses bytes diretamente pela conexão. Como a operação `write` pode falhar, usamos `unwrap` em qualquer resultado de erro, como antes. Novamente, em uma aplicação real, você adicionaria tratamento de erros aqui.

Finalmente, `flush` aguardará e impedirá que o programa continue até que todos os bytes sejam gravados na conexão; `TcpStream` contém um buffer interno para minimizar chamadas ao sistema operacional.

Execute este código e faça uma requisição. Seu navegador não deve mais mostrar um erro, mas apenas uma página em branco no navegador:

<img alt="Página em branco no navegador" src="img/trpl20-01.png" class="center" />

<span class="caption">Figura 20-1: Uma página em branco no seu navegador</span>

Você acabou de codificar manualmente uma resposta HTTP!

## Retornando HTML Real

Vamos implementar a funcionalidade de retornar mais do que uma página em branco. Crie um novo arquivo `hello.html` na raiz do seu diretório de projeto, não no diretório `src`. Você pode inserir qualquer HTML que desejar; A Listagem 20-4 mostra uma possibilidade.

<span class="filename">Nome do arquivo: hello.html</span>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
```

<span class="caption">Listagem 20-4: Um arquivo HTML de exemplo para retornar em uma resposta</span>

Este é um documento HTML5 mínimo com um cabeçalho e algum texto. Para retornar isso do servidor quando uma requisição for recebida, modificaremos `handle_connection` como mostrado na Listagem 20-5 para ler o arquivo HTML, adicioná-lo à resposta como o corpo e enviá-lo.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
use std::fs;
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let contents = fs::read_to_string("hello.html").unwrap();

    let response = format!(
        "HTTP/1.1 200 OK\r\nContent-Length: {}\r\n\r\n{}",
        contents.len(),
        contents
    );

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

<span class="caption">Listagem 20-5: Enviando o conteúdo de *hello.html* como o corpo da resposta</span>

Adicionamos `use std::fs` às declarações `use` para trazer o módulo de sistema de arquivos da biblioteca padrão para o escopo. O código para ler o conteúdo do arquivo para uma string deve parecer familiar.

Em seguida, usamos `format!` para adicionar o conteúdo do arquivo como o corpo da resposta de sucesso. Para garantir uma resposta HTTP válida, adicionamos o cabeçalho `Content-Length` que é definido com o tamanho do corpo da nossa resposta, neste caso o tamanho de `hello.html`.

Execute este código com `cargo run` e carregue `127.0.0.1:7878` no seu navegador; você deve ver seu HTML renderizado!

<img alt="Página HTML renderizada no navegador" src="img/trpl20-02.png" class="center" />

<span class="caption">Figura 20-2: O HTML do *hello.html* renderizado no navegador</span>

Atualmente, estamos ignorando os dados da requisição em `buffer` e enviando de volta o conteúdo do arquivo HTML incondicionalmente. Isso significa que se você tentar solicitar `127.0.0.1:7878/alguma-coisa-mais` no seu navegador, você ainda receberá essa mesma resposta HTML. Nosso servidor é muito limitado e não é o que a maioria dos servidores web faz. Queremos personalizar nossas respostas dependendo da requisição e apenas enviar de volta o arquivo HTML para uma requisição bem formada para `/`.

## Validando a Requisição e Respondendo Seletivamente

Agora vamos implementar a funcionalidade de verificar se o navegador está solicitando `/` antes de retornar o arquivo HTML e retornar um erro se o navegador estiver solicitando qualquer outra coisa. Vamos modificar `handle_connection` como mostrado na Listagem 20-6. Esse novo código verifica o conteúdo da requisição recebida em relação ao que sabemos que uma requisição para `/` se parece e adiciona blocos `if` e `else` para tratar as requisições de maneira diferente.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    if buffer.starts_with(get) {
        let contents = fs::read_to_string("hello.html").unwrap();

        let response = format!(
            "HTTP/1.1 200 OK\r\nContent-Length: {}\r\n\r\n{}",
            contents.len(),
            contents
        );

        stream.write(response.as_bytes()).unwrap();
        stream.flush().unwrap();
    } else {
        // algum outro pedido
    }
}
```

<span class="caption">Listagem 20-6: Tratando requisições para `/` de forma diferente de outras requisições</span>

Primeiro, codificamos os dados correspondentes à requisição `/` na variável `get`. Como estamos lendo bytes brutos no buffer, transformamos `get` em uma string de bytes adicionando a sintaxe de string de bytes `b""` no início dos dados de conteúdo. Em seguida, verificamos se `buffer` começa com os bytes em `get`. Se começar, significa que recebemos uma requisição bem formada para `/`, que é o caso de sucesso que tratamos no bloco `if` que retorna o conteúdo do nosso arquivo HTML.

Se `buffer` *não* começar com os bytes em `get`, significa que recebemos alguma outra requisição. Adicionaremos código ao bloco `else` na próxima listagem para responder a todas as outras requisições.

Execute este código e solicite `127.0.0.1:7878`; você deve obter o HTML em `hello.html`. Se você fizer qualquer outra requisição, como `127.0.0.1:7878/alguma-coisa-mais`, você obterá um erro de conexão como os que viu ao executar o código na Listagem 20-1 e Listagem 20-2.

Agora vamos adicionar o código à Listagem 20-7 para enviar uma resposta com o código de status 404, que sinaliza que o conteúdo da requisição não foi encontrado. Também retornaremos algum HTML para uma página a ser renderizada no navegador indicando a resposta ao usuário final.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    if buffer.starts_with(get) {
        let contents = fs::read_to_string("hello.html").unwrap();

        let response = format!(
            "HTTP/1.1 200 OK\r\nContent-Length: {}\r\n\r\n{}",
            contents.len(),
            contents
        );

        stream.write(response.as_bytes()).unwrap();
        stream.flush().unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();

        let response = format!(
            "{}\r\nContent-Length: {}\r\n\r\n{}",
            status_line,
            contents.len(),
            contents
        );

        stream.write(response.as_bytes()).unwrap();
        stream.flush().unwrap();
    }
}
```

<span class="caption">Listagem 20-7: Respondendo com o código de status 404 e uma página de erro se qualquer coisa que não seja `/` foi solicitada</span>

Aqui, nossa resposta tem uma linha de status com o código de status 404 e a frase de razão NOT FOUND. O conteúdo da resposta será o HTML no arquivo `404.html`. Você precisará criar um arquivo `404.html` próximo ao `hello.html` para a página de erro; novamente, sinta-se à vontade para usar qualquer HTML que quiser ou use o exemplo na Listagem 20-8.

<span class="filename">Nome do arquivo: 404.html</span>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
  </body>
</html>
```

<span class="caption">Listagem 20-8: Conteúdo de amostra para a página a ser enviada com qualquer resposta 404</span>

Com essas alterações, execute seu servidor novamente. Solicitar `127.0.0.1:7878` deve retornar o conteúdo de `hello.html`, e qualquer outra requisição, como `127.0.0.1:7878/foo`, deve retornar o erro HTML de `404.html`!

## Um Toque de Refatoração

No momento, os blocos `if` e `else` têm muita repetição: ambos estão lendo arquivos e escrevendo o conteúdo no fluxo. As únicas diferenças são a linha de status e o nome do arquivo. Vamos tornar o código mais conciso extraindo essas diferenças em linhas `if` e `else` separadas que atribuirão os valores da linha de status e do nome do arquivo a variáveis; podemos então usar essas variáveis incondicionalmente no código para ler o arquivo e escrever a resposta. O código resultante é mostrado na Listagem 20-9.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,no_run
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
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

<span class="caption">Listagem 20-9: Refatorando os blocos `if` e `else` para conter apenas o código que difere entre os dois casos</span>

Agora os blocos `if` e `else` retornam apenas os valores apropriados para a linha de status e o nome do arquivo em uma tupla; então usamos a desestruturação para atribuir esses dois valores a `status_line` e `filename` usando um padrão na instrução `let`, como discutimos no Capítulo 18.

O código lido anteriormente duplicado agora está fora dos blocos `if` e `else` e usa as variáveis `status_line` e `filename`. Isso facilita ver a diferença entre os dois casos e significa que temos apenas um lugar para atualizar o código se quisermos mudar a forma como a leitura de arquivos e a escrita de respostas funcionam. O comportamento do código na Listagem 20-9 será o mesmo que na Listagem 20-7.

Fantástico! Agora temos um servidor web simples em aproximadamente 40 linhas de código Rust que responde a uma requisição com uma página de conteúdo e a todas as outras requisições com uma resposta 404.

Como nosso servidor roda em uma única thread, ele só pode atender a uma solicitação por vez. Vamos examinar como isso pode ser um problema simulando algumas requisições lentas.
