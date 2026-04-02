# Exercício de Revisão — Preparação para P1

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 08](./Aula_08_Joins_Subconsultas_Views.md) · [Voltar ao README](../README.md)

---

## 📌 Sobre este Exercício

Este exercício **não vale nota**, mas cobre exatamente os temas que serão cobrados na **Avaliação P1**. A proposta é que você construa, do zero, um script SQL completo para um sistema de gestão escolar, passando por todas as etapas que estudamos nas aulas anteriores: DDL, DML e DQL.

> 💡 **Dica de estudo:** tente resolver sem consultar as aulas primeiro. Use as aulas de referência apenas para tirar dúvidas pontuais. Quanto mais autônomo for o processo, mais preparado você estará para a prova.

---

## 🏫 Contexto do Sistema

Você irá modelar e implementar um banco de dados para uma **escola técnica** — com gestão de usuários do sistema, alunos, professores, funcionários administrativos, cursos, disciplinas, matrículas e notas por matrícula.

O script completo deve ser executável do zero em um banco limpo — ou seja, ao rodar o arquivo pela segunda vez, ele deve recriar tudo sem erros.

---

## 📋 Regras de Negócio

Leia com atenção antes de começar qualquer etapa.

- A escola gerencia **usuários do sistema** que realizam login. Um usuário pode ser aluno, professor ou funcionário administrativo (generalização e especialização).
- Cada usuário possui nome completo, e-mail único, senha (armazenar apenas o hash) e status de conta (ativa ou inativa).
- Um **aluno** possui, além dos dados de usuário, número de matrícula (único), data de ingresso e turno preferencial (matutino, vespertino ou noturno).
- Um **professor** possui, além dos dados de usuário, registro funcional (único), titulação (graduação, especialização, mestrado ou doutorado) e área de atuação.
- Um **funcionário administrativo** possui, além dos dados de usuário, matrícula funcional (único) e setor de trabalho.
- Um **curso** possui nome, carga horária total e nível (técnico, graduação ou pós-graduação). Um aluno está vinculado a exatamente um curso.
- Uma **disciplina** pertence a exatamente um curso e é lecionada por exatamente um professor. Possui nome, código único (ex: `BD001`), carga horária e período (semestre em que é ofertada normalmente).
- A **matrícula** representa o vínculo entre um aluno e uma disciplina em um semestre específico (ex: `2026-1`). Um aluno pode se matricular na mesma disciplina em semestres diferentes. A matrícula tem status: cursando, aprovado, reprovado ou trancado.
- Cada matrícula pode ter **múltiplas notas** registradas, identificadas pelo tipo (prova_1, prova_2, trabalho, recuperacao) e pelo valor (0,00 a 10,00). Cada tipo de nota ocorre no máximo uma vez por matrícula.
- Toda tabela deve ter campos de auditoria temporal (`criado_em`, `alterado_em`, `deletado_em`) e o campo `alterado_por_id` registrando **qual usuário** fez a última modificação.

---

## 🗂️ Estrutura do Script

Organize o seu script `.sql` nas seções a seguir, **nessa ordem**, com comentários separando cada bloco.

---

## Parte 1 — DDL: Criação do Banco de Dados

### 1.1 O que fazer

Crie o banco de dados e todas as tabelas necessárias para atender às regras de negócio descritas acima, seguindo **rigorosamente** as convenções de nomenclatura da disciplina:

- snake_case e minúsculas para todos os nomes criados pelo usuário
- Palavras reservadas SQL em MAIÚSCULAS
- Nomes de tabelas sempre no plural
- PK no padrão `id_nome_tabela_singular` como `BIGINT UNSIGNED`
- FK no padrão `tabela_referenciada_id` (Regra 6) ou papel semântico quando necessário (Regra 7)
- `ENGINE=InnoDB`, `DEFAULT CHARSET=utf8mb4`, `COLLATE=utf8mb4_unicode_ci`
- `DROP DATABASE IF EXISTS` e `CREATE DATABASE` no início para garantir execução limpa

