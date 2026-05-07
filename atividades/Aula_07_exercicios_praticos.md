# Aula 07 — Exercícios Práticos: SQL DQL — Agregação

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> Banco de dados: `sakila_pt` · Base teórica: Aula 07 — SQL DQL: Consultas e Agregação

---

## Instruções Gerais

- Utilize **exclusivamente** o banco de dados `sakila_pt`.
- Aplique sempre o filtro de soft-delete: `deletado_em IS NULL`.
- Use `AS` para nomear colunas calculadas com rótulos claros.
- Lembre-se da **regra fundamental do GROUP BY**: toda coluna no `SELECT` que não for função de agregação deve estar no `GROUP BY`.
- Diferencie `WHERE` (filtra linhas antes do agrupamento) de `HAVING` (filtra grupos após a agregação).
- Os exercícios estão organizados em blocos de complexidade crescente.

---

## Bloco 1 — COUNT: Contando Registros (Exercícios 1 a 8)

---

### Exercício 1 — Total de Filmes no Acervo

O gerente da locadora quer saber quantos filmes estão cadastrados no sistema para planejar a expansão do acervo. Exiba um único número com o total de filmes ativos, com o rótulo `total_filmes`.

**Tabela:** `filmes`
**Conceitos:** `COUNT(*)`

---

### Exercício 2 — Filmes com Descrição Cadastrada

A equipe de conteúdo quer saber quantos filmes já possuem descrição preenchida (campo `descricao` não nulo), pois os demais precisam ser revisados. Exiba o total com o rótulo `filmes_com_descricao`.

**Tabela:** `filmes`
**Conceitos:** `COUNT(coluna)` — conta apenas valores não NULL

---

### Exercício 3 — Total de Clientes por Loja

O setor comercial precisa saber quantos clientes ativos cada loja possui para planejar a capacidade de atendimento. Exiba o `loja_id` e o total de clientes ativos (`total_clientes`), ordenado do maior para o menor.

**Tabela:** `clientes`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `ORDER BY`

---

### Exercício 4 — Quantidade de Filmes por Classificação Indicativa

O responsável pelo catálogo quer entender a distribuição do acervo por faixa etária. Liste cada `classificacao` e a quantidade de filmes ativos nela (`total_filmes`), ordenando da maior para a menor quantidade.

**Tabela:** `filmes`
**Conceitos:** `COUNT(*)`, `GROUP BY` em ENUM

---

### Exercício 5 — Filmes por Idioma

A locadora está avaliando a variedade de idiomas do catálogo. Para cada `idioma_id`, conte quantos filmes ativos existem (`total_filmes`). Ordene pelo total de forma decrescente.

**Tabela:** `filmes`
**Conceitos:** `COUNT(*)`, `GROUP BY`

---

### Exercício 6 — Quantidade de Atores por Filme

O curador de conteúdo quer saber quantos atores cada filme possui no elenco, para identificar produções com elenco extenso. Para cada `filme_id`, conte o número de atores (`total_atores`) na tabela de relacionamento, excluindo registros deletados. Ordene pelo total de atores de forma decrescente e limite a 10.

**Tabela:** `filmes_atores`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `ORDER BY`, `LIMIT`

---

### Exercício 7 — Quantidade de Itens de Inventário por Loja

O logístico quer saber quantos itens físicos cada loja possui em estoque. Agrupe o inventário ativo por `loja_id` e conte os itens (`total_itens`). Ordene por loja crescente.

**Tabela:** `inventarios`
**Conceitos:** `COUNT(*)`, `GROUP BY`

---

### Exercício 8 — Aluguéis por Funcionário

O gerente quer avaliar a produtividade individual. Para cada `funcionario_id`, conte quantos aluguéis ativos foram registrados (`total_alugueis`). Ordene do mais produtivo para o menos produtivo.

**Tabela:** `alugueis`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `ORDER BY DESC`

---

## Bloco 2 — SUM e AVG: Somando e Calculando Médias (Exercícios 9 a 18)

