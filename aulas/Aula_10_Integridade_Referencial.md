# Aula 10 — Integridade Referencial e Restrições

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 09](./Aula_09_Avaliacao_P1.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_11_Stored_Procedures.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você compreenderá profundamente as restrições de domínio e integridade referencial, saberá configurar as ações `ON DELETE` e `ON UPDATE` para cada cenário de negócio, e entenderá como índices básicos funcionam e por que são criados automaticamente por constraints.

---

## 🎥 Vídeos de Apoio

- 📺 [Integridade Referencial e FK no MySQL](https://www.youtube.com/watch?v=oBSHWGmjpjQ) — Bóson Treinamentos
- 📺 [Índices no MySQL — Como funcionam](https://www.youtube.com/watch?v=HubezKbFL7E) — Bóson Treinamentos

---

## 1. Restrições de Integridade

Restrições de integridade são regras que o próprio banco de dados aplica automaticamente para garantir que os dados permaneçam válidos e consistentes — independentemente de como foram inseridos (via aplicação, script ou acesso direto).

[prompt para nanobanana: "Educational diagram showing four types of database integrity constraints as separate labeled boxes: 1) Integridade de Domínio - icon of a filter with data types (INT, VARCHAR, CHECK), 2) Integridade de Entidade - icon of a key with PRIMARY KEY label, 3) Integridade Referencial - icon of two tables connected by a chain link with FOREIGN KEY label, 4) Integridade de Unicidade - icon of a stamp with UNIQUE label. Clean flat design, blue color scheme, white background, Portuguese labels."]
![Tipos de Restrições de Integridade](../imgs/Aula_10_img_01.png)

### 1.1 Integridade de Domínio

Define os valores válidos para cada coluna através de tipos de dados, `NOT NULL`, `DEFAULT` e `CHECK`.

```sql
-- Tipo de dado restringe o domínio (TEXT não aceita números, DATE não aceita strings inválidas)
-- CHECK restringe o intervalo de valores válidos
ALTER TABLE produtos
    ADD CONSTRAINT ck_estoque_min CHECK (estoque >= 0),
    ADD CONSTRAINT ck_preco_min   CHECK (preco > 0);

-- DEFAULT define o valor usado quando não informado
ALTER TABLE pedidos
    ALTER COLUMN status SET DEFAULT 'pendente';
```

### 1.2 Integridade de Entidade (Chave Primária)

Toda tabela deve ter uma PK. A PK implica `NOT NULL` + `UNIQUE`.

```sql
-- Uma tabela sem PK é tecnicamente possível no MariaDB, mas nunca recomendada
-- Verifica tabelas sem PK no banco atual:
SELECT t.table_name
FROM   information_schema.tables t
LEFT JOIN information_schema.table_constraints tc
       ON tc.table_name    = t.table_name
      AND tc.table_schema  = t.table_schema
      AND tc.constraint_type = 'PRIMARY KEY'
WHERE  t.table_schema = DATABASE()
  AND  tc.constraint_name IS NULL;
```

---

## 2. FOREIGN KEY — Integridade Referencial em Detalhe

A FK garante que qualquer valor na coluna filha **existe** na coluna pai referenciada. Vamos explorar cada combinação de ação possível com cenários reais.

### 2.1 RESTRICT (padrão mais seguro)

Impede a operação no pai se existirem filhos vinculados. A verificação ocorre imediatamente.

```sql
-- Uma categoria não pode ser excluída se tiver produtos
-- Um produto não pode ser excluído se tiver itens em pedidos
CONSTRAINT fk_produto_cat FOREIGN KEY (id_categoria)
    REFERENCES categorias (id_categoria)
    ON DELETE RESTRICT
    ON UPDATE CASCADE
```

**Quando usar:** sempre que a exclusão do pai seria uma perda de dados grave — como excluir um cliente que tem pedidos históricos.

### 2.2 CASCADE

Propaga a operação do pai para todos os filhos.

```sql
-- Excluir um pedido automaticamente exclui seus itens
CONSTRAINT fk_item_pedido FOREIGN KEY (id_pedido)
    REFERENCES pedidos (id_pedido)
    ON DELETE CASCADE
    ON UPDATE CASCADE
```

**Quando usar:** quando o filho não tem sentido sem o pai (itens de pedido, telefones de pessoa, imagens de produto). Use com cuidado — um DELETE no pai pode remover cascata de dados.

### 2.3 SET NULL

Define a FK como NULL nos filhos quando o pai é removido.

```sql
-- Se o funcionário for desligado, os pedidos que atendeu ficam sem atendente
CONSTRAINT fk_pedido_funcionario FOREIGN KEY (funcionario_id)
    REFERENCES pessoas (id_pessoa)
    ON DELETE SET NULL
    ON UPDATE CASCADE
```

**Quando usar:** quando a associação é opcional e o filho pode existir independentemente do pai.

### 2.4 Testando o Comportamento das FKs

```sql
USE ecommerce;

-- Tentativa de excluir categoria que tem produtos (RESTRICT):
DELETE FROM categorias WHERE id_categoria = 1;
-- ERROR 1451: Cannot delete or update a parent row: a foreign key constraint fails

-- Excluir pedido (CASCADE exclui itens automaticamente):
DELETE FROM pedidos WHERE id_pedido = 1;
-- Sucesso — os itens_pedidos com id_pedido=1 são removidos junto

-- Verificar: itens do pedido 1 foram removidos?
SELECT * FROM itens_pedidos WHERE id_pedido = 1;  -- retorna vazio
```

---

## 3. Índices — Como Funcionam e Por Que Existem

Um **índice** é uma estrutura de dados separada que o banco mantém para acelerar buscas. Funciona como o índice remissivo de um livro: em vez de varrer todas as páginas, você vai diretamente à referência.

[prompt para nanobanana: "Comparison diagram showing database performance with and without index. Left side labeled 'Sem índice - Full Table Scan': shows a magnifying glass scanning every row of a large table (all rows highlighted in yellow), with label '10.000 linhas verificadas'. Right side labeled 'Com índice - Index Scan': shows a B-tree index structure pointing directly to 3 matching rows in the table, with label '3 linhas verificadas'. Include a clock icon showing much faster time on the indexed side. Clean educational flat design, white background, red for slow, green for fast, Portuguese labels."]
![Comparação de performance com e sem índice](../imgs/Aula_10_img_02.png)

### 3.1 Índices Criados Automaticamente por Constraints

```sql
-- Toda PK gera automaticamente um índice UNIQUE (B-Tree)
-- Toda UNIQUE gera automaticamente um índice
-- Toda FK gera automaticamente um índice no MariaDB (diferente do MySQL!)

-- Ver índices de uma tabela:
SHOW INDEX FROM produtos;

-- Resultado esperado para 'produtos':
-- Table     | Key_name              | Column_name  | Non_unique
-- produtos  | PRIMARY               | id_produto   | 0
-- produtos  | uq_prod_nome          | nome         | 0  (se existir)
-- produtos  | fk_produto_cat        | id_categoria | 1
```

### 3.2 Criando Índices Manualmente

```sql
-- Índice simples em coluna frequentemente consultada
CREATE INDEX idx_produtos_ativo ON produtos (ativo);

-- Índice composto — útil para queries que filtram por múltiplas colunas juntas
CREATE INDEX idx_pedidos_cliente_status ON pedidos (cliente_id, status);

-- Índice único (equivalente a UNIQUE constraint, mas criado separadamente)
CREATE UNIQUE INDEX idx_uq_cpf ON pessoas (cpf);

-- Removendo um índice
DROP INDEX idx_produtos_ativo ON produtos;
```

### 3.3 Quando criar índices

```sql
-- Crie índices em colunas que aparecem frequentemente em:
-- WHERE, JOIN ON, ORDER BY, GROUP BY

-- Exemplos típicos:
CREATE INDEX idx_pedidos_data     ON pedidos (data_pedido);
CREATE INDEX idx_pedidos_status   ON pedidos (status);
CREATE INDEX idx_produtos_preco   ON produtos (preco);
```

> ⚠️ **Índices têm custo:** cada índice ocupa espaço em disco e **torna as operações de escrita mais lentas** (INSERT, UPDATE, DELETE precisam atualizar o índice). Crie índices cirurgicamente nas colunas certas — não em todas as colunas.

---

## 4. Exercícios de Fixação

**Exercício 1:** no banco `ecommerce`, tente excluir uma categoria que possui produtos. Observe o erro. Depois, altere a FK para `ON DELETE SET NULL` e repita. O que aconteceu com os produtos?

**Exercício 2:** crie um índice composto em `itens_pedidos` que cubra a consulta: `SELECT * FROM itens_pedidos WHERE id_produto = ? AND quantidade > ?`.

**Exercício 3:** identifique quais colunas das tabelas do e-commerce seriam boas candidatas a índices manuais (além dos já criados automaticamente) e justifique sua escolha.

---

## 📚 Referências

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 5 — Restrições de Integridade. São Paulo: Pearson, 2018.

---

> **Próxima aula:** [Aula 11 — Stored Procedures](./Aula_11_Stored_Procedures.md)

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
