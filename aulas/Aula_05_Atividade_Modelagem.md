# Aula 05 — Atividade Prática: Modelagem de um Sistema de Streaming Integrado

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 04](./Aula_04_SQL_DML.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_06_SQL_Consultas_Basicas.md)

---

## 📌 Objetivos da Aula

Esta é uma **aula de atividade prática** — o conteúdo teórico foi apresentado nas aulas anteriores. Aqui você consolidará os aprendizados das Aulas 01 a 03 desenvolvendo o modelo conceitual e o script DDL de um sistema de streaming integrado. Esta atividade corresponde ao **Trabalho T1** da disciplina e **vale 2,0 pontos** da média final.

---

## 📋 Descrição do Trabalho T1

Você deverá projetar e implementar o banco de dados da **StreamAll**, uma plataforma de entretenimento digital que unifica **música**, **filmes** e **séries** em um único serviço — combinando o melhor de plataformas como Spotify e Netflix.

A entrega inclui dois produtos:

1. **Diagrama Conceitual (MER):** Usando um arquivo mermaid, dbdiagrams, draw.io ou equivalente.
2. **Script DDL completo e funcional:** Um Arquivo chamado trabalho.sql com seu nome completo sendo a primeira linha via comentário.
   **NÃO COMPACTE OS ARQUIVOS!**

> ⚠️ **A entrega deve ser feita pela atividade específica criada no Google Classroom da turma.** Não envie por e-mail nem por outros canais.

---

## 1. Contexto do Sistema

A StreamAll é uma plataforma de assinatura que oferece acesso ilimitado a músicas, filmes e séries. Os usuários consomem conteúdo, organizam suas experiências por meio de playlists personalizadas e interagem com o catálogo por meio de avaliações e curtidas.

O grande diferencial da plataforma é que **uma playlist pode conter qualquer combinação de músicas, episódios de séries e filmes**, permitindo ao usuário montar sequências de entretenimento completamente livres — uma playlist de "Tarde de domingo", por exemplo, pode começar com músicas, continuar com um episódio de série e terminar com um filme.

---

## 2. Regras de Negócio

Leia atentamente **todas** as regras antes de iniciar qualquer etapa da modelagem. As regras de negócio determinam a cardinalidade de cada relacionamento, a necessidade de generalização e especialização, a existência de entidades fracas e quais atributos pertencem a cada entidade.

### 2.1 Usuários e Assinaturas

- Cada usuário possui nome completo, e-mail (único no sistema), CPF (único no sistema), data de nascimento e um nome de exibição escolhido por ele.
- Um usuário pode ter vários endereços de cobrança cadastrados; cada endereço pertence a exatamente um usuário.
- A plataforma oferece diferentes planos de assinatura. Cada plano tem nome, preço mensal e um conjunto de permissões que define qualidade máxima de áudio, qualidade máxima de vídeo e número de telas simultâneas permitidas.
- Um usuário possui no máximo uma assinatura ativa por vez. O histórico completo de assinaturas de um usuário deve ser preservado, com as datas de início e término de cada período e o status de cada assinatura (ativa, cancelada, suspensa).
- Um usuário pode configurar vários idiomas de preferência para legendas e áudio; o sistema deve registrar quais idiomas cada usuário configurou.

### 2.2 Catálogo — Conteúdo Geral

- A plataforma disponibiliza três tipos de conteúdo reproduzível: **músicas**, **filmes** e **episódios de séries**.
- Todo conteúdo reproduzível — independentemente do tipo — possui título, duração em segundos e data de lançamento.
- Além dos atributos comuns, cada tipo de conteúdo possui atributos e relacionamentos próprios, conforme detalhado nas seções seguintes.
- Todo conteúdo pode ser classificado por um ou mais gêneros. Um gênero pode classificar qualquer tipo de conteúdo — músicas, filmes e episódios compartilham o mesmo catálogo de gêneros da plataforma.

### 2.3 Catálogo — Músicas

- Cada música pertence a exatamente um álbum e pode ter uma letra cadastrada (opcional).
- Um álbum possui título, data de lançamento e URL da imagem de capa.
- Um álbum é lançado por exatamente um artista.
- Um artista possui nome artístico, país de origem e biografia (opcional). Um artista pode ter vários nomes alternativos registrados — como nome real, apelidos ou nomes anteriores de carreira.

### 2.4 Catálogo — Filmes

