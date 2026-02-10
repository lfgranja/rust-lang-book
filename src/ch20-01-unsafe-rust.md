# Rust Inseguro

Todo o código que discutimos até agora teve as garantias de segurança de memória de Rust impostas em tempo de compilação. No entanto, Rust tem uma segunda linguagem escondida dentro dele que não impõe essas garantias de segurança de memória: chama-se *Rust inseguro* (unsafe Rust) e funciona exatamente como o Rust regular, mas nos dá superpoderes extras.

O Rust inseguro existe porque, por natureza, a análise estática é conservadora. Quando o compilador tenta determinar se o código mantém ou não as garantias, é melhor para ele rejeitar alguns programas válidos do que aceitar alguns programas inválidos. Embora o código *possa* estar correto, se o compilador Rust não tiver informações suficientes para ter certeza, ele rejeitará o código. Nesses casos, você pode usar código inseguro para dizer ao compilador: "Confie em mim, eu sei o que estou fazendo". Esteja avisado, no entanto, que você usa Rust inseguro por sua própria conta e risco: se você usar código inseguro incorretamente, problemas podem ocorrer devido à insegurança de memória, como desreferência de ponteiro nulo.

Outra razão pela qual Rust tem um alter ego inseguro é que o hardware do computador subjacente é inerentemente inseguro. Se Rust não permitisse que você fizesse operações inseguras, você não poderia realizar certas tarefas. Rust precisa permitir que você faça programação de sistemas de baixo nível, como interagir diretamente com o sistema operacional ou até mesmo escrever seu próprio sistema operacional. Trabalhar com programação de sistemas de baixo nível é um dos objetivos da linguagem. Vamos explorar o que podemos fazer com Rust inseguro e como fazê-lo.

## Realizando Superpoderes Inseguros

Para mudar para Rust inseguro, use a palavra-chave `unsafe` e inicie um novo bloco que contém o código inseguro. Você pode realizar cinco ações em Rust inseguro que não pode em Rust seguro, que chamamos de *superpoderes inseguros*. Esses superpoderes incluem a capacidade de:

1. Desreferenciar um ponteiro bruto (raw pointer).
2. Chamar uma função ou método inseguro.
3. Acessar ou modificar uma variável estática mutável.
4. Implementar um trait inseguro.
5. Acessar campos de `union`s.

É importante entender que `unsafe` não desliga o verificador de empréstimo (borrow checker) ou desabilita qualquer outra verificação de segurança de Rust: se você usar uma referência em código inseguro, ela ainda será verificada. A palavra-chave `unsafe` apenas lhe dá acesso a essas cinco funcionalidades que não são verificadas pelo compilador quanto à segurança de memória. Você ainda obterá algum grau de segurança dentro de um bloco inseguro.

Além disso, `unsafe` não significa que o código dentro do bloco é necessariamente perigoso ou que definitivamente terá problemas de segurança de memória: a intenção é que, como programador, você garanta que o código dentro de um bloco `unsafe` acessará a memória de maneira válida.

As pessoas são falíveis e erros acontecerão, mas ao exigir que essas cinco operações inseguras estejam dentro de blocos anotados com `unsafe`, você saberá que quaisquer erros relacionados à segurança de memória devem estar dentro de um bloco `unsafe`. Mantenha os blocos `unsafe` pequenos; você ficará grato mais tarde quando investigar bugs de memória.

Para isolar o código inseguro o máximo possível, é melhor encerrar tal código dentro de uma abstração segura e fornecer uma API segura, o que discutiremos mais tarde no capítulo quando examinarmos funções e métodos inseguros. Partes da biblioteca padrão são implementadas como abstrações seguras sobre código inseguro que foi auditado. Envolver código inseguro em uma abstração segura impede que usos de `unsafe` vazem para todos os lugares que você ou seus usuários possam querer usar a funcionalidade implementada com código `unsafe`, porque usar uma abstração segura é seguro.

Vamos olhar para cada um dos cinco superpoderes inseguros, um por um. Também veremos algumas abstrações que fornecem uma interface segura para código inseguro.

## Desreferenciando um Ponteiro Bruto

No Capítulo 4, na seção "Referências Pendentes", mencionamos que o compilador garante que as referências são sempre válidas. O Rust inseguro tem dois novos tipos chamados *ponteiros brutos* (raw pointers) que são semelhantes às referências. Como com referências, ponteiros brutos podem ser imutáveis ou mutáveis e são escritos como `*const T` e `*mut T`, respectivamente. O asterisco não é o operador de desreferência; faz parte do nome do tipo. No contexto de ponteiros brutos, *imutável* significa que o ponteiro não pode ser atribuído diretamente após ser desreferenciado.

