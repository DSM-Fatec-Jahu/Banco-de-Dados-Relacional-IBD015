# Aula 06 — SQL: Consultas Básicas

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 05](./Aula_05_Atividade_Modelagem_Ecommerce.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_07_SQL_DQL_Agregacao.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de escrever consultas SQL com `SELECT`, filtrar resultados com `WHERE`, utilizar operadores lógicos e relacionais, e ordenar resultados com `ORDER BY`.

---

## 🧭 Contexto

A **DQL** — *Data Query Language* — é o subconjunto do SQL responsável pela consulta de dados. O `SELECT` é o comando mais usado em qualquer banco de dados: enquanto um sistema típico executa muito mais leituras do que escritas, saber construir consultas eficientes é uma das habilidades mais valorizadas no mercado.

[prompt para nanobanana: "Educational diagram showing the anatomy of a SQL SELECT statement with labeled parts: SELECT (columns), FROM (table), WHERE (filter condition), ORDER BY (sorting). Each part highlighted in a different color (blue for SELECT, green for FROM, orange for WHERE, purple for ORDER BY). Clean flat design, white background, monospace font for SQL code, labels in Portuguese pointing to each clause."]
![Anatomia do SELECT](../imgs/Aula_06_img_01.png)

---

## 🎥 Vídeos de Apoio

- 📺 [SELECT, WHERE e ORDER BY — MySQL do zero](https://www.youtube.com/watch?v=7S_tz1z_5bA) — Bóson Treinamentos
- 📺 [Operadores SQL — LIKE, BETWEEN, IN, IS NULL](https://www.youtube.com/watch?v=g3xi4ZHQCM8) — CFBCursos

---

## 1. Estrutura Básica do SELECT

```sql
SELECT coluna1, coluna2, ...
FROM   tabela
WHERE  condicao
ORDER BY coluna [ASC | DESC];
```

### 1.1 Selecionando colunas específicas vs todas

```sql
-- Todas as colunas (evite em produção — traz dados desnecessários)
SELECT * FROM produtos;

-- Colunas específicas (recomendado)
SELECT id_produto, nome, preco, estoque
FROM   produtos;

-- Alias: renomeia a coluna no resultado
SELECT
    nome           AS produto,
    preco          AS "Preço (R$)",
    estoque        AS quantidade_disponivel
FROM produtos;
```

### 1.2 DISTINCT — eliminando duplicatas

```sql
-- Lista todas as cidades cadastradas (sem repetição)
SELECT DISTINCT cidade FROM enderecos ORDER BY cidade;

-- Combinação de colunas distintas
SELECT DISTINCT estado, cidade FROM enderecos ORDER BY estado, cidade;
```

---

## 2. WHERE — Filtrando Resultados

### 2.1 Operadores relacionais

```sql
-- Igualdade e diferença
SELECT * FROM produtos WHERE ativo = 1;
SELECT * FROM produtos WHERE categoria_id <> 2;  -- diferente

-- Comparações numéricas
SELECT nome, preco FROM produtos WHERE preco > 500.00;
SELECT nome, preco FROM produtos WHERE preco <= 199.90;

-- Combinando condições com AND e OR
SELECT nome, preco FROM produtos
WHERE  preco > 100 AND preco < 1000 AND ativo = 1;

SELECT nome FROM pessoas
WHERE  cidade = 'São Paulo' OR cidade = 'Campinas';
```

### 2.2 BETWEEN — intervalos inclusivos

```sql
-- Equivale a: preco >= 100 AND preco <= 500
SELECT nome, preco FROM produtos WHERE preco BETWEEN 100.00 AND 500.00;

-- Funciona com datas também
SELECT id_pedido, data_pedido FROM pedidos
WHERE  data_pedido BETWEEN '2026-01-01' AND '2026-03-31';
```

### 2.3 IN — lista de valores

```sql
-- Muito mais legível que múltiplos OR
SELECT nome, status FROM pedidos
WHERE  status IN ('confirmado', 'em_separacao', 'enviado');

-- NOT IN — exclui os valores da lista
SELECT nome FROM produtos WHERE categoria_id NOT IN (1, 3, 5);
```

### 2.4 LIKE — busca por padrão

```sql
-- % substitui qualquer sequência de caracteres (zero ou mais)
-- _ substitui exatamente um caractere

SELECT nome FROM pessoas WHERE nome LIKE 'Ana%';        -- começa com Ana
SELECT nome FROM pessoas WHERE nome LIKE '%Lima';       -- termina com Lima
SELECT nome FROM pessoas WHERE nome LIKE '%Silva%';     -- contém Silva em qualquer posição
SELECT cpf  FROM pessoas WHERE cpf  LIKE '111_____44';  -- _ = um caractere qualquer
```

### 2.5 IS NULL / IS NOT NULL

```sql
-- NULL nunca é igual a nada — use IS NULL, nunca = NULL
SELECT nome FROM pessoas WHERE telefone IS NULL;
SELECT nome FROM pessoas WHERE telefone IS NOT NULL;

-- ❌ NUNCA funciona:
SELECT nome FROM pessoas WHERE telefone = NULL;  -- sempre retorna vazio!
```

---

## 3. ORDER BY — Ordenando Resultados

```sql
-- Ordem crescente (padrão)
SELECT nome, preco FROM produtos ORDER BY preco;
SELECT nome, preco FROM produtos ORDER BY preco ASC;

-- Ordem decrescente
SELECT nome, preco FROM produtos ORDER BY preco DESC;

-- Múltiplos critérios de ordenação
SELECT nome, categoria_id, preco
FROM   produtos
ORDER BY categoria_id ASC, preco DESC;

-- Ordenando por alias ou posição da coluna (posição não recomendada em código mantido)
SELECT nome AS produto, preco AS valor
FROM   produtos
ORDER BY valor DESC;
```

---

## 4. LIMIT e OFFSET — Paginação

```sql
-- Os 5 produtos mais caros
SELECT nome, preco FROM produtos ORDER BY preco DESC LIMIT 5;

-- Paginação: página 2 com 10 itens por página
-- OFFSET = (pagina - 1) * itens_por_pagina = (2-1)*10 = 10
SELECT nome, preco FROM produtos ORDER BY id_produto LIMIT 10 OFFSET 10;

-- Sintaxe alternativa: LIMIT offset, count
SELECT nome, preco FROM produtos ORDER BY id_produto LIMIT 10, 10;
```

---

## 5. Funções de String e Data Úteis

```sql
-- Funções de string
SELECT UPPER(nome), LOWER(email) FROM pessoas;
SELECT LENGTH(nome) AS tamanho, nome FROM pessoas ORDER BY tamanho DESC;
SELECT CONCAT(nome, ' — CPF: ', cpf) AS identificacao FROM pessoas;
SELECT SUBSTRING(cpf, 1, 3) AS primeiros_digitos FROM pessoas;  -- 3 primeiros dígitos

-- Funções de data
SELECT nome, data_nascimento,
       YEAR(data_nascimento)                      AS ano_nasc,
       TIMESTAMPDIFF(YEAR, data_nascimento, NOW()) AS idade
FROM   pessoas
ORDER BY idade DESC;

SELECT id_pedido, data_pedido,
       DATE_FORMAT(data_pedido, '%d/%m/%Y %H:%i') AS data_formatada
FROM   pedidos;
```

---

## 6. Exercícios de Fixação

**Exercício 1:** liste o nome e preço de todos os produtos com preço entre R$ 100 e R$ 500, em ordem decrescente de preço.

**Exercício 2:** encontre todos os clientes cujo nome começa com a letra 'A' ou termina com 'Lima'.

**Exercício 3:** liste os 3 produtos mais baratos que estejam ativos (ativo=1) e com estoque maior que zero.

**Exercício 4:** liste todos os pedidos com status diferente de 'cancelado', mostrando id_pedido, data formatada (DD/MM/AAAA) e valor_total, ordenados por valor_total decrescente.

**Exercício 5:** quais produtos não possuem descrição cadastrada (campo NULL)?

---

## 📚 Referências desta Aula

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 6.3 — SELECT. São Paulo: Pearson, 2018.
- FORTA, B. *SQL em 10 Minutos por Dia*. 5 ed. Lições 2–7. São Paulo: Novatec, 2021.

---

> **Próxima aula:** na [Aula 07](./Aula_07_SQL_DQL_Agregacao.md) veremos funções de agregação, GROUP BY e HAVING para criar consultas que resumem e agrupam dados.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
