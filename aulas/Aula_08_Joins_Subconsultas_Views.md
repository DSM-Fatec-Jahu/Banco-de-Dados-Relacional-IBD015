# Aula 08 — Junções (JOINs), Subconsultas e Visões (VIEW)

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 07](./Aula_07_SQL_DQL_Agregacao.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_09_Avaliacao_P1.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de combinar dados de múltiplas tabelas com `INNER JOIN`, `LEFT JOIN` e `RIGHT JOIN`, escrever subconsultas escalares e tabelares, e criar e utilizar Views para simplificar consultas complexas.

---

## 🎥 Vídeos de Apoio

- 📺 [SQL JOIN — Explicação Visual Completa](https://www.youtube.com/watch?v=9yeOJ0ZMUYw) — Bóson Treinamentos
- 📺 [Subqueries no MySQL — do básico ao avançado](https://www.youtube.com/watch?v=nJILuzw5UaU) — Bóson Treinamentos
- 📺 [CREATE VIEW no MySQL](https://www.youtube.com/watch?v=Eh1dz_RDKqM) — CFBCursos

---

## 1. Por que precisamos de JOINs?

Um banco de dados normalizado distribui as informações em múltiplas tabelas para eliminar redundâncias. Para obter uma visão completa dos dados — por exemplo, um pedido com nome do cliente, produtos e valores — precisamos **juntar** essas tabelas. O `JOIN` é o mecanismo que faz isso.

[prompt para nanobanana: "Visual diagram showing three types of SQL JOINs as Venn diagrams side by side. Left: INNER JOIN showing only the overlapping (intersection) area highlighted in blue, labeled 'INNER JOIN - Apenas correspondências'. Middle: LEFT JOIN showing left circle fully highlighted plus intersection in blue, labeled 'LEFT JOIN - Todos da esquerda + correspondências'. Right: RIGHT JOIN showing right circle fully highlighted plus intersection in blue, labeled 'RIGHT JOIN - Todos da direita + correspondências'. Clean flat design, white background, Portuguese labels."]
![Tipos de JOIN - Diagrama Venn](../imgs/Aula_08_img_01.png)

---

## 2. INNER JOIN

Retorna apenas as linhas que têm correspondência **em ambas** as tabelas.

```sql
-- Produtos com o nome de sua categoria
SELECT
    p.id_produto,
    p.nome              AS produto,
    p.preco,
    c.nome              AS categoria
FROM   produtos p
INNER JOIN categorias c ON c.id_categoria = p.categoria_id
ORDER BY c.nome, p.nome;

-- Pedidos com dados do cliente
SELECT
    pd.id_pedido,
    pe.nome             AS cliente,
    pe.email,
    pd.data_pedido,
    pd.status,
    pd.valor_total
FROM   pedidos pd
INNER JOIN pessoas pe ON pe.id_pessoa = pd.cliente_id
ORDER BY pd.data_pedido DESC;
```

> 💡 **Use sempre alias de tabela em JOINs:** `produtos p`, `categorias c`. Isso evita ambiguidade quando duas tabelas têm colunas com o mesmo nome (ex: `nome`) e torna o código muito mais legível.

---

## 3. LEFT JOIN

Retorna **todas as linhas da tabela da esquerda** e as correspondências da direita. Quando não há correspondência, os campos da tabela da direita aparecem como `NULL`.

```sql
-- TODOS os clientes, com seus pedidos (se houver)
-- Clientes sem pedidos aparecem com NULL nas colunas de pedido
SELECT
    pe.nome             AS cliente,
    pe.email,
    pd.id_pedido,
    pd.status,
    pd.valor_total
FROM   pessoas pe
LEFT JOIN pedidos pd ON pd.cliente_id = pe.id_pessoa
ORDER BY pe.nome;

-- Uso clássico: encontrar quem NÃO tem correspondência
-- Clientes que NUNCA fizeram um pedido:
SELECT pe.nome, pe.email
FROM   pessoas pe
LEFT JOIN pedidos pd ON pd.cliente_id = pe.id_pessoa
WHERE  pd.id_pedido IS NULL;
```

---

## 4. RIGHT JOIN

Retorna **todas as linhas da tabela da direita** e as correspondências da esquerda. Na prática, é menos comum — o `LEFT JOIN` com a ordem das tabelas invertida produz o mesmo resultado.

```sql
-- Equivalente: todas as categorias com seus produtos
SELECT
    c.nome              AS categoria,
    p.nome              AS produto,
    p.preco
FROM   produtos p
RIGHT JOIN categorias c ON c.id_categoria = p.categoria_id
ORDER BY c.nome;

-- Categorias SEM produtos cadastrados:
SELECT c.nome AS categoria_vazia
FROM   produtos p
RIGHT JOIN categorias c ON c.id_categoria = p.categoria_id
WHERE  p.id_produto IS NULL;
```

---

## 5. JOINs Múltiplos

```sql
-- Pedido completo: cliente, funcionário, itens e produtos
SELECT
    pd.id_pedido,
    cl.nome                               AS cliente,
    fn.nome                               AS atendente,
    pr.nome                               AS produto,
    ip.quantidade,
    ip.preco_unitario,
    (ip.quantidade * ip.preco_unitario)   AS subtotal
FROM   pedidos pd
INNER JOIN pessoas   cl ON cl.id_pessoa  = pd.cliente_id
LEFT  JOIN pessoas   fn ON fn.id_pessoa  = pd.funcionario_id
INNER JOIN itens_pedidos ip ON ip.pedido_id  = pd.id_pedido
INNER JOIN produtos  pr ON pr.id_produto = ip.produto_id
ORDER BY pd.id_pedido, pr.nome;
```

[prompt para nanobanana: "Database join diagram showing four tables connected by arrows: pedidos in the center, connecting to pessoas (twice: as cliente and as funcionario via LEFT JOIN), itens_pedidos (INNER JOIN), and produtos (INNER JOIN via itens_pedidos). Each connection labeled with the JOIN type and the FK column used. Clean flat design, tables as rectangles with column lists, colored arrows for each JOIN type, white background, Portuguese labels."]
![JOIN múltiplo com 4 tabelas](../imgs/Aula_08_img_02.png)

---

## 6. Subconsultas (Subqueries)

Uma subconsulta é um `SELECT` dentro de outro `SELECT`. Existem dois tipos principais.

### 6.1 Subconsulta Escalar — retorna um único valor

```sql
-- Produtos com preço acima da média
SELECT nome, preco
FROM   produtos
WHERE  preco > (SELECT AVG(preco) FROM produtos WHERE ativo = 1)
ORDER BY preco DESC;

-- Último pedido de cada cliente
SELECT id_pedido, cliente_id, data_pedido, valor_total
FROM   pedidos
WHERE  data_pedido = (
    SELECT MAX(data_pedido)
    FROM   pedidos pd2
    WHERE  pd2.cliente_id = pedidos.cliente_id  -- subconsulta correlacionada
);
```

### 6.2 Subconsulta Tabelar — retorna um conjunto de linhas

```sql
-- Clientes que já fizeram pelo menos um pedido entregue
SELECT nome, email
FROM   pessoas
WHERE  id_pessoa IN (
    SELECT DISTINCT cliente_id
    FROM   pedidos
    WHERE  status = 'entregue'
);

-- Usando EXISTS (mais eficiente que IN para grandes volumes)
SELECT nome, email
FROM   pessoas pe
WHERE  EXISTS (
    SELECT 1
    FROM   pedidos pd
    WHERE  pd.cliente_id = pe.id_pessoa
      AND  pd.status = 'entregue'
);
```

---

## 7. Views (Visões)

Uma **VIEW** é uma consulta SQL salva no banco de dados com um nome, que pode ser usada como se fosse uma tabela. Ela não armazena dados — apenas a definição da consulta.

### 7.1 Criando uma VIEW

```sql
-- View que apresenta pedidos com dados completos
CREATE OR REPLACE VIEW vw_pedidos_completos AS
SELECT
    pd.id_pedido,
    pd.data_pedido,
    pd.status,
    pd.valor_total,
    cl.nome                     AS cliente,
    cl.email                    AS email_cliente,
    COALESCE(fn.nome, 'Online') AS atendente
FROM   pedidos pd
INNER JOIN pessoas cl ON cl.id_pessoa = pd.cliente_id
LEFT  JOIN pessoas fn ON fn.id_pessoa = pd.funcionario_id;

-- View de produtos com categoria
CREATE OR REPLACE VIEW vw_produtos_catalogo AS
SELECT
    p.id_produto,
    p.nome,
    p.descricao,
    p.preco,
    p.estoque,
    c.nome AS categoria
FROM   produtos p
INNER JOIN categorias c ON c.id_categoria = p.id_categoria
WHERE  p.ativo = 1;
```

### 7.2 Usando uma VIEW

```sql
-- A view se comporta exatamente como uma tabela em consultas
SELECT * FROM vw_pedidos_completos WHERE status = 'enviado';
SELECT * FROM vw_produtos_catalogo WHERE preco < 500 ORDER BY preco;

-- Views podem ser usadas em JOINs
SELECT v.cliente, v.data_pedido, p.nome AS produto
FROM   vw_pedidos_completos v
JOIN   itens_pedidos ip ON ip.pedido_id   = v.id_pedido
JOIN   produtos      p  ON p.id_produto   = ip.produto_id;
```

### 7.3 Gerenciando Views

```sql
-- Listar views do banco atual
SHOW FULL TABLES WHERE table_type = 'VIEW';

-- Ver a definição de uma view
SHOW CREATE VIEW vw_pedidos_completos;

-- Remover uma view
DROP VIEW IF EXISTS vw_pedidos_completos;
```

> 💡 **Quando usar Views:** (1) simplificar consultas complexas que são usadas frequentemente; (2) criar camadas de abstração (esconder colunas sensíveis como CPF); (3) padronizar a forma como dados são apresentados para relatórios. Views **não melhoram performance** por si só — elas executam a query toda vez que são consultadas.

---

## 8. Exercícios de Fixação

**Exercício 1:** liste todos os produtos com seu nome de categoria, ordenados por categoria e depois por preço.

**Exercício 2:** encontre todos os clientes que nunca fizeram um pedido (use LEFT JOIN e IS NULL).

**Exercício 3:** liste os pedidos com nome do cliente, total de itens e valor total, para pedidos com mais de 2 itens diferentes.

**Exercício 4:** crie uma view chamada `vw_estoque_critico` que liste produtos ativos com estoque menor que 5, mostrando nome, estoque atual e categoria.

**Exercício 5:** usando subconsulta, encontre os produtos que nunca foram pedidos.

---

## 📚 Referências desta Aula

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 6.4 — JOIN e Cap. 7.1 — Views. São Paulo: Pearson, 2018.
- BEAULIEU, A. *Aprendendo SQL*. Cap. 5 — Consultando Múltiplas Tabelas. São Paulo: Novatec, 2010.

---

> **Próxima aula:** [Aula 09 — Avaliação P1](./Aula_09_Avaliacao_P1.md).

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