---

### Exercício 9 — Faturamento Total da Locadora

O financeiro precisa do faturamento bruto total de todos os pagamentos ativos para o relatório mensal. Exiba a soma de todos os valores recebidos com o rótulo `faturamento_total`.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`

---

### Exercício 10 — Faturamento por Funcionário

O gestor de RH quer avaliar o volume financeiro movimentado por cada funcionário. Para cada `funcionario_id`, some os valores dos pagamentos ativos (`faturamento`) e ordene do maior para o menor.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `GROUP BY`, `ORDER BY`

---

### Exercício 11 — Valor Médio dos Pagamentos

O financeiro quer conhecer o ticket médio da locadora, ou seja, o valor médio pago por transação. Exiba a média de todos os pagamentos ativos com o rótulo `ticket_medio`. Arredonde para 2 casas decimais usando `ROUND()`.

**Tabela:** `pagamentos`
**Conceitos:** `AVG()`, `ROUND()`

---

### Exercício 12 — Receita Total por Ano

O setor de planejamento quer analisar a evolução do faturamento ano a ano. Para cada ano encontrado em `data_pagamento`, some o total de pagamentos ativos (`faturamento_total`) e ordene pelo ano crescente.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `YEAR()`, `GROUP BY`, `ORDER BY`

---

### Exercício 13 — Receita Total por Mês (Ano de 2005)

O analista financeiro precisa de um relatório mensal de receita apenas para o ano de 2005. Para cada mês, exiba o número do mês (`mes`) e a soma dos pagamentos ativos (`receita_mensal`). Ordene pelo mês crescente.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `YEAR()`, `MONTH()`, `GROUP BY`, `WHERE`

---

### Exercício 14 — Média de Duração dos Filmes por Classificação

A equipe de programação quer conhecer a duração média dos filmes de cada classificação para organizar grades de exibição. Agrupe os filmes ativos por `classificacao` e exiba a duração média (`duracao_media`) arredondada para 1 casa decimal. Ordene pela duração média decrescente.

**Tabela:** `filmes`
**Conceitos:** `AVG()`, `ROUND()`, `GROUP BY`

---

### Exercício 15 — Total Gasto por Cliente

O programa de fidelidade precisa identificar quanto cada cliente gastou no total. Para cada `cliente_id`, some os valores dos pagamentos ativos (`total_gasto`). Ordene do maior gasto para o menor e limite a 20 resultados.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `GROUP BY`, `ORDER BY DESC`, `LIMIT`

---

### Exercício 16 — Custo Médio de Reposição por Classificação

O setor de compras precisa prever investimentos para reposição de estoque por faixa etária. Para cada `classificacao` de filmes ativos, calcule o custo médio de reposição (`custo_medio_reposicao`) com 2 casas decimais. Ordene pelo custo médio decrescente.

**Tabela:** `filmes`
**Conceitos:** `AVG()`, `ROUND()`, `GROUP BY`

---

### Exercício 17 — Soma e Média de Pagamentos por Dia da Semana

O analista de operações quer saber em quais dias da semana a locadora recebe mais. Use `DAYOFWEEK()` (1=domingo, 7=sábado) sobre `data_pagamento` para agrupar os pagamentos ativos. Exiba o dia (`dia_semana`), o total de transações (`total_transacoes`), a soma dos valores (`receita`) e a média por transação (`media_por_transacao`). Ordene pelo dia da semana crescente.

**Tabela:** `pagamentos`
**Conceitos:** `COUNT(*)`, `SUM()`, `AVG()`, `DAYOFWEEK()`, `GROUP BY`

---

### Exercício 18 — Taxa de Aluguel Média por Duração do Período de Aluguel

O gerente suspeita que filmes com período de aluguel mais longo (campo `duracao_aluguel`) têm taxas diferentes. Para cada valor de `duracao_aluguel` nos filmes ativos, calcule a taxa média de aluguel (`taxa_media`) com 2 casas decimais e o total de filmes nessa faixa (`total_filmes`). Ordene pelo período de aluguel crescente.

**Tabela:** `filmes`
**Conceitos:** `AVG()`, `COUNT()`, `ROUND()`, `GROUP BY`

---

## Bloco 3 — MIN e MAX: Mínimos e Máximos (Exercícios 19 a 25)

---

### Exercício 19 — Filme Mais Barato e Mais Caro para Alugar

O atendente precisa informar ao cliente os extremos de preço do catálogo em uma única consulta. Exiba o menor valor de `taxa_aluguel` (`aluguel_minimo`) e o maior valor (`aluguel_maximo`) entre os filmes ativos.

**Tabela:** `filmes`
**Conceitos:** `MIN()`, `MAX()`

---

### Exercício 20 — Menor e Maior Custo de Reposição por Classificação

O setor de compras quer conhecer os extremos de custo de reposição para cada faixa etária de filmes ativos. Exiba a `classificacao`, o menor custo (`menor_custo`) e o maior custo (`maior_custo`). Ordene pela classificação.

**Tabela:** `filmes`
**Conceitos:** `MIN()`, `MAX()`, `GROUP BY`

---

### Exercício 21 — Primeiro e Último Aluguel Registrado por Cliente

O suporte ao cliente precisa saber quando cada cliente realizou seu primeiro e seu último aluguel para calcular o tempo de relacionamento. Para cada `cliente_id`, exiba a data do primeiro aluguel ativo (`primeiro_aluguel`) e a data do último (`ultimo_aluguel`). Ordene pelo cliente_id crescente.

**Tabela:** `alugueis`
**Conceitos:** `MIN()`, `MAX()`, `GROUP BY`

---

### Exercício 22 — Maior Pagamento Único Registrado

O auditor quer identificar o maior valor pago em uma única transação ativa para verificar possíveis erros de lançamento. Exiba o valor com o rótulo `maior_pagamento`.

**Tabela:** `pagamentos`
**Conceitos:** `MAX()`

---

### Exercício 23 — Menor e Maior Duração de Filmes por Idioma

A curadoria quer entender a amplitude de duração dos filmes em cada idioma. Para cada `idioma_id` com filmes ativos que possuam `duracao` preenchida, exiba o menor (`duracao_minima`) e o maior (`duracao_maxima`) valor em minutos. Ordene pelo idioma.

**Tabela:** `filmes`
**Conceitos:** `MIN()`, `MAX()`, `GROUP BY`, `WHERE` com `IS NOT NULL`

---

### Exercício 24 — Primeiro e Último Pagamento por Funcionário

O RH quer saber o período de atividade financeira de cada funcionário. Para cada `funcionario_id`, exiba a data do primeiro pagamento ativo processado (`primeiro_pagamento`) e a data do último (`ultimo_pagamento`). Ordene pelo funcionário.

**Tabela:** `pagamentos`
**Conceitos:** `MIN()`, `MAX()`, `GROUP BY`

---

### Exercício 25 — Resumo Geral de Preços do Catálogo por Classificação

O gerente quer um painel completo com mínimo, máximo e amplitude (diferença entre maior e menor taxa) de aluguel por classificação de filmes ativos. Exiba a `classificacao`, o `aluguel_minimo`, o `aluguel_maximo` e a `amplitude` (MAX menos MIN). Ordene pela amplitude decrescente.

**Tabela:** `filmes`
**Conceitos:** `MIN()`, `MAX()`, expressão calculada no `SELECT`, `GROUP BY`

---

## Bloco 4 — GROUP BY Composto e Funções de Data (Exercícios 26 a 35)

---

### Exercício 26 — Aluguéis por Mês e Ano

O setor de operações quer um histórico de volume de aluguéis por período. Para cada combinação de ano e mês de `data_aluguel`, conte o total de aluguéis ativos (`total_alugueis`). Exiba `ano`, `mes` e o total. Ordene pelo ano e mês crescentes.

**Tabela:** `alugueis`
**Conceitos:** `COUNT(*)`, `YEAR()`, `MONTH()`, `GROUP BY` composto

---

### Exercício 27 — Faturamento por Loja e por Mês

O financeiro precisa de um breakdown de receita por loja e período. Para cada combinação de `funcionario_id` (que representa a loja operacionalmente) e mês de `data_pagamento` em 2005, some os valores ativos (`receita`). Exiba `funcionario_id`, `mes` e `receita`. Ordene pelo funcionário e depois pelo mês.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `YEAR()`, `MONTH()`, `GROUP BY` composto, `WHERE`

---

### Exercício 28 — Quantidade de Aluguéis por Dia da Semana e por Funcionário

O gerente quer entender o padrão de distribuição de trabalho. Para cada combinação de `funcionario_id` e dia da semana (`DAYOFWEEK`), conte os aluguéis ativos (`total_alugueis`). Exiba `funcionario_id`, `dia_semana` e o total. Ordene pelo funcionário e pelo dia.

**Tabela:** `alugueis`
**Conceitos:** `COUNT(*)`, `DAYOFWEEK()`, `GROUP BY` composto

---

### Exercício 29 — Total de Filmes por Categoria e por Classificação

O curador quer cruzar categoria com classificação indicativa para identificar padrões editoriais. Para cada combinação de `categoria_id` e `classificacao` na tabela de relacionamento (filmes ativos e registros ativos de filmes_categorias), conte os filmes (`total_filmes`). Ordene pela categoria e depois pela classificação.

**Tabelas:** `filmes_categorias`, `filmes`
**Conceitos:** `COUNT(*)`, `GROUP BY` composto com duas tabelas e `INNER JOIN` simples (use apenas `FROM tabela1, tabela2 WHERE chave1 = chave2` se ainda não estudou JOIN formalmente, ou INNER JOIN se já estudou)

> **Nota:** Este exercício requer cruzamento de tabelas. Use a sintaxe `FROM filmes_categorias fc, filmes f WHERE fc.filme_id = f.id_filme` ou `INNER JOIN` conforme orientação do professor.

---

### Exercício 30 — Aluguéis por Hora do Dia

O analista de operações quer saber em quais horários do dia a locadora é mais movimentada. Use `HOUR(data_aluguel)` para extrair a hora. Para cada hora (0-23), conte os aluguéis ativos (`total_alugueis`). Ordene pela hora crescente.

**Tabela:** `alugueis`
**Conceitos:** `COUNT(*)`, `HOUR()`, `GROUP BY`

---

### Exercício 31 — Receita por Trimestre

O conselho da locadora analisa resultados trimestralmente. Use `QUARTER(data_pagamento)` para agrupar os pagamentos ativos de 2005 por trimestre. Exiba o `trimestre` e a `receita_total`. Ordene pelo trimestre.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `QUARTER()`, `YEAR()`, `GROUP BY`, `WHERE`

---

### Exercício 32 — Contagem de Clientes Cadastrados por Mês e Ano

O marketing quer analisar o crescimento da base de clientes ao longo do tempo. Para cada combinação de ano e mês de `criado_em`, conte os clientes ativos cadastrados (`novos_clientes`). Exiba `ano`, `mes` e o total. Ordene por ano e mês crescentes.

**Tabela:** `clientes`
**Conceitos:** `COUNT(*)`, `YEAR()`, `MONTH()`, `GROUP BY` composto

---

### Exercício 33 — Duração Total de Filmes por Idioma e Classificação

A programação quer estimar o tempo de exibição disponível por idioma e faixa etária. Para cada combinação de `idioma_id` e `classificacao` em filmes ativos com `duracao` preenchida, some a duração total (`duracao_total_minutos`) e conte os filmes (`total_filmes`). Ordene pelo idioma e depois pela classificação.

**Tabela:** `filmes`
**Conceitos:** `SUM()`, `COUNT()`, `GROUP BY` composto, `WHERE` com `IS NOT NULL`

---

### Exercício 34 — Aluguéis Devolvidos e Não Devolvidos por Funcionário

O operacional quer monitorar quantos itens cada funcionário registrou com devolução efetuada (`data_devolucao IS NOT NULL`) e quantos sem devolução (`data_devolucao IS NULL`). Use `SUM()` com expressões condicionais: `SUM(data_devolucao IS NOT NULL)` e `SUM(data_devolucao IS NULL)`. Exiba o `funcionario_id`, `devolvidos` e `pendentes`. Ordene pelo funcionário.

**Tabela:** `alugueis`
**Conceitos:** `SUM()` com expressão booleana, `GROUP BY`

> **Dica:** No MariaDB, `(data_devolucao IS NOT NULL)` retorna 1 ou 0. Ao somar esses valores com `SUM()`, obtém-se a contagem de registros que atendem à condição, sem precisar de `CASE WHEN`.

---

### Exercício 35 — Média de Pagamentos por Cliente por Mês em 2005

O financeiro quer entender o padrão de gasto mensal de cada cliente em 2005. Para cada combinação de `cliente_id` e mês (`MONTH(data_pagamento)`) em pagamentos ativos de 2005, calcule a média dos valores (`media_mensal`) com 2 casas decimais. Ordene pelo cliente e depois pelo mês.

**Tabela:** `pagamentos`
**Conceitos:** `AVG()`, `ROUND()`, `YEAR()`, `MONTH()`, `GROUP BY` composto, `WHERE`

---

## Bloco 5 — HAVING: Filtrando Grupos (Exercícios 36 a 45)

---

### Exercício 36 — Classificações com Mais de 200 Filmes

O diretor de conteúdo quer identificar quais classificações indicativas têm acervo expressivo. Liste apenas as `classificacoes` com mais de 200 filmes ativos (`total_filmes`), exibindo o total. Ordene pelo total decrescente.

**Tabela:** `filmes`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `HAVING`

---

### Exercício 37 — Clientes com Mais de 30 Aluguéis

O programa de fidelidade VIP é destinado a clientes com histórico intenso de locação. Liste apenas os `cliente_id` com mais de 30 aluguéis ativos (`total_alugueis`). Ordene pelo total decrescente.

**Tabela:** `alugueis`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `HAVING`

---

### Exercício 38 — Funcionários com Faturamento Acima de R$ 5.000

O financeiro quer identificar os funcionários responsáveis pelo maior volume de receita. Liste apenas os `funcionario_id` cujos pagamentos ativos somem mais de R$ 5.000 (`faturamento_total`). Ordene pelo faturamento decrescente.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `GROUP BY`, `HAVING`

---

### Exercício 39 — Idiomas com Menos de 5 Filmes

O setor de aquisição quer identificar idiomas sub-representados no catálogo. Liste apenas os `idioma_id` com menos de 5 filmes ativos (`total_filmes`).

**Tabela:** `filmes`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `HAVING`

---

### Exercício 40 — Lojas com Mais de 2 Funcionários Ativos

O RH quer identificar as lojas com equipes maiores. Liste apenas o `loja_id` das lojas com mais de 2 funcionários ativos (`total_funcionarios`).

**Tabela:** `funcionarios`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `HAVING`

---

### Exercício 41 — Filmes com Elenco de Mais de 10 Atores

A curadoria quer destacar produções com elenco extenso no catálogo premium. Liste os `filme_id` com mais de 10 atores (`total_atores`) na tabela de relacionamento ativa. Ordene pelo total decrescente.

**Tabela:** `filmes_atores`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `HAVING`, `ORDER BY`

---

### Exercício 42 — Clientes com Gasto Total Acima de R$ 150

O programa de benefícios quer recompensar clientes de alto valor. Liste os `cliente_id` cujo total de pagamentos ativos supere R$ 150 (`total_gasto`). Ordene pelo total gasto decrescente e limite a 10.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`

