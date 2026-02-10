# Construindo um Servidor Web Single-Threaded

Começaremos fazendo um servidor web single-threaded funcionar. Antes de começarmos, vamos ver uma visão geral rápida dos protocolos envolvidos na construção de servidores web. Os detalhes desses protocolos estão além do escopo deste livro, mas uma breve visão geral fornecerá as informações de que você precisa.

Os dois principais protocolos envolvidos em servidores web são o *Hypertext Transfer Protocol (HTTP)* e o *Transmission Control Protocol (TCP)*. Ambos os protocolos são protocolos de *requisição-resposta*, o que significa que um *cliente* inicia requisições e um *servidor* ouve as requisições e fornece uma resposta ao cliente. O conteúdo dessas requisições e respostas é definido pelos protocolos.

TCP é o protocolo de nível inferior que descreve os detalhes de como a informação vai de um servidor para outro, mas não especifica o que é essa informação. HTTP constrói sobre o TCP definindo o conteúdo das requisições e respostas. É tecnicamente possível usar HTTP com outros protocolos, mas na grande maioria dos casos, o HTTP envia seus dados por TCP. Trabalharemos com os bytes brutos das requisições e respostas TCP e HTTP.

### Ouvindo a Conexão TCP

Nosso servidor web precisa ouvir uma conexão TCP, então essa é a primeira parte em que trabalharemos. A biblioteca padrão oferece um módulo `std::net` que nos permite fazer isso. Vamos criar um novo projeto da maneira usual:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Agora insira o código no Listagem 21-1 em *src/main.rs* para começar. Este código ouvirá no endereço local `127.0.0.1:7878` por fluxos TCP de entrada. Quando receber um fluxo de entrada, imprimirá `Conexão estabelecida!`.

Listagem 21-1: Ouvindo fluxos de entrada e imprimindo uma mensagem quando recebemos um fluxo

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

Usando `TcpListener`, podemos ouvir conexões TCP no endereço `127.0.0.1:7878`. No endereço, a seção antes dos dois pontos é um endereço IP representando seu computador (este é o mesmo em todos os computadores e não representa o computador dos autores especificamente), e `7878` é a porta. Escolhemos esta porta por dois motivos: HTTP não é normalmente aceito nesta porta, então é improvável que nosso servidor entre em conflito com qualquer outro servidor web que você possa ter rodando em sua máquina, e 7878 é *rust* digitado em um telefone.

A função `bind` neste cenário funciona como a função `new` no sentido de que retornará uma nova instância de `TcpListener`. A função é chamada `bind` (ligar) porque, em redes, conectar-se a uma porta para ouvir é conhecido como "ligar-se a uma porta" (binding to a port).

A função `bind` retorna um `Result<T, E>`, o que indica que é possível que a ligação falhe, por exemplo, se executássemos duas instâncias do nosso programa e tivéssemos dois programas ouvindo a mesma porta. Como estamos escrevendo um servidor básico apenas para fins de aprendizado, não nos preocuparemos em lidar com esses tipos de erros; em vez disso, usamos `unwrap` para parar o programa se ocorrerem erros.

O método `incoming` em `TcpListener` retorna um iterador que nos dá uma sequência de fluxos (mais especificamente, fluxos do tipo `TcpStream`). Um único *fluxo* representa uma conexão aberta entre o cliente e o servidor. *Conexão* é o nome para o processo completo de requisição e resposta no qual um cliente se conecta ao servidor, o servidor gera uma resposta e o servidor fecha a conexão. Como tal, leremos do `TcpStream` para ver o que o cliente enviou e depois escreveremos nossa resposta no fluxo para enviar dados de volta ao cliente. No geral, este loop `for` processará cada conexão por vez e produzirá uma série de fluxos para lidarmos.

Por enquanto, nosso tratamento do fluxo consiste em chamar `unwrap` para encerrar nosso programa se o fluxo tiver algum erro; se não houver erros, o programa imprime uma mensagem. Adicionaremos mais funcionalidade para o caso de sucesso na próxima listagem. A razão pela qual podemos receber erros do método `incoming` quando um cliente se conecta ao servidor é que não estamos realmente iterando sobre conexões. Em vez disso, estamos iterando sobre *tentativas de conexão*. A conexão pode não ser bem-sucedida por vários motivos, muitos deles específicos do sistema operacional. Por exemplo, muitos sistemas operacionais têm um limite para o número de conexões abertas simultâneas que podem suportar; novas tentativas de conexão além desse número produzirão um erro até que algumas das conexões abertas sejam fechadas.

