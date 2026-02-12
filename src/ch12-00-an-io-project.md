# Um Projeto de E/S: Construindo um Programa de Linha de Comando

Este capítulo é uma recapitulação das muitas habilidades que você aprendeu até agora e uma exploração de mais alguns recursos da biblioteca padrão. Construiremos uma ferramenta de linha de comando que interage com a entrada/saída de arquivo e linha de comando para praticar alguns dos conceitos de Rust que você agora tem em seu currículo.

A velocidade, a segurança, a saída binária única e o suporte multiplataforma do Rust o tornam uma linguagem ideal para criar ferramentas de linha de comando, portanto, para nosso projeto, faremos nossa própria versão da clássica ferramenta de pesquisa de linha de comando `grep` (**g**lobally search a **r**egular **e**xpression and **p**rint - pesquisar globalmente uma expressão regular e imprimir). No caso de uso mais simples, o `grep` pesquisa um arquivo especificado por uma string especificada. Para fazer isso, o `grep` recebe como argumentos um caminho de arquivo e uma string. Em seguida, ele lê o arquivo, encontra as linhas nesse arquivo que contêm o argumento da string e imprime essas linhas.

Ao longo do caminho, mostraremos como fazer nossa ferramenta de linha de comando usar os recursos de terminal que muitas outras ferramentas de linha de comando usam. Leremos o valor de uma variável de ambiente para permitir que o usuário configure o comportamento de nossa ferramenta. Também imprimiremos mensagens de erro no fluxo de console de erro padrão (`stderr`) em vez da saída padrão (`stdout`), para que, por exemplo, o usuário possa redirecionar a saída bem-sucedida para um arquivo enquanto ainda vê mensagens de erro na tela.

Um membro da comunidade Rust, Andrew Gallant, já criou uma versão completa e muito rápida do `grep`, chamada `ripgrep`. Em comparação, nossa versão será bastante simples, mas este capítulo fornecerá parte do conhecimento básico necessário para entender um projeto do mundo real, como o `ripgrep`.

Nosso projeto `grep` combinará vários conceitos que você aprendeu até agora:

- Organização de código ([[ch07-00-managing-growing-projects-with-packages-crates-and-modules|Capítulo 7]])
- Uso de vetores e strings ([[ch08-00-common-collections|Capítulo 8]])
- Tratamento de erros ([[ch09-00-error-handling|Capítulo 9]])
- Uso de traits e lifetimes onde apropriado ([[ch10-00-generics|Capítulo 10]])
- Escrita de testes ([[ch11-00-testing|Capítulo 11]])

Também apresentaremos brevemente closures, iteradores e trait objects, que o [[ch13-00-functional-features|Capítulo 13]] e o [[ch18-00-oop|Capítulo 18]] cobrirão em detalhes.