---

### Exercício 43 — Meses com Receita Acima da Média Geral (2005)

O planejamento quer identificar os meses de pico de receita em 2005. Liste apenas os meses (`mes`) cujo faturamento mensal ativo (`receita_mensal`) supere R$ 5.000. Exiba o mês e a receita. Ordene pelo mês.

**Tabela:** `pagamentos`
**Conceitos:** `SUM()`, `YEAR()`, `MONTH()`, `GROUP BY`, `HAVING`, `WHERE`

---

### Exercício 44 — Dias da Semana com Mais de 1.000 Aluguéis

O operacional quer reforçar a equipe nos dias de maior movimento. Liste apenas os dias da semana (`dia_semana`, usando `DAYOFWEEK`) com mais de 1.000 aluguéis ativos no total (`total_alugueis`). Ordene pelo total decrescente.

**Tabela:** `alugueis`
**Conceitos:** `COUNT(*)`, `DAYOFWEEK()`, `GROUP BY`, `HAVING`

---

### Exercício 45 — Classificações com Custo Médio de Reposição Acima de R$ 20

O setor de compras quer focar o orçamento nas classificações mais custosas de repor. Liste apenas as `classificacoes` de filmes ativos com custo médio de reposição (`custo_medio`) acima de R$ 20,00, exibindo o valor com 2 casas decimais. Ordene pelo custo médio decrescente.

