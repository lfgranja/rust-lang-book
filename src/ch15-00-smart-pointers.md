# Ponteiros Inteligentes

Um *ponteiro* é um conceito geral para uma variável que contém um endereço de memória. Este endereço refere-se a, ou "aponta para", algum outro dado. O tipo mais comum de ponteiro em Rust é uma referência, sobre a qual você aprendeu no Capítulo 4. Referências são indicadas pelo símbolo `&` e pegam emprestado o valor para o qual apontam. Elas não têm nenhuma capacidade especial além de se referir aos dados, e não têm custo adicional (overhead).

*Ponteiros inteligentes* (smart pointers), por outro lado, são estruturas de dados que agem como um ponteiro, mas também têm metadados e capacidades adicionais. O conceito de ponteiros inteligentes não é exclusivo de Rust: ponteiros inteligentes originaram-se em C++ e existem em outras linguagens também. Rust tem uma variedade de ponteiros inteligentes definidos na biblioteca padrão que fornecem funcionalidades além das fornecidas por referências. Para explorar o conceito geral, veremos alguns exemplos diferentes de ponteiros inteligentes, incluindo um tipo de ponteiro inteligente com *contagem de referências*. Este ponteiro permite que você permita que dados tenham múltiplos donos, mantendo o controle do número de donos e, quando nenhum dono restar, limpando os dados.

Em Rust, com seu conceito de posse (ownership) e empréstimo (borrowing), há uma diferença adicional entre referências e ponteiros inteligentes: enquanto referências apenas pegam dados emprestados, em muitos casos ponteiros inteligentes *possuem* os dados para os quais apontam.

Embora não os tenhamos chamado assim na época, já encontramos alguns ponteiros inteligentes neste livro, incluindo `String` e `Vec<T>` no Capítulo 8. Ambos os tipos contam como ponteiros inteligentes porque possuem alguma memória e permitem que você a manipule. Eles também têm metadados e capacidades extras (como sua capacidade e garantia de que os dados adicionais serão sempre UTF-8 válido).

Ponteiros inteligentes são geralmente implementados usando structs. Diferente de uma struct comum, ponteiros inteligentes implementam as traits `Deref` e `Drop`. A trait `Deref` permite que uma instância da struct de ponteiro inteligente se comporte como uma referência, para que você possa escrever seu código para funcionar tanto com referências quanto com ponteiros inteligentes. A trait `Drop` permite que você personalize o código que é executado quando uma instância do ponteiro inteligente sai de escopo. Neste capítulo, discutiremos ambas as traits e demonstraremos por que elas são importantes para ponteiros inteligentes.

Dado que o padrão de ponteiro inteligente é um padrão de design geral usado frequentemente em Rust, este capítulo não cobrirá todos os ponteiros inteligentes existentes. Muitas bibliotecas têm seus próprios ponteiros inteligentes, e você pode até escrever os seus próprios. Cobriremos os ponteiros inteligentes mais comuns na biblioteca padrão:

- `Box<T>`, para alocar valores na heap
- `Rc<T>`, um tipo com contagem de referências que permite múltipla posse
- `Ref<T>` e `RefMut<T>`, acessados através de `RefCell<T>`, um tipo que impõe as regras de empréstimo em tempo de execução em vez de tempo de compilação

Além disso, cobriremos o padrão de *mutabilidade interior* (interior mutability), onde um tipo imutável expõe uma API para mutar um valor interior. Também discutiremos *ciclos de referência*: como eles podem vazar memória e como preveni-los.

Vamos mergulhar nisso!
