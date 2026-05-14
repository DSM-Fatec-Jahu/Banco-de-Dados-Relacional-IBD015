# Projeto AnimeTube: Levantamento de Requisitos

## Como ler este documento

Este documento descreve um sistema que você vai modelar e criar no banco de dados. Ele foi escrito para ser lido com calma, do começo ao fim.

Algumas dicas antes de começar:

- Leia uma seção de cada vez. Não precisa entender tudo de uma vez.
- Cada **tabela** que você precisa criar está descrita em uma seção própria, com um título claro.
- Toda vez que aparecer uma palavra técnica, ela vem explicada logo ao lado, entre parênteses.
- No final do documento há um **glossário** com as palavras mais importantes.
- Você **não precisa tomar decisões difíceis de modelagem** neste projeto. As escolhas já estão prontas e explicadas. Seu trabalho é entender o modelo e escrever o SQL que cria as tabelas.

Se em algum momento algo não ficar claro, anote a dúvida e pergunte ao professor. Perguntar faz parte da atividade.

---

## 1. Visão Geral do Sistema

O **AnimeTube** é um site onde uma pessoa organiza os animes que ela assiste.

Funciona assim: o usuário se cadastra no site. Depois, ele monta um catálogo dos animes que conhece. Para cada anime, ele pode marcar se já assistiu, se está assistindo ou se quer assistir. Ele também pode dar uma nota e organizar os animes em listas (por exemplo, uma lista chamada "Favoritos" e outra chamada "Para assistir nas férias").

O sistema é simples e pessoal. Cada usuário vê e organiza apenas os seus próprios animes. Não há pagamento, não há vídeo sendo hospedado. O AnimeTube só guarda **informações** sobre os animes: nome, ano, gênero, e o que o usuário marcou sobre cada um.

### 1.1 O que o sistema guarda

O sistema vai guardar cinco tipos de informação. Cada tipo vira uma tabela no banco de dados:

1. **Usuários**: as pessoas que usam o site.
2. **Gêneros**: os tipos de anime (ação, comédia, romance, etc.).
3. **Animes**: o catálogo de animes, com nome, ano e gênero.
4. **Listas**: as listas que cada usuário cria para organizar seus animes.
5. **Animes em listas**: a ligação que diz quais animes estão em quais listas.

Não se preocupe agora em entender como essas tabelas se conectam. As seções a seguir explicam cada uma com detalhe.

---

## 2. Atores do Sistema

Ator é o nome que damos para quem usa o sistema.

| Ator | Descrição | Faz Login? |
|------|-----------|:----------:|
| **Usuário** | Pessoa que se cadastra no site para organizar seus animes. | Sim |

Este sistema é simples e tem apenas um tipo de usuário. Não há administrador, não há outros perfis.

---

## 3. Requisitos Funcionais (RF)

Requisito funcional é cada coisa que o sistema **precisa fazer**. Cada requisito abaixo está ligado a uma ou mais tabelas que você vai criar.

### 3.1 Cadastro de Usuário

**RF01. Cadastro de Usuário**

O sistema deve permitir que uma pessoa se cadastre. Cada usuário tem as seguintes informações:

- **Nome**: o nome da pessoa.
- **E-mail**: o e-mail da pessoa. Cada e-mail só pode ser usado por um usuário (dois usuários não podem ter o mesmo e-mail).
- **Senha**: a senha para entrar no site. A senha deve ser guardada como *hash* (um texto embaralhado, para que ninguém consiga ler a senha verdadeira olhando o banco de dados).

Isso vira uma tabela chamada **usuario**.

### 3.2 Gêneros de Anime

**RF02. Cadastro de Gêneros**

O sistema deve guardar os gêneros de anime. Gênero é o tipo do anime: ação, comédia, romance, terror, e assim por diante.

Cada gênero tem apenas uma informação:

- **Nome**: o nome do gênero (por exemplo: "Ação").

Isso vira uma tabela chamada **genero**.

> **Por que o gênero é uma tabela separada?**
> Poderíamos escrever o gênero direto na tabela de animes, como um texto. Mas aí, se duas pessoas escrevessem "Ação" e "ação", o sistema entenderia como dois gêneros diferentes. Tendo uma tabela só de gêneros, cada gênero existe uma única vez e fica organizado. Esta decisão já está tomada: **gênero é uma tabela**.

