# Escrevendo Testes Automatizados

Em seu ensaio de 1972 “The Humble Programmer” (O Programador Humilde), Edsger W.
Dijkstra disse que “o teste de programa pode ser uma maneira muito eficaz de
mostrar a presença de bugs, mas é irremediavelmente inadequado para mostrar sua
ausência”. Isso não significa que não devemos tentar testar o máximo que
pudermos!

_Corretude_ em nossos programas é a medida em que nosso código faz o que
pretendemos que ele faça. Rust é projetado com um alto grau de preocupação com
a corretude dos programas, mas a corretude é complexa e não é fácil de provar.
O sistema de tipos do Rust assume uma grande parte desse fardo, mas o sistema
de tipos não pode capturar tudo. Como tal, Rust inclui suporte para escrever
testes de software automatizados.

Digamos que escrevemos uma função `add_two` que adiciona 2 a qualquer número
passado para ela. A assinatura dessa função aceita um inteiro como parâmetro e
retorna um inteiro como resultado. Quando implementamos e compilamos essa
função, Rust faz toda a verificação de tipo e verificação de empréstimo que
você aprendeu até agora para garantir que, por exemplo, não estejamos passando
um valor `String` ou uma referência inválida para essa função. Mas Rust _não
pode_ verificar se essa função fará exatamente o que pretendemos, que é
retornar o parâmetro mais 2 em vez de, digamos, o parâmetro mais 10 ou o
parâmetro menos 50! É aí que entram os testes.

Podemos escrever testes que afirmam, por exemplo, que quando passamos `3` para
a função `add_two`, o valor retornado é `5`. Podemos executar esses testes
sempre que fizermos alterações em nosso código para garantir que qualquer
comportamento correto existente não tenha mudado.

Testar é uma habilidade complexa: Embora não possamos cobrir em um capítulo
cada detalhe sobre como escrever bons testes, neste capítulo discutiremos a
mecânica das instalações de teste do Rust. Falaremos sobre as anotações e
macros disponíveis para você ao escrever seus testes, o comportamento padrão e
as opções fornecidas para executar seus testes e como organizar testes em
testes de unidade e testes de integração.
