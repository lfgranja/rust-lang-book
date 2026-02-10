# A Linguagem de Programação Rust

bem-vindo a *A Linguagem de Programação Rust*, um livro introdutório sobre Rust. A linguagem de programação Rust ajuda você a escrever software mais rápido e confiável. A ergonomia de alto nível e o controle de baixo nível estão frequentemente em desacordo no design de linguagens de programação; Rust desafia esse conflito. Através do equilíbrio de uma capacidade técnica poderosa e uma ótima experiência de desenvolvedor, Rust oferece a opção de controlar detalhes de baixo nível (como uso de memória) sem todo o incômodo tradicionalmente associado a tal controle.

## Para Quem é Este Livro

Este livro pressupõe que você já escreveu código em outra linguagem de programação, mas não faz suposições sobre qual. Tentamos tornar o material amplamente acessível a pessoas de uma ampla variedade de origens de programação. Não passamos muito tempo falando sobre o que é programação ou como pensar sobre isso. Se você é totalmente novo em programação, estaria melhor servido lendo um livro que forneça uma introdução à programação especificamente.

## Como Usar Este Livro

Em geral, este livro assume que você o está lendo em sequência, da frente para trás. Capítulos posteriores constroem sobre conceitos em capítulos anteriores, e capítulos anteriores podem não se aprofundar em detalhes sobre um tópico; normalmente revisitamos o tópico em um capítulo posterior.

Você encontrará dois tipos de capítulos neste livro: capítulos conceituais e capítulos de projeto. Nos capítulos conceituais, aprenderemos sobre um aspecto do Rust. Nos capítulos de projeto, construiremos pequenos programas juntos, aplicando o que aprendemos até agora. Os capítulos 2, 12 e 20 são capítulos de projeto; o resto são capítulos conceituais.

O Capítulo 1 explica como instalar Rust, como escrever um programa "Hello, world!", e como usar Cargo, o gerenciador de pacotes e ferramenta de construção do Rust. O Capítulo 2 é uma introdução prática à escrita de um programa em Rust, onde fazemos um jogo de adivinhação de números. Aqui cobrimos conceitos em um nível alto, e capítulos posteriores fornecerão detalhes adicionais. Se você quiser sujar as mãos imediatamente, o Capítulo 2 é o lugar para isso. O Capítulo 3 cobre recursos do Rust semelhantes aos de outras linguagens de programação, e no Capítulo 4 você aprenderá sobre o sistema de propriedade do Rust. Se você é um aprendiz particularmente meticuloso que prefere aprender cada detalhe antes de passar para o próximo, você pode pular o Capítulo 2 e ir direto para o Capítulo 3, retornando ao Capítulo 2 quando quiser trabalhar em um projeto aplicando os detalhes que aprendeu.

O Capítulo 5 discute structs e métodos, e o Capítulo 6 cobre enums, expressões `match` e a construção de controle de fluxo `if let`. Você usará structs e enums para criar tipos personalizados em Rust.

No Capítulo 7, você aprenderá sobre o sistema de módulos do Rust e sobre regras de privacidade para organizar seu código e sua interface pública de programação de aplicativos (API). O Capítulo 8 discute algumas estruturas de dados comuns de coleção que a biblioteca padrão fornece, como vetores, strings e mapas hash. O Capítulo 9 explora a filosofia e as técnicas de tratamento de erros do Rust.

O Capítulo 10 aprofunda-se em genéricos, traits e tempos de vida, que lhe dão o poder de definir código que se aplica a múltiplos tipos. O Capítulo 11 é tudo sobre testes, que é crítico para garantir que a lógica do seu programa esteja correta, mesmo com as garantias de segurança do Rust. No Capítulo 12, construiremos nossa própria implementação de um subconjunto da funcionalidade da ferramenta de linha de comando `grep` que busca texto dentro de arquivos. Para isso, usaremos muitos dos conceitos que discutimos nos capítulos anteriores.

O Capítulo 13 explora closures e iteradores: características do Rust que vêm de linguagens de programação funcional. No Capítulo 14, examinaremos Cargo em mais profundidade e falaremos sobre as melhores práticas para compartilhar suas bibliotecas com outros. O Capítulo 15 discute ponteiros inteligentes que a biblioteca padrão fornece e os traits que permitem sua funcionalidade.

No Capítulo 16, percorreremos diferentes modelos de programação concorrente e falaremos sobre como o Rust ajuda você a programar em múltiplas threads sem medo. O Capítulo 17 analisa como o idioma Rust se compara aos paradigmas de programação orientada a objetos que você pode estar familiarizado.

O Capítulo 18 é uma referência sobre padrões e correspondência de padrões, que são maneiras poderosas de expressar ideias em programas Rust. O Capítulo 19 contém um smorgasbord de tópicos avançados de interesse, incluindo Rust inseguro, macros e mais sobre tempos de vida, traits, tipos, funções e closures.

No Capítulo 20, concluiremos um projeto no qual construiremos um servidor web multithreaded de baixo nível!

Finalmente, alguns apêndices contêm informações úteis sobre a linguagem em um formato mais de referência. O Apêndice A cobre as palavras-chave do Rust, o Apêndice B cobre operadores e símbolos, o Apêndice C cobre traits deriváveis fornecidos pela biblioteca padrão, o Apêndice D cobre algumas ferramentas de desenvolvimento úteis e o Apêndice E explica as edições do Rust.

Não há maneira errada de ler este livro: se você quiser pular adiante, vá em frente! Você pode ter que voltar aos capítulos anteriores se experimentar alguma confusão. Faça o que funcionar para você.

<span class="caption">Ferris the crab</span>
<!-- <img src="img/ferris.png" class="center" style="width: 25%;" alt="Ferris the crab" /> -->

Uma parte importante do processo de aprendizado do Rust é aprender como ler as mensagens de erro que o compilador exibe: elas o guiarão para um código funcional. Como tal, forneceremos muitos exemplos que não compilam junto com a mensagem de erro que o compilador mostrará em cada situação. Saiba que se você inserir e executar um exemplo aleatório, ele pode não compilar! Certifique-se de ler o texto ao redor para ver se o exemplo que você está tentando executar deve dar erro. Na maioria das situações, levaremos você à versão correta do código que não compila.

## Código Fonte

Os arquivos de origem a partir dos quais este livro é gerado podem ser encontrados no [GitHub](https://github.com/rust-lang/book).

[ferris]: img/ferris.png
