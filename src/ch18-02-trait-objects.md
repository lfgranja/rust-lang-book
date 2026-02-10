# Usando Objetos de Trait que Permitem Valores de Tipos Diferentes

No Capítulo 8, mencionamos que uma limitação dos vetores é que eles podem armazenar apenas elementos de um único tipo. Criamos uma solução alternativa no Listagem 8-10, onde definimos um enum `CelulaPlanilha` que tinha variantes para conter inteiros, floats e textos. Isso significava que poderíamos armazenar diferentes tipos de dados em cada célula e ainda ter um vetor que representasse uma linha de células. Esta é uma solução perfeita quando nossos itens intercambiáveis são um conjunto fixo de tipos que conhecemos quando nosso código é compilado.

No entanto, às vezes queremos que nosso usuário da biblioteca seja capaz de estender o conjunto de tipos que são válidos em uma situação específica. Para mostrar como podemos conseguir isso, criaremos uma ferramenta de interface gráfica de usuário (GUI) de exemplo que itera através de uma lista de itens, chamando um método `desenhar` em cada um para desenhá-lo na tela — uma técnica comum para ferramentas GUI. Criaremos uma caixa (crate) de biblioteca chamada `gui` que contém a estrutura de uma biblioteca GUI. Essa caixa pode incluir alguns tipos para as pessoas usarem, como `Botao` ou `CampoTexto`. Além disso, os usuários de `gui` desejarão criar seus próprios tipos que podem ser desenhados: por exemplo, um programador pode adicionar uma `Imagem` e outro pode adicionar uma `CaixaSelecao`.

Não implementaremos uma biblioteca GUI completa para este exemplo, mas mostraremos como as peças se encaixariam. No momento de escrever a biblioteca, não podemos saber e definir todos os tipos que outros programadores podem querer criar. Mas sabemos que `gui` precisa acompanhar muitos valores de tipos diferentes, e precisa chamar um método `desenhar` em cada um desses valores de tipos diferentes. Não precisa saber exatamente o que acontecerá quando chamarmos o método `desenhar`, apenas que o valor terá esse método disponível para chamarmos.

Para fazer isso em uma linguagem com herança, poderíamos definir uma classe chamada `Componente` que tem um método chamado `desenhar` nela. As outras classes, como `Botao`, `Imagem` e `CaixaSelecao`, herdariam de `Componente` e, portanto, herdariam o método `desenhar`. Elas poderiam cada uma sobrescrever o método `desenhar` para definir seu comportamento personalizado, mas o framework poderia tratar todos os tipos como se fossem instâncias de `Componente` e chamar `desenhar` nelas. Mas como Rust não tem herança, precisamos de outra maneira de estruturar a biblioteca `gui` para permitir que os usuários a estendam com novos tipos.

## Definindo um Trait para Comportamento Comum

Para implementar o comportamento que queremos que `gui` tenha, definiremos um [[Traits|trait]] chamado `Desenhavel` que terá um método chamado `desenhar`. Então, podemos definir um vetor que recebe um *trait object*. Um trait object aponta para uma instância de um tipo que implementa nosso trait especificado, bem como uma tabela usada para procurar métodos desse trait em tempo de execução. Criamos um trait object especificando algum tipo de ponteiro, como uma referência `&` ou um ponteiro inteligente `Box<T>`, depois a palavra-chave `dyn`, e então especificando o trait relevante. (Falaremos sobre o motivo pelo qual trait objects devem usar um ponteiro no Capítulo 19 na seção "Tipos de Tamanho Dinâmico e o Trait `Sized`".) Podemos usar trait objects no lugar de um tipo genérico ou concreto. Onde quer que usemos um trait object, o sistema de tipos de Rust garantirá em tempo de compilação que qualquer valor usado nesse contexto implementará o trait do trait object. Consequentemente, não precisamos saber todos os tipos possíveis em tempo de compilação.

Mencionamos que em Rust, evitamos usar os termos "objeto" e "herança" para diferenciar de outras linguagens, porque eles têm significados diferentes. No entanto, **trait objects** (objetos de trait) em Rust são bastante semelhantes a objetos em outras linguagens, no sentido de que eles combinam dados e comportamento. Mas trait objects diferem de objetos tradicionais porque não podemos adicionar dados a um trait object. Trait objects não são tão úteis quanto objetos em outras linguagens: seu propósito específico é permitir abstração sobre comportamento comum.

Listagem 17-3: Definição do trait `Desenhavel`

```rust
pub trait Desenhavel {
    fn desenhar(&self);
}
```

