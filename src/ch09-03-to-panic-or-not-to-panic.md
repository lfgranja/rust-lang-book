## Entrar em `panic!` ou Não Entrar em `panic!`

Então, como você decide quando deve chamar `panic!` e quando deve retornar
`Result`? Quando o código entra em pânico, não há como se recuperar. Você
poderia chamar `panic!` para qualquer situação de erro, quer haja uma maneira
possível de se recuperar ou não, mas então você está tomando a decisão de que
uma situação é irrecuperável em nome do código chamador. Quando você escolhe
retornar um valor `Result`, você dá opções ao código chamador. O código
chamador pode escolher tentar se recuperar de uma maneira que seja apropriada
para sua situação, ou pode decidir que um valor `Err` neste caso é
irrecuperável, então pode chamar `panic!` e transformar seu erro recuperável em
um irrecuperável. Portanto, retornar `Result` é uma boa escolha padrão quando
você está definindo uma função que pode falhar.

Em situações como exemplos, código de protótipo e testes, é mais apropriado
escrever código que entra em pânico em vez de retornar um `Result`. Vamos
explorar o porquê, depois discutir situações em que o compilador não pode dizer
que a falha é impossível, mas você como humano pode. O capítulo terminará com
algumas diretrizes gerais sobre como decidir se deve entrar em pânico no código
da biblioteca.

### Exemplos, Código de Protótipo e Testes

Quando você está escrevendo um exemplo para ilustrar algum conceito, incluir
também código robusto de tratamento de erros pode tornar o exemplo menos claro.
Em exemplos, entende-se que uma chamada a um método como `unwrap` que pode
entrar em pânico é destinada como um espaço reservado para a maneira como você
desejaria que seu aplicativo tratasse erros, o que pode diferir com base no que
o restante do seu código está fazendo.

Da mesma forma, os métodos `unwrap` e `expect` são muito úteis quando você está
prototipando e ainda não está pronto para decidir como lidar com erros. Eles
deixam marcadores claros em seu código para quando você estiver pronto para
tornar seu programa mais robusto.

Se uma chamada de método falhar em um teste, você desejaria que o teste inteiro
falhasse, mesmo que esse método não seja a funcionalidade em teste. Como
`panic!` é como um teste é marcado como falha, chamar `unwrap` ou `expect` é
exatamente o que deve acontecer.

<!-- Old headings. Do not remove or links may break. -->

<a id="cases-in-which-you-have-more-information-than-the-compiler"></a>

### Quando Você Tem Mais Informação Que o Compilador

Também seria apropriado chamar `expect` quando você tem alguma outra lógica que
garante que o `Result` terá um valor `Ok`, mas a lógica não é algo que o
compilador entende. Você ainda terá um valor `Result` que precisa lidar:
Qualquer operação que você esteja chamando ainda tem a possibilidade de falhar
em geral, mesmo que seja logicamente impossível em sua situação específica. Se
você puder garantir inspecionando manualmente o código que nunca terá uma
variante `Err`, é perfeitamente aceitável chamar `expect` e documentar o motivo
pelo qual você acha que nunca terá uma variante `Err` no texto do argumento.
Aqui está um exemplo:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Estamos criando uma instância `IpAddr` analisando uma string codificada.
Podemos ver que `127.0.0.1` é um endereço IP válido, então é aceitável usar
`expect` aqui. No entanto, ter uma string válida codificada não altera o tipo
de retorno do método `parse`: ainda recebemos um valor `Result`, e o
compilador ainda nos fará lidar com o `Result` como se a variante `Err` fosse
uma possibilidade, porque o compilador não é inteligente o suficiente para ver
que esta string é sempre um endereço IP válido. Se a string de endereço IP
viesse de um usuário em vez de ser codificada no programa e, portanto,
tivesse uma possibilidade de falha, definitivamente gostaríamos de lidar com o
`Result` de uma maneira mais robusta. Mencionar a suposição de que este
endereço IP é codificado nos levará a mudar `expect` para um código de
tratamento de erros melhor se, no futuro, precisarmos obter o endereço IP de
alguma outra fonte.

### Diretrizes para Tratamento de Erros

É aconselhável que seu código entre em pânico quando for possível que seu
código termine em um estado ruim. Neste contexto, um _estado ruim_ é quando
alguma suposição, garantia, contrato ou invariante foi quebrada, como quando
valores inválidos, valores contraditórios ou valores ausentes são passados
para o seu código — mais um ou mais dos seguintes:

