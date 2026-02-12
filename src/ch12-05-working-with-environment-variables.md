## Trabalhando com Variáveis de Ambiente

Melhoraremos o binário `minigrep` adicionando um recurso extra: uma opção para pesquisa que não diferencia maiúsculas de minúsculas que o usuário pode ativar por meio de uma variável de ambiente. Poderíamos tornar esse recurso uma opção de linha de comando e exigir que os usuários o digitem toda vez que quiserem aplicá-lo, mas, ao torná-lo uma variável de ambiente, permitimos que nossos usuários definam a variável de ambiente uma vez e tenham todas as suas pesquisas sem diferenciar maiúsculas de minúsculas nessa sessão de terminal.

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### Escrevendo um Teste com Falha para Pesquisa Insensível a Maiúsculas e Minúsculas

Primeiro, adicionamos uma nova função `search_case_insensitive` à biblioteca `minigrep` que será chamada quando a variável de ambiente tiver um valor. Continuaremos seguindo o processo TDD, então o primeiro passo é escrever novamente um teste com falha. Adicionaremos um novo teste para a nova função `search_case_insensitive` e renomearemos nosso teste antigo de `one_result` para `case_sensitive` para esclarecer as diferenças entre os dois testes, conforme mostrado na Listagem 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Adicionando um novo teste com falha para a função insensível a maiúsculas e minúsculas que estamos prestes a adicionar">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Observe que editamos o `contents` do teste antigo também. Adicionamos uma nova linha com o texto `"Duct tape."` usando um _D_ maiúsculo que não deve corresponder à consulta `"duct"` quando estamos pesquisando de maneira sensível a maiúsculas e minúsculas. Alterar o teste antigo dessa maneira ajuda a garantir que não quebremos acidentalmente a funcionalidade de pesquisa sensível a maiúsculas e minúsculas que já implementamos. Este teste deve passar agora e deve continuar passando enquanto trabalhamos na pesquisa insensível a maiúsculas e minúsculas.

O novo teste para a pesquisa _insensível_ a maiúsculas e minúsculas usa `"rUsT"` como sua consulta. Na função `search_case_insensitive` que estamos prestes a adicionar, a consulta `"rUsT"` deve corresponder à linha contendo `"Rust:"` com um _R_ maiúsculo e corresponder à linha `"Trust me."` mesmo que ambas tenham caixas diferentes da consulta. Este é o nosso teste com falha e ele falhará ao compilar porque ainda não definimos a função `search_case_insensitive`. Sinta-se à vontade para adicionar uma implementação de esqueleto que sempre retorna um vetor vazio, semelhante à maneira como fizemos para a função `search` na Listagem 12-16 para ver o teste compilar e falhar.

### Implementando a Função `search_case_insensitive`

A função `search_case_insensitive`, mostrada na Listagem 12-21, será quase a mesma que a função `search`. A única diferença é que colocaremos em minúsculas a `query` e cada `line` para que, seja qual for a caixa dos argumentos de entrada, eles tenham a mesma caixa quando verificarmos se a linha contém a consulta.

<Listing number="12-21" file-name="src/lib.rs" caption="Definindo a função `search_case_insensitive` para colocar a consulta e a linha em minúsculas antes de compará-las">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Primeiro, colocamos a string `query` em minúsculas e a armazenamos em uma nova variável com o mesmo nome, sombreando a `query` original. Chamar `to_lowercase` na consulta é necessário para que não importa se a consulta do usuário é `"rust"`, `"RUST"`, `"Rust"` ou `"rUsT"`, trataremos a consulta como se fosse `"rust"` e seremos insensíveis à caixa. Embora `to_lowercase` lide com Unicode básico, não será 100% preciso. Se estivéssemos escrevendo um aplicativo real, gostaríamos de fazer um pouco mais de trabalho aqui, mas esta seção é sobre variáveis de ambiente, não Unicode, então vamos deixar por isso mesmo aqui.

Observe que `query` agora é uma `String` em vez de uma fatia de string porque chamar `to_lowercase` cria novos dados em vez de referenciar dados existentes. Digamos que a consulta seja `"rUsT"`, como exemplo: essa fatia de string não contém um `u` ou `t` minúsculo para usarmos, então temos que alocar uma nova `String` contendo `"rust"`. Quando passamos `query` como um argumento para o método `contains` agora, precisamos adicionar um 'e' comercial (&) porque a assinatura de `contains` é definida para receber uma fatia de string.

