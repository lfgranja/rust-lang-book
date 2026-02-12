## Traits Avançados

Primeiro cobrimos traits na seção [“Definindo Comportamento Compartilhado com Traits”][traits]<!-- ignore --> no Capítulo 10, mas não discutimos os detalhes mais avançados. Agora que você sabe mais sobre Rust, podemos entrar nos detalhes.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### Definindo Traits com Tipos Associados

_Tipos associados_ conectam um marcador de posição de tipo com uma trait de modo que as definições de método da trait possam usar esses tipos de marcador de posição em suas assinaturas. O implementador de uma trait especificará o tipo concreto a ser usado em vez do tipo de marcador de posição para a implementação específica. Dessa forma, podemos definir uma trait que usa alguns tipos sem precisar saber exatamente quais são esses tipos até que a trait seja implementada.

Descrevemos a maioria das funcionalidades avançadas neste capítulo como sendo raramente necessárias. Tipos associados estão em algum lugar no meio: eles são usados mais raramente do que as funcionalidades explicadas no resto do livro, mas mais comumente do que muitas das outras funcionalidades discutidas neste capítulo.

Um exemplo de uma trait com um tipo associado é a trait `Iterator` que a biblioteca padrão fornece. O tipo associado é chamado `Item` e substitui o tipo dos valores sobre os quais o tipo que implementa a trait `Iterator` está iterando. A definição da trait `Iterator` é conforme mostrado na Listagem 20-13.

<Listing number="20-13" caption="A definição da trait `Iterator` que tem um tipo associado `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

O tipo `Item` é um marcador de posição, e a definição do método `next` mostra que ele retornará valores do tipo `Option<Self::Item>`. Implementadores da trait `Iterator` especificarão o tipo concreto para `Item`, e o método `next` retornará um `Option` contendo um valor desse tipo concreto.

Tipos associados podem parecer um conceito semelhante aos genéricos, pois estes últimos nos permitem definir uma função sem especificar quais tipos ela pode manipular. Para examinar a diferença entre os dois conceitos, vamos olhar para uma implementação da trait `Iterator` em um tipo chamado `Counter` que especifica que o tipo `Item` é `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Esta sintaxe parece comparável à dos genéricos. Então, por que não apenas definir a trait `Iterator` com genéricos, como mostrado na Listagem 20-14?

<Listing number="20-14" caption="Uma definição hipotética da trait `Iterator` usando genéricos">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

A diferença é que, ao usar genéricos, como na Listagem 20-14, devemos anotar os tipos em cada implementação; como também podemos implementar `Iterator<String> for Counter` ou qualquer outro tipo, poderíamos ter múltiplas implementações de `Iterator` para `Counter`. Em outras palavras, quando uma trait tem um parâmetro genérico, ela pode ser implementada para um tipo várias vezes, alterando os tipos concretos dos parâmetros de tipo genérico a cada vez. Quando usamos o método `next` em `Counter`, teríamos que fornecer anotações de tipo para indicar qual implementação de `Iterator` queremos usar.

Com tipos associados, não precisamos anotar tipos, porque não podemos implementar uma trait em um tipo várias vezes. Na Listagem 20-13 com a definição que usa tipos associados, podemos escolher qual será o tipo de `Item` apenas uma vez, porque só pode haver uma `impl Iterator for Counter`. Não precisamos especificar que queremos um iterador de valores `u32` em todos os lugares onde chamamos `next` em `Counter`.

Tipos associados também se tornam parte do contrato da trait: implementadores da trait devem fornecer um tipo para substituir o marcador de posição do tipo associado. Tipos associados geralmente têm um nome que descreve como o tipo será usado, e documentar o tipo associado na documentação da API é uma boa prática.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### Usando Parâmetros Genéricos Padrão e Sobrecarga de Operador

Quando usamos parâmetros de tipo genérico, podemos especificar um tipo concreto padrão para o tipo genérico. Isso elimina a necessidade de implementadores da trait especificarem um tipo concreto se o tipo padrão funcionar. Você especifica um tipo padrão ao declarar um tipo genérico com a sintaxe `<TipoPlaceholder=TipoConcreto>`.

Um ótimo exemplo de uma situação em que essa técnica é útil é com _sobrecarga de operador_, na qual você personaliza o comportamento de um operador (como `+`) em situações específicas.

O Rust não permite que você crie seus próprios operadores ou sobrecarregue operadores arbitrários. Mas você pode sobrecarregar as operações e traits correspondentes listadas em `std::ops` implementando as traits associadas ao operador. Por exemplo, na Listagem 20-15, sobrecarregamos o operador `+` para adicionar duas instâncias de `Point`. Fazemos isso implementando a trait `Add` em uma struct `Point`.

<Listing number="20-15" file-name="src/main.rs" caption="Implementando a trait `Add` para sobrecarregar o operador `+` para instâncias de `Point`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

O método `add` adiciona os valores `x` de duas instâncias de `Point` e os valores `y` de duas instâncias de `Point` para criar um novo `Point`. A trait `Add` tem um tipo associado chamado `Output` que determina o tipo retornado do método `add`.

O tipo genérico padrão neste código está dentro da trait `Add`. Aqui está sua definição:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Este código deve parecer geralmente familiar: uma trait com um método e um tipo associado. A parte nova é `Rhs=Self`: esta sintaxe é chamada de _parâmetros de tipo padrão_. O parâmetro de tipo genérico `Rhs` (abreviação de "right-hand side", lado direito) define o tipo do parâmetro `rhs` no método `add`. Se não especificarmos um tipo concreto para `Rhs` quando implementarmos a trait `Add`, o tipo de `Rhs` será `Self`, que será o tipo no qual estamos implementando `Add`.

Quando implementamos `Add` para `Point`, usamos o padrão para `Rhs` porque queríamos adicionar duas instâncias de `Point`. Vamos ver um exemplo de implementação da trait `Add` onde queremos personalizar o tipo `Rhs` em vez de usar o padrão.

Temos duas structs, `Millimeters` e `Meters`, contendo valores em unidades diferentes. Esse encapsulamento fino de um tipo existente em outra struct é conhecido como o _padrão newtype_, que descrevemos com mais detalhes na seção [“Implementando Traits Externas com o Padrão Newtype”][newtype]<!-- ignore -->. Queremos adicionar valores em milímetros a valores em metros e ter a implementação de `Add` fazendo a conversão corretamente. Podemos implementar `Add` para `Millimeters` com `Meters` como o `Rhs`, como mostrado na Listagem 20-16.

<Listing number="20-16" file-name="src/lib.rs" caption="Implementando a trait `Add` em `Millimeters` para adicionar `Millimeters` e `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

