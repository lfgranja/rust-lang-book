## `RefCell<T>` e o Padrão de Mutabilidade Interior

*Mutabilidade interior* (interior mutability) é um padrão de design em Rust que permite que você mute dados mesmo quando há referências imutáveis para esses dados; normalmente, esta ação não é permitida pelas regras de empréstimo. Para mutar dados, o padrão usa código `unsafe` dentro de uma estrutura de dados para contornar as regras usuais de Rust que governam a mutação e o empréstimo. Código unsafe indica ao compilador que estamos verificando as regras manualmente em vez de confiar no compilador para verificá-las por nós; discutiremos código unsafe mais no Capítulo 19.

Podemos usar tipos que usam o padrão de mutabilidade interior apenas quando podemos garantir que as regras de empréstimo serão seguidas em tempo de execução, mesmo que o compilador não possa garantir isso. O código `unsafe` envolvido é então envolvido em uma API segura, e o tipo externo ainda é imutável.

Vamos explorar este conceito olhando para o tipo `RefCell<T>` que segue o padrão de mutabilidade interior.

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### Impondo Regras de Empréstimo em Tempo de Execução

Ao contrário de `Rc<T>`, o tipo `RefCell<T>` representa posse única sobre os dados que contém. Então, o que torna `RefCell<T>` diferente de um tipo como `Box<T>`? Relembre as regras de empréstimo que você aprendeu no Capítulo 4:

- A qualquer momento, você pode ter *ou* uma referência mutável *ou* qualquer número de referências imutáveis (mas não ambas).
- Referências devem ser sempre válidas.

Com referências e `Box<T>`, os invariantes das regras de empréstimo são impostos em tempo de compilação. Com `RefCell<T>`, esses invariantes são impostos *em tempo de execução*. Com referências, se você quebrar essas regras, você receberá um erro de compilador. Com `RefCell<T>`, se você quebrar essas regras, seu programa entrará em pânico e sairá.

As vantagens de verificar as regras de empréstimo em tempo de compilação são que os erros serão detectados mais cedo no processo de desenvolvimento, e não há impacto na performance em tempo de execução porque toda a análise é concluída de antemão. Por essas razões, verificar as regras de empréstimo em tempo de compilação é a melhor escolha na maioria dos casos, e é por isso que este é o padrão de Rust.

A vantagem de verificar as regras de empréstimo em tempo de execução é que certos cenários seguros de memória são permitidos, onde eles teriam sido proibidos pelas verificações em tempo de compilação. Análise estática, como o compilador Rust, é inerentemente conservadora. Algumas propriedades de código são impossíveis de detectar analisando o código: o exemplo mais famoso é o Problema da Parada, que está fora do escopo deste livro, mas é um tópico interessante para pesquisar.

Como algumas análises são impossíveis, se o compilador Rust não puder ter certeza de que o código cumpre as regras de posse, ele pode rejeitar um programa correto; dessa forma, ele é conservador. Se Rust aceitasse um programa incorreto, os usuários não poderiam confiar nas garantias que Rust faz. No entanto, se Rust rejeita um programa correto, o programador será incomodado, mas nada catastrófico pode ocorrer. O tipo `RefCell<T>` é útil quando você tem certeza de que seu código segue as regras de empréstimo, mas o compilador é incapaz de entender e garantir isso.

Semelhante a `Rc<T>`, `RefCell<T>` é apenas para uso em cenários de thread única e lhe dará um erro em tempo de compilação se você tentar usá-lo em um contexto multithreaded. Falaremos sobre como obter a funcionalidade de `RefCell<T>` em um programa multithreaded no Capítulo 16.

Aqui está uma recapitulação das razões para escolher `Box<T>`, `Rc<T>` ou `RefCell<T>`:

- `Rc<T>` permite múltiplos donos dos mesmos dados; `Box<T>` e `RefCell<T>` têm donos únicos.
- `Box<T>` permite empréstimos imutáveis ou mutáveis verificados em tempo de compilação; `Rc<T>` permite apenas empréstimos imutáveis verificados em tempo de compilação; `RefCell<T>` permite empréstimos imutáveis ou mutáveis verificados em tempo de execução.
- Porque `RefCell<T>` permite empréstimos mutáveis verificados em tempo de execução, você pode mutar o valor dentro do `RefCell<T>` mesmo quando o `RefCell<T>` é imutável.

Mutar o valor dentro de um valor imutável é o padrão de mutabilidade interior. Vamos ver uma situação em que a mutabilidade interior é útil e examinar como é possível.

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### Usando Mutabilidade Interior

