# Apêndice A: Palavras-chave

A lista a seguir contém palavras-chave que são reservadas para uso atual ou futuro pela linguagem Rust. Como tal, elas não podem ser usadas como identificadores (exceto como identificadores brutos, conforme discutiremos na seção "Identificadores Brutos"), incluindo nomes de funções, variáveis, parâmetros, campos de struct, módulos, crates, constantes e macros.

## Palavras-chave Atualmente em Uso

As seguintes palavras-chave têm atualmente a funcionalidade descrita.

* `as` - realizar conversão primitiva, desambiguar o trait específico contendo um item ou renomear itens em instruções `use` e `extern crate`
* `async` - retornar um `Future` em vez de bloquear a thread atual
* `await` - suspender a execução até que o resultado de um `Future` esteja pronto
* `break` - sair de um loop imediatamente
* `const` - definir itens constantes ou ponteiros brutos constantes
* `continue` - continuar para a próxima iteração do loop
* `crate` - vincular uma crate externa ou uma variável de macro representando a crate em que a macro é definida
* `dyn` - despacho dinâmico para um trait object
* `else` - fallback para construções de fluxo de controle `if` e `if let`
* `enum` - definir uma enumeração
* `extern` - vincular uma crate externa, função ou variável
* `false` - literal booleano falso
* `fn` - definir uma função ou o tipo de ponteiro de função
* `for` - iterar sobre itens de um iterador, implementar um trait ou especificar um tempo de vida de classificação superior (higher-ranked lifetime)
* `if` - ramificar com base no resultado de uma expressão condicional
* `impl` - implementar funcionalidade inerente ou de trait
* `in` - parte da sintaxe do loop `for`
* `let` - vincular uma variável
* `loop` - iterar incondicionalmente
* `match` - corresponder um valor a padrões
* `mod` - definir um módulo
* `move` - fazer uma closure tomar posse de todas as suas capturas
* `mut` - denotar mutabilidade em referências, ponteiros brutos ou ligações de padrões
* `pub` - denotar visibilidade pública em campos de struct, blocos `impl` ou módulos
* `ref` - vincular por referência
* `return` - retornar de uma função
* `Self` - um apelido de tipo para o tipo que estamos definindo ou implementando
* `self` - sujeito do método ou módulo atual
* `static` - variável global ou tempo de vida que dura toda a execução do programa
* `struct` - definir uma estrutura
* `super` - módulo pai do módulo atual
* `trait` - definir um trait
* `true` - literal booleano verdadeiro
* `type` - definir um apelido de tipo ou tipo associado
* `union` - definir uma união e é apenas uma palavra-chave quando usada em uma declaração de união
* `unsafe` - denotar código inseguro, funções, traits ou implementações
* `use` - trazer símbolos para o escopo
* `where` - denotar cláusulas que restringem um tipo
* `while` - iterar condicionalmente com base no resultado de uma expressão

## Palavras-chave Reservadas para Uso Futuro

As seguintes palavras-chave ainda não têm nenhuma funcionalidade, mas são reservadas por Rust para uso futuro potencial.

* `abstract`
* `become`
* `box`
* `do`
* `final`
* `macro`
* `override`
* `priv`
* `try`
* `typeof`
* `unsized`
* `virtual`
* `yield`

## Identificadores Brutos

*Identificadores brutos* (raw identifiers) permitem que você use palavras-chave onde elas normalmente não seriam permitidas. Você usa um identificador bruto prefixando uma palavra-chave com `r#`.

Por exemplo, `match` é uma palavra-chave. Se você tentar compilar a seguinte função que usa `match` como seu nome:

```rust
fn match() {
    let needle = 42;
    let haystack = [1, 2, 3];
    for item in &haystack {
        if *item == needle {
            println!("{}", item);
        }
    }
}
```

você receberá este erro:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match() {
  |    ^^^^^ expected identifier, found keyword
```

A prova é que você não pode usar `match` como um identificador de função. Para usar `match` como um nome de função, você precisa usar a sintaxe de identificador bruto, assim:

```rust
fn r#match() {
    let needle = 42;
    let haystack = [1, 2, 3];
    for item in &haystack {
        if *item == needle {
            println!("{}", item);
        }
    }
}

fn main() {
    r#match();
}
```

Este código compilará sem erros. Observe o prefixo `r#` na definição da função, bem como onde a função é chamada em `main`.

Identificadores brutos permitem que você use qualquer palavra que escolher como um identificador, mesmo que essa palavra seja uma palavra-chave reservada. Além disso, identificadores brutos permitem que você use bibliotecas escritas em uma edição diferente de Rust do que a sua crate usa. Por exemplo, `try` não é uma palavra-chave na edição de 2015, mas é na edição de 2018. Se você depende de uma biblioteca escrita usando a edição de 2015 e ela tem uma função `try`, você precisará usar a sintaxe de identificador bruto, `r#try`, para chamar essa função do seu código da edição de 2018. Consulte o Apêndice E para obter mais informações sobre edições.
