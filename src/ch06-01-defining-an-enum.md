# Definindo um Enum

Enquanto structs dão a você uma maneira de agrupar campos e dados relacionados, como
um `Rectangle` com sua `width` e `height`, enums dão a você uma maneira de dizer que um
valor é um de um conjunto possível de valores. Por exemplo, podemos querer dizer que
`Rectangle` é um de um conjunto de formas possíveis que também inclui `Circle` e
`Triangle`. Para fazer isso, Rust nos permite codificar essas possibilidades como um enum.

Vamos ver uma situação que podemos querer expressar em código e ver por que enums
são úteis e mais apropriados do que structs neste caso. Digamos que precisamos trabalhar
com endereços IP. Atualmente, dois padrões principais são usados para endereços IP:
versão quatro e versão seis. Como essas são as únicas possibilidades para um
endereço IP que nosso programa encontrará, podemos *enumerar* todas as possíveis
variantes, que é de onde a enumeração recebe seu nome.

Qualquer endereço IP pode ser um endereço versão quatro ou versão seis, mas não
ambos ao mesmo tempo. Essa propriedade dos endereços IP torna a estrutura de dados enum
apropriada porque um valor enum só pode ser uma de suas variantes.
Tanto endereços versão quatro quanto versão seis ainda são fundamentalmente endereços IP,
então eles devem ser tratados como o mesmo tipo quando o código está lidando com
situações que se aplicam a qualquer tipo de endereço IP.