Essa sintaxe deve ser familiar de nossas discussões sobre como definir traits no Capítulo 10. Em seguida vem a nova sintaxe: Listagem 17-4 define uma struct chamada `Tela` que contém um vetor chamado `componentes`. Este vetor é do tipo `Box<dyn Desenhavel>`, que é um trait object; é um substituto para qualquer tipo dentro de um `Box` que implementa o trait `Desenhavel`.

Listagem 17-4: Definição da struct `Tela` com um campo `componentes` contendo um vetor de trait objects que implementam `Desenhavel`

```rust
pub struct Tela {
    pub componentes: Vec<Box<dyn Desenhavel>>,
}

impl Tela {
    pub fn run(&self) {
        for componente in self.componentes.iter() {
            componente.desenhar();
        }
    }
}
```

Na struct `Tela`, armazenaremos um vetor de trait objects `Box<dyn Desenhavel>`. Na struct `Tela`, definiremos um método `run` que chama o método `desenhar` em cada um de seus `componentes`.

Isso funciona de forma diferente de definir uma struct que usa um parâmetro de tipo genérico com trait bounds. Um parâmetro de tipo genérico só pode ser substituído por um tipo concreto de cada vez, enquanto trait objects permitem que múltiplos tipos concretos preencham o trait object em tempo de execução. Por exemplo, poderíamos ter definido a struct `Tela` usando um tipo genérico e um trait bound como no Listagem 17-5:

Listagem 17-5: Uma implementação alternativa da struct `Tela` e seu método `run` usando genéricos e trait bounds

```rust
pub struct Tela<T: Desenhavel> {
    pub componentes: Vec<T>,
}

impl<T> Tela<T>
where
    T: Desenhavel,
{
    pub fn run(&self) {
        for componente in self.componentes.iter() {
            componente.desenhar();
        }
    }
}
```

Isso nos restringe a uma instância de `Tela` que tem uma lista de componentes todos do tipo `Botao` ou todos do tipo `CampoTexto`. Se você tiver apenas coleções homogêneas, usar genéricos e trait bounds é preferível porque as definições serão monomorfizadas em tempo de compilação para usar os tipos concretos.

Por outro lado, com o método usando trait objects, uma instância de `Tela` pode conter um `Vec<T>` que contém um `Box<Botao>` bem como um `Box<CampoTexto>`. Vamos ver como isso funciona e depois falaremos sobre as implicações de desempenho em tempo de execução.

## Implementando o Trait

Agora vamos adicionar alguns tipos que implementam o trait `Desenhavel`. Forneceremos o tipo `Botao`. Novamente, a implementação real de uma biblioteca GUI está além do escopo deste livro, então o corpo do método `desenhar` não terá nenhuma implementação útil em seu corpo. Para imaginar como a implementação se pareceria, uma struct `Botao` poderia ter campos para `largura`, `altura` e `rotulo`, como mostrado no Listagem 17-6:

Listagem 17-6: Uma struct `Botao` que implementa o trait `Desenhavel`

```rust
pub struct Botao {
    pub largura: u32,
    pub altura: u32,
    pub rotulo: String,
}

impl Desenhavel for Botao {
    fn desenhar(&self) {
        // código para desenhar um botão na verdade
    }
}
```

Os campos `largura`, `altura` e `rotulo` em `Botao` serão diferentes dos campos em outros componentes; por exemplo, um tipo `CampoTexto` pode ter esses mesmos campos, mais um campo `placeholder`. Cada um dos tipos que queremos desenhar na tela implementará o trait `Desenhavel` mas usará um código diferente no método `desenhar` para definir como desenhar esse tipo específico, como `Botao` tem aqui (sem o código GUI real, como mencionado). O tipo `Botao`, por exemplo, pode ter um bloco `impl` separado contendo métodos relacionados ao que acontece quando o botão é clicado. Esses tipos de métodos não se aplicam a um `CampoTexto`.

Se alguém usando nossa biblioteca decidir implementar uma struct `CaixaSelecao` que tem `largura`, `altura` e `opcoes`, eles podem implementar o trait `Desenhavel` na struct `CaixaSelecao` também, como mostrado no Listagem 17-7:

Listagem 17-7: Outro tipo, `CaixaSelecao`, implementando o trait `Desenhavel`, em outra crate

```rust
use gui::Desenhavel;

struct CaixaSelecao {
    largura: u32,
    altura: u32,
    opcoes: Vec<String>,
}

impl Desenhavel for CaixaSelecao {
    fn desenhar(&self) {
        // código para desenhar uma caixa de seleção na verdade
    }
}
```

