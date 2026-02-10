# Sintaxe de Padrões

Nesta seção, reunimos toda a sintaxe que é válida em padrões e discutimos por que e quando você pode querer usar cada uma.

## Correspondendo a Literais

Como você viu no Capítulo 6, você pode corresponder padrões a literais diretamente. O código a seguir fornece alguns exemplos:

```rust
    let x = 1;

    match x {
        1 => println!("um"),
        2 => println!("dois"),
        3 => println!("três"),
        _ => println!("qualquer coisa"),
    }
```

Este código imprime `um` porque o valor em `x` é `1`. Essa sintaxe é útil quando você quer que seu código tome uma ação se obtiver um valor concreto específico.

## Correspondendo a Variáveis Nomeadas

Variáveis nomeadas são padrões irrefutáveis que correspondem a qualquer valor, e nós as usamos muitas vezes neste livro. No entanto, há uma complicação quando você usa variáveis nomeadas em expressões `match`, `if let` ou `while let`. Como cada uma dessas expressões inicia um novo escopo, as variáveis declaradas como parte de um padrão dentro dessas expressões sombrearão aquelas com o mesmo nome fora das construções, como é o caso de todas as variáveis. No Listagem 19-11, declaramos uma variável chamada `x` com o valor `Some(5)` e uma variável `y` com o valor `10`. Em seguida, criamos uma expressão `match` no valor `x`. Olhe para os padrões nos braços de correspondência e `println!` no final, e tente descobrir o que o código imprimirá antes de executar este código ou ler mais.

Listagem 19-11: Uma expressão `match` com um braço que introduz uma nova variável que sombreia uma variável existente `y`

```rust
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Obteve 50"),
        Some(y) => println!("Correspondeu, y = {:?}", y),
        _ => println!("Caso padrão, x = {:?}", x),
    }

    println!("no final: x = {:?}, y = {:?}", x, y);
```

Vamos ver o que acontece quando a expressão `match` é executada. O padrão no primeiro braço de correspondência não corresponde ao valor definido de `x`, então o código continua.

O padrão no segundo braço de correspondência introduz uma nova variável chamada `y` que corresponderá a qualquer valor dentro de um valor `Some`. Como estamos em um novo escopo dentro da expressão `match`, esta é uma nova variável `y`, não a `y` que declaramos no início com o valor `10`. Esta nova ligação `y` corresponderá a qualquer valor dentro de um `Some`, que é o que temos em `x`. Portanto, este novo `y` se vincula ao valor interno do `Some` em `x`. Esse valor é `5`, então a expressão para esse braço é executada e imprime `Correspondeu, y = 5`.

Se `x` tivesse sido um valor `None` em vez de `Some(5)`, os padrões nos dois primeiros braços não teriam correspondido, então o valor teria correspondido ao sublinhado. Não introduzimos a variável `x` no padrão do braço de sublinhado, então o `x` na expressão ainda é o `x` externo que não foi sombreado. Neste caso hipotético, o `match` imprimiria `Caso padrão, x = None`.

Quando a expressão `match` termina, seu escopo termina, e o escopo do `y` interno também. O último `println!` produz `no final: x = Some(5), y = 10`.

Para criar uma expressão `match` que compare os valores do `x` e `y` externos, em vez de introduzir uma nova variável que sombreie a variável `y` existente, precisaríamos usar uma condicional *match guard* (guarda de correspondência). Falaremos sobre match guards mais tarde na seção "Adicionando Condicionais com Match Guards".

## Correspondendo a Múltiplos Padrões

Em expressões `match`, você pode corresponder a múltiplos padrões usando a sintaxe `|`, que é o operador *ou* (or) de padrão. Por exemplo, no código a seguir, correspondemos o valor de `x` aos braços de correspondência, o primeiro dos quais tem uma opção *ou*, significando que se o valor de `x` corresponder a qualquer um dos valores naquele braço, o código daquele braço será executado:

```rust
    let x = 1;

    match x {
        1 | 2 => println!("um ou dois"),
        3 => println!("três"),
        _ => println!("qualquer coisa"),
    }
```

Este código imprime `um ou dois`.

## Correspondendo a Intervalos de Valores com `..=`

A sintaxe `..=` nos permite corresponder a um intervalo inclusivo de valores. No código a seguir, quando um padrão corresponde a qualquer um dos valores dentro do intervalo fornecido, esse braço será executado:

