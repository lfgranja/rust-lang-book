# Refutabilidade: Se um Padrão Pode Falhar em Casar

Padrões vêm em duas formas: refutáveis e irrefutáveis. Padrões que corresponderão a qualquer valor possível passado são *irrefutáveis*. Um exemplo seria `x` na declaração `let x = 5;` porque `x` corresponde a qualquer coisa e, portanto, não pode falhar em corresponder. Padrões que podem falhar em corresponder a algum valor possível são *refutáveis*. Um exemplo seria `Some(x)` na expressão `if let Some(x) = um_valor` porque se o valor na variável `um_valor` for `None` em vez de `Some`, o padrão `Some(x)` não corresponderá.

Parâmetros de função, declarações `let` e loops `for` só podem aceitar padrões irrefutáveis porque o programa não pode fazer nada significativo quando os valores não correspondem. As expressões `if let` e `while let` e a declaração `let...else` aceitam padrões refutáveis e irrefutáveis, mas o compilador adverte contra padrões irrefutáveis porque, por definição, eles destinam-se a lidar com possíveis falhas: a funcionalidade de uma condicional está em sua capacidade de executar de forma diferente dependendo do sucesso ou falha.

Em geral, você não deve se preocupar com a distinção entre padrões refutáveis e irrefutáveis; no entanto, você precisa estar familiarizado com o conceito de refutabilidade para que possa responder quando vê-lo em uma mensagem de erro. Nesses casos, você precisará alterar o padrão ou a construção que está usando com o padrão, dependendo do comportamento pretendido do código.

Vamos ver um exemplo do que acontece quando tentamos usar um padrão refutável onde Rust exige um padrão irrefutável e vice-versa. O Listagem 19-8 mostra uma declaração `let`, mas para o padrão, especificamos `Some(x)`, um padrão refutável. Como você pode esperar, este código não será compilado.

Listagem 19-8: Tentando usar um padrão refutável com `let`

```rust,ignore,does_not_compile
let Some(x) = some_option_value;
```

Se `some_option_value` fosse um valor `None`, ele falharia em corresponder ao padrão `Some(x)`, o que significa que o padrão é refutável. No entanto, a declaração `let` só pode aceitar um padrão irrefutável porque não há nada válido que o código possa fazer com um valor `None`. Em tempo de compilação, Rust reclamará que tentamos usar um padrão refutável onde um padrão irrefutável é necessário:

```console
error[E0005]: refutable pattern in local binding: `None` not covered
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
```

Como não cobrimos (e não poderíamos cobrir!) todos os valores válidos com o padrão `Some(x)`, Rust produz corretamente um erro de compilador.

Se tivermos um padrão refutável onde um padrão irrefutável é necessário, podemos consertá-lo alterando o código que usa o padrão: em vez de usar `let`, podemos usar `let...else`. Então, se o padrão não corresponder, o código nas chaves lidará com o valor. O Listagem 19-9 mostra como corrigir o código no Listagem 19-8.

Listagem 19-9: Usando `let...else` e um bloco com padrões refutáveis em vez de `let`

```rust
if let Some(x) = some_option_value {
    println!("{}", x);
}
```

Demos uma saída ao código! Este código é perfeitamente válido, embora signifique que não podemos usar um padrão irrefutável sem receber um aviso. Se dermos a `if let` um padrão que sempre corresponderá, como `x`, como mostrado no Listagem 19-10, o compilador dará um aviso.

Listagem 19-10: Tentando usar um padrão irrefutável com `if let`

```rust
if let x = 5 {
    println!("{}", x);
};
```

Rust reclama que não faz sentido usar `if let` com um padrão irrefutável:

```console
warning: irrefutable if-let pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^^ irrefutable pattern
  |
  = note: `#[warn(irrefutable_let_patterns)]` on by default
```

Por esse motivo, os braços de correspondência devem usar padrões refutáveis, exceto pelo último braço, que deve corresponder a quaisquer valores restantes com um padrão irrefutável. Rust nos permite usar um padrão irrefutável em um `match` com apenas um braço, mas essa sintaxe não é particularmente útil e poderia ser substituída por uma declaração `let` mais simples.

Agora que você sabe onde usar padrões e a diferença entre padrões refutáveis e irrefutáveis, vamos cobrir toda a sintaxe que podemos usar para criar padrões.