Para adicionar `Millimeters` e `Meters`, especificamos `impl Add<Meters>` para definir o valor do parâmetro de tipo `Rhs` em vez de usar o padrão de `Self`.

Você usará parâmetros de tipo padrão de duas maneiras principais:

1. Para estender um tipo sem quebrar o código existente
2. Para permitir personalização em casos específicos que a maioria dos usuários não precisará

A trait `Add` da biblioteca padrão é um exemplo do segundo propósito: geralmente, você adicionará dois tipos semelhantes, mas a trait `Add` fornece a capacidade de personalizar além disso. Usar um parâmetro de tipo padrão na definição da trait `Add` significa que você não precisa especificar o parâmetro extra na maioria das vezes. Em outras palavras, um pouco de código repetitivo de implementação não é necessário, tornando mais fácil usar a trait.

O primeiro propósito é semelhante ao segundo, mas ao contrário: se você quiser adicionar um parâmetro de tipo a uma trait existente, pode dar a ele um padrão para permitir a extensão da funcionalidade da trait sem quebrar o código de implementação existente.

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### Desambiguando Entre Métodos com Nomes Idênticos

Nada no Rust impede que uma trait tenha um método com o mesmo nome que o método de outra trait, nem o Rust impede que você implemente ambas as traits em um tipo. Também é possível implementar um método diretamente no tipo com o mesmo nome que métodos de traits.

Ao chamar métodos com o mesmo nome, você precisará dizer ao Rust qual deles deseja usar. Considere o código na Listagem 20-17, onde definimos duas traits, `Pilot` e `Wizard`, que têm um método chamado `fly`. Em seguida, implementamos ambas as traits em um tipo `Human` que já tem um método chamado `fly` implementado nele. Cada método `fly` faz algo diferente.

<Listing number="20-17" file-name="src/main.rs" caption="Duas traits são definidas para ter um método `fly` e são implementadas no tipo `Human`, e um método `fly` é implementado em `Human` diretamente.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Quando chamamos `fly` em uma instância de `Human`, o compilador padroniza chamar o método que é implementado diretamente no tipo, como mostrado na Listagem 20-18.

<Listing number="20-18" file-name="src/main.rs" caption="Chamando `fly` em uma instância de `Human`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Executar este código imprimirá `*waving arms furiously*`, mostrando que o Rust chamou o método `fly` implementado em `Human` diretamente.

Para chamar os métodos `fly` da trait `Pilot` ou da trait `Wizard`, precisamos usar uma sintaxe mais explícita para especificar qual método `fly` queremos dizer. A Listagem 20-19 demonstra esta sintaxe.

