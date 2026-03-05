# Aula 13 — Functions (UDF) e Transações

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 12](./Aula_12_Triggers.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_14_Backup_Seguranca.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de criar funções definidas pelo usuário (UDFs), entender a diferença fundamental entre FUNCTION e PROCEDURE, compreender as propriedades ACID de transações e implementar controle de concorrência com níveis de isolamento.

---

## 🎥 Vídeos de Apoio

- 📺 [Functions vs Procedures no MySQL](https://www.youtube.com/watch?v=cw4-OaAB_OU) — Bóson Treinamentos
- 📺 [Transações ACID no MySQL](https://www.youtube.com/watch?v=AcqtAEzuoj0) — CFBCursos

---

## 1. FUNCTION vs PROCEDURE — A Diferença Fundamental

[prompt para nanobanana: "Side-by-side comparison table showing FUNCTION vs PROCEDURE differences. FUNCTION column (blue): 'Retorna UM valor via RETURN', 'Pode ser usada dentro de SELECT', 'Não pode executar INSERT/UPDATE/DELETE por padrão', 'Chamada: SELECT fn_nome()'. PROCEDURE column (orange): 'Retorna via parâmetros OUT', 'Chamada com CALL', 'Pode executar qualquer DML', 'Pode ter múltiplos resultados'. Clean flat design, white background, Portuguese labels."]
![FUNCTION vs PROCEDURE](../imgs/Aula_13_img_01.png)

| Característica | FUNCTION | PROCEDURE |
|---|---|---|
| Retorno | `RETURN` — um único valor | Parâmetros `OUT` / `INOUT` |
| Uso no SELECT | ✅ `SELECT fn_nome(arg)` | ❌ Não pode |
| DML interno | ⚠️ Restrito (sem efeitos colaterais) | ✅ Qualquer operação |
| Chamada | `SELECT fn_nome()` ou dentro de query | `CALL nome_proc()` |
| Ideal para | Cálculos e transformações | Operações com múltiplos passos |

---

## 2. Criando Functions

```sql
DELIMITER $$

-- Função: calcula o preço com desconto para um cliente
CREATE FUNCTION fn_preco_com_desconto (
    p_preco      DECIMAL(10,2),
    p_id_cliente INT UNSIGNED
) RETURNS DECIMAL(10,2)
READS SQL DATA  -- indica que a função apenas lê dados (necessário no MariaDB)
DETERMINISTIC   -- o mesmo input sempre produz o mesmo output
BEGIN
    DECLARE v_categoria VARCHAR(20);
    DECLARE v_desconto  DECIMAL(5,2) DEFAULT 0.00;

    -- Reutiliza a procedure de classificação (chama procedure de dentro de function não é permitido)
    -- Então reimplementamos a lógica diretamente:
    SELECT CASE
        WHEN COALESCE(SUM(valor_total),0) >= 5000 THEN 15.00
        WHEN COALESCE(SUM(valor_total),0) >= 1000 THEN 10.00
        WHEN COUNT(*) > 0                          THEN  5.00
        ELSE 0.00
    END
    INTO v_desconto
    FROM pedidos
    WHERE cliente_id = p_id_cliente AND status = 'entregue';

    RETURN ROUND(p_preco * (1 - v_desconto / 100), 2);
END$$

-- Função: formata CPF com pontuação
CREATE FUNCTION fn_formatar_cpf (
    p_cpf CHAR(11)
) RETURNS VARCHAR(14)
NO SQL            -- não acessa nenhuma tabela
DETERMINISTIC
BEGIN
    RETURN CONCAT(
        SUBSTRING(p_cpf, 1, 3), '.',
        SUBSTRING(p_cpf, 4, 3), '.',
        SUBSTRING(p_cpf, 7, 3), '-',
        SUBSTRING(p_cpf, 10, 2)
    );
END$$

DELIMITER ;

-- Usando as functions diretamente em SELECT:
SELECT
    nome,
    fn_formatar_cpf(cpf)                          AS cpf_formatado
FROM pessoas;

SELECT
    p.nome                                         AS produto,
    p.preco                                        AS preco_cheio,
    fn_preco_com_desconto(p.preco, 1)              AS preco_cliente_1
FROM produtos p;
```

---

## 3. Gerenciando Functions

```sql
SHOW FUNCTION STATUS WHERE db = DATABASE();
SHOW CREATE FUNCTION fn_formatar_cpf;
DROP FUNCTION IF EXISTS fn_formatar_cpf;
```

---

## 4. Propriedades ACID

As propriedades ACID definem o que uma transação confiável deve garantir.

[prompt para nanobanana: "Infographic showing the four ACID properties as colored blocks: A - Atomicidade (red block with atom icon): 'Tudo ou nada - se uma operação falha, todas são revertidas'. C - Consistência (blue block with balance icon): 'O banco vai de um estado válido para outro estado válido'. I - Isolamento (green block with shield icon): 'Transações concorrentes não interferem entre si'. D - Durabilidade (purple block with lock icon): 'Uma vez confirmado, o dado persiste mesmo após falha do sistema'. Clean flat design, white background, Portuguese labels."]
![Propriedades ACID](../imgs/Aula_13_img_02.png)

**Atomicidade:** todas as operações da transação são executadas por completo ou nenhuma é. Garantida pelo mecanismo de `ROLLBACK`.

**Consistência:** a transação leva o banco de um estado consistente (satisfazendo todas as constraints) para outro estado consistente.

**Isolamento:** o grau em que uma transação em andamento é invisível para outras transações concorrentes. Controlado pelos **níveis de isolamento**.

**Durabilidade:** após um `COMMIT`, as mudanças persistem mesmo em caso de falha de hardware. Garantida pelo **redo log** do InnoDB.

---

## 5. Níveis de Isolamento

```sql
-- Ver o nível de isolamento atual da sessão:
-- MariaDB/MySQL:
SELECT @@transaction_isolation;
-- MySQL 5.7 e anteriores:
SELECT @@tx_isolation;

-- Alterar para a sessão atual:
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

| Nível | Dirty Read | Non-Repeatable Read | Phantom Read | Uso típico |
|---|---|---|---|---|
| `READ UNCOMMITTED` | ✅ Possível | ✅ Possível | ✅ Possível | Nunca recomendado |
| `READ COMMITTED` | ❌ Protegido | ✅ Possível | ✅ Possível | Relatórios, PostgreSQL padrão |
| `REPEATABLE READ` | ❌ Protegido | ❌ Protegido | ✅ Possível | **MariaDB/MySQL padrão** |
| `SERIALIZABLE` | ❌ Protegido | ❌ Protegido | ❌ Protegido | Alta consistência, baixa concorrência |

---

## 6. Exemplo Prático — Transação Segura com Verificação

```sql
DELIMITER $$

CREATE PROCEDURE sp_confirmar_pedido (
    IN  p_id_pedido  INT UNSIGNED,
    OUT p_sucesso    TINYINT,
    OUT p_mensagem   VARCHAR(255)
)
BEGIN
    DECLARE v_estoque_ok TINYINT DEFAULT 1;
    DECLARE v_produto_id INT;
    DECLARE v_qtd_pedida INT;
    DECLARE v_estoque    INT;
    DECLARE done         INT DEFAULT 0;

    DECLARE cur CURSOR FOR
        SELECT produto_id, quantidade FROM itens_pedidos WHERE pedido_id = p_id_pedido;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_sucesso = 0;
        SET p_mensagem = 'Erro interno. Transação revertida.';
    END;

    START TRANSACTION;

    -- Verifica estoque de cada produto do pedido
    OPEN cur;
    check_loop: LOOP
        FETCH cur INTO v_produto_id, v_qtd_pedida;
        IF done THEN LEAVE check_loop; END IF;

        SELECT estoque INTO v_estoque FROM produtos WHERE id_produto = v_produto_id FOR UPDATE;

        IF v_estoque < v_qtd_pedida THEN
            SET v_estoque_ok = 0;
            LEAVE check_loop;
        END IF;
    END LOOP;
    CLOSE cur;

    IF v_estoque_ok = 0 THEN
        ROLLBACK;
        SET p_sucesso  = 0;
        SET p_mensagem = CONCAT('Estoque insuficiente para o produto ', v_produto_id);
    ELSE
        UPDATE pedidos SET status = 'confirmado' WHERE id_pedido = p_id_pedido;
        COMMIT;
        SET p_sucesso  = 1;
        SET p_mensagem = 'Pedido confirmado com sucesso.';
    END IF;
END$$

DELIMITER ;
```

---

## 7. Exercícios de Fixação

**Exercício 1:** crie uma function `fn_calcular_total_pedido` que recebe `p_id_pedido` e retorna o valor total calculado a partir dos itens (sem usar o campo `valor_total` do pedido).

**Exercício 2:** crie uma function `fn_dias_ate_vencimento` que recebe uma data e retorna quantos dias faltam para ela (negativo se já passou).

**Exercício 3:** em qual nível de isolamento dois usuários simultâneos podem ler valores diferentes para a mesma linha sem que nenhum deles tenha feito commit? Por que isso é problemático?

---

## 📚 Referências

- SILBERSCHATZ, A.; KORTH, H. F. *Sistema de banco de dados*. 6 ed. Cap. 14 — Transações. Rio de Janeiro: Elsevier, 2016.
- Documentação MariaDB — [Stored Functions](https://mariadb.com/kb/en/stored-function-overview/)

---

> **Próxima aula:** [Aula 14 — Backup, Restauração e Segurança](./Aula_14_Backup_Seguranca.md)

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
