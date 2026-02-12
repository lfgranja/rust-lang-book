<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## Redirecionando Erros para Erro Padrão

No momento, estamos escrevendo toda a nossa saída no terminal usando a macro `println!`. Na maioria dos terminais, existem dois tipos de saída: _saída padrão_ (`stdout`) para informações gerais e _erro padrão_ (`stderr`) para mensagens de erro. Essa distinção permite que os usuários escolham direcionar a saída bem-sucedida de um programa para um arquivo, mas ainda imprimir mensagens de erro na tela.

A macro `println!` só é capaz de imprimir na saída padrão, então temos que usar outra coisa para imprimir no erro padrão.

### Verificando Onde os Erros São Escritos

Primeiro, vamos observar como o conteúdo impresso pelo `minigrep` está sendo gravado na saída padrão, incluindo quaisquer mensagens de erro que queremos gravar no erro padrão. Faremos isso redirecionando o fluxo de saída padrão para um arquivo enquanto causamos intencionalmente um erro. Não redirecionaremos o fluxo de erro padrão, portanto, qualquer conteúdo enviado para o erro padrão continuará sendo exibido na tela.

Espera-se que os programas de linha de comando enviem mensagens de erro para o fluxo de erro padrão para que ainda possamos ver mensagens de erro na tela, mesmo que redirecionemos o fluxo de saída padrão para um arquivo. Nosso programa não está se comportando bem atualmente: estamos prestes a ver que ele salva a saída da mensagem de erro em um arquivo!

Para demonstrar esse comportamento, executaremos o programa com `>` e o caminho do arquivo, _output.txt_, para o qual queremos redirecionar o fluxo de saída padrão. Não passaremos nenhum argumento, o que deve causar um erro:

```console
$ cargo run > output.txt
```

A sintaxe `>` diz ao shell para gravar o conteúdo da saída padrão em _output.txt_ em vez da tela. Não vimos a mensagem de erro que esperávamos impressa na tela, o que significa que ela deve ter acabado no arquivo. Isto é o que _output.txt_ contém:

```text
Problem parsing arguments: not enough arguments
```

Sim, nossa mensagem de erro está sendo impressa na saída padrão. É muito mais útil para mensagens de erro como esta serem impressas no erro padrão para que apenas dados de uma execução bem-sucedida acabem no arquivo. Vamos mudar isso.

### Imprimindo Erros no Erro Padrão

Usaremos o código na Listagem 12-24 para alterar como as mensagens de erro são impressas. Por causa da refatoração que fizemos anteriormente neste capítulo, todo o código que imprime mensagens de erro está em uma função, `main`. A biblioteca padrão fornece a macro `eprintln!` que imprime no fluxo de erro padrão, então vamos alterar os dois locais onde estávamos chamando `println!` para imprimir erros para usar `eprintln!` em vez disso.

<Listing number="12-24" file-name="src/main.rs" caption="Escrevendo mensagens de erro no erro padrão em vez da saída padrão usando `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Vamos agora executar o programa novamente da mesma maneira, sem argumentos e redirecionando a saída padrão com `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Agora vemos o erro na tela e _output.txt_ não contém nada, que é o comportamento que esperamos de programas de linha de comando.

Vamos executar o programa novamente com argumentos que não causam erro, mas ainda redirecionam a saída padrão para um arquivo, assim:

```console
$ cargo run -- to poem.txt > output.txt
```

Não veremos nenhuma saída no terminal, e _output.txt_ conterá nossos resultados:

<span class="filename">Nome do arquivo: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Isso demonstra que agora estamos usando a saída padrão para saída bem-sucedida e erro padrão para saída de erro, conforme apropriado.

## Resumo

Este capítulo recapitulou alguns dos principais conceitos que você aprendeu até agora e abordou como executar operações comuns de E/S em Rust. Usando argumentos de linha de comando, arquivos, variáveis de ambiente e a macro `eprintln!` para imprimir erros, você agora está preparado para escrever aplicativos de linha de comando. Combinado com os conceitos dos capítulos anteriores, seu código será bem organizado, armazenará dados efetivamente nas estruturas de dados apropriadas, lidará com erros de forma agradável e será bem testado.

Em seguida, exploraremos alguns recursos do Rust que foram influenciados por linguagens funcionais: closures e iteradores.
