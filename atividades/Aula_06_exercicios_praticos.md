# Aula 06 — Exercícios Práticos: SQL Consultas Básicas

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> Banco de dados: `sakila_pt` · Base teórica: Aula 06 — SQL: Consultas Básicas

---

## Instruções Gerais

- Utilize **exclusivamente** o banco de dados `sakila_pt`.
- Todos os exercícios devem aplicar **filtro de soft-delete**: registros com `deletado_em IS NOT NULL` devem ser excluídos dos resultados, salvo indicação contrária.
- Use `AS` para nomear colunas quando o enunciado indicar um rótulo específico.
- Leia cada enunciado com atenção: o contexto de negócio faz parte do exercício.

---

## Bloco 1 — SELECT e Colunas (Exercícios 1 a 8)

---

### Exercício 1 — Catálogo de Filmes para o Site

A equipe de marketing precisa montar a página inicial da locadora com uma lista básica de filmes disponíveis. Exiba o **título**, a **classificação indicativa** e o **ano de lançamento** de todos os filmes ativos, ordenados alfabeticamente pelo título.

**Tabela:** `filmes`
**Conceitos:** `SELECT`, colunas específicas, `ORDER BY ASC`

---

### Exercício 2 — Ficha de Identificação de Atores

O setor de cadastro solicitou uma listagem completa dos atores para conferência de dados. Exiba o **id_ator**, o nome completo em uma única coluna chamada `nome_completo` (concatenando `primeiro_nome`, um espaço e `ultimo_nome`) e a data de `alterado_em`, para todos os atores ativos.

**Tabela:** `atores`
**Conceitos:** `SELECT`, `CONCAT`, alias com `AS`

---

### Exercício 3 — Classificações Disponíveis no Acervo

O gerente quer saber quais **classificações indicativas** distintas existem no acervo de filmes ativos. Exiba apenas os valores únicos da coluna `classificacao`, em ordem alfabética.

**Tabela:** `filmes`
**Conceitos:** `SELECT DISTINCT`, `ORDER BY`

---

### Exercício 4 — Relatório de Clientes Cadastrados

A atendente precisa de uma lista de clientes para realizar ligações de retenção. Exiba o **id_cliente**, o **primeiro_nome**, o **ultimo_nome** (renomeado como `sobrenome`) e o **email** de todos os clientes ativos, ordenados pelo sobrenome e depois pelo primeiro nome, ambos em ordem crescente.

**Tabela:** `clientes`
**Conceitos:** `SELECT`, alias, `ORDER BY` com múltiplos critérios

---

### Exercício 5 — Painel de Funcionários

O RH precisa de uma ficha resumida de todos os funcionários ativos. Exiba o **id_funcionario**, o nome completo (coluna `nome_funcionario`), o **email** e se está ativo (`ativo`). Ordene pelo id em ordem crescente.

**Tabela:** `funcionarios`
**Conceitos:** `SELECT`, `CONCAT`, alias, `ORDER BY`

---

### Exercício 6 — Países Cadastrados no Sistema

A equipe de expansão internacional precisa saber em quais países a locadora possui clientes. Liste todos os **nomes de países** ativos em ordem alfabética, exibindo também o **id_pais**.

**Tabela:** `paises`
**Conceitos:** `SELECT`, `ORDER BY`

---

### Exercício 7 — Tabela de Preços para Painel Interno

O setor financeiro precisa de uma tabela com os filmes, seus preços de aluguel e custos de reposição, formatados com dois rótulos claros. Exiba o **titulo** como `filme`, a **taxa_aluguel** como `preco_aluguel` e o **custo_reposicao** como `custo_repor`, apenas para filmes ativos. Ordene pelo custo de reposição de forma decrescente.

**Tabela:** `filmes`
**Conceitos:** `SELECT`, alias, `ORDER BY DESC`

---

### Exercício 8 — Lista de Categorias para o App

O desenvolvedor do aplicativo mobile precisa carregar as categorias disponíveis para exibição no menu de navegação. Liste o **id_categoria** e o **nome** de todas as categorias ativas, em ordem alfabética pelo nome.

**Tabela:** `categorias`
**Conceitos:** `SELECT`, `ORDER BY`

---

## Bloco 2 — WHERE com Operadores Relacionais e Lógicos (Exercícios 9 a 18)

---

### Exercício 9 — Filmes Longos para Sessão Especial