Vamos tentar executar este código! Invoque `cargo run` no terminal e depois carregue *127.0.0.1:7878* em um navegador web. O navegador deve mostrar uma mensagem de erro como "Conexão redefinida" porque o servidor não está enviando nenhum dado de volta atualmente. Mas quando você olha para o seu terminal, deve ver várias mensagens que foram impressas quando o navegador se conectou ao servidor!

```text
     Running `target/debug/hello`
Conexão estabelecida!
Conexão estabelecida!
Conexão estabelecida!
```

Às vezes, você verá várias mensagens impressas para uma requisição do navegador; a razão pode ser que o navegador está fazendo uma requisição para a página, bem como uma requisição para outros recursos, como o ícone *favicon.ico* que aparece na guia do navegador.

Também pode ser que o navegador esteja tentando se conectar ao servidor várias vezes porque o servidor não está respondendo com nenhum dado. Quando `stream` sai do escopo e é descartado no final do loop, a conexão é fechada como parte da implementação de `drop`. Os navegadores às vezes lidam com conexões fechadas tentando novamente, porque o problema pode ser temporário.

Os navegadores também às vezes abrem várias conexões com o servidor sem enviar nenhuma requisição, para que, se *fizerem* requisições posteriormente, essas requisições possam acontecer mais rapidamente. Quando isso ocorre, nosso servidor verá cada conexão, independentemente de haver alguma requisição sobre essa conexão. Muitas versões de navegadores baseados no Chrome fazem isso, por exemplo; você pode desativar essa otimização usando o modo de navegação privada ou usando um navegador diferente.

O fator importante é que conseguimos obter um identificador para uma conexão TCP com sucesso!

Lembre-se de parar o programa pressionando <kbd>ctrl</kbd>-<kbd>C</kbd> quando terminar de executar uma versão específica do código. Em seguida, reinicie o programa invocando o comando `cargo run` depois de fazer cada conjunto de alterações no código para garantir que você esteja executando o código mais novo.

### Lendo a Requisição

Vamos implementar a funcionalidade para ler a requisição do navegador! Para separar as preocupações de primeiro obter uma conexão e depois tomar alguma ação com a conexão, iniciaremos uma nova função para processar conexões. Nesta nova função `handle_connection`, leremos dados do fluxo TCP e os imprimiremos para que possamos ver os dados sendo enviados do navegador. Altere o código para ficar como o Listagem 21-2.

Listagem 21-2: Lendo do `TcpStream` e imprimindo os dados

```rust,no_run
use std::io::{BufRead, BufReader};
use std::net::{TcpListener, TcpStream};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Requisição: {:#?}", http_request);
}
```

Trazemos `std::io::BufReader` e `std::io::prelude` para o escopo para obter acesso a traits e tipos que nos permitem ler e escrever no fluxo. No loop `for` na função `main`, em vez de imprimir uma mensagem dizendo que fizemos uma conexão, agora chamamos a nova função `handle_connection` e passamos o `stream` para ela.

Na função `handle_connection`, criamos uma nova instância de `BufReader` que envolve uma referência ao `stream`. O `BufReader` adiciona buffer gerenciando chamadas aos métodos do trait `std::io::Read` para nós.

Criamos uma variável chamada `http_request` para coletar as linhas da requisição que o navegador envia ao nosso servidor. Indicamos que queremos coletar essas linhas em um vetor adicionando a anotação de tipo `Vec<_>`.

`BufReader` implementa o trait `std::io::BufRead`, que fornece o método `lines`. O método `lines` retorna um iterador de `Result<String, std::io::Error>` dividindo o fluxo de dados sempre que vê um byte de nova linha. Para obter cada `String`, mapeamos e desembrulhamos cada `Result`. O `Result` pode ser um erro se os dados não forem UTF-8 válidos ou se houver um problema ao ler do fluxo. Novamente, um programa de produção deve lidar com esses erros de forma mais graciosa, mas estamos optando por parar o programa no caso de erro por simplicidade.

