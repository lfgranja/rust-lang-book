# Traits Avançados

Primeiro cobrimos [[Traits]] na seção "Definindo Comportamento Compartilhado com Traits" no Capítulo 10, mas não discutimos os detalhes mais avançados. Agora que você sabe mais sobre Rust, podemos entrar nos detalhes.

## Especificando Tipos de Espaço Reservado em Definições de Trait com Tipos Associados

*Tipos associados* (associated types) conectam um espaço reservado de tipo com um trait de modo que as definições de método do trait possam usar esses tipos de espaço reservado em suas assinaturas. O implementador de um trait especificará o tipo concreto a ser usado em vez do tipo de espaço reservado para a implementação específica. Dessa forma, podemos definir um trait que usa alguns tipos sem precisar saber exatamente quais são esses tipos até que o trait seja implementado.

Descrevemos a maioria das funcionalidades avançadas neste capítulo como sendo raramente necessárias. Tipos associados estão em algum lugar no meio: eles são usados mais raramente do que as funcionalidades explicadas no resto do livro, mas mais comumente do que muitas das outras funcionalidades discutidas neste capítulo.

Um exemplo de um trait com um tipo associado é o trait `Iterator` que a biblioteca padrão fornece. O tipo associado é nomeado `Item` e representa o tipo dos valores sobre os quais o tipo que implementa o trait `Iterator` está iterando. A definição do trait `Iterator` é mostrada no Listagem 19-12.

Listagem 19-12: A definição do trait `Iterator` que tem um tipo associado `Item`

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

O tipo `Item` é um espaço reservado, e a definição do método `next` mostra que ele retornará valores do tipo `Option<Self::Item>`. Os implementadores do trait `Iterator` especificarão o tipo concreto para `Item`, e o método `next` retornará um `Option` contendo um valor desse tipo concreto.

Tipos associados podem parecer um conceito semelhante aos genéricos, na medida em que os últimos nos permitem definir uma função sem especificar quais tipos ela pode manipular. Para examinar a diferença entre os dois conceitos, veremos uma implementação do trait `Iterator` em um tipo chamado `Contador` que especifica que o tipo `Item` é `u32`:

```rust
impl Iterator for Contador {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --trecho omitido--
        Some(0)
    }
}
```

Esta sintaxe parece comparável à de genéricos. Então, por que não apenas definir o trait `Iterator` com genéricos, como mostrado no Listagem 19-13?

Listagem 19-13: Uma definição hipotética do trait `Iterator` usando genéricos

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

A diferença é que, ao usar genéricos, como no Listagem 19-13, devemos anotar os tipos em cada implementação; porque também podemos implementar `Iterator<String> para Contador` ou qualquer outro tipo, poderíamos ter múltiplas implementações de `Iterator` para `Contador`. Em outras palavras, quando um trait tem um parâmetro genérico, ele pode ser implementado para um tipo várias vezes, alterando os tipos concretos dos parâmetros de tipo genérico a cada vez. Quando usamos o método `next` em `Contador`, teríamos que fornecer anotações de tipo para indicar qual implementação de `Iterator` queremos usar.

Com tipos associados, não precisamos anotar tipos porque não podemos implementar um trait em um tipo várias vezes. No Listagem 19-12 com a definição que usa tipos associados, podemos escolher qual será o tipo de `Item` apenas uma vez, porque só pode haver um `impl Iterator for Contador`. Não precisamos especificar que queremos um iterador de valores `u32` em todos os lugares que chamamos `next` em `Contador`.

Tipos associados também se tornam parte do contrato do trait: os implementadores do trait devem fornecer um tipo para substituir o espaço reservado de tipo associado. Tipos associados geralmente têm um nome que descreve como o tipo será usado, e documentar o tipo associado na documentação da API é uma boa prática.

## Parâmetros de Tipo Genérico Padrão e Sobrecarga de Operador

Quando usamos parâmetros de tipo genérico, podemos especificar um tipo concreto padrão para o tipo genérico. Isso elimina a necessidade de os implementadores do trait especificarem um tipo concreto se o tipo padrão funcionar. Você especifica um tipo padrão ao declarar um tipo genérico com a sintaxe `<TipoPlaceholder=TipoConcreto>`.

Um ótimo exemplo de uma situação em que essa técnica é útil é com a *sobrecarga de operador* (operator overloading), na qual você personaliza o comportamento de um operador (como `+`) em situações específicas.