```rust
    let x = 5;

    match x {
        1..=5 => println!("um até cinco"),
        _ => println!("alguma outra coisa"),
    }
```

Se `x` for `1`, `2`, `3`, `4` ou `5`, o primeiro braço corresponderá. Essa sintaxe é mais conveniente para múltiplos valores de correspondência do que usar o operador `|` para expressar a mesma ideia; se fôssemos usar `|`, teríamos que especificar `1 | 2 | 3 | 4 | 5`. Especificar um intervalo é muito mais curto, especialmente se quisermos corresponder, digamos, qualquer número entre 1 e 1.000!

O compilador verifica se o intervalo não está vazio em tempo de compilação, e como os únicos tipos para os quais Rust pode dizer se um intervalo está vazio ou não são `char` e valores numéricos, intervalos só são permitidos com valores numéricos ou `char`.

Aqui está um exemplo usando intervalos de valores `char`:

```rust
    let x = 'c';

    match x {
        'a'..='j' => println!("letra ASCII inicial"),
        'k'..='z' => println!("letra ASCII tardia"),
        _ => println!("algo mais"),
    }
```

Rust pode dizer que `'c'` está dentro do intervalo do primeiro padrão e imprime `letra ASCII inicial`.

## Desestruturando para Quebrar Valores

Também podemos usar padrões para [[Destructuring]] structs, enums e tuplas para usar diferentes partes desses valores. Vamos percorrer cada valor.

### Structs

O Listagem 19-12 mostra uma struct `Ponto` com dois campos, `x` e `y`, que podemos quebrar usando um padrão com uma declaração `let`.

Listagem 19-12: Desestruturando os campos de uma struct em variáveis separadas

```rust
struct Ponto {
    x: i32,
    y: i32,
}

fn main() {
    let p = Ponto { x: 0, y: 7 };

    let Ponto { x: a, y: b } = p;

    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

Este código cria as variáveis `a` e `b` que correspondem aos valores dos campos `x` e `y` da struct `p`. Este exemplo mostra que os nomes das variáveis no padrão não precisam corresponder aos nomes dos campos da struct. No entanto, é comum corresponder os nomes das variáveis aos nomes dos campos para facilitar a lembrança de quais variáveis vieram de quais campos. Por causa desse uso comum, e porque escrever `let Ponto { x: x, y: y } = p;` contém muita duplicação, Rust tem uma abreviação para padrões que correspondem a campos de struct: você só precisa listar o nome do campo da struct, e as variáveis criadas a partir do padrão terão os mesmos nomes. O Listagem 19-13 se comporta da mesma maneira que o código no Listagem 19-12, mas as variáveis criadas no padrão `let` são `x` e `y` em vez de `a` e `b`.

Listagem 19-13: Desestruturando campos de struct usando abreviação de campo de struct

```rust
struct Ponto {
    x: i32,
    y: i32,
}