O navegador sinaliza o fim de uma requisição HTTP enviando dois caracteres de nova linha seguidos, então, para obter uma requisição do fluxo, pegamos linhas até obtermos uma linha que seja a string vazia. Depois de coletarmos as linhas no vetor, as imprimimos usando formatação de depuração bonita para que possamos dar uma olhada nas instruções que o navegador da web está enviando ao nosso servidor.

Vamos tentar este código! Inicie o programa e faça uma requisição em um navegador da web novamente. Note que ainda receberemos uma página de erro no navegador, mas a saída do nosso programa no terminal agora será semelhante a esta:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Requisição: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Dependendo do seu navegador, você pode obter uma saída ligeiramente diferente. Agora que estamos imprimindo os dados da requisição, podemos ver por que obtemos várias conexões de uma requisição do navegador olhando para o caminho após `GET` na primeira linha da requisição. Se as conexões repetidas estiverem todas solicitando */*, sabemos que o navegador está tentando buscar */* repetidamente porque não está obtendo uma resposta do nosso programa.

Vamos dividir esses dados de requisição para entender o que o navegador está pedindo ao nosso programa.

### Olhando Mais de Perto para uma Requisição HTTP

HTTP é um protocolo baseado em texto, e uma requisição assume este formato:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

A primeira linha é a *linha de requisição* (request line) que contém informações sobre o que o cliente está solicitando. A primeira parte da linha de requisição indica o método sendo usado, como `GET` ou `POST`, que descreve como o cliente está fazendo esta requisição. Nosso cliente usou uma requisição `GET`, o que significa que está pedindo informações.

A próxima parte da linha de requisição é */*, que indica o *uniform resource identifier (URI)* que o cliente está solicitando: um URI é quase, mas não exatamente, o mesmo que um *uniform resource locator (URL)*. A diferença entre URIs e URLs não é importante para nossos propósitos neste capítulo, mas a especificação HTTP usa o termo *URI*, então podemos apenas substituir mentalmente *URL* por *URI* aqui.

A última parte é a versão HTTP que o cliente usa, e então a linha de requisição termina em uma sequência CRLF. (*CRLF* significa *carriage return* e *line feed*, que são termos dos dias da máquina de escrever!) A sequência CRLF também pode ser escrita como `\r\n`, onde `\r` é um retorno de carro e `\n` é uma alimentação de linha. A *sequência CRLF* separa a linha de requisição do restante dos dados da requisição. Note que quando o CRLF é impresso, vemos uma nova linha começar em vez de `\r\n`.

Olhando para os dados da linha de requisição que recebemos ao executar nosso programa até agora, vemos que `GET` é o método, */* é o URI da requisição e `HTTP/1.1` é a versão.

Após a linha de requisição, as linhas restantes começando de `Host:` em diante são cabeçalhos. Requisições `GET` não têm corpo.

Tente fazer uma requisição de um navegador diferente ou pedir um endereço diferente, como *127.0.0.1:7878/teste*, para ver como os dados da requisição mudam.

Agora que sabemos o que o navegador está pedindo, vamos enviar de volta alguns dados!

### Escrevendo uma Resposta

Vamos implementar o envio de dados em resposta a uma requisição do cliente. As respostas têm o seguinte formato:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

A primeira linha é uma *linha de status* (status line) que contém a versão HTTP usada na resposta, um código de status numérico que resume o resultado da requisição e uma frase de razão que fornece uma descrição de texto do código de status. Após a sequência CRLF estão quaisquer cabeçalhos, outra sequência CRLF e o corpo da resposta.

Aqui está um exemplo de resposta que usa a versão HTTP 1.1 e tem um código de status 200, uma frase de razão OK, sem cabeçalhos e sem corpo:

```text
HTTP/1.1 200 OK\r\n\r\n
```

O código de status 200 é a resposta de sucesso padrão. O texto é uma pequena resposta HTTP bem-sucedida. Vamos escrever isso no fluxo como nossa resposta a uma requisição bem-sucedida! Da função `handle_connection`, remova o `println!` que estava imprimindo os dados da requisição e substitua-o pelo código no Listagem 21-3.

Listagem 21-3: Escrevendo uma pequena resposta HTTP bem-sucedida no fluxo

