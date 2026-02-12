## Armazenando Texto Codificado em UTF-8 com Strings

Falamos sobre strings no Capítulo 4, mas vamos analisá-las mais a fundo agora.
Novos Rustaceans geralmente ficam presos em strings por uma combinação de três
razões: a propensão do Rust para expor possíveis erros, strings sendo uma
estrutura de dados mais complicada do que muitos programadores lhes dão crédito,
e UTF-8. Esses fatores se combinam de uma maneira que pode parecer difícil
quando você está vindo de outras linguagens de programação.

Discutimos strings no contexto de coleções porque strings são implementadas
como uma coleção de bytes, além de alguns métodos para fornecer funcionalidade
útil quando esses bytes são interpretados como texto. Nesta seção, falaremos
sobre as operações em `String` que todo tipo de coleção tem, como criar,
atualizar e ler. Também discutiremos as maneiras pelas quais `String` é
diferente das outras coleções, a saber, como indexar em uma `String` é
complicado pelas diferenças entre como as pessoas e os computadores interpretam
dados `String`.

### O Que É uma String?

Primeiro, vamos definir o que queremos dizer com o termo *string*. O Rust tem
apenas um tipo de string na linguagem central, que é a fatia de string `str` que
geralmente é vista em sua forma emprestada `&str`. No Capítulo 4, falamos sobre
*fatias de string*, que são referências a alguns dados de string codificados em
UTF-8 armazenados em outro lugar. Literais de string, por exemplo, são
armazenados no binário do programa e são, portanto, fatias de string.

O tipo `String`, que é fornecido pela biblioteca padrão do Rust em vez de
embutido na linguagem central, é um tipo de string mutável, possuído, expansível
e codificado em UTF-8. Quando Rustaceans se referem a "strings" em Rust, eles
podem estar se referindo a qualquer um dos tipos `String` ou fatias de string
`&str`, não apenas a um desses tipos. Embora esta seção seja em grande parte
sobre `String`, ambos os tipos são usados pesadamente na biblioteca padrão do
Rust, e ambos `String` e fatias de string são codificados em UTF-8.

### Criando uma Nova String

Muitas das mesmas operações disponíveis com `Vec<T>` também estão disponíveis
com `String`, porque `String` é na verdade implementada como um invólucro em
torno de um vetor de bytes com algumas garantias, restrições e capacidades
extras. Um exemplo de uma função que funciona da mesma maneira com `Vec<T>` e
`String` é a função `new` para criar uma instância, mostrada na Listagem 8-11.

<Listing number="8-11" caption="Criando uma nova e vazia `String`">

```rust
let mut s = String::new();
```

</Listing>

Esta linha cria uma nova string vazia chamada `s`, na qual podemos carregar
dados. Muitas vezes, teremos alguns dados iniciais com os quais queremos
começar a string. Para isso, usamos o método `to_string`, que está disponível
em qualquer tipo que implemente a trait `Display`, como literais de string
fazem. A Listagem 8-12 mostra dois exemplos.

<Listing number="8-12" caption="Usando o método `to_string` para criar uma `String` a partir de um literal de string">

```rust
let data = "initial contents";

let s = data.to_string();

// o método também funciona em um literal diretamente:
let s = "initial contents".to_string();
```

</Listing>

Este código cria uma string contendo `initial contents`.

Também podemos usar a função `String::from` para criar uma `String` a partir de
um literal de string. O código na Listagem 8-13 é equivalente ao código da
Listagem 8-12 que usa `to_string`.

<Listing number="8-13" caption="Usando a função `String::from` para criar uma `String` a partir de um literal de string">

```rust
let s = String::from("initial contents");
```

</Listing>

Como strings são usadas para tantas coisas, podemos usar muitas APIs genéricas
diferentes para strings, fornecendo-nos muitas opções. Algumas delas podem
parecer redundantes, mas todas têm seu lugar! Neste caso, `String::from` e
`to_string` fazem a mesma coisa, então qual você escolhe é uma questão de
estilo e legibilidade.

Lembre-se de que strings são codificadas em UTF-8, então podemos incluir
quaisquer dados devidamente codificados nelas, como mostrado na Listagem 8-14.

<Listing number="8-14" caption="Armazenando saudações em diferentes idiomas em strings">