A locadora vai organizar uma "Sessão Especial de Longas" com filmes que tenham **duração superior a 150 minutos**. Liste o **titulo** e a **duracao** (em minutos) de todos os filmes ativos que atendam a esse critério, ordenados pela duração de forma decrescente.

**Tabela:** `filmes`
**Conceitos:** `WHERE`, operador `>`

---

### Exercício 10 — Filmes Baratos para Promoção

A locadora quer montar uma vitrine de filmes econômicos. Selecione o **titulo** e a **taxa_aluguel** dos filmes ativos com taxa de aluguel **menor ou igual a R$ 2,99**, em ordem crescente de preço.

**Tabela:** `filmes`
**Conceitos:** `WHERE`, operador `<=`

---

### Exercício 11 — Filtro por Classificação Indicativa

Um cliente ligou pedindo sugestões de filmes adequados para toda a família. Liste o **titulo**, a **classificacao** e a **duracao** de todos os filmes ativos classificados como `'G'` (livre para todos), ordenados pelo título.

**Tabela:** `filmes`
**Conceitos:** `WHERE` com igualdade em ENUM

---

### Exercício 12 — Clientes Inativos para Campanha de Reativação

O marketing quer contactar clientes que foram desativados (e que **não** foram deletados logicamente, isto é, `deletado_em IS NULL`). Exiba o **id_cliente**, o nome completo (`nome_completo`) e o **email** de todos os clientes com `ativo = 0` e `deletado_em IS NULL`, ordenados pelo sobrenome.

**Tabela:** `clientes`
**Conceitos:** `WHERE`, `CONCAT`, operador de igualdade com booleano, `IS NULL`

---

### Exercício 13 — Filmes de Duração Moderada com Aluguel Acessível

O gerente quer criar uma sessão de filmes "custo-benefício": duração entre 90 e 120 minutos **e** taxa de aluguel abaixo de R$ 3,00. Liste o **titulo**, a **duracao** e a **taxa_aluguel**, em ordem crescente de duração.

**Tabela:** `filmes`
**Conceitos:** `WHERE` com `AND`, múltiplos operadores relacionais

---

### Exercício 14 — Busca de Clientes por Duas Cidades

Um representante vai visitar pessoalmente clientes em duas cidades específicas (ids 1 e 2). Exiba o **primeiro_nome**, o **ultimo_nome** e o **endereco_id** dos clientes ativos dessas duas cidades. Use o campo `loja_id` para distinguir: mostre apenas clientes da `loja_id = 1` **ou** `loja_id = 2`.

**Tabela:** `clientes`
**Conceitos:** `WHERE` com `OR`

---

### Exercício 15 — Filmes Sem Descrição Cadastrada

A equipe de conteúdo precisa identificar filmes que ainda não têm descrição para priorizá-los na revisão editorial. Liste o **id_filme** e o **titulo** de todos os filmes ativos cuja `descricao` seja `NULL`.

**Tabela:** `filmes`
**Conceitos:** `IS NULL`

---

### Exercício 16 — Funcionários com E-mail Cadastrado

Para enviar comunicados internos, o RH precisa apenas dos funcionários que possuem e-mail cadastrado no sistema. Exiba o **id_funcionario**, o nome completo (`nome_funcionario`) e o **email** de todos os funcionários ativos com e-mail informado.

**Tabela:** `funcionarios`
**Conceitos:** `IS NOT NULL`, `CONCAT`, filtro de soft-delete

---

### Exercício 17 — Filmes Classificados como Adulto que Não São Muito Caros

Um cliente adulto quer alugar um filme classificado `'NC-17'` ou `'R'` mas não quer gastar mais de R$ 4,00. Liste o **titulo**, a **classificacao** e a **taxa_aluguel** dos filmes ativos que atendam a esses critérios, ordenados pelo preço crescente.

**Tabela:** `filmes`
**Conceitos:** `WHERE` com `OR` e `AND`, precedência de operadores lógicos

---

### Exercício 18 — Filmes com Recursos Extras para Destaque no App

A equipe de TI quer exibir um badge especial nos filmes que tenham o recurso `'Trailers'` em `recursos_especiais`. Liste o **titulo** e os **recursos_especiais** dos filmes ativos que contenham `'Trailers'`, usando `LIKE`, ordenados pelo título.

**Tabela:** `filmes`
**Conceitos:** `WHERE`, `LIKE` com `%`

---

## Bloco 3 — BETWEEN, IN e NOT IN (Exercícios 19 a 26)

---

### Exercício 19 — Filmes para a Seção "Preço Justo"