Podemos expressar esse conceito em código definindo uma enumeração `IpAddrKind` e
listando os possíveis tipos que um endereço IP pode ser, `V4` e `V6`. Estas são as
variantes do enum:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` é agora um tipo de dado personalizado que podemos usar em outro lugar em nosso código.

### Valores de Enum

Podemos criar instâncias de cada uma das duas variantes de `IpAddrKind` assim:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Note que as variantes do enum estão sob o namespace de seu identificador, e nós
usamos dois pontos duplos para separar os dois. Isso é útil porque agora ambos os valores
`IpAddrKind::V4` e `IpAddrKind::V6` são do mesmo tipo: `IpAddrKind`. Nós
podemos então, por exemplo, definir uma função que recebe qualquer `IpAddrKind`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

E podemos chamar essa função com qualquer variante:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Usar enums tem ainda mais vantagens. Pensando mais sobre nosso tipo de endereço IP,
no momento não temos uma maneira de armazenar os *dados* reais do endereço IP; nós
apenas sabemos que *tipo* ele é. Dado que você acabou de aprender sobre structs no
Capítulo 5, você pode ficar tentado a resolver esse problema com structs como mostrado na
Listagem 6-1.

<Listing number="6-1" caption="Armazenando os dados e a variante `IpAddrKind` de um endereço IP usando uma `struct`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

Aqui, definimos uma struct `IpAddr` que tem dois campos: um campo `kind` que
é do tipo `IpAddrKind` (o enum que definimos anteriormente) e um campo `address`
do tipo `String`. Temos duas instâncias dessa struct. A primeira é `home`,
e tem o valor `IpAddrKind::V4` como seu `kind` com dados de endereço associados
de `127.0.0.1`. A segunda instância é `loopback`. Ela tem a outra
variante de `IpAddrKind` como seu valor `kind`, `V6`, e tem o endereço `::1`
associado a ela. Usamos uma struct para agrupar os valores `kind` e `address`,
então agora a variante está associada ao valor.

No entanto, representar o mesmo conceito usando apenas um enum é mais conciso:
Em vez de um enum dentro de uma struct, podemos colocar dados diretamente em cada variante do enum.
Essa nova definição do enum `IpAddr` diz que ambas as variantes `V4` e `V6`
terão valores `String` associados:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Anexamos dados a cada variante do enum diretamente, então não há necessidade de uma
struct extra. Aqui, também é mais fácil ver outro detalhe de como enums funcionam:
O nome de cada variante de enum que definimos também se torna uma função que
constrói uma instância do enum. Ou seja, `IpAddr::V4()` é uma chamada de função
que recebe um argumento `String` e retorna uma instância do tipo `IpAddr`. Nós
obtemos automaticamente essa função construtora definida como resultado da definição do
enum.

Há outra vantagem em usar um enum em vez de uma struct: Cada variante
pode ter diferentes tipos e quantidades de dados associados. Endereços IP versão quatro
sempre terão quatro componentes numéricos que terão valores
entre 0 e 255. Se quiséssemos armazenar endereços `V4` como quatro valores `u8` mas
ainda expressar endereços `V6` como um valor `String`, não seríamos capazes com
uma struct. Enums lidam com esse caso com facilidade:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Mostramos várias maneiras diferentes de definir estruturas de dados para armazenar endereços IP
versão quatro e versão seis. No entanto, como acontece, querer armazenar
endereços IP e codificar qual tipo eles são é tão comum que [a biblioteca padrão
tem uma definição que podemos usar!][IpAddr]<!-- ignore --> Vamos ver como
a biblioteca padrão define `IpAddr`. Ela tem o enum exato e variantes que
definimos e usamos, mas incorpora os dados de endereço dentro das variantes na
forma de duas structs diferentes, que são definidas de forma diferente para cada
variante:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Este código ilustra que você pode colocar qualquer tipo de dados dentro de uma variante de enum:
strings, tipos numéricos ou structs, por exemplo. Você pode até incluir outro
enum! Além disso, os tipos da biblioteca padrão geralmente não são muito mais complicados do que
o que você pode criar.

Note que, embora a biblioteca padrão contenha uma definição para `IpAddr`,
ainda podemos criar e usar nossa própria definição sem conflito porque nós
não trouxemos a definição da biblioteca padrão para o nosso escopo. Falaremos
mais sobre trazer tipos para o escopo no Capítulo 7.

Vamos ver outro exemplo de um enum na Listagem 6-2: Este tem uma ampla
variedade de tipos incorporados em suas variantes.

<Listing number="6-2" caption="Um enum `Message` cujas variantes armazenam diferentes quantidades e tipos de valores">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Este enum tem quatro variantes com tipos diferentes:

- `Quit`: Não tem dados associados a ele
- `Move`: Tem campos nomeados, como uma struct
- `Write`: Inclui uma única `String`
- `ChangeColor`: Inclui três valores `i32`

Definir um enum com variantes como as da Listagem 6-2 é semelhante a
definir diferentes tipos de definições de struct, exceto que o enum não usa a
palavra-chave `struct` e todas as variantes são agrupadas sob o tipo `Message`.
As seguintes structs poderiam conter os mesmos dados que as variantes de enum anteriores
contêm:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Mas se usássemos as diferentes structs, cada uma das quais tem seu próprio tipo, nós
não poderíamos definir tão facilmente uma função para receber qualquer um desses tipos de mensagens como
poderíamos com o enum `Message` definido na Listagem 6-2, que é um único tipo.

Há mais uma semelhança entre enums e structs: Assim como somos capazes de
definir métodos em structs usando `impl`, também somos capazes de definir métodos em
enums. Aqui está um método chamado `call` que poderíamos definir em nosso enum `Message`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

O corpo do método usaria `self` para obter o valor no qual chamamos o
método. Neste exemplo, criamos uma variável `m` que tem o valor
`Message::Write(String::from("hello"))`, e é isso que `self` será no
corpo do método `call` quando `m.call()` for executado.

Vamos ver outro enum na biblioteca padrão que é muito comum e
útil: `Option`.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-option-enum-and-its-advantages-over-null-values"></a>

### O Enum `Option`

Esta seção explora um estudo de caso de `Option`, que é outro enum definido
pela biblioteca padrão. O tipo `Option` codifica o cenário muito comum em
que um valor pode ser algo ou pode ser nada.

Por exemplo, se você solicitar o primeiro item em uma lista não vazia, você obterá
um valor. Se você solicitar o primeiro item em uma lista vazia, você não obterá nada.
Expressar esse conceito em termos do sistema de tipos significa que o compilador pode
verificar se você lidou com todos os casos que deveria estar lidando; esta
funcionalidade pode prevenir bugs que são extremamente comuns em outras linguagens de
programação.

O design de linguagem de programação é frequentemente pensado em termos de quais recursos você
inclui, mas os recursos que você exclui também são importantes. Rust não tem o
recurso null que muitas outras linguagens têm. *Null* é um valor que significa que não há
valor lá. Em linguagens com null, as variáveis sempre podem estar em um de
dois estados: null ou não-null.

Em sua apresentação de 2009 “Null References: The Billion Dollar Mistake”, Tony
Hoare, o inventor do null, disse o seguinte:

> Eu chamo isso de meu erro de um bilhão de dólares. Naquela época, eu estava projetando o primeiro
> sistema de tipos abrangente para referências em uma linguagem orientada a objetos. Meu
> objetivo era garantir que todo uso de referências fosse absolutamente seguro, com
> verificação realizada automaticamente pelo compilador. Mas não resisti à
> tentação de colocar uma referência nula, simplesmente porque era tão fácil de
> implementar. Isso levou a inúmeros erros, vulnerabilidades e falhas de sistema,
> que provavelmente causaram um bilhão de dólares em dor e danos nos
> últimos quarenta anos.

O problema com valores nulos é que, se você tentar usar um valor nulo como um
valor não nulo, obterá um erro de algum tipo. Como essa propriedade nula ou não nula
é onipresente, é extremamente fácil cometer esse tipo de erro.

No entanto, o conceito que o null está tentando expressar ainda é útil: Um
nulo é um valor que é atualmente inválido ou ausente por algum motivo.

O problema não é realmente com o conceito, mas com a implementação específica.
Como tal, Rust não tem nulos, mas tem um enum
que pode codificar o conceito de um valor estar presente ou ausente. Este enum é
`Option<T>`, e é [definido pela biblioteca padrão][option]<!-- ignore -->
da seguinte forma:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

O enum `Option<T>` é tão útil que está até incluído no prelúdio; você
não precisa trazê-lo para o escopo explicitamente. Suas variantes também estão incluídas no
prelúdio: Você pode usar `Some` e `None` diretamente sem o prefixo `Option::`.
O enum `Option<T>` ainda é apenas um enum regular, e `Some(T)` e
`None` ainda são variantes do tipo `Option<T>`.

A sintaxe `<T>` é um recurso do Rust sobre o qual ainda não falamos. É um
parâmetro de tipo genérico, e cobriremos genéricos em mais detalhes no Capítulo 10.
Por enquanto, tudo o que você precisa saber é que `<T>` significa que a variante `Some` do
enum `Option` pode conter um dado de qualquer tipo, e que cada
tipo concreto que é usado no lugar de `T` torna o tipo geral `Option<T>`
um tipo diferente. Aqui estão alguns exemplos de uso de valores `Option` para conter
tipos numéricos e tipos de caracteres:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

O tipo de `some_number` é `Option<i32>`. O tipo de `some_char` é
`Option<char>`, que é um tipo diferente. Rust pode inferir esses tipos porque
especificamos um valor dentro da variante `Some`. Para `absent_number`, Rust
exige que anotemos o tipo geral `Option`: O compilador não pode inferir o
tipo que a variante `Some` correspondente conterá olhando apenas para um
valor `None`. Aqui, dizemos ao Rust que queremos que `absent_number` seja do tipo
`Option<i32>`.

Quando temos um valor `Some`, sabemos que um valor está presente e o valor é
mantido dentro do `Some`. Quando temos um valor `None`, em certo sentido, significa a
mesma coisa que null: Não temos um valor válido. Então, por que ter `Option<T>` é
melhor do que ter null?

Em resumo, porque `Option<T>` e `T` (onde `T` pode ser qualquer tipo) são tipos diferentes,
o compilador não nos deixará usar um valor `Option<T>` como se fosse
definitivamente um valor válido. Por exemplo, este código não será compilado porque está
tentando adicionar um `i8` a um `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Se executarmos este código, receberemos uma mensagem de erro como esta:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Intenso! Com efeito, esta mensagem de erro significa que o Rust não entende como
adicionar um `i8` e um `Option<i8>`, porque eles são tipos diferentes. Quando
temos um valor de um tipo como `i8` em Rust, o compilador garantirá que
sempre tenhamos um valor válido. Podemos prosseguir com confiança sem ter que verificar
por nulo antes de usar esse valor. Somente quando temos um `Option<i8>` (ou
qualquer tipo de valor com o qual estejamos trabalhando) é que temos que nos preocupar em possivelmente
não ter um valor, e o compilador garantirá que lidemos com esse caso antes de
usar o valor.

