## Trazendo Caminhos para o Escopo com a Palavra-chave `use`

Ter que escrever os caminhos para chamar funções pode parecer inconveniente e
repetitivo. Na Listagem 7-7, quer tenhamos escolhido o caminho absoluto ou relativo para
a função `add_to_waitlist`, toda vez que queríamos chamar `add_to_waitlist`
tínhamos que especificar `front_of_house` e `hosting` também. Felizmente, há
uma maneira de simplificar esse processo: podemos criar um atalho para um caminho com a palavra-chave `use`
uma vez e depois usar o nome mais curto em qualquer outro lugar no escopo.

Na Listagem 7-11, trazemos o módulo `crate::front_of_house::hosting` para o
escopo da função `eat_at_restaurant` para que só tenhamos que especificar
`hosting::add_to_waitlist` para chamar a função `add_to_waitlist` em
`eat_at_restaurant`.

<Listing number="7-11" file-name="src/lib.rs" caption="Trazendo um módulo para o escopo com `use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Adicionar `use` e um caminho em um escopo é semelhante a criar um link simbólico no
sistema de arquivos. Ao adicionar `use crate::front_of_house::hosting` na raiz do crate,
`hosting` é agora um nome válido nesse escopo, assim como se o módulo `hosting`
tivesse sido definido na raiz do crate. Caminhos trazidos para o escopo com `use`
também verificam privacidade, como quaisquer outros caminhos.

Note que `use` apenas cria o atalho para o escopo particular em que o
`use` ocorre. A Listagem 7-12 move a função `eat_at_restaurant` para um novo
módulo filho chamado `customer`, que é então um escopo diferente da instrução `use`,
então o corpo da função não compilará.

<Listing number="7-12" file-name="src/lib.rs" caption="Uma instrução `use` só se aplica no escopo em que está.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

O erro do compilador mostra que o atalho não se aplica mais dentro do
módulo `customer`:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Observe que também há um aviso de que o `use` não é mais usado em seu escopo! Para
corrigir esse problema, mova o `use` para dentro do módulo `customer` também, ou referencie
o atalho no módulo pai com `super::hosting` dentro do módulo filho
`customer`.

### Criando Caminhos `use` Idiomáticos

Na Listagem 7-11, você pode ter se perguntado por que especificamos `use
crate::front_of_house::hosting` e depois chamamos `hosting::add_to_waitlist` em
`eat_at_restaurant`, em vez de especificar o caminho `use` até a
função `add_to_waitlist` para alcançar o mesmo resultado, como na Listagem 7-13.

<Listing number="7-13" file-name="src/lib.rs" caption="Trazendo a função `add_to_waitlist` para o escopo com `use`, o que não é idiomático">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Embora tanto a Listagem 7-11 quanto a Listagem 7-13 realizem a mesma tarefa, a Listagem
7-11 é a maneira idiomática de trazer uma função para o escopo com `use`. Trazer o
módulo pai da função para o escopo com `use` significa que temos que especificar o
módulo pai ao chamar a função. Especificar o módulo pai ao
chamar a função deixa claro que a função não é definida localmente
enquanto ainda minimiza a repetição do caminho completo. O código na Listagem 7-13 é
pouco claro quanto a onde `add_to_waitlist` é definida.

Por outro lado, ao trazer structs, enums e outros itens com `use`,
é idiomático especificar o caminho completo. A Listagem 7-14 mostra a maneira idiomática
de trazer a struct `HashMap` da biblioteca padrão para o escopo de um crate binário.

<Listing number="7-14" file-name="src/main.rs" caption="Trazendo `HashMap` para o escopo de uma maneira idiomática">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Não há uma razão forte por trás desse idioma: é apenas a convenção que
surgiu, e as pessoas se acostumaram a ler e escrever código Rust dessa maneira.

A exceção a este idioma é se estamos trazendo dois itens com o mesmo nome
para o escopo com instruções `use`, porque Rust não permite isso. A Listagem 7-15
mostra como trazer dois tipos `Result` para o escopo que têm o mesmo nome mas
módulos pais diferentes, e como se referir a eles.