A locadora vai criar uma seção chamada "Preço Justo" no catálogo, com filmes cujo aluguel custe **entre R$ 2,99 e R$ 4,99** (inclusive). Exiba o **titulo** e a **taxa_aluguel** dos filmes ativos nesse intervalo, em ordem crescente de preço.

**Tabela:** `filmes`
**Conceitos:** `BETWEEN`

---

### Exercício 20 — Acervo de Filmes Lançados entre 2004 e 2006

O setor de curadoria quer revisar os filmes de uma época específica. Liste o **titulo** e o **ano_lancamento** dos filmes ativos lançados entre 2004 e 2006 (inclusive), ordenados pelo ano e depois pelo título.

**Tabela:** `filmes`
**Conceitos:** `BETWEEN` com `YEAR`

---

### Exercício 21 — Aluguéis Registrados no Primeiro Trimestre de 2005

O setor financeiro precisa auditar os aluguéis realizados entre **1 de janeiro e 31 de março de 2005**. Exiba o **id_aluguel**, a **data_aluguel** e o **cliente_id**, ordenados pela data de aluguel crescente.

**Tabela:** `alugueis`
**Conceitos:** `BETWEEN` com datas

---

### Exercício 22 — Filmes das Categorias de Ação, Comédia e Drama

O cliente quer navegar por três categorias favoritas ao mesmo tempo. Liste o **titulo** dos filmes ativos cujo `idioma_id` esteja entre os valores `1`, `2` ou `3` (representando idiomas de interesse), usando `IN`. Ordene pelo título.

**Tabela:** `filmes`
**Conceitos:** `IN`

---

### Exercício 23 — Pagamentos de Valores Específicos

O financeiro está investigando pagamentos de valores exatos (R$ 0,99, R$ 2,99 e R$ 9,99) que podem indicar promoções aplicadas. Exiba o **id_pagamento**, o **cliente_id**, o **valor** e a **data_pagamento** desses registros ativos, ordenados pela data.

**Tabela:** `pagamentos`
**Conceitos:** `IN` com decimais

---

### Exercício 24 — Filmes Excluindo Classificações Infantis

Uma plataforma parceira exibe apenas filmes para maiores de 13 anos. Liste o **titulo** e a **classificacao** dos filmes ativos que **não** sejam classificados como `'G'` nem `'PG'`, usando `NOT IN`, ordenados pela classificação e pelo título.

**Tabela:** `filmes`
**Conceitos:** `NOT IN`

---

### Exercício 25 — Inventário Excluindo Filmes de Certas IDs

A loja 1 está retirando temporariamente do acervo físico os filmes de ids 1, 2, 3 e 4 por revisão. Liste o **id_inventario** e o **filme_id** dos itens ativos do inventário da `loja_id = 1` que **não** pertençam a esses filmes, ordenados pelo filme_id.

**Tabela:** `inventarios`
**Conceitos:** `NOT IN`, `AND`

---

### Exercício 26 — Atores com Sobrenome em uma Lista de Destaque

O setor de marketing quer destacar atores de sobrenomes famosos para uma campanha. Liste o **nome_completo** (concatenando primeiro e último nome) dos atores ativos cujo `ultimo_nome` esteja na lista: `'KILMER'`, `'BERRY'`, `'PITT'`, `'JOHANSSON'`. Ordene pelo sobrenome.

**Tabela:** `atores`
**Conceitos:** `IN`, `CONCAT`, alias

---

## Bloco 4 — LIKE e Busca por Padrão (Exercícios 27 a 33)

---

### Exercício 27 — Busca de Filmes pelo Título (Campo de Pesquisa)

O sistema de busca da locadora recebeu o texto `"GOLD"` digitado pelo cliente. Implemente a consulta que retorna o **titulo** e a **taxa_aluguel** de todos os filmes ativos cujo título **contenha** a palavra `GOLD` em qualquer posição. Ordene pelo título.

**Tabela:** `filmes`
**Conceitos:** `LIKE` com `%` nos dois lados

---

### Exercício 28 — Clientes com Sobrenome Iniciando por "SM"

A atendente está procurando um cliente e só sabe que o sobrenome começa com "SM". Exiba o **primeiro_nome**, o **ultimo_nome** e o **email** de todos os clientes ativos cujo `ultimo_nome` comece com `'SM'`, em ordem alfabética pelo sobrenome.

**Tabela:** `clientes`
**Conceitos:** `LIKE` com `%` no final

---

### Exercício 29 — Filmes cujo Título Termina com "ABLE"

