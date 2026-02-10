## O Tipo Slice

*Slices* (fatias) permitem que você referencie uma sequência contígua de elementos em uma [coleção][ch8]<!-- ignore -->. Uma *slice* é um tipo de referência, então ela não tem *ownership*.

Aqui está um pequeno problema de programação: escreva uma função que receba uma string de palavras separadas por espaços e retorne a primeira palavra que encontrar nessa string. Se a função não encontrar um espaço na string, a string inteira deve ser uma palavra, então a string inteira deve ser retornada.

> Nota: Para os propósitos de introduzir *slices*, estamos assumindo ASCII apenas nesta seção; uma discussão mais completa sobre manipulação de UTF-8 está na seção [“Armazenando Texto Codificado em UTF-8 com Strings”][strings]<!-- ignore --> do Capítulo 8.

Vamos trabalhar em como escreveríamos a assinatura dessa função sem usar *slices*, para entender o problema que *slices* resolverão:

```rust,ignore
fn first_word(s: &String) -> ?
```

A função `first_word` tem um parâmetro do tipo `&String`. Não precisamos de *ownership*, então isso está bem. (Em Rust idiomático, funções não tomam *ownership* de seus argumentos a menos que precisem, e as razões para isso ficarão claras à medida que continuarmos.) Mas o que devemos retornar? Nós realmente não temos uma maneira de falar sobre *parte* de uma string. No entanto, poderíamos retornar o índice do final da palavra, indicado por um espaço. Vamos tentar isso, como mostrado na Listagem 4-7.

<Listing number="4-7" file-name="src/main.rs" caption="A função `first_word` que retorna um valor de índice de byte dentro do parâmetro `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Porque precisamos percorrer a `String` elemento por elemento e verificar se um valor é um espaço, converteremos nossa `String` para um array de bytes usando o método `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Em seguida, criamos um iterador sobre o array de bytes usando o método `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Discutiremos iteradores em mais detalhes no [Capítulo 13][ch13]<!-- ignore -->. Por enquanto, saiba que `iter` é um método que retorna cada elemento em uma coleção e que `enumerate` envolve o resultado de `iter` e retorna cada elemento como parte de uma tupla. O primeiro elemento da tupla retornada de `enumerate` é o índice, e o segundo elemento é uma referência ao elemento. Isso é um pouco mais conveniente do que calcular o índice nós mesmos.

Como o método `enumerate` retorna uma tupla, podemos usar padrões para desestruturar essa tupla. Estaremos discutindo padrões mais no [Capítulo 6][ch6]<!-- ignore -->. No loop `for`, especificamos um padrão que tem `i` para o índice na tupla e `&item` para o único byte na tupla. Como recebemos uma referência ao elemento de `.iter().enumerate()`, usamos `&` no padrão.

Dentro do loop `for`, procuramos pelo byte que representa o espaço usando a sintaxe de literal de byte. Se encontrarmos um espaço, retornamos a posição. Caso contrário, retornamos o comprimento da string usando `s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Agora temos uma maneira de descobrir o índice do final da primeira palavra na string, mas há um problema. Estamos retornando um `usize` por conta própria, mas ele é apenas um número significativo no contexto da `&String`. Em outras palavras, porque é um valor separado da `String`, não há garantia de que ele ainda será válido no futuro. Considere o programa na Listagem 4-8 que usa a função `first_word` da Listagem 4-7.

<Listing number="4-8" file-name="src/main.rs" caption="Armazenando o resultado da chamada da função `first_word` e depois mudando o conteúdo da `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Este programa compila sem erros e também o faria se usássemos `word` depois de chamar `s.clear()`. Porque `word` não está conectado ao estado de `s` de forma alguma, `word` ainda contém o valor `5`. Poderíamos usar esse valor `5` com a variável `s` para tentar extrair a primeira palavra, mas isso seria um bug porque o conteúdo de `s` mudou desde que salvamos `5` em `word`.