Diferente de referências e ponteiros inteligentes, ponteiros brutos:

- Têm permissão para ignorar as regras de empréstimo tendo ponteiros imutáveis e mutáveis ou múltiplos ponteiros mutáveis para o mesmo local
- Não são garantidos de apontar para memória válida
- Têm permissão para ser nulos
- Não implementam nenhuma limpeza automática

Ao optar por não ter Rust impondo essas garantias, você pode abrir mão da segurança garantida em troca de maior desempenho ou da capacidade de interagir com outra linguagem ou hardware onde as garantias de Rust não se aplicam.

O Listagem 19-1 mostra como criar um ponteiro bruto imutável e um mutável.

Listagem 19-1: Criando ponteiros brutos a partir de referências

```rust
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
```

Observe que não incluímos a palavra-chave `unsafe` neste código. Podemos criar ponteiros brutos em código seguro; apenas não podemos desreferenciar ponteiros brutos fora de um bloco inseguro, como você verá em breve.

Criamos ponteiros brutos usando `as` para converter uma referência imutável e uma mutável em seus tipos de ponteiro bruto correspondentes. Como os criamos diretamente de referências garantidas como válidas, sabemos que esses ponteiros brutos específicos são válidos, mas não podemos fazer essa suposição sobre qualquer ponteiro bruto.

Para demonstrar isso, a seguir criaremos um ponteiro bruto cuja validade não podemos ter tanta certeza. O Listagem 19-2 mostra como criar um ponteiro bruto para uma localização arbitrária na memória. Tentar usar memória arbitrária é indefinido: pode haver dados nesse endereço ou não, o compilador pode otimizar o código para que não haja acesso à memória, ou o programa pode terminar com um erro de segmentação. Geralmente, não há uma boa razão para escrever código como este, mas é possível.

Listagem 19-2: Criando um ponteiro bruto para um endereço de memória arbitrário

```rust
    let endereco = 0x012345usize;
    let r = endereco as *const i32;
```

Lembre-se de que podemos criar ponteiros brutos em código seguro, mas não podemos desreferenciar ponteiros brutos e ler os dados para os quais apontam. No Listagem 19-3, usamos o operador de desreferência `*` em um ponteiro bruto que requer um bloco `unsafe`.

Listagem 19-3: Desreferenciando ponteiros brutos dentro de um bloco `unsafe`

```rust
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    unsafe {
        println!("r1 é: {}", *r1);
        println!("r2 é: {}", *r2);
    }
```

Criar um ponteiro não faz mal; é apenas quando tentamos acessar o valor para o qual ele aponta que podemos acabar lidando com um valor inválido.

Observe também que nos Listagens 19-1 e 19-3, criamos ponteiros brutos `*const i32` e `*mut i32` que apontavam para o mesmo local de memória, onde `num` está armazenado. Se tentássemos criar uma referência imutável e uma mutável para `num`, o código não teria compilado porque as regras de propriedade de Rust não permitem uma referência mutável ao mesmo tempo que quaisquer referências imutáveis. Com ponteiros brutos, podemos criar um ponteiro mutável e um ponteiro imutável para o mesmo local e alterar dados através do ponteiro mutável, criando potencialmente uma corrida de dados. Tenha cuidado!

Com todos esses perigos, por que você usaria ponteiros brutos? Um caso de uso importante é ao interagir com código C, como você verá na próxima seção. Outro caso é ao construir abstrações seguras que o verificador de empréstimo não entende. Introduziremos funções inseguras e, em seguida, veremos um exemplo de uma abstração segura que usa código inseguro.

## Chamando uma Função ou Método Inseguro

O segundo tipo de operação que você pode realizar em um bloco inseguro é chamar funções inseguras. Funções e métodos inseguros parecem exatamente com funções e métodos regulares, mas têm um `unsafe` extra antes do resto da definição. A palavra-chave `unsafe` neste contexto indica que a função tem requisitos que precisamos manter quando chamamos essa função, porque Rust não pode garantir que atendemos a esses requisitos. Ao chamar uma função insegura dentro de um bloco `unsafe`, estamos dizendo que lemos a documentação dessa função e assumimos a responsabilidade de manter os contratos da função.

Aqui está uma função insegura chamada `perigoso` que não faz nada em seu corpo:

```rust
    unsafe fn perigoso() {}

    unsafe {
        perigoso();
    }
```

Devemos chamar a função `perigoso` dentro de um bloco `unsafe` separado. Se tentarmos chamar `perigoso` sem o bloco `unsafe`, receberemos um erro:

