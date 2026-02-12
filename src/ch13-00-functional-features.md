# Recursos de Linguagem Funcional: Iteradores e Closures

O design do Rust se inspirou em muitas linguagens e técnicas existentes, e uma influência significativa é a *programação funcional*. Programar em um estilo funcional geralmente inclui usar funções como valores, passando-as em argumentos, retornando-as de outras funções, atribuindo-as a variáveis para execução posterior e assim por diante.

Neste capítulo, não debateremos a questão do que é ou não é programação funcional, mas, em vez disso, discutiremos alguns recursos do Rust que são semelhantes aos recursos em muitas linguagens frequentemente chamadas de funcionais.

Mais especificamente, cobriremos:

- *Closures*, uma construção semelhante a uma função que você pode armazenar em uma variável
- *Iteradores*, uma maneira de processar uma série de elementos
- Como usar closures e iteradores para melhorar o projeto de E/S no Capítulo 12
- O desempenho de closures e iteradores (alerta de spoiler: eles são mais rápidos do que você imagina!)

Já cobrimos alguns outros recursos do Rust, como correspondência de padrões e enums, que também são influenciados pelo estilo funcional. Como dominar closures e iteradores é uma parte importante da escrita de código Rust rápido e idiomático, dedicaremos este capítulo inteiro a eles.
