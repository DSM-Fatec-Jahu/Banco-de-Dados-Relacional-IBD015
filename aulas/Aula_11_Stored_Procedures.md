# Aula 11 — Stored Procedures

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 10](./Aula_10_Integridade_Referencial.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_12_Triggers.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de criar Stored Procedures com parâmetros de entrada e saída, implementar lógica procedural com variáveis, condicionais e loops, tratar erros com handlers e identificar cenários onde procedures são a solução mais adequada.

---

## 🎥 Vídeos de Apoio

- 📺 [Stored Procedures no MySQL — do zero](https://www.youtube.com/watch?v=lx7PFtCJtGQ) — Bóson Treinamentos
- 📺 [Parâmetros IN, OUT e INOUT](https://www.youtube.com/watch?v=Ot9FD1BYVQA) — CFBCursos

---

## 1. O que é uma Stored Procedure?

Uma **Stored Procedure** (procedimento armazenado) é um bloco de código SQL salvo no banco de dados com um nome, que pode ser chamado quando necessário. Ela encapsula lógica de negócio dentro do banco, reduzindo o tráfego de rede e centralizando regras críticas.

[prompt para nanobanana: "Architecture diagram comparing two approaches. Left labeled 'Sem Procedure': shows application server sending 5 separate SQL queries (arrows) to database server, with label 'Alto tráfego de rede - 5 roundtrips'. Right labeled 'Com Procedure': shows application server sending 1 CALL command to database server, the procedure icon (gear) processing all 5 operations internally, with label 'Baixo tráfego - 1 roundtrip'. Clean flat design, blue for application, green for database, white background, Portuguese labels."]
![Stored Procedure reduz tráfego de rede](../imgs/Aula_11_img_01.png)

**Quando usar Stored Procedures:**
- Operações complexas com múltiplos passos interdependentes
- Lógica que deve ser executada identicamente independente da linguagem da aplicação
- Operações em lote (batch) com processamento linha a linha
- Operações que exigem tratamento de erro sofisticado dentro do banco

---

## 2. Sintaxe Básica

O `DELIMITER` é necessário porque o terminal normalmente usa `;` para encerrar comandos — mas dentro de uma procedure existem múltiplos `;`. Ao mudar o delimiter para `$$`, o MariaDB aguarda o `$$` para executar o bloco inteiro.

```sql
-- Altera o delimitador para que o ';' interno não encerre o comando prematuramente
DELIMITER $$

CREATE PROCEDURE nome_procedure (
    -- lista de parâmetros
)
BEGIN
    -- corpo da procedure
    -- instruções SQL e lógica procedural
END$$

-- Restaura o delimitador padrão
DELIMITER ;

-- Chamando a procedure:
CALL nome_procedure(argumentos);
```

---

## 3. Tipos de Parâmetros

```sql
-- IN: o chamador passa um valor para a procedure (entrada, somente leitura internamente)
-- OUT: a procedure retorna um valor para o chamador (saída, começa como NULL)
-- INOUT: o chamador passa um valor E recebe de volta o valor modificado

DELIMITER $$

CREATE PROCEDURE sp_exemplo_parametros (
    IN  p_entrada   INT,       -- recebe valor
    OUT p_saida     VARCHAR(50), -- retorna valor
    INOUT p_inout   INT        -- recebe e retorna
)
BEGIN
    SET p_saida = CONCAT('Recebi: ', p_entrada);
    SET p_inout = p_inout * 2;
END$$

DELIMITER ;

-- Usando:
SET @resultado = '';
SET @valor = 5;
CALL sp_exemplo_parametros(10, @resultado, @valor);
SELECT @resultado, @valor;
-- @resultado = 'Recebi: 10'
-- @valor     = 10 (5 * 2)
```

---

## 4. Variáveis, Condicionais e Loops

### 4.1 Variáveis locais com DECLARE

```sql
DELIMITER $$

CREATE PROCEDURE sp_calcular_desconto (
    IN  p_id_produto  INT UNSIGNED,
    IN  p_percentual  DECIMAL(5,2),
    OUT p_preco_final DECIMAL(10,2)
)
BEGIN
    DECLARE v_preco_atual DECIMAL(10,2);
    DECLARE v_desconto    DECIMAL(10,2);

    -- Busca o preço atual
    SELECT preco INTO v_preco_atual
    FROM   produtos
    WHERE  id_produto = p_id_produto;

    -- Calcula o desconto
    SET v_desconto    = v_preco_atual * (p_percentual / 100);
    SET p_preco_final = v_preco_atual - v_desconto;
END$$

DELIMITER ;

-- Usando:
CALL sp_calcular_desconto(1, 10.00, @preco_com_desconto);
SELECT @preco_com_desconto;
```

### 4.2 Condicionais IF / ELSEIF / ELSE

```sql
DELIMITER $$

CREATE PROCEDURE sp_classificar_cliente (
    IN  p_id_cliente INT UNSIGNED,
    OUT p_categoria  VARCHAR(20)
)
BEGIN
    DECLARE v_total_pedidos INT;
    DECLARE v_gasto_total   DECIMAL(12,2);

    SELECT COUNT(*), COALESCE(SUM(valor_total), 0)
    INTO   v_total_pedidos, v_gasto_total
    FROM   pedidos
    WHERE  cliente_id = p_id_cliente
      AND  status = 'entregue';

    IF v_gasto_total >= 5000 THEN
        SET p_categoria = 'VIP';
    ELSEIF v_gasto_total >= 1000 THEN
        SET p_categoria = 'Premium';
    ELSEIF v_total_pedidos > 0 THEN
        SET p_categoria = 'Regular';
    ELSE
        SET p_categoria = 'Novo';
    END IF;
END$$

DELIMITER ;
```

### 4.3 CASE

```sql
DELIMITER $$

CREATE PROCEDURE sp_desconto_por_status_cliente (
    IN  p_id_cliente  INT UNSIGNED,
    OUT p_desconto    DECIMAL(5,2)
)
BEGIN
    DECLARE v_categoria VARCHAR(20);
    CALL sp_classificar_cliente(p_id_cliente, v_categoria);

    CASE v_categoria
        WHEN 'VIP'     THEN SET p_desconto = 15.00;
        WHEN 'Premium' THEN SET p_desconto = 10.00;
        WHEN 'Regular' THEN SET p_desconto =  5.00;
        ELSE                SET p_desconto =  0.00;
    END CASE;
END$$

DELIMITER ;
```

### 4.4 Loops — WHILE e REPEAT

```sql
DELIMITER $$

CREATE PROCEDURE sp_gerar_relatorio_mensal (
    IN p_ano INT
)
BEGIN
    DECLARE v_mes INT DEFAULT 1;

    -- Cria tabela temporária para o relatório
    CREATE TEMPORARY TABLE IF NOT EXISTS tmp_relatorio (
        mes          INT,
        total_pedidos INT,
        faturamento  DECIMAL(12,2)
    );

    WHILE v_mes <= 12 DO
        INSERT INTO tmp_relatorio (mes, total_pedidos, faturamento)
        SELECT v_mes, COUNT(*), COALESCE(SUM(valor_total), 0)
        FROM   pedidos
        WHERE  YEAR(data_pedido)  = p_ano
          AND  MONTH(data_pedido) = v_mes
          AND  status             = 'entregue';

        SET v_mes = v_mes + 1;
    END WHILE;

    -- Retorna o relatório
    SELECT * FROM tmp_relatorio ORDER BY mes;
    DROP TEMPORARY TABLE tmp_relatorio;
END$$

DELIMITER ;

CALL sp_gerar_relatorio_mensal(2026);
```

---

## 5. Tratamento de Erros com HANDLER

```sql
DELIMITER $$

CREATE PROCEDURE sp_novo_pedido_seguro (
    IN  p_cliente_id    INT UNSIGNED,
    IN  p_id_produto    INT UNSIGNED,
    IN  p_quantidade    INT,
    OUT p_sucesso       TINYINT,
    OUT p_mensagem      VARCHAR(255)
)
BEGIN
    -- Handler captura qualquer erro SQL e define o comportamento
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_sucesso  = 0;
        SET p_mensagem = 'Erro ao processar pedido. Operação revertida.';
    END;

    -- Handler específico para violação de constraint
    DECLARE EXIT HANDLER FOR 1452  -- FK violation
    BEGIN
        ROLLBACK;
        SET p_sucesso  = 0;
        SET p_mensagem = 'Cliente ou produto não encontrado.';
    END;

    START TRANSACTION;

    INSERT INTO pedidos (cliente_id, status, valor_total)
    VALUES (p_cliente_id, 'pendente', 0.00);

    INSERT INTO itens_pedidos (pedido_id, produto_id, quantidade, preco_unitario)
    SELECT LAST_INSERT_ID(), id_produto, p_quantidade, preco
    FROM   produtos
    WHERE  id_produto = p_id_produto;

    -- Atualiza valor total do pedido
    UPDATE pedidos
    SET    valor_total = (
        SELECT SUM(quantidade * preco_unitario)
        FROM   itens_pedidos
        WHERE  pedido_id = LAST_INSERT_ID()
    )
    WHERE  id_pedido = LAST_INSERT_ID();

    COMMIT;
    SET p_sucesso  = 1;
    SET p_mensagem = 'Pedido criado com sucesso.';
END$$

DELIMITER ;
```

---

## 6. Gerenciando Procedures

```sql
-- Listar todas as procedures do banco atual
SHOW PROCEDURE STATUS WHERE db = DATABASE();

-- Ver o código de uma procedure
SHOW CREATE PROCEDURE sp_classificar_cliente;

-- Remover uma procedure
DROP PROCEDURE IF EXISTS sp_classificar_cliente;
```

---

## 7. Exercícios de Fixação

**Exercício 1:** crie uma procedure `sp_aplicar_cupom` que recebe `p_id_pedido` e `p_codigo_cupom` como parâmetros IN, valida se o cupom existe e está dentro da validade, aplica o desconto no `valor_total` do pedido e retorna o novo valor. Use tratamento de erro.

**Exercício 2:** crie uma procedure `sp_relatorio_estoque` que liste todos os produtos com estoque abaixo de um valor informado como parâmetro IN, agrupados por categoria.

**Exercício 3:** qual a diferença entre uma procedure e uma function? Em que situações cada uma é mais adequada? (Resposta na Aula 13.)

---

## 📚 Referências

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 13.3 — Procedimentos Armazenados. São Paulo: Pearson, 2018.
- Documentação MariaDB — [Stored Procedures](https://mariadb.com/kb/en/stored-procedures/)

---

> **Próxima aula:** [Aula 12 — Triggers](./Aula_12_Triggers.md)

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