```console
error[E0133]: call to unsafe function requires unsafe function or block
 --> src/main.rs:4:5
  |
4 |     perigoso();
  |     ^^^^^^^^^^ call to unsafe function
  |
  = note: consult the function's documentation for information on how to avoid undefined behavior
```

Com o bloco `unsafe`, estamos afirmando para Rust que lemos a documentação da função, entendemos como usá-la corretamente e verificamos que estamos cumprindo o contrato da função.

Os corpos de funções inseguras são efetivamente blocos `unsafe`, então, para realizar outras operações inseguras dentro de uma função insegura, não precisamos adicionar outro bloco `unsafe`.

### Criando uma Abstração Segura sobre Código Inseguro

Apenas porque uma função contém código inseguro não significa que precisamos marcar toda a função como insegura. De fato, envolver código inseguro em uma função segura é uma abstração comum. Como exemplo, vamos estudar a função `split_at_mut` da biblioteca padrão, que requer algum código inseguro. Exploraremos como poderíamos implementá-la. Este método seguro é definido em fatias mutáveis: ele pega uma fatia e a transforma em duas dividindo a fatia no índice fornecido como argumento. O Listagem 19-4 mostra como usar `split_at_mut`.

Listagem 19-4: Usando a função segura `split_at_mut`

```rust
    let mut v = vec![1, 2, 3, 4, 5, 6];

    let r = &mut v[..];

    let (a, b) = r.split_at_mut(3);

    assert_eq!(a, &mut [1, 2, 3]);
    assert_eq!(b, &mut [4, 5, 6]);
```

Não podemos implementar esta função usando apenas Rust seguro. Uma tentativa pode parecer algo como o Listagem 19-5, que não compilará. Para simplificar, implementaremos `split_at_mut` como uma função em vez de um método e apenas para fatias de valores `i32` em vez de para um tipo genérico `T`.

Listagem 19-5: Uma tentativa de implementação de `split_at_mut` usando apenas Rust seguro

```rust,ignore,does_not_compile
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);

    (&mut values[..mid], &mut values[mid..])
}
```

Esta função primeiro obtém o comprimento total da fatia. Em seguida, afirma que o índice fornecido como parâmetro está dentro da fatia, verificando se é menor ou igual ao comprimento. A afirmação significa que, se passarmos um índice maior que o comprimento para dividir a fatia, a função entrará em pânico antes de tentar usar esse índice.

Então, retornamos duas fatias mutáveis em uma tupla: uma do início da fatia original até o índice `mid` e outra de `mid` até o final da fatia.

Quando tentamos compilar o código no Listagem 19-5, receberemos um erro:

```console
error[E0499]: cannot borrow `*values` as mutable more than once at a time
 --> src/main.rs:6:31
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let's call the lifetime of this reference `'1`
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------------^^^^^^--------
  |     |     |                   |
  |     |     |                   second mutable borrow occurs here
  |     |     first mutable borrow occurs here
  |     returning this value requires that `*values` is borrowed for `'1`
```

O verificador de empréstimo de Rust não consegue entender que estamos emprestando diferentes partes da fatia; ele só sabe que estamos emprestando da mesma fatia duas vezes. Emprestar diferentes partes de uma fatia é fundamentalmente ok porque as duas fatias não estão se sobrepondo, mas Rust não é inteligente o suficiente para saber disso. Quando sabemos que o código está ok, mas Rust não, é hora de recorrer ao código inseguro.

O Listagem 19-6 mostra como usar um bloco `unsafe`, um ponteiro bruto e algumas chamadas para funções inseguras para fazer a implementação de `split_at_mut` funcionar.

Listagem 19-6: Usando código inseguro na implementação da função `split_at_mut`

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

Lembre-se da seção "O Tipo Slice" no Capítulo 4 que uma fatia é um ponteiro para alguns dados e o comprimento da fatia. Usamos o método `len` para obter o comprimento de uma fatia e o método `as_mut_ptr` para acessar o ponteiro bruto de uma fatia. Neste caso, porque temos uma fatia mutável para valores `i32`, `as_mut_ptr` retorna um ponteiro bruto com o tipo `*mut i32`, que armazenamos na variável `ptr`.

Mantemos a afirmação de que o índice `mid` está dentro da fatia. Então chegamos ao código inseguro: a função `slice::from_raw_parts_mut` pega um ponteiro bruto e um comprimento, e cria uma fatia. Usamos essa função para criar uma fatia que começa em `ptr` e tem `mid` itens de comprimento. Em seguida, chamamos o método `add` em `ptr` com `mid` como argumento para obter um ponteiro bruto que começa em `mid`, e criamos uma fatia usando esse ponteiro e o número restante de itens após `mid` como o comprimento.

A função `slice::from_raw_parts_mut` é insegura porque pega um ponteiro bruto e deve confiar que esse ponteiro é válido. O método `add` em ponteiros brutos também é inseguro porque deve confiar que o local de deslocamento também é um ponteiro válido. Portanto, tivemos que colocar um bloco `unsafe` em torno de nossas chamadas para `slice::from_raw_parts_mut` e `add` para que pudéssemos chamá-las. Olhando para o código e adicionando a afirmação de que `mid` deve ser menor ou igual a `len`, podemos dizer que todos os ponteiros brutos usados dentro do bloco `unsafe` serão ponteiros válidos para dados dentro da fatia. Este é um uso aceitável e apropriado de `unsafe`.

Observe que não precisamos marcar a função `split_at_mut` resultante como `unsafe`, e podemos chamar essa função de Rust seguro. Criamos uma abstração segura para o código inseguro com uma implementação da função que usa código `unsafe` de uma maneira segura, porque cria apenas ponteiros válidos a partir dos dados aos quais esta função tem acesso.

Em contraste, o uso de `slice::from_raw_parts_mut` no Listagem 19-7 provavelmente travaria quando a fatia fosse usada. Este código pega um local de memória arbitrário e cria uma fatia de 10.000 itens de comprimento.

Listagem 19-7: Criando uma fatia a partir de um local de memória arbitrário

```rust
    use std::slice;

    let address = 0x01234usize;
    let r = address as *mut i32;

    let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