```rust
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
let hello = String::from("שָׁלוֹם");
let hello = String::from("नमस्ते");
let hello = String::from("こんにちは");
let hello = String::from("안녕하세요");
let hello = String::from("你好");
let hello = String::from("Olá");
let hello = String::from("Здравствуйте");
let hello = String::from("Hola");
```

</Listing>

Todos esses são valores `String` válidos.

### Atualizando uma String

Uma `String` pode crescer em tamanho e seu conteúdo pode mudar, assim como o
conteúdo de um `Vec<T>`, se você empurrar mais dados para ela. Além disso,
você pode usar convenientemente o operador `+` ou o macro `format!` para
concatenar valores `String`.

#### Anexando a uma String com `push_str` e `push`

Podemos crescer uma `String` usando o método `push_str` para anexar uma fatia
de string, como mostrado na Listagem 8-15.

<Listing number="8-15" caption="Anexando uma fatia de string a uma `String` usando o método `push_str`">

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

</Listing>

Após essas duas linhas, `s` conterá `foobar`. O método `push_str` recebe uma
fatia de string porque não queremos necessariamente tomar posse do parâmetro.
Por exemplo, na Listagem 8-16, queremos poder usar `s2` depois de anexar seu
conteúdo a `s1`.

<Listing number="8-16" caption="Usando uma fatia de string depois de anexar seu conteúdo a uma `String`">

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 é {s2}");
```

</Listing>

Se o método `push_str` tomasse posse de `s2`, não seríamos capazes de imprimir
seu valor na última linha. No entanto, este código funciona como esperamos!

O método `push` recebe um único caractere como parâmetro e o adiciona à
`String`. A Listagem 8-17 adiciona a letra "l" a uma `String` usando o método
`push`.

<Listing number="8-17" caption="Adicionando um caractere a um valor `String` usando `push`">

```rust
let mut s = String::from("lo");
s.push('l');
```

</Listing>

Como resultado, `s` conterá `lol`.

#### Concatenação com o Operador `+` ou o Macro `format!`

Muitas vezes, você vai querer combinar duas strings existentes. Uma maneira de
fazer isso é usar o operador `+`, como mostrado na Listagem 8-18.

<Listing number="8-18" caption="Usando o operador `+` para combinar dois valores `String` em um novo valor `String`">

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note que s1 foi movido aqui e não pode mais ser usado
```

</Listing>

A string `s3` conterá `Hello, world!`. A razão pela qual `s1` não é mais válida
após a adição, e a razão pela qual usamos uma referência a `s2`, tem a ver com
a assinatura do método que é chamado quando usamos o operador `+`. O operador
`+` usa o método `add`, cuja assinatura se parece com isso:

```rust,ignore
fn add(self, s: &str) -> String {
```

Na biblioteca padrão, você verá `add` definido usando genéricos e tipos
associados. Aqui, substituímos por tipos concretos, que é o que acontece quando
chamamos este método com valores `String`. Discutiremos genéricos no Capítulo
10. Esta assinatura nos dá as pistas que precisamos para entender as partes
complicadas do operador `+`.

Primeiro, `s2` tem um `&`, significando que estamos adicionando uma
*referência* da segunda string à primeira string. Isso é por causa do parâmetro
`s` na função `add`: podemos adicionar apenas uma `&str` a uma `String`; não
podemos adicionar duas valores `String` juntos. Mas espere—o tipo de `&s2` é
`&String`, não `&str`, como especificado no segundo parâmetro para `add`. Então
por que a Listagem 8-18 compila?

A razão pela qual podemos usar `&s2` na chamada para `add` é que o compilador
pode *coagir* o argumento `&String` em um `&str`. Quando chamamos o método
`add`, o Rust usa uma *coerção de desreferenciamento* (deref coercion), que
aqui transforma `&s2` em `&s2[..]`. Discutiremos a coerção de desreferenciamento
em mais profundidade no Capítulo 15. Como `add` não toma posse do parâmetro
`s`, `s2` ainda será uma `String` válida após esta operação.

Segundo, podemos ver na assinatura que `add` toma posse de `self`, porque `self`
*não* tem um `&`. Isso significa que `s1` na Listagem 8-18 será movido para a
chamada `add` e não será mais válido depois disso. Então, embora `let s3 = s1 +
&s2;` pareça que copiará ambas as strings e criará uma nova, essa instrução na
verdade toma posse de `s1`, anexa uma cópia do conteúdo de `s2` e então retorna
a posse do resultado. Em outras palavras, parece que está fazendo muitas cópias,
mas não está; a implementação é mais eficiente do que copiar.

