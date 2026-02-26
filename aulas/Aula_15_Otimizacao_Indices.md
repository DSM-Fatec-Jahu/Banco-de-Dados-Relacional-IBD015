# Aula 15 — Otimização de Consultas e Índices

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 14](./Aula_14_Backup_Seguranca.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_16_Revisao_Geral.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de usar `EXPLAIN` e `EXPLAIN ANALYZE` para entender como o banco executa uma query, identificar gargalos de performance, criar índices estratégicos e reescrever consultas lentas de forma mais eficiente.

---

## 🎥 Vídeos de Apoio

- 📺 [EXPLAIN no MySQL — Entendendo planos de execução](https://www.youtube.com/watch?v=MuqMoO7KBm0) — Bóson Treinamentos
- 📺 [Índices B-Tree e Hash no MySQL](https://www.youtube.com/watch?v=HubezKbFL7E) — Otávio Collodel

---

## 1. Como o Banco Executa uma Query — O Query Optimizer

Antes de executar qualquer `SELECT`, o MariaDB passa a query pelo **Query Optimizer** — um componente que analisa diferentes formas de executar a consulta e escolhe o plano mais eficiente baseado em estatísticas sobre as tabelas e índices disponíveis.

[prompt para nanobanana: "Flowchart showing the SQL query execution pipeline. Stages from left to right: 1) SQL Query (text document icon) → 2) Parser (syntax check icon) → 3) Query Optimizer (brain/gear icon, labeled 'Analisa planos de execução') → 4) Execution Plan (blueprint icon) → 5) Storage Engine (cylinder icon, labeled 'InnoDB') → 6) Result Set (table icon). Clean flat design, blue pipeline arrows, white background, Portuguese labels."]
![Pipeline de execução de queries](../imgs/Aula_15_img_01.png)

---

## 2. EXPLAIN — Lendo o Plano de Execução

O `EXPLAIN` mostra **como** o banco pretende executar uma query — sem realmente executá-la.

```sql
-- Adicione EXPLAIN antes de qualquer SELECT
EXPLAIN SELECT p.nome, c.nome AS categoria
FROM   produtos p
INNER JOIN categorias c ON c.id_categoria = p.id_categoria
WHERE  p.preco > 500;
```

### 2.1 Interpretando as colunas do EXPLAIN

| Coluna | O que significa | O que você quer ver |
|---|---|---|
| `id` | Ordem de execução | — |
| `select_type` | Tipo de SELECT (SIMPLE, SUBQUERY...) | SIMPLE = mais eficiente |
| `table` | Tabela sendo acessada | — |
| `type` | **Tipo de acesso** — o mais importante | `const` > `ref` > `range` > `ALL` |
| `possible_keys` | Índices que poderiam ser usados | Quanto mais opções, melhor |
| `key` | Índice que **foi** escolhido | NULL = sem índice = problema! |
| `key_len` | Bytes do índice usados | — |
| `rows` | Estimativa de linhas a varrer | Quanto menor, melhor |
| `Extra` | Informações adicionais | Evite "Using filesort", "Using temporary" |

### 2.2 Tipos de Acesso (coluna `type`) — Do melhor ao pior

```
const    → acessa exatamente 1 linha pela PK ou UNIQUE (ótimo)
eq_ref   → uma linha por combinação em JOIN (ótimo)
ref      → múltiplas linhas por valor de índice não-único (bom)
range    → intervalo de valores em índice (aceitável)
index    → varre todo o índice (ruim)
ALL      → Full Table Scan — varre toda a tabela (péssimo!)
```

```sql
-- Exemplo de query com Full Table Scan (type=ALL):
EXPLAIN SELECT * FROM pedidos WHERE YEAR(data_pedido) = 2026;
-- type=ALL porque a função YEAR() impede o uso do índice em data_pedido

-- Reescrita para usar índice:
EXPLAIN SELECT * FROM pedidos
WHERE data_pedido BETWEEN '2026-01-01' AND '2026-12-31 23:59:59';
-- type=range — usa o índice em data_pedido
```

---

## 3. EXPLAIN ANALYZE — Executa e Mede

No MariaDB 10.9+ e MySQL 8.0.18+, o `EXPLAIN ANALYZE` realmente executa a query e mostra os tempos reais.

```sql
-- Executa a query e retorna o plano com tempos reais
EXPLAIN ANALYZE
SELECT p.nome, COUNT(ip.id_produto) AS vezes_vendido
FROM   produtos p
LEFT JOIN itens_pedidos ip ON ip.id_produto = p.id_produto
GROUP BY p.id_produto, p.nome
ORDER BY vezes_vendido DESC;
```

---

## 4. Estruturas de Índice — B-Tree vs Hash

[prompt para nanobanana: "Side-by-side diagram comparing B-Tree and Hash index structures. Left labeled 'B-Tree Index': shows a balanced tree structure with root node, internal nodes, and leaf nodes containing sorted key-value pairs, labeled 'Suporta: =, <, >, BETWEEN, LIKE prefix, ORDER BY'. Right labeled 'Hash Index': shows key values being passed through a hash function to bucket slots, labeled 'Apenas: = (igualdade exata), muito rápido para lookups simples'. Clean flat design, blue for B-Tree, orange for Hash, white background, Portuguese labels."]
![B-Tree vs Hash Index](../imgs/Aula_15_img_02.png)

**B-Tree (padrão do InnoDB):** organiza as chaves em uma árvore balanceada. Suporta buscas por igualdade, intervalos (`BETWEEN`, `>`, `<`), prefixos de `LIKE` e ordenação. É o tipo correto para a maioria dos casos.

**Hash:** usa uma função de hash para localizar registros em O(1). Extremamente rápido para buscas de igualdade exata, mas **não suporta** buscas por intervalo ou ordenação. No InnoDB, índices hash são criados automaticamente pelo buffer pool (adaptive hash) — você não os cria manualmente.

```sql
-- Criando índices explícitos com B-Tree (padrão):
CREATE INDEX idx_pedidos_status     ON pedidos (status);
CREATE INDEX idx_pedidos_data       ON pedidos (data_pedido);

-- Índice composto: a ordem das colunas importa!
-- Útil para queries que filtram por ambas as colunas OU só pela primeira
CREATE INDEX idx_pedidos_cli_status ON pedidos (cliente_id, status);
```

---

## 5. Índices Compostos — A Regra do Prefixo

Para um índice composto `(A, B, C)`, o MariaDB pode usá-lo para filtros em:
- `A` sozinho
- `A + B` juntos
- `A + B + C` juntos

Mas **não** pode usar o índice para filtros em `B` sozinho ou `C` sozinho.

```sql
-- Dado o índice: (cliente_id, status, data_pedido)

-- ✅ Usa o índice (prefixo completo):
WHERE cliente_id = 1 AND status = 'entregue' AND data_pedido > '2026-01-01'

-- ✅ Usa parte do índice:
WHERE cliente_id = 1 AND status = 'entregue'
WHERE cliente_id = 1

-- ❌ NÃO usa o índice (pula a primeira coluna):
WHERE status = 'entregue'
WHERE data_pedido > '2026-01-01'
```

---

## 6. Padrões que Impedem o Uso de Índices

```sql
-- ❌ Funções na coluna indexada — o índice não é usado:
WHERE YEAR(data_pedido) = 2026
WHERE UPPER(nome) = 'ANA LIMA'
WHERE LENGTH(cpf) = 11

-- ✅ Reescreva para manter a coluna limpa:
WHERE data_pedido BETWEEN '2026-01-01' AND '2026-12-31'
WHERE nome = 'Ana Lima'  -- use collation case-insensitive no índice

-- ❌ LIKE com % no início — não usa índice B-Tree:
WHERE nome LIKE '%Lima'

-- ✅ LIKE com % apenas no fim — usa índice:
WHERE nome LIKE 'Ana%'

-- ❌ Conversão implícita de tipo — índice pode ser ignorado:
WHERE cpf = 11122233344  -- cpf é CHAR, comparando com INT

-- ✅ Use o tipo correto:
WHERE cpf = '11122233344'
```

---

## 7. Otimizando JOINs

```sql
-- ❌ JOIN sem índice na coluna de junção: resulta em nested loop com full scan
-- Se não houver índice em itens_pedidos.id_produto, cada linha de produtos
-- causa uma varredura completa de itens_pedidos

-- ✅ Garanta índices em TODAS as colunas usadas em ON de JOINs:
-- A FK já cria o índice automaticamente no MariaDB — verifique com SHOW INDEX

-- ✅ Filtros mais restritivos primeiro (o otimizador geralmente já faz isso,
-- mas em queries complexas pode ajudar a especificar):
EXPLAIN
SELECT p.nome, SUM(ip.quantidade) AS total
FROM   produtos p
INNER JOIN itens_pedidos ip ON ip.id_produto = p.id_produto
INNER JOIN pedidos pd ON pd.id_pedido = ip.id_pedido
WHERE  pd.status = 'entregue'           -- filtra pedidos primeiro
  AND  pd.data_pedido >= '2026-01-01'   -- restringe por data
GROUP BY p.id_produto, p.nome
ORDER BY total DESC;
```

---

## 8. Queries Lentas — Identificando e Corrigindo

```sql
-- Ativar o log de queries lentas (queries > 1 segundo):
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 1;
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';

-- Ver as queries mais pesadas na sessão atual:
SHOW STATUS LIKE 'Handler_read%';
-- Handler_read_rnd_next alto = muitos full table scans

-- Analisar uma query problemática passo a passo:
-- 1. Execute o EXPLAIN
-- 2. Identifique tables com type=ALL e rows alto
-- 3. Verifique se falta índice nas colunas do WHERE/JOIN
-- 4. Crie o índice
-- 5. Execute EXPLAIN novamente e compare
-- 6. Execute ANALYZE TABLE para atualizar as estatísticas
ANALYZE TABLE pedidos;
```

---

## 9. Checklist de Otimização

```
[ ] Toda coluna usada em WHERE tem índice?
[ ] Toda coluna usada em JOIN ON tem índice?
[ ] Existem funções aplicadas sobre colunas indexadas no WHERE?
[ ] Há LIKE com % no início?
[ ] O tipo de dado da comparação bate com o tipo da coluna?
[ ] O EXPLAIN mostra type=ALL em alguma tabela grande?
[ ] O EXPLAIN mostra "Using filesort" ou "Using temporary"?
[ ] A query usa SELECT * desnecessário?
[ ] Há subconsultas que poderiam ser reescritas como JOIN?
[ ] Existem índices criados que nunca são usados (desperdício)?
```

---

## 10. Exercícios de Fixação

**Exercício 1:** execute `EXPLAIN` nas 5 queries que você escreveu nos exercícios das Aulas 07 e 08. Para cada uma, identifique se há `type=ALL` e proponha um índice que poderia melhorar a performance.

**Exercício 2:** reescreva a query abaixo para eliminar o Full Table Scan:
```sql
SELECT * FROM pedidos WHERE MONTH(data_pedido) = 3 AND YEAR(data_pedido) = 2026;
```

**Exercício 3:** dado o índice `(status, data_pedido)` na tabela `pedidos`, quais das seguintes queries se beneficiam desse índice? Justifique.
- a) `WHERE status = 'entregue'`
- b) `WHERE data_pedido > '2026-01-01'`
- c) `WHERE status = 'entregue' AND data_pedido > '2026-01-01'`

---

## 📚 Referências

- SILBERSCHATZ, A.; KORTH, H. F. *Sistema de banco de dados*. 6 ed. Cap. 12 — Indexação e Hashing. Rio de Janeiro: Elsevier, 2016.
- Documentação MariaDB — [EXPLAIN](https://mariadb.com/kb/en/explain/) | [Optimization and Indexes](https://mariadb.com/kb/en/optimization-and-indexes/)

---

> **Próxima aula:** [Aula 16 — Revisão Geral](./Aula_16_Revisao_Geral.md)

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
