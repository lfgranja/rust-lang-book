# Características de Linguagens Orientadas a Objetos

Não existe um consenso na comunidade de programação sobre quais características uma linguagem deve ter para ser considerada orientada a objetos. Rust é influenciado por muitos paradigmas de programação, incluindo POO; por exemplo, exploramos as funcionalidades de programação funcional no Capítulo 13. Provavelmente, POO é o paradigma mais comum de desenvolvimento de software atualmente. As linguagens POO compartilham certas características comuns, a saber: objetos, encapsulamento e herança. Vamos ver o que cada uma dessas características significa e se Rust as suporta.

## Objetos Contêm Dados e Comportamento

O livro *Design Patterns: Elements of Reusable Object-Oriented Software*, escrito por Erich Gamma, Richard Helm, Ralph Johnson e John Vlissides (conhecidos como a "Gangue dos Quatro" ou GoF), define POO desta maneira:

> Programas orientados a objetos são feitos de objetos. Um objeto empacota dados e os procedimentos que operam sobre esses dados. Os procedimentos são tipicamente chamados de *métodos* ou *operações*.

Usando essa definição, Rust é orientado a objetos: struct e enums têm dados, e blocos `impl` fornecem métodos em structs e enums. Embora structs e enums com métodos não sejam chamados de *objetos*, eles fornecem a mesma funcionalidade, de acordo com a definição de objetos da GoF.

## Encapsulamento que Oculta Detalhes de Implementação

Outro aspecto comumente associado à POO é a ideia de *encapsulamento*, o que significa que os detalhes de implementação de um objeto não são acessíveis ao código que usa esse objeto. Portanto, a única maneira de interagir com o objeto é através de sua API pública; o código que usa o objeto não deve ser capaz de alcançar as partes internas do objeto e alterar dados ou comportamento diretamente. Isso permite que o programador altere e refatore as partes internas de um objeto sem precisar alterar o código que usa o objeto.

Discutimos como controlar o encapsulamento no Capítulo 7: podemos usar a palavra-chave `pub` para decidir quais módulos, tipos, funções e métodos em nosso código devem ser públicos, e por padrão tudo é privado. Por exemplo, podemos definir uma struct `MediaColecao` que tem um campo contendo um vetor de inteiros `Vec<i32>`. A struct também pode ter um campo que contém a média dos valores na lista, o que significa que, quando um valor é adicionado ou removido da lista, a média também deve ser atualizada.

```rust
pub struct MediaColecao {
    lista: Vec<i32>,
    media: f64,
}

impl MediaColecao {
    pub fn add(&mut self, valor: i32) {
        self.lista.push(valor);
        self.atualizar_media();
    }

    pub fn remover(&mut self) -> Option<i32> {
        let resultado = self.lista.pop();
        match resultado {
            Some(valor) => {
                self.atualizar_media();
                Some(valor)
            }
            None => None,
        }
    }

    pub fn media(&self) -> f64 {
        self.media
    }

    fn atualizar_media(&mut self) {
        let total: i32 = self.lista.iter().sum();
        self.media = total as f64 / self.lista.len() as f64;
    }
}
```

A struct `MediaColecao` mantém uma lista de inteiros e a média dos itens na lista.

## Herança como um Sistema de Tipos e de Compartilhamento de Código

*Herança* é um mecanismo pelo qual um objeto pode herdar elementos da definição de outro objeto, ganhando assim os dados e o comportamento do outro objeto sem ter que defini-los novamente.

Se uma linguagem deve ter herança para ser uma linguagem orientada a objetos, então Rust não é uma. Não há como definir uma struct que herde os campos e implementações de métodos de outra struct. No entanto, se você está acostumado a usar herança em sua programação, pode usar outras soluções em Rust, dependendo do motivo pelo qual você queria herança em primeiro lugar.

Você escolheria herança por dois motivos principais. Um é para reutilização de código: você pode implementar um comportamento específico para um tipo, e a herança permite reutilizar essa implementação para um tipo diferente. Você pode compartilhar código em Rust usando implementações de métodos padrão em [[Traits]], que você viu no Listagem 10-14 quando adicionamos uma implementação padrão ao método `resumir` na trait `Resumo`. Qualquer tipo que implemente a trait `Resumo` teria o método `resumir` disponível sem qualquer código adicional. Isso é semelhante a uma classe pai tendo uma implementação de método e uma classe filha herdando essa implementação. Também podemos sobrescrever a implementação padrão do método `resumir` quando implementamos a trait `Resumo`, o que é semelhante a uma classe filha sobrescrevendo a implementação de um método herdado de uma classe pai.

O outro motivo para usar herança está relacionado ao sistema de tipos: permitir que um tipo filho seja usado nos mesmos lugares que o tipo pai. Isso também é chamado de *polimorfismo*, o que significa que você pode substituir vários objetos uns pelos outros em tempo de execução se eles compartilharem certas características.

> ### Polimorfismo
>
> Para muitas pessoas, polimorfismo é sinônimo de herança. Mas na verdade é um conceito mais geral que se refere a código que pode trabalhar com dados de múltiplos tipos. Para herança, esses tipos são geralmente subclasses.
>
> Rust usa genéricos para abstrair sobre diferentes tipos possíveis e *trait bounds* para impor restrições sobre o que esses tipos devem fornecer. Isso às vezes é chamado de *polimorfismo paramétrico limitado*.

A herança caiu em desuso recentemente como solução de design em muitas linguagens de programação porque muitas vezes corre o risco de compartilhar mais código do que o necessário. Subclasses não devem sempre compartilhar todas as características de sua classe pai, mas farão isso com herança. Isso pode tornar o design de um programa menos flexível. Também introduz a possibilidade de chamar métodos em subclasses que não fazem sentido ou que causam erros porque os métodos não se aplicam à subclasse. Além disso, algumas linguagens permitem apenas herança única (o que significa que uma subclasse pode herdar apenas de uma classe), restringindo ainda mais a flexibilidade do design de um programa.

Por essas razões, Rust adota uma abordagem diferente, usando [[Trait Objects]] em vez de herança. Vamos ver como os [[Trait Objects]] permitem o polimorfismo em Rust.