Se precisarmos concatenar múltiplas strings, o comportamento do operador `+`
fica desajeitado:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

Neste ponto, `s` será `tic-tac-toe`. Com todos os caracteres `+` e `"`, é
difícil ver o que está acontecendo. Para combinações de strings mais
complicadas, podemos usar o macro `format!`:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}");
```

Este código também define `s` para `tic-tac-toe`. O macro `format!` funciona
como `println!`, mas em vez de imprimir a saída na tela, ele retorna uma
`String` com o conteúdo. A versão do código usando `format!` é muito mais fácil
de ler, e o código gerado pelo macro `format!` usa referências para que esta
chamada não tome posse de nenhum de seus parâmetros.

### Indexando em Strings

Em muitas outras linguagens de programação, acessar caracteres individuais em
uma string referenciando-os por índice é uma operação válida e comum. No
entanto, se você tentar acessar partes de uma `String` usando sintaxe de
indexação em Rust, obterá um erro. Considere o código inválido na Listagem 8-19.

<Listing number="8-19" caption="Tentando usar sintaxe de indexação com uma String">

```rust,ignore,does_not_compile
let s1 = String::from("hello");
let h = s1[0];
```

</Listing>

Este código resultará no seguinte erro:

```text
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `String` cannot be indexed by `{integer}`
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`
  = help: the following other types implement trait `Index<Idx>`:
            <String as Index<RangeFrom<usize>>>
            <String as Index<RangeFull>>
            <String as Index<RangeInclusive<usize>>>
            <String as Index<RangeTo<usize>>>
            <String as Index<RangeToInclusive<usize>>>
            <String as Index<std::ops::Range<usize>>>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` due to 1 previous error
