# Tipos Avançados

O sistema de tipos de Rust tem algumas funcionalidades que mencionamos até agora, mas ainda não discutimos. Começaremos discutindo o padrão newtype em geral, enquanto examinamos por que newtypes são úteis como tipos. Em seguida, passaremos para apelidos de tipo (type aliases), uma funcionalidade semelhante aos newtypes, mas com semânticas ligeiramente diferentes. Também discutiremos o tipo `!` e tipos de tamanho dinâmico.

## Usando o Padrão Newtype para Segurança de Tipo e Abstração

Esta seção pressupõe que você leu a seção anterior "Usando o Padrão Newtype para Implementar Traits Externos em Tipos Externos". O padrão newtype também é útil para tarefas além das que discutimos até agora, incluindo impor estaticamente que os valores nunca sejam confundidos e indicar as unidades de um valor. Você viu um exemplo de uso de newtypes para indicar unidades no Listagem 19-15: lembre-se de que as structs `Milimetros` e `Metros` envolviam valores `u32` em um newtype. Se escrevêssemos uma função com um parâmetro do tipo `Milimetros`, não conseguiríamos compilar um programa que acidentalmente tentasse chamar essa função com um valor do tipo `Metros` ou um `u32` simples.

Também podemos usar o padrão newtype para abstrair alguns detalhes de implementação de um tipo: o novo tipo pode expor uma API pública que é diferente da API do tipo interno privado.

Newtypes também podem ocultar a implementação interna. Por exemplo, poderíamos fornecer um tipo `Pessoas` para envolver um `HashMap<i32, String>` que armazena o ID de uma pessoa associado ao seu nome. O código usando `Pessoas` interagiria apenas com a API pública que fornecemos, como um método para adicionar uma string de nome à coleção `Pessoas`; esse código não precisaria saber que atribuímos um ID `i32` aos nomes internamente. O padrão newtype é uma maneira leve de alcançar o encapsulamento para ocultar detalhes de implementação, o que discutimos na seção "Encapsulamento que Oculta Detalhes de Implementação" no Capítulo 17.

## Criando Sinônimos de Tipo com Apelidos de Tipo

Rust fornece a capacidade de declarar um *apelido de tipo* (type alias) para dar a um tipo existente outro nome. Para isso, usamos a palavra-chave `type`. Por exemplo, podemos criar o apelido `Quilometros` para `i32` assim:

```rust
    type Quilometros = i32;

    let x: i32 = 5;
    let y: Quilometros = 5;

    println!("x + y = {}", x + y);
```

Agora, o apelido `Quilometros` é um *sinônimo* para `i32`; ao contrário dos tipos `Milimetros` e `Metros` que criamos no Listagem 19-15, `Quilometros` não é um tipo novo e separado. Valores que têm o tipo `Quilometros` serão tratados da mesma forma que valores do tipo `i32`:

```rust
    type Quilometros = i32;

    let x: i32 = 5;
    let y: Quilometros = 5;

    println!("x + y = {}", x + y);
```

Como `Quilometros` e `i32` são o mesmo tipo, podemos adicionar valores de ambos os tipos e passar valores `Quilometros` para funções que recebem parâmetros `i32`. No entanto, usando este método, não obtemos os benefícios de verificação de tipo que obtemos do padrão newtype discutido anteriormente. Em outras palavras, se misturarmos valores `Quilometros` e `i32` em algum lugar, o compilador não nos dará um erro.

O principal caso de uso para sinônimos de tipo é reduzir a repetição. Por exemplo, podemos ter um tipo longo como este:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Escrever esse tipo longo em assinaturas de função e como anotações de tipo em todo o código pode ser cansativo e propenso a erros. Imagine ter um projeto cheio de código como o do Listagem 19-24.

Listagem 19-24: Usando um tipo longo em muitos lugares

```rust
    let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("oi"));

    fn leva_tipo_longo(f: Box<dyn Fn() + Send + 'static>) {
        // --trecho omitido--
    }

    fn retorna_tipo_longo() -> Box<dyn Fn() + Send + 'static> {
        // --trecho omitido--
        Box::new(|| ())
    }
```

Um apelido de tipo torna este código mais gerenciável reduzindo a repetição. No Listagem 19-25, introduzimos um apelido chamado `Thunk` para o tipo verboso e podemos substituir todos os usos do tipo pelo apelido mais curto `Thunk`.

Listagem 19-25: Introduzindo um apelido de tipo `Thunk` para reduzir a repetição

```rust
    type Thunk = Box<dyn Fn() + Send + 'static>;

    let f: Thunk = Box::new(|| println!("oi"));

    fn leva_tipo_longo(f: Thunk) {
        // --trecho omitido--
    }

    fn retorna_tipo_longo() -> Thunk {
        // --trecho omitido--
        Box::new(|| ())
    }
```

