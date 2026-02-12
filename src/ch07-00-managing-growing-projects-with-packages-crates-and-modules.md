<!-- Old headings. Do not remove or links may break. -->

<a id="managing-growing-projects-with-packages-crates-and-modules"></a>

# Gerenciando Projetos Crescentes com Pacotes, Crates e Módulos

À medida que você escreve programas grandes, organizar seu código se tornará cada vez mais
importante. Ao agrupar funcionalidades relacionadas e separar códigos com características
distintas, você esclarece onde encontrar o código que implementa uma determinada
funcionalidade e onde ir para mudar como uma funcionalidade funciona.

Os programas que escrevemos até agora estavam em um módulo em um arquivo. À medida que um
projeto cresce, você deve organizar o código dividindo-o em vários módulos
e, em seguida, em vários arquivos. Um pacote pode conter vários crates binários e
opcionalmente um crate de biblioteca. À medida que um pacote cresce, você pode extrair partes em
crates separados que se tornam dependências externas. Este capítulo cobre todas
essas técnicas. Para projetos muito grandes compreendendo um conjunto de pacotes
inter-relacionados que evoluem juntos, o Cargo fornece *workspaces*, que cobriremos em
["Workspaces do Cargo"][workspaces]<!-- ignore --> no Capítulo 14.

Também discutiremos o encapsulamento de detalhes de implementação, o que permite reutilizar
código em um nível mais alto: uma vez que você implementou uma operação, outro código pode
chamar seu código através de sua interface pública sem ter que saber como a
implementação funciona. A maneira como você escreve código define quais partes são públicas para
outro código usar e quais partes são detalhes de implementação privada que você
reserva o direito de mudar. Esta é outra maneira de limitar a quantidade de detalhes
que você tem que manter em sua cabeça.

Um conceito relacionado é o escopo: o contexto aninhado no qual o código é escrito tem um
conjunto de nomes que são definidos como "no escopo". Ao ler, escrever e
compilar código, programadores e compiladores precisam saber se um determinado
nome em um determinado local se refere a uma variável, função, struct, enum, módulo,
constante ou outro item e o que esse item significa. Você pode criar escopos e
alterar quais nomes estão dentro ou fora do escopo. Você não pode ter dois itens com o
mesmo nome no mesmo escopo; ferramentas estão disponíveis para resolver conflitos de nomes.

Rust tem uma série de recursos que permitem gerenciar a organização do seu código,
incluindo quais detalhes são expostos, quais detalhes são privados e
quais nomes estão em cada escopo em seus programas. Esses recursos, às vezes
coletivamente referidos como o *sistema de módulos*, incluem:

* **Pacotes** (*Packages*): Um recurso do Cargo que permite construir, testar e compartilhar crates
* **Crates**: Uma árvore de módulos que produz uma biblioteca ou executável
* **Módulos** (*Modules*) e **use**: Permitem controlar a organização, escopo e privacidade de
caminhos
* **Caminhos** (*Paths*): Uma maneira de nomear um item, como uma struct, função ou módulo

Neste capítulo, cobriremos todos esses recursos, discutiremos como eles interagem e
explicaremos como usá-los para gerenciar escopo. Ao final, você deve ter um entendimento sólido
do sistema de módulos e ser capaz de trabalhar com escopos como um profissional!

[workspaces]: ch14-03-cargo-workspaces.html
