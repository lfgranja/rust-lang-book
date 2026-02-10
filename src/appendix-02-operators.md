# Apêndice B: Operadores e Símbolos

Este apêndice contém um glossário da sintaxe de Rust, incluindo operadores e outros símbolos que aparecem sozinhos ou no contexto de caminhos, genéricos, limites de trait, macros, atributos, comentários, tuplas e colchetes.

## Operadores

A Tabela B-1 contém os operadores em Rust, uma descrição de como o operador apareceria no contexto, uma breve explicação e se esse operador é sobrecarregável. Se um operador for sobrecarregável, o trait relevante a ser usado para sobrecarregar esse operador está listado.

Tabela B-1: Operadores

| Operador | Exemplo | Explicação | Sobrecarregável? |
|----------|---------|-------------|----------------|
| `!` | `ident!(...)`, `ident!{...}`, `ident![...]` | Expansão de macro | |
| `!` | `!expr` | Negação lógica ou bit a bit | `Not` |
| `!=` | `expr != expr` | Desigualdade | `PartialEq` |
| `%` | `expr % expr` | Resto aritmético | `Rem` |
| `%=` | `var %= expr` | Atribuição de resto aritmético e atribuição | `RemAssign` |
| `&` | `&expr`, `&mut expr` | Empréstimo | |
| `&` | `&type`, `&mut type` | Tipo de ponteiro emprestado | |
| `&` | `expr & expr` | E (AND) bit a bit | `BitAnd` |
| `&=` | `var &= expr` | Atribuição de E (AND) bit a bit | `BitAndAssign` |
| `&&` | `expr && expr` | E (AND) lógico curto-circuito | |
| `*` | `expr * expr` | Multiplicação aritmética | `Mul` |
| `*` | `*expr` | Desreferência de ponteiro | `Deref` |
| `*` | `*const type`, `*mut type` | Ponteiro bruto | |
| `*=` | `var *= expr` | Atribuição de multiplicação aritmética | `MulAssign` |
| `+` | `expr + expr` | Adição aritmética | `Add` |
| `+` | `trait + trait`, `'a + trait` | Restrição de tipo composto | |
| `+=` | `var += expr` | Atribuição de adição aritmética | `AddAssign` |
| `,` | `expr, expr` | Separador de argumentos e elementos | |
| `-` | `- expr` | Negação aritmética | `Neg` |
| `-` | `expr - expr` | Subtração aritmética | `Sub` |
| `-=` | `var -= expr` | Atribuição de subtração aritmética | `SubAssign` |
| `->` | `fn(...) -> type`, `|...| -> type` | Tipo de retorno de função e closure | |
| `.` | `expr.ident` | Acesso a membro | |
| `..` | `..`, `expr..`, `..expr`, `expr..expr` | Literal de intervalo exclusivo à direita | `PartialOrd` |
| `..=` | `..=expr`, `expr..=expr` | Literal de intervalo inclusivo à direita | `PartialOrd` |
| `..` | `..expr` | Padrão de atualização de struct literal | |
| `..` | `variant(x, ..)`, `struct_type { x, .. }` | Ligação de padrão "e o resto" | |
| `...` | `expr...expr` | (Obsoleto, use `..=`) Em um padrão: literal de intervalo inclusivo | |
| `/` | `expr / expr` | Divisão aritmética | `Div` |
| `/=` | `var /= expr` | Atribuição de divisão aritmética | `DivAssign` |
| `:` | `pat: type`, `ident: type` | Restrições | |
| `:` | `ident: expr` | Inicializador de campo de struct | |
| `:` | `'a: loop {...}` | Rótulo de loop | |
| `;` | `expr;` | Terminador de instrução e item | |
| `;` | `[...; len]` | Parte da sintaxe de array de tamanho fixo | |
| `<<` | `expr << expr` | Deslocamento à esquerda (left-shift) | `Shl` |
| `<<=` | `var <<= expr` | Atribuição de deslocamento à esquerda | `ShlAssign` |
| `<` | `expr < expr` | Menor que | `PartialOrd` |
| `<=` | `expr <= expr` | Menor ou igual a | `PartialOrd` |
| `=` | `var = expr`, `ident = type` | Atribuição/equivalência | |
| `==` | `expr == expr` | Igualdade | `PartialEq` |
| `=>` | `pat => expr` | Parte da sintaxe do braço match | |
| `>` | `expr > expr` | Maior que | `PartialOrd` |
| `>=` | `expr >= expr` | Maior ou igual a | `PartialOrd` |
| `>>` | `expr >> expr` | Deslocamento à direita (right-shift) | `Shr` |
| `>>=` | `var >>= expr` | Atribuição de deslocamento à direita | `ShrAssign` |
| `@` | `ident @ pat` | Ligação de padrão | |
| `^` | `expr ^ expr` | OU Exclusivo (XOR) bit a bit | `BitXor` |
| `^=` | `var ^= expr` | Atribuição de OU Exclusivo (XOR) bit a bit | `BitXorAssign` |
| `|` | `pat | pat` | Alternativas de padrão | |
| `|` | `expr | expr` | OU (OR) bit a bit | `BitOr` |
| `|=` | `var |= expr` | Atribuição de OU (OR) bit a bit | `BitOrAssign` |
| `||` | `expr || expr` | OU (OR) lógico curto-circuito | |
| `?` | `expr?` | Propagação de erro | |

