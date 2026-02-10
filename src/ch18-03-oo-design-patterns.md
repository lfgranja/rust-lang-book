# Implementando um Padrão de Projeto Orientado a Objetos

O *state pattern* (padrão de estado) é um padrão de projeto orientado a objetos. O cerne do padrão é que um valor tem algum estado interno, que é representado por um conjunto de *state objects* (objetos de estado), e o comportamento do valor muda com base no estado interno. Os objetos de estado compartilham funcionalidade: em Rust, é claro, usamos structs e traits em vez de objetos e herança. Cada objeto de estado é responsável por seu próprio comportamento e por governar quando deve transicionar para outro estado. O valor que detém um objeto de estado não sabe nada sobre o comportamento diferente dos estados ou quando transicionar entre estados.

Usar o padrão de estado significa que quando os requisitos de negócios do programa mudam, não precisaremos alterar o código do valor que detém o estado ou o código que usa o valor. Precisaremos apenas atualizar o código dentro de um dos objetos de estado para alterar suas regras ou talvez adicionar mais objetos de estado. Vamos ver um exemplo do padrão de projeto de estado e como usá-lo em Rust.

Implementaremos um fluxo de trabalho de postagem de blog de maneira incremental. A funcionalidade final do blog será assim:

1. Uma postagem de blog começa como um rascunho vazio.
2. Quando o rascunho estiver pronto, uma revisão do post é solicitada.
3. Quando a postagem é aprovada, ela é publicada.
4. Apenas postagens publicadas retornam conteúdo para imprimir, então postagens não aprovadas não podem ser publicadas acidentalmente.

Qualquer outra alteração tentada em uma postagem não deve ter efeito. Por exemplo, se tentarmos aprovar um rascunho de postagem antes de solicitarmos uma revisão, a postagem deve permanecer um rascunho não publicado.

A Listagem 17-11 mostra este fluxo de trabalho em forma de código: este é um exemplo de uso da API que implementaremos em uma crate de biblioteca chamada `blog`. Isso ainda não compilará porque ainda não implementamos a crate `blog`.

Listagem 17-11: Código que demonstra o comportamento desejado que queremos que nossa crate `blog` tenha

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_texto("Eu comi uma salada no almoço hoje");
    assert_eq!("", post.conteudo());

    post.solicitar_revisao();
    assert_eq!("", post.conteudo());

    post.aprovar();
    assert_eq!("Eu comi uma salada no almoço hoje", post.conteudo());
}
```

Queremos permitir que o usuário crie um novo rascunho de postagem com `Post::new`. Em seguida, queremos permitir que algum texto seja adicionado ao rascunho da postagem. Se tentarmos obter o conteúdo da postagem imediatamente, não devemos obter nenhum texto porque a postagem ainda é um rascunho. Adicionamos `assert_eq!` no código para fins de demonstração. Uma excelente prova unitária para isso seria afirmar que um rascunho de postagem retorna uma string vazia de seu método `conteudo`.

Em seguida, queremos permitir uma solicitação de revisão da postagem, e queremos que `conteudo` retorne uma string vazia enquanto aguarda a revisão. Quando a postagem recebe aprovação, ela deve ser publicada, o que significa que o texto adicionado ao ser chamada `conteudo` será retornado.

Observe que a única interação com a postagem é através do tipo `Post`. O tipo `Post` (e o código que usa `Post`) não precisa saber sobre os vários estados que uma postagem pode ter (rascunho, aguardando revisão e publicado). Essas regras de transição de estado e o comportamento da postagem em cada estado são gerenciados pelos objetos de estado que veremos em breve.

## Definindo `Post` e Criando uma Nova Instância no Estado de Rascunho

Vamos começar a implementação da biblioteca! Sabemos que precisamos de uma struct pública `Post` que contenha algum conteúdo, então começaremos com a definição da struct e uma implementação pública associada de `new` para criar uma instância de `Post`, como mostrado na Listagem 17-12. Também criaremos um trait privado `Estado`. O `Post` conterá um trait object de `Box<dyn Estado>` dentro de um `Option<T>` em um campo privado `estado`. Você verá por que o `Option` é necessário em breve.

Listagem 17-12: Definição da struct `Post` e do trait `Estado`

```rust
pub struct Post {
    estado: Option<Box<dyn Estado>>,
    conteudo: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            estado: Some(Box::new(Rascunho {})),
            conteudo: String::new(),
        }
    }
}

