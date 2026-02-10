# Padrões e Casamento de Padrões

[[Padrões]] (Patterns) são uma sintaxe especial em Rust para corresponder à estrutura de tipos, tanto complexos quanto simples. Usar padrões em conjunto com expressões `match` e outras construções dá a você mais controle sobre o fluxo de controle de um programa. Um padrão consiste em alguma combinação do seguinte:

- Literais
- Arrays, enums, structs ou tuplas desestruturados
- Variáveis
- Curingas (Wildcards)
- Espaços reservados (Placeholders)

Alguns exemplos de padrões incluem `x`, `(a, 3)` e `Some(Color::Red)`. Nos contextos em que os padrões são válidos, esses componentes descrevem a forma dos dados. Nosso programa então combina valores com os padrões para determinar se tem a forma correta de dados para continuar executando uma determinada parte do código.

Para usar um padrão, nós o comparamos com algum valor. Se o padrão corresponder ao valor, usamos as partes do valor em nosso código. Lembre-se das expressões `match` no Capítulo 6 que usavam padrões, como o exemplo da máquina de classificação de moedas. Se o valor se encaixar na forma do padrão, podemos usar as peças nomeadas. Se não, o código associado ao padrão não será executado.

Este capítulo é uma referência sobre tudo relacionado a padrões. Cobriremos os lugares válidos para usar padrões, a diferença entre padrões refutáveis e irrefutáveis, e os diferentes tipos de sintaxe de padrão que você pode ver. Ao final do capítulo, você saberá como usar padrões para expressar muitos conceitos de uma maneira clara.