**Tabela:** `filmes`
**Conceitos:** `AVG()`, `ROUND()`, `GROUP BY`, `HAVING`

---

## Bloco 6 — WHERE + GROUP BY + HAVING + ORDER BY Combinados (Exercícios 46 a 50)

---

### Exercício 46 — Resumo Financeiro Mensal de 2005 com Filtro de Pico

O diretor financeiro quer um relatório dos meses de 2005 que tiveram tanto alto volume de transações quanto alta receita. Filtre os pagamentos ativos do ano de 2005 (`WHERE`), agrupe por mês, exiba `mes`, total de transações (`total_transacoes`), receita total (`receita_total`) e ticket médio (`ticket_medio`) com 2 casas decimais. Aplique `HAVING` para manter apenas meses com mais de 3.000 transações **e** receita acima de R$ 10.000. Ordene pela receita decrescente.

**Tabela:** `pagamentos`
**Conceitos:** `COUNT(*)`, `SUM()`, `AVG()`, `ROUND()`, `YEAR()`, `MONTH()`, `GROUP BY`, `WHERE`, `HAVING` com múltiplas condições

---

### Exercício 47 — Perfil de Locação dos Clientes Ativos com Alto Volume

O CRM precisa de um perfil quantitativo dos clientes com perfil intensivo de locação em 2005. Filtre os aluguéis ativos de 2005 (`WHERE`), agrupe por `cliente_id`, exiba o `cliente_id`, total de aluguéis (`total_alugueis`), data do primeiro aluguel no período (`primeiro_aluguel`) e data do último (`ultimo_aluguel`). Aplique `HAVING` para manter apenas clientes com mais de 25 aluguéis no período. Ordene pelo total decrescente.

