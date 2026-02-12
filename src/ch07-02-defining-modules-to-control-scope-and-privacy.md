<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## Controlando Escopo e Privacidade com Módulos

Nesta seção, falaremos sobre módulos e outras partes do sistema de módulos,
nomeadamente *caminhos*, que permitem nomear itens; a palavra-chave `use` que traz um
caminho para o escopo; e a palavra-chave `pub` para tornar itens públicos. Também discutiremos
a palavra-chave `as`, pacotes externos e o operador glob.

### Folha de Dicas de Módulos

Antes de entrarmos nos detalhes de módulos e caminhos, aqui fornecemos uma referência rápida
sobre como módulos, caminhos, a palavra-chave `use` e a palavra-chave `pub` funcionam no
compilador, e como a maioria dos desenvolvedores organiza seu código. Passaremos
por exemplos de cada uma dessas regras ao longo deste capítulo, mas este é um
ótimo lugar para consultar como um lembrete de como os módulos funcionam.

- **Comece da raiz do crate**: Ao compilar um crate, o compilador primeiro
  olha no arquivo raiz do crate (geralmente *src/lib.rs* para um crate de biblioteca e
  *src/main.rs* para um crate binário) por código para compilar.
- **Declarando módulos**: No arquivo raiz do crate, você pode declarar novos módulos;
  digamos que você declare um módulo "garden" com `mod garden;`. O compilador procurará
  pelo código do módulo nesses lugares:
  - Inline, dentro de chaves que substituem o ponto e vírgula após `mod
    garden`
  - No arquivo *src/garden.rs*
  - No arquivo *src/garden/mod.rs*
- **Declarando submódulos**: Em qualquer arquivo diferente da raiz do crate, você pode
  declarar submódulos. Por exemplo, você pode declarar `mod vegetables;` em
  *src/garden.rs*. O compilador procurará pelo código do submódulo dentro do
  diretório nomeado para o módulo pai nesses lugares:
  - Inline, diretamente após `mod vegetables`, dentro de chaves em vez
    do ponto e vírgula
  - No arquivo *src/garden/vegetables.rs*
  - No arquivo *src/garden/vegetables/mod.rs*
- **Caminhos para código em módulos**: Uma vez que um módulo faz parte do seu crate, você pode
  se referir ao código nesse módulo de qualquer outro lugar no mesmo crate, desde que
  as regras de privacidade permitam, usando o caminho para o código. Por exemplo, um
  tipo `Asparagus` no módulo de vegetais do jardim seria encontrado em
  `crate::garden::vegetables::Asparagus`.
- **Privado vs. público**: O código dentro de um módulo é privado de seus módulos pais
  por padrão. Para tornar um módulo público, declare-o com `pub mod`
  em vez de `mod`. Para tornar itens dentro de um módulo público públicos também, use
  `pub` antes de suas declarações.
- **A palavra-chave `use`**: Dentro de um escopo, a palavra-chave `use` cria atalhos para
  itens para reduzir a repetição de caminhos longos. Em qualquer escopo que possa se referir a
  `crate::garden::vegetables::Asparagus`, você pode criar um atalho com `use
  crate::garden::vegetables::Asparagus;`, e a partir de então você só precisa
  escrever `Asparagus` para fazer uso desse tipo no escopo.

Aqui, criamos um crate binário chamado `backyard` que ilustra essas regras.
O diretório do crate, também chamado *backyard*, contém esses arquivos e
diretórios:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

O arquivo raiz do crate neste caso é *src/main.rs*, e ele contém:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

A linha `pub mod garden;` diz ao compilador para incluir o código que ele encontrar em
*src/garden.rs*, que é:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Aqui, `pub mod vegetables;` significa que o código em *src/garden/vegetables.rs* é
incluído também. Esse código é:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Agora vamos entrar nos detalhes dessas regras e demonstrá-las em ação!

### Agrupando Código Relacionado em Módulos

_Módulos_ nos permitem organizar código dentro de um crate para legibilidade e fácil reutilização.
Módulos também nos permitem controlar a _privacidade_ dos itens porque o código dentro de um
módulo é privado por padrão. Itens privados são detalhes de implementação interna
não disponíveis para uso externo. Podemos escolher tornar módulos e os itens
dentro deles públicos, o que os expõe para permitir que código externo os use e dependa
deles.

Como exemplo, vamos escrever um crate de biblioteca que fornece a funcionalidade de um
restaurante. Definiremos as assinaturas das funções, mas deixaremos seus corpos
vazios para nos concentrar na organização do código em vez da
implementação de um restaurante.

Na indústria de restaurantes, algumas partes de um restaurante são chamadas de frente
da casa e outras de fundos da casa. _Frente da casa_ é onde os clientes estão;
isso engloba onde os anfitriões sentam os clientes, garçons anotam pedidos e
pagamento, e bartenders fazem bebidas. _Fundos da casa_ é onde os chefs e
cozinheiros trabalham na cozinha, lavadores de pratos limpam, e gerentes fazem trabalho
administrativo.

Para estruturar nosso crate dessa maneira, podemos organizar suas funções em módulos
aninhados. Crie uma nova biblioteca chamada `restaurant` executando `cargo new
restaurant --lib`. Em seguida, insira o código na Listagem 7-1 em *src/lib.rs* para
definir alguns módulos e assinaturas de função; este código é a seção frente da casa.

<Listing number="7-1" file-name="src/lib.rs" caption="Um módulo `front_of_house` contendo outros módulos que então contêm funções">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

Nós definimos um módulo com a palavra-chave `mod` seguida pelo nome do módulo
(neste caso, `front_of_house`). O corpo do módulo então vai dentro de chaves.
Dentro de módulos, podemos colocar outros módulos, como neste caso com os
módulos `hosting` e `serving`. Módulos também podem conter definições para outros
itens, como structs, enums, constantes, traits e, como na Listagem 7-1,
funções.

Ao usar módulos, podemos agrupar definições relacionadas e nomear por que
elas estão relacionadas. Programadores usando este código podem navegar pelo código com base nos
grupos em vez de ter que ler todas as definições, tornando mais fácil
encontrar as definições relevantes para eles. Programadores adicionando novas funcionalidades
a este código saberiam onde colocar o código para manter o programa organizado.

Anteriormente, mencionamos que *src/main.rs* e *src/lib.rs* são chamados de *raízes do
crate*. A razão para o nome deles é que o conteúdo de qualquer um desses dois
arquivos forma um módulo chamado `crate` na raiz da estrutura de módulos do crate,
conhecida como a *árvore de módulos*.

A Listagem 7-2 mostra a árvore de módulos para a estrutura na Listagem 7-1.

<Listing number="7-2" caption="A árvore de módulos para o código na Listagem 7-1">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Esta árvore mostra como alguns dos módulos se aninham dentro de outros módulos; por exemplo,
`hosting` aninha dentro de `front_of_house`. A árvore também mostra que alguns módulos
são *irmãos*, significando que são definidos no mesmo módulo; `hosting` e
`serving` são irmãos definidos dentro de `front_of_house`. Se o módulo A está
contido dentro do módulo B, dizemos que o módulo A é o *filho* do módulo B e
que o módulo B é o *pai* do módulo A. Observe que toda a árvore de módulos
está enraizada sob o módulo implícito chamado `crate`.

A árvore de módulos pode lembrar a árvore de diretórios do sistema de arquivos no seu
computador; esta é uma comparação muito apropriada! Assim como diretórios em um sistema de arquivos,
você usa módulos para organizar seu código. E assim como arquivos em um diretório, nós
precisamos de uma maneira de encontrar nossos módulos.