Ter que se preocupar com o índice em `word` ficando fora de sincronia com os dados em `s` é tedioso e propenso a erros! Gerenciar esses índices é ainda mais frágil se escrevermos uma função `second_word`. Sua assinatura teria que se parecer com isso:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Agora estamos rastreando um índice inicial *e* um final, e temos ainda mais valores que foram calculados a partir de dados em um estado particular, mas não estão atrelados a esse estado de forma alguma. Temos três variáveis não relacionadas flutuando que precisam ser mantidas em sincronia.

Felizmente, o Rust tem uma solução para esse problema: *string slices*.

### String Slices

Uma *string slice* é uma referência para uma sequência contígua de elementos de uma `String`, e se parece com isso:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Em vez de uma referência para a `String` inteira, `hello` é uma referência para uma porção da `String`, especificada na parte extra `[0..5]`. Criamos *slices* usando um intervalo dentro de colchetes especificando `[índice_inicial..índice_final]`, onde *`índice_inicial`* é a primeira posição na *slice* e *`índice_final`* é um a mais que a última posição na *slice*. Internamente, a estrutura de dados da *slice* armazena a posição inicial e o comprimento da *slice*, que corresponde a *`índice_final`* menos *`índice_inicial`*. Então, no caso de `let world = &s[6..11];`, `world` seria uma *slice* que contém um ponteiro para o byte no índice 6 de `s` com um valor de comprimento de `5`.

A Figura 4-7 mostra isso em um diagrama.

<img alt="Three tables: a table representing the stack data of s, which points
to the byte at index 0 in a table of the string data &quot;hello world&quot; on
the heap. The third table represents the stack data of the slice world, which
has a length value of 5 and points to byte 6 of the heap data table."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-7: Uma *string slice* referindo-se a parte de uma `String`</span>

Com a sintaxe de intervalo `..` do Rust, se você quiser começar no índice 0, você pode descartar o valor antes dos dois pontos. Em outras palavras, estes são iguais:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Pelo mesmo motivo, se sua *slice* inclui o último byte da `String`, você pode descartar o número final. Isso significa que estes são iguais:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Você também pode descartar ambos os valores para pegar uma *slice* da string inteira. Então, estes são iguais:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Nota: Índices de intervalo de *string slice* devem ocorrer em limites de caracteres UTF-8 válidos. Se você tentar criar uma *string slice* no meio de um caractere multibyte, seu programa sairá com um erro.

Com toda essa informação em mente, vamos reescrever `first_word` para retornar uma *slice*. O tipo que significa "string slice" é escrito como `&str`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Pegamos o índice para o final da palavra da mesma maneira que fizemos na Listagem 4-7, procurando pela primeira ocorrência de um espaço. Quando encontramos um espaço, retornamos uma *string slice* usando o início da string e o índice do espaço como os índices inicial e final.

Agora, quando chamamos `first_word`, recebemos de volta um único valor que está atrelado aos dados subjacentes. O valor é composto por uma referência ao ponto inicial da *slice* e o número de elementos na *slice*.