**Tabela:** `alugueis`
**Conceitos:** `COUNT(*)`, `MIN()`, `MAX()`, `YEAR()`, `GROUP BY`, `WHERE`, `HAVING`, `ORDER BY`

---

### Exercício 48 — Análise de Elenco: Filmes com Elenco entre 5 e 8 Atores

A vitrine editorial vai destacar filmes com elenco de porte médio (nem muito pequeno, nem muito extenso). Considere apenas registros ativos da tabela de relacionamento, agrupe por `filme_id`, exiba o `filme_id` e o total de atores (`total_atores`). Aplique `HAVING` com `BETWEEN` para manter apenas filmes com elenco entre 5 e 8 atores (inclusive). Ordene pelo total de atores decrescente.

**Tabela:** `filmes_atores`
**Conceitos:** `COUNT(*)`, `GROUP BY`, `HAVING` com `BETWEEN`

---

### Exercício 49 — Inventário Concentrado: Filmes com Mais de 7 Cópias em Estoque

O logístico suspeita que alguns filmes estão com estoque excessivo, gerando custo de armazenagem desnecessário. Agrupe o inventário ativo por `filme_id` e `loja_id`, exiba ambos os campos e o total de cópias (`total_copias`). Aplique `HAVING` para manter apenas combinações com mais de 7 cópias. Ordene pelo total de cópias decrescente.