```

Não possuímos a memória neste local arbitrário, e não há garantia de que a fatia que este código cria contenha valores `i32` válidos. Tentar usar `values` como se fosse uma fatia válida resulta em comportamento indefinido.

### Usando Funções `extern` para Chamar Código Externo

Às vezes, seu código Rust pode precisar interagir com código escrito em outra linguagem. Para isso, Rust tem a palavra-chave `extern` que facilita a criação e o uso de uma *Foreign Function Interface (FFI)* (Interface de Função Estrangeira), que é uma maneira de uma linguagem de programação definir funções e permitir que uma linguagem de programação diferente (estrangeira) chame essas funções.

O Listagem 19-8 demonstra como configurar uma integração com a função `abs` da biblioteca padrão C. Funções declaradas dentro de blocos `extern` são sempre inseguras para chamar do código Rust. A razão é que outras linguagens não impõem as regras e garantias de Rust, e Rust não pode verificá-las, então a responsabilidade recai sobre o programador para garantir a segurança.

Listagem 19-8: Declarando e chamando uma função `extern` definida em outra linguagem

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Valor absoluto de -3 de acordo com C: {}", abs(-3));
    }
}
```

Dentro do bloco `extern "C"`, listamos os nomes e assinaturas de funções externas de outra linguagem que queremos chamar. A parte `"C"` define qual *interface binária de aplicação (ABI)* a função externa usa: a ABI define como chamar a função no nível de montagem (assembly). A ABI `"C"` é a mais comum e segue a ABI da linguagem de programação C.

> ### Chamando Funções Rust de Outras Linguagens
>
> Também podemos usar `extern` para criar uma interface que permite que outras linguagens chamem funções Rust. Em vez de um bloco `extern`, adicionamos a palavra-chave `extern` e especificamos a ABI a ser usada logo antes da palavra-chave `fn`. Também precisamos adicionar uma anotação `#[no_mangle]` para dizer ao compilador Rust para não desfigurar (mangle) o nome desta função. *Mangling* é quando um compilador altera o nome que demos a uma função para um nome diferente que contém mais informações para outras partes do processo de compilação consumirem, mas é menos legível para humanos. Todo compilador de linguagem de programação desfigura nomes ligeiramente diferente, então, para que uma função Rust seja nomeável por outras linguagens, devemos desabilitar a desfiguração de nomes do compilador Rust.
>
> No exemplo a seguir, tornamos a função `call_from_c` acessível a partir do código C, depois de compilada para uma biblioteca compartilhada e vinculada a partir de C:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Acabei de chamar uma função Rust de C!");
> }
> ```
>
> Este uso de `extern` não requer `unsafe`.

## Acessando ou Modificando uma Variável Estática Mutável

Neste livro, ainda não falamos sobre variáveis globais, que Rust suporta, mas que podem ser problemáticas com as regras de propriedade de Rust. Se duas threads estiverem acessando a mesma variável global mutável, isso pode causar uma corrida de dados.

Em Rust, variáveis globais são chamadas de variáveis *estáticas* (static variables). O Listagem 19-9 mostra um exemplo de declaração e uso de uma variável estática com uma fatia de string como valor.

Listagem 19-9: Definindo e usando uma variável estática imutável

```rust
static HELLO_WORLD: &str = "Olá, mundo!";

