## Como Escrever Testes

_Testes_ são funções Rust que verificam se o código não-teste está funcionando
da maneira esperada. Os corpos das funções de teste geralmente realizam estas
três ações:

- Configurar quaisquer dados ou estados necessários.
- Executar o código que você deseja testar.
- Afirmar (assert) que os resultados são o que você espera.

Vamos ver os recursos que o Rust fornece especificamente para escrever testes
que realizam essas ações, que incluem o atributo `test`, algumas macros e o
atributo `should_panic`.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### Estruturando Funções de Teste

Na sua forma mais simples, um teste em Rust é uma função anotada com o atributo
`test`. Atributos são metadados sobre pedaços de código Rust; um exemplo é o
atributo `derive` que usamos com structs no Capítulo 5. Para transformar uma
função em uma função de teste, adicione `#[test]` na linha antes de `fn`.
Quando você executa seus testes com o comando `cargo test`, Rust constrói um
binário de executor de testes que executa as funções anotadas e relata se cada
função de teste passa ou falha.

Sempre que criamos um novo projeto de biblioteca com Cargo, um módulo de teste
com uma função de teste nele é gerado automaticamente para nós. Este módulo lhe
dá um modelo para escrever seus testes para que você não precise procurar a
estrutura e sintaxe exatas toda vez que iniciar um novo projeto. Você pode
adicionar quantas funções de teste adicionais e quantos módulos de teste
quiser!

Vamos explorar alguns aspectos de como os testes funcionam experimentando com o
teste modelo antes de realmente testarmos qualquer código. Em seguida,
escreveremos alguns testes do mundo real que chamam algum código que escrevemos
e afirmam que seu comportamento está correto.

Vamos criar um novo projeto de biblioteca chamado `adder` que adicionará dois
números:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

O conteúdo do arquivo _src/lib.rs_ em sua biblioteca `adder` deve se parecer
com a Listagem 11-1.

<Listing number="11-1" file-name="src/lib.rs" caption="O código gerado automaticamente por `cargo new`">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

O arquivo começa com um exemplo de função `add` para que tenhamos algo para
testar.

Por enquanto, vamos nos concentrar apenas na função `it_works`. Observe a
anotação `#[test]`: Este atributo indica que esta é uma função de teste, para
que o executor de testes saiba tratar esta função como um teste. Também podemos
ter funções não-teste no módulo `tests` para ajudar a configurar cenários
comuns ou realizar operações comuns, então sempre precisamos indicar quais
funções são testes.

O corpo da função de exemplo usa a macro `assert_eq!` para afirmar que `result`,
que contém o resultado da chamada de `add` com 2 e 2, é igual a 4. Esta
afirmação serve como um exemplo do formato para um teste típico. Vamos executá-lo
para ver que este teste passa.

O comando `cargo test` executa todos os testes em nosso projeto, conforme
mostrado na Listagem 11-2.

<Listing number="11-2" caption="A saída da execução do teste gerado automaticamente">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo compilou e executou o teste. Vemos a linha `running 1 test`. A próxima
linha mostra o nome da função de teste gerada, chamada `tests::it_works`, e que
o resultado da execução desse teste é `ok`. O resumo geral `test result: ok.`
significa que todos os testes passaram, e a parte que diz `1 passed; 0 failed`
totaliza o número de testes que passaram ou falharam.

É possível marcar um teste como ignorado para que ele não seja executado em uma
instância específica; abordaremos isso na seção [“Ignorando Testes A Menos Que
Solicitados Especificamente”][ignoring]<!-- ignore --> mais adiante neste
capítulo. Como não fizemos isso aqui, o resumo mostra `0 ignored`. Também
podemos passar um argumento para o comando `cargo test` para executar apenas
testes cujo nome corresponda a uma string; isso é chamado de _filtragem_, e
abordaremos isso na seção [“Executando um Subconjunto de Testes por
Nome”][subset]<!-- ignore -->. Aqui, não filtramos os testes sendo executados,
então o final do resumo mostra `0 filtered out`.

A estatística `0 measured` é para testes de benchmark que medem desempenho.
Testes de benchmark estão, no momento em que este livro foi escrito, disponíveis
apenas no Rust nightly. Veja [a documentação sobre testes de benchmark][bench]
para saber mais.