**Tabela:** `inventarios`
**Conceitos:** `COUNT(*)`, `GROUP BY` composto, `HAVING`, `ORDER BY`

---

### Exercício 50 — Painel Executivo: Top 5 Clientes por Receita em 2005 com Alta Frequência

O relatório executivo anual precisa dos 5 melhores clientes em receita no ano de 2005, mas apenas entre aqueles que realizaram mais de 20 pagamentos no período (clientes frequentes e de alto valor). Filtre pagamentos ativos de 2005 (`WHERE`), agrupe por `cliente_id`, exiba o `cliente_id`, total de pagamentos (`total_pagamentos`), receita total (`receita_total`) e ticket médio (`ticket_medio`) com 2 casas decimais. Aplique `HAVING` para manter apenas clientes com mais de 20 pagamentos no período. Ordene pela receita total decrescente e limite a 5.

**Tabela:** `pagamentos`
**Conceitos:** `COUNT(*)`, `SUM()`, `AVG()`, `ROUND()`, `YEAR()`, `GROUP BY`, `WHERE`, `HAVING`, `ORDER BY`, `LIMIT`

---

## Referência Rápida: Funções Utilizadas nesta Aula

| Função | O que faz | Exemplo de uso |
|--------|-----------|---------------|
| `COUNT(*)` | Conta todas as linhas do grupo, incluindo NULLs | `COUNT(*) AS total` |
| `COUNT(coluna)` | Conta apenas valores não NULL da coluna | `COUNT(descricao) AS com_descricao` |
| `SUM(coluna)` | Soma os valores numéricos do grupo | `SUM(valor) AS receita` |
| `AVG(coluna)` | Calcula a média dos valores do grupo (ignora NULL) | `AVG(taxa_aluguel) AS media` |
| `MIN(coluna)` | Retorna o menor valor do grupo | `MIN(data_aluguel) AS primeiro` |
| `MAX(coluna)` | Retorna o maior valor do grupo | `MAX(data_aluguel) AS ultimo` |
| `ROUND(valor, n)` | Arredonda para n casas decimais | `ROUND(AVG(valor), 2)` |
| `YEAR(data)` | Extrai o ano de um campo de data | `YEAR(data_pagamento)` |
| `MONTH(data)` | Extrai o mês (1-12) | `MONTH(data_aluguel)` |
| `HOUR(data)` | Extrai a hora (0-23) | `HOUR(data_aluguel)` |
| `DAYOFWEEK(data)` | Retorna o dia da semana (1=dom, 7=sáb) | `DAYOFWEEK(data_pagamento)` |
| `QUARTER(data)` | Retorna o trimestre (1-4) | `QUARTER(data_pagamento)` |

---

## Ordem de Execução do SELECT (Relembrete)

```
1. FROM        — define a(s) tabela(s) de origem
2. WHERE       — filtra linhas ANTES do agrupamento
3. GROUP BY    — agrupa as linhas restantes
4. HAVING      — filtra grupos APÓS a agregação
5. SELECT      — calcula as colunas de saída
6. DISTINCT    — remove duplicatas
7. ORDER BY    — ordena o resultado
8. LIMIT       — limita a quantidade retornada
```

> **Regra de ouro:** use `WHERE` para condições sobre colunas individuais (mais eficiente, usa índices). Use `HAVING` apenas para condições sobre valores agregados (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).

---

> **Lembrete:** Todos os exercícios devem incluir `WHERE tabela.deletado_em IS NULL` (ou equivalente) para respeitar o padrão de soft-delete do banco `sakila_pt`.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