Rust não permite que você crie seus próprios operadores ou sobrecarregue operadores arbitrários. Mas você pode sobrecarregar as operações e traits correspondentes listados em `std::ops` implementando os traits associados ao operador. Por exemplo, no Listagem 19-14, sobrecarregamos o operador `+` para adicionar duas instâncias de `Ponto` juntas. Fazemos isso implementando o trait `Add` em uma struct `Ponto`.

Listagem 19-14: Implementando o trait `Add` para sobrecarregar o operador `+` para instâncias de `Ponto`

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Ponto {
    x: i32,
    y: i32,
}

impl Add for Ponto {
    type Output = Ponto;

    fn add(self, other: Ponto) -> Ponto {
        Ponto {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Ponto { x: 1, y: 0 } + Ponto { x: 2, y: 3 },
        Ponto { x: 3, y: 3 }
    );
}
```

O método `add` adiciona os valores `x` de duas instâncias de `Ponto` e os valores `y` de duas instâncias de `Ponto` para criar um novo `Ponto`. O trait `Add` tem um tipo associado chamado `Output` que determina o tipo retornado do método `add`.

O tipo genérico padrão neste código está dentro do trait `Add`. Aqui está sua definição:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Este código deve parecer geralmente familiar: um trait com um método e um tipo associado. A nova parte é `Rhs=Self`: esta sintaxe é chamada de *parâmetros de tipo padrão*. O parâmetro de tipo genérico `Rhs` (abreviação de "right hand side", ou lado direito) define o tipo do parâmetro `rhs` no método `add`. Se não especificarmos um tipo concreto para `Rhs` quando implementarmos o trait `Add`, o tipo de `Rhs` será `Self` por padrão, que será o tipo em que estamos implementando `Add`.

Quando implementamos `Add` para `Ponto`, usamos o padrão para `Rhs` porque queríamos adicionar duas instâncias de `Ponto`. Vamos ver um exemplo de implementação do trait `Add` onde queremos personalizar o tipo `Rhs` em vez de usar o padrão.

Temos duas structs, `Milimetros` e `Metros`, contendo valores em unidades diferentes. Esse encapsulamento fino de um tipo existente em outra struct é conhecido como o *padrão newtype*, que descrevemos com mais detalhes na seção "Usando o Padrão Newtype para Implementar Traits Externos em Tipos Externos". Queremos adicionar valores em milímetros a valores em metros e ter a implementação de `Add` fazendo a conversão corretamente. Podemos implementar `Add` para `Milimetros` com `Metros` como o `Rhs`, como mostrado no Listagem 19-15.

Listagem 19-15: Implementando o trait `Add` em `Milimetros` para adicionar `Milimetros` e `Metros`

```rust
use std::ops::Add;

struct Milimetros(u32);
struct Metros(u32);

impl Add<Metros> for Milimetros {
    type Output = Milimetros;

