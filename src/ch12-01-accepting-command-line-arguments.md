## Aceitando Argumentos de Linha de Comando

Vamos criar um novo projeto com, como sempre, `cargo new`. Chamaremos nosso projeto de `minigrep` para distingui-lo da ferramenta `grep` que você já pode ter em seu sistema:

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

A primeira tarefa é fazer com que o `minigrep` aceite seus dois argumentos de linha de comando: o caminho do arquivo e uma string para pesquisar. Ou seja, queremos poder executar nosso programa com `cargo run`, dois hífens para indicar que os argumentos a seguir são para o nosso programa e não para o `cargo`, uma string para pesquisar e um caminho para um arquivo para pesquisar, assim:

```console
$ cargo run -- searchstring example-filename.txt
```

No momento, o programa gerado por `cargo new` não pode processar os argumentos que fornecemos a ele. Algumas bibliotecas existentes em [crates.io](https://crates.io/) podem ajudar a escrever um programa que aceita argumentos de linha de comando, mas como você está apenas aprendendo esse conceito, vamos implementar esse recurso nós mesmos.

### Lendo os Valores dos Argumentos

Para permitir que o `minigrep` leia os valores dos argumentos de linha de comando que passamos para ele, precisaremos da função `std::env::args` fornecida na biblioteca padrão do Rust. Esta função retorna um iterador dos argumentos de linha de comando passados para o `minigrep`. Cobriremos iteradores totalmente no [[ch13-00-functional-features|Capítulo 13]]. Por enquanto, você só precisa saber dois detalhes sobre iteradores: iteradores produzem uma série de valores e podemos chamar o método `collect` em um iterador para transformá-lo em uma coleção, como um vetor, que contém todos os elementos que o iterador produz.

O código na Listagem 12-1 permite que seu programa `minigrep` leia quaisquer argumentos de linha de comando passados para ele e, em seguida, colete os valores em um vetor.

<Listing number="12-1" file-name="src/main.rs" caption="Coletando os argumentos da linha de comando em um vetor e imprimindo-os">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Primeiro, trazemos o módulo `std::env` para o escopo com uma instrução `use` para que possamos usar sua função `args`. Observe que a função `std::env::args` está aninhada em dois níveis de módulos. Como discutimos no [[ch07-04-bringing-paths-into-scope-with-the-use-keyword|Capítulo 7]], nos casos em que a função desejada está aninhada em mais de um módulo, escolhemos trazer o módulo pai para o escopo em vez da função. Ao fazer isso, podemos usar facilmente outras funções de `std::env`. Também é menos ambíguo do que adicionar `use std::env::args` e depois chamar a função apenas com `args`, porque `args` pode ser facilmente confundido com uma função definida no módulo atual.

> ### A Função `args` e Unicode Inválido
>
> Observe que `std::env::args` entrará em pânico se algum argumento contiver Unicode inválido. Se o seu programa precisar aceitar argumentos contendo Unicode inválido, use `std::env::args_os` em vez disso. Essa função retorna um iterador que produz valores `OsString` em vez de valores `String`. Escolhemos usar `std::env::args` aqui por simplicidade, porque os valores `OsString` diferem por plataforma e são mais complexos de trabalhar do que os valores `String`.

Na primeira linha de `main`, chamamos `env::args` e usamos imediatamente `collect` para transformar o iterador em um vetor contendo todos os valores produzidos pelo iterador. Podemos usar a função `collect` para criar muitos tipos de coleções, então anotamos explicitamente o tipo de `args` para especificar que queremos um vetor de strings. Embora você raramente precise anotar tipos em Rust, `collect` é uma função que você geralmente precisa anotar porque o Rust não consegue inferir o tipo de coleção que você deseja.

Finalmente, imprimimos o vetor usando a macro de depuração. Vamos tentar executar o código primeiro sem argumentos e depois com dois argumentos:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Observe que o primeiro valor no vetor é `"target/debug/minigrep"`, que é o nome do nosso binário. Isso corresponde ao comportamento da lista de argumentos em C, permitindo que os programas usem o nome pelo qual foram invocados em sua execução. Geralmente é conveniente ter acesso ao nome do programa caso você queira imprimi-lo em mensagens ou alterar o comportamento do programa com base no alias de linha de comando usado para invocar o programa. Mas, para os fins deste capítulo, vamos ignorá-lo e salvar apenas os dois argumentos de que precisamos.

### Salvando os Valores dos Argumentos em Variáveis

O programa atualmente é capaz de acessar os valores especificados como argumentos de linha de comando. Agora precisamos salvar os valores dos dois argumentos em variáveis para que possamos usar os valores em todo o resto do programa. Fazemos isso na Listagem 12-2.

<Listing number="12-2" file-name="src/main.rs" caption="Criando variáveis para conter o argumento de consulta e o argumento de caminho de arquivo">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Como vimos quando imprimimos o vetor, o nome do programa ocupa o primeiro valor no vetor em `args[0]`, então estamos iniciando os argumentos no índice 1. O primeiro argumento que `minigrep` recebe é a string que estamos procurando, então colocamos uma referência ao primeiro argumento na variável `query`. O segundo argumento será o caminho do arquivo, então colocamos uma referência ao segundo argumento na variável `file_path`.

Imprimimos temporariamente os valores dessas variáveis para provar que o código está funcionando como pretendemos. Vamos executar este programa novamente com os argumentos `test` e `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Ótimo, o programa está funcionando! Os valores dos argumentos de que precisamos estão sendo salvos nas variáveis corretas. Mais tarde, adicionaremos algum tratamento de erro para lidar com certas situações errôneas em potencial, como quando o usuário não fornece argumentos; por enquanto, ignoraremos essa situação e trabalharemos na adição de recursos de leitura de arquivo.