trait Estado {}

struct Rascunho {}

impl Estado for Rascunho {}
```

O trait `Estado` define o comportamento compartilhado por todos os estados de uma postagem. Os estados são `Rascunho`, `AguardandoRevisao` e `Publicado`. Por enquanto, o trait `Estado` não tem métodos, e definiremos apenas a struct `Rascunho` porque esse é o estado em que queremos iniciar uma postagem.

Quando criamos um novo `Post`, definimos seu campo `estado` como uma variante `Some` que contém um `Box`. Este `Box` aponta para uma nova instância da struct `Rascunho`. Isso garante que sempre que criamos uma nova instância de `Post`, ela começará como um rascunho. Como o campo `estado` de `Post` é privado, não há como criar um `Post` em qualquer outro estado! Na função `Post::new`, definimos o campo `conteudo` para uma nova `String` vazia.

## Armazenando o Texto do Conteúdo da Postagem

Vimos no Listagem 17-11 que queremos poder chamar um método chamado `add_texto` e passar a ele uma `&str` que é então adicionada ao conteúdo de texto do `Post`. Implementamos isso como um método, em vez de expor o campo `conteudo` como `pub`. Isso significa que podemos implementar um método que controlará como os dados do campo `conteudo` são lidos. O método `add_texto` é bastante simples, então vamos adicionar a implementação no Listagem 17-13 ao bloco `impl Post`:

Listagem 17-13: Implementando o método `add_texto` para adicionar texto ao `conteudo` da postagem

```rust
impl Post {
    // ...trecho omitido...
    pub fn add_texto(&mut self, text: &str) {
        self.conteudo.push_str(text);
    }
}
```

O método `add_texto` pega uma referência mutável para `self`, porque estamos mudando a instância `Post` que estamos chamando. Em seguida, chamamos `push_str` na `String` em `conteudo` e passamos o argumento `text`. Este código não depende do estado em que a postagem está, o que não faz parte do padrão de estado. O método `add_texto` não interage com o campo `estado`, mas faz parte do comportamento que queremos suportar.

## Garantindo que o Conteúdo de um Rascunho de Postagem Esteja Vazio

Mesmo depois de chamarmos `add_texto` e adicionarmos algum conteúdo à nossa postagem, ainda queremos que o método `conteudo` retorne uma fatia de string vazia porque a postagem ainda está no estado de rascunho, como mostrado no Listagem 17-11. Por enquanto, vamos implementar o método `conteudo` com a coisa mais simples que funcionará: sempre retornando uma fatia de string vazia. Mudaremos isso mais tarde, uma vez que implementamos a capacidade de mudar o estado de uma postagem para que ela possa ser publicada. Até agora, as postagens só podem estar no estado de rascunho, então o conteúdo da postagem deve estar sempre vazio. O Listagem 17-14 mostra esta implementação placeholder:

Listagem 17-14: Adicionando uma implementação placeholder para o método `conteudo` de `Post` que sempre retorna uma string vazia

```rust
impl Post {
    // ...trecho omitido...
    pub fn conteudo(&self) -> &str {
        ""
    }
}
```

Com este método `conteudo` adicionado, tudo no Listagem 17-11 até a linha 7 funciona como pretendido.

## Solicitando uma Revisão da Postagem Muda Seu Estado

Em seguida, precisamos adicionar funcionalidade para solicitar uma revisão de uma postagem, o que deve mudar seu estado de `Rascunho` para `AguardandoRevisao`. Para fazer isso, chamaremos um método público chamado `solicitar_revisao` que pegará uma referência mutável para `self`. Então chamaremos um método interno `solicitar_revisao` no estado atual de `Post`, e este segundo método `solicitar_revisao` consumirá o estado atual e retornará um novo estado.

Adicionamos o método `solicitar_revisao` ao trait `Estado`; todos os tipos que implementam o trait agora precisarão implementar o método `solicitar_revisao`. Note que o primeiro argumento para o método é `Box<Self>`, em vez de `self`, `&self` ou `&mut self`. Esta sintaxe significa que o método é válido apenas quando chamado em um `Box` segurando o tipo. Este método toma posse do `Box<Self>`, invalidando o estado antigo para que o valor de estado do `Post` possa se transformar em um novo estado.

Para consumir o estado antigo, o método `solicitar_revisao` precisa tomar posse do valor do estado. É aqui que o `Option` no campo `estado` de `Post` entra: chamamos o método `take` para tirar o valor `Some` do campo `estado` e deixar um `None` em seu lugar, porque Rust não nos deixa ter campos não preenchidos em structs. Isso nos permite mover o valor `estado` para fora de `Post` em vez de emprestá-lo. Então definiremos o valor do estado da postagem para o resultado desta operação.

Precisamos definir `estado` para `Option::None` temporariamente em vez de defini-lo diretamente com código como `self.estado = self.estado.solicitar_revisao();` para obter a propriedade do valor `estado`. Isso garante que `Post` não possa usar o valor `estado` antigo depois de tê-lo transformado em um novo estado.

O método `solicitar_revisao` no trait `Estado` retorna `Box<dyn Estado>`. Isso significa que o novo estado que retorna também deve implementar o trait `Estado`. O estado `Rascunho` retornará uma nova instância de `AguardandoRevisao`, que representa o estado em que a postagem está aguardando revisão. A struct `AguardandoRevisao` também implementará `Estado` e o método `solicitar_revisao`. O método `solicitar_revisao` em `AguardandoRevisao` apenas retornará a si mesmo, porque solicitar uma revisão em uma postagem que já está em revisão não deve mudar o estado.

Listagem 17-15: Implementando métodos `solicitar_revisao` em `Post` e no trait `Estado`

```rust
pub struct Post {
    estado: Option<Box<dyn Estado>>,
    conteudo: String,
}

