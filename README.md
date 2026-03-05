# 🗄️ Banco de Dados — Relacional (IBD015)

> **Fatec Jahu** · Tecnologia em Desenvolvimento de Software Multiplataforma · 1º Semestre/2026

---

## 👨‍🏫 Informações do Professor

| Campo | Informação |
|---|---|
| **Professor** | Ronan Adriel Zenatti |
| **E-mail** | ronan.zenatti@cps.sp.gov.br |
| **Instituição** | Fatec Jahu — Centro Paula Souza |
| **Curso** | Tecnologia em Desenvolvimento de Software Multiplataforma |
| **Disciplina** | Banco de Dados — Relacional |
| **Sigla** | IBD015 |
| **Semestre** | 1º Semestre / 2026 |

---

## 📋 Ementa

Esta disciplina abrange o projeto e a implementação de bancos de dados relacionais, desde a modelagem conceitual até aspectos avançados de programação e administração. Os principais temas trabalhados são:

Projeto e implementação de banco de dados relacionais; consultas complexas com agrupamentos e subconsultas; implementação de restrições de integridade; criação de consultas utilizando visões; aspectos de programação em ambiente de banco de dados com procedimentos armazenados, gatilhos e funções; cópia de segurança e restauração de bancos de dados; estruturas de índices; processamento e otimização de consultas; processamento de transações e controle de concorrência; recuperação de falhas; e novas tecnologias aplicadas a banco de dados.

---

## 🎯 Objetivos

Ao final da disciplina, o aluno será capaz de:

- Aplicar normalização para implementação de Banco de Dados, utilizando adequadamente os conceitos de linguagem de definição, manipulação e consulta de dados (DDL, DML e DQL);
- Implementar *Stored Procedures* e Gatilhos (*Triggers*) para soluções de problemas em sistemas;
- Identificar as características de recuperação após falha e de segurança dos SGBDs.

---

## ⏱️ Carga Horária

| Tipo | Horas |
|---|---|
| Carga Horária Semanal | 4 horas |
| **Carga Horária Semestral** | **80 horas** |

---

## 📐 Metodologia

As aulas são conduzidas no formato **expositivo e prático**, combinando explicações conceituais com exercícios aplicados diretamente no SGBD. A ênfase é sempre na resolução de problemas reais, preparando o aluno para o mercado de trabalho.

---

## 📊 Critérios de Avaliação

A nota final é calculada pela seguinte fórmula:

```
Nota Final = (T1 + P1 + T2 + P2) × 1 + R
```

| Componente | Descrição |
|---|---|
| **T1** | Modelagem de um sistema de E-commerce (Conceitual, Lógico e DDL) |
| **P1** | Avaliação individual teórica e prática sobre modelagem e SQL fundamental |
| **T2** | Desenvolvimento de relatórios complexos e *views* para tomada de decisão |
| **P2** | Avaliação individual sobre consultas avançadas e programação em banco de dados |
| **R** | Avaliação Substitutiva — substitui a menor nota entre P1 e P2 |

> 💡 **Dica:** O Trabalho 2 (T2 — Aula 17) é interdisciplinar e integrado com as disciplinas de **Desenvolvimento Web II** e **Engenharia de Software II**. Planeje-se com antecedência!

---

## 📚 Sumário de Aulas

Cada aula está organizada na pasta [`aulas/`](./aulas/) com conteúdo teórico, exemplos de código SQL e exercícios práticos.

### Bloco 1 — Fundamentos e Modelagem

| Aula | Título | Conteúdo Principal |
|---|---|---|
| [01](./aulas/Aula_01_Revisao_Modelagem_Conceitual.md) | Revisão de Modelagem de Dados (Conceitual) | Abordagem Entidade-Relacionamento (MER); Entidades, Atributos e Relacionamentos |
| [02](./aulas/Aula_02_Normalizacao.md) | Normalização | Dependências funcionais; 1ª, 2ª e 3ª Formas Normais; modelo conceitual ao lógico relacional |
| [03](./aulas/Aula_03_SQL_DDL.md) | SQL — DDL: Definição de Estruturas | Comandos DDL (CREATE, ALTER, DROP); Tipos de dados; Restrições básicas (PK) |
| [04](./aulas/Aula_04_SQL_DML.md) | SQL — DML: Manipulação de Dados | Comandos DML (INSERT, UPDATE, DELETE); Controle de transação básico |
| [05](./aulas/Aula_05_Atividade_Modelagem.md) | Atividade Prática — Modelagem E-commerce | Modelagem completa de um sistema de E-commerce (Conceitual, Lógico e DDL) |