### 1.2 Campos de auditoria obrigatórios em todas as tabelas

Toda tabela do sistema deve conter os seguintes campos de auditoria, criados **inline** na definição da tabela:

| Campo | Tipo | Comportamento |
|---|---|---|
| `criado_em` | `TIMESTAMP NOT NULL` | Valor padrão `CURRENT_TIMESTAMP`, nunca alterado após inserção |
| `alterado_em` | `TIMESTAMP NOT NULL` | Valor padrão `CURRENT_TIMESTAMP`, atualizado automaticamente a cada `UPDATE` |
| `deletado_em` | `TIMESTAMP NULL` | Fica `NULL` enquanto o registro está ativo; preenchido na exclusão lógica |

```sql
-- Exemplo de como esses campos devem aparecer em cada tabela:
criado_em    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
alterado_em  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
                                ON UPDATE CURRENT_TIMESTAMP,
deletado_em  TIMESTAMP NULL     DEFAULT NULL
             COMMENT 'Preenchido na exclusão lógica — NULL indica registro ativo',
```

> 💡 O campo `deletado_em` é a chave da **exclusão lógica**: em vez de apagar o registro com `DELETE`, a aplicação registra o momento em que ele foi "deletado". Consultas que exibem apenas registros ativos devem sempre filtrar `WHERE deletado_em IS NULL`.

### 1.3 Campo de auditoria `alterado_por_id` — via ALTER TABLE

O campo `alterado_por_id` referencia a tabela `usuarios` e identifica **qual usuário do sistema** realizou a última modificação em cada registro. Como esse campo cria uma dependência entre todas as tabelas e a tabela `usuarios`, ele apresenta uma dificuldade técnica: não é possível criá-lo como FK inline nas tabelas criadas **antes** de `usuarios`, e tentá-lo na própria tabela `usuarios` geraria uma auto-referência que pode conflitar com os dados iniciais.

Por isso, a solução correta é: **crie todas as tabelas sem o campo `alterado_por_id` e, após a criação completa do banco, adicione a coluna e a FK em cada tabela via `ALTER TABLE`.**

```sql
-- Exemplo do padrão esperado — repita para CADA tabela:
ALTER TABLE usuarios
    ADD COLUMN alterado_por_id BIGINT UNSIGNED NULL
        COMMENT 'FK para usuarios — usuário que fez a última alteração',
    ADD CONSTRAINT fk_usuarios_alterado_por
        FOREIGN KEY (alterado_por_id)
        REFERENCES usuarios (id_usuario)
        ON DELETE SET NULL
        ON UPDATE CASCADE;

-- Repita o padrão acima para: alunos, professores, funcionarios,
-- cursos, disciplinas, matriculas, notas
```

> ⚠️ **Atenção à Regra 7:** o campo se chama `alterado_por_id` — e não `usuario_id` — porque ele referencia a tabela `usuarios` exercendo um **papel semântico específico** (o papel de "responsável pela última alteração"). Isso é exatamente o caso de uso da Regra 7 de nomenclatura.

### 1.4 Checklist do DDL

Antes de avançar, verifique se o seu script contém:

- [ ] `DROP DATABASE IF EXISTS escola; CREATE DATABASE escola;`
- [ ] Tabela `usuarios` com generalização (campos base de todos os perfis)
- [ ] Tabelas `alunos`, `professores` e `funcionarios` como especializações de `usuarios`
- [ ] Tabelas `cursos` e `disciplinas` com seus relacionamentos corretos
- [ ] Tabela `matriculas` resolvendo o N:M entre alunos e disciplinas, com semestre
- [ ] Tabela `notas` como entidade fraca de `matriculas`
- [ ] Campos `criado_em`, `alterado_em` e `deletado_em` em **todas** as tabelas (inline na criação)
- [ ] Campo `alterado_por_id` adicionado via `ALTER TABLE` em **todas** as tabelas, **após** a criação completa do banco
- [ ] `PRIMARY KEY`, `FOREIGN KEY` com ações de integridade, `UNIQUE`, `CHECK`, `ENUM` onde aplicável
- [ ] Tipos de dados coerentes: `DECIMAL(4,2)` para nota, `ENUM` para status e titulação, `VARCHAR` adequado para cada campo
- [ ] Apagar e recriar no início para garantir idempotência