```rust,no_run
use std::io::{BufRead, BufReader, Write};
use std::net::{TcpListener, TcpStream};

// --trecho omitido--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let response = "HTTP/1.1 200 OK\r\n\r\n";

    stream.write_all(response.as_bytes()).unwrap();
}
```

A primeira nova linha define a variável `response` que contém os dados da mensagem de sucesso. Em seguida, chamamos `as_bytes` em nossa `response` para converter os dados da string em bytes. O método `write_all` em `stream` recebe um `&[u8]` e envia esses bytes diretamente pela conexão. Como a operação `write_all` pode falhar, usamos `unwrap` em qualquer resultado de erro como antes. Novamente, em uma aplicação real, você adicionaria tratamento de erros aqui.

Com essas alterações, vamos executar nosso código e fazer uma requisição. Não estamos mais imprimindo nenhum dado no terminal, então não veremos nenhuma saída além da saída do Cargo. Quando você carregar *127.0.0.1:7878* em um navegador web, você deve obter uma página em branco em vez de um erro. Você acabou de codificar manualmente o recebimento de uma requisição HTTP e o envio de uma resposta!

### Retornando HTML Real

Vamos implementar a funcionalidade para retornar mais do que uma página em branco. Crie o novo arquivo *hello.html* na raiz do diretório do seu projeto, não no diretório *src*. Você pode inserir qualquer HTML que quiser; o Listagem 21-4 mostra uma possibilidade.

Listagem 21-4: Um arquivo HTML de exemplo para retornar em uma resposta

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Olá!</title>
  </head>
  <body>
    <h1>Olá!</h1>
    <p>Oi do Rust</p>
  </body>
</html>
```

Este é um documento HTML5 mínimo com um título e algum texto. Para retornar isso do servidor quando uma requisição for recebida, modificaremos `handle_connection` conforme mostrado no Listagem 21-5 para ler o arquivo HTML, adicioná-lo à resposta como um corpo e enviá-lo.

Listagem 21-5: Enviando o conteúdo de *hello.html* como o corpo da resposta

```rust,no_run
use std::fs;
use std::io::{BufRead, BufReader, Write};
use std::net::{TcpListener, TcpStream};

// --trecho omitido--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
}
```

Adicionamos `fs` à instrução `use` para trazer o módulo de sistema de arquivos da biblioteca padrão para o escopo. O código para ler o conteúdo de um arquivo para uma string deve parecer familiar; nós o usamos quando lemos o conteúdo de um arquivo para nosso projeto de E/S no Listagem 12-4.

Em seguida, usamos `format!` para adicionar o conteúdo do arquivo como o corpo da resposta de sucesso. Para garantir uma resposta HTTP válida, adicionamos o cabeçalho `Content-Length`, que é definido como o tamanho do corpo da nossa resposta—neste caso, o tamanho de `hello.html`.

Execute este código com `cargo run` e carregue *127.0.0.1:7878* no seu navegador; você deve ver seu HTML renderizado!

Atualmente, estamos ignorando os dados da requisição em `http_request` e apenas enviando de volta o conteúdo do arquivo HTML incondicionalmente. Isso significa que se você tentar solicitar *127.0.0.1:7878/outra-coisa* no seu navegador, ainda receberá de volta essa mesma resposta HTML. No momento, nosso servidor é muito limitado e não faz o que a maioria dos servidores web faz. Queremos personalizar nossas respostas dependendo da requisição e enviar de volta o arquivo HTML apenas para uma requisição bem formada para */*.

### Validando a Requisição e Respondendo Seletivamente

No momento, nosso servidor web retornará o HTML no arquivo não importa o que o cliente solicitou. Vamos adicionar funcionalidade para verificar se o navegador está solicitando */* antes de retornar o arquivo HTML e retornar um erro se o navegador solicitar qualquer outra coisa. Para isso, precisamos modificar `handle_connection`, conforme mostrado no Listagem 21-6. Este novo código verifica o conteúdo da requisição recebida em relação ao que sabemos que uma requisição para */* se parece e adiciona blocos `if` e `else` para tratar as requisições de forma diferente.

Listagem 21-6: Tratando requisições para */* de forma diferente de outras requisições

```rust,no_run
// --trecho omitido--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        // alguma outra requisição
    }
}
```

Vamos olhar apenas para a primeira linha da requisição HTTP, então, em vez de ler toda a requisição em um vetor, estamos chamando `next` para obter o primeiro item do iterador. O primeiro `unwrap` cuida do `Option` e para o programa se o iterador não tiver itens. O segundo `unwrap` lida com o `Result` e tem o mesmo efeito que o `unwrap` que estava no `map` adicionado no Listagem 21-2.

Em seguida, verificamos a `request_line` para ver se ela é igual à linha de requisição de uma requisição GET para o caminho */*. Se for, o bloco `if` retorna o conteúdo do nosso arquivo HTML.

Se a `request_line` *não* for igual à requisição GET para o caminho */*, significa que recebemos alguma outra requisição. Adicionaremos código ao bloco `else` em um momento para responder a todas as outras requisições.

Execute este código agora e solicite *127.0.0.1:7878*; você deve obter o HTML em *hello.html*. Se você fizer qualquer outra requisição, como *127.0.0.1:7878/foo*, você obterá um erro de conexão como os que viu ao executar o código no Listagem 21-1 e Listagem 21-2.

Agora vamos adicionar o código no Listagem 21-7 ao bloco `else` para retornar uma resposta com o código de status 404, que sinaliza que o conteúdo para a requisição não foi encontrado. Também retornaremos algum HTML para uma página a ser renderizada no navegador indicando a resposta ao usuário final.

Listagem 21-7: Respondendo com código de status 404 e uma página de erro se qualquer coisa diferente de */* for solicitada

```rust,no_run
    // --trecho omitido--
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
```

Aqui, nossa resposta tem uma linha de status com código de status 404 e a frase de razão `NOT FOUND`. O corpo da resposta será o HTML no arquivo *404.html*. Você precisará criar um arquivo *404.html* ao lado de *hello.html* para a página de erro; novamente, sinta-se à vontade para usar qualquer HTML que quiser ou usar o HTML de exemplo no Listagem 21-8.

Listagem 21-8: Conteúdo de exemplo para a página a ser enviada de volta com qualquer resposta 404

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Olá!</title>
  </head>
  <body>
    <h1>Ops!</h1>
    <p>Desculpe, não sei o que você está procurando.</p>
  </body>
</html>
```

