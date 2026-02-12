## Estendendo o Cargo com Comandos Personalizados

O Cargo foi projetado para que você possa estendê-lo com novos subcomandos sem precisar modificá-lo. Se um binário em seu `$PATH` for nomeado `cargo-something`, você pode executá-lo como se fosse um subcomando do Cargo executando `cargo something`. Comandos personalizados como este também são listados quando você executa `cargo --list`. Ser capaz de usar `cargo install` para instalar extensões e depois executá-las exatamente como as ferramentas integradas do Cargo é um benefício super conveniente do design do Cargo!

## Resumo

Compartilhar código com Cargo e [crates.io](https://crates.io/)<!-- ignore --> é parte do que torna o ecossistema Rust útil para muitas tarefas diferentes. A biblioteca padrão do Rust é pequena e estável, mas os crates são fáceis de compartilhar, usar e melhorar em um cronograma diferente do da linguagem. Não tenha vergonha de compartilhar código que é útil para você em [crates.io](https://crates.io/)<!-- ignore -->; é provável que seja útil para outra pessoa também!
