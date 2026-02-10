# Variáveis e Mutabilidade

Como mencionado na seção [[ch02-00-guessing-game-tutorial.md#storing-values-with-variables|“Armazenando Valores com Variáveis”]]<!-- ignore -->, por padrão,
variáveis são imutáveis. Este é um dos muitos empurrõezinhos que o Rust lhe dá para escrever
seu código de uma maneira que tire vantagem da segurança e fácil concorrência que
o Rust oferece. No entanto, você ainda tem a opção de tornar suas variáveis mutáveis.
Vamos explorar como e por que o Rust encoraja você a favorecer a imutabilidade e por que
às vezes você pode querer optar por não usá-la.

Quando uma variável é imutável, uma vez que um valor é vinculado a um nome, você não pode mudar
esse valor. Para ilustrar isso, gere um novo projeto chamado _variables_ no
seu diretório _projects_ usando `cargo new variables`.

Então, no seu novo diretório _variables_, abra _src/main.rs_ e substitua seu
código pelo seguinte código, que não compilará ainda:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Salve e execute o programa usando `cargo run`. Você deve receber uma mensagem de erro
referente a um erro de imutabilidade, como mostrado nesta saída:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Este exemplo mostra como o compilador ajuda você a encontrar erros em seus programas.
Erros de compilador podem ser frustrantes, mas na verdade eles significam apenas que seu programa
ainda não está fazendo com segurança o que você quer que ele faça; eles _não_ significam que você
não é um bom programador! Rustaceans experientes ainda recebem erros de compilador.

Você recebeu a mensagem de erro `` cannot assign twice to immutable variable `x` `` porque você tentou atribuir um segundo valor à variável imutável `x`.

É importante que recebamos erros em tempo de compilação quando tentamos mudar um
valor que é designado como imutável, porque essa situação pode levar a
bugs. Se uma parte do nosso código opera na suposição de que um valor
nunca mudará e outra parte do nosso código muda esse valor, é possível
que a primeira parte do código não faça o que foi projetada para fazer. A causa
desse tipo de bug pode ser difícil de rastrear depois do fato, especialmente
quando a segunda peça de código muda o valor apenas _às vezes_. O compilador Rust
garante que quando você afirma que um valor não mudará, ele realmente
não mudará, então você não precisa acompanhá-lo você mesmo. Seu código é assim
mais fácil de raciocinar.

Mas a mutabilidade pode ser muito útil e pode tornar o código mais conveniente de escrever.
Embora as variáveis sejam imutáveis por padrão, você pode torná-las mutáveis
adicionando `mut` na frente do nome da variável como você fez no [[ch02-00-guessing-game-tutorial.md#storing-values-with-variables|Capítulo 2]]<!-- ignore -->. Adicionar `mut` também transmite
intenção aos futuros leitores do código, indicando que outras partes do código
estarão mudando o valor desta variável.

Por exemplo, vamos mudar _src/main.rs_ para o seguinte:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Quando executamos o programa agora, obtemos isto:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Temos permissão para mudar o valor vinculado a `x` de `5` para `6` quando `mut` é
usado. Em última análise, decidir se deve usar mutabilidade ou não depende de você e
do que você acha que é mais claro naquela situação específica.

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

### Declarando Constantes

Como variáveis imutáveis, _constantes_ são valores que estão vinculados a um nome e
não têm permissão para mudar, mas existem algumas diferenças entre constantes
e variáveis.

Primeiro, você não tem permissão para usar `mut` com constantes. Constantes não são apenas
imutáveis por padrão—elas são sempre imutáveis. Você declara constantes usando a
palavra-chave `const` em vez da palavra-chave `let`, e o tipo do valor _deve_
ser anotado. Cobriremos tipos e anotações de tipo na próxima seção,
[[ch03-02-data-types.md#data-types|“Tipos de Dados”]]<!-- ignore -->, então não se preocupe com os detalhes
agora. Apenas saiba que você deve sempre anotar o tipo.

Constantes podem ser declaradas em qualquer escopo, incluindo o escopo global, o que as torna
úteis para valores que muitas partes do código precisam conhecer.

A última diferença é que constantes podem ser definidas apenas para uma expressão constante,
não o resultado de um valor que só poderia ser computado em tempo de execução.

Aqui está um exemplo de uma declaração de constante:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

O nome da constante é `THREE_HOURS_IN_SECONDS`, e seu valor é definido para o
resultado da multiplicação de 60 (o número de segundos em um minuto) por 60 (o número
de minutos em uma hora) por 3 (o número de horas que queremos contar neste
programa). A convenção de nomenclatura do Rust para constantes é usar tudo em maiúsculas com
sublinhados entre as palavras. O compilador é capaz de avaliar um conjunto limitado de
operações em tempo de compilação, o que nos permite escolher escrever este valor de uma
maneira que é mais fácil de entender e verificar, em vez de definir esta constante
para o valor 10.800. Veja a [seção de avaliação de constantes da Referência do Rust](https://doc.rust-lang.org/reference/const_eval.html) para mais informações sobre quais operações podem ser usadas
ao declarar constantes.

Constantes são válidas por todo o tempo que um programa executa, dentro do escopo no
qual elas foram declaradas. Essa propriedade torna as constantes úteis para valores no
domínio da sua aplicação que múltiplas partes do programa podem precisar conhecer,
como o número máximo de pontos que qualquer jogador de um jogo pode
ganhar, ou a velocidade da luz.

Nomear valores hardcoded usados em todo o seu programa como constantes é útil para
transmitir o significado desse valor para futuros mantenedores do código. Também
ajuda a ter apenas um lugar em seu código que você precisaria mudar se o
valor hardcoded precisasse ser atualizado no futuro.

### Sombreamento (Shadowing)

Como você viu no tutorial do jogo de adivinhação no [[ch02-00-guessing-game-tutorial.md#comparing-the-guess-to-the-secret-number|Capítulo 2]]<!-- ignore -->, você pode declarar uma
nova variável com o mesmo nome de uma variável anterior. Rustaceans dizem que a
primeira variável é _sombreada_ (shadowed) pela segunda, o que significa que a segunda
variável é o que o compilador verá quando você usar o nome da variável.
Com efeito, a segunda variável ofusca a primeira, tomando qualquer uso do
nome da variável para si mesma até que ela mesma seja sombreada ou o escopo termine.
Podemos sombrear uma variável usando o mesmo nome de variável e repetindo o
uso da palavra-chave `let` da seguinte forma:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Este programa primeiro vincula `x` a um valor de `5`. Então, ele cria uma nova variável
`x` repetindo `let x =`, pegando o valor original e adicionando `1` para que
o valor de `x` seja `6`. Então, dentro de um escopo interno criado com as chaves,
a terceira instrução `let` também sombreia `x` e cria uma nova
variável, multiplicando o valor anterior por `2` para dar a `x` um valor de `12`.
Quando esse escopo termina, o sombreamento interno termina e `x` retorna a ser `6`.
Quando executamos este programa, ele produzirá o seguinte:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Sombreamento é diferente de marcar uma variável como `mut` porque teremos um
erro em tempo de compilação se acidentalmente tentarmos reatribuir a esta variável sem
usar a palavra-chave `let`. Usando `let`, podemos realizar algumas transformações
em um valor, mas ter a variável imutável após essas transformações terem
sido concluídas.

A outra diferença entre `mut` e sombreamento é que, como estamos
efetivamente criando uma nova variável quando usamos a palavra-chave `let` novamente, podemos
mudar o tipo do valor, mas reutilizar o mesmo nome. Por exemplo, digamos que nosso
programa peça a um usuário para mostrar quantos espaços eles querem entre algum texto
digitando caracteres de espaço, e então queremos armazenar essa entrada como um número:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

A primeira variável `spaces` é um tipo string, e a segunda variável `spaces`
é um tipo número. O sombreamento, assim, nos poupa de ter que inventar
nomes diferentes, como `spaces_str` e `spaces_num`; em vez disso, podemos reutilizar
o nome `spaces` mais simples. No entanto, se tentarmos usar `mut` para isso, como mostrado
aqui, teremos um erro em tempo de compilação:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

O erro diz que não temos permissão para mutar o tipo de uma variável:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Agora que exploramos como as variáveis funcionam, vamos olhar para mais tipos de dados que elas
podem ter.
