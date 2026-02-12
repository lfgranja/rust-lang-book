# Tratamento de Erros

Erros são um fato da vida no software, então Rust tem uma série de recursos para
lidar com situações em que algo dá errado. Em muitos casos, Rust exige que
você reconheça a possibilidade de um erro e tome alguma ação antes que seu
código compile. Esse requisito torna seu programa mais robusto, garantindo
que você descobrirá erros e lidará com eles adequadamente antes de implantar seu
código em produção!

Rust agrupa erros em duas categorias principais: erros recuperáveis e erros
irrecuperáveis. Para um _erro recuperável_, como um erro de _arquivo não
encontrado_, provavelmente queremos apenas relatar o problema ao usuário e
tentar a operação novamente. _Erros irrecuperáveis_ são sempre sintomas de
bugs, como tentar acessar um local além do final de um array, e por isso
queremos parar imediatamente o programa.

A maioria das linguagens não distingue entre esses dois tipos de erros e trata
ambos da mesma forma, usando mecanismos como exceções. Rust não tem exceções.
Em vez disso, ele tem o tipo `Result<T, E>` para erros recuperáveis e a macro
`panic!` que interrompe a execução quando o programa encontra um erro
irrecuperável. Este capítulo aborda a chamada de `panic!` primeiro e depois
fala sobre o retorno de valores `Result<T, E>`. Além disso, exploraremos
considerações ao decidir se devemos tentar nos recuperar de um erro ou parar
a execução.