### 3.3 Catálogo de Animes

**RF03. Cadastro de Animes**

O sistema deve permitir que o usuário cadastre os animes que ele conhece. Cada anime tem as seguintes informações:

- **Título**: o nome do anime (por exemplo: "Naruto").
- **Ano de lançamento**: o ano em que o anime foi lançado.
- **Quantidade de episódios**: quantos episódios o anime tem.
- **Gênero**: a qual gênero o anime pertence. Cada anime tem **um** gênero.

Cada anime pertence ao usuário que o cadastrou. Os animes de um usuário não aparecem para outro usuário.

Isso vira uma tabela chamada **anime**.

> **Como o anime se liga ao gênero e ao usuário?**
> A tabela de animes vai guardar duas ligações:
> - Uma ligação para a tabela **genero**, dizendo qual é o gênero daquele anime.
> - Uma ligação para a tabela **usuario**, dizendo de quem é aquele anime.
> Essas ligações se chamam **chave estrangeira**. O glossário no final explica esse termo com mais calma.

### 3.4 Marcar o Status do Anime

**RF04. Status de Cada Anime**

Para cada anime cadastrado, o usuário marca em que situação ele está. O **status** pode ser um destes três valores:

- **Quero assistir**: o usuário ainda não viu o anime.
- **Assistindo**: o usuário está vendo o anime agora.
- **Assistido**: o usuário já terminou o anime.

O status faz parte da tabela **anime**. Não é uma tabela nova. É só mais uma informação dentro de cada anime, ao lado do título e do ano.

### 3.5 Dar Nota a um Anime

**RF05. Nota do Anime**

O usuário pode dar uma nota para cada anime. A nota é um número de 0 a 10.

A nota também faz parte da tabela **anime**. Não é uma tabela nova.

Um anime pode ainda não ter nota (por exemplo, um anime que o usuário ainda não assistiu). Nesse caso, o campo da nota fica vazio.

### 3.6 Criar Listas

**RF06. Criar Listas de Animes**

O usuário pode criar listas para organizar seus animes. Por exemplo: uma lista chamada "Favoritos", outra chamada "Para assistir nas férias".

Cada lista tem as seguintes informações:

- **Nome**: o nome da lista.
- **Usuário**: de quem é a lista. Cada lista pertence a um usuário.

Isso vira uma tabela chamada **lista**.

### 3.7 Colocar Animes nas Listas

**RF07. Adicionar Animes a uma Lista**

O usuário pode colocar seus animes dentro das listas que ele criou.

Aqui acontece uma coisa importante:

- Um mesmo anime pode estar em **várias listas** ao mesmo tempo (por exemplo, o anime "One Piece" pode estar na lista "Favoritos" e também na lista "Maiores animes").
- Uma mesma lista pode ter **vários animes** dentro dela.

Quando uma situação é assim (muitos de um lado, muitos do outro), precisamos de uma tabela só para guardar essas ligações.

Isso vira uma tabela chamada **anime_lista**. Cada linha dessa tabela diz "este anime está nesta lista". Ela guarda:

- **Anime**: qual anime.
- **Lista**: em qual lista ele está.

> **Por que essa tabela existe?**
> Pense numa tabela comum: ela tem um anime por linha, ou uma lista por linha. Mas aqui precisamos guardar **a ligação entre os dois**. Como um anime pode estar em muitas listas e uma lista pode ter muitos animes, a única forma de organizar isso é uma tabela no meio, que só serve para ligar. Cada linha dela é uma ligação. Esta decisão já está tomada: **a tabela anime_lista deve existir**.

---

## 4. Requisitos Não Funcionais (RNF)

Requisito não funcional é uma qualidade que o sistema precisa ter. Ele **não vira tabela** no banco de dados. Está aqui só para você conhecer o sistema por inteiro.

- **RNF01**: A senha do usuário deve ser guardada como hash, nunca como texto normal.
- **RNF02**: O site deve funcionar nos navegadores comuns (Chrome, Firefox, Edge).
- **RNF03**: As telas do site devem ser simples e fáceis de usar.

---

## 5. Resumo das Tabelas

Este é o resumo de tudo que você vai criar. São **cinco tabelas**.

