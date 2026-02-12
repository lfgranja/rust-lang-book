# Coleções Comuns

A biblioteca padrão do Rust inclui uma série de estruturas de dados muito úteis
chamadas *coleções*. A maioria dos outros tipos de dados representa um valor
específico, mas as coleções podem conter múltiplos valores. Diferentemente dos
tipos embutidos de array e tupla, os dados que essas coleções apontam são
armazenados na heap, o que significa que a quantidade de dados não precisa ser
conhecida em tempo de compilação e pode crescer ou diminuir à medida que o
programa roda. Cada tipo de coleção tem capacidades e custos diferentes, e
escolher um apropriado para sua situação atual é uma habilidade que você
desenvolverá ao longo do tempo. Neste capítulo, discutiremos três coleções que
são usadas muito frequentemente em programas Rust:

* Um *vetor* permite armazenar um número variável de valores um ao lado do
  outro.
* Uma *string* é uma coleção de caracteres. Mencionamos o tipo `String`
  anteriormente, mas neste capítulo falaremos sobre ele em profundidade.
* Um *hash map* permite associar um valor a uma chave específica. É uma
  implementação particular da estrutura de dados mais geral chamada *mapa*.

Para aprender sobre as outras coleções fornecidas pela biblioteca padrão, veja
[a documentação][documentation].

Discutiremos como criar e atualizar vetores, strings e hash maps, bem como o que
torna cada um especial.

[documentation]: ../std/collections/index.html
