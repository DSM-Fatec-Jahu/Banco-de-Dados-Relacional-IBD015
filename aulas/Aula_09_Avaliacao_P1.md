# Aula 09 — Avaliação P1

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 08](./Aula_08_Joins_Subconsultas_Views.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_10_Integridade_Referencial.md)

---

## 📋 Informações da Avaliação

| Item | Descrição |
|---|---|
| **Tipo** | Avaliação individual teórica e prática |
| **Conteúdo** | Aulas 01 a 08: Modelagem, Normalização, DDL, DML e DQL fundamental |
| **Duração** | 2 horas |
| **Recursos** | Consulta ao MariaDB via terminal/ferramenta — sem consulta a anotações ou internet |

---

## 📚 Conteúdo Cobrado

- Modelagem conceitual: entidades, atributos, relacionamentos e cardinalidades
- Normalização: 1FN, 2FN, 3FN e dependências funcionais
- DDL: `CREATE DATABASE`, `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, constraints
- DML: `INSERT`, `UPDATE`, `DELETE`, controle de transações
- DQL: `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, `GROUP BY`, `HAVING`, `JOIN`, subconsultas e Views
- Convenções de nomenclatura da disciplina

---

## 🗂️ Roteiro de Revisão

Use este roteiro para guiar seus estudos antes da avaliação.

### Modelagem e Normalização

- [ ] Consigo identificar entidades, atributos e relacionamentos a partir de um texto de regras de negócio?
- [ ] Consigo determinar a cardinalidade (1:1, 1:N, N:M) de qualquer relacionamento fazendo as perguntas corretas?
- [ ] Sei identificar violações de 1FN, 2FN e 3FN numa tabela?
- [ ] Consigo normalizar uma tabela até a 3FN passo a passo?
- [ ] Sei converter um MER em modelo lógico (PKs, FKs, tabelas intermediárias)?

### DDL

- [ ] Sei criar um banco de dados com charset e collation corretos?
- [ ] Sei criar tabelas com todos os tipos de dados apropriados?
- [ ] Sei definir PK simples, PK composta, FK com ON DELETE/UPDATE, UNIQUE e CHECK?
- [ ] Sei usar `ALTER TABLE` para adicionar, modificar e remover colunas e constraints?
- [ ] Sei seguir todas as convenções de nomenclatura da disciplina?

### DML e DQL

- [ ] Sei usar INSERT com lista de colunas e INSERT múltiplo?
- [ ] Sei usar UPDATE e DELETE com WHERE adequado?
- [ ] Sei usar `BEGIN`, `COMMIT` e `ROLLBACK`?
- [ ] Sei escrever SELECT com WHERE, ORDER BY, LIMIT?
- [ ] Sei usar GROUP BY + HAVING e entendo a diferença com WHERE?
- [ ] Sei fazer INNER JOIN, LEFT JOIN e RIGHT JOIN?
- [ ] Sei escrever subconsultas escalares e tabelares?
- [ ] Sei criar e usar Views?

---

## 💡 Dicas para a Avaliação

Leia o enunciado completo antes de começar. Identifique quais regras de negócio definem cada cardinalidade. Comece sempre pelo modelo — um DDL correto só é possível com um modelo correto. Nomeie todas as constraints explicitamente. Execute e teste seu script antes de entregar.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
