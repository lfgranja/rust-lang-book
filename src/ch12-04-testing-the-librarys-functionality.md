<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## Adicionando Funcionalidade com Desenvolvimento Orientado a Testes

Agora que temos a lógica de pesquisa em _src/lib.rs_ separada da função `main`, é muito mais fácil escrever testes para a funcionalidade principal do nosso código. Podemos chamar funções diretamente com vários argumentos e verificar valores de retorno sem ter que chamar nosso binário da linha de comando.

Nesta seção, adicionaremos a lógica de pesquisa ao programa `minigrep` usando o processo de desenvolvimento orientado a testes (TDD) com as seguintes etapas:

1. Escreva um teste que falhe e execute-o para ter certeza de que ele falha pelo motivo esperado.
2. Escreva ou modifique apenas o código suficiente para fazer o novo teste passar.
3. Refatore o código que você acabou de adicionar ou alterar e certifique-se de que os testes continuem passando.
4. Repita a partir do passo 1!

Embora seja apenas uma das muitas maneiras de escrever software, o TDD pode ajudar a impulsionar o design de código. Escrever o teste antes de escrever o código que faz o teste passar ajuda a manter a alta cobertura de teste durante todo o processo.

Testaremos a implementação da funcionalidade que realmente fará a pesquisa da string de consulta no conteúdo do arquivo e produzirá uma lista de linhas que correspondem à consulta. Adicionaremos essa funcionalidade em uma função chamada `search`.

### Escrevendo um Teste que Falha

