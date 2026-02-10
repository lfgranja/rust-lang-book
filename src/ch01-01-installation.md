# Instalação

O primeiro passo é instalar o Rust. Faremos o download do Rust através do `rustup`, uma
ferramenta de linha de comando para gerenciar versões do Rust e ferramentas associadas. Você precisará
de uma conexão com a internet para o download.

> Nota: Se você preferir não usar o `rustup` por algum motivo, consulte a
> página de [Outros Métodos de Instalação do Rust](https://forge.rust-lang.org/infra/other-installation-methods.html) para mais opções.

Os passos a seguir instalam a versão estável mais recente do compilador Rust.
As garantias de estabilidade do Rust asseguram que todos os exemplos no livro que
compilam continuarão a compilar com versões mais recentes do Rust. A saída pode
diferir ligeiramente entre as versões porque o Rust frequentemente melhora mensagens de erro e
avisos. Em outras palavras, qualquer versão estável mais recente do Rust que você instalar usando
estes passos deve funcionar conforme o esperado com o conteúdo deste livro.

> ### Notação de Linha de Comando
>
> Neste capítulo e ao longo do livro, mostraremos alguns comandos usados no
> terminal. Linhas que você deve digitar em um terminal começam todas com `$`. Você
> não precisa digitar o caractere `$`; ele é o prompt da linha de comando mostrado para
> indicar o início de cada comando. Linhas que não começam com `$` tipicamente
> mostram a saída do comando anterior. Além disso, exemplos específicos para PowerShell
> usarão `>` em vez de `$`.

### Instalando `rustup` no Linux ou macOS

Se você está usando Linux ou macOS, abra um terminal e digite o seguinte comando:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

O comando baixa um script e inicia a instalação da ferramenta `rustup`,
que instala a versão estável mais recente do Rust. Você pode ser solicitado
a digitar sua senha. Se a instalação for bem-sucedida, a seguinte linha aparecerá:

```text
Rust is installed now. Great!
```

Você também precisará de um _linker_ (ligador), que é um programa que o Rust usa para juntar suas
saídas compiladas em um único arquivo. É provável que você já tenha um. Se você receber
erros de linker, você deve instalar um compilador C, que tipicamente incluirá um
linker. Um compilador C também é útil porque alguns pacotes Rust comuns dependem de
código C e precisarão de um compilador C.

No macOS, você pode obter um compilador C executando:

```console
$ xcode-select --install
```

Usuários de Linux geralmente devem instalar GCC ou Clang, de acordo com a documentação de sua
distribuição. Por exemplo, se você usa Ubuntu, pode instalar
o pacote `build-essential`.

### Instalando `rustup` no Windows

No Windows, vá para [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)<!-- ignore
--> e siga as instruções para instalar o Rust. Em algum momento da
instalação, você será solicitado a instalar o Visual Studio. Isso fornece um
linker e as bibliotecas nativas necessárias para compilar programas. Se você precisar de mais
ajuda com este passo, veja
[https://rust-lang.github.io/rustup/installation/windows-msvc.html](https://rust-lang.github.io/rustup/installation/windows-msvc.html)<!--
ignore -->.

O restante deste livro usa comandos que funcionam tanto no _cmd.exe_ quanto no PowerShell.
Se houver diferenças específicas, explicaremos qual usar.

### Solução de Problemas

Para verificar se você tem o Rust instalado corretamente, abra um shell e digite esta
linha:

```console
$ rustc --version
```

Você deve ver o número da versão, hash do commit e data do commit para a versão estável
mais recente que foi lançada, no seguinte formato:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Se você vir esta informação, você instalou o Rust com sucesso! Se você não
vir esta informação, verifique se o Rust está na sua variável de sistema `%PATH%` como
segue.

No CMD do Windows, use:

```console
> echo %PATH%
```

No PowerShell, use:

```powershell
> echo $env:Path
```

No Linux e macOS, use:

```console
$ echo $PATH
```

Se tudo estiver correto e o Rust ainda não estiver funcionando, há vários
lugares onde você pode obter ajuda. Descubra como entrar em contato com outros Rustaceans (um
apelido bobo que usamos para nós mesmos) na [página da comunidade](https://www.rust-lang.org/community).

### Atualizando e Desinstalando

Uma vez que o Rust esteja instalado via `rustup`, atualizar para uma versão recém-lançada é
fácil. Do seu shell, execute o seguinte script de atualização:

```console
$ rustup update
```

Para desinstalar o Rust e o `rustup`, execute o seguinte script de desinstalação do seu
shell:

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### Lendo a Documentação Local

A instalação do Rust também inclui uma cópia local da documentação para
que você possa lê-la offline. Execute `rustup doc` para abrir a documentação local
no seu navegador.

Sempre que um tipo ou função for fornecido pela biblioteca padrão e você não
tiver certeza do que faz ou como usar, use a documentação da interface de programação de aplicações
(API) para descobrir!

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### Usando Editores de Texto e IDEs

Este livro não faz suposições sobre quais ferramentas você usa para escrever código Rust.
Quase qualquer editor de texto fará o trabalho! No entanto, muitos editores de texto e
ambientes de desenvolvimento integrado (IDEs) têm suporte nativo para Rust. Você
pode sempre encontrar uma lista razoavelmente atual de muitos editores e IDEs na [página de ferramentas](https://www.rust-lang.org/tools) no site do Rust.

### Trabalhando Offline com Este Livro

Em vários exemplos, usaremos pacotes Rust além da biblioteca padrão. Para
trabalhar nesses exemplos, você precisará ter uma conexão com a internet
ou ter baixado essas dependências antecipadamente. Para baixar as
dependências antecipadamente, você pode executar os seguintes comandos. (Explicaremos
o que é o `cargo` e o que cada um desses comandos faz em detalhes mais tarde.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Isso fará o cache dos downloads para esses pacotes para que você não precise
baixá-los mais tarde. Uma vez que você tenha executado este comando, você não precisa manter a
pasta `get-dependencies`. Se você executou este comando, pode usar a
flag `--offline` com todos os comandos `cargo` no restante do livro para usar essas
versões em cache em vez de tentar usar a rede.
