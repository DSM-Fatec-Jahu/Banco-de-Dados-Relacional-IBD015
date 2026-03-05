# Aula 12 — Triggers (Gatilhos)

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 11](./Aula_11_Stored_Procedures.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_13_Functions_Transacoes.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de criar Triggers `BEFORE` e `AFTER` para eventos `INSERT`, `UPDATE` e `DELETE`, entender a diferença entre triggers por linha (ROW) e por instrução (STATEMENT), implementar auditoria de dados com triggers e aplicar boas práticas para evitar triggers que causam efeitos colaterais difíceis de depurar.

---

## 🎥 Vídeos de Apoio

- 📺 [Triggers no MySQL — Gatilhos BEFORE e AFTER](https://www.youtube.com/watch?v=ibeW6BsUhv0) — Bóson Treinamentos
- 📺 [Auditoria com Triggers no MySQL](https://www.youtube.com/watch?v=G3Qc1KPe1SM) — CFBCursos

---

## 1. O que é um Trigger?

Um **Trigger** é um bloco de código que é executado **automaticamente** pelo banco de dados em resposta a um evento DML (`INSERT`, `UPDATE` ou `DELETE`) em uma tabela específica. Ao contrário de procedures (que são chamadas explicitamente), triggers disparam de forma transparente — o usuário nem sabe que estão executando.

[prompt para nanobanana: "Flowchart showing how a database trigger works. Center: a table labeled 'produtos' with INSERT/UPDATE/DELETE arrows pointing at it. Each arrow connects to a trigger box (gear icon) with timing labels: 'BEFORE - executa antes da operação' and 'AFTER - executa depois da operação'. The trigger boxes connect to actions like 'Validação', 'Auditoria', 'Atualização de outra tabela'. Clean flat design, orange for triggers, blue for tables, white background, Portuguese labels."]
![Como um Trigger funciona](../imgs/Aula_12_img_01.png)

---

## 2. Sintaxe Básica

```sql
DELIMITER $$

CREATE TRIGGER nome_trigger
    [BEFORE | AFTER] [INSERT | UPDATE | DELETE]
    ON nome_tabela
    FOR EACH ROW
BEGIN
    -- corpo do trigger
    -- acesso aos dados: OLD.coluna e NEW.coluna
END$$

DELIMITER ;
```

**Referências `OLD` e `NEW`:**

| Evento | `OLD` disponível? | `NEW` disponível? |
|---|---|---|
| INSERT | ❌ (não há linha antiga) | ✅ (a linha que está sendo inserida) |
| UPDATE | ✅ (valores antes da mudança) | ✅ (valores depois da mudança) |
| DELETE | ✅ (a linha que está sendo excluída) | ❌ (não haverá linha nova) |

---

## 3. BEFORE vs AFTER

**BEFORE:** executa antes da operação ser aplicada. Pode **modificar** os valores de `NEW` (útil para validação e normalização) e pode **cancelar** a operação lançando um erro.

**AFTER:** executa depois que a operação foi confirmada. Ideal para **auditoria** e **propagação** de efeitos colaterais para outras tabelas. Não pode modificar `NEW` (a linha já foi gravada).

---

## 4. Trigger BEFORE INSERT — Validação e Normalização

```sql
DELIMITER $$

-- Garante que o CPF seja armazenado apenas com dígitos (remove formatação)
-- e que o email seja sempre em minúsculas
CREATE TRIGGER trg_pessoas_before_insert
    BEFORE INSERT ON pessoas
    FOR EACH ROW
BEGIN
    -- Remove caracteres não numéricos do CPF
    SET NEW.cpf = REGEXP_REPLACE(NEW.cpf, '[^0-9]', '');

    -- Normaliza email para minúsculas
    SET NEW.email = LOWER(NEW.email);

    -- Valida CPF: deve ter exatamente 11 dígitos
    IF LENGTH(NEW.cpf) <> 11 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'CPF inválido: deve conter 11 dígitos numéricos.';
    END IF;
END$$

DELIMITER ;
```

> 💡 **`SIGNAL SQLSTATE`** é o mecanismo para lançar erros customizados no MariaDB/MySQL. O código `'45000'` é reservado para erros definidos pelo usuário. A mensagem aparece na aplicação como qualquer outro erro SQL.

---

## 5. Trigger AFTER INSERT — Efeito Colateral Automático

```sql
DELIMITER $$

-- Ao inserir um item em um pedido, decrementa automaticamente o estoque
CREATE TRIGGER trg_itens_pedidos_after_insert
    AFTER INSERT ON itens_pedidos
    FOR EACH ROW
BEGIN
    UPDATE produtos
    SET    estoque = estoque - NEW.quantidade
    WHERE  id_produto = NEW.produto_id;
END$$

DELIMITER ;
```

---

## 6. Trigger BEFORE UPDATE — Impedindo Mudanças Indevidas

```sql
DELIMITER $$

-- Impede que um pedido entregue seja alterado para qualquer outro status
CREATE TRIGGER trg_pedidos_before_update
    BEFORE UPDATE ON pedidos
    FOR EACH ROW
BEGIN
    IF OLD.status = 'entregue' AND NEW.status <> 'entregue' THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Um pedido entregue não pode ter seu status alterado.';
    END IF;

    -- Registra a data de entrega automaticamente quando o status muda para 'entregue'
    -- (assumindo que existe a coluna data_entrega em pedidos)
    IF OLD.status <> 'entregue' AND NEW.status = 'entregue' THEN
        SET NEW.data_entrega = NOW();
    END IF;
END$$

DELIMITER ;
```

---

## 7. Trigger AFTER UPDATE — Restaurando Estoque

```sql
DELIMITER $$

-- Quando a quantidade de um item é atualizada, ajusta o estoque
CREATE TRIGGER trg_itens_pedidos_after_update
    AFTER UPDATE ON itens_pedidos
    FOR EACH ROW
BEGIN
    -- Estorna a quantidade antiga e aplica a nova
    UPDATE produtos
    SET    estoque = estoque + OLD.quantidade - NEW.quantidade
    WHERE  id_produto = NEW.produto_id;
END$$

DELIMITER ;
```

---

## 8. Trigger AFTER DELETE — Restaurando Estoque na Exclusão

```sql
DELIMITER $$

-- Quando um item de pedido é removido, devolve a quantidade ao estoque
CREATE TRIGGER trg_itens_pedidos_after_delete
    AFTER DELETE ON itens_pedidos
    FOR EACH ROW
BEGIN
    UPDATE produtos
    SET    estoque = estoque + OLD.quantidade
    WHERE  id_produto = OLD.produto_id;
END$$

DELIMITER ;
```

---

## 9. Auditoria de Dados com Triggers

Um dos usos mais comuns de triggers é criar um **log de auditoria** — um registro histórico de todas as alterações feitas em uma tabela crítica.

```sql
-- Tabela de auditoria para preços de produtos
CREATE TABLE IF NOT EXISTS auditoria_precos (
    id_auditoria  INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    id_produto    INT UNSIGNED    NOT NULL,
    preco_antigo  DECIMAL(10,2)   NOT NULL,
    preco_novo    DECIMAL(10,2)   NOT NULL,
    alterado_em   TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    alterado_por  VARCHAR(100)    NOT NULL DEFAULT USER(),
    CONSTRAINT pk_auditoria PRIMARY KEY (id_auditoria)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

DELIMITER $$

CREATE TRIGGER trg_produtos_audit_preco
    AFTER UPDATE ON produtos
    FOR EACH ROW
BEGIN
    -- Registra apenas quando o preço de fato mudou
    IF OLD.preco <> NEW.preco THEN
        INSERT INTO auditoria_precos (id_produto, preco_antigo, preco_novo, alterado_por)
        VALUES (NEW.id_produto, OLD.preco, NEW.preco, USER());
    END IF;
END$$

DELIMITER ;

-- Testando:
UPDATE produtos SET preco = 3299.90 WHERE id_produto = 1;
SELECT * FROM auditoria_precos;
```

---

## 10. Gerenciando Triggers

```sql
-- Listar todos os triggers do banco atual
SHOW TRIGGERS;

-- Ver o código de um trigger específico
SHOW CREATE TRIGGER trg_produtos_audit_preco;

-- Remover um trigger
DROP TRIGGER IF EXISTS trg_produtos_audit_preco;
```

---

## 11. Boas Práticas e Armadilhas

**Nomeie triggers de forma descritiva:** o padrão `trg_[tabela]_[timing]_[evento]` (ex: `trg_produtos_after_update`) torna imediatamente claro quando e onde o trigger atua.

**Evite triggers em cascata:** um trigger que modifica a tabela A dispara o trigger da tabela A, que modifica B, que dispara o trigger de B... isso cria cadeias difíceis de rastrear e depurar.

**Não coloque lógica de negócio complexa em triggers:** use procedures para isso. Triggers devem ser curtos e focados em um único propósito.

**Triggers tornam o DML imprevisível para quem não os conhece:** sempre documente a existência de triggers em comentários no DDL das tabelas que eles monitoram.

> ⚠️ **Diferença MariaDB vs MySQL:** no MariaDB, é possível ter **múltiplos triggers** do mesmo tipo na mesma tabela (ex: dois `AFTER INSERT`). No MySQL 5.6 e anteriores, apenas um trigger por tipo por tabela era permitido. A partir do MySQL 5.7+, múltiplos triggers são suportados com a cláusula `FOLLOWS` / `PRECEDES` para ordenação.

---

## 12. Exercícios de Fixação

**Exercício 1:** crie um trigger que, ao excluir um pedido, registre na tabela `auditoria_pedidos` (que você deve criar) o id do pedido, o cliente, o valor total e a data/hora da exclusão.

**Exercício 2:** crie um trigger BEFORE INSERT em `avaliacoes` que impeça que um cliente avalie um produto que ele não comprou (sem um pedido com status 'entregue' contendo aquele produto).

**Exercício 3:** qual é a diferença entre usar uma PROCEDURE e um TRIGGER para manter o estoque atualizado? Quais são as vantagens e desvantagens de cada abordagem?

---

## 📚 Referências

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 13.4 — Triggers. São Paulo: Pearson, 2018.
- Documentação MariaDB — [Triggers](https://mariadb.com/kb/en/triggers/)

---

> **Próxima aula:** [Aula 13 — Functions e Transações](./Aula_13_Functions_Transacoes.md)

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