Este código é muito mais fácil de ler e escrever! Escolher um nome significativo para um apelido de tipo também pode ajudar a comunicar sua intenção (*thunk* é uma palavra para código a ser avaliado em um momento posterior, então é um nome apropriado para uma closure que é armazenada).

Apelidos de tipo também são comumente usados com o tipo `Result<T, E>` para reduzir a repetição. Considere o módulo `std::io` na biblioteca padrão. Operações de E/S frequentemente retornam um `Result<T, E>` para lidar com situações em que as operações falham. Esta biblioteca tem uma struct `std::io::Error` que representa todos os possíveis erros de E/S. Muitas das funções em `std::io` retornarão `Result<T, E>` onde o `E` é `std::io::Error`, como estas funções no trait `Write`:

```rust
use std::fmt;
use std::io::Error;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

O `Result<..., Error>` é repetido muito. Como tal, `std::io` tem esta declaração de apelido de tipo:

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
```

Como esta declaração está no módulo `std::io`, podemos usar o apelido totalmente qualificado `std::io::Result<T>`; ou seja, um `Result<T, E>` com o `E` preenchido como `std::io::Error`. As assinaturas de função do trait `Write` acabam parecendo com isso:

```rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
```

O apelido de tipo ajuda de duas maneiras: torna o código mais fácil de escrever *e* nos dá uma interface consistente em todo o `std::io`. Como é um apelido, é apenas outro `Result<T, E>`, o que significa que podemos usar quaisquer métodos que funcionem em `Result<T, E>` com ele, bem como sintaxe especial como o operador `?`.

## O Tipo Never que Nunca Retorna

Rust tem um tipo especial chamado `!` que é conhecido na gíria da teoria dos tipos como o *tipo vazio* porque não tem valores. Preferimos chamá-lo de *tipo never* (never type) porque ele fica no lugar do tipo de retorno quando uma função nunca retornará. Aqui está um exemplo:

```rust
fn bar() -> ! {
    // --trecho omitido--
    panic!();
}
```

Este código é lido como "a função `bar` retorna never". Funções que retornam never são chamadas de *funções divergentes*. Não podemos criar valores do tipo `!`, então `bar` nunca pode retornar.

Mas qual é a utilidade de um tipo para o qual você nunca pode criar valores? Lembre-se do código do Listagem 2-5, parte do jogo de adivinhação de números; reproduzimos um pouco dele aqui no Listagem 19-26.

Listagem 19-26: Um `match` com um braço que termina em `continue`

```rust
        let palpite: u32 = match palpite.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
```

Na época, pulamos alguns detalhes neste código. Na seção "A Construção de Controle de Fluxo `match`" no Capítulo 6, discutimos que os braços de `match` devem todos retornar o mesmo tipo. Então, por exemplo, o seguinte código não funciona:

```rust,ignore,does_not_compile
    let palpite = match palpite.trim().parse() {
        Ok(_) => 5,
        Err(_) => "olá",
    };
```

O tipo de `palpite` neste código teria que ser um inteiro *e* uma string, e Rust exige que `palpite` tenha apenas um tipo. Então, o que `continue` retorna? Como fomos autorizados a retornar um `u32` de um braço e ter outro braço que termina com `continue` no Listagem 19-26?

Como você pode ter adivinhado, `continue` tem um valor `!`. Ou seja, quando Rust calcula o tipo de `palpite`, ele olha para ambos os braços de correspondência, o primeiro com um valor de `u32` e o segundo com um valor `!`. Como `!` nunca pode ter um valor, Rust decide que o tipo de `palpite` é `u32`.

A maneira formal de descrever esse comportamento é que expressões do tipo `!` podem ser coagidas a qualquer outro tipo. Temos permissão para terminar este braço de `match` com `continue` porque `continue` não retorna um valor; em vez disso, move o controle de volta para o topo do loop, então no caso `Err`, nunca atribuímos um valor a `palpite`.

O tipo never é útil com a macro `panic!` também. Lembre-se da função `unwrap` que chamamos em valores `Option<T>` para produzir um valor ou entrar em pânico com esta definição:

```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("chamou `Option::unwrap()` num valor `None`"),
        }
    }
}
```

Neste código, a mesma coisa acontece que no `match` no Listagem 19-26: Rust vê que `val` tem o tipo `T` e `panic!` tem o tipo `!`, então o resultado da expressão `match` geral é `T`. Este código funciona porque `panic!` não produz um valor; ele termina o programa. No caso `None`, não estaremos retornando um valor de `unwrap`, então este código é válido.

Uma expressão final que tem o tipo `!` é um `loop`:

```rust
    print!("para sempre ");

    loop {
        print!("e sempre ");
    }
```

Aqui, o loop nunca termina, então `!` é o valor da expressão. No entanto, isso não seria verdade se incluíssemos um `break`, porque o loop terminaria quando chegasse ao `break`.