Uma consequência das regras de empréstimo é que quando você tem um valor imutável, você não pode tomá-lo emprestado mutavelmente. Por exemplo, este código não compilará:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Se você tentasse compilar este código, receberia o seguinte erro:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

No entanto, há situações em que seria útil para um valor mutar a si mesmo em seus métodos, mas parecer imutável para outro código. O código fora dos métodos do valor não seria capaz de mutar o valor. Usar `RefCell<T>` é uma maneira de obter a capacidade de ter mutabilidade interior, mas `RefCell<T>` não contorna as regras de empréstimo completamente: o verificador de empréstimo (borrow checker) no compilador permite essa mutabilidade interior, e as regras de empréstimo são verificadas em tempo de execução. Se você violar as regras, receberá um `panic!` em vez de um erro de compilador.

Vamos trabalhar em um exemplo prático onde podemos usar `RefCell<T>` para mutar um valor imutável e ver por que isso é útil.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### Testando com Objetos Mock

Às vezes, durante o teste, um programador usará um tipo no lugar de outro tipo, a fim de observar um comportamento específico e afirmar que ele está implementado corretamente. Este tipo de espaço reservado é chamado de *test double* (dublê de teste). Pense nisso no sentido de um dublê em filmagens, onde uma pessoa entra e substitui um ator para fazer uma cena particularmente complicada. Dublês de teste substituem outros tipos quando estamos executando testes. *Objetos Mock* (objetos simulados) são tipos específicos de dublês de teste que registram o que acontece durante um teste para que você possa afirmar que as ações corretas ocorreram.

Rust não tem objetos no mesmo sentido que outras linguagens têm objetos, e Rust não tem funcionalidade de objeto mock embutida na biblioteca padrão como algumas outras linguagens têm. No entanto, você pode definitivamente criar uma struct que servirá aos mesmos propósitos que um objeto mock.

Aqui está o cenário que testaremos: criaremos uma biblioteca que rastreia um valor em relação a um valor máximo e envia mensagens com base em quão próximo do valor máximo o valor atual está. Esta biblioteca poderia ser usada para manter o controle da cota de um usuário para o número de chamadas de API que eles podem fazer, por exemplo.

Nossa biblioteca fornecerá apenas a funcionalidade de rastrear o quão próximo do máximo um valor está e quais devem ser as mensagens em quais momentos. Espera-se que as aplicações que usam nossa biblioteca forneçam o mecanismo para enviar as mensagens: a aplicação poderia mostrar a mensagem ao usuário diretamente, enviar um e-mail, enviar uma mensagem de texto ou fazer outra coisa. A biblioteca não precisa saber esse detalhe. Tudo o que ela precisa é de algo que implemente uma trait que forneceremos, chamada `Messenger`. A Listagem 15-20 mostra o código da biblioteca.

<Listing number="15-20" file-name="src/lib.rs" caption="Uma biblioteca para acompanhar o quão próximo um valor está de um valor máximo e avisar quando o valor estiver em certos níveis">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Uma parte importante deste código é que a trait `Messenger` tem um método chamado `send` que recebe uma referência imutável para `self` e o texto da mensagem. Esta trait é a interface que nosso objeto mock precisa implementar para que o mock possa ser usado da mesma maneira que um objeto real. A outra parte importante é que queremos testar o comportamento do método `set_value` no `LimitTracker`. Podemos mudar o que passamos para o parâmetro `value`, mas `set_value` não retorna nada para fazermos asserções. Queremos ser capazes de dizer que se criarmos um `LimitTracker` com algo que implementa a trait `Messenger` e um valor específico para `max`, o mensageiro é instruído a enviar as mensagens apropriadas quando passamos números diferentes para `value`.

Precisamos de um objeto mock que, em vez de enviar um e-mail ou mensagem de texto quando chamamos `send`, apenas manterá o controle das mensagens que é instruído a enviar. Podemos criar uma nova instância do objeto mock, criar um `LimitTracker` que usa o objeto mock, chamar o método `set_value` em `LimitTracker` e então verificar se o objeto mock tem as mensagens que esperamos. A Listagem 15-21 mostra uma tentativa de implementar um objeto mock para fazer exatamente isso, mas o verificador de empréstimo não permitirá.