<Listing number="7-15" file-name="src/lib.rs" caption="Trazer dois tipos com o mesmo nome para o mesmo escopo requer o uso de seus módulos pais.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Como você pode ver, usar os módulos pais distingue os dois tipos `Result`.
Se em vez disso especificássemos `use std::fmt::Result` e `use std::io::Result`, teríamos
dois tipos `Result` no mesmo escopo, e o Rust não saberia qual deles
queríamos dizer quando usamos `Result`.

### Fornecendo Novos Nomes com a Palavra-chave `as`

Há outra solução para o problema de trazer dois tipos com o mesmo nome
para o mesmo escopo com `use`: Após o caminho, podemos especificar `as` e um novo
nome local, ou *alias*, para o tipo. A Listagem 7-16 mostra outra maneira de escrever
o código na Listagem 7-15 renomeando um dos dois tipos `Result` usando `as`.

<Listing number="7-16" file-name="src/lib.rs" caption="Renomeando um tipo quando ele é trazido para o escopo com a palavra-chave `as`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

Na segunda instrução `use`, escolhemos o novo nome `IoResult` para o
tipo `std::io::Result`, que não entrará em conflito com o `Result` de `std::fmt`
que também trouxemos para o escopo. A Listagem 7-15 e a Listagem 7-16 são
consideradas idiomáticas, então a escolha é sua!

### Reexportando Nomes com `pub use`

Quando trazemos um nome para o escopo com a palavra-chave `use`, o nome é privado para
o escopo no qual o importamos. Para permitir que código fora desse escopo se refira
a esse nome como se tivesse sido definido nesse escopo, podemos combinar `pub` e
`use`. Esta técnica é chamada de *reexportação* porque estamos trazendo um item
para o escopo, mas também tornando esse item disponível para outros trazerem para seu
escopo.

A Listagem 7-17 mostra o código na Listagem 7-11 com `use` no módulo raiz
alterado para `pub use`.

<Listing number="7-17" file-name="src/lib.rs" caption="Tornando um nome disponível para qualquer código usar de um novo escopo com `pub use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Antes dessa mudança, código externo teria que chamar a função `add_to_waitlist`
usando o caminho
`restaurant::front_of_house::hosting::add_to_waitlist()`, o que também teria
exigido que o módulo `front_of_house` fosse marcado como `pub`. Agora que este `pub
use` reexportou o módulo `hosting` do módulo raiz, código externo
pode usar o caminho `restaurant::hosting::add_to_waitlist()` em vez disso.

Reexportar é útil quando a estrutura interna do seu código é diferente
de como os programadores chamando seu código pensariam sobre o domínio. Por
exemplo, nesta metáfora de restaurante, as pessoas administrando o restaurante pensam
sobre "frente da casa" e "fundos da casa". Mas clientes visitando um restaurante
provavelmente não pensarão sobre as partes do restaurante nesses termos. Com `pub
use`, podemos escrever nosso código com uma estrutura, mas expor uma estrutura diferente.
Fazer isso torna nossa biblioteca bem organizada para programadores trabalhando na biblioteca
e programadores chamando a biblioteca. Veremos outro exemplo de `pub use`
e como isso afeta a documentação do seu crate em ["Exportando uma API Pública Conveniente"][ch14-pub-use]<!-- ignore --> no Capítulo 14.

### Usando Pacotes Externos

No Capítulo 2, programamos um projeto de jogo de adivinhação que usava um pacote
externo chamado `rand` para obter números aleatórios. Para usar `rand` em nosso projeto,
adicionamos esta linha ao *Cargo.toml*:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

