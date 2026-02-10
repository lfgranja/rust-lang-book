# Todos os Lugares Onde Padrões Podem Ser Usados

Padrões aparecem em vários lugares em Rust, e você os tem usado muito sem perceber! Esta seção discute todos os lugares onde os padrões são válidos.

## Braços de `match`

Como discutido no Capítulo 6, usamos padrões nos braços de expressões `match`. Formalmente, expressões `match` são definidas como a palavra-chave `match`, um valor para corresponder e um ou mais braços de correspondência que consistem em um padrão e uma expressão para executar se o valor corresponder ao padrão desse braço, assim:

```text
match VALOR {
    PADRÃO => EXPRESSÃO,
    PADRÃO => EXPRESSÃO,
    PADRÃO => EXPRESSÃO,
}
```

Por exemplo, aqui está a expressão `match` do Listagem 6-5 que corresponde a um valor `Option<i32>` na variável `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Os padrões nesta expressão `match` são o `None` e `Some(i)` à esquerda de cada seta.

Um requisito para expressões `match` é que elas precisam ser exaustivas, no sentido de que todas as possibilidades para o valor na expressão `match` devem ser contabilizadas. Uma maneira de garantir que você cobriu todas as possibilidades é ter um padrão "pega-tudo" (catch-all) para o último braço: por exemplo, um nome de variável correspondendo a qualquer valor nunca pode falhar e, portanto, cobre todos os casos restantes.

O padrão particular `_` corresponderá a qualquer coisa, mas nunca se vincula a uma variável, por isso é frequentemente usado no último braço de correspondência. O padrão `_` pode ser útil quando você deseja ignorar qualquer valor não especificado, por exemplo. Cobriremos o padrão `_` com mais detalhes na seção "Ignorando Valores em um Padrão" mais adiante neste capítulo.

## Declarações `let`

Antes deste capítulo, havíamos discutido explicitamente apenas o uso de padrões com `match` e `if let`, mas, de fato, usamos padrões em outros lugares também, incluindo em declarações `let`. Por exemplo, considere esta atribuição de variável direta com `let`:

```rust
let x = 5;
```

Toda vez que você usou uma declaração `let` como essa, você estava usando padrões, embora possa não ter percebido! Mais formalmente, uma declaração `let` se parece com isso:

```text
let PADRÃO = EXPRESSÃO;
```

Em declarações como `let x = 5;` com um nome de variável no espaço PADRÃO, o nome da variável é apenas uma forma particularmente simples de um padrão. Rust compara a expressão com o padrão e atribui quaisquer nomes que encontrar. Portanto, no exemplo `let x = 5;`, `x` é um padrão que significa "vincule o que corresponder aqui à variável `x`". Como o nome `x` é todo o padrão, esse padrão efetivamente significa "vincule tudo à variável `x`, qualquer que seja o valor".

Para ver o aspecto de correspondência de padrão de `let` mais claramente, considere o Listagem 19-1, que usa um padrão com `let` para desestruturar uma tupla.

Listagem 19-1: Usando um padrão para desestruturar uma tupla e criar três variáveis de uma vez

```rust
let (x, y, z) = (1, 2, 3);
```

Aqui, correspondemos uma tupla a um padrão. Rust compara o valor `(1, 2, 3)` com o padrão `(x, y, z)` e vê que o valor corresponde ao padrão — isto é, vê que o número de elementos é o mesmo em ambos — então Rust vincula `1` a `x`, `2` a `y` e `3` a `z`. Você pode pensar neste padrão de tupla como aninhando três padrões de variáveis individuais dentro dele.

Se o número de elementos no padrão não corresponder ao número de elementos na tupla, o tipo geral não corresponderá e teremos um erro de compilador. Por exemplo, o Listagem 19-2 mostra uma tentativa de desestruturar uma tupla com três elementos em duas variáveis, o que não funcionará.

Listagem 19-2: Construindo incorretamente um padrão cujas variáveis não correspondem ao número de elementos na tupla

```rust,ignore,does_not_compile
let (x, y) = (1, 2, 3);
```

Tentar compilar este código resulta neste erro de tipo:

```console
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`
```

Para corrigir o erro, poderíamos ignorar um ou mais dos valores na tupla usando `_` ou `..`, como você verá na seção "Ignorando Valores em um Padrão". Se o problema for que temos muitas variáveis no padrão, a solução é fazer os tipos corresponderem removendo variáveis para que o número de variáveis seja igual ao número de elementos na tupla.

## Expressões Condicionais `if let`

No Capítulo 6, discutimos como usar expressões `if let` principalmente como uma maneira mais curta de escrever o equivalente a um `match` que corresponde apenas a um caso. Opcionalmente, `if let` pode ter um `else` correspondente contendo código para executar se o padrão no `if let` não corresponder.

O Listagem 19-3 mostra que também é possível misturar e combinar expressões `if let`, `else if` e `else if let`. Fazer isso nos dá mais flexibilidade do que uma expressão `match` na qual podemos expressar apenas um valor para comparar com os padrões. Além disso, Rust não exige que as condições em uma série de braços `if let`, `else if` e `else if let` se relacionem entre si.

O código no Listagem 19-3 determina qual cor usar no fundo com base em uma série de verificações para várias condições. Para este exemplo, criamos variáveis com valores codificados que um programa real poderia receber da entrada do usuário.

Listagem 19-3: Misturando `if let`, `else if`, `else if let` e `else`

