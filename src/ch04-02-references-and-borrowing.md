## Referências e Borrowing

O problema com o código de tupla na Listagem 4-5 é que temos que retornar a `String` para a função chamadora para que possamos ainda usar a `String` depois da chamada para `calculate_length`, porque a `String` foi movida para dentro de `calculate_length`. Em vez disso, podemos fornecer uma referência ao valor da `String`. Uma referência é como um ponteiro no sentido de que é um endereço que podemos seguir para acessar os dados armazenados naquele endereço; esses dados pertencem a alguma outra variável. Diferente de um ponteiro, uma referência é garantida para apontar para um valor válido de um tipo particular durante a vida daquela referência.

Aqui está como você definiria e usaria uma função `calculate_length` que tem uma referência a um objeto como parâmetro em vez de tomar *ownership* do valor:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

Primeiro, note que todo o código de tupla na declaração da variável e no valor de retorno da função se foi. Segundo, note que passamos `&s1` para `calculate_length` e, em sua definição, recebemos `&String` em vez de `String`. Esses "e comerciais" representam referências, e eles permitem que você se refira a algum valor sem tomar *ownership* dele. A Figura 4-6 descreve esse conceito.

<img alt="Three tables: the table for s contains only a pointer to the table
for s1. The table for s1 contains the stack data for s1 and points to the
string data on the heap." src="img/trpl04-06.svg" class="center" />

<span class="caption">Figura 4-6: Um diagrama de `&String` `s` apontando para `String` `s1`</span>

> Nota: O oposto de referenciar usando `&` é *dereferencing* (desreferenciar), que é realizado com o operador de desreferência, `*`. Veremos alguns usos do operador de desreferência no Capítulo 8 e discutiremos detalhes de desreferência no Capítulo 15.

Vamos dar uma olhada mais de perto na chamada da função aqui:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

A sintaxe `&s1` nos permite criar uma referência que se *refere* ao valor de `s1` mas não o possui. Como a referência não o possui, o valor para o qual ela aponta não será descartado quando a referência parar de ser usada.

Da mesma forma, a assinatura da função usa `&` para indicar que o tipo do parâmetro `s` é uma referência. Vamos adicionar algumas anotações explicativas:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

O escopo no qual a variável `s` é válida é o mesmo que o escopo de qualquer parâmetro de função, mas o valor apontado pela referência não é descartado quando `s` para de ser usada, porque `s` não tem *ownership*. Quando funções têm referências como parâmetros em vez dos valores reais, não precisaremos retornar os valores para devolver o *ownership*, porque nunca tivemos *ownership*.

Chamamos a ação de criar uma referência de *borrowing* (empréstimo). Assim como na vida real, se uma pessoa possui algo, você pode pegar emprestado dela. Quando você terminar, você tem que devolver. Você não possui aquilo.

Então, o que acontece se tentarmos modificar algo que estamos pegando emprestado? Tente o código na Listagem 4-6. Alerta de spoiler: não funciona!

<Listing number="4-6" file-name="src/main.rs" caption="Tentando modificar um valor emprestado">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

Aqui está o erro:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Assim como variáveis são imutáveis por padrão, referências também são. Não temos permissão para modificar algo para o qual temos uma referência.

### Referências Mutáveis

Podemos consertar o código da Listagem 4-6 para nos permitir modificar um valor emprestado com apenas alguns pequenos ajustes que usam, em vez disso, uma *referência mutável*:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

Primeiro, mudamos `s` para ser `mut`. Então, criamos uma referência mutável com `&mut s` onde chamamos a função `change` e atualizamos a assinatura da função para aceitar uma referência mutável com `some_string: &mut String`. Isso torna muito claro que a função `change` irá mutar o valor que ela empresta.

Referências mutáveis têm uma grande restrição: se você tem uma referência mutável para um valor, você não pode ter outras referências para aquele valor. Este código que tenta criar duas referências mutáveis para `s` falhará:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

Aqui está o erro:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Esse erro diz que este código é inválido porque não podemos emprestar `s` como mutável mais de uma vez por vez. O primeiro empréstimo mutável está em `r1` e deve durar até que seja usado no `println!`, mas entre a criação daquela referência mutável e seu uso, tentamos criar outra referência mutável em `r2` que empresta os mesmos dados que `r1`.