Em seguida, adicionamos uma chamada para `to_lowercase` em cada `line` para colocar todos os caracteres em minúsculas. Agora que convertemos `line` e `query` para minúsculas, encontraremos correspondências não importa qual seja a caixa da consulta.

Vamos ver se esta implementação passa nos testes:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Ótimo! Eles passaram. Agora vamos chamar a nova função `search_case_insensitive` da função `run`. Primeiro, adicionaremos uma opção de configuração à struct `Config` para alternar entre pesquisa sensível a maiúsculas e minúsculas e insensível a maiúsculas e minúsculas. Adicionar esse campo causará erros de compilador porque não estamos inicializando esse campo em nenhum lugar ainda:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Adicionamos o campo `ignore_case` que contém um booleano. Em seguida, precisamos da função `run` para verificar o valor do campo `ignore_case` e usá-lo para decidir se deve chamar a função `search` ou a função `search_case_insensitive`, conforme mostrado na Listagem 12-22. Isso ainda não será compilado.

<Listing number="12-22" file-name="src/main.rs" caption="Chamando `search` ou `search_case_insensitive` com base no valor em `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Finalmente, precisamos verificar a variável de ambiente. As funções para trabalhar com variáveis de ambiente estão no módulo `env` na biblioteca padrão, que já está no escopo no topo de _src/main.rs_. Usaremos a função `var` do módulo `env` para verificar se algum valor foi definido para uma variável de ambiente chamada `IGNORE_CASE`, conforme mostrado na Listagem 12-23.

<Listing number="12-23" file-name="src/main.rs" caption="Verificando qualquer valor em uma variável de ambiente chamada `IGNORE_CASE`">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Aqui, criamos uma nova variável, `ignore_case`. Para definir seu valor, chamamos a função `env::var` e passamos a ela o nome da variável de ambiente `IGNORE_CASE`. A função `env::var` retorna um `Result` que será a variante `Ok` bem-sucedida que contém o valor da variável de ambiente se a variável de ambiente estiver definida com qualquer valor. Ela retornará a variante `Err` se a variável de ambiente não estiver definida.

Estamos usando o método `is_ok` no `Result` para verificar se a variável de ambiente está definida, o que significa que o programa deve fazer uma pesquisa insensível a maiúsculas e minúsculas. Se a variável de ambiente `IGNORE_CASE` não estiver definida com nada, `is_ok` retornará `false` e o programa executará uma pesquisa sensível a maiúsculas e minúsculas. Não nos importamos com o _valor_ da variável de ambiente, apenas se ela está definida ou não, então estamos verificando `is_ok` em vez de usar `unwrap`, `expect` ou qualquer um dos outros métodos que vimos em `Result`.

Passamos o valor na variável `ignore_case` para a instância `Config` para que a função `run` possa ler esse valor e decidir se deve chamar `search_case_insensitive` ou `search`, como implementamos na Listagem 12-22.

Vamos tentar! Primeiro, executaremos nosso programa sem a variável de ambiente definida e com a consulta `to`, que deve corresponder a qualquer linha que contenha a palavra _to_ em todas as minúsculas:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Parece que ainda funciona! Agora vamos executar o programa com `IGNORE_CASE` definido como `1`, mas com a mesma consulta `to`:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Se você estiver usando o PowerShell, precisará definir a variável de ambiente e executar o programa como comandos separados:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Isso fará com que `IGNORE_CASE` persista pelo restante da sua sessão de shell. Ele pode ser desativado com o cmdlet `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Devemos obter linhas que contenham _to_ que podem ter letras maiúsculas:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Excelente, também conseguimos linhas contendo _To_! Nosso programa `minigrep` agora pode fazer pesquisas insensíveis a maiúsculas e minúsculas controladas por uma variável de ambiente. Agora você sabe como gerenciar opções definidas usando argumentos de linha de comando ou variáveis de ambiente.

Alguns programas permitem argumentos _e_ variáveis de ambiente para a mesma configuração. Nesses casos, os programas decidem que um ou outro tem precedência. Para outro exercício por conta própria, tente controlar a sensibilidade a maiúsculas e minúsculas por meio de um argumento de linha de comando ou de uma variável de ambiente. Decida se o argumento de linha de comando ou a variável de ambiente deve ter precedência se o programa for executado com um definido como sensível a maiúsculas e minúsculas e outro definido como ignorar maiúsculas e minúsculas.

O módulo `std::env` contém muitos outros recursos úteis para lidar com variáveis de ambiente: verifique sua documentação para ver o que está disponível.
