## Lendo um Arquivo

Agora vamos adicionar funcionalidade para ler o arquivo especificado no argumento `file_path`. Primeiro, precisamos de um arquivo de amostra para testá-lo: usaremos um arquivo com uma pequena quantidade de texto em várias linhas com algumas palavras repetidas. A Listagem 12-3 tem um poema de Emily Dickinson que funcionará bem! Crie um arquivo chamado _poem.txt_ no nível raiz do seu projeto e insira o poema "I'm Nobody! Who are you?"

<Listing number="12-3" file-name="poem.txt" caption="Um poema de Emily Dickinson é um bom caso de teste.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Com o texto no lugar, edite _src/main.rs_ e adicione código para ler o arquivo, conforme mostrado na Listagem 12-4.

<Listing number="12-4" file-name="src/main.rs" caption="Lendo o conteúdo do arquivo especificado pelo segundo argumento">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Primeiro, trazemos uma parte relevante da biblioteca padrão com uma instrução `use`: precisamos de `std::fs` para lidar com arquivos.

Em `main`, a nova instrução `fs::read_to_string` recebe o `file_path`, abre esse arquivo e retorna um valor do tipo `std::io::Result<String>` que contém o conteúdo do arquivo.

Depois disso, adicionamos novamente uma instrução `println!` temporária que imprime o valor de `contents` após a leitura do arquivo, para que possamos verificar se o programa está funcionando até agora.

Vamos executar este código com qualquer string como o primeiro argumento de linha de comando (porque ainda não implementamos a parte de pesquisa) e o arquivo _poem.txt_ como o segundo argumento:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Ótimo! O código leu e imprimiu o conteúdo do arquivo. Mas o código tem algumas falhas. No momento, a função `main` tem várias responsabilidades: geralmente, as funções são mais claras e fáceis de manter se cada função for responsável por apenas uma ideia. O outro problema é que não estamos lidando com erros tão bem quanto poderíamos. O programa ainda é pequeno, então essas falhas não são um grande problema, mas à medida que o programa cresce, será mais difícil corrigi-las de forma limpa. É uma boa prática começar a refatorar desde o início ao desenvolver um programa, porque é muito mais fácil refatorar quantidades menores de código. Faremos isso a seguir.