fn main() {
    println!("o nome é: {}", HELLO_WORLD);
}
```

Variáveis estáticas são semelhantes a constantes, que discutimos na seção "Diferenças Entre Variáveis e Constantes" no Capítulo 3. Os nomes de variáveis estáticas estão em `SCREAMING_SNAKE_CASE` por convenção. Variáveis estáticas só podem armazenar referências com o tempo de vida `'static`, o que significa que o compilador Rust pode descobrir o tempo de vida e não somos obrigados a anotá-lo explicitamente. Acessar uma variável estática imutável é seguro.

Uma diferença sutil entre constantes e variáveis estáticas imutáveis é que os valores em uma variável estática têm um endereço fixo na memória. Usar o valor sempre acessará os mesmos dados. Constantes, por outro lado, podem duplicar seus dados sempre que forem usadas. Outra diferença é que variáveis estáticas podem ser mutáveis. Acessar e modificar variáveis estáticas mutáveis é *inseguro*. O Listagem 19-10 mostra como declarar, acessar e modificar uma variável estática mutável chamada `COUNTER`.

Listagem 19-10: Lendo ou escrevendo em uma variável estática mutável é inseguro

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

Assim como com variáveis regulares, especificamos a mutabilidade usando a palavra-chave `mut`. Qualquer código que leia ou escreva em `COUNTER` deve estar dentro de um bloco `unsafe`. Este código compila e imprime `COUNTER: 3` como esperaríamos porque é single-threaded. Ter múltiplas threads acessando `COUNTER` provavelmente resultaria em corridas de dados.

Com dados mutáveis que são globalmente acessíveis, é difícil garantir que não haja corridas de dados, e é por isso que Rust considera variáveis estáticas mutáveis inseguras. Sempre que possível, é preferível usar as técnicas de concorrência e ponteiros inteligentes thread-safe que discutimos no Capítulo 16, para que o compilador verifique se o acesso aos dados de diferentes threads é feito com segurança.

## Implementando um Trait Inseguro

Podemos usar `unsafe` para implementar um trait inseguro. Um trait é inseguro quando pelo menos um de seus métodos tem algum invariante que o compilador não pode verificar. Declaramos que um trait é `unsafe` adicionando a palavra-chave `unsafe` antes de `trait` e marcando a implementação do trait como `unsafe` também, como mostrado no Listagem 19-11.

Listagem 19-11: Definindo e implementando um trait inseguro

```rust
unsafe trait Foo {
    // métodos vão aqui
}

unsafe impl Foo for i32 {
    // implementações de métodos vão aqui
}

fn main() {}
```

Ao usar `unsafe impl`, estamos prometendo que manteremos os invariantes que o compilador não pode verificar.

Como exemplo, lembre-se dos marcadores de trait `Send` e `Sync` que discutimos na seção "Concorrência Extensível com os Traits `Send` e `Sync`" no Capítulo 16: o compilador implementa esses traits automaticamente se nossos tipos forem compostos inteiramente de outros tipos que implementam `Send` e `Sync`. Se implementarmos um tipo que contém um tipo que não é `Send` ou `Sync`, como ponteiros brutos, e quisermos marcar esse tipo como `Send` ou `Sync`, devemos usar `unsafe`. Rust não pode verificar se nosso tipo mantém as garantias de que pode ser enviado com segurança entre threads ou acessado de múltiplas threads; portanto, precisamos fazer essas verificações manualmente e indicar isso com `unsafe`.

## Acessando Campos de uma Union

A ação final que funciona apenas com `unsafe` é acessar campos de uma *union*. Uma `union` é semelhante a uma `struct`, mas apenas um campo declarado é usado em uma instância específica de cada vez. Unions são usadas principalmente para interagir com unions em código C. Acessar campos de union é inseguro porque Rust não pode garantir o tipo dos dados atualmente armazenados na instância da union. Você pode aprender mais sobre unions na Referência do Rust.

## Quando Usar Código Inseguro

Usar `unsafe` para realizar uma das cinco ações (superpoderes) que acabamos de discutir não é errado ou mesmo desaprovado. Mas é mais difícil acertar o código `unsafe` porque o compilador não pode ajudar a manter a segurança da memória. Quando você tem um motivo para usar código `unsafe`, pode fazê-lo, e ter a anotação `unsafe` explícita torna mais fácil rastrear a fonte dos problemas quando eles ocorrem.