<Listing number="20-19" file-name="src/main.rs" caption="Especificando qual método `fly` da trait queremos chamar">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Especificar o nome da trait antes do nome do método esclarece ao Rust qual implementação de `fly` queremos chamar. Poderíamos também escrever `Human::fly(&person)`, que é equivalente a `person.fly()` que usamos na Listagem 20-19, mas isso é um pouco mais longo de escrever se não precisarmos desambiguar.

Executar este código imprime o seguinte:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Como o método `fly` recebe um parâmetro `self`, se tivéssemos dois _tipos_ que implementam uma _trait_, o Rust poderia descobrir qual implementação de uma trait usar com base no tipo de `self`.

No entanto, funções associadas que não são métodos não têm um parâmetro `self`. Quando há vários tipos ou traits que definem funções não-método com o mesmo nome de função, o Rust nem sempre sabe qual tipo você quer dizer, a menos que use a sintaxe totalmente qualificada. Por exemplo, na Listagem 20-20, criamos uma trait para um abrigo de animais que quer nomear todos os cães bebês de Spot. Fazemos uma trait `Animal` com uma função associada não-método `baby_name`. A trait `Animal` é implementada para a struct `Dog`, na qual também fornecemos uma função associada não-método `baby_name` diretamente.

<Listing number="20-20" file-name="src/main.rs" caption="Uma trait com uma função associada e um tipo com uma função associada de mesmo nome que também implementa a trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Implementamos o código para nomear todos os filhotes de Spot na função associada `baby_name` que é definida em `Dog`. O tipo `Dog` também implementa a trait `Animal`, que descreve características que todos os animais têm. Cães bebês são chamados de filhotes (puppies), e isso é expresso na implementação da trait `Animal` em `Dog` na função `baby_name` associada à trait `Animal`.

Em `main`, chamamos a função `Dog::baby_name`, que chama a função associada definida em `Dog` diretamente. Este código imprime o seguinte:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Esta saída não é o que queríamos. Queremos chamar a função `baby_name` que faz parte da trait `Animal` que implementamos em `Dog` para que o código imprima `A baby dog is called a puppy`. A técnica de especificar o nome da trait que usamos na Listagem 20-19 não ajuda aqui; se mudarmos `main` para o código na Listagem 20-21, teremos um erro de compilação.

<Listing number="20-21" file-name="src/main.rs" caption="Tentando chamar a função `baby_name` da trait `Animal`, mas o Rust não sabe qual implementação usar">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Como `Animal::baby_name` não tem um parâmetro `self`, e pode haver outros tipos que implementam a trait `Animal`, o Rust não consegue descobrir qual implementação de `Animal::baby_name` queremos. Teremos este erro do compilador:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Para desambiguar e dizer ao Rust que queremos usar a implementação de `Animal` para `Dog` em oposição à implementação de `Animal` para algum outro tipo, precisamos usar a sintaxe totalmente qualificada. A Listagem 20-22 demonstra como usar a sintaxe totalmente qualificada.

<Listing number="20-22" file-name="src/main.rs" caption="Usando sintaxe totalmente qualificada para especificar que queremos chamar a função `baby_name` da trait `Animal` conforme implementada em `Dog`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Estamos fornecendo ao Rust uma anotação de tipo dentro dos colchetes angulares, o que indica que queremos chamar o método `baby_name` da trait `Animal` conforme implementada em `Dog` dizendo que queremos tratar o tipo `Dog` como um `Animal` para esta chamada de função. Este código agora imprimirá o que queremos:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Em geral, a sintaxe totalmente qualificada é definida da seguinte forma:

```rust,ignore
<Tipo as Trait>::function(receptor_se_metodo, prox_arg, ...);
```

Para funções associadas que não são métodos, não haveria um `receptor`: haveria apenas a lista de outros argumentos. Você pode usar a sintaxe totalmente qualificada em todos os lugares em que chama funções ou métodos. No entanto, você tem permissão para omitir qualquer parte desta sintaxe que o Rust possa descobrir a partir de outras informações no programa. Você só precisa usar essa sintaxe mais verbosa em casos onde há múltiplas implementações que usam o mesmo nome e o Rust precisa de ajuda para identificar qual implementação você deseja chamar.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Usando Supertraits

Às vezes, você pode escrever uma definição de trait que depende de outra trait: para um tipo implementar a primeira trait, você quer exigir que esse tipo também implemente a segunda trait. Você faria isso para que sua definição de trait possa fazer uso dos itens associados da segunda trait. A trait na qual sua definição de trait depende é chamada de _supertrait_ de sua trait.

