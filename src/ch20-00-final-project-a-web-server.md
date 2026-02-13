# Projeto Final: Construindo um Servidor Web Multithreaded

Chegamos ao fim da nossa jornada! Neste capítulo final, vamos usar tudo o que aprendemos para construir um projeto maior. Vamos construir um servidor web que diz "olá", e vamos fazer isso usando threads para que o servidor possa lidar com múltiplas requisições ao mesmo tempo.

Aqui está o plano:

1. Aprender um pouco sobre TCP e HTTP.
2. Ouvir conexões TCP em um soquete.
3. Analisar um pequeno número de requisições HTTP.
4. Criar uma resposta HTTP adequada.
5. Melhorar a taxa de transferência do nosso servidor com um pool de threads.

Mas antes de começarmos, preciso mencionar um pequeno detalhe: não vamos construir um servidor web *completo* e pronto para produção. O objetivo é ensinar conceitos de Rust, não como escrever o servidor web mais rápido e seguro do mundo. Vamos manter as coisas simples e focar nas partes que nos ajudam a praticar o que aprendemos.

Vamos começar!
