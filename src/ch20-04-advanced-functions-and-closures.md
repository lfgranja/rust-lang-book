# Funções e Closures Avançadas

Esta seção explora algumas funcionalidades avançadas relacionadas a funções e closures, incluindo ponteiros de função e retorno de closures.

## Ponteiros de Função

Falamos sobre como passar closures para funções; você também pode passar funções regulares para funções! Essa técnica é útil quando você deseja passar uma função que já definiu em vez de definir uma nova closure. Funções coagem para o tipo `fn` (com um *f* minúsculo), não deve ser confundido com o trait de closure `Fn`. O tipo `fn` é chamado de *ponteiro de função* (function pointer). Passar funções com ponteiros de função permitirá que você use funções como argumentos para outras funções.

A sintaxe para especificar que um parâmetro é um ponteiro de função é semelhante à das closures, como mostrado no Listagem 19-27, onde definimos uma função `add_one` que adiciona 1 ao seu parâmetro. A função `do_twice` recebe dois parâmetros: um ponteiro de função para qualquer função que receba um parâmetro `i32` e retorne um `i32`, e um valor `i32`. A função `do_twice` chama a função `f` duas vezes, passando a ela o valor `arg`, depois soma os dois resultados da chamada de função. A função `main` chama `do_twice` com os argumentos `add_one` e `5`.

Listagem 19-27: Usando o tipo `fn` para aceitar um ponteiro de função como argumento

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let resposta = do_twice(add_one, 5);

    println!("A resposta é: {}", resposta);
}
```

Este código imprime `A resposta é: 12`. Especificamos que o parâmetro `f` em `do_twice` é um `fn` que recebe um parâmetro do tipo `i32` e retorna um `i32`. Podemos então chamar `f` no corpo de `do_twice`. Em `main`, podemos passar o nome da função `add_one` como o primeiro argumento para `do_twice`.

Ao contrário das closures, `fn` é um tipo em vez de um trait, então especificamos `fn` como o tipo de parâmetro diretamente, em vez de declarar um parâmetro de tipo genérico com um dos traits `Fn` como um limite de trait.

Ponteiros de função implementam todos os três traits de closure (`Fn`, `FnMut` e `FnOnce`), o que significa que você sempre pode passar um ponteiro de função como argumento para uma função que espera uma closure. É melhor escrever funções usando um tipo genérico e um dos traits de closure para que suas funções possam aceitar funções ou closures.

Dito isso, um exemplo de onde você gostaria de aceitar apenas `fn` e não closures é ao interagir com código externo que não tem closures: funções C podem aceitar funções como argumentos, mas C não tem closures.

Como exemplo de onde você poderia usar uma closure definida inline ou uma função nomeada, vamos ver um uso do método `map` fornecido pelo trait `Iterator` na biblioteca padrão. Para usar o método `map` para transformar um vetor de números em um vetor de strings, poderíamos usar uma closure, como no Listagem 19-28.

Listagem 19-28: Usando uma closure com o método `map` para converter números em strings

```rust
    let lista_de_numeros = vec![1, 2, 3];
    let lista_de_strings: Vec<String> = lista_de_numeros.iter().map(|i| i.to_string()).collect();
```

Ou poderíamos nomear uma função como o argumento para `map` em vez da closure. O Listagem 19-29 mostra como isso ficaria.

Listagem 19-29: Usando a função `String::to_string` com o método `map` para converter números em strings

```rust
    let lista_de_numeros = vec![1, 2, 3];
    let lista_de_strings: Vec<String> = lista_de_numeros.iter().map(ToString::to_string).collect();
```

Observe que devemos usar a sintaxe totalmente qualificada sobre a qual falamos na seção "Traits Avançados" anteriormente, porque há várias funções disponíveis chamadas `to_string`. Aqui, estamos usando a função `to_string` definida no trait `ToString`, que a biblioteca padrão implementou para qualquer tipo que implementa `Display`.

Lembre-se da seção "Valores Enum" no Capítulo 6 que o nome de cada variante de enum que definimos também se torna uma função inicializadora. Podemos usar essas funções inicializadoras como ponteiros de função que implementam os traits de closure, o que significa que podemos especificar as funções inicializadoras como argumentos para métodos que recebem closures, como visto no Listagem 19-30.

Listagem 19-30: Usando um inicializador de enum com o método `map` para criar uma instância `Status` a partir de números

```rust
    enum Status {
        Valor(u32),
        Parar,
    }

    let lista_de_status: Vec<Status> = (0u32..20).map(Status::Valor).collect();