---

## Parte 2 — DML: Manipulação de Dados

### 2.1 Inserção — populando o banco (mínimo 3 registros por tabela)

Insira ao menos **3 registros** em cada tabela. Atenção às dependências: insira na ordem correta (tabelas pai antes das filhas).

Use `INSERT` com lista de colunas explícita. Lembre-se: nunca `INSERT INTO tabela VALUES (...)` sem listar as colunas. Os campos `criado_em` e `alterado_em` não precisam ser informados — eles são preenchidos automaticamente pelo banco com `CURRENT_TIMESTAMP`.

**Sugestão de dados para a tabela `usuarios`:**

| nome | email | senha_hash | status |
|---|---|---|---|
| Ana Beatriz Ferreira | ana.ferreira@escola.edu.br | `$2b$12$ExemploHashBcryptAqui` | ativa |
| Carlos Eduardo Lima | carlos.lima@escola.edu.br | `$2b$12$ExemploHashBcryptAqui` | ativa |
| Mariana Costa Souza | mariana.souza@escola.edu.br | `$2b$12$ExemploHashBcryptAqui` | ativa |
| Prof. Roberto Alves | roberto.alves@escola.edu.br | `$2b$12$ExemploHashBcryptAqui` | ativa |
| Prof. Sandra Melo | sandra.melo@escola.edu.br | `$2b$12$ExemploHashBcryptAqui` | ativa |
| Func. Juliana Pires | juliana.pires@escola.edu.br | `$2b$12$ExemploHashBcryptAqui` | ativa |

> 💡 Para o campo `senha_hash`, use uma string qualquer no formato de hash — ex: `'$2b$12$ExemploHashBcryptAqui'`. Em produção, o hash seria gerado pela aplicação antes de chegar ao banco.

### 2.2 Atualização — 1 registro por tabela

Atualize **1 registro em cada tabela**. Use sempre `WHERE` com a chave primária para limitar o escopo. Para cada `UPDATE`, preencha o campo `alterado_por_id` com o `id_usuario` do responsável pela alteração. O campo `alterado_em` será atualizado automaticamente pelo banco via `ON UPDATE CURRENT_TIMESTAMP`.

```sql
-- Exemplo de padrão esperado:
UPDATE usuarios
SET    status          = 'inativa',
       alterado_por_id = 1        -- id_usuario do administrador que fez a alteração
WHERE  id_usuario      = 3;
```

### 2.3 Inserção com relacionamento e exclusão

Esta seção testa sua compreensão de integridade referencial e os dois tipos de exclusão de dados.

**Passo 1:** insira **2 matrículas** que referenciem alunos e disciplinas já cadastradas. Registre também ao menos uma nota para cada matrícula recém-inserida.

**Passo 2 — Exclusão física:** para a **primeira matrícula**, use `DELETE`. As notas associadas devem ser removidas automaticamente pela ação `ON DELETE CASCADE` definida na FK de `notas` para `matriculas`.

```sql
-- Exclusão física: o registro some permanentemente do banco
DELETE FROM matriculas WHERE id_matricula = ?;
```

**Passo 3 — Exclusão lógica:** para a **segunda matrícula**, preencha `deletado_em` com `NOW()` e registre o responsável em `alterado_por_id`. O registro permanece no banco e pode ser auditado.