A restrição que impede múltiplas referências mutáveis para os mesmos dados ao mesmo tempo permite mutação, mas de uma forma muito controlada. É algo com que novos *Rustaceans* lutam porque a maioria das linguagens permite que você mute sempre que quiser. O benefício de ter essa restrição é que o Rust pode prevenir *data races* (corridas de dados) em tempo de compilação. Uma *data race* é similar a uma condição de corrida e acontece quando estes três comportamentos ocorrem:

- Dois ou mais ponteiros acessam os mesmos dados ao mesmo tempo.
- Pelo menos um dos ponteiros está sendo usado para escrever nos dados.
- Não há mecanismo sendo usado para sincronizar o acesso aos dados.

*Data races* causam comportamento indefinido e podem ser difíceis de diagnosticar e corrigir quando você está tentando rastreá-los em tempo de execução; o Rust previne esse problema recusando-se a compilar código com *data races*!

Como sempre, podemos usar chaves para criar um novo escopo, permitindo múltiplas referências mutáveis, apenas não *simultâneas*:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

O Rust impõe uma regra similar para combinar referências mutáveis e imutáveis. Este código resulta em um erro:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Aqui está o erro:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Ufa! Nós *também* não podemos ter uma referência mutável enquanto temos uma imutável para o mesmo valor.

Usuários de uma referência imutável não esperam que o valor mude repentinamente debaixo deles! No entanto, múltiplas referências imutáveis são permitidas porque ninguém que está apenas lendo os dados tem a capacidade de afetar a leitura dos dados de qualquer outra pessoa.

Note que o escopo de uma referência começa de onde ela é introduzida e continua até a última vez que essa referência é usada. Por exemplo, este código compilará porque o último uso das referências imutáveis está no `println!`, antes que a referência mutável seja introduzida:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Os escopos das referências imutáveis `r1` e `r2` terminam após o `println!` onde elas são usadas pela última vez, que é antes da referência mutável `r3` ser criada. Esses escopos não se sobrepõem, então este código é permitido: o compilador pode dizer que a referência não está mais sendo usada em um ponto antes do fim do escopo.

Mesmo que erros de empréstimo possam ser frustrantes às vezes, lembre-se de que é o compilador do Rust apontando um bug potencial cedo (em tempo de compilação em vez de em tempo de execução) e mostrando exatamente onde o problema está. Então, você não tem que rastrear por que seus dados não são o que você pensava que eram.

### Referências Pendentes (Dangling References)

Em linguagens com ponteiros, é fácil criar erroneamente um *dangling pointer* (ponteiro pendente) — um ponteiro que referencia uma localização na memória que pode ter sido dada a outra pessoa — liberando alguma memória enquanto preserva um ponteiro para aquela memória. No Rust, por outro lado, o compilador garante que referências nunca serão referências pendentes: se você tem uma referência para alguns dados, o compilador garantirá que os dados não sairão de escopo antes que a referência para os dados saia.

Vamos tentar criar uma referência pendente para ver como o Rust as previne com um erro em tempo de compilação:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

Aqui está o erro:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Essa mensagem de erro refere-se a uma característica que não cobrimos ainda: *lifetimes* (tempos de vida). Discutiremos *lifetimes* em detalhes no Capítulo 10. Mas, se você desconsiderar as partes sobre *lifetimes*, a mensagem contém a chave para o porquê deste código ser um problema:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

Vamos dar uma olhada mais de perto exatamente no que está acontecendo em cada estágio do nosso código `dangle`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

Como `s` é criada dentro de `dangle`, quando o código de `dangle` termina, `s` será desalocada. Mas tentamos retornar uma referência a ela. Isso significa que essa referência estaria apontando para uma `String` inválida. Isso não é bom! O Rust não nos deixará fazer isso.

A solução aqui é retornar a `String` diretamente:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Isso funciona sem problemas. O *ownership* é movido para fora, e nada é desalocado.

### As Regras de Referências

Vamos recapitular o que discutimos sobre referências:

- A qualquer momento, você pode ter *ou* uma referência mutável *ou* qualquer número de referências imutáveis.
- Referências devem ser sempre válidas.

A seguir, olharemos para um tipo diferente de referência: *slices*.