Por exemplo, digamos que queremos fazer uma trait `OutlinePrint` com um método `outline_print` que imprimirá um determinado valor formatado para que seja enquadrado em asteriscos. Ou seja, dada uma struct `Point` que implementa a trait da biblioteca padrão `Display` para resultar em `(x, y)`, quando chamamos `outline_print` em uma instância de `Point` que tem `1` para `x` e `3` para `y`, ele deve imprimir o seguinte:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Na implementação do método `outline_print`, queremos usar a funcionalidade da trait `Display`. Portanto, precisamos especificar que a trait `OutlinePrint` funcionará apenas para tipos que também implementam `Display` e fornecem a funcionalidade que `OutlinePrint` precisa. Podemos fazer isso na definição da trait especificando `OutlinePrint: Display`. Esta técnica é semelhante a adicionar um limite de trait à trait. A Listagem 20-23 mostra uma implementação da trait `OutlinePrint`.

<Listing number="20-23" file-name="src/main.rs" caption="Implementando a trait `OutlinePrint` que requer a funcionalidade de `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

Como especificamos que `OutlinePrint` requer a trait `Display`, podemos usar a função `to_string` que é implementada automaticamente para qualquer tipo que implemente `Display`. Se tentássemos usar `to_string` sem adicionar dois pontos e especificar a trait `Display` após o nome da trait, teríamos um erro dizendo que nenhum método chamado `to_string` foi encontrado para o tipo `&Self` no escopo atual.

Vamos ver o que acontece quando tentamos implementar `OutlinePrint` em um tipo que não implementa `Display`, como a struct `Point`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Recebemos um erro dizendo que `Display` é necessário, mas não implementado:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Para corrigir isso, implementamos `Display` em `Point` e satisfazemos a restrição que `OutlinePrint` exige, assim:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Então, implementar a trait `OutlinePrint` em `Point` compilará com sucesso, e podemos chamar `outline_print` em uma instância de `Point` para exibi-la dentro de um contorno de asteriscos.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### Implementando Traits Externas com o Padrão Newtype

Na seção [“Implementando uma Trait em um Tipo”][implementing-a-trait-on-a-type]<!-- ignore --> no Capítulo 10, mencionamos a regra órfã que afirma que só temos permissão para implementar uma trait em um tipo se a trait ou o tipo, ou ambos, forem locais para o nosso crate. É possível contornar essa restrição usando o padrão newtype, que envolve a criação de um novo tipo em uma struct de tupla. (Cobrimos structs de tupla na seção [“Criando Tipos Diferentes com Structs de Tupla”][tuple-structs]<!-- ignore --> no Capítulo 5.) A struct de tupla terá um campo e será um invólucro fino em torno do tipo para o qual queremos implementar uma trait. Então, o tipo wrapper é local para o nosso crate e podemos implementar a trait no wrapper. _Newtype_ é um termo que se origina da linguagem de programação Haskell. Não há penalidade de desempenho em tempo de execução para usar esse padrão, e o tipo wrapper é omitido em tempo de compilação.

Como exemplo, digamos que queremos implementar `Display` em `Vec<T>`, o que a regra órfã nos impede de fazer diretamente porque a trait `Display` e o tipo `Vec<T>` são definidos fora do nosso crate. Podemos fazer uma struct `Wrapper` que contém uma instância de `Vec<T>`; então, podemos implementar `Display` em `Wrapper` e usar o valor `Vec<T>`, como mostrado na Listagem 20-24.

<Listing number="20-24" file-name="src/main.rs" caption="Criando um tipo `Wrapper` em torno de `Vec<String>` para implementar `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

A implementação de `Display` usa `self.0` para acessar o `Vec<T>` interno porque `Wrapper` é uma struct de tupla e `Vec<T>` é o item no índice 0 na tupla. Então, podemos usar a funcionalidade da trait `Display` em `Wrapper`.

A desvantagem de usar essa técnica é que `Wrapper` é um novo tipo, então ele não tem os métodos do valor que está segurando. Teríamos que implementar todos os métodos de `Vec<T>` diretamente em `Wrapper` de modo que os métodos deleguem para `self.0`, o que nos permitiria tratar `Wrapper` exatamente como um `Vec<T>`. Se quiséssemos que o novo tipo tivesse todos os métodos que o tipo interno tem, implementar a trait `Deref` no `Wrapper` para retornar o tipo interno seria uma solução (discutimos a implementação da trait `Deref` na seção [“Tratando Ponteiros Inteligentes como Referências Regulares”][smart-pointer-deref]<!-- ignore --> no Capítulo 15). Se não quiséssemos que o tipo `Wrapper` tivesse todos os métodos do tipo interno — por exemplo, para restringir o comportamento do tipo `Wrapper` — teríamos que implementar apenas os métodos que queremos manualmente.

Este padrão newtype também é útil mesmo quando traits não estão envolvidas. Vamos mudar o foco e olhar para algumas maneiras avançadas de interagir com o sistema de tipos do Rust.

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
