# Aula 04 — SQL DML: Manipulação de Dados

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 03](./Aula_03_SQL_DDL.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_05_Atividade_Modelagem_Ecommerce.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de inserir, atualizar e excluir dados em tabelas relacionais usando os comandos DML do SQL, compreender o funcionamento básico de transações e saber quando usar `COMMIT` e `ROLLBACK` para garantir a integridade dos dados.

---

## 🧭 Contexto

Na Aula 03 você criou as estruturas (tabelas, colunas, constraints). Agora vamos **popular** essas estruturas com dados reais. A DML — **Data Manipulation Language** — é o subconjunto do SQL responsável por inserir, modificar e remover registros. Os três comandos centrais são `INSERT`, `UPDATE` e `DELETE`.

[prompt para nanobanana: "Educational illustration showing three database operations as colored icons: INSERT as a green plus symbol adding a row to a table, UPDATE as a blue pencil editing a row, DELETE as a red trash can removing a row. Clean flat design, white background, labeled in Portuguese below each icon."]
![Operações DML](../imgs/Aula_04_img_01.png)

---

## 🎥 Vídeos de Apoio

- 📺 [INSERT, UPDATE e DELETE no MySQL](https://www.youtube.com/watch?v=7S_tz1z_5bA) — Programming with Mosh (legendas PT)
- 📺 [Transações no MySQL — COMMIT e ROLLBACK](https://www.youtube.com/watch?v=P3dITOSvpXU) — Bóson Treinamentos

---

## 1. INSERT — Inserindo Dados

O `INSERT` adiciona novas linhas a uma tabela. Existem três formas principais de uso.

### 1.1 INSERT com lista de colunas (forma recomendada)

```sql
-- Sempre especifique as colunas: o código fica resistente a mudanças no schema
INSERT INTO pessoas (nome, cpf, email, data_nascimento, telefone)
VALUES ('Ana Lima', '11122233344', 'ana@email.com', '1995-03-15', '14999990001');
```

### 1.2 INSERT múltiplo — várias linhas em um único comando

```sql
-- Muito mais eficiente que múltiplos INSERTs individuais
INSERT INTO categorias (nome, descricao) VALUES
    ('Eletrônicos',  'Computadores, smartphones e acessórios'),
    ('Vestuário',    'Roupas e calçados'),
    ('Livros',       'Livros técnicos e literatura'),
    ('Casa',         'Utensílios e decoração');
```

### 1.3 INSERT com SELECT — inserindo a partir de uma consulta

```sql
-- Copia dados de outra tabela (útil em migrações e relatórios)
INSERT INTO clientes_vip (id_pessoa, data_promocao)
SELECT id_pessoa, NOW()
FROM   pessoas
WHERE  id_pessoa IN (SELECT cliente_id FROM pedidos GROUP BY cliente_id HAVING COUNT(*) > 10);
```

### 1.4 INSERT OR IGNORE / ON DUPLICATE KEY UPDATE

```sql
-- MariaDB/MySQL: ignora o INSERT se violar uma constraint UNIQUE
INSERT IGNORE INTO categorias (nome) VALUES ('Eletrônicos');

-- Atualiza o registro existente se houver conflito de PK ou UNIQUE
INSERT INTO produtos (id_produto, nome, preco, estoque, id_categoria)
VALUES (1, 'Notebook Pro', 3800.00, 10, 1)
ON DUPLICATE KEY UPDATE
    preco   = VALUES(preco),
    estoque = VALUES(estoque);
```

> 💡 **Boas práticas no INSERT:** sempre especifique a lista de colunas. Um `INSERT INTO tabela VALUES (...)` sem lista de colunas quebra silenciosamente se a ordem ou quantidade de colunas mudar. Isso causa bugs difíceis de rastrear em produção.

---

## 2. UPDATE — Atualizando Dados

O `UPDATE` modifica valores em linhas existentes. É o comando que exige mais atenção: um `UPDATE` sem `WHERE` altera **todas as linhas** da tabela.

### 2.1 UPDATE básico

```sql
-- Atualiza o email de uma pessoa específica
UPDATE pessoas
SET    email = 'ana.lima@novoemail.com'
WHERE  id_pessoa = 1;
```

### 2.2 UPDATE múltiplas colunas

```sql
-- Várias colunas separadas por vírgula
UPDATE produtos
SET    preco   = 3200.00,
       estoque = estoque + 50,
       atualizado_em = NOW()
WHERE  id_produto = 5;
```

### 2.3 UPDATE com expressão e subquery

```sql
-- Aplica 10% de desconto em todos os produtos de uma categoria
UPDATE produtos
SET    preco = preco * 0.90
WHERE  id_categoria = (SELECT id_categoria FROM categorias WHERE nome = 'Eletrônicos');
```

### 2.4 O perigo do UPDATE sem WHERE

```sql
-- ⚠️ MUITO CUIDADO: este comando afeta TODAS as linhas da tabela
UPDATE produtos SET estoque = 0;  -- Zera o estoque de TODOS os produtos!

-- ✅ Sempre use WHERE para limitar o escopo:
UPDATE produtos SET estoque = 0 WHERE ativo = 0;
```

> ⚠️ **Dica de segurança:** no MariaDB/MySQL, você pode ativar o modo `sql_safe_updates` para que o banco rejeite `UPDATE` e `DELETE` sem `WHERE` ou sem usar uma coluna indexada. Ative com `SET sql_safe_updates = 1;`.

---

## 3. DELETE — Excluindo Dados

O `DELETE` remove linhas de uma tabela. Assim como o `UPDATE`, exige `WHERE` na maioria dos casos.

### 3.1 DELETE básico

```sql
-- Remove um registro específico
DELETE FROM itens_pedidos
WHERE  id_pedido = 5 AND id_produto = 3;
```

### 3.2 DELETE com subquery

```sql
-- Remove todos os pedidos cancelados há mais de 1 ano
DELETE FROM pedidos
WHERE  status = 'cancelado'
  AND  data_pedido < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

### 3.3 TRUNCATE — limpeza total de uma tabela

```sql
-- Remove TODAS as linhas e reinicia o AUTO_INCREMENT
-- Muito mais rápido que DELETE sem WHERE, pois não registra cada linha no log
TRUNCATE TABLE itens_pedidos;

-- Diferença do DELETE sem WHERE:
DELETE FROM itens_pedidos;    -- lento, registra cada linha, NÃO reseta AUTO_INCREMENT
TRUNCATE TABLE itens_pedidos; -- rápido, NÃO pode ser usado dentro de transação em alguns SGBDs
```

> ⚠️ **TRUNCATE vs DELETE:** o `TRUNCATE` não pode ser revertido com `ROLLBACK` no MariaDB (é um comando DDL internamente). Se precisar de segurança transacional, use `DELETE FROM tabela` dentro de uma transação.

---

## 4. Controle Básico de Transações

Uma **transação** é uma unidade de trabalho que deve ser executada por completo ou não executada de forma alguma. Isso é o princípio **A** do ACID (Atomicidade).

[prompt para nanobanana: "Educational diagram of a database transaction showing two scenarios side by side. Left side labeled 'COMMIT - Sucesso': shows steps INSERT, UPDATE, DELETE all with green checkmarks, then a COMMIT arrow saving to database cylinder. Right side labeled 'ROLLBACK - Falha': shows INSERT with green check, UPDATE with red X (error), then ROLLBACK arrow returning database to original state. Clean flat design, blue and green for success, red for failure, white background, labels in Portuguese."]
![Transações: COMMIT e ROLLBACK](../imgs/Aula_04_img_02.png)

### 4.1 Modo AUTOCOMMIT

Por padrão, o MariaDB opera em modo **AUTOCOMMIT ON**: cada comando DML é automaticamente confirmado assim que é executado. Isso significa que um `DELETE` executado sem `BEGIN` não pode ser desfeito.

```sql
-- Verificar o modo atual:
SELECT @@autocommit;  -- 1 = ativado, 0 = desativado

-- Desativar para a sessão atual:
SET autocommit = 0;
```

### 4.2 BEGIN, COMMIT e ROLLBACK

```sql
-- Inicia uma transação explícita
BEGIN;
-- ou equivalentemente:
START TRANSACTION;

-- Operações dentro da transação:
INSERT INTO pedidos (cliente_id, valor_total)
VALUES (1, 450.00);

UPDATE produtos
SET    estoque = estoque - 1
WHERE  id_produto = 10;

-- Se tudo correu bem, confirma permanentemente:
COMMIT;

-- Se algo deu errado, desfaz tudo desde o BEGIN:
ROLLBACK;
```

### 4.3 Exemplo prático — transferência segura

```sql
-- Simulando uma transferência que exige múltiplas operações atômicas:
BEGIN;

-- Débito na conta origem
UPDATE contas SET saldo = saldo - 500.00 WHERE id_conta = 1;

-- Verifica se o saldo ficou negativo (regra de negócio)
-- Em aplicações reais, isso seria feito na camada de aplicação ou com stored procedure
-- Aqui usamos um exemplo didático com SELECT

-- Crédito na conta destino
UPDATE contas SET saldo = saldo + 500.00 WHERE id_conta = 2;

-- Confirma apenas se ambas as operações foram bem-sucedidas
COMMIT;
```

### 4.4 SAVEPOINT — pontos de retorno parcial

```sql
BEGIN;

INSERT INTO pedidos (cliente_id, valor_total) VALUES (2, 200.00);

SAVEPOINT sp_pedido_inserido;  -- marca um ponto de retorno

INSERT INTO itens_pedidos (id_pedido, id_produto, quantidade, preco_unitario)
VALUES (LAST_INSERT_ID(), 5, 2, 100.00);

-- Se o INSERT de itens falhar, volta apenas até o savepoint
-- (o pedido ainda existirá na transação)
ROLLBACK TO SAVEPOINT sp_pedido_inserido;

-- Ou confirma tudo:
COMMIT;
```

---

## 5. Script Completo — Populando o E-commerce

```sql
USE loja_virtual;

-- Pessoas (clientes e funcionários)
INSERT INTO pessoas (nome, cpf, email, data_nascimento, telefone) VALUES
    ('Ana Lima',      '11122233344', 'ana@email.com',     '1990-05-10', '14999990001'),
    ('Carlos Melo',   '22233344455', 'carlos@email.com',  '1985-11-22', '19988880002'),
    ('Beatriz Costa', '33344455566', 'bi@email.com',       '1998-07-03', NULL),
    ('Diego Rocha',   '44455566677', 'diego@email.com',   '1992-02-28', '11977770003'),
    ('Prof. Ronan',   '55566677788', 'ronan@fatec.edu.br','1988-09-15', '14966660004');

-- Categorias
INSERT INTO categorias (nome, descricao) VALUES
    ('Eletrônicos',  'Computadores, smartphones e acessórios'),
    ('Periféricos',  'Mouse, teclado, headset e similares'),
    ('Livros',       'Técnicos, acadêmicos e literatura');

-- Produtos
INSERT INTO produtos (id_categoria, nome, preco, estoque) VALUES
    (1, 'Notebook Lenovo IdeaPad', 3499.90, 15),
    (1, 'Smartphone Samsung A55',  1899.00, 30),
    (2, 'Mouse Logitech MX Master',  399.90, 50),
    (2, 'Teclado Mecânico Redragon', 299.90, 40),
    (3, 'Sistemas de Banco de Dados — Elmasri', 189.90, 20);

-- Pedido de Ana Lima (cliente_id=1) atendido por Prof. Ronan (funcionario_id=5)
BEGIN;

INSERT INTO pedidos (cliente_id, funcionario_id, status, valor_total)
VALUES (1, 5, 'confirmado', 3899.80);

INSERT INTO itens_pedidos (id_pedido, id_produto, quantidade, preco_unitario)
VALUES
    (LAST_INSERT_ID(), 1, 1, 3499.90),
    (LAST_INSERT_ID(), 3, 1, 399.90);

COMMIT;
```

---

## 6. Exercícios de Fixação

**Exercício 1:** insira 3 novos produtos na tabela `produtos`, sendo um deles com `estoque = 0`.

**Exercício 2:** atualize o preço de todos os produtos da categoria 'Periféricos' aumentando 5%.

**Exercício 3:** usando uma transação, simule a criação de um pedido com dois itens. Ao final, faça `ROLLBACK` e verifique com `SELECT` que os dados não foram persistidos.

**Exercício 4:** qual a diferença entre `DELETE FROM produtos` e `TRUNCATE TABLE produtos`? Em que situação você usaria cada um?

---

## 📚 Referências desta Aula

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 6.3 — Comandos SQL de Atualização. São Paulo: Pearson, 2018.
- Documentação MariaDB — [INSERT](https://mariadb.com/kb/en/insert/), [UPDATE](https://mariadb.com/kb/en/update/), [DELETE](https://mariadb.com/kb/en/delete/)

---

> **Próxima aula:** na [Aula 05](./Aula_05_Atividade_Modelagem_Ecommerce.md), você aplicará tudo que aprendeu nas aulas anteriores para modelar e implementar um sistema de e-commerce completo.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