```

O erro e a nota contam a história: strings Rust não suportam indexação. Mas por
que não? Para responder a essa pergunta, temos que discutir como o Rust armazena
strings na memória.

#### Representação Interna

Uma `String` é um invólucro sobre um `Vec<u8>`. Vamos olhar para algumas de
nossas strings de exemplo em UTF-8 devidamente codificadas da Listagem 8-14.
Primeiro, esta:

```rust
let hello = String::from("Hola");
```

Neste caso, `len` será 4, o que significa que o vetor armazenando a string
"Hola" tem 4 bytes de comprimento. Cada uma dessas letras leva 1 byte quando
codificada em UTF-8. A linha a seguir, no entanto, pode surpreendê-lo. (Note que
esta string começa com a letra cirílica Ze maiúscula, não o número árabe 3.)

```rust
let hello = String::from("Здравствуйте");
```

Perguntado qual é o comprimento da string, você pode dizer 12. Na verdade, a
resposta do Rust é 24: esse é o número de bytes que leva para codificar
"Здравствуйте" em UTF-8, porque cada valor escalar Unicode nessa string leva 2
bytes de armazenamento. Portanto, um índice nos bytes da string nem sempre se
correlacionará a um valor escalar Unicode válido. Para demonstrar, considere
este código inválido em Rust:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

Você já sabe que `answer` não será `З`, a primeira letra. Quando codificado em
UTF-8, o primeiro byte de `З` é `208` e o segundo é `151`, então pareceria que
`answer` deveria ser na verdade `208`, mas `208` não é um caractere válido por
si só. Retornar `208` provavelmente não é o que um usuário desejaria se pedisse
pela primeira letra desta string; no entanto, esses são os únicos dados que o
Rust tem no índice de byte 0. Os usuários geralmente não querem o valor do byte
retornado, mesmo que a string contenha apenas letras latinas: se `&"hello"[0]`
fosse um código válido que retornasse o valor do byte, ele retornaria `104`,
não `h`.

A resposta, então, é que, para evitar retornar um valor inesperado e causar
bugs que podem não ser descobertos imediatamente, o Rust não compila este código
de forma alguma e evita mal-entendidos no início do processo de desenvolvimento.

#### Bytes e Valores Escalares e Clusters de Grafemas! Meu Deus!

Outro ponto sobre UTF-8 é que na verdade existem três maneiras relevantes de
olhar para strings da perspectiva do Rust: como bytes, valores escalares e
clusters de grafemas (a coisa mais próxima do que chamaríamos de *letras*).

Se olharmos para a palavra hindi "नमस्ते" escrita na escrita Devanagari, ela é
armazenada como um vetor de valores `u8` que se parece com isso:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

São 18 bytes e é como os computadores armazenam esses dados. Se olharmos para
eles como valores escalares Unicode, que é o que o tipo `char` do Rust é, esses
bytes se parecem com isso:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Existem seis valores `char` aqui, mas o quarto e o sexto não são letras: são
diacríticos que não fazem sentido por si mesmos. Finalmente, se olharmos para
eles como clusters de grafemas, obteríamos o que uma pessoa chamaria de as
quatro letras que compõem a palavra hindi:

```text
["न", "म", "स्", "ते"]
```

O Rust fornece diferentes maneiras de interpretar os dados brutos da string que
os computadores armazenam para que cada programa possa escolher a interpretação
que precisa, não importa em que linguagem humana os dados estejam.

Uma razão final pela qual o Rust não nos permite indexar em uma `String` para
obter um caractere é que as operações de indexação devem sempre levar tempo
constante (O(1)). Mas não é possível garantir esse desempenho com uma `String`,
porque o Rust teria que percorrer o conteúdo desde o início até o índice para
determinar quantos caracteres válidos existem.

### Fatiando Strings

Indexar em uma string é muitas vezes uma má ideia porque não está claro qual
deve ser o tipo de retorno da operação de indexação de string: um valor de byte,
um caractere, um cluster de grafemas ou uma fatia de string. Se você realmente
precisar usar índices para criar fatias de string, portanto, o Rust pede que
você seja mais específico.

Em vez de indexar usando `[]` com um único número, você pode usar `[]` com um
intervalo para criar uma fatia de string contendo bytes específicos:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aqui, `s` será um `&str` que contém os primeiros 4 bytes da string. Antes,
mencionamos que cada um desses caracteres era de 2 bytes, o que significa que
`s` será `Зд`.

Se tentássemos fatiar apenas parte dos bytes de um caractere com algo como
`&hello[0..1]`, o Rust entraria em pânico em tempo de execução da mesma maneira
que se um índice inválido fosse acessado em um vetor:

```text
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/collections`
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/main.rs:4:14
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Você deve ter cuidado ao criar fatias de string com intervalos, porque fazer
isso pode travar seu programa.

### Métodos para Iterar Sobre Strings

A melhor maneira de operar em pedaços de strings é ser explícito sobre se você
quer caracteres ou bytes. Para valores escalares Unicode individuais, use o
método `chars`. Chamar `chars` em "Зд" separa e retorna dois valores do tipo
`char`, e você pode iterar sobre o resultado para acessar cada elemento:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Este código imprimirá o seguinte:

```text
З
д
```

Alternativamente, o método `bytes` retorna cada byte bruto, que pode ser
apropriado para o seu domínio:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Este código imprimirá os quatro bytes que compõem esta string:

```text
208
151
208
180
```

Mas certifique-se de lembrar que valores escalares Unicode válidos podem ser
compostos por mais de 1 byte.

Obter clusters de grafemas de strings como na escrita Devanagari é complexo,
então essa funcionalidade não é fornecida pela biblioteca padrão. Crates estão
disponíveis em [crates.io](https://crates.io/) se esta for a funcionalidade que
você precisa.

### Strings Não São Tão Simples

Para resumir, strings são complicadas. Diferentes linguagens de programação
fazem escolhas diferentes sobre como apresentar essa complexidade ao
programador. O Rust escolheu tornar o tratamento correto de dados `String` o
comportamento padrão para todos os programas Rust, o que significa que os
programadores têm que pensar mais sobre o tratamento de UTF-8 antecipadamente.
Essa compensação expõe mais da complexidade das strings do que é aparente em
outras linguagens de programação, mas impede que você tenha que lidar com erros
envolvendo caracteres não-ASCII mais tarde em seu ciclo de vida de
desenvolvimento.

A boa notícia é que a biblioteca padrão oferece muita funcionalidade construída
a partir dos tipos `String` e `&str` para ajudar a lidar com essas situações
complexas corretamente. Certifique-se de verificar a documentação para métodos
úteis como `contains` para pesquisar em uma string e `replace` para substituir
partes de uma string por outra string.

Vamos mudar para algo um pouco menos complexo: hash maps!
