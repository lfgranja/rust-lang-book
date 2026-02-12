## Armazenando Chaves com Valores Associados em Hash Maps

A última de nossas coleções comuns é o *hash map*. O tipo `HashMap<K, V>`
armazena um mapeamento de chaves do tipo `K` para valores do tipo `V` usando
uma *função de hashing*, que determina como ele coloca essas chaves e valores
na memória. Muitas linguagens de programação diferentes suportam este tipo de
estrutura de dados, mas muitas vezes usam um nome diferente, como hash, map,
object, hash table, dictionary ou associative array, apenas para citar alguns.

Hash maps são úteis quando você quer procurar dados não usando um índice, como
você pode com vetores, mas usando uma chave que pode ser de qualquer tipo. Por
exemplo, em um jogo, você poderia acompanhar a pontuação de cada equipe em um
hash map no qual cada chave é o nome de uma equipe e os valores são a pontuação
de cada equipe. Dado o nome de uma equipe, você pode recuperar sua pontuação.

Passaremos pela API básica de hash maps nesta seção, mas muitas outras
guloseimas estão escondidas nas funções definidas em `HashMap<K, V>` pela
biblioteca padrão. Como sempre, verifique a documentação da biblioteca padrão
para mais informações.

### Criando um Novo Hash Map

Uma maneira de criar um hash map vazio é usar `new` e adicionar elementos com
`insert`. Na Listagem 8-20, estamos acompanhando as pontuações de duas equipes
cujos nomes são *Blue* e *Yellow*. A equipe Blue começa com 10 pontos, e a
equipe Yellow começa com 50.

<Listing number="8-20" caption="Criando um novo hash map e inserindo algumas chaves e valores">

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

</Listing>

Observe que precisamos primeiro `use` o `HashMap` da parte de coleções da
biblioteca padrão. De nossas três coleções comuns, esta é a menos usada, então
não está incluída nos recursos trazidos automaticamente para o escopo no
prelúdio. Hash maps também têm menos suporte da biblioteca padrão; não há, por
exemplo, um macro embutido para construí-los.

Assim como vetores, hash maps armazenam seus dados na heap. Este `HashMap` tem
chaves do tipo `String` e valores do tipo `i32`. Como vetores, hash maps são
homogêneos: todas as chaves devem ter o mesmo tipo entre si, e todos os valores
devem ter o mesmo tipo.

### Acessando Valores em um Hash Map

Podemos obter um valor do hash map fornecendo sua chave para o método `get`,
como mostrado na Listagem 8-21.

<Listing number="8-21" caption="Acessando a pontuação da equipe Blue armazenada no hash map">

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0);
```

</Listing>

Aqui, `score` terá o valor associado à equipe Blue, e o resultado será `10`. O
método `get` retorna um `Option<&V>`; se não houver valor para essa chave no
hash map, `get` retornará `None`. Este programa lida com o `Option` chamando
`copied` para obter um `Option<i32>` em vez de um `Option<&i32>`, e então
`unwrap_or` para definir `score` como zero se `scores` não tiver uma entrada
para a chave.

Podemos iterar sobre cada par chave/valor em um hash map de maneira semelhante
à que fazemos com vetores, usando um loop `for`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{key}: {value}");
}
```

Este código imprimirá cada par em uma ordem arbitrária:

```text
Yellow: 50
Blue: 10
```

### Hash Maps e Posse

Para tipos que implementam a trait `Copy`, como `i32`, os valores são copiados
para o hash map. Para valores possuídos como `String`, os valores serão movidos
e o hash map será o proprietário desses valores, como demonstrado na Listagem
8-22.