O curador de conteúdo quer fazer um lote temático com filmes terminados em "ABLE". Liste o **titulo** e a **classificacao** dos filmes ativos com título terminando em `'ABLE'`, ordenados pelo título.

**Tabela:** `filmes`
**Conceitos:** `LIKE` com `%` no início

---

### Exercício 30 — Busca por E-mail em Domínio Específico

O administrador precisa identificar todos os clientes que usam e-mail com domínio `'sakilacustomer.org'`. Exiba o **id_cliente**, o nome completo e o **email** dos clientes ativos com esse domínio, ordenados pelo id.

**Tabela:** `clientes`
**Conceitos:** `LIKE`, `CONCAT`

---

### Exercício 31 — Clientes com Telefone no Formato de Tamanho Específico

O sistema de SMS precisa filtrar telefones com exatamente 12 dígitos numéricos (sem formatação). Use `LIKE` com `_` (underscore) para selecionar o **id_endereco** e o **telefone** de endereços ativos cujo telefone tenha exatamente 12 caracteres. Ordene pelo telefone.

**Tabela:** `enderecos`
**Conceitos:** `LIKE` com `_` repetido (padrão de tamanho fixo)

---

### Exercício 32 — Filmes com Descrição Mencionando "Drama"

A equipe de conteúdo quer verificar filmes cuja descrição mencione a palavra `'Drama'`. Exiba o **titulo** e a **descricao** dos filmes ativos que contenham essa palavra na descrição, ordenados pelo título.

**Tabela:** `filmes`
**Conceitos:** `LIKE` em campo TEXT

---

### Exercício 33 — Atores com Primeiro Nome de Exatamente 4 Letras

Uma pesquisadora está estudando nomes curtos no banco de dados. Liste o **primeiro_nome** e o **ultimo_nome** dos atores ativos cujo `primeiro_nome` tenha **exatamente 4 caracteres**. Ordene pelo primeiro nome.

**Tabela:** `atores`
**Conceitos:** `LIKE` com 4 underscores (`____`)

---

## Bloco 5 — ORDER BY, LIMIT e OFFSET (Exercícios 34 a 40)

---

### Exercício 34 — Top 5 Filmes Mais Caros para Alugar

O gerente quer exibir no topo do site os 5 filmes com maior taxa de aluguel. Liste o **titulo** e a **taxa_aluguel** dos filmes ativos, ordenados pela taxa de forma decrescente, limitando ao top 5.

**Tabela:** `filmes`
**Conceitos:** `ORDER BY DESC`, `LIMIT`

---

### Exercício 35 — Os 10 Clientes Mais Antigos

O time de fidelidade quer homenagear os 10 clientes mais antigos do sistema (baseado em `criado_em`). Exiba o **id_cliente**, o nome completo (`nome_completo`) e a data de cadastro (`criado_em`) desses clientes ativos, do mais antigo para o mais recente.

**Tabela:** `clientes`
**Conceitos:** `ORDER BY ASC`, `LIMIT`, `CONCAT`

---

### Exercício 36 — Paginação do Catálogo (Página 3)

O app mobile exibe 10 filmes por página. Retorne a **3a página** do catálogo: exiba o **titulo** e a **taxa_aluguel** dos filmes ativos, ordenados pelo título crescente, com `LIMIT` e `OFFSET` adequados.

**Tabela:** `filmes`
**Conceitos:** `LIMIT`, `OFFSET` (offset = 20 para página 3)

---

### Exercício 37 — Filme Mais Barato Disponível

Um cliente quer saber qual é o filme mais barato para alugar hoje. Retorne apenas 1 registro com o **titulo** e a **taxa_aluguel** do filme ativo de menor preço.

**Tabela:** `filmes`
**Conceitos:** `ORDER BY ASC`, `LIMIT 1`

---

### Exercício 38 — Últimos 5 Aluguéis Registrados

O gerente quer ver os 5 aluguéis mais recentes para conferir a movimentação do dia. Exiba o **id_aluguel**, a **data_aluguel** e o **cliente_id** dos registros ativos, do mais recente para o mais antigo, limitando a 5.

**Tabela:** `alugueis`
**Conceitos:** `ORDER BY DESC`, `LIMIT`

---

### Exercício 39 — Segundo Filme Mais Longo do Acervo

Um cliente quer saber qual é o segundo filme mais longo, porque o primeiro ele já assistiu. Use `LIMIT` e `OFFSET` para retornar o **titulo** e a **duracao** do segundo filme mais longo ativo.

