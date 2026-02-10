# Apêndice C: Traits Deriváveis

Em vários lugares do livro, discutimos o atributo `derive`, que você pode aplicar a uma definição de struct ou enum. O atributo `derive` gera código que implementará um trait com sua própria implementação padrão no tipo que você anotou com o atributo `derive`.

Neste apêndice, fornecemos uma referência de todos os traits na biblioteca padrão que você pode usar com `derive`. Cada um deles está listado com uma explicação do que ele faz e quais comportamentos ele habilita.

## `Debug` para Saída de Programador

O trait `Debug` habilita a formatação de depuração em strings de formato, que você indica adicionando `:?` dentro de marcadores `{}`. O trait `Debug` permite que você imprima instâncias de um tipo para fins de depuração, para que você e outros programadores usando seu tipo possam inspecionar uma instância em um ponto específico na execução de um programa.

O trait `Debug` é necessário, por exemplo, no uso da macro `assert_eq!`. Esta macro imprime os valores das instâncias dadas como argumentos se a afirmação de igualdade falhar para que os programadores possam ver por que as duas instâncias não eram iguais.

## `PartialEq` e `Eq` para Comparações de Igualdade

O trait `PartialEq` permite que você compare instâncias de um tipo para verificar a igualdade e permite o uso dos operadores `==` e `!=`.

Ao derivar `PartialEq`, o método `eq` verifica a igualdade estrutural: dois campos são iguais se seus valores forem iguais. Por exemplo, duas instâncias de um struct `Ponto` são iguais se seus campos `x` forem iguais e seus campos `y` forem iguais.

Se um tipo não tiver uma implementação de `PartialEq`, você não poderá usar os operadores `==` ou `!=` para comparar suas instâncias.

Derivar `PartialEq` também é necessário para usar a macro `assert_eq!`, que precisa ser capaz de comparar duas instâncias de um tipo para igualdade.

O trait `Eq` não tem métodos. Seu objetivo é sinalizar que para cada valor do tipo anotado, o valor é igual a si mesmo. O trait `Eq` pode ser aplicado a qualquer tipo que também implemente `PartialEq`, exceto para tipos de ponto flutuante, que não implementam `Eq`. A biblioteca padrão implementa `PartialEq` para tipos de ponto flutuante, mas não `Eq`, porque os valores de ponto flutuante incluem um valor especial, `NaN` (Não é um Número), que não é igual a si mesmo. Se você tiver um tipo que contém um campo de ponto flutuante, você pode derivar `PartialEq` para ele, mas não `Eq`.

## `PartialOrd` e `Ord` para Comparações de Ordenação

O trait `PartialOrd` permite que você compare instâncias de um tipo para fins de classificação. Um tipo que implementa `PartialOrd` pode ser usado com os operadores `<`, `>`, `<=` e `>=`. Você só pode aplicar o atributo `derive` a tipos que implementam `PartialEq`.

Derivar `PartialOrd` implementa o método `partial_cmp` para retornar um `Option<Ordering>` que será `None` quando os valores dados não produzirem uma ordenação. Um exemplo de um valor que não produz uma ordenação, mesmo sendo do mesmo tipo, é o valor de ponto flutuante `NaN`. Chamar `partial_cmp` com qualquer número de ponto flutuante e o valor `NaN` retornará `None`.

Ao derivar em structs, `PartialOrd` compara dois valores comparando o valor em cada campo na ordem em que os campos aparecem na definição da struct. Ao derivar em enums, as variantes da enum declaradas anteriormente na definição da enum são consideradas menores do que as variantes listadas posteriormente.

O trait `Ord` permite que você saiba que para quaisquer dois valores do tipo anotado, uma ordenação válida existirá. O trait `Ord` implementa o método `cmp`, que retorna um `Ordering` em vez de um `Option<Ordering>` porque uma ordenação válida sempre será possível. Você só pode aplicar o atributo `derive` a tipos que implementam `PartialOrd` e `Eq` (e `Eq` requer `PartialEq`). Ao derivar em structs e enums, `cmp` se comporta da mesma maneira que a implementação derivada para `partial_cmp` faz com `PartialOrd`.

Um exemplo de quando `Ord` é necessário é quando armazenamos valores em um `BTreeSet<T>`, uma estrutura de dados que armazena dados com base na ordem de classificação dos valores.

## `Clone` e `Copy` para Duplicação de Valores

O trait `Clone` permite que você crie explicitamente uma cópia profunda de um valor, e o processo de duplicação pode envolver a execução de código arbitrário e a cópia de dados da pilha. Consulte a seção "Maneiras de Interagir com Variáveis e Dados: Clonar" no Capítulo 4 para mais informações sobre `Clone`.

Derivar `Clone` implementa o método `clone`, que para uma implementação de tipo inteira, chama `clone` em cada parte do tipo. Isso significa que todos os campos ou valores no tipo também devem implementar `Clone` para derivar `Clone`.

O trait `Copy` permite que você duplique um valor apenas copiando bits armazenados na pilha; nenhum código arbitrário é necessário. Consulte a seção "Dados Apenas na Pilha: Copiar" no Capítulo 4 para mais informações sobre `Copy`.

O trait `Copy` não define nenhum método para evitar que os programadores sobrecarreguem esses métodos e violem a suposição de que nenhum código arbitrário está sendo executado. Dessa forma, todos os programadores podem assumir que copiar um valor será muito rápido.

Você pode derivar `Copy` em qualquer tipo cujas partes todas implementem `Copy`. Você só pode aplicar o atributo `derive` a tipos que também implementam `Clone`, porque um tipo que implementa `Copy` tem uma implementação trivial de `Clone` que realiza a mesma tarefa que `Copy`.

## `Hash` para Mapear um Valor para um Valor de Tamanho Fixo

O trait `Hash` permite que você pegue uma instância de um tipo de tamanho arbitrário e mapeie essa instância para um valor de tamanho fixo (um *hash*) usando uma função hash. Derivar `Hash` implementa o método `hash`. A implementação derivada de `hash` combina o resultado de chamar `hash` em cada uma das partes do tipo, o que significa que todos os campos ou valores também devem implementar `Hash` para derivar `Hash`.

Um exemplo de quando `Hash` é necessário é ao armazenar chaves em um `HashMap<K, V>` para armazenar dados de forma eficiente.

## `Default` para Valores Padrão

O trait `Default` permite que você crie um valor padrão para um tipo. Derivar `Default` implementa a função `default`. A implementação derivada da função `default` chama a função `default` em cada parte do tipo, o que significa que todos os campos ou valores no tipo também devem implementar `Default` para derivar `Default`.

O trait `Default` é frequentemente usado em combinação com o método `unwrap_or_default` em instâncias de `Option<T>`. Se o `Option<T>` for `None`, o método `unwrap_or_default` retornará o resultado de `Default::default` para o tipo `T`.