impl Post {
    // ...trecho omitido...
    pub fn solicitar_revisao(&mut self) {
        if let Some(s) = self.estado.take() {
            self.estado = Some(s.solicitar_revisao())
        }
    }
}

trait Estado {
    fn solicitar_revisao(self: Box<Self>) -> Box<dyn Estado>;
}

struct Rascunho {}

impl Estado for Rascunho {
    fn solicitar_revisao(self: Box<Self>) -> Box<dyn Estado> {
        Box::new(AguardandoRevisao {})
    }
}

struct AguardandoRevisao {}

impl Estado for AguardandoRevisao {
    fn solicitar_revisao(self: Box<Self>) -> Box<dyn Estado> {
        self
    }
}
```

Agora podemos ver as vantagens do padrão de estado: o método `solicitar_revisao` em `Post` é o mesmo, não importa em que estado o `estado` esteja. Cada estado é responsável por suas próprias regras.

## Adicionando o Método `aprovar` que Muda o Comportamento do `conteudo`

O método `aprovar` será semelhante ao método `solicitar_revisao`: ele definirá `estado` para o valor que o estado atual diz que deve ter quando esse estado é aprovado. Adicionamos o método `aprovar` ao trait `Estado` e adicionamos uma nova struct que implementa `Estado`, o estado `Publicado`.

Da mesma forma que `solicitar_revisao` em `AguardandoRevisao`, se chamarmos o método `aprovar` em um `Rascunho`, ele não deve ter efeito porque `aprovar` só deve ter efeito se a postagem já estiver no estado `AguardandoRevisao`. Se chamarmos `aprovar` em `AguardandoRevisao`, ele deve retornar uma nova instância de `Publicado`, representando o estado aprovado. Se chamarmos `aprovar` em `Publicado`, ele deve retornar a si mesmo, porque a postagem já está publicada.

Listagem 17-16: Implementando o método `aprovar` em `Post` e no trait `Estado`

```rust
impl Post {
    // ...trecho omitido...
    pub fn aprovar(&mut self) {
        if let Some(s) = self.estado.take() {
            self.estado = Some(s.aprovar())
        }
    }
}