**Tabela:** `filmes`
**Conceitos:** `ORDER BY DESC`, `LIMIT 1 OFFSET 1`

---

### Exercício 40 — Filmes PG Mais Baratos (Top 3)

Um pai quer alugar até 3 filmes classificados `'PG'` pelos preços mais acessíveis para o fim de semana. Liste o **titulo**, a **classificacao** e a **taxa_aluguel** dos 3 filmes ativos mais baratos com classificação `'PG'`.

**Tabela:** `filmes`
**Conceitos:** `WHERE`, `ORDER BY`, `LIMIT`

---

## Bloco 6 — Funções de String e Data (Exercícios 41 a 50)

---

### Exercício 41 — Catálogo com Títulos em Minúsculas para Exportação

O sistema legado parceiro só aceita títulos em letras minúsculas. Exiba o **id_filme** e o titulo convertido para minúsculas (coluna `titulo_lower`) de todos os filmes ativos, ordenados pelo id.

**Tabela:** `filmes`
**Conceitos:** `LOWER()`

---

### Exercício 42 — Nomes de Atores em Maiúsculas para Certificados

A locadora vai emitir certificados de "Ator do Mês" e precisa dos nomes em maiúsculas. Exiba o `primeiro_nome` e o `ultimo_nome` dos atores ativos, ambos convertidos para maiúsculas (colunas `nome` e `sobrenome`), ordenados pelo sobrenome.

**Tabela:** `atores`
**Conceitos:** `UPPER()`

---

### Exercício 43 — Comprimento dos Títulos para Validação de Layout

A equipe de design quer identificar títulos muito longos que podem quebrar o layout do site. Exiba o **titulo** e o número de caracteres do título (coluna `tamanho_titulo`), para filmes ativos, ordenados pelo tamanho de forma decrescente. Mostre apenas os 10 maiores.

**Tabela:** `filmes`
**Conceitos:** `LENGTH()`, `ORDER BY DESC`, `LIMIT`

---

### Exercício 44 — Etiqueta de Identificação do Inventário

A equipe do depósito precisa imprimir etiquetas no formato: `"[ID_INV] - Filme: [ID_FILME] | Loja: [ID_LOJA]"`. Gere essa coluna chamada `etiqueta` para todos os itens ativos do inventário, ordenados pelo id_inventario.

**Tabela:** `inventarios`
**Conceitos:** `CONCAT()`, alias, conversão implícita de inteiros em string

---

### Exercício 45 — Primeiros 20 Caracteres da Descrição (Prévia)

O app mobile exibe apenas uma prévia da descrição do filme. Exiba o **titulo** e os primeiros **20 caracteres** da `descricao` (coluna `previa`) dos filmes ativos que possuem descrição, ordenados pelo título.

**Tabela:** `filmes`
**Conceitos:** `SUBSTRING()`, `IS NOT NULL`

---

### Exercício 46 — Ano e Mês de Cada Aluguel

O setor de BI precisa analisar o volume de aluguéis por período. Exiba o **id_aluguel**, o **ano** e o **mês** da `data_aluguel` (colunas `ano_aluguel` e `mes_aluguel`) dos aluguéis ativos realizados em 2005, ordenados pela data crescente.

**Tabela:** `alugueis`
**Conceitos:** `YEAR()`, `MONTH()`, `WHERE` com `YEAR()`

---

### Exercício 47 — Data de Pagamento Formatada para Relatório

O contador pediu um relatório com a data de pagamento no formato brasileiro `DD/MM/AAAA HH:MM`. Exiba o **id_pagamento**, o **valor** e a `data_pagamento` formatada (coluna `data_formatada`) dos pagamentos ativos de 2005, ordenados pela data.

**Tabela:** `pagamentos`
**Conceitos:** `DATE_FORMAT()`, `YEAR()`

---

### Exercício 48 — Tempo desde o Último Cadastro de Clientes

O gerente quer saber há quantos **anos** cada cliente está cadastrado no sistema. Exiba o **id_cliente**, o nome completo (`nome_completo`) e a quantidade de anos desde o `criado_em` (coluna `anos_cadastro`), para clientes ativos, ordenados pelo maior tempo de cadastro.

**Tabela:** `clientes`
**Conceitos:** `TIMESTAMPDIFF()`, `NOW()`, `CONCAT()`

---

### Exercício 49 — Filmes com Título Contendo Exatamente 5 Caracteres Iniciais "DESTI"