### Bloco 2 — Consultas e Visões

| Aula | Título | Conteúdo Principal |
|---|---|---|
| [06](./aulas/Aula_06_SQL_Consultas_Basicas.md) | SQL: Consultas Básicas | SELECT; WHERE; Operadores lógicos e relacionais; ORDER BY |
| [07](./aulas/Aula_07_SQL_DQL_Agregacao.md) | SQL — DQL: Consultas e Agregação | Filtros avançados (LIKE, BETWEEN, IN); Funções de agregação (COUNT, SUM, AVG, MIN, MAX); GROUP BY e HAVING |
| [08](./aulas/Aula_08_Joins_Subconsultas_Views.md) | Junções (JOINs), Subconsultas e Visões | INNER JOIN, LEFT JOIN, RIGHT JOIN; Subqueries; Criação e uso de VIEW |
| [09](./aulas/Aula_09_Avaliacao_P1.md) | ✏️ Avaliação P1 | Avaliação individual — Modelagem e SQL fundamental |

### Bloco 3 — Programação e Administração de BD

| Aula | Título | Conteúdo Principal |
|---|---|---|
| [10](./aulas/Aula_10_Integridade_Referencial.md) | Integridade Referencial e Restrições | FOREIGN KEY com ON DELETE/UPDATE CASCADE e RESTRICT; CHECK; Índices básicos |
| [11](./aulas/Aula_11_Stored_Procedures.md) | Stored Procedures | Lógica procedural; PROCEDURE com parâmetros IN/OUT; Tratamento de erros |
| [12](./aulas/Aula_12_Triggers.md) | Triggers (Gatilhos) | Gatilhos BEFORE/AFTER (ROW e STATEMENT); Auditoria de dados; Boas práticas |
| [13](./aulas/Aula_13_Functions_Transacoes.md) | Functions (UDF) e Transações | UDFs escalares e de tabela; diferença entre FUNCTION e PROCEDURE; ACID; BEGIN, COMMIT, ROLLBACK; Controle de concorrência |
| [14](./aulas/Aula_14_Backup_Seguranca.md) | Backup, Restauração e Segurança | Backup completo, incremental e diferencial; DUMP e restauração; CREATE USER, GRANT, REVOKE |
| [15](./aulas/Aula_15_Otimizacao_Indices.md) | Otimização de Consultas e Índices | Estruturas B-Tree e Hash; EXPLAIN / EXPLAIN ANALYZE; Otimização de JOINs e subconsultas |

### Bloco 4 — Encerramento e Avaliações Finais

| Aula | Título | Conteúdo Principal |
|---|---|---|
| [16](./aulas/Aula_16_Revisao_Geral.md) | Revisão Geral | Consolidação de todos os tópicos; Exercícios integradores |
| [17](./aulas/Aula_17_Trabalho_Interdisciplinar_T2.md) | 📦 T2 — Trabalho Interdisciplinar | Entrega do Projeto Integrado com Desenvolvimento Web II e Engenharia de Software II |
| [18](./aulas/Aula_18_Avaliacao_P2.md) | ✏️ Avaliação P2 | Avaliação individual — Consultas avançadas, Procedures, Triggers, Transações, Backup e Otimização |
| [19](./aulas/Aula_19_Avaliacao_Substitutiva.md) | ✏️ Avaliação Substitutiva | Substitui a menor nota entre P1 e P2 |
| [20](./aulas/Aula_20_Novas_Tecnologias_Encerramento.md) | Novas Tecnologias em BD e Encerramento | NewSQL, DBaaS, Big Data, introdução a BD Não Relacional; Fechamento e portfólio |

---

## 📝 Atividades

Todas as atividades avaliativas e exercícios práticos estão organizados na pasta [`atividades/`](./atividades/).