- O estado ruim é algo inesperado, ao contrário de algo que provavelmente
  acontecerá ocasionalmente, como um usuário inserindo dados no formato errado.
- Seu código após este ponto precisa confiar em não estar neste estado ruim, em
  vez de verificar o problema a cada passo.
- Não há uma boa maneira de codificar essa informação nos tipos que você usa.
  Trabalharemos em um exemplo do que queremos dizer em [“Codificando Estados e
  Comportamentos como Tipos”][encoding]<!-- ignore --> no Capítulo 18.

Se alguém chamar seu código e passar valores que não fazem sentido, é melhor
retornar um erro se você puder para que o usuário da biblioteca possa decidir o
que quer fazer nesse caso. No entanto, em casos onde continuar pode ser
inseguro ou prejudicial, a melhor escolha pode ser chamar `panic!` e alertar a
pessoa usando sua biblioteca sobre o bug em seu código para que ela possa
consertá-lo durante o desenvolvimento. Da mesma forma, `panic!` é muitas vezes
apropriado se você estiver chamando código externo que está fora de seu controle
e retorna um estado inválido que você não tem como corrigir.

No entanto, quando a falha é esperada, é mais apropriado retornar um `Result`
do que fazer uma chamada `panic!`. Exemplos incluem um analisador recebendo
dados malformados ou uma solicitação HTTP retornando um status que indica que
você atingiu um limite de taxa. Nesses casos, retornar um `Result` indica que a
falha é uma possibilidade esperada que o código chamador deve decidir como
tratar.

Quando seu código executa uma operação que pode colocar um usuário em risco se
for chamada usando valores inválidos, seu código deve verificar se os valores
são válidos primeiro e entrar em pânico se os valores não forem válidos. Isso é
principalmente por razões de segurança: Tentar operar em dados inválidos pode
expor seu código a vulnerabilidades. Essa é a principal razão pela qual a
biblioteca padrão chamará `panic!` se você tentar um acesso de memória fora dos
limites: Tentar acessar a memória que não pertence à estrutura de dados atual é
um problema de segurança comum. As funções geralmente têm _contratos_: Seu
comportamento só é garantido se as entradas atenderem a requisitos específicos.
Entrar em pânico quando o contrato é violado faz sentido porque uma violação de
contrato sempre indica um bug do lado do chamador, e não é um tipo de erro que
você deseja que o código chamador tenha que lidar explicitamente. De fato, não
há maneira razoável para o código chamador se recuperar; os _programadores_
chamadores precisam consertar o código. Contratos para uma função,
especialmente quando uma violação causará um pânico, devem ser explicados na
documentação da API para a função.

No entanto, ter muitas verificações de erro em todas as suas funções seria
verboso e irritante. Felizmente, você pode usar o sistema de tipos do Rust (e,
portanto, a verificação de tipo feita pelo compilador) para fazer muitas das
verificações para você. Se sua função tiver um tipo específico como parâmetro,
você pode prosseguir com a lógica do seu código sabendo que o compilador já
garantiu que você tem um valor válido. Por exemplo, se você tiver um tipo em
vez de um `Option`, seu programa espera ter _algo_ em vez de _nada_. Seu
código então não precisa lidar com dois casos para as variantes `Some` e `None`:
Ele terá apenas um caso para ter definitivamente um valor. O código que tenta
passar nada para sua função nem sequer compilará, então sua função não precisa
verificar esse caso em tempo de execução. Outro exemplo é usar um tipo inteiro
sem sinal, como `u32`, que garante que o parâmetro nunca seja negativo.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-custom-types-for-validation"></a>

### Criando Tipos Personalizados para Validação

Vamos levar a ideia de usar o sistema de tipos do Rust para garantir que temos
um valor válido um passo adiante e ver como criar um tipo personalizado para
validação. Lembre-se do jogo de adivinhação no Capítulo 2, no qual nosso código
pedia ao usuário para adivinhar um número entre 1 e 100. Nunca validamos se o
palpite do usuário estava entre esses números antes de verificá-lo em relação ao
nosso número secreto; apenas validamos que o palpite era positivo. Neste caso,
as consequências não foram muito terríveis: Nossa saída de "Muito alto" ou
"Muito baixo" ainda estaria correta. Mas seria uma melhoria útil guiar o
usuário em direção a palpites válidos e ter um comportamento diferente quando o
usuário adivinha um número que está fora do intervalo em comparação com quando
o usuário digita, por exemplo, letras.

