<!-- Old headings. Do not remove or links may break. -->
<a id="comparing-performance-loops-vs-iterators"></a>

## Desempenho em Loops vs. Iteradores

Para determinar se deve usar loops ou iteradores, você precisa saber qual implementação é mais rápida: a versão da função `search` com um loop `for` explícito ou a versão com iteradores.

Executamos um benchmark carregando todo o conteúdo de *As Aventuras de Sherlock Holmes* de Sir Arthur Conan Doyle em uma `String` e procurando a palavra *the* no conteúdo. Aqui estão os resultados do benchmark na versão de `search` usando o loop `for` e na versão usando iteradores:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

As duas implementações têm desempenho semelhante! Não explicaremos o código de benchmark aqui porque o objetivo não é provar que as duas versões são equivalentes, mas obter uma noção geral de como essas duas implementações se comparam em termos de desempenho.

Para um benchmark mais abrangente, você deve verificar usando vários textos de vários tamanhos como `contents`, diferentes palavras e palavras de diferentes comprimentos como `query` e todos os tipos de outras variações. O ponto é este: Iteradores, embora uma abstração de alto nível, são compilados em aproximadamente o mesmo código como se você tivesse escrito o código de nível inferior você mesmo. Iteradores são uma das *abstrações de custo zero* do Rust, o que significa que usar a abstração não impõe nenhuma sobrecarga de tempo de execução adicional. Isso é análogo a como Bjarne Stroustrup, o designer e implementador original do C++, define custo zero em sua apresentação principal do ETAPS 2012 “Foundations of C++”:

> Em geral, as implementações de C++ obedecem ao princípio de custo zero: O que você não usa, você não paga. E ainda: O que você usa, você não poderia escrever melhor manualmente.

Em muitos casos, o código Rust usando iteradores compila para o mesmo assembly que você escreveria à mão. Otimizações como desenrolamento de loop e eliminação de verificação de limites no acesso a array se aplicam e tornam o código resultante extremamente eficiente. Agora que você sabe disso, pode usar iteradores e closures sem medo! Eles fazem o código parecer de nível superior, mas não impõem uma penalidade de desempenho em tempo de execução por fazê-lo.

## Resumo

Closures e iteradores são recursos do Rust inspirados em ideias de linguagens de programação funcional. Eles contribuem para a capacidade do Rust de expressar claramente ideias de alto nível com desempenho de baixo nível. As implementações de closures e iteradores são tais que o desempenho em tempo de execução não é afetado. Isso faz parte do objetivo do Rust de se esforçar para fornecer abstrações de custo zero.

Agora que melhoramos a expressividade do nosso projeto de E/S, vamos ver mais alguns recursos do `cargo` que nos ajudarão a compartilhar o projeto com o mundo.