```sql
-- Exclusão lógica: o registro permanece, deletado_em indica quando foi "removido"
UPDATE matriculas
SET    deletado_em     = NOW(),
       status          = 'trancado',
       alterado_por_id = 1
WHERE  id_matricula    = ?;
```

> 💡 **Diferença conceitual importante:** a exclusão física apaga os dados para sempre (sem possibilidade de recuperação sem backup). A exclusão lógica preserva o histórico — inclusive **quando** (`deletado_em`) e **quem** (`alterado_por_id`) realizou a operação. Em sistemas reais, a exclusão lógica é quase sempre preferida.

---

## Parte 3 — DQL: Consultas

Escreva as consultas a seguir. Requisitos que valem para **todas**:

- Use sempre **alias de tabela** em qualquer consulta com mais de uma tabela
- Use **exclusivamente `INNER JOIN`** explícito — sem `WHERE` com igualdade de FKs
- **Não use subconsultas** de nenhuma forma (nem no `SELECT`, nem no `FROM`, nem no `WHERE`)
- **Não use `CASE WHEN`**
- Filtre sempre `tabela.deletado_em IS NULL` para cada tabela envolvida, exibindo apenas registros ativos

---

### 🟢 Consultas Simples

**CS-1:** liste o nome completo, e-mail e status de todos os usuários **ativos**, ordenados pelo nome em ordem alfabética.

**CS-2:** liste o código e o nome de todas as disciplinas **ativas** que pertencem ao curso com `id_curso = 1`, ordenadas pela carga horária de forma decrescente.

---

### 🟡 Consultas Intermediárias

**CI-1:** liste o nome completo dos alunos junto com o nome do curso ao qual estão vinculados. Considere apenas alunos e usuários ativos. Ordene pelo nome do curso e depois pelo nome do aluno.

**CI-2:** para cada professor ativo, exiba o nome completo, a titulação e a **quantidade de disciplinas ativas** que ele leciona. Ordene do professor com mais disciplinas para o com menos. Justifique em comentário no script por que professores sem disciplinas não aparecem nesta consulta com `INNER JOIN`, e o que seria necessário para incluí-los.

---

### 🔴 Consultas Complexas

**CC-1 — Boletim por aluno:** para cada aluno ativo, exiba o nome completo do aluno, o nome da disciplina, o semestre da matrícula, o status da matrícula e a **média das notas** daquela matrícula, arredondada para 2 casas decimais com `ROUND`. Considere apenas matrículas ativas e com status diferente de `trancado`. Ordene pelo nome do aluno e pelo semestre.

> 💡 Você precisará unir: `usuarios`, `alunos`, `matriculas`, `disciplinas` e `notas`. Pense na ordem dos JOINs e em quais tabelas precisam do filtro `deletado_em IS NULL`.

**CC-2 — Resumo de desempenho por disciplina:** para cada disciplina ativa, exiba o nome da disciplina, o nome do curso ao qual pertence, o **total de matrículas ativas**, a **média geral das notas** de todas essas matrículas (arredondada com `ROUND` para 2 casas decimais) e a **maior nota individual** registrada (`MAX`). Exiba apenas disciplinas que tenham ao menos 3 matrículas ativas. Ordene pela média geral de forma decrescente.

> 💡 Esta consulta envolve: `cursos`, `disciplinas`, `matriculas` e `notas`. Agrupe por disciplina usando `GROUP BY` e filtre a quantidade mínima de matrículas com `HAVING`.

**CC-3 — Carga horária por professor e titulação:** para cada combinação de titulação e professor ativo, exiba a titulação, o nome completo do professor, a **quantidade de disciplinas ativas** que ele leciona e a **soma total da carga horária** dessas disciplinas. Filtre apenas professores cuja soma de carga horária seja superior a 60 horas. Ordene pela titulação e, dentro de cada titulação, pelo total de carga horária de forma decrescente.

> 💡 Esta consulta envolve apenas: `usuarios`, `professores` e `disciplinas`. Use `GROUP BY` com as colunas não-agregadas e `HAVING` para o filtro de carga total.