Uma maneira de fazer isso seria analisar o palpite como um `i32` em vez de
apenas um `u32` para permitir números potencialmente negativos e, em seguida,
adicionar uma verificação se o número está no intervalo, assim:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

A expressão `if` verifica se nosso valor está fora do intervalo, informa o
usuário sobre o problema e chama `continue` para iniciar a próxima iteração do
loop e pedir outro palpite. Após a expressão `if`, podemos prosseguir com as
comparações entre `guess` e o número secreto sabendo que `guess` está entre 1 e
100.

No entanto, esta não é uma solução ideal: Se fosse absolutamente crítico que o
programa operasse apenas em valores entre 1 e 100, e tivesse muitas funções com
esse requisito, ter uma verificação como essa em cada função seria tedioso (e
poderia impactar o desempenho).

Em vez disso, podemos criar um novo tipo em um módulo dedicado e colocar as
validações em uma função para criar uma instância do tipo em vez de repetir as
validações em todos os lugares. Dessa forma, é seguro para as funções usarem o
novo tipo em suas assinaturas e usar com confiança os valores que recebem. A
Listagem 9-13 mostra uma maneira de definir um tipo `Guess` que só criará uma
instância de `Guess` se a função `new` receber um valor entre 1 e 100.

<Listing number="9-13" caption="Um tipo `Guess` que só continuará com valores entre 1 e 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Observe que este código em *src/guessing_game.rs* depende da adição de uma
declaração de módulo `mod guessing_game;` em *src/lib.rs* que não mostramos
aqui. Dentro do arquivo deste novo módulo, definimos uma struct chamada `Guess`
que tem um campo chamado `value` que contém um `i32`. É aqui que o número será
armazenado.

Em seguida, implementamos uma função associada chamada `new` em `Guess` que
cria instâncias de valores `Guess`. A função `new` é definida para ter um
parâmetro chamado `value` do tipo `i32` e retornar um `Guess`. O código no
corpo da função `new` testa `value` para garantir que esteja entre 1 e 100. Se
`value` não passar neste teste, fazemos uma chamada `panic!`, o que alertará o
programador que está escrevendo o código chamador de que ele tem um bug que
precisa consertar, porque criar um `Guess` com um `value` fora deste intervalo
violaria o contrato em que `Guess::new` está confiando. As condições em que
`Guess::new` pode entrar em pânico devem ser discutidas em sua documentação de
API pública; abordaremos convenções de documentação indicando a possibilidade
de um `panic!` na documentação da API que você criará no Capítulo 14. Se
`value` passar no teste, criamos um novo `Guess` com seu campo `value` definido
para o parâmetro `value` e retornamos o `Guess`.

Em seguida, implementamos um método chamado `value` que empresta `self`, não
tem outros parâmetros e retorna um `i32`. Esse tipo de método às vezes é
chamado de _getter_ porque seu objetivo é obter alguns dados de seus campos e
retorná-los. Este método público é necessário porque o campo `value` da struct
`Guess` é privado. É importante que o campo `value` seja privado para que o
código usando a struct `Guess` não tenha permissão para definir `value`
diretamente: O código fora do módulo `guessing_game` _deve_ usar a função
`Guess::new` para criar uma instância de `Guess`, garantindo assim que não haja
maneira de um `Guess` ter um `value` que não tenha sido verificado pelas
condições na função `Guess::new`.

Uma função que tem um parâmetro ou retorna apenas números entre 1 e 100 poderia
então declarar em sua assinatura que recebe ou retorna um `Guess` em vez de um
`i32` e não precisaria fazer nenhuma verificação adicional em seu corpo.

## Resumo

Os recursos de tratamento de erros do Rust são projetados para ajudá-lo a
escrever código mais robusto. A macro `panic!` sinaliza que seu programa está
em um estado que não pode lidar e permite que você diga ao processo para parar
em vez de tentar prosseguir com valores inválidos ou incorretos. A enumeração
`Result` usa o sistema de tipos do Rust para indicar que as operações podem
falhar de uma maneira que seu código poderia se recuperar. Você pode usar
`Result` para dizer ao código que chama seu código que ele precisa lidar com o
potencial sucesso ou falha também. Usar `panic!` e `Result` nas situações
apropriadas tornará seu código mais confiável diante de problemas inevitáveis.

Agora que você viu maneiras úteis de como a biblioteca padrão usa genéricos com
as enumerações `Option` e `Result`, falaremos sobre como os genéricos funcionam
e como você pode usá-los em seu código.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
