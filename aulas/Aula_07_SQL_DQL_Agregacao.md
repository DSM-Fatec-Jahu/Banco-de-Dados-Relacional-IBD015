# Aula 07 — SQL DQL: Consultas e Agregação

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 06](./Aula_06_SQL_Consultas_Basicas.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_08_Joins_Subconsultas_Views.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de usar as funções de agregação `COUNT`, `SUM`, `AVG`, `MIN` e `MAX`, agrupar resultados com `GROUP BY` e filtrar grupos com `HAVING`.

---

## 🎥 Vídeos de Apoio

- 📺 [GROUP BY e HAVING — MySQL](https://www.youtube.com/watch?v=HXV3zeQKqGY) — Bóson Treinamentos
- 📺 [Funções de Agregação no SQL](https://www.youtube.com/watch?v=yPu6qV5byu4) — CFBCursos

---

## 1. Funções de Agregação

Funções de agregação **calculam um único valor** a partir de múltiplas linhas. Elas ignoram valores `NULL` (exceto `COUNT(*)`).

[prompt para nanobanana: "Educational infographic showing five SQL aggregate functions as icons: COUNT as a numbered list icon, SUM as a sigma symbol with numbers, AVG as a balance scale showing average, MIN as a downward arrow pointing to smallest number, MAX as an upward arrow pointing to largest number. Each with a brief Portuguese label and example value. Clean flat design, blue color scheme, white background."]
![Funções de Agregação SQL](../imgs/Aula_07_img_01.png)

```sql
-- COUNT(*): conta todas as linhas, incluindo NULLs
SELECT COUNT(*) AS total_produtos FROM produtos;

-- COUNT(coluna): conta apenas valores não-NULL
SELECT COUNT(telefone) AS com_telefone FROM pessoas;

-- SUM: soma valores numéricos
SELECT SUM(valor_total) AS faturamento_total FROM pedidos WHERE status = 'entregue';

-- AVG: média aritmética
SELECT AVG(preco) AS preco_medio FROM produtos WHERE ativo = 1;

-- MIN e MAX
SELECT MIN(preco) AS mais_barato,
       MAX(preco) AS mais_caro
FROM   produtos WHERE ativo = 1;
```

---

## 2. GROUP BY — Agrupando Resultados

O `GROUP BY` divide as linhas em grupos com base em uma ou mais colunas e aplica a função de agregação em cada grupo.

[prompt para nanobanana: "Diagram showing how SQL GROUP BY works: on the left, a flat table with rows having different category values (Eletrônicos x3, Vestuário x2, Livros x4). In the middle, an arrow labeled GROUP BY categoria. On the right, a grouped result table showing one row per category with COUNT values (3, 2, 4). Clean educational flat design, blue arrows, white background, Portuguese labels."]
![Como funciona o GROUP BY](../imgs/Aula_07_img_02.png)

```sql
-- Quantidade de produtos por categoria
SELECT
    categoria_id,
    COUNT(*)           AS total_produtos,
    AVG(preco)         AS preco_medio,
    MIN(preco)         AS menor_preco,
    MAX(preco)         AS maior_preco
FROM   produtos
WHERE  ativo = 1
GROUP BY categoria_id
ORDER BY total_produtos DESC;
```

> ⚠️ **Regra fundamental do GROUP BY:** toda coluna no `SELECT` que **não** for uma função de agregação **deve** aparecer no `GROUP BY`. Isso é uma regra do padrão SQL. O MariaDB tem o modo `ONLY_FULL_GROUP_BY` que, quando ativado (padrão no MySQL 5.7.5+), aplica essa regra rigorosamente.

### 2.1 GROUP BY com múltiplas colunas

```sql
-- Total de pedidos e faturamento por status e mês
SELECT
    status,
    YEAR(data_pedido)  AS ano,
    MONTH(data_pedido) AS mes,
    COUNT(*)           AS total_pedidos,
    SUM(valor_total)   AS faturamento
FROM   pedidos
GROUP BY status, YEAR(data_pedido), MONTH(data_pedido)
ORDER BY ano, mes, faturamento DESC;
```

---

## 3. HAVING — Filtrando Grupos

O `HAVING` filtra grupos **depois** que a agregação foi aplicada. É o equivalente do `WHERE` para grupos.

```sql
-- Categorias com mais de 5 produtos ativos
SELECT
    categoria_id,
    COUNT(*) AS total
FROM   produtos
WHERE  ativo = 1
GROUP BY categoria_id
HAVING total > 5;

-- Clientes que fizeram mais de 3 pedidos
SELECT
    cliente_id,
    COUNT(*)        AS total_pedidos,
    SUM(valor_total) AS gasto_total
FROM   pedidos
WHERE  status <> 'cancelado'
GROUP BY cliente_id
HAVING total_pedidos > 3
ORDER BY gasto_total DESC;
```

### 3.1 WHERE vs HAVING — a diferença crítica

```sql
-- WHERE filtra LINHAS ANTES da agregação (mais eficiente — usa índices)
-- HAVING filtra GRUPOS DEPOIS da agregação

-- ✅ Correto: WHERE para filtros de linha, HAVING para filtros de grupo
SELECT categoria_id, COUNT(*) AS total, AVG(preco) AS media
FROM   produtos
WHERE  ativo = 1           -- filtra produtos inativos ANTES de agrupar
GROUP BY categoria_id
HAVING media > 500         -- filtra grupos onde a média é maior que 500
ORDER BY media DESC;

-- ❌ Ineficiente: usar HAVING para filtrar o que poderia ser feito com WHERE
SELECT categoria_id, COUNT(*) AS total, AVG(preco) AS media
FROM   produtos
GROUP BY categoria_id
HAVING ativo = 1           -- funciona, mas é mais lento pois agrupa tudo antes
   AND media > 500;
```

[prompt para nanobanana: "Two-panel comparison diagram. Left panel labeled 'WHERE - filtra antes do GROUP BY' showing a funnel icon filtering rows before they reach a GROUP BY box, with label 'Mais eficiente - usa índices'. Right panel labeled 'HAVING - filtra depois do GROUP BY' showing all rows going through GROUP BY first, then a filter removing some groups, with label 'Necessário quando filtra por valor agregado'. Blue and orange color scheme, clean flat design, white background, Portuguese labels."]
![WHERE vs HAVING](../imgs/Aula_07_img_03.png)

---

## 4. Ordem de Execução do SELECT

Entender a ordem em que o SQL processa cada cláusula é fundamental para escrever consultas corretas:

```
Ordem de execução (internamente):
  1. FROM        — define a(s) tabela(s) de origem
  2. WHERE       — filtra linhas individuais
  3. GROUP BY    — agrupa as linhas restantes
  4. HAVING      — filtra os grupos
  5. SELECT      — calcula as colunas de saída
  6. DISTINCT    — remove duplicatas
  7. ORDER BY    — ordena o resultado
  8. LIMIT       — limita a quantidade retornada
```

Isso explica por que você **não pode usar um alias definido no SELECT dentro do WHERE** — o `WHERE` é processado antes do `SELECT`. Mas pode usar o alias no `HAVING` e no `ORDER BY` (em alguns SGBDs, incluindo o MariaDB).

---

## 5. Exemplos Avançados com o Schema E-commerce

```sql
-- Top 5 produtos mais vendidos (por quantidade total)
SELECT
    ip.produto_id,
    p.nome,
    SUM(ip.quantidade)         AS total_vendido,
    SUM(ip.quantidade * ip.preco_unitario) AS receita
FROM   itens_pedidos ip
JOIN   produtos p ON p.id_produto = ip.produto_id
JOIN   pedidos  pd ON pd.id_pedido = ip.pedido_id
WHERE  pd.status = 'entregue'
GROUP BY ip.produto_id, p.nome
ORDER BY total_vendido DESC
LIMIT 5;

-- Faturamento mensal do ano de 2026
SELECT
    MONTH(data_pedido)     AS mes,
    COUNT(*)               AS pedidos,
    SUM(valor_total)       AS faturamento,
    AVG(valor_total)       AS ticket_medio
FROM   pedidos
WHERE  YEAR(data_pedido) = 2026
  AND  status = 'entregue'
GROUP BY MONTH(data_pedido)
ORDER BY mes;
```

---

## 6. Exercícios de Fixação

**Exercício 1:** quantos produtos existem por categoria? Liste o nome da categoria e a quantidade, em ordem decrescente.

**Exercício 2:** qual o valor médio dos pedidos por status? Mostre apenas os status com média acima de R$ 200.

**Exercício 3:** liste os clientes que gastaram mais de R$ 1.000 no total (somando todos os pedidos com status diferente de 'cancelado').

**Exercício 4:** qual é o produto com maior número de avaliações? Mostre o nome, total de avaliações e nota média.

**Exercício 5:** para cada mês de 2026, mostre quantos pedidos foram feitos e o faturamento total.

---

## 📚 Referências desta Aula

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 6.5 — Funções de Agregação. São Paulo: Pearson, 2018.
- TANIMURA, C. *SQL para Análise de Dados*. Cap. 3 — Agregações. São Paulo: Novatec, 2022.

---

> **Próxima aula:** na [Aula 08](./Aula_08_Joins_Subconsultas_Views.md) veremos como combinar dados de múltiplas tabelas com JOINs, usar subconsultas e criar Views.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
