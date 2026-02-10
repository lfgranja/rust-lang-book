# Entendendo Ownership

O *Ownership* (propriedade) é a característica mais única do Rust e tem implicações profundas para o resto da linguagem. Ele permite que o Rust faça garantias de segurança de memória sem precisar de um *garbage collector* (coletor de lixo), por isso é importante entender como o *ownership* funciona. Neste capítulo, falaremos sobre *ownership*, bem como várias características relacionadas: *borrowing* (empréstimo), *slices* (fatias) e como o Rust organiza dados na memória.