<Listing number="15-21" file-name="src/lib.rs" caption="Uma tentativa de implementar um `MockMessenger` que não é permitida pelo verificador de empréstimo">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Este código de teste define uma struct `MockMessenger` que tem um campo `sent_messages` com um `Vec` de valores `String` para manter o controle das mensagens que é instruído a enviar. Também definimos uma função associada `new` para tornar conveniente criar novos valores `MockMessenger` que começam com uma lista vazia de mensagens. Então implementamos a trait `Messenger` para `MockMessenger` para que possamos dar um `MockMessenger` a um `LimitTracker`. Na definição do método `send`, pegamos a mensagem passada como parâmetro e a armazenamos na lista de `sent_messages` de `MockMessenger`.

No teste, estamos testando o que acontece quando o `LimitTracker` é instruído a definir `value` para algo que é mais de 75 por cento do valor `max`. Primeiro, criamos um novo `MockMessenger`, que começará com uma lista vazia de mensagens. Então, criamos um novo `LimitTracker` e damos a ele uma referência ao novo `MockMessenger` e um valor `max` de `100`. Chamamos o método `set_value` no `LimitTracker` com um valor de `80`, que é mais de 75 por cento de 100. Então, afirmamos que a lista de mensagens que o `MockMessenger` está rastreando deve agora ter uma mensagem nela.

No entanto, há um problema com este teste, como mostrado aqui:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Não podemos modificar o `MockMessenger` para manter o controle das mensagens, porque o método `send` recebe uma referência imutável para `self`. Também não podemos aceitar a sugestão do texto de erro para usar `&mut self` tanto na implementação do método quanto na definição da trait. Não queremos mudar a trait `Messenger` apenas por causa dos testes. Em vez disso, precisamos encontrar uma maneira de fazer nosso código de teste funcionar corretamente com nosso design existente.

Esta é uma situação em que a mutabilidade interior pode ajudar! Armazenaremos as `sent_messages` dentro de um `RefCell<T>`, e então o método `send` será capaz de modificar `sent_messages` para armazenar as mensagens que vimos. A Listagem 15-22 mostra como isso se parece.

<Listing number="15-22" file-name="src/lib.rs" caption="Usando `RefCell<T>` para mutar um valor interno enquanto o valor externo é considerado imutável">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

O campo `sent_messages` é agora do tipo `RefCell<Vec<String>>` em vez de `Vec<String>`. Na função `new`, criamos uma nova instância de `RefCell<Vec<String>>` em torno do vetor vazio.

Para a implementação do método `send`, o primeiro parâmetro ainda é um empréstimo imutável de `self`, que corresponde à definição da trait. Chamamos `borrow_mut` no `RefCell<Vec<String>>` em `self.sent_messages` para obter uma referência mutável ao valor dentro do `RefCell<Vec<String>>`, que é o vetor. Então, podemos chamar `push` na referência mutável ao vetor para manter o controle das mensagens enviadas durante o teste.

A última mudança que temos que fazer é na asserção: para ver quantos itens estão no vetor interno, chamamos `borrow` no `RefCell<Vec<String>>` para obter uma referência imutável ao vetor.

Agora que você viu como usar `RefCell<T>`, vamos aprofundar em como ele funciona!

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### Rastreando Empréstimos em Tempo de Execução

Ao criar referências imutáveis e mutáveis, usamos a sintaxe `&` e `&mut`, respectivamente. Com `RefCell<T>`, usamos os métodos `borrow` e `borrow_mut`, que fazem parte da API segura que pertence a `RefCell<T>`. O método `borrow` retorna o tipo de ponteiro inteligente `Ref<T>`, e `borrow_mut` retorna o tipo de ponteiro inteligente `RefMut<T>`. Ambos os tipos implementam `Deref`, então podemos tratá-los como referências regulares.

O `RefCell<T>` mantém o controle de quantos ponteiros inteligentes `Ref<T>` e `RefMut<T>` estão atualmente ativos. Toda vez que chamamos `borrow`, o `RefCell<T>` aumenta sua contagem de quantos empréstimos imutáveis estão ativos. Quando um valor `Ref<T>` sai de escopo, a contagem de empréstimos imutáveis diminui em 1. Assim como as regras de empréstimo em tempo de compilação, `RefCell<T>` nos permite ter muitos empréstimos imutáveis ou um empréstimo mutável em qualquer ponto no tempo.

Se tentarmos violar essas regras, em vez de receber um erro de compilador como faríamos com referências, a implementação de `RefCell<T>` entrará em pânico em tempo de execução. A Listagem 15-23 mostra uma modificação da implementação de `send` na Listagem 15-22. Estamos deliberadamente tentando criar dois empréstimos mutáveis ativos para o mesmo escopo para ilustrar que `RefCell<T>` nos impede de fazer isso em tempo de execução.