Em outras palavras, você tem que converter um `Option<T>` em um `T` antes de poder
realizar operações `T` com ele. Geralmente, isso ajuda a detectar um dos problemas mais
comuns com null: assumir que algo não é nulo quando na verdade é.

Eliminar o risco de assumir incorretamente um valor não nulo ajuda você a ter mais
confiança em seu código. Para ter um valor que possivelmente pode ser nulo, você
deve optar explicitamente tornando o tipo desse valor `Option<T>`. Então, quando
você usa esse valor, é obrigado a lidar explicitamente com o caso em que o
valor é nulo. Em todos os lugares em que um valor tem um tipo que não é um `Option<T>`,
você *pode* assumir com segurança que o valor não é nulo. Esta foi uma decisão de design deliberada
para o Rust limitar a onipresença do null e aumentar a segurança do código Rust.

Então, como você obtém o valor `T` de uma variante `Some` quando tem um valor
do tipo `Option<T>` para que possa usar esse valor? O enum `Option<T>` tem um
grande número de métodos que são úteis em uma variedade de situações; você pode
conferi-los em [sua documentação][docs]<!-- ignore -->. Familiarizar-se
com os métodos em `Option<T>` será extremamente útil em sua jornada com
Rust.

Em geral, para usar um valor `Option<T>`, você deseja ter um código que
lidará com cada variante. Você quer algum código que será executado apenas quando você tiver um
valor `Some(T)`, e este código pode usar o `T` interno. Você quer algum
outro código para ser executado apenas se você tiver um valor `None`, e esse código não tem um
valor `T` disponível. A expressão `match` é uma construção de fluxo de controle que
faz exatamente isso quando usada com enums: Ela executará códigos diferentes dependendo
de qual variante do enum ela tem, e esse código pode usar os dados dentro do
valor correspondente.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html