Retornar uma *slice* também funcionaria para uma função `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Agora temos uma API direta que é muito mais difícil de bagunçar porque o compilador garantirá que as referências para a `String` permaneçam válidas. Lembre-se do bug no programa na Listagem 4-8, quando pegamos o índice para o final da primeira palavra mas então limpamos a string para que nosso índice fosse inválido? Aquele código estava logicamente incorreto, mas não mostrou nenhum erro imediato. Os problemas apareceriam mais tarde se continuássemos tentando usar o índice da primeira palavra com uma string esvaziada. *Slices* tornam esse bug impossível e nos deixam saber muito mais cedo que temos um problema com nosso código. Usar a versão de *slice* de `first_word` lançará um erro em tempo de compilação:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

Aqui está o erro do compilador:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Lembre-se das regras de empréstimo que, se tivermos uma referência imutável para algo, não podemos também tomar uma referência mutável. Como `clear` precisa truncar a `String`, ele precisa pegar uma referência mutável. O `println!` após a chamada para `clear` usa a referência em `word`, então a referência imutável deve ainda estar ativa naquele ponto. O Rust proíbe a referência mutável em `clear` e a referência imutável em `word` de existirem ao mesmo tempo, e a compilação falha. Não apenas o Rust tornou nossa API mais fácil de usar, mas também eliminou uma classe inteira de erros em tempo de compilação!

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### Strings Literais São Slices

Lembre-se que falamos sobre strings literais sendo armazenadas dentro do binário. Agora que sabemos sobre *slices*, podemos entender adequadamente strings literais:

```rust
let s = "Olá, mundo!";
```

O tipo de `s` aqui é `&str`: é uma *slice* apontando para aquele ponto específico do binário. É também por isso que strings literais são imutáveis; `&str` é uma referência imutável.

#### String Slices como Parâmetros

Saber que você pode tomar *slices* de literais e valores `String` nos leva a mais uma melhoria em `first_word`, e essa é sua assinatura:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Um *Rustacean* mais experiente escreveria a assinatura mostrada na Listagem 4-9 em vez disso, porque ela nos permite usar a mesma função em ambos os valores `&String` e valores `&str`.

<Listing number="4-9" caption="Melhorando a função `first_word` usando uma *string slice* para o tipo do parâmetro `s`">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Se temos uma *string slice*, podemos passá-la diretamente. Se temos uma `String`, podemos passar uma *slice* da `String` ou uma referência para a `String`. Essa flexibilidade tira vantagem de *deref coercions* (coerções de desreferência), uma característica que cobriremos na seção [“Usando Deref Coercions em Funções e Métodos”][deref-coercions]<!-- ignore --> do Capítulo 15.

Definir uma função para receber uma *string slice* em vez de uma referência para uma `String` torna nossa API mais geral e útil sem perder nenhuma funcionalidade:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Outras Slices

*String slices*, como você pode imaginar, são específicas para strings. Mas há um tipo de *slice* mais geral também. Considere este array:

```rust
let a = [1, 2, 3, 4, 5];
```

Assim como podemos querer nos referir a parte de uma string, podemos querer nos referir a parte de um array. Faríamos assim:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Essa *slice* tem o tipo `&[i32]`. Ela funciona da mesma maneira que *string slices* fazem, armazenando uma referência para o primeiro elemento e um comprimento. Você usará esse tipo de *slice* para todos os tipos de outras coleções. Discutiremos essas coleções em detalhes quando falarmos sobre vetores no Capítulo 8.

## Resumo

Os conceitos de *ownership*, *borrowing* e *slices* garantem segurança de memória em programas Rust em tempo de compilação. A linguagem Rust dá a você controle sobre seu uso de memória da mesma maneira que outras linguagens de programação de sistemas. Mas ter o *owner* dos dados limpando automaticamente esses dados quando o *owner* sai de escopo significa que você não tem que escrever e depurar código extra para obter esse controle.

*Ownership* afeta como muitas outras partes do Rust funcionam, então falaremos sobre esses conceitos mais adiante ao longo do resto do livro. Vamos para o Capítulo 5 e olhar para o agrupamento de pedaços de dados juntos em uma `struct`.

[ch13]: [[ch13-02-iterators.md|Capítulo 13]]
[ch6]: [[ch06-02-match.md#patterns-that-bind-to-values|Capítulo 6]]
[ch8]: [[ch08-00-common-collections.md|Capítulo 8]]
[strings]: [[ch08-02-strings.md#storing-utf-8-encoded-text-with-strings|Armazenando Texto Codificado em UTF-8 com Strings]]
[deref-coercions]: [[ch15-02-deref.md#using-deref-coercions-in-functions-and-methods|Usando Deref Coercions em Funções e Métodos]]