fn main() {
    let p = Ponto { x: 0, y: 7 };

    let Ponto { x, y } = p;

    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

Este código cria as variáveis `x` e `y` que correspondem aos campos `x` e `y` da variável `p`. O resultado é que as variáveis `x` e `y` contêm os valores da struct `p`.

Também podemos desestruturar com valores literais como parte do padrão da struct em vez de criar variáveis para todos os campos. Fazer isso nos permite testar alguns dos campos para valores particulares enquanto criamos variáveis para desestruturar os outros campos.

No Listagem 19-14, temos uma expressão `match` que separa valores `Ponto` em três casos: pontos que estão diretamente no eixo `x` (o que é verdade quando `y = 0`), no eixo `y` (`x = 0`), ou em nenhum dos eixos.

Listagem 19-14: Desestruturando e correspondendo valores literais em um padrão

```rust
struct Ponto {
    x: i32,
    y: i32,
}

fn main() {
    let p = Ponto { x: 0, y: 7 };

    match p {
        Ponto { x, y: 0 } => println!("No eixo x em {}", x),
        Ponto { x: 0, y } => println!("No eixo y em {}", y),
        Ponto { x, y } => println!("Em nenhum eixo: ({}, {})", x, y),
    }
}
```

O primeiro braço corresponderá a qualquer ponto que esteja no eixo `x` especificando que o campo `y` corresponde se seu valor corresponder ao literal `0`. O padrão ainda cria uma variável `x` que podemos usar no código para este braço.

Da mesma forma, o segundo braço corresponde a qualquer ponto no eixo `y` especificando que o campo `x` corresponde se seu valor for `0` e cria uma variável `y` para o valor do campo `y`. O terceiro braço não especifica nenhum literal, então ele corresponde a qualquer outro `Ponto` e cria variáveis para os campos `x` e `y`.

Neste exemplo, o valor `p` corresponde ao segundo braço em virtude de `x` conter um `0`, então este código imprimirá `No eixo y em 7`.

### Enums

Desestruturamos enums neste livro (por exemplo, Listagem 6-5 no Capítulo 6), mas ainda não discutimos explicitamente que o padrão para desestruturar um enum corresponde à maneira como os dados armazenados dentro do enum são definidos. Como exemplo, no Listagem 19-15, usamos o enum `Mensagem` do Listagem 6-2 e escrevemos um `match` com padrões que desestruturarão cada valor interno.

Listagem 19-15: Desestruturando variantes de enum que contêm diferentes tipos de valores

```rust
enum Mensagem {
    Sair,
    Mover { x: i32, y: i32 },
    Escrever(String),
    MudarCor(i32, i32, i32),
}

fn main() {
    let msg = Mensagem::MudarCor(0, 160, 255);

    match msg {
        Mensagem::Sair => {
            println!("A variante Sair não tem dados para desestruturar.")
        }
        Mensagem::Mover { x, y } => {
            println!(
                "Mover na direção x {} e na direção y {}",
                x, y
            );
        }
        Mensagem::Escrever(text) => println!("Mensagem de texto: {}", text),
        Mensagem::MudarCor(r, g, b) => {
            println!(
                "Mudar a cor para vermelho {}, verde {}, e azul {}",
                r, g, b
            )
        }
    }
}
```

Este código imprimirá `Mudar a cor para vermelho 0, verde 160, e azul 255`. Tente mudar o valor de `msg` para ver o código dos outros braços serem executados.

Para variantes de enum sem dados, como `Mensagem::Sair`, não podemos desestruturar o valor mais. Podemos apenas corresponder ao valor literal `Mensagem::Sair`, e nenhuma variável está nesse padrão.

Para variantes de enum semelhantes a structs, como `Mensagem::Mover`, podemos usar um padrão semelhante ao padrão que especificamos para corresponder a structs. Após o nome da variante, colocamos chaves e listamos os campos com variáveis para que separemos as peças para usar no código deste braço. Aqui usamos a forma abreviada como fizemos no Listagem 19-13.

Para variantes de enum semelhantes a tuplas, como `Mensagem::Escrever` que contém uma tupla com um elemento e `Mensagem::MudarCor` que contém uma tupla com três elementos, o padrão é semelhante ao padrão que especificamos para corresponder a tuplas. O número de variáveis no padrão deve corresponder ao número de elementos na variante que estamos correspondendo.

### Structs e Enums Aninhados

Até agora, nossos exemplos foram todos correspondendo structs ou enums com um nível de profundidade, mas a correspondência pode funcionar em itens aninhados também! Por exemplo, podemos refatorar o código no Listagem 19-15 para suportar cores RGB e HSV na mensagem `MudarCor`, como mostrado no Listagem 19-16.

Listagem 19-16: Correspondendo em enums aninhados

```rust
enum Cor {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Mensagem {
    Sair,
    Mover { x: i32, y: i32 },
    Escrever(String),
    MudarCor(Cor),
}

fn main() {
    let msg = Mensagem::MudarCor(Cor::Hsv(0, 160, 255));

    match msg {
        Mensagem::MudarCor(Cor::Rgb(r, g, b)) => {
            println!(
                "Mudar a cor para vermelho {}, verde {}, e azul {}",
                r, g, b
            )
        }
        Mensagem::MudarCor(Cor::Hsv(h, s, v)) => {
            println!(
                "Mudar a cor para matiz {}, saturação {}, e valor {}",
                h, s, v
            )
        }
        _ => (),
    }
}
```

O padrão do primeiro braço na expressão `match` corresponde a uma variante de enum `Mensagem::MudarCor` que contém uma variante `Cor::Rgb`; então, o padrão se vincula aos três valores `i32` internos. O padrão do segundo braço também corresponde a uma variante de enum `Mensagem::MudarCor`, mas o enum interno corresponde a `Cor::Hsv`. Podemos especificar essas condições complexas em uma expressão `match`, mesmo que dois enums estejam envolvidos.

### Structs e Tuplas

Podemos misturar, combinar e aninhar padrões de desestruturação de maneiras ainda mais complexas. O exemplo a seguir mostra uma desestruturação complicada onde aninhamos structs e tuplas dentro de uma tupla e desestruturamos todos os valores primitivos:

```rust
    struct Ponto {
        x: i32,
        y: i32,
    }

    let ((pés, polegadas), Ponto { x, y }) = ((3, 10), Ponto { x: 3, y: -10 });
```

Este código nos permite quebrar tipos complexos em suas partes componentes para que possamos usar os valores em que estamos interessados separadamente.

## Ignorando Valores em um Padrão

Você viu que às vezes é útil ignorar valores em um padrão, como no último braço de um `match`, para obter um pega-tudo que não faz nada, mas contabiliza todos os valores possíveis restantes. Existem algumas maneiras de ignorar valores inteiros ou partes de valores em um padrão: usando o padrão `_` (que você viu), usando o padrão `_` dentro de outro padrão, usando um nome que começa com um sublinhado, ou usando `..` para ignorar partes restantes de um valor. Vamos explorar como e por que usar cada um desses padrões.

### Ignorando um Valor Inteiro com `_`

Usamos o sublinhado como um padrão curinga que corresponderá a qualquer valor, mas não se vinculará ao valor. Isso é especialmente útil como o último braço em uma expressão `match`, mas também podemos usá-lo em qualquer padrão, incluindo parâmetros de função, como mostrado no Listagem 19-17.

Listagem 19-17: Usando `_` em uma assinatura de função

```rust
fn foo(_: i32, y: i32) {
    println!("Este código usa apenas o parâmetro y: {}", y);
}

fn main() {
    foo(3, 4);
}
```

Este código ignorará completamente o valor `3` passado como o primeiro argumento e imprimirá `Este código usa apenas o parâmetro y: 4`.

Na maioria dos casos, quando você não precisa mais de um parâmetro de função específico, você alteraria a assinatura para que ela não inclua o parâmetro não utilizado. Ignorar um parâmetro de função pode ser especialmente útil em casos em que, por exemplo, você está implementando um trait quando precisa de uma determinada assinatura de tipo, mas o corpo da função em sua implementação não precisa de um dos parâmetros. Você então evita receber um aviso do compilador sobre parâmetros de função não utilizados, como aconteceria se usasse um nome.

### Ignorando Partes de um Valor com um `_` Aninhado

Também podemos usar `_` dentro de outro padrão para ignorar apenas parte de um valor, por exemplo, quando queremos testar apenas parte de um valor, mas não temos uso para as outras partes no código correspondente que queremos executar. O Listagem 19-18 mostra código responsável por gerenciar o valor de uma configuração. Os requisitos de negócios são que o usuário não deve ter permissão para sobrescrever uma personalização existente de uma configuração, mas pode desabilitar a configuração e dar a ela um valor se ela estiver atualmente desabilitada.

Listagem 19-18: Usando um sublinhado dentro de padrões que correspondem a variantes `Some` quando não precisamos usar o valor dentro do `Some`

```rust
    let mut config_valor = Some(5);
    let nova_config_valor = Some(10);

    match (config_valor, nova_config_valor) {
        (Some(_), Some(_)) => {
            println!("Não é possível sobrescrever um valor personalizado existente");
        }
        _ => {
            config_valor = nova_config_valor;
        }
    }

    println!("configuração é {:?}", config_valor);
```

Este código imprimirá `Não é possível sobrescrever um valor personalizado existente` e depois `configuração é Some(5)`. No primeiro braço de correspondência, não precisamos corresponder ou usar os valores dentro de nenhuma variante `Some`, mas precisamos testar o caso quando `config_valor` e `nova_config_valor` são a variante `Some`. Nesse caso, imprimimos o motivo de não alterar `config_valor`, e ele não é alterado.

Em todos os outros casos (se `config_valor` ou `nova_config_valor` for `None`) expressos pelo padrão `_` no segundo braço, queremos permitir que `nova_config_valor` se torne `config_valor`.

Também podemos usar sublinhados em vários lugares dentro de um padrão para ignorar valores específicos. O Listagem 19-19 mostra um exemplo de ignorar o segundo e o quarto valores em uma tupla de cinco itens.

Listagem 19-19: Ignorando várias partes de uma tupla

```rust
    let numeros = (2, 4, 8, 16, 32);

    match numeros {
        (primeiro, _, terceiro, _, quinto) => {
            println!("Alguns números: {}, {}, {}", primeiro, terceiro, quinto)
        }
    }
```

Este código imprimirá `Alguns números: 2, 8, 32`, e os valores `4` e `16` serão ignorados.

### Ignorando uma Variável Não Utilizada Começando Seu Nome com `_`

Se você criar uma variável, mas não a usar em nenhum lugar, Rust geralmente emitirá um aviso porque uma variável não utilizada pode ser um bug. No entanto, às vezes é útil poder criar uma variável que você não usará ainda, como quando você está prototipando ou apenas começando um projeto. Nessa situação, você pode dizer a Rust para não avisá-lo sobre a variável não utilizada começando o nome da variável com um sublinhado. No Listagem 19-20, criamos duas variáveis não utilizadas, mas quando compilamos este código, devemos receber apenas um aviso sobre uma delas.

Listagem 19-20: Começando um nome de variável com um sublinhado para evitar receber avisos de variáveis não utilizadas

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

Aqui, recebemos um aviso sobre não usar a variável `y`, mas não recebemos um aviso sobre não usar `_x`.

Note que há uma diferença sutil entre usar apenas `_` e usar um nome que começa com um sublinhado. A sintaxe `_x` ainda vincula o valor à variável, enquanto `_` não vincula de forma alguma. Para mostrar um caso em que essa distinção importa, o Listagem 19-21 nos fornecerá um erro.

Listagem 19-21: Uma variável não utilizada começando com um sublinhado ainda vincula o valor, o que pode tomar posse do valor

```rust,ignore,does_not_compile
    let s = Some(String::from("Olá!"));

    if let Some(_s) = s {
        println!("encontrou uma string");
    }

    println!("{:?}", s);
```

Receberemos um erro porque o valor `s` ainda será movido para `_s`, o que nos impede de usar `s` novamente. No entanto, usar o sublinhado por si só nunca vincula ao valor. O Listagem 19-22 compilará sem erros porque `s` não é movido para `_`.

Listagem 19-22: Usando um sublinhado não vincula o valor

```rust
    let s = Some(String::from("Olá!"));

    if let Some(_) = s {
        println!("encontrou uma string");
    }

    println!("{:?}", s);
```

Este código funciona muito bem porque nunca vinculamos `s` a nada; ele não é movido.

### Ignorando Partes Restantes de um Valor com `..`

Com valores que têm muitas partes, podemos usar a sintaxe `..` para usar partes específicas e ignorar o resto, evitando a necessidade de listar sublinhados para cada valor ignorado. O padrão `..` ignora quaisquer partes de um valor que não correspondemos explicitamente no resto do padrão. No Listagem 19-23, temos uma struct `Ponto` que contém uma coordenada no espaço tridimensional. Na expressão `match`, queremos operar apenas na coordenada `x` e ignorar os valores nos campos `y` e `z`.

Listagem 19-23: Ignorando todos os campos de um `Ponto` exceto `x` usando `..`

```rust
    struct Ponto {
        x: i32,
        y: i32,
        z: i32,
    }

    let origem = Ponto { x: 0, y: 0, z: 0 };

    match origem {
        Ponto { x, .. } => println!("x é {}", x),
    }
```

Listamos o valor `x` e depois apenas incluímos o padrão `..`. Isso é mais rápido do que ter que listar `y: _` e `z: _`, particularmente quando estamos trabalhando com structs que têm muitos campos em situações onde apenas um ou dois campos são relevantes.

A sintaxe `..` se expandirá para quantos valores forem necessários. O Listagem 19-24 mostra como usar `..` com uma tupla.

Listagem 19-24: Correspondendo apenas ao primeiro e último valores em uma tupla e ignorando todos os outros valores

```rust
fn main() {
    let numeros = (2, 4, 8, 16, 32);

    match numeros {
        (primeiro, .., ultimo) => {
            println!("Alguns números: {}, {}", primeiro, ultimo);
        }
    }
}
```

Neste código, o primeiro e o último valores são correspondidos com `primeiro` e `ultimo`. O `..` corresponderá e ignorará tudo no meio.

No entanto, usar `..` deve ser inequívoco. Se não estiver claro quais valores são destinados à correspondência e quais devem ser ignorados, Rust nos dará um erro. O Listagem 19-25 mostra um exemplo de uso de `..` de forma ambígua, então ele não será compilado.

Listagem 19-25: Uma tentativa de usar `..` de forma ambígua

```rust,ignore,does_not_compile
fn main() {
    let numeros = (2, 4, 8, 16, 32);

    match numeros {
        (.., segundo, ..) => {
            println!("Alguns números: {}", segundo)
        },
    }
}
```

Quando compilamos este exemplo, recebemos este erro:

```console
error: `..` can only be used once per tuple pattern
 --> src/main.rs:5:22
  |
5 |         (.., segundo, ..) => {
  |          --           ^^ can only be used once per tuple pattern
  |          |
  |          previously used here
```

É impossível para Rust determinar quantos valores na tupla ignorar antes de corresponder um valor com `segundo` e depois quantos valores adicionais ignorar depois disso. Este código poderia significar que queremos ignorar `2`, vincular `segundo` a `4` e depois ignorar `8`, `16` e `32`; ou que queremos ignorar `2` e `4`, vincular `segundo` a `8` e depois ignorar `16` e `32`; e assim por diante. O nome da variável `segundo` não significa nada especial para Rust, então recebemos um erro de compilador porque usar `..` em dois lugares como este é ambíguo.

## Adicionando Condicionais com Match Guards

Um *match guard* (guarda de correspondência) é uma condição `if` adicional, especificada após o padrão em um braço `match`, que também deve corresponder para que esse braço seja escolhido. Match guards são úteis para expressar ideias mais complexas do que um padrão sozinho permite. Note, no entanto, que eles estão disponíveis apenas em expressões `match`, não em expressões `if let` ou `while let`.

A condição pode usar variáveis criadas no padrão. O Listagem 19-26 mostra um `match` onde o primeiro braço tem o padrão `Some(x)` e também tem um match guard de `if x % 2 == 0` (que será `true` se o número for par).

Listagem 19-26: Adicionando um match guard a um padrão

```rust
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("O número {} é par", x),
        Some(x) => println!("O número {} é ímpar", x),
        None => (),
    }
```

Este exemplo imprimirá `O número 4 é par`. Quando `num` é comparado ao padrão no primeiro braço, ele corresponde porque `Some(4)` corresponde a `Some(x)`. Então, o match guard verifica se o resto da divisão de `x` por 2 é igual a 0, e como é, o primeiro braço é selecionado.

Se `num` tivesse sido `Some(5)` em vez disso, o match guard no primeiro braço teria sido `false` porque o resto da divisão de 5 por 2 é 1, que não é igual a 0. Rust iria então para o segundo braço, que corresponderia porque o segundo braço não tem um match guard e, portanto, corresponde a qualquer variante `Some`.

Não há como expressar a condição `if x % 2 == 0` dentro de um padrão, então o match guard nos dá a capacidade de expressar essa lógica. A desvantagem dessa expressividade adicional é que o compilador não tenta verificar a exaustividade quando expressões de match guard estão envolvidas.

Ao discutir o Listagem 19-11, mencionamos que poderíamos usar match guards para resolver nosso problema de sombreamento de padrões. Lembre-se de que criamos uma nova variável dentro do padrão na expressão `match` em vez de usar a variável fora do `match`. Essa nova variável significava que não podíamos testar contra o valor da variável externa. O Listagem 19-27 mostra como podemos usar um match guard para corrigir esse problema.

Listagem 19-27: Usando um match guard para testar a igualdade com uma variável externa

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Obteve 50"),
        Some(n) if n == y => println!("Correspondeu, n = {}", n),
        _ => println!("Caso padrão, x = {:?}", x),
    }

    println!("no final: x = {:?}, y = {}", x, y);
}
```

Este código agora imprimirá `Caso padrão, x = Some(5)`. O padrão no segundo braço de correspondência não introduz uma nova variável `y` que sombrearia o `y` externo, o que significa que podemos usar o `y` externo no match guard. Em vez de especificar o padrão como `Some(y)`, que teria sombreado o `y` externo, especificamos `Some(n)`. Isso cria uma nova variável `n` que não sombreia nada porque não há variável `n` fora do `match`.

O match guard `if n == y` não é um padrão e, portanto, não introduz novas variáveis. Este `y` *é* o `y` externo em vez de um novo `y` sombreando-o, e podemos procurar por um valor que tenha o mesmo valor que o `y` externo comparando `n` com `y`.

Você também pode usar o operador *ou* `|` em um match guard para especificar múltiplos padrões; a condição do match guard se aplicará a todos os padrões. O Listagem 19-28 mostra a precedência ao combinar um padrão que usa `|` com um match guard. A parte importante deste exemplo é que o match guard `if y` se aplica a `4`, `5` *e* `6`, mesmo que pareça que `if y` se aplica apenas a `6`.

Listagem 19-28: Combinando múltiplos padrões com um match guard

```rust
    let x = 4;
    let y = false;

    match x {
        4 | 5 | 6 if y => println!("sim"),
        _ => println!("não"),
    }