## Tipos de Tamanho Dinâmico e o Trait `Sized`

Rust precisa saber certos detalhes sobre seus tipos, como quanto espaço alocar para um valor de um determinado tipo. Isso deixa um canto de seu sistema de tipos um pouco confuso no início: o conceito de *tipos de tamanho dinâmico* (dynamically sized types). Às vezes referidos como *DSTs* ou *unsized types*, esses tipos nos permitem escrever código usando valores cujo tamanho podemos saber apenas em tempo de execução.

Vamos cavar nos detalhes de um tipo de tamanho dinâmico chamado `str`, que temos usado ao longo do livro. Isso mesmo, não `&str`, mas `str` por si só, é um DST. Em muitos casos, como ao armazenar texto inserido por um usuário, não podemos saber quão longa é a string até o tempo de execução. Isso significa que não podemos criar uma variável do tipo `str`, nem podemos receber um argumento do tipo `str`. Considere o seguinte código, que não funciona:

```rust,ignore,does_not_compile
    let s1: str = "Olá aí!";
    let s2: str = "Como vai?";
```

Rust precisa saber quanta memória alocar para qualquer valor de um determinado tipo, e todos os valores de um tipo devem usar a mesma quantidade de memória. Se Rust nos permitisse escrever este código, esses dois valores `str` precisariam ocupar a mesma quantidade de espaço. Mas eles têm comprimentos diferentes: `s1` precisa de 12 bytes de armazenamento e `s2` precisa de 15. É por isso que não é possível criar uma variável contendo um tipo de tamanho dinâmico.

Então, o que fazemos? Neste caso, você já sabe a resposta: tornamos o tipo de `s1` e `s2` uma fatia de string (`&str`) em vez de `str`. Lembre-se da seção "Fatias de String" no Capítulo 4 que a estrutura de dados da fatia armazena apenas a posição inicial e o comprimento da fatia. Então, embora `&T` seja um único valor que armazena o endereço de memória de onde o `T` está localizado, uma fatia de string são *dois* valores: o endereço do `str` e seu comprimento. Como tal, podemos saber o tamanho de um valor de fatia de string em tempo de compilação: é duas vezes o comprimento de um `usize`. Ou seja, sempre sabemos o tamanho de uma fatia de string, não importa quão longa seja a string a que ela se refere. Em geral, é assim que os tipos de tamanho dinâmico são usados em Rust: eles têm um bit extra de metadados que armazena o tamanho da informação dinâmica. A regra de ouro dos tipos de tamanho dinâmico é que devemos sempre colocar valores de tipos de tamanho dinâmico atrás de um ponteiro de algum tipo.

Podemos combinar `str` com todos os tipos de ponteiros: por exemplo, `Box<str>` ou `Rc<str>`. De fato, você já viu isso antes, mas com um tipo de tamanho dinâmico diferente: traits. Cada trait é um tipo de tamanho dinâmico ao qual podemos nos referir usando o nome do trait. Na seção "Usando Objetos de Trait que Permitem Valores de Tipos Diferentes" no Capítulo 17, mencionamos que para usar traits como trait objects, devemos colocá-los atrás de um ponteiro, como `&dyn Trait` ou `Box<dyn Trait>` (`Rc<dyn Trait>` funcionaria também).

Para trabalhar com DSTs, Rust fornece o trait `Sized` para determinar se o tamanho de um tipo é conhecido em tempo de compilação ou não. Este trait é implementado automaticamente para tudo cujo tamanho é conhecido em tempo de compilação. Além disso, Rust implicitamente adiciona um limite em `Sized` a cada função genérica. Ou seja, uma definição de função genérica como esta:

```rust
fn generic<T>(t: T) {
    // --trecho omitido--
}
```

é realmente tratada como se tivéssemos escrito isto:

```rust
fn generic<T: Sized>(t: T) {
    // --trecho omitido--
}
```

Por padrão, funções genéricas funcionarão apenas em tipos que têm um tamanho conhecido em tempo de compilação. No entanto, você pode usar a seguinte sintaxe especial para relaxar esta restrição:

```rust
fn generic<T: ?Sized>(t: &T) {
    // --trecho omitido--
}
```

Um limite de trait em `?Sized` significa "`T` pode ou não ser `Sized`", e esta notação substitui o padrão de que tipos genéricos devem ter um tamanho conhecido em tempo de compilação. A sintaxe `?Trait` com este significado está disponível apenas para `Sized`, não para quaisquer outros traits.

Observe também que mudamos o tipo do parâmetro `t` de `T` para `&T`. Como o tipo pode não ser `Sized`, precisamos usá-lo atrás de algum tipo de ponteiro. Neste caso, escolhemos uma referência.

A seguir, falaremos sobre funções e closures!