<Listing number="15-23" file-name="src/lib.rs" caption="Criando duas referências mutáveis no mesmo escopo para ver que `RefCell<T>` entrará em pânico">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Criamos uma variável `one_borrow` para o ponteiro inteligente `RefMut<T>` retornado de `borrow_mut`. Então, criamos outro empréstimo mutável da mesma maneira na variável `two_borrow`. Isso faz duas referências mutáveis no mesmo escopo, o que não é permitido. Quando executamos os testes para nossa biblioteca, o código na Listagem 15-23 compilará sem erros, mas o teste falhará:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Observe que o código entrou em pânico com a mensagem `already borrowed: BorrowMutError`. É assim que `RefCell<T>` lida com violações das regras de empréstimo em tempo de execução.

Escolher capturar erros de empréstimo em tempo de execução em vez de tempo de compilação, como fizemos aqui, significa que você potencialmente encontraria erros em seu código mais tarde no processo de desenvolvimento: possivelmente não até que seu código fosse implantado em produção. Além disso, seu código incorreria em uma pequena penalidade de performance em tempo de execução como resultado de manter o controle dos empréstimos em tempo de execução em vez de tempo de compilação. No entanto, usar `RefCell<T>` torna possível escrever um objeto mock que pode se modificar para manter o controle das mensagens que viu enquanto você o está usando em um contexto onde apenas valores imutáveis são permitidos. Você pode usar `RefCell<T>` apesar de seus compromissos para obter mais funcionalidade do que referências regulares fornecem.

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>
<a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### Permitindo Múltiplos Donos de Dados Mutáveis

Uma maneira comum de usar `RefCell<T>` é em combinação com `Rc<T>`. Lembre-se de que `Rc<T>` permite que você tenha múltiplos donos de alguns dados, mas apenas dá acesso imutável a esses dados. Se você tiver um `Rc<T>` que contém um `RefCell<T>`, você pode obter um valor que pode ter múltiplos donos *e* que você pode mutar!

Por exemplo, relembre o exemplo da cons list na Listagem 15-18, onde usamos `Rc<T>` para permitir que várias listas compartilhassem a posse de outra lista. Como `Rc<T>` contém apenas valores imutáveis, não podemos mudar nenhum dos valores na lista depois de criá-los. Vamos adicionar `RefCell<T>` por sua capacidade de mudar os valores nas listas. A Listagem 15-24 mostra que, usando um `RefCell<T>` na definição de `Cons`, podemos modificar o valor armazenado em todas as listas.

<Listing number="15-24" file-name="src/main.rs" caption="Usando `Rc<RefCell<i32>>` para criar uma `List` que podemos mutar">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Criamos um valor que é uma instância de `Rc<RefCell<i32>>` e o armazenamos em uma variável chamada `value` para que possamos acessá-lo diretamente mais tarde. Então, criamos uma `List` em `a` com uma variante `Cons` que contém `value`. Precisamos clonar `value` para que tanto `a` quanto `value` tenham posse do valor interno `5` em vez de transferir a posse de `value` para `a` ou fazer `a` tomar emprestado de `value`.

Envolvemos a lista `a` em um `Rc<T>` para que, quando criarmos as listas `b` e `c`, ambas possam se referir a `a`, o que fizemos na Listagem 15-18.

Depois de criarmos as listas em `a`, `b` e `c`, queremos adicionar 10 ao valor em `value`. Fazemos isso chamando `borrow_mut` em `value`, o que usa o recurso de desreferência automática que discutimos em ["Onde está o Operador `->`?"][wheres-the---operator] no Capítulo 5 para desreferenciar o `Rc<T>` para o valor interno `RefCell<T>`. O método `borrow_mut` retorna um ponteiro inteligente `RefMut<T>`, e usamos o operador de desreferência nele e mudamos o valor interno.

Quando imprimimos `a`, `b` e `c`, podemos ver que todos eles têm o valor modificado de `15` em vez de `5`:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Esta técnica é muito legal! Usando `RefCell<T>`, temos um valor `List` externamente imutável. Mas podemos usar os métodos em `RefCell<T>` que fornecem acesso à sua mutabilidade interior para que possamos modificar nossos dados quando precisarmos. As verificações em tempo de execução das regras de empréstimo nos protegem de corridas de dados, e às vezes vale a pena trocar um pouco de velocidade por essa flexibilidade em nossas estruturas de dados. Note que `RefCell<T>` não funciona para código multithreaded! `Mutex<T>` é a versão thread-safe de `RefCell<T>`, e discutiremos `Mutex<T>` no Capítulo 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