| # | Atividade | Descrição | Peso |
|---|---|---|---|
| T1 | [Modelagem E-commerce](./atividades/T1_Modelagem_Ecommerce.md) | Modelagem Conceitual, Lógica e DDL de um sistema de E-commerce | 2 pts |
| P1 | [Avaliação P1](./atividades/P1_Avaliacao_Modelagem_SQL.md) | Prova individual sobre modelagem e SQL fundamental | 3 pts |
| T2 | [Projeto Interdisciplinar](./atividades/T2_Projeto_Interdisciplinar.md) | Sistema BD integrado ao projeto de Desenvolvimento Web II e Engenharia de Software II | 2 pts |
| P2 | [Avaliação P2](./atividades/P2_Avaliacao_Avancada.md) | Prova individual sobre consultas avançadas e programação em BD | 3 pts |
| R | [Avaliação Substitutiva](./atividades/R_Avaliacao_Substitutiva.md) | Substitui a menor nota entre P1 e P2 | 3 pts** |

---

## 📖 Bibliografia

### Básica

- DATE, C. J. *Introdução a sistemas de bancos de dados*. Rio de Janeiro: Elsevier/Campus, 2004.
- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. São Paulo: Pearson, 2018.
- SILBERSCHATZ, A.; SUNDARSHAN, S.; KORTH, H. F. *Sistema de banco de dados*. Rio de Janeiro: Elsevier Brasil, 2016.

### Complementar

- BEAULIEU, A. *Aprendendo SQL*. São Paulo: Novatec, 2010.
- GILLENSON, M. L. *Fundamentos de Sistemas de Gerência de Banco de Dados*. Rio de Janeiro: LTC, 2006.
- MACHADO, F. N. R. *Banco de Dados: Projeto e Implementação*. São Paulo: Érica, 2005.
- RAMAKRISHNAN, R.; GEHRKE, J. *Sistemas de Gerenciamento de Bancos de Dados*. 3 ed. Porto Alegre: Bookman, 2008.
- ROB, P.; CORONEL, C. *Sistemas de Banco de Dados: Projeto, Implementação e Gerenciamento*. 8 ed. São Paulo: Cengage Learning, 2011.
- TEOREY, T.; LIGHTSTONE, S.; NADEAU, T. *Projeto e Modelagem de Bancos de Dados*. São Paulo: Campus, 2006.

### Referência

- DEBARROS, A. *SQL Prático: um guia para iniciantes*. 2 ed. São Paulo: Novatec, 2022.
- TANIMURA, C. *SQL para Análise de Dados: técnicas avançadas para transformar dados em insights*. São Paulo: Novatec, 2022.
- FORTA, B. *SQL em 10 Minutos por Dia*. 5 ed. São Paulo: Novatec, 2021.

---

## 🗂️ Estrutura do Repositório

```
📦 IBD015-Banco-de-Dados-Relacional/
│
├── 📄 README.md                  ← Você está aqui
│
├── 📁 aulas/
│   ├── Aula_01_Revisao_Modelagem_Conceitual.md
│   ├── Aula_02_Normalizacao.md
│   ├── Aula_03_SQL_DDL.md
│   ├── Aula_04_SQL_DML.md
│   ├── Aula_05_Atividade_Modelagem.md
│   ├── Aula_06_SQL_Consultas_Basicas.md
│   ├── Aula_07_SQL_DQL_Agregacao.md
│   ├── Aula_08_Joins_Subconsultas_Views.md
│   ├── Aula_09_Avaliacao_P1.md
│   ├── Aula_10_Integridade_Referencial.md
│   ├── Aula_11_Stored_Procedures.md
│   ├── Aula_12_Triggers.md
│   ├── Aula_13_Functions_Transacoes.md
│   ├── Aula_14_Backup_Seguranca.md
│   ├── Aula_15_Otimizacao_Indices.md
│   ├── Aula_16_Revisao_Geral.md
│   ├── Aula_17_Trabalho_Interdisciplinar_T2.md
│   ├── Aula_18_Avaliacao_P2.md
│   ├── Aula_19_Avaliacao_Substitutiva.md
│   └── Aula_20_Novas_Tecnologias_Encerramento.md
│
└── 📁 atividades/
    ├── T1_Modelagem_Ecommerce.md
    ├── P1_Avaliacao_Modelagem_SQL.md
    ├── T2_Projeto_Interdisciplinar.md
    ├── P2_Avaliacao_Avancada.md
    └── R_Avaliacao_Substitutiva.md
```

---

## 💬 Contato e Dúvidas

Dúvidas sobre o conteúdo, atividades ou avaliações? Entre em contato:

📧 **ronan.zenatti@cps.sp.gov.br**

> *"Dados bem modelados são a fundação de qualquer sistema confiável."*

---

<div align="center">
  <sub>Fatec Jahu · Centro Paula Souza · Governo do Estado de São Paulo · 2026</sub>
</div>