## Símbolos Não Operadores

A lista a seguir contém todos os símbolos que não funcionam como operadores; ou seja, eles não se comportam como uma chamada de função ou método.

A Tabela B-2 mostra símbolos que aparecem sozinhos e são válidos em uma variedade de locais.

Tabela B-2: Sintaxe Independente

| Símbolo | Explicação |
|--------|-------------|
| `'ident` | Tempo de vida nomeado ou rótulo de loop |
| `...u8`, `...i32`, `...f64`, `...usize`, etc. | Literal numérico de um tipo específico |
| `"..."` | Literal de string |
| `r"..."`, `r#"..."#`, `r##"..."##`, etc. | Literal de string bruta, caracteres de escape não processados |
| `b"..."` | Literal de string de bytes; constrói um array de bytes em vez de uma string |
| `br"..."`, `br#"..."#`, `br##"..."##`, etc. | Literal de string de bytes bruta, combinação de literal de string de bytes e bruta |
| `'...'` | Literal de caractere |
| `b'...'` | Literal de byte ASCII |
| `|...| expr` | Closure |
| `!` | Tipo vazio *Never* (nunca) para funções divergentes |
| `_` | "Ignorado" ligação de padrão; também usado para tornar literais inteiros legíveis |

A Tabela B-3 mostra símbolos que aparecem no contexto de um caminho através da hierarquia de módulos para um item.

Tabela B-3: Sintaxe Relacionada a Caminhos

| Símbolo | Explicação |
|--------|-------------|
| `ident::ident` | Caminho do namespace |
| `::path` | Caminho relativo à raiz da crate (ou seja, um caminho explicitamente absoluto) |
| `self::path` | Caminho relativo ao módulo atual (ou seja, um caminho explicitamente relativo) |
| `super::path` | Caminho relativo ao pai do módulo atual |
| `type::ident`, `<type as trait>::ident` | Constantes, funções e tipos associados |
| `<type>::...` | Método associado para um tipo que não pode ser nomeado diretamente (por exemplo, `<*const T>::...`) |
| `trait::method(...)` | Desambiguar uma chamada de método nomeando o trait que o define |
| `type::method(...)` | Desambiguar uma chamada de método nomeando o tipo para o qual ele é definido |
| `<type as trait>::method(...)` | Desambiguar uma chamada de método nomeando o trait e o tipo |

A Tabela B-4 mostra símbolos que aparecem no contexto do uso de parâmetros de tipo genérico.