A próxima parte da saída de teste começando em `Doc-tests adder` é para os
resultados de quaisquer testes de documentação. Não temos nenhum teste de
documentação ainda, mas Rust pode compilar quaisquer exemplos de código que
apareçam em nossa documentação da API. Esse recurso ajuda a manter seus docs e
seu código em sincronia! Discutiremos como escrever testes de documentação na
seção [“Comentários de Documentação como Testes”][doc-comments]<!-- ignore -->
do Capítulo 14. Por enquanto, ignoraremos a saída `Doc-tests`.

Vamos começar a personalizar o teste para nossas próprias necessidades.
Primeiro, mude o nome da função `it_works` para um nome diferente, como
`exploration`, assim:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Então, execute `cargo test` novamente. A saída agora mostra `exploration` em vez
de `it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Agora vamos adicionar outro teste, mas desta vez faremos um teste que falha! Os
testes falham quando algo na função de teste entra em pânico. Cada teste é
executado em uma nova thread, e quando a thread principal vê que uma thread de
teste morreu, o teste é marcado como falha. No Capítulo 9, falamos sobre como a
maneira mais simples de entrar em pânico é chamar a macro `panic!`. Digite o
novo teste como uma função chamada `another`, para que seu arquivo _src/lib.rs_
se pareça com a Listagem 11-3.

<Listing number="11-3" file-name="src/lib.rs" caption="Adicionando um segundo teste que falhará porque chamamos a macro `panic!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

Execute os testes novamente usando `cargo test`. A saída deve se parecer com a
Listagem 11-4, que mostra que nosso teste `exploration` passou e `another`
falhou.

<Listing number="11-4" caption="Resultados do teste quando um teste passa e um teste falha">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

Em vez de `ok`, a linha `test tests::another` mostra `FAILED`. Duas novas
seções aparecem entre os resultados individuais e o resumo: A primeira exibe a
razão detalhada para cada falha de teste. Neste caso, obtemos os detalhes de
que `tests::another` falhou porque entrou em pânico com a mensagem `Make this
test fail` na linha 17 no arquivo _src/lib.rs_. A próxima seção lista apenas os
nomes de todos os testes com falha, o que é útil quando há muitos testes e
muita saída detalhada de teste com falha. Podemos usar o nome de um teste com
falha para executar apenas esse teste para depurá-lo mais facilmente; falaremos
mais sobre maneiras de executar testes na seção [“Controlando Como os Testes São
Executados”][controlling-how-tests-are-run]<!-- ignore -->.

A linha de resumo exibe no final: No geral, nosso resultado de teste é `FAILED`.
Tivemos um teste passado e um teste falho.

Agora que você viu como os resultados dos testes se parecem em diferentes
cenários, vamos ver algumas macros além de `panic!` que são úteis em testes.

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### Verificando Resultados com `assert!`

A macro `assert!`, fornecida pela biblioteca padrão, é útil quando você deseja
garantir que alguma condição em um teste seja avaliada como `true`. Damos à
macro `assert!` um argumento que é avaliado como um booleano. Se o valor for
`true`, nada acontece e o teste passa. Se o valor for `false`, a macro
`assert!` chama `panic!` para fazer o teste falhar. Usar a macro `assert!` nos
ajuda a verificar se nosso código está funcionando da maneira que pretendemos.

No Capítulo 5, Listagem 5-15, usamos uma struct `Rectangle` e um método
`can_hold`, que são repetidos aqui na Listagem 11-5. Vamos colocar este código
no arquivo _src/lib.rs_, e depois escrever alguns testes para ele usando a
macro `assert!`.

<Listing number="11-5" file-name="src/lib.rs" caption="A struct `Rectangle` e seu método `can_hold` do Capítulo 5">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

O método `can_hold` retorna um booleano, o que significa que é um caso de uso
perfeito para a macro `assert!`. Na Listagem 11-6, escrevemos um teste que
exercita o método `can_hold` criando uma instância `Rectangle` que tem uma
largura de 8 e uma altura de 7 e afirmando que ela pode conter outra instância
`Rectangle` que tem uma largura de 5 e uma altura de 1.