Em _src/lib.rs_, adicionaremos um módulo `tests` com uma função de teste, como fizemos no [[ch11-01-writing-tests#the-anatomy-of-a-test-function|Capítulo 11]]. A função de teste especifica o comportamento que queremos que a função `search` tenha: ela receberá uma consulta e o texto a ser pesquisado e retornará apenas as linhas do texto que contêm a consulta. A Listagem 12-15 mostra este teste.

<Listing number="12-15" file-name="src/lib.rs" caption="Criando um teste com falha para a função `search` para a funcionalidade que desejamos ter">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Este teste procura a string `"duct"`. O texto que estamos pesquisando tem três linhas, das quais apenas uma contém `"duct"` (observe que a barra invertida após as aspas duplas de abertura diz ao Rust para não colocar um caractere de nova linha no início do conteúdo deste literal de string). Afirmamos que o valor retornado da função `search` contém apenas a linha que esperamos.

Se executarmos esse teste, ele falhará atualmente porque a macro `unimplemented!` entra em pânico com a mensagem "não implementado". De acordo com os princípios do TDD, daremos um pequeno passo de adicionar apenas código suficiente para fazer com que o teste não entre em pânico ao chamar a função, definindo a função `search` para sempre retornar um vetor vazio, conforme mostrado na Listagem 12-16. Então, o teste deve compilar e falhar porque um vetor vazio não corresponde a um vetor contendo a linha `"safe, fast, productive."`.

<Listing number="12-16" file-name="src/lib.rs" caption="Definindo apenas o suficiente da função `search` para que chamá-la não entre em pânico">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Agora vamos discutir por que precisamos definir um tempo de vida explícito `'a` na assinatura de `search` e usar esse tempo de vida com o argumento `contents` e o valor de retorno. Lembre-se no [[ch10-03-lifetime-syntax|Capítulo 10]] que os parâmetros de tempo de vida especificam qual tempo de vida do argumento está conectado ao tempo de vida do valor de retorno. Nesse caso, indicamos que o vetor retornado deve conter fatias de string que referenciam fatias do argumento `contents` (em vez do argumento `query`).

Em outras palavras, dizemos ao Rust que os dados retornados pela função `search` viverão tanto quanto os dados passados para a função `search` no argumento `contents`. Isto é importante! Os dados referenciados _por_ uma fatia precisam ser válidos para que a referência seja válida; se o compilador assumir que estamos fazendo fatias de string de `query` em vez de `contents`, ele fará sua verificação de segurança incorretamente.

Se esquecermos as anotações de tempo de vida e tentarmos compilar esta função, obteremos este erro:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust não pode saber qual dos dois parâmetros precisamos para a saída, então precisamos dizer explicitamente. Observe que o texto de ajuda sugere especificar o mesmo parâmetro de tempo de vida para todos os parâmetros e o tipo de saída, o que está incorreto! Como `contents` é o parââmetro que contém todo o nosso texto e queremos retornar as partes desse texto que correspondem, sabemos que `contents` é o único parâmetro que deve ser conectado ao valor de retorno usando a sintaxe de tempo de vida.

Outras linguagens de programação não exigem que você conecte argumentos para retornar valores na assinatura, mas essa prática ficará mais fácil com o tempo. Você pode querer comparar este exemplo com os exemplos na seção ["Validating References with Lifetimes"][[ch10-03-lifetime-syntax#validating-references-with-lifetimes|validating-references-with-lifetimes]] no Capítulo 10.

### Escrevendo Código para Passar no Teste

Atualmente, nosso teste está falhando porque sempre retornamos um vetor vazio. Para corrigir isso e implementar `search`, nosso programa precisa seguir estas etapas:

1. Itere por cada linha do conteúdo.
2. Verifique se a linha contém nossa string de consulta.
3. Se contiver, adicione-a à lista de valores que estamos retornando.
4. Se não contiver, não faça nada.
5. Retorne a lista de resultados que correspondem.

Vamos trabalhar em cada etapa, começando com a iteração por linhas.

#### Iterando Pelas Linhas com o Método `lines`

Rust tem um método útil para lidar com a iteração linha por linha de strings, convenientemente chamado `lines`, que funciona conforme mostrado na Listagem 12-17. Observe que isso não será compilado ainda.

<Listing number="12-17" file-name="src/lib.rs" caption="Iterando por cada linha em `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

O método `lines` retorna um iterador. Falaremos sobre iteradores em profundidade no [[ch13-02-iterators|Capítulo 13]]. Mas lembre-se de que você viu essa maneira de usar um iterador na [Listagem 3-5][[ch03-05-control-flow#looping-through-a-collection-with-for|ch3-iter]], onde usamos um loop `for` com um iterador para executar algum código em cada item de uma coleção.

#### Pesquisando Cada Linha pela Consulta

Em seguida, verificaremos se a linha atual contém nossa string de consulta. Felizmente, strings têm um método útil chamado `contains` que faz isso por nós! Adicione uma chamada ao método `contains` na função `search`, conforme mostrado na Listagem 12-18. Observe que isso ainda não será compilado.

<Listing number="12-18" file-name="src/lib.rs" caption="Adicionando funcionalidade para ver se a linha contém a string em `query`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

No momento, estamos construindo funcionalidade. Para compilar o código, precisamos retornar um valor do corpo, conforme indicamos que faríamos na assinatura da função.

#### Armazenando Linhas Correspondentes

Para finalizar esta função, precisamos de uma maneira de armazenar as linhas correspondentes que queremos retornar. Para isso, podemos criar um vetor mutável antes do loop `for` e chamar o método `push` para armazenar uma `line` no vetor. Após o loop `for`, retornamos o vetor, conforme mostrado na Listagem 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Armazenando as linhas que correspondem para que possamos retorná-las">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Agora, a função `search` deve retornar apenas as linhas que contêm `query`, e nosso teste deve passar. Vamos executar o teste:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Nosso teste passou, então sabemos que funciona!

Neste ponto, poderíamos considerar oportunidades para refatorar a implementação da função de pesquisa, mantendo os testes passando para manter a mesma funcionalidade. O código na função de pesquisa não é tão ruim, mas não aproveita alguns recursos úteis de iteradores. Voltaremos a este exemplo no [[ch13-02-iterators|Capítulo 13]], onde exploraremos iteradores em detalhes e veremos como melhorá-lo.

Agora o programa inteiro deve funcionar! Vamos testá-lo, primeiro com uma palavra que deve retornar exatamente uma linha do poema de Emily Dickinson: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Legal! Agora vamos tentar uma palavra que corresponderá a várias linhas, como _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

E, finalmente, vamos ter certeza de que não obtemos nenhuma linha quando procuramos por uma palavra que não está em lugar nenhum no poema, como _monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Excelente! Construímos nossa própria mini versão de uma ferramenta clássica e aprendemos muito sobre como estruturar aplicativos. Também aprendemos um pouco sobre entrada e saída de arquivo, tempos de vida, testes e análise de linha de comando.

Para completar este projeto, demonstraremos brevemente como trabalhar com variáveis de ambiente e como imprimir no erro padrão, ambos úteis quando você está escrevendo programas de linha de comando.