Tabela B-4: Genéricos

| Símbolo | Explicação |
|--------|-------------|
| `path<...>` | Especifica parâmetros para o tipo genérico em um tipo (por exemplo, `Vec<u8>`) |
| `path::<...>`, `method::<...>` | Especifica parâmetros para o tipo, função ou método genérico em uma expressão; frequentemente referido como turbofish (por exemplo, `"42".parse::<i32>()`) |
| `fn ident<...> ...` | Define função genérica |
| `struct ident<...> ...` | Define estrutura genérica |
| `enum ident<...> ...` | Define enumeração genérica |
| `impl<...> ...` | Define implementação genérica |
| `for<...> type` | Limites de tempo de vida de classificação superior (Higher-ranked lifetime bounds) |
| `type<ident=type>` | Um tipo genérico onde um ou mais tipos associados têm atribuições específicas (por exemplo, `Iterator<Item=T>`) |

A Tabela B-5 mostra símbolos que aparecem no contexto de restringir parâmetros de tipo genérico com limites de trait.

Tabela B-5: Restrições de Limite de Trait

| Símbolo | Explicação |
|--------|-------------|
| `T: U` | Parâmetro genérico `T` restrito a tipos que implementam `U` |
| `T: 'a` | Tipo genérico `T` deve sobreviver ao tempo de vida `'a` (significando que o tipo não pode conter transitivamente quaisquer referências com tempos de vida menores que `'a`) |
| `T: ?Sized` | Permitir que o parâmetro de tipo genérico seja um tipo de tamanho dinâmico |
| `'a: 'b` | Tempo de vida `'a` deve sobreviver ao tempo de vida `'b` |

A Tabela B-6 mostra símbolos que aparecem no contexto de chamar ou definir macros e especificar atributos em um item.

Tabela B-6: Macros e Atributos

| Símbolo | Explicação |
|--------|-------------|
| `#[meta]` | Atributo externo |
| `#![meta]` | Atributo interno |
| `$ident` | Substituição de macro |
| `$ident:kind` | Captura de macro |
| `$(...)…` | Repetição de macro |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Invocação de macro |

A Tabela B-7 mostra símbolos que criam comentários.

Tabela B-7: Comentários

| Símbolo | Explicação |
|--------|-------------|
| `//` | Comentário de linha |
| `//!` | Comentário de documento de linha interna |
| `///` | Comentário de documento de linha externa |
| `/*...*/` | Comentário de bloco |
| `/*!...*/` | Comentário de documento de bloco interno |
| `/**...*/` | Comentário de documento de bloco externo |

A Tabela B-8 mostra símbolos que aparecem no contexto do uso de tuplas.

Tabela B-8: Tuplas

| Símbolo | Explicação |
|--------|-------------|
| `()` | Tupla vazia (também conhecida como unidade), tanto literal quanto tipo |
| `(expr)` | Expressão entre parênteses |
| `(type)` | Tipo entre parênteses |
| `(expr, ...)` | Expressão de tupla |
| `(type, ...)` | Tipo de tupla |
| `expr(expr, ...)` | Expressão de chamada de função; também usado para inicializar structs de tupla e variantes de enum de tupla |
| `ident.0`, `ident.1`, etc. | Acesso a campo de tupla |

A Tabela B-9 mostra os contextos em que colchetes são usados.

Tabela B-9: Colchetes

| Símbolo | Explicação |
|--------|-------------|
| `[...]` | Literal de array |
| `[expr; len]` | Literal de array contendo `len` cópias de `expr` |
| `[type; len]` | Tipo de array contendo `len` instâncias de `type` |
| `expr[expr]` | Indexação de coleção. Sobrecarga (`Index`, `IndexMut`) |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Indexação de coleção fingindo ser fatias de coleção, usando `Range`, `RangeFrom`, `RangeTo` ou `RangeFull` como o "índice" |
