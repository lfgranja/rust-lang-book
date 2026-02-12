## Separando Módulos em Diferentes Arquivos

Até agora, todos os exemplos neste capítulo definiram vários módulos em um arquivo.
Quando os módulos ficam grandes, você pode querer mover suas definições para um arquivo
separado para tornar o código mais fácil de navegar.

Por exemplo, vamos começar a partir do código na Listagem 7-17 que tinha vários
módulos de restaurante. Extrairemos módulos em arquivos em vez de ter todos os
módulos definidos no arquivo raiz do crate. Neste caso, o arquivo raiz do crate é
*src/lib.rs*, mas este procedimento também funciona com crates binários cujo arquivo raiz
do crate é *src/main.rs*.

Primeiro, extrairemos o módulo `front_of_house` para seu próprio arquivo. Remova o
código dentro das chaves para o módulo `front_of_house`, deixando apenas
a declaração `mod front_of_house;`, para que *src/lib.rs* contenha o código
mostrado na Listagem 7-21. Note que isso não compilará até criarmos o
arquivo *src/front_of_house.rs* na Listagem 7-22.

<Listing number="7-21" file-name="src/lib.rs" caption="Declarando o módulo `front_of_house` cujo corpo estará em *src/front_of_house.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

Em seguida, coloque o código que estava nas chaves em um novo arquivo chamado
*src/front_of_house.rs*, conforme mostrado na Listagem 7-22. O compilador sabe procurar
neste arquivo porque encontrou a declaração do módulo na raiz do crate
com o nome `front_of_house`.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="Definições dentro do módulo `front_of_house` em *src/front_of_house.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

Note que você só precisa carregar um arquivo usando uma declaração `mod` *uma vez* em sua
árvore de módulos. Uma vez que o compilador sabe que o arquivo é parte do projeto (e sabe
onde na árvore de módulos o código reside por causa de onde você colocou a instrução `mod`),
outros arquivos em seu projeto devem se referir ao código do arquivo carregado
usando um caminho para onde ele foi declarado, como coberto na seção ["Caminhos para Referenciar
um Item na Árvore de Módulos"][paths]<!-- ignore -->. Em outras palavras,
`mod` *não* é uma operação "include" que você pode ter visto em outras
linguagens de programação.

Em seguida, extrairemos o módulo `hosting` para seu próprio arquivo. O processo é um pouco
diferente porque `hosting` é um módulo filho de `front_of_house`, não do
módulo raiz. Colocaremos o arquivo para `hosting` em um novo diretório que será
nomeado para seus ancestrais na árvore de módulos, neste caso *src/front_of_house*.

Para começar a mover `hosting`, mudamos *src/front_of_house.rs* para conter apenas
a declaração do módulo `hosting`:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

Então, criamos um diretório *src/front_of_house* e um arquivo *hosting.rs* para
conter as definições feitas no módulo `hosting`:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

Se em vez disso colocássemos *hosting.rs* no diretório *src*, o compilador
esperaria que o código *hosting.rs* estivesse em um módulo `hosting` declarado na raiz
do crate e não declarado como um filho do módulo `front_of_house`. As
regras do compilador para quais arquivos verificar para o código de quais módulos significam
que os diretórios e arquivos correspondem mais de perto à árvore de módulos.

> ### Caminhos de Arquivo Alternativos
>
> Até agora cobrimos os caminhos de arquivo mais idiomáticos que o compilador Rust usa,
> mas Rust também suporta um estilo mais antigo de caminho de arquivo. Para um módulo chamado
> `front_of_house` declarado na raiz do crate, o compilador procurará pelo
> código do módulo em:
>
> - *src/front_of_house.rs* (o que cobrimos)
> - *src/front_of_house/mod.rs* (estilo mais antigo, caminho ainda suportado)
>
> Para um módulo chamado `hosting` que é um submódulo de `front_of_house`, o
> compilador procurará pelo código do módulo em:
>
> - *src/front_of_house/hosting.rs* (o que cobrimos)
> - *src/front_of_house/hosting/mod.rs* (estilo mais antigo, caminho ainda suportado)
>
> Se você usar ambos os estilos para o mesmo módulo, receberá um erro do compilador.
> Usar uma mistura de ambos os estilos para diferentes módulos no mesmo projeto é
> permitido, mas pode ser confuso para pessoas navegando em seu projeto.
>
> A principal desvantagem do estilo que usa arquivos chamados *mod.rs* é que seu
> projeto pode acabar com muitos arquivos chamados *mod.rs*, o que pode ficar confuso
> quando você os tem abertos em seu editor ao mesmo tempo.

Movemos o código de cada módulo para um arquivo separado, e a árvore de módulos permanece
a mesma. As chamadas de função em `eat_at_restaurant` funcionarão sem qualquer
modificação, mesmo que as definições vivam em arquivos diferentes. Esta
técnica permite mover módulos para novos arquivos à medida que crescem em tamanho.

Note que a instrução `pub use crate::front_of_house::hosting` em
*src/lib.rs* também não mudou, nem `use` tem qualquer impacto em quais arquivos
são compilados como parte do crate. A palavra-chave `mod` declara módulos, e o Rust
procura em um arquivo com o mesmo nome que o módulo pelo código que vai para
dentro desse módulo.

## Resumo

Rust permite que você divida um pacote em vários crates e um crate em módulos para
que você possa se referir a itens definidos em um módulo de outro módulo. Você pode
fazer isso especificando caminhos absolutos ou relativos. Esses caminhos podem ser trazidos
para o escopo com uma instrução `use` para que você possa usar um caminho mais curto para
vários usos do item nesse escopo. O código do módulo é privado por padrão, mas
você pode tornar as definições públicas adicionando a palavra-chave `pub`.

No próximo capítulo, veremos algumas estruturas de dados de coleção na
biblioteca padrão que você pode usar em seu código bem organizado.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