Agora que implementamos `Desenhavel` em `Botao` (dentro da biblioteca) e `CaixaSelecao` (fora da biblioteca), o usuário da nossa biblioteca pode criar uma instância de `Tela` e adicionar instâncias de `Botao` e `CaixaSelecao` a ela, e então chamar `run`, como mostrado no Listagem 17-8.

Listagem 17-8: Usando trait objects para armazenar valores de tipos diferentes que implementam o mesmo trait

```rust
use gui::{Desenhavel, Botao, Tela};

fn main() {
    let tela = Tela {
        componentes: vec![
            Box::new(CaixaSelecao {
                largura: 75,
                altura: 10,
                opcoes: vec![
                    String::from("Sim"),
                    String::from("Talvez"),
                    String::from("Não"),
                ],
            }),
            Box::new(Botao {
                largura: 50,
                altura: 10,
                rotulo: String::from("OK"),
            }),
        ],
    };

    tela.run();
}
```

Quando escrevemos a biblioteca, não sabíamos que alguém poderia adicionar o tipo `CaixaSelecao`, mas nossa implementação de `Tela` foi capaz de operar na nova variante e desenhá-la. Este conceito—de se preocupar apenas com as mensagens que um valor responde, em vez do tipo concreto do valor—é semelhante ao conceito de *duck typing* em linguagens tipadas dinamicamente: se anda como um pato e grasna como um pato, então deve ser um pato! Na implementação de `run` na `Tela` no Listagem 17-4, `run` não precisa saber qual é o tipo concreto de cada componente. Ele não verifica se um componente é uma instância de um `Botao` ou uma `CaixaSelecao`, ele apenas chama o método `desenhar` no componente. Ao especificar `Box<dyn Desenhavel>` como o tipo dos valores no vetor `componentes`, definimos que a `Tela` precisa de valores que possamos chamar o método `desenhar`.

A vantagem de usar trait objects e o sistema de tipos de Rust para escrever código semelhante ao código usando duck typing é que nunca precisamos verificar se um valor implementa um método específico em tempo de execução ou nos preocupar em obter erros se invocarmos um método mas o valor não o implementar. Rust não compilará nosso código se os valores não implementarem os traits que os trait objects precisam.

Por exemplo, o Listagem 17-9 mostra o que acontece se tentarmos criar uma `Tela` com uma `String` como componente:

Listagem 17-9: Tentando usar um tipo que não implementa o trait do trait object

```rust
use gui::Tela;

fn main() {
    let tela = Tela {
        componentes: vec![
            Box::new(String::from("Olá")),
        ],
    };

    tela.run();
}
```

Receberemos este erro porque `String` não implementa o trait `Desenhavel`:

```console
$ cargo run
   Compiling gui v0.1.0 (file:///projects/gui)
error[E0277]: the trait bound `String: Desenhavel` is not satisfied
  --> src/main.rs:5:13
   |
5  |             Box::new(String::from("Olá")),
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `Desenhavel` is not implemented for `String`
   |
   = note: required for the cast to the object type `dyn Desenhavel`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `gui` due to previous error
```

Este erro nos avisa que estamos passando algo para a `Tela` que não queríamos passar ou que devemos implementar `Desenhavel` na `String` para que a `Tela` possa chamar `desenhar` nela.

## Trait Objects Realizam Despacho Dinâmico

Lembre-se da seção "Desempenho de Código Usando Genéricos" no Capítulo 10, nossa discussão sobre o processo de monomorfização realizado pelo compilador quando usamos *trait bounds* em genéricos: o compilador gera implementações não genéricas de funções e métodos para cada tipo concreto que usamos no lugar de um parâmetro de tipo genérico. O código que resulta da monomorfização está fazendo *despacho estático* (static dispatch), que é quando o compilador sabe qual método você está chamando em tempo de compilação. Isso se opõe ao *despacho dinâmico* (dynamic dispatch), que é quando o compilador não pode dizer em tempo de compilação qual método você está chamando. Em casos de despacho dinâmico, o compilador emite código que descobrirá em tempo de execução qual método chamar.

Quando usamos trait objects, Rust deve usar despacho dinâmico. O compilador não sabe todos os tipos que podem ser usados com o código que está usando trait objects, então ele não sabe qual método implementado em qual tipo chamar. Em vez disso, em tempo de execução, Rust usa os ponteiros dentro do trait object para saber qual método chamar. Há um custo de tempo de execução quando essa busca acontece que não ocorre com o despacho estático. O despacho dinâmico também impede que o compilador escolha fazer *inline* do código de um método, o que por sua vez impede algumas otimizações. No entanto, ganhamos flexibilidade extra no código que escrevemos e no suporte no Listagem 17-5 e fomos capazes de suportar no Listagem 17-9, então é uma troca a considerar.