<Listing number="8-22" caption="Mostrando que chaves e valores são possuídos pelo hash map uma vez que são inseridos">

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name e field_value são inválidos neste ponto, tente usá-los e
// veja que erro de compilador você obtém!
```

</Listing>

Não podemos usar as variáveis `field_name` e `field_value` depois que elas
foram movidas para o hash map com a chamada para `insert`.

Se inserirmos referências a valores no hash map, os valores não serão movidos
para o hash map. Os valores que as referências apontam devem ser válidos pelo
menos enquanto o hash map for válido. Falaremos mais sobre essas questões na
seção ["Validando Referências com Lifetimes"][validating-references-with-lifetimes]<!-- ignore --> no Capítulo 10.

### Atualizando um Hash Map

Embora o número de pares chave e valor seja expansível, cada chave única pode
ter apenas um valor associado a ela por vez (mas não vice-versa: por exemplo,
tanto a equipe Blue quanto a equipe Yellow poderiam ter valor 10 armazenado no
hash map `scores`).

Quando você quer mudar os dados em um hash map, você tem que decidir como lidar
com o caso em que uma chave já tem um valor atribuído. Você poderia substituir
o valor antigo pelo novo valor, desconsiderando completamente o valor antigo.
Você poderia manter o valor antigo e ignorar o novo valor, adicionando o novo
valor apenas se a chave *não* tiver já um valor. Ou você poderia combinar o
valor antigo e o novo valor. Vamos ver como fazer cada um desses!

#### Sobrescrevendo um Valor

Se inserirmos uma chave e um valor em um hash map e depois inserirmos essa
mesma chave com um valor diferente, o valor associado a essa chave será
substituído. Embora o código na Listagem 8-23 chame `insert` duas vezes, o hash
map conterá apenas um par chave/valor porque estamos inserindo o valor para a
chave da equipe Blue ambas as vezes.

<Listing number="8-23" caption="Substituindo um valor armazenado com uma chave particular">

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

</Listing>

Este código imprimirá `{"Blue": 25}`. O valor original de `10` foi sobrescrito.

#### Adicionando uma Chave e Valor Apenas Se uma Chave Não Estiver Presente

É comum verificar se uma chave particular já tem um valor com ela dentro do hash
map e então tomar as seguintes ações: se a chave existir no hash map, o valor
existente deve permanecer como está. Se a chave não existir, insira-a e um
valor para ela.

Hash maps têm uma API especial para isso chamada `entry` que recebe a chave que
você deseja verificar como parâmetro. O valor de retorno do método `entry` é um
enum chamado `Entry` que representa um valor que pode ou não existir. Vamos
dizer que queremos verificar se a chave para a equipe Yellow tem um valor
associado a ela. Se não tiver, queremos inserir o valor 50, e o mesmo para a
equipe Blue. Usando a API `entry`, o código se parece com a Listagem 8-24.

<Listing number="8-24" caption="Usando o método `entry` para inserir apenas se a chave ainda não tiver um valor">

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

</Listing>

O método `or_insert` em `Entry` é definido para retornar uma referência mutável
ao valor para a chave `Entry` correspondente se essa chave existir, e se não,
insere o parâmetro para esta chave e retorna uma referência mutável ao novo
valor. Esta técnica é muito mais limpa do que escrever a lógica nós mesmos e,
além disso, joga mais bem com o borrow checker.

Rodar o código na Listagem 8-24 imprimirá `{"Yellow": 50, "Blue": 10}`. A
primeira chamada para `entry` inserirá a chave para a equipe Yellow com o valor
50 porque a equipe Yellow não tem um valor. A segunda chamada para `entry` não
mudará o hash map porque a equipe Blue já tem o valor 10.

#### Atualizando um Valor com Base no Valor Antigo

Outro caso de uso comum para hash maps é procurar o valor de uma chave e depois
atualizá-lo com base no valor antigo. Por exemplo, a Listagem 8-25 mostra um
código que conta quantas vezes cada palavra aparece em algum texto. Usamos um
hash map com as palavras como chaves e incrementamos o valor para acompanhar
quantas vezes vimos essa palavra. Se for a primeira vez que vimos uma palavra,
primeiro inseriremos o valor 0.

<Listing number="8-25" caption="Contando ocorrências de palavras usando um hash map que armazena palavras e contagens">

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

</Listing>

Este código imprimirá `{"world": 2, "hello": 1, "wonderful": 1}`. Você pode ver
os mesmos pares chave/valor impressos em uma ordem diferente: lembre-se da
seção ["Acessando Valores em um Hash Map"][access]<!-- ignore --> que iterar
sobre um hash map acontece em uma ordem arbitrária.

O método `split_whitespace` retorna um iterador sobre sub-fatias, separadas por
espaço em branco, do valor em `text`. O método `or_insert` retorna uma
referência mutável (`&mut V`) ao valor para a chave especificada. Aqui
armazenamos essa referência mutável na variável `count`, então, para atribuir a
esse valor, devemos primeiro desreferenciar `count` usando o asterisco (`*`). A
referência mutável sai de escopo no final do loop `for`, então todas essas
mudanças são seguras e permitidas pelas regras de empréstimo.

### Funções de Hashing

Por padrão, `HashMap` usa uma função de hashing chamada *SipHash* que pode
fornecer resistência a ataques de negação de serviço (DoS) envolvendo tabelas
hash [^siphash]<!-- ignore -->. Este não é o algoritmo de hashing mais rápido
disponível, mas a compensação por melhor segurança que vem com a queda no
desempenho vale a pena. Se você perfilar seu código e achar que a função de
hashing padrão é muito lenta para seus propósitos, você pode mudar para outra
função especificando um hasher diferente. Um *hasher* é um tipo que implementa
a trait `BuildHasher`. Falaremos sobre traits e como implementá-las no
Capítulo 10. Você não precisa necessariamente implementar seu próprio hasher do
zero; [crates.io](https://crates.io/) tem bibliotecas compartilhadas por
outros membros da comunidade Rust que fornecem hashers implementando muitos
algoritmos de hashing comuns.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Resumo

Vetores, strings e hash maps fornecerão uma grande quantidade de
funcionalidade necessária em programas quando você precisa armazenar, acessar e
modificar dados. Aqui estão alguns exercícios que você deve estar equipado para
resolver agora:

1. Dada uma lista de inteiros, use um vetor e retorne a mediana (quando
   ordenada, o valor na posição do meio) e a moda (o valor que ocorre com mais
   frequência; um hash map será útil aqui) da lista.
2. Converta strings para pig latin. A primeira consoante de cada palavra é
   movida para o final da palavra e “ay” é adicionado, então “first” torna-se
   “irst-fay”. Palavras que começam com uma vogal têm “hay” adicionado ao final
   (“apple” torna-se “apple-hay”). Tenha em mente os detalhes sobre codificação
   UTF-8!
3. Usando um hash map e vetores, crie uma interface de texto para permitir que
   um usuário adicione nomes de funcionários a um departamento em uma empresa.
   Por exemplo, “Adicione Sally à Engenharia” ou “Adicione Amir a Vendas”.
   Então deixe o usuário recuperar uma lista de todas as pessoas em um
   departamento ou todas as pessoas na empresa por departamento, ordenadas
   alfabeticamente.

A documentação da API da biblioteca padrão descreve métodos que vetores, strings
e hash maps têm que serão úteis para esses exercícios!

Estamos entrando em programas mais complexos nos quais as operações podem
falhar, então é um momento perfeito para discutir tratamento de erros. Faremos
isso a seguir!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validando-referências-com-lifetimes
[access]: #acessando-valores-em-um-hash-map
