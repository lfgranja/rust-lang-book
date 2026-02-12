# Tipos Genéricos, Traits e Lifetimes

Toda linguagem de programação tem ferramentas para lidar efetivamente com a
duplicação de conceitos. Em Rust, uma dessas ferramentas são os _genéricos_:
substitutos abstratos para tipos concretos ou outras propriedades. Podemos
expressar o comportamento de genéricos ou como eles se relacionam com outros
genéricos sem saber o que estará em seu lugar ao compilar e executar o código.

As funções podem receber parâmetros de algum tipo genérico, em vez de um tipo
concreto como `i32` ou `String`, da mesma forma que recebem parâmetros com
valores desconhecidos para executar o mesmo código em vários valores concretos.
Na verdade, já usamos genéricos no Capítulo 6 com `Option<T>`, no Capítulo 8
com `Vec<T>` e `HashMap<K, V>`, e no Capítulo 9 com `Result<T, E>`. Neste
capítulo, você explorará como definir seus próprios tipos, funções e métodos
com genéricos!

Primeiro, revisaremos como extrair uma função para reduzir a duplicação de
código. Em seguida, usaremos a mesma técnica para criar uma função genérica a
partir de duas funções que diferem apenas nos tipos de seus parâmetros. Também
explicaremos como usar tipos genéricos em definições de struct e enum.

Em seguida, você aprenderá como usar traits para definir comportamento de uma
maneira genérica. Você pode combinar traits com tipos genéricos para restringir
um tipo genérico a aceitar apenas os tipos que têm um comportamento particular,
em vez de apenas qualquer tipo.

Finalmente, discutiremos _lifetimes_ (tempos de vida): uma variedade de
genéricos que dão ao compilador informações sobre como as referências se
relacionam umas com as outras. Lifetimes nos permitem dar ao compilador
informações suficientes sobre valores emprestados para que ele possa garantir
que as referências serão válidas em mais situações do que poderia sem nossa
ajuda.

## Removendo Duplicação Extraindo uma Função

Genéricos nos permitem substituir tipos específicos por um espaço reservado que
representa vários tipos para remover a duplicação de código. Antes de mergulhar
na sintaxe de genéricos, vamos primeiro ver como remover a duplicação de uma
maneira que não envolva tipos genéricos extraindo uma função que substitui
valores específicos por um espaço reservado que representa vários valores. Em
seguida, aplicaremos a mesma técnica para extrair uma função genérica! Ao ver
como reconhecer código duplicado que você pode extrair em uma função, você
começará a reconhecer código duplicado que pode usar genéricos.

Começaremos com o programa curto na Listagem 10-1 que encontra o maior número
em uma lista.

<Listing number="10-1" file-name="src/main.rs" caption="Encontrando o maior número em uma lista de números">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Armazenamos uma lista de inteiros na variável `number_list` e colocamos uma
referência ao primeiro número da lista em uma variável chamada `largest`. Em
seguida, iteramos por todos os números da lista e, se o número atual for maior
que o número armazenado em `largest`, substituímos a referência nessa variável.
No entanto, se o número atual for menor ou igual ao maior número visto até
agora, a variável não muda e o código passa para o próximo número da lista.
Depois de considerar todos os números na lista, `largest` deve se referir ao
maior número, que neste caso é 100.

Agora fomos encarregados de encontrar o maior número em duas listas diferentes
de números. Para fazer isso, podemos optar por duplicar o código na Listagem
10-1 e usar a mesma lógica em dois lugares diferentes no programa, conforme
mostrado na Listagem 10-2.

<Listing number="10-2" file-name="src/main.rs" caption="Código para encontrar o maior número em *duas* listas de números">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Embora esse código funcione, duplicar código é tedioso e propenso a erros.
Também temos que lembrar de atualizar o código em vários lugares quando
quisermos alterá-lo.

Para eliminar essa duplicação, criaremos uma abstração definindo uma função que
opera em qualquer lista de inteiros passada como parâmetro. Essa solução torna
nosso código mais claro e nos permite expressar o conceito de encontrar o maior
número em uma lista de forma abstrata.

Na Listagem 10-3, extraímos o código que encontra o maior número em uma função
chamada `largest`. Então, chamamos a função para encontrar o maior número nas
duas listas da Listagem 10-2. Também poderíamos usar a função em qualquer outra
lista de valores `i32` que possamos ter no futuro.

<Listing number="10-3" file-name="src/main.rs" caption="Código abstraído para encontrar o maior número em duas listas">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

A função `largest` tem um parâmetro chamado `list`, que representa qualquer
fatia concreta de valores `i32` que possamos passar para a função. Como
resultado, quando chamamos a função, o código é executado nos valores
específicos que passamos.

Em resumo, aqui estão as etapas que seguimos para alterar o código da Listagem
10-2 para a Listagem 10-3:

1. Identificar código duplicado.
1. Extrair o código duplicado para o corpo da função e especificar as entradas
   e valores de retorno desse código na assinatura da função.
1. Atualizar as duas instâncias de código duplicado para chamar a função.

Em seguida, usaremos essas mesmas etapas com genéricos para reduzir a
duplicação de código. Da mesma forma que o corpo da função pode operar em uma
`list` abstrata em vez de valores específicos, os genéricos permitem que o
código opere em tipos abstratos.

Por exemplo, digamos que tivéssemos duas funções: uma que encontra o maior item
em uma fatia de valores `i32` e uma que encontra o maior item em uma fatia de
valores `char`. Como eliminaríamos essa duplicação? Vamos descobrir!