<Listing number="11-6" file-name="src/lib.rs" caption="Um teste para `can_hold` que verifica se um retângulo maior pode de fato conter um retângulo menor">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

Observe a linha `use super::*;` dentro do módulo `tests`. O módulo `tests` é um
módulo regular que segue as regras de visibilidade usuais que cobrimos no
Capítulo 7 na seção [“Caminhos para Referenciar um Item na Árvore de
Módulos”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->.
Como o módulo `tests` é um módulo interno, precisamos trazer o código sob teste
no módulo externo para o escopo do módulo interno. Usamos um glob aqui, para
que qualquer coisa que definamos no módulo externo esteja disponível para este
módulo `tests`.

Nomeamos nosso teste `larger_can_hold_smaller`, e criamos as duas instâncias de
`Rectangle` que precisamos. Em seguida, chamamos a macro `assert!` e passamos a
ela o resultado de chamar `larger.can_hold(&smaller)`. Esta expressão deve
retornar `true`, então nosso teste deve passar. Vamos descobrir!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Passou! Vamos adicionar outro teste, desta vez afirmando que um retângulo menor
não pode conter um retângulo maior:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Como o resultado correto da função `can_hold` neste caso é `false`, precisamos
negar esse resultado antes de passá-lo para a macro `assert!`. Como resultado,
nosso teste passará se `can_hold` retornar `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Dois testes que passam! Agora vamos ver o que acontece com nossos resultados de
teste quando introduzimos um bug em nosso código. Vamos alterar a implementação
do método `can_hold` substituindo o sinal de maior que (`>`) por um sinal de
menor que (`<`) quando ele compara as larguras:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Executar os testes agora produz o seguinte:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Nossos testes detectaram o bug! Como `larger.width` é `8` e `smaller.width` é
`5`, a comparação das larguras em `can_hold` agora retorna `false`: 8 não é
menor que 5.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### Testando Igualdade com `assert_eq!` e `assert_ne!`

Uma maneira comum de verificar a funcionalidade é testar a igualdade entre o
resultado do código sob teste e o valor que você espera que o código retorne.
Você poderia fazer isso usando a macro `assert!` e passando a ela uma expressão
usando o operador `==`. No entanto, este é um teste tão comum que a biblioteca
padrão fornece um par de macros—`assert_eq!` e `assert_ne!`—para realizar este
teste mais convenientemente. Essas macros comparam dois argumentos para
igualdade ou desigualdade, respectivamente. Elas também imprimirão os dois
valores se a afirmação falhar, o que torna mais fácil ver _por que_ o teste
falhou; inversamente, a macro `assert!` indica apenas que obteve um valor
`false` para a expressão `==`, sem imprimir os valores que levaram ao valor
`false`.

Na Listagem 11-7, escrevemos uma função chamada `add_two` que adiciona `2` ao
seu parâmetro e, em seguida, testamos essa função usando a macro `assert_eq!`.

<Listing number="11-7" file-name="src/lib.rs" caption="Testando a função `add_two` usando a macro `assert_eq!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Vamos verificar se passa!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Criamos uma variável chamada `result` que contém o resultado da chamada
`add_two(2)`. Em seguida, passamos `result` e `4` como os argumentos para a
macro `assert_eq!`. A linha de saída para este teste é `test tests::it_adds_two
... ok`, e o texto `ok` indica que nosso teste passou!

Vamos introduzir um bug em nosso código para ver como `assert_eq!` se parece
quando falha. Mude a implementação da função `add_two` para adicionar `3` em
vez disso:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Execute os testes novamente:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Nosso teste detectou o bug! O teste `tests::it_adds_two` falhou, e a mensagem
nos diz que a afirmação que falhou foi `left == right` e quais são os valores
`left` e `right`. Esta mensagem nos ajuda a começar a depurar: O argumento
`left`, onde tínhamos o resultado da chamada `add_two(2)`, era `5`, mas o
argumento `right` era `4`. Você pode imaginar que isso seria especialmente útil
quando temos muitos testes em andamento.

Observe que em algumas linguagens e estruturas de teste, os parâmetros para
funções de afirmação de igualdade são chamados de `expected` e `actual`, e a
ordem em que especificamos os argumentos importa. No entanto, em Rust, eles são
chamados de `left` e `right`, e a ordem em que especificamos o valor que
esperamos e o valor que o código produz não importa. Poderíamos escrever a
afirmação neste teste como `assert_eq!(4, result)`, o que resultaria na mesma
mensagem de falha que exibe `` assertion `left == right` failed ``.

A macro `assert_ne!` passará se os dois valores que dermos a ela não forem
iguais e falhará se forem iguais. Esta macro é mais útil para casos em que não
temos certeza de qual valor _será_, mas sabemos qual valor definitivamente
_não_ deve ser. Por exemplo, se estamos testando uma função que é garantida
para alterar sua entrada de alguma forma, mas a maneira pela qual a entrada é
alterada depende do dia da semana em que executamos nossos testes, a melhor
coisa a afirmar pode ser que a saída da função não é igual à entrada.

Sob a superfície, as macros `assert_eq!` e `assert_ne!` usam os operadores `==`
e `!=`, respectivamente. Quando as afirmações falham, essas macros imprimem
seus argumentos usando formatação de depuração, o que significa que os valores
sendo comparados devem implementar as traits `PartialEq` e `Debug`. Todos os
tipos primitivos e a maioria dos tipos da biblioteca padrão implementam essas
traits. Para structs e enums que você define, você precisará implementar
`PartialEq` para afirmar a igualdade desses tipos. Você também precisará
implementar `Debug` para imprimir os valores quando a afirmação falhar. Como
ambas as traits são traits deriváveis, como mencionado na Listagem 5-12 no
Capítulo 5, isso geralmente é tão simples quanto adicionar a anotação
`#[derive(PartialEq, Debug)]` à sua definição de struct ou enum. Veja o
Apêndice C, [“Traits Deriváveis,”][derivable-traits]<!-- ignore --> para mais
detalhes sobre essas e outras traits deriváveis.

### Adicionando Mensagens de Falha Personalizadas

Você também pode adicionar uma mensagem personalizada para ser impressa com a
mensagem de falha como argumentos opcionais para as macros `assert!`,
`assert_eq!` e `assert_ne!`. Quaisquer argumentos especificados após os
argumentos obrigatórios são passados para a macro `format!` (discutida em
[“Concatenando com `+` ou `format!`”][concatenating]<!-- ignore --> no
Capítulo 8), para que você possa passar uma string de formato que contenha `{}`
placeholders e valores para ir nesses placeholders. Mensagens personalizadas
são úteis para documentar o que uma afirmação significa; quando um teste falha,
você terá uma ideia melhor de qual é o problema com o código.

Por exemplo, digamos que temos uma função que cumprimenta as pessoas pelo nome
e queremos testar se o nome que passamos para a função aparece na saída:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Os requisitos para este programa ainda não foram acordados, e temos certeza de
que o texto `Hello` no início da saudação mudará. Decidimos que não queremos ter
que atualizar o teste quando os requisitos mudarem, então, em vez de verificar a
igualdade exata com o valor retornado da função `greeting`, apenas afirmaremos
que a saída contém o texto do parâmetro de entrada.

Agora vamos introduzir um bug neste código alterando `greeting` para excluir
`name` para ver como a falha de teste padrão se parece:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Executar este teste produz o seguinte:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Este resultado indica apenas que a afirmação falhou e em qual linha a afirmação
está. Uma mensagem de falha mais útil imprimiria o valor da função `greeting`.
Vamos adicionar uma mensagem de falha personalizada composta por uma string de
formato com um placeholder preenchido com o valor real que obtivemos da função
`greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Agora, quando executarmos o teste, obteremos uma mensagem de erro mais
informativa:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Podemos ver o valor que realmente obtivemos na saída do teste, o que nos
ajudaria a depurar o que aconteceu em vez do que esperávamos que acontecesse.

### Verificando Pânicos com `should_panic`

Além de verificar valores de retorno, é importante verificar se nosso código
lida com condições de erro conforme o esperado. Por exemplo, considere o tipo
`Guess` que criamos no Capítulo 9, Listagem 9-13. Outro código que usa `Guess`
depende da garantia de que as instâncias de `Guess` conterão apenas valores
entre 1 e 100. Podemos escrever um teste que garante que tentar criar uma
instância de `Guess` com um valor fora desse intervalo entra em pânico.

Fazemos isso adicionando o atributo `should_panic` à nossa função de teste. O
teste passa se o código dentro da função entrar em pânico; o teste falha se o
código dentro da função não entrar em pânico.

A Listagem 11-8 mostra um teste que verifica se as condições de erro de
`Guess::new` acontecem quando esperamos.

<Listing number="11-8" file-name="src/lib.rs" caption="Testando se uma condição causará um `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

Colocamos o atributo `#[should_panic]` após o atributo `#[test]` e antes da
função de teste à qual ele se aplica. Vamos ver o resultado quando este teste
passa:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Parece bom! Agora vamos introduzir um bug em nosso código removendo a condição
de que a função `new` entrará em pânico se o valor for maior que 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Quando executarmos o teste na Listagem 11-8, ele falhará:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Não recebemos uma mensagem muito útil neste caso, mas quando olhamos para a
função de teste, vemos que ela está anotada com `#[should_panic]`. A falha que
obtivemos significa que o código na função de teste não causou um pânico.

Testes que usam `should_panic` podem ser imprecisos. Um teste `should_panic`
passaria mesmo se o teste entrasse em pânico por um motivo diferente do que
estávamos esperando. Para tornar os testes `should_panic` mais precisos,
podemos adicionar um parâmetro opcional `expected` ao atributo `should_panic`. O
executor de testes garantirá que a mensagem de falha contenha o texto
fornecido. Por exemplo, considere o código modificado para `Guess` na Listagem
11-9, onde a função `new` entra em pânico com mensagens diferentes dependendo
se o valor é muito pequeno ou muito grande.

<Listing number="11-9" file-name="src/lib.rs" caption="Testando um `panic!` com uma mensagem de pânico contendo uma substring especificada">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Este teste passará porque o valor que colocamos no parâmetro `expected` do
atributo `should_panic` é uma substring da mensagem com a qual a função
`Guess::new` entra em pânico. Poderíamos ter especificado toda a mensagem de
pânico que esperamos, que neste caso seria `Guess value must be less than or
equal to 100, got 200`. O que você escolhe especificar depende de quanto da
mensagem de pânico é única ou dinâmica e quão preciso você deseja que seu teste
seja. Neste caso, uma substring da mensagem de pânico é suficiente para
garantir que o código na função de teste execute o caso `else if value > 100`.

Para ver o que acontece quando um teste `should_panic` com uma mensagem
`expected` falha, vamos introduzir novamente um bug em nosso código trocando os
corpos dos blocos `if value < 1` e `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Desta vez, quando executarmos o teste `should_panic`, ele falhará:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

A mensagem de falha indica que este teste realmente entrou em pânico como
esperávamos, mas a mensagem de pânico não incluiu a string esperada `less than
or equal to 100`. A mensagem de pânico que obtivemos neste caso foi `Guess
value must be greater than or equal to 1, got 200`. Agora podemos começar a
descobrir onde está nosso bug!

### Usando `Result<T, E>` em Testes

Todos os nossos testes até agora entram em pânico quando falham. Também podemos
escrever testes que usam `Result<T, E>`! Aqui está o teste da Listagem 11-1,
reescrito para usar `Result<T, E>` e retornar um `Err` em vez de entrar em
pânico:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

A função `it_works` agora tem o tipo de retorno `Result<(), String>`. No corpo
da função, em vez de chamar a macro `assert_eq!`, retornamos `Ok(())` quando o
teste passa e um `Err` com uma `String` dentro quando o teste falha.

Escrever testes para que eles retornem um `Result<T, E>` permite que você use o
operador de ponto de interrogação no corpo dos testes, o que pode ser uma
maneira conveniente de escrever testes que devem falhar se qualquer operação
dentro deles retornar uma variante `Err`.

Você não pode usar a anotação `#[should_panic]` em testes que usam `Result<T,
E>`. Para afirmar que uma operação retorna uma variante `Err`, _não_ use o
operador de ponto de interrogação no valor `Result<T, E>`. Em vez disso, use
`assert!(value.is_err())`.

Agora que você conhece várias maneiras de escrever testes, vamos ver o que está
acontecendo quando executamos nossos testes e explorar as diferentes opções que
podemos usar com `cargo test`.

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