Adicionar `rand` como uma dependência no *Cargo.toml* diz ao Cargo para baixar o
pacote `rand` e quaisquer dependências do [crates.io](https://crates.io/) e
tornar `rand` disponível para nosso projeto.

Então, para trazer definições de `rand` para o escopo do nosso pacote, adicionamos uma
linha `use` começando com o nome do crate, `rand`, e listamos os itens que
queríamos trazer para o escopo. Lembre-se que em ["Gerando um Número
Aleatório"][rand]<!-- ignore --> no Capítulo 2, trouxemos a trait `Rng` para o
escopo e chamamos a função `rand::thread_rng`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Membros da comunidade Rust disponibilizaram muitos pacotes em
[crates.io](https://crates.io/), e puxar qualquer um deles para o seu pacote
envolve esses mesmos passos: listá-los no arquivo *Cargo.toml* do seu pacote e
usar `use` para trazer itens de seus crates para o escopo.

Observe que a biblioteca padrão `std` também é um crate que é externo ao nosso
pacote. Como a biblioteca padrão é enviada com a linguagem Rust, nós
não precisamos mudar o *Cargo.toml* para incluir `std`. Mas precisamos nos referir a
ela com `use` para trazer itens de lá para o escopo do nosso pacote. Por exemplo,
com `HashMap` usaríamos esta linha:

```rust
use std::collections::HashMap;
```

Este é um caminho absoluto começando com `std`, o nome do crate da biblioteca padrão.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### Usando Caminhos Aninhados para Limpar Grandes Listas `use`

Se estamos usando vários itens definidos no mesmo crate ou mesmo módulo, listar
cada item em sua própria linha pode ocupar muito espaço vertical em nossos arquivos. Por
exemplo, estas duas instruções `use` que tivemos no jogo de adivinhação na Listagem 2-4
trazem itens de `std` para o escopo:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Em vez disso, podemos usar caminhos aninhados para trazer os mesmos itens para o escopo em uma
linha. Fazemos isso especificando a parte comum do caminho, seguida por dois
dois pontos, e então chaves ao redor de uma lista das partes dos caminhos que
diferem, conforme mostrado na Listagem 7-18.

<Listing number="7-18" file-name="src/main.rs" caption="Especificando um caminho aninhado para trazer vários itens com o mesmo prefixo para o escopo">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Em programas maiores, trazer muitos itens para o escopo do mesmo crate ou
módulo usando caminhos aninhados pode reduzir muito o número de instruções `use` separadas
necessárias!

Podemos usar um caminho aninhado em qualquer nível em um caminho, o que é útil ao combinar
duas instruções `use` que compartilham um subcaminho. Por exemplo, a Listagem 7-19 mostra duas
instruções `use`: uma que traz `std::io` para o escopo e uma que traz
`std::io::Write` para o escopo.

<Listing number="7-19" file-name="src/lib.rs" caption="Duas instruções `use` onde uma é um subcaminho da outra">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

A parte comum desses dois caminhos é `std::io`, e esse é o primeiro caminho
completo. Para fundir esses dois caminhos em uma instrução `use`, podemos usar `self` no
caminho aninhado, conforme mostrado na Listagem 7-20.

<Listing number="7-20" file-name="src/lib.rs" caption="Combinando os caminhos na Listagem 7-19 em uma instrução `use`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Esta linha traz `std::io` e `std::io::Write` para o escopo.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### Importando Itens com o Operador Glob

Se quisermos trazer *todos* os itens públicos definidos em um caminho para o escopo, podemos
especificar esse caminho seguido pelo operador glob `*`:

```rust
use std::collections::*;
```

Esta instrução `use` traz todos os itens públicos definidos em `std::collections` para
o escopo atual. Tenha cuidado ao usar o operador glob! Glob pode tornar
mais difícil dizer quais nomes estão no escopo e onde um nome usado em seu programa
foi definido. Além disso, se a dependência mudar suas definições, o que
você importou muda também, o que pode levar a erros de compilação quando você
atualiza a dependência se a dependência adicionar uma definição com o mesmo nome
que uma definição sua no mesmo escopo, por exemplo.

O operador glob é frequentemente usado ao testar para trazer tudo sob teste para
o módulo `tests`; falaremos sobre isso em ["Como Escrever
Testes"][writing-tests]<!-- ignore --> no Capítulo 11. O operador glob também é
às vezes usado como parte do padrão prelude: Veja [a documentação da biblioteca
padrão](../std/prelude/index.html#other-preludes)<!-- ignore --> para mais
informações sobre esse padrão.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