O sistema de busca avançada precisa filtrar filmes cujo título comece com `'DESTI'` e ao mesmo tempo exibir a **versão em maiúsculas** do título (coluna `titulo_upper`) e o **tamanho** do título (coluna `tamanho`). Filmes ativos, ordenados pelo tamanho decrescente.

**Tabela:** `filmes`
**Conceitos:** `LIKE`, `UPPER()`, `LENGTH()`

---

### Exercício 50 — Relatório Completo de Aluguéis com Atraso Potencial

A locadora cobra multa por atraso. Considere que o prazo padrão de devolução é de `duracao_aluguel` dias (disponível na tabela `filmes`, mas para este exercício use o valor fixo de **3 dias**). Selecione da tabela `alugueis` os registros ativos onde a `data_devolucao` **não é nula** (o filme foi devolvido) e a devolução ocorreu **após 2005-08-01**. Exiba o **id_aluguel**, a **data_aluguel** formatada como `DD/MM/AAAA` (coluna `data_aluguel_fmt`), a **data_devolucao** formatada da mesma forma (coluna `data_devolucao_fmt`), e o **numero de dias** entre aluguel e devolucao (coluna `dias_uso`) usando `TIMESTAMPDIFF`. Ordene pelos dias de uso de forma decrescente e mostre apenas os 10 primeiros.

**Tabela:** `alugueis`
**Conceitos:** `DATE_FORMAT()`, `TIMESTAMPDIFF()`, `IS NOT NULL`, `WHERE` com data, `ORDER BY DESC`, `LIMIT`

---

## Gabarito Conceitual (Referência Rápida)

| Exercício | Tabela Principal | Conceito-Chave |
|-----------|-----------------|----------------|
| 1 | filmes | SELECT + ORDER BY |
| 2 | atores | CONCAT + alias |
| 3 | filmes | DISTINCT |
| 4 | clientes | ORDER BY múltiplos campos |
| 5 | funcionarios | CONCAT + alias |
| 6 | paises | SELECT básico |
| 7 | filmes | alias duplos |
| 8 | categorias | SELECT básico |
| 9 | filmes | WHERE > |
| 10 | filmes | WHERE <= |
| 11 | filmes | WHERE = (ENUM) |
| 12 | clientes | WHERE booleano + IS NULL |
| 13 | filmes | WHERE AND múltiplo |
| 14 | clientes | WHERE OR |
| 15 | filmes | IS NULL |
| 16 | funcionarios | IS NOT NULL |
| 17 | filmes | OR + AND com parênteses |
| 18 | filmes | LIKE % |
| 19 | filmes | BETWEEN decimal |
| 20 | filmes | BETWEEN YEAR |
| 21 | alugueis | BETWEEN datas |
| 22 | filmes | IN |
| 23 | pagamentos | IN decimal |
| 24 | filmes | NOT IN |
| 25 | inventarios | NOT IN + AND |
| 26 | atores | IN + CONCAT |
| 27 | filmes | LIKE %texto% |
| 28 | clientes | LIKE texto% |
| 29 | filmes | LIKE %texto |
| 30 | clientes | LIKE domínio |
| 31 | enderecos | LIKE com _ |
| 32 | filmes | LIKE em TEXT |
| 33 | atores | LIKE com ____ |
| 34 | filmes | ORDER BY DESC + LIMIT |
| 35 | clientes | ORDER BY ASC + LIMIT |
| 36 | filmes | LIMIT + OFFSET |
| 37 | filmes | ORDER BY + LIMIT 1 |
| 38 | alugueis | ORDER BY DESC + LIMIT |
| 39 | filmes | LIMIT 1 OFFSET 1 |
| 40 | filmes | WHERE + ORDER BY + LIMIT |
| 41 | filmes | LOWER() |
| 42 | atores | UPPER() |
| 43 | filmes | LENGTH() + LIMIT |
| 44 | inventarios | CONCAT() com inteiros |
| 45 | filmes | SUBSTRING() + IS NOT NULL |
| 46 | alugueis | YEAR() + MONTH() |
| 47 | pagamentos | DATE_FORMAT() |
| 48 | clientes | TIMESTAMPDIFF() + NOW() |
| 49 | filmes | LIKE + UPPER() + LENGTH() |
| 50 | alugueis | DATE_FORMAT() + TIMESTAMPDIFF() + LIMIT |

---

> **Lembrete:** Todos os exercícios devem incluir `WHERE tabela.deletado_em IS NULL` (ou equivalente) para respeitar o padrão de soft-delete do banco `sakila_pt`.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