---

## 📐 Gabarito Parcial — Estrutura Esperada de Tabelas

Use como referência para validar sua modelagem. Os campos de auditoria estão listados separadamente pois se repetem em todas as tabelas.

**Colunas principais de cada tabela:**

```
usuarios      → id_usuario, nome, email, senha_hash, status
alunos        → id_aluno, usuario_id, curso_id, numero_matricula, data_ingresso, turno
professores   → id_professor, usuario_id, registro_funcional, titulacao, area_atuacao
funcionarios  → id_funcionario, usuario_id, matricula_funcional, setor
cursos        → id_curso, nome, carga_horaria_total, nivel
disciplinas   → id_disciplina, curso_id, professor_id, nome, codigo, carga_horaria, periodo
matriculas    → id_matricula, aluno_id, disciplina_id, semestre, status
notas         → id_nota, matricula_id, tipo, valor
```

**Campos de auditoria presentes em todas as tabelas:**

```
criado_em       TIMESTAMP NOT NULL  DEFAULT CURRENT_TIMESTAMP
alterado_em     TIMESTAMP NOT NULL  DEFAULT CURRENT_TIMESTAMP  ON UPDATE CURRENT_TIMESTAMP
deletado_em     TIMESTAMP NULL      DEFAULT NULL
alterado_por_id BIGINT UNSIGNED NULL  →  FK para usuarios (id_usuario)  [via ALTER TABLE]
```

> ⚠️ O gabarito mostra apenas os nomes das colunas. Você é responsável por definir os tipos de dados, constraints e ações de integridade referencial adequados para cada coluna.

---

## ✅ Checklist Final — antes de considerar o exercício concluído

- [ ] O script executa do zero sem erros no MariaDB/MySQL
- [ ] Todas as tabelas seguem as convenções de nomenclatura da disciplina
- [ ] Campos `criado_em`, `alterado_em` e `deletado_em` estão **inline** em todas as tabelas com os valores padrão corretos
- [ ] Campo `alterado_por_id` adicionado via `ALTER TABLE` ao final, com FK para `usuarios` em todas as tabelas
- [ ] Ao menos 3 registros inseridos em cada tabela
- [ ] 1 `UPDATE` por tabela, com `alterado_por_id` preenchido com o `id_usuario` correto
- [ ] 2 matrículas inseridas com notas; exclusão física (`DELETE`) na primeira e exclusão lógica (`deletado_em = NOW()`) na segunda
- [ ] Todas as 7 consultas retornam resultados sem erros
- [ ] Consultas filtram `deletado_em IS NULL` em todas as tabelas envolvidas
- [ ] Consultas complexas usam `COUNT`, `SUM`, `AVG`, `MAX`, `ROUND`, `GROUP BY` e `HAVING`
- [ ] Nenhuma consulta usa `CASE WHEN`
- [ ] Nenhuma consulta usa subconsultas
- [ ] Todas as consultas com múltiplas tabelas usam alias e `INNER JOIN` explícito

---

## 📚 Referências para Revisão

| Tópico | Aula de Referência |
|---|---|
| Convenções de nomenclatura (Regras 1–7) | Aula 03 — Seção 1 |
| CREATE TABLE, tipos de dados, constraints | Aula 03 — Seções 4, 5, 7 |
| ALTER TABLE (coluna + FK) | Aula 03 — Seção 8 |
| INSERT, UPDATE, DELETE | Aula 04 |
| Exclusão lógica com `deletado_em` | Aula 04 — Seção 3 |
| SELECT, WHERE, ORDER BY | Aula 06 |
| GROUP BY, HAVING, funções de agregação | Aula 07 |
| INNER JOIN | Aula 08 |

---

> 📝 **Boa revisão!** Lembre-se: o aprendizado de banco de dados acontece na prática. Quanto mais você digitar o código, mais natural ele se tornará.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