Com essas alterações, execute seu servidor novamente. Solicitar *127.0.0.1:7878* deve retornar o conteúdo de *hello.html*, e qualquer outra requisição, como *127.0.0.1:7878/foo*, deve retornar o HTML de erro de *404.html*.

### Refatoração

No momento, os blocos `if` e `else` têm muita repetição: ambos estão lendo arquivos e escrevendo o conteúdo dos arquivos no fluxo. As únicas diferenças são a linha de status e o nome do arquivo. Vamos tornar o código mais conciso extraindo essas diferenças para linhas `if` e `else` separadas que atribuirão os valores da linha de status e do nome do arquivo a variáveis; podemos então usar essas variáveis incondicionalmente no código para ler o arquivo e escrever a resposta. O Listagem 21-9 mostra o código resultante após substituir os grandes blocos `if` e `else`.

Listagem 21-9: Refatorando os blocos `if` e `else` para conter apenas o código que difere entre os dois casos

```rust,no_run
// --trecho omitido--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
}
```

Agora, os blocos `if` e `else` retornam apenas os valores apropriados para a linha de status e o nome do arquivo em uma tupla; nós então usamos a desestruturação para atribuir esses dois valores a `status_line` e `filename` usando um padrão na declaração `let`, conforme discutido no Capítulo 19.

O código anteriormente duplicado agora está fora dos blocos `if` e `else` e usa as variáveis `status_line` e `filename`. Isso torna mais fácil ver a diferença entre os dois casos e significa que temos apenas um lugar para atualizar o código se quisermos mudar como a leitura do arquivo e a escrita da resposta funcionam. O comportamento do código no Listagem 21-9 será o mesmo que no Listagem 21-7.

Incrível! Agora temos um servidor web simples em aproximadamente 40 linhas de código Rust que responde a uma requisição com uma página de conteúdo e responde a todas as outras requisições com uma resposta 404.

Atualmente, nosso servidor roda em uma única thread, o que significa que ele só pode atender a uma requisição por vez. Vamos examinar como isso pode ser um problema simulando algumas requisições lentas. Em seguida, vamos corrigi-lo para que nosso servidor possa lidar com várias requisições de uma vez.