```

A condição de correspondência afirma que o braço só corresponde se o valor de `x` for igual a `4`, `5` ou `6` *e* se `y` for `true`. Quando este código é executado, o padrão do primeiro braço corresponde porque `x` é `4`, mas o match guard `if y` é `false`, então o primeiro braço não é escolhido. O código passa para o segundo braço, que corresponde, e este programa imprime `não`. A razão é que a condição `if` se aplica a todo o padrão `4 | 5 | 6`, não apenas ao último valor `6`. Em outras palavras, a precedência de um match guard em relação a um padrão se comporta assim:

```text
(4 | 5 | 6) if y => ...
```

em vez de assim:

```text
4 | 5 | (6 if y) => ...
```

Depois de executar o código, o comportamento de precedência é evidente: se o match guard fosse aplicado apenas ao valor final na lista de valores especificados usando o operador `|`, o braço teria correspondido e o programa teria impresso `sim`.

## Vínculos `@`

O operador *at* (em) `@` nos permite criar uma variável que contém um valor ao mesmo tempo em que estamos testando esse valor para uma correspondência de padrão. No Listagem 19-29, queremos testar se um campo `id` de `Mensagem::Ola` está dentro do intervalo `3..=7`. Também queremos vincular o valor à variável `id` para que possamos usá-lo no código associado ao braço.

Listagem 19-29: Usando `@` para vincular a um valor em um padrão enquanto também o testa

```rust
    enum Mensagem {
        Ola { id: i32 },
    }

    let msg = Mensagem::Ola { id: 5 };

    match msg {
        Mensagem::Ola {
            id: id_variavel @ 3..=7,
        } => println!("Encontrou um id no intervalo: {}", id_variavel),
        Mensagem::Ola { id: 10..=12 } => {
            println!("Encontrou um id em outro intervalo")
        }
        Mensagem::Ola { id } => println!("Encontrou algum outro id: {}", id),
    }