- Cada filme possui sinopse (opcional) e classificação etária.
- Um filme pode ter vários diretores; um diretor pode ter dirigido vários filmes.
- Um filme possui um elenco composto por vários atores. Para cada participação de um ator em um filme, o sistema registra o nome do personagem interpretado e a posição nos créditos (protagonista, coadjuvante ou participação especial).
- Diretores e atores são profissionais do audiovisual. Todo profissional do audiovisual possui nome, data de nascimento e nacionalidade.
- Um profissional pode atuar como diretor em alguns filmes e como ator em outros ao longo de sua carreira.

### 2.5 Catálogo — Séries e Episódios

- Cada série possui título, sinopse (opcional), classificação etária e status de produção (em andamento, encerrada ou cancelada).
- Uma série é organizada em temporadas. Cada temporada pertence a exatamente uma série, possui número de ordem e ano de lançamento, e não tem existência independente da série à qual pertence.
- Cada episódio pertence a exatamente uma temporada, possui número de ordem dentro da temporada e sinopse (opcional), e não tem existência independente da temporada à qual pertence.
- Uma série pode ter vários gêneros associados, compartilhando o mesmo catálogo de gêneros da plataforma.

### 2.6 Playlists

- Um usuário pode criar nenhuma ou muitas playlists.
- Cada playlist pertence a exatamente um usuário criador e possui nome, descrição (opcional), visibilidade (pública ou privada) e data de criação.
- Uma playlist pode conter músicas, filmes e episódios de séries em qualquer combinação. A ordem de cada item dentro da playlist deve ser registrada.
- Um usuário pode seguir playlists públicas criadas por outros usuários. O sistema deve registrar a data em que cada seguimento teve início.

### 2.7 Histórico, Avaliações e Curtidas (BÔNUS, NÃO OBRIGATÓRIO)

- O sistema registra o histórico de reprodução de cada usuário. Para cada reprodução, registra-se qual conteúdo foi reproduzido, quando, quantos segundos foram efetivamente assistidos ou ouvidos e em qual tipo de dispositivo (celular, televisão, computador ou tablet).
- Um usuário pode avaliar filmes e séries com uma nota de 1 a 5 e um comentário opcional. Cada usuário avalia cada filme ou série no máximo uma vez.
- Um usuário pode curtir músicas individualmente. O sistema deve registrar quais músicas cada usuário curtiu e quando.

---

## 3. O que deve ser entregue

### Entrega 1 — Diagrama Conceitual (MER)

O diagrama deve representar fielmente todas as regras de negócio da Seção 2 e conter:

- Todas as entidades identificadas
- Os atributos de cada entidade, com identificação do tipo e da chave identificadora (PK ou FK).
- Todos os relacionamentos, cardinalidade em ambos os lados
- A hierarquia de generalização e especialização onde aplicável.

### Entrega 2 — Script DDL

O script deve ser executável do zero, seguindo as melhores práticas abordadas em aula e conter :

- Criação do banco de dados. 
- Criação de todas as tabelas.
- Tipos de dados adequados para cada coluna, conforme critérios discutidos em aula.
- Todas as constraints pertinentes: chaves primárias, chaves estrangeiras com ações de integridade referencial coerentes com cada relacionamento, restrições de unicidade, restrições de domínio e valores padrão onde aplicável
- Seguimento rigoroso de todas as convenções de nomenclatura da disciplina
- Comentários que identifiquem seções e justifiquem decisões de projeto não óbvias
- Apague o banco e tabelas antes de cada criação para garantir a criação completa da versão final do script SQL.

---
 **NÃO COMPACTE OS ARQUIVOS!**
---

## 4. Critérios de Avaliação do T1

| Critério | Pontos |
|---|---|
| Diagrama Conceitual — entidades, atributos, relacionamentos e cardinalidades | 0,6 |
| Generalização e especialização — hierarquia correta com restrições adequadas e justificadas | 0,4 |
| Script DDL — funcional, sem erros de sintaxe, tipos de dados adequados e criação do banco | 0,4 |
| Integridade referencial — chaves estrangeiras com ações coerentes, constraints de unicidade, domínio e valores padrão | 0,3 |
| Convenções de nomenclatura, organização e comentários | 0,3 |
| **Total** | **2,0** |

> 💡 **Antes de entregar:** execute o script do zero em um banco limpo e verifique se todas as tabelas são criadas sem erros. Um script que não executa compromete diretamente a avaliação do DDL e da integridade referencial.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