trait Estado {
    fn solicitar_revisao(self: Box<Self>) -> Box<dyn Estado>;
    fn aprovar(self: Box<Self>) -> Box<dyn Estado>;
}

struct Rascunho {}

impl Estado for Rascunho {
    // ...trecho omitido...
    fn aprovar(self: Box<Self>) -> Box<dyn Estado> {
        self
    }
}

struct AguardandoRevisao {}

impl Estado for AguardandoRevisao {
    // ...trecho omitido...
    fn aprovar(self: Box<Self>) -> Box<dyn Estado> {
        Box::new(Publicado {})
    }
}

struct Publicado {}

impl Estado for Publicado {
    fn solicitar_revisao(self: Box<Self>) -> Box<dyn Estado> {
        self
    }

    fn aprovar(self: Box<Self>) -> Box<dyn Estado> {
        self
    }
}
```

Agora precisamos atualizar o método `conteudo` em `Post`. Queremos que o valor retornado de `conteudo` dependa do estado atual do `Post`, então teremos o `Post` delegando a um método `conteudo` definido em seu `estado`.

Listagem 17-17: Atualizando o método `conteudo` em `Post` para delegar a um método `conteudo` em `Estado`

```rust
// ...trecho omitido...
impl Post {
    // ...trecho omitido...
    pub fn conteudo(&self) -> &str {
        self.estado.as_ref().unwrap().conteudo(self)
    }
    // ...trecho omitido...
}
```

Porque o objetivo é manter todos esses regras dentro das structs que implementam `Estado`, chamamos um método `conteudo` no valor em `estado` e passamos a instância da postagem (isto é, `self`) como um argumento. Então retornamos o valor que é retornado do método `conteudo` no valor `estado`.

Chamamos o método `as_ref` no `Option` porque queremos uma referência ao valor dentro do `Option` em vez de tomar posse dele. Como `estado` é um `Option<Box<dyn Estado>>`, quando chamamos `as_ref`, um `Option<&Box<dyn Estado>>` é retornado. Se não chamássemos `as_ref`, obteríamos um erro porque não podemos mover `estado` para fora da função emprestada `&self`.

Então chamamos o método `unwrap`, que sabemos que nunca causará pânico, porque sabemos que os métodos em `Post` garantem que `estado` sempre conterá um valor `Some` quando esses métodos terminarem.

Neste ponto, quando chamamos `conteudo` no `&Box<dyn Estado>`, o desreferenciamento coercivo entrará em vigor no `&` e no `Box` para que o método `conteudo` seja chamado no tipo que implementa o trait `Estado`. Isso significa que precisamos adicionar `conteudo` à definição do trait `Estado` e é onde colocaremos a lógica para o que retornar dependendo do estado.

Listagem 17-18: Adicionando o método `conteudo` ao trait `Estado`

```rust
trait Estado {
    // ...trecho omitido...
    fn conteudo<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// ...trecho omitido...
struct Publicado {}

impl Estado for Publicado {
    // ...trecho omitido...
    fn conteudo<'a>(&self, post: &'a Post) -> &'a str {
        &post.conteudo
    }
}
```

Adicionamos uma implementação padrão para o método `conteudo` que retorna uma fatia de string vazia. Isso significa que não precisamos implementar `conteudo` nas structs `Rascunho` e `AguardandoRevisao`. A struct `Publicado` substituirá o método `conteudo` e retornará o valor em `post.conteudo`.

Note que precisamos de anotações de tempo de vida (lifetimes) neste método, como discutimos no Capítulo 10. Estamos pegando uma referência a um `post` como argumento e retornando uma referência a parte desse `post`, então o tempo de vida da referência retornada está relacionado ao tempo de vida do argumento `post`.

E pronto! Implementamos o padrão de estado em Rust. O código `main` no Listagem 17-11 agora funciona como pretendido.