    fn add(self, other: Metros) -> Milimetros {
        Milimetros(self.0 + (other.0 * 1000))
    }
}
```

Para adicionar `Milimetros` e `Metros`, especificamos `impl Add<Metros>` para definir o valor do parâmetro de tipo `Rhs` em vez de usar o padrão de `Self`.

Você usará parâmetros de tipo padrão de duas maneiras principais:

1. Para estender um tipo sem quebrar o código existente
2. Para permitir personalização em casos específicos que a maioria dos usuários não precisará

O trait `Add` da biblioteca padrão é um exemplo do segundo propósito: geralmente, você adicionará dois tipos semelhantes, mas o trait `Add` fornece a capacidade de personalizar além disso. Usar um parâmetro de tipo padrão na definição do trait `Add` significa que você não precisa especificar o parâmetro extra na maioria das vezes. Em outras palavras, um pouco de código repetitivo de implementação não é necessário, tornando mais fácil usar o trait.

O primeiro propósito é semelhante ao segundo, mas ao contrário: se você quiser adicionar um parâmetro de tipo a um trait existente, pode dar a ele um padrão para permitir a extensão da funcionalidade do trait sem quebrar o código de implementação existente.

## Sintaxe Totalmente Qualificada para Desambiguação: Chamando Métodos com o Mesmo Nome

Nada em Rust impede que um trait tenha um método com o mesmo nome que o método de outro trait, nem Rust impede que você implemente ambos os traits em um tipo. Também é possível implementar um método diretamente no tipo com o mesmo nome que métodos de traits.

Ao chamar métodos com o mesmo nome, você precisará dizer a Rust qual deles deseja usar. Considere o código no Listagem 19-16, onde definimos dois traits, `Piloto` e `Mago`, que ambos têm um método chamado `voar`. Em seguida, implementamos ambos os traits em um tipo `Humano` que já tem um método chamado `voar` implementado nele. Cada método `voar` faz algo diferente.

Listagem 19-16: Dois traits são definidos para ter um método `voar` e são implementados no tipo `Humano`, e um método `voar` é implementado em `Humano` diretamente

```rust
trait Piloto {
    fn voar(&self);
}

trait Mago {
    fn voar(&self);
}

struct Humano;

impl Piloto for Humano {
    fn voar(&self) {
        println!("Este é o seu capitão falando.");
    }
}

impl Mago for Humano {
    fn voar(&self) {
        println!("Para cima!");
    }
}

impl Humano {
    fn voar(&self) {
        println!("*acenando os braços furiosamente*");
    }
}
```

Quando chamamos `voar` em uma instância de `Humano`, o compilador padroniza para chamar o método que é implementado diretamente no tipo, como mostrado no Listagem 19-17.

Listagem 19-17: Chamando `voar` em uma instância de `Humano`

```rust
fn main() {
    let pessoa = Humano;
    pessoa.voar();
}
```

Executar este código imprimirá `*acenando os braços furiosamente*`, mostrando que Rust chamou o método `voar` implementado em `Humano` diretamente.

Para chamar os métodos `voar` do trait `Piloto` ou do trait `Mago`, precisamos usar uma sintaxe mais explícita para especificar qual método `voar` queremos dizer. O Listagem 19-18 demonstra esta sintaxe.

Listagem 19-18: Especificando qual método `voar` do trait queremos chamar

```rust
fn main() {
    let pessoa = Humano;
    Piloto::voar(&pessoa);
    Mago::voar(&pessoa);
    pessoa.voar();
}
```

Especificar o nome do trait antes do nome do método esclarece para Rust qual implementação de `voar` queremos chamar. Também poderíamos escrever `Humano::voar(&pessoa)`, que é equivalente ao `pessoa.voar()` que usamos no Listagem 19-18, mas isso é um pouco mais longo para escrever se não precisarmos desambiguar.

Executar este código imprime o seguinte:

```console
Este é o seu capitão falando.
Para cima!
*acenando os braços furiosamente*
```

Como o método `voar` recebe um parâmetro `self`, se tivéssemos dois *tipos* que implementam um *trait*, Rust poderia descobrir qual implementação de um trait usar com base no tipo de `self`.

No entanto, funções associadas que não são métodos não têm um parâmetro `self`. Quando há vários tipos ou traits que definem funções não-método com o mesmo nome de função, Rust nem sempre sabe qual tipo você quer dizer, a menos que você use a sintaxe totalmente qualificada. Por exemplo, no Listagem 19-19, criamos um trait para um abrigo de animais que quer nomear todos os filhotes de cachorro de Spot. Fazemos um trait `Animal` com uma função associada não-método `nome_filhote`. O trait `Animal` é implementado para a struct `Cachorro`, na qual também fornecemos uma função associada não-método `nome_filhote` diretamente.

Listagem 19-19: Um trait com uma função associada e um tipo com uma função associada do mesmo nome que também implementa o trait

```rust
trait Animal {
    fn nome_filhote() -> String;
}

struct Cachorro;

impl Cachorro {
    fn nome_filhote() -> String {
        String::from("Spot")
    }
}

impl Animal for Cachorro {
    fn nome_filhote() -> String {
        String::from("filhote")
    }
}

fn main() {
    println!("Um filhote de cachorro é chamado de {}", Cachorro::nome_filhote());
}
```

Implementamos o código para nomear todos os filhotes Spot na função associada `nome_filhote` que é definida em `Cachorro`. O tipo `Cachorro` também implementa o trait `Animal`, que descreve características que todos os animais têm. Filhotes de cachorro são chamados de *puppies* (filhotes), e isso é expresso na implementação do trait `Animal` em `Cachorro` na função `nome_filhote` associada ao trait `Animal`.

Em `main`, chamamos a função `Cachorro::nome_filhote`, que chama a função associada definida em `Cachorro` diretamente. Este código imprime o seguinte:

```console
Um filhote de cachorro é chamado de Spot
```

Esta saída não é o que queríamos. Queremos chamar a função `nome_filhote` que faz parte do trait `Animal` que implementamos em `Cachorro` para que o código imprima `Um filhote de cachorro é chamado de filhote`. A técnica de especificar o nome do trait que usamos no Listagem 19-18 não ajuda aqui; se mudarmos `main` para o código no Listagem 19-20, receberemos um erro de compilação.

Listagem 19-20: Tentando chamar a função `nome_filhote` do trait `Animal`, mas Rust não sabe qual implementação usar

```rust,ignore,does_not_compile
fn main() {
    println!("Um filhote de cachorro é chamado de {}", Animal::nome_filhote());
}
```

Como `Animal::nome_filhote` não tem um parâmetro `self`, e poderia haver outros tipos que implementam o trait `Animal`, Rust não consegue descobrir qual implementação de `Animal::nome_filhote` queremos. Receberemos este erro de compilador:

```console
error[E0283]: type annotations needed
 --> src/main.rs:20:43
  |
20 |     println!("Um filhote de cachorro é chamado de {}", Animal::nome_filhote());
  |                                                        ^^^^^^^^^^^^^^^^^^^^ cannot infer type
  |
  = note: cannot satisfy `_: Animal`
```

Para desambiguar e dizer a Rust que queremos usar a implementação de `Animal` para `Cachorro` em oposição à implementação de `Animal` para algum outro tipo, precisamos usar a sintaxe totalmente qualificada. O Listagem 19-21 demonstra como usar a sintaxe totalmente qualificada.

Listagem 19-21: Usando sintaxe totalmente qualificada para especificar que queremos chamar a função `nome_filhote` do trait `Animal` conforme implementado em `Cachorro`

```rust
fn main() {
    println!("Um filhote de cachorro é chamado de {}", <Cachorro as Animal>::nome_filhote());
}
```

Estamos fornecendo a Rust uma anotação de tipo dentro dos colchetes angulares, o que indica que queremos chamar o método `nome_filhote` do trait `Animal` conforme implementado em `Cachorro` dizendo que queremos tratar o tipo `Cachorro` como um `Animal` para esta chamada de função. Este código agora imprimirá o que queremos:

```console
Um filhote de cachorro é chamado de filhote
```

Em geral, a sintaxe totalmente qualificada é definida da seguinte forma:

```rust,ignore
<Tipo as Trait>::funcao(receptor_se_metodo, prox_arg, ...);
```

Para funções associadas que não são métodos, não haveria um `receptor`: haveria apenas a lista de outros argumentos. Você poderia usar a sintaxe totalmente qualificada em todos os lugares que chama funções ou métodos. No entanto, você tem permissão para omitir qualquer parte dessa sintaxe que Rust possa descobrir a partir de outras informações no programa. Você só precisa usar essa sintaxe mais verbosa em casos em que há múltiplas implementações que usam o mesmo nome e Rust precisa de ajuda para identificar qual implementação você deseja chamar.

## Usando Supertraits para Exigir a Funcionalidade de Um Trait Dentro de Outro Trait

Às vezes, você pode escrever uma definição de trait que depende de outro trait: para um tipo implementar o primeiro trait, você deseja exigir que esse tipo também implemente o segundo trait. Você faria isso para que sua definição de trait pudesse fazer uso dos itens associados do segundo trait. O trait do qual sua definição de trait depende é chamado de *supertrait* do seu trait.

Por exemplo, digamos que queremos criar um trait `ImprimirEsboco` com um método `imprimir_esboco` que imprimirá um determinado valor formatado para que seja emoldurado em asteriscos. Ou seja, dado uma struct `Ponto` que implementa o trait da biblioteca padrão `Display` para resultar em `(x, y)`, quando chamamos `imprimir_esboco` em uma instância de `Ponto` que tem `1` para `x` e `3` para `y`, ele deve imprimir o seguinte:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Na implementação do método `imprimir_esboco`, queremos usar a funcionalidade do trait `Display`. Portanto, precisamos especificar que o trait `ImprimirEsboco` funcionará apenas para tipos que também implementam `Display` e fornecem a funcionalidade que `ImprimirEsboco` precisa. Podemos fazer isso na definição do trait especificando `ImprimirEsboco: Display`. Essa técnica é semelhante a adicionar um limite de trait ao trait. O Listagem 19-22 mostra uma implementação do trait `ImprimirEsboco`.

Listagem 19-22: Implementando o trait `ImprimirEsboco` que requer a funcionalidade de `Display`

```rust
use std::fmt;

trait ImprimirEsboco: fmt::Display {
    fn imprimir_esboco(&self) {
        let saida = self.to_string();
        let len = saida.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", saida);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

Como especificamos que `ImprimirEsboco` requer o trait `Display`, podemos usar a função `to_string` que é implementada automaticamente para qualquer tipo que implementa `Display`. Se tentássemos usar `to_string` sem adicionar dois pontos e especificar o trait `Display` após o nome do trait, receberíamos um erro dizendo que nenhum método chamado `to_string` foi encontrado para o tipo `&Self` no escopo atual.

Vamos ver o que acontece quando tentamos implementar `ImprimirEsboco` em um tipo que não implementa `Display`, como a struct `Ponto`:

```rust
struct Ponto {
    x: i32,
    y: i32,
}

impl ImprimirEsboco for Ponto {}
```

Recebemos um erro dizendo que `Display` é necessário, mas não implementado:

```console
error[E0277]: the trait bound `Ponto: std::fmt::Display` is not satisfied
  --> src/main.rs:20:6
   |
20 | impl ImprimirEsboco for Ponto {}
   |      ^^^^^^^^^^^^^^ `Ponto` cannot be formatted with the default formatter; try using `:?` instead if you are using a format string
   |
   = help: the trait `std::fmt::Display` is not implemented for `Ponto`
```

Para corrigir isso, implementamos `Display` em `Ponto` e satisfazemos a restrição que `ImprimirEsboco` requer, assim:

```rust
use std::fmt;

impl fmt::Display for Ponto {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

Então, implementar o trait `ImprimirEsboco` em `Ponto` compilará com sucesso, e podemos chamar `imprimir_esboco` em uma instância de `Ponto` para exibi-lo dentro de um contorno de asteriscos.

## Usando o Padrão Newtype para Implementar Traits Externos em Tipos Externos

Na seção "Implementando um Trait em um Tipo" no Capítulo 10, mencionamos a regra do órfão (orphan rule) que afirma que só temos permissão para implementar um trait em um tipo se o trait ou o tipo, ou ambos, forem locais para nossa crate. É possível contornar essa restrição usando o *padrão newtype* (novo tipo), que envolve criar um novo tipo em uma struct de tupla. (Cobrimos structs de tupla na seção "Usando Structs de Tupla sem Campos Nomeados para Criar Tipos Diferentes" no Capítulo 5.) A struct de tupla terá um campo e será um invólucro fino em torno do tipo para o qual queremos implementar um trait. Então, o tipo de invólucro é local para nossa crate, e podemos implementar o trait no invólucro. *Newtype* é um termo que se origina da linguagem de programação Haskell. Não há penalidade de desempenho em tempo de execução para usar esse padrão, e o tipo de invólucro é elidido em tempo de compilação.

Como exemplo, digamos que queremos implementar `Display` em `Vec<T>`, o que a regra do órfão nos impede de fazer diretamente porque o trait `Display` e o tipo `Vec<T>` são definidos fora de nossa crate. Podemos fazer uma struct `Wrapper` que contém uma instância de `Vec<T>`; então, podemos implementar `Display` em `Wrapper` e usar o valor `Vec<T>`, como mostrado no Listagem 19-23.

Listagem 19-23: Criando um tipo `Wrapper` em torno de `Vec<String>` para implementar `Display`

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("olá"), String::from("mundo")]);
    println!("w = {}", w);
}
```

A implementação de `Display` usa `self.0` para acessar o `Vec<T>` interno porque `Wrapper` é uma struct de tupla e `Vec<T>` é o item no índice 0 na tupla. Então, podemos usar a funcionalidade do trait `Display` em `Wrapper`.

A desvantagem de usar essa técnica é que `Wrapper` é um novo tipo, portanto, não tem os métodos do valor que está segurando. Teríamos que implementar todos os métodos de `Vec<T>` diretamente em `Wrapper` de modo que os métodos deleguem para `self.0`, o que nos permitiria tratar `Wrapper` exatamente como um `Vec<T>`. Se quiséssemos que o novo tipo tivesse todos os métodos que o tipo interno tem, implementar o trait `Deref` no `Wrapper` para retornar o tipo interno seria uma solução (discutimos a implementação do trait `Deref` na seção "Tratando Ponteiros Inteligentes Como Referências Regulares com o Trait `Deref`" no Capítulo 15). Se não quiséssemos que o tipo `Wrapper` tivesse todos os métodos do tipo interno — por exemplo, para restringir o comportamento do tipo `Wrapper` — teríamos que implementar apenas os métodos que queremos manualmente.

Esse padrão newtype também é útil mesmo quando traits não estão envolvidos. Vamos mudar o foco e ver algumas maneiras avançadas de interagir com o sistema de tipos de Rust.