```

Aqui, criamos instâncias de `Status::Valor` usando cada valor `u32` no intervalo em que `map` é chamado, usando a função inicializadora de `Status::Valor`. Algumas pessoas preferem este estilo e outras preferem usar closures. Eles compilam para o mesmo código, então use o estilo que for mais claro para você.

## Retornando Closures

[[Closures]] são representadas por traits, o que significa que você não pode retornar closures diretamente. Na maioria dos casos em que você pode querer retornar um trait, você pode usar o tipo concreto que implementa o trait como o valor de retorno da função. No entanto, você geralmente não pode fazer isso com closures porque elas não têm um tipo concreto que seja retornável; você não tem permissão para usar o ponteiro de função `fn` como um tipo de retorno se a closure capturar quaisquer valores de seu escopo, por exemplo.

Em vez disso, você normalmente usará a sintaxe `impl Trait` que aprendemos no Capítulo 10. Você pode retornar qualquer tipo de função, usando `Fn`, `FnOnce` e `FnMut`. Por exemplo, o código a seguir funcionará muito bem:

```rust
fn retorna_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

No entanto, como observamos na seção "Inferindo e Anotando Tipos de Closure" no Capítulo 13, cada closure também é seu próprio tipo distinto. Se você precisar trabalhar com múltiplas funções que têm a mesma assinatura, mas implementações diferentes, você precisará usar um trait object para elas. Considere o que acontece se você escrever código como o mostrado no Listagem 19-31.

Listagem 19-31: Criando um `Vec<T>` de closures definidas por funções que retornam tipos `impl Fn`

```rust,ignore,does_not_compile
fn retorna_closure(a: i32) -> impl Fn(i32) -> i32 {
    if a > 0 {
        move |b| a + b
    } else {
        move |b| a - b
    }
}
```

Aqui temos uma função `retorna_closure` que retorna uma closure que adiciona ou subtrai `a` de seu argumento `b`, dependendo se `a` é maior que 0. Observe que as closures que ela retorna são diferentes, embora implementem o mesmo trait `Fn(i32) -> i32`. Se tentarmos compilar isso, Rust nos avisa que não funcionará:

```console
error[E0308]: `if` and `else` have incompatible types
 --> src/lib.rs:5:9
  |
2 | /     if a > 0 {
3 | |         move |b| a + b
  | |         -------------- expected because of this
4 | |     } else {
5 | |         move |b| a - b
  | |         ^^^^^^^^^^^^^^ expected closure, found a different closure
6 | |     }
  | |_____- `if` and `else` have incompatible types
  |
  = note: expected type `[closure@src/lib.rs:3:9: 3:23 a:_]`
             found type `[closure@src/lib.rs:5:9: 5:23 a:_]`
  = note: no two closures, even if identical, have the same type
  = help: consider boxing your closure and/or using it as a trait object
```

A mensagem de erro nos diz que `if` e `else` têm tipos incompatíveis e aponta que não há duas closures, mesmo que idênticas, que tenham o mesmo tipo. Mesmo que retornem `impl Fn(i32) -> i32`, os tipos opacos que Rust gera para cada uma são distintos. (Isso é semelhante a como Rust produz tipos concretos diferentes para blocos assíncronos distintos, mesmo quando têm o mesmo tipo de saída, como vimos na seção "O Tipo `Pin` e o Trait `Unpin`" no Capítulo 17.) Vimos uma solução para este problema algumas vezes agora: podemos usar um trait object, como no Listagem 19-32.

Listagem 19-32: Criando um `Vec<T>` de closures definidas por funções que retornam `Box<dyn Fn>` para que tenham o mesmo tipo

```rust
fn retorna_closure(a: i32) -> Box<dyn Fn(i32) -> i32> {
    if a > 0 {
        Box::new(move |b| a + b)
    } else {
        Box::new(move |b| a - b)
    }
}
```

Este código compilará perfeitamente. Para saber mais sobre trait objects, consulte a seção "Usando Objetos de Trait que Permitem Valores de Tipos Diferentes" no Capítulo 17.

A seguir, vamos ver macros!