| Tabela | O que guarda | Liga-se a |
|--------|--------------|-----------|
| **usuario** | As pessoas que usam o site. | (não se liga a nenhuma outra) |
| **genero** | Os tipos de anime. | (não se liga a nenhuma outra) |
| **anime** | O catálogo de animes de cada usuário. | Liga-se a **usuario** e a **genero**. |
| **lista** | As listas criadas pelos usuários. | Liga-se a **usuario**. |
| **anime_lista** | Diz quais animes estão em quais listas. | Liga-se a **anime** e a **lista**. |

### 5.1 Ordem para criar as tabelas

No SQL, uma tabela só pode se ligar a outra que **já existe**. Por isso, crie as tabelas nesta ordem:

1. Primeiro **usuario** e **genero** (elas não dependem de ninguém).
2. Depois **anime** (ela depende de usuario e de genero).
3. Depois **lista** (ela depende de usuario).
4. Por último **anime_lista** (ela depende de anime e de lista).

Se você criar fora dessa ordem, o banco de dados vai dar erro, porque vai tentar se ligar a uma tabela que ainda não foi criada.

### 5.2 Lembretes das convenções da disciplina

Use as mesmas convenções que você aprendeu nas aulas e que estão no GitHub da disciplina. Os pontos mais importantes:

- Toda tabela tem uma **chave primária** com o nome no padrão `id_nome_da_tabela` (por exemplo: `id_usuario`, `id_anime`).
- Toda tabela tem os dois **campos de auditoria** obrigatórios: `criado_em` e `atualizado_em`.
- Use `utf8mb4_unicode_ci` como collation.
- O banco de dados alvo é o **MariaDB 10.4** (XAMPP).
- Comece o arquivo com os comandos de limpeza (`DROP`) antes dos comandos de criação (`CREATE`).

### 5.3 Decisão opcional (Soft-delete)

O *soft-delete* (apagar um registro apenas marcando uma data no campo `deletado_em`, sem remover de verdade) é uma decisão livre, conforme a Seção 4.1 do enunciado da atividade. Você pode usar em todas as tabelas, em algumas, ou em nenhuma. Se usar, escreva o motivo no bloco de comentários do SQL. Se preferir não usar, também está tudo bem para este projeto.

---

## 6. Glossário

As palavras mais importantes deste documento, explicadas de forma simples.

- **Tabela**: um lugar no banco de dados que guarda um tipo de informação. Por exemplo, a tabela `usuario` guarda os usuários.

- **Registro** (ou linha): uma informação dentro de uma tabela. Por exemplo, cada usuário cadastrado é um registro na tabela `usuario`.

- **Campo** (ou coluna): cada pedaço de informação de um registro. Por exemplo, o nome e o e-mail são campos da tabela `usuario`.

- **Chave primária**: o campo que identifica cada registro de forma única. Dois registros nunca têm a mesma chave primária. É como o número de matrícula de um aluno: cada aluno tem o seu, e ninguém repete.

- **Chave estrangeira**: um campo que liga um registro de uma tabela a um registro de outra tabela. Por exemplo, na tabela `anime`, a chave estrangeira do gênero diz a qual gênero aquele anime pertence.

- **Hash**: um texto embaralhado. Usamos para guardar senhas. Mesmo quem olhar o banco de dados não consegue descobrir a senha verdadeira.

- **Status**: a situação em que algo está. No AnimeTube, o status de um anime diz se o usuário quer assistir, está assistindo ou já assistiu.

- **Auditoria**: guardar quando um registro foi criado (`criado_em`) e quando foi alterado pela última vez (`atualizado_em`). Serve para saber a história de cada registro.

- **Soft-delete**: apagar um registro sem remover de verdade. Em vez de sumir com a informação, o sistema só marca a data em que ela foi "apagada", no campo `deletado_em`.

- **DDL**: o grupo de comandos SQL que cria e organiza as tabelas. Os principais são `CREATE` (cria) e `DROP` (apaga).

- **MariaDB**: o programa de banco de dados que vamos usar. Ele roda dentro do XAMPP no seu computador.

---

*Atividade da disciplina IBD015 (Banco de Dados Relacional), Fatec Jahu, Prof. Ronan Adriel Zenatti.*