```rust
fn main() {
    let cor_favorita: Option<&str> = None;
    let e_terca_feira = false;
    let idade: Result<u8, _> = "34".parse();

    if let Some(cor) = cor_favorita {
        println!("Usando sua cor favorita, {}, como fundo", cor);
    } else if e_terca_feira {
        println!("Terça-feira é dia verde!");
    } else if let Ok(idade) = idade {
        if idade > 30 {
            println!("Usando roxo como cor de fundo");
        } else {
            println!("Usando laranja como cor de fundo");
        }
    } else {
        println!("Usando azul como cor de fundo");
    }
}
```

Se o usuário especificar uma cor favorita, essa cor é usada como fundo. Se nenhuma cor favorita for especificada e hoje for terça-feira, a cor de fundo é verde. Caso contrário, se o usuário especificar sua idade como uma string e pudermos analisá-la como um número com sucesso, a cor será roxa ou laranja, dependendo do valor do número. Se nenhuma dessas condições se aplicar, a cor de fundo será azul.

Essa estrutura condicional nos permite suportar requisitos complexos. Com os valores codificados que temos aqui, este exemplo imprimirá `Usando roxo como cor de fundo`.

Você pode ver que `if let` também pode introduzir novas variáveis que sombreiam variáveis existentes da mesma maneira que os braços `match` podem: A linha `if let Ok(idade) = idade` introduz uma nova variável `idade` que contém o valor dentro da variante `Ok`, sombreando a variável `idade` existente. Isso significa que precisamos colocar a condição `if idade > 30` dentro desse bloco: não podemos combinar essas duas condições em `if let Ok(idade) = idade && idade > 30`. A nova `idade` que queremos comparar com 30 não é válida até que o novo escopo comece com a chave.

A desvantagem de usar expressões `if let` é que o compilador não verifica a exaustividade, enquanto com expressões `match` ele verifica. Se omitíssemos o último bloco `else` e, portanto, deixássemos de lidar com alguns casos, o compilador não nos alertaria sobre o possível bug lógico.

## Loops Condicionais `while let`

Semelhante em construção ao `if let`, o loop condicional `while let` permite que um loop `while` execute enquanto um padrão continuar correspondendo. No Listagem 19-4, mostramos um loop `while let` que espera mensagens enviadas entre threads, mas neste caso verificando um `Result` em vez de um `Option`.

Listagem 19-4: Usando um loop `while let` para imprimir valores enquanto `rx.recv()` retornar `Ok`

```rust
    let mut pilha = Vec::new();

    pilha.push(1);
    pilha.push(2);
    pilha.push(3);

    while let Some(top) = pilha.pop() {
        println!("{}", top);
    }
```

Este exemplo imprime `3`, `2` e depois `1`. O método `pop` retira o último elemento do vetor e retorna `Some(valor)`. Se o vetor estiver vazio, `pop` retorna `None`. O loop `while` continua executando o código em seu bloco enquanto `pop` retornar `Some`. Quando `pop` retornar `None`, o loop para. Podemos usar `while let` para retirar cada elemento de nossa pilha.

## Loops `for`

Em um loop `for`, o valor que segue diretamente a palavra-chave `for` é um padrão. Por exemplo, em `for x in y`, o `x` é o padrão. O Listagem 19-5 demonstra como usar um padrão em um loop `for` para desestruturar, ou separar, uma tupla como parte do loop `for`.

Listagem 19-5: Usando um padrão em um loop `for` para desestruturar uma tupla

```rust
    let v = vec!['a', 'b', 'c'];

    for (index, valor) in v.iter().enumerate() {
        println!("{} está no índice {}", valor, index);
    }
```

O código no Listagem 19-5 imprimirá o seguinte:

```console
a está no índice 0
b está no índice 1
c está no índice 2
```

Adaptamos um iterador usando o método `enumerate` para que ele produza um valor e o índice para esse valor, colocados em uma tupla. O primeiro valor produzido é a tupla `(0, 'a')`. Quando esse valor é correspondido ao padrão `(index, valor)`, `index` será `0` e `valor` será `'a'`, imprimindo a primeira linha da saída.

## Parâmetros de Função

Parâmetros de função também podem ser padrões. O código no Listagem 19-6, que declara uma função chamada `foo` que recebe um parâmetro chamado `x` do tipo `i32`, deve parecer familiar agora.

Listagem 19-6: Uma assinatura de função usando padrões nos parâmetros

```rust
fn foo(x: i32) {
    // código vai aqui
}
```

A parte `x` é um padrão! Como fizemos com `let`, poderíamos corresponder uma tupla nos argumentos de uma função ao padrão. O Listagem 19-7 divide os valores em uma tupla à medida que a passamos para uma função.

Listagem 19-7: Uma função com parâmetros que desestruturam uma tupla

```rust
fn print_coordenadas(&(x, y): &(i32, i32)) {
    println!("Localização atual: ({}, {})", x, y);
}

fn main() {
    let ponto = (3, 5);
    print_coordenadas(&ponto);
}
```

Este código imprime `Localização atual: (3, 5)`. Os valores `&(3, 5)` correspondem ao padrão `&(x, y)`, então `x` é o valor `3` e `y` é o valor `5`.

Também podemos usar padrões em listas de parâmetros de closure da mesma maneira que em listas de parâmetros de função, porque closures são semelhantes a funções, conforme discutido no Capítulo 13.

Neste ponto, você viu várias maneiras de usar padrões, mas os padrões não funcionam da mesma maneira em todos os lugares em que podemos usá-los. Em alguns lugares, os padrões devem ser irrefutáveis; em outras circunstâncias, eles podem ser refutáveis. Discutiremos esses dois conceitos a seguir.
