## Fluxo de Controle Conciso com `if let` e `let...else`

A sintaxe `if let` permite combinar `if` e `let` em uma maneira menos verbosa de
lidar com valores que correspondem a um padrão enquanto ignora o resto. Considere o
programa na Listagem 6-6 que corresponde a um valor `Option<u8>` na
variável `config_max` mas só quer executar código se o valor for a variante `Some`.

<Listing number="6-6" caption="Um `match` que só se importa em executar código quando o valor é `Some`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Se o valor for `Some`, imprimimos o valor na variante `Some` vinculando
o valor à variável `max` no padrão. Não queremos fazer nada
com o valor `None`. Para satisfazer a expressão `match`, temos que adicionar `_ =>
()` após processar apenas uma variante, o que é um código clichê irritante para
adicionar.

Em vez disso, poderíamos escrever isso de uma forma mais curta usando `if let`. O seguinte
código se comporta da mesma forma que o `match` na Listagem 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

A sintaxe `if let` recebe um padrão e uma expressão separados por um sinal de igual.
Funciona da mesma maneira que um `match`, onde a expressão é dada ao
`match` e o padrão é seu primeiro braço. Neste caso, o padrão é
`Some(max)`, e o `max` se vincula ao valor dentro do `Some`. Podemos então
usar `max` no corpo do bloco `if let` da mesma maneira que usamos `max` no
braço de correspondência correspondente. O código no bloco `if let` só é executado se
o valor corresponder ao padrão.

Usar `if let` significa menos digitação, menos indentação e menos código clichê.
No entanto, você perde a verificação exaustiva que o `match` impõe, que garante que
você não está esquecendo de lidar com nenhum caso. Escolher entre `match` e `if
let` depende do que você está fazendo em sua situação particular e se
ganhar concisão é uma troca apropriada para perder a verificação exaustiva.

Em outras palavras, você pode pensar em `if let` como açúcar sintático para um `match` que
executa código quando o valor corresponde a um padrão e depois ignora todos os outros valores.

Podemos incluir um `else` com um `if let`. O bloco de código que vai com o
`else` é o mesmo que o bloco de código que iria com o caso `_` na
expressão `match` que é equivalente ao `if let` e `else`. Relembre a
definição do enum `Coin` na Listagem 6-4, onde a variante `Quarter` também continha um
valor `UsState`. Se quiséssemos contar todas as moedas não-quarter que vemos enquanto também
anunciamos o estado dos quarters, poderíamos fazer isso com uma expressão `match`,
como esta:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Ou poderíamos usar uma expressão `if let` e `else`, como esta:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## Permanecendo no "Caminho Feliz" com `let...else`

O padrão comum é realizar algum cálculo quando um valor está presente e
retornar um valor padrão caso contrário. Continuando com nosso exemplo de moedas com um
valor `UsState`, se quiséssemos dizer algo engraçado dependendo da idade do
estado na moeda, poderíamos introduzir um método em `UsState` para verificar a
idade de um estado, assim:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Então, poderíamos usar `if let` para corresponder ao tipo de moeda, introduzindo uma variável `state`
dentro do corpo da condição, como na Listagem 6-7.

<Listing number="6-7" caption="Verificando se um estado existia em 1900 usando condicionais aninhadas dentro de um `if let`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Isso faz o trabalho, mas empurrou o trabalho para o corpo da instrução `if
let`, e se o trabalho a ser feito for mais complicado, pode ser
difícil seguir exatamente como os ramos de nível superior se relacionam. Também poderíamos tirar
proveito do fato de que expressões produzem um valor para produzir o
`state` do `if let` ou para retornar antecipadamente, como na Listagem 6-8. (Você poderia fazer
algo semelhante com um `match` também.)

<Listing number="6-8" caption="Usando `if let` para produzir um valor ou retornar antecipadamente">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

Isso é um pouco irritante de seguir à sua maneira, no entanto! Um ramo do `if
let` produz um valor, e o outro retorna da função inteiramente.

Para tornar esse padrão comum mais agradável de expressar, Rust tem `let...else`. A
sintaxe `let...else` recebe um padrão no lado esquerdo e uma expressão no
lado direito, muito semelhante a `if let`, mas não tem um ramo `if`, apenas um
ramo `else`. Se o padrão corresponder, ele vinculará o valor do padrão
no escopo externo. Se o padrão *não* corresponder, o programa fluirá para
o braço `else`, que deve retornar da função.

Na Listagem 6-9, você pode ver como a Listagem 6-8 fica ao usar `let...else` em
lugar de `if let`.

<Listing number="6-9" caption="Usando `let...else` para esclarecer o fluxo através da função">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Observe que ele permanece no "caminho feliz" no corpo principal da função desta
maneira, sem ter um fluxo de controle significativamente diferente para dois ramos da
maneira que o `if let` tinha.

Se você tem uma situação em que seu programa tem uma lógica que é muito verbosa para
expressar usando um `match`, lembre-se de que `if let` e `let...else` estão em sua
caixa de ferramentas Rust também.

## Resumo

Agora cobrimos como usar enums para criar tipos personalizados que podem ser um de um
conjunto de valores enumerados. Mostramos como o tipo `Option<T>` da biblioteca padrão
ajuda você a usar o sistema de tipos para evitar erros. Quando valores de enum têm
dados dentro deles, você pode usar `match` ou `if let` para extrair e usar esses
valores, dependendo de quantos casos você precisa lidar.

Seus programas Rust agora podem expressar conceitos em seu domínio usando structs e
enums. Criar tipos personalizados para usar em sua API garante segurança de tipo: O
compilador garantirá que suas funções recebam apenas valores do tipo que cada
função espera.

A fim de fornecer uma API bem organizada para seus usuários que seja direta
de usar e exponha apenas exatamente o que seus usuários precisarão, vamos agora nos voltar para
os módulos de Rust.