```

Este exemplo imprimirá `Encontrou um id no intervalo: 5`. Ao especificar `id_variavel @` antes do intervalo `3..=7`, estamos capturando qualquer valor que correspondeu ao intervalo em uma variável chamada `id_variavel` enquanto também testamos se o valor correspondeu ao padrão de intervalo.

No segundo braço, onde temos apenas um intervalo especificado no padrão, o código associado ao braço não tem uma variável que contenha o valor real do campo `id`. O valor do campo `id` poderia ter sido 10, 11 ou 12, mas o código que vai com esse padrão não sabe qual é. O código do padrão não é capaz de usar o valor do campo `id` porque não salvamos o valor `id` em uma variável.

No último braço, onde especificamos uma variável sem intervalo, temos o valor disponível para usar no código do braço em uma variável chamada `id`. A razão é que usamos a sintaxe abreviada de campo de struct. Mas não aplicamos nenhum teste ao valor no campo `id` neste braço, como fizemos com os dois primeiros braços: qualquer valor corresponderia a este padrão.

Usar `@` nos permite testar um valor e salvá-lo em uma variável dentro de um padrão.

## Resumo

Os padrões de Rust são muito úteis para distinguir entre diferentes tipos de dados. Quando usados em expressões `match`, Rust garante que seus padrões cubram todos os valores possíveis, ou seu programa não compilará. Padrões em declarações `let` e parâmetros de função tornam essas construções mais úteis, permitindo a desestruturação de valores em partes menores e atribuindo essas partes a variáveis. Podemos criar padrões simples ou complexos para atender às nossas necessidades.

A seguir, para o penúltimo capítulo do livro, veremos alguns aspectos avançados de uma variedade de recursos de Rust.
