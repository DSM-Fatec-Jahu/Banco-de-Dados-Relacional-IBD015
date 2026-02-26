# Aula 18 — Avaliação P2

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 17](./Aula_17_Trabalho_Interdisciplinar_T2.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_19_Avaliacao_Substitutiva.md)

---

## 📋 Informações da Avaliação

| Item | Descrição |
|---|---|
| **Tipo** | Avaliação individual teórica e prática |
| **Conteúdo** | Aulas 10 a 15: Integridade Referencial, Procedures, Triggers, Functions, Transações, Backup, Otimização |
| **Duração** | 2 horas |
| **Recursos** | Acesso ao MariaDB — sem consulta a anotações ou internet |

---

## 📚 Conteúdo Cobrado

- Integridade referencial: ações ON DELETE / ON UPDATE e seus efeitos
- Índices: criação, tipos (B-Tree), impacto em queries
- Stored Procedures: parâmetros IN/OUT, variáveis, condicionais, loops, tratamento de erro
- Triggers: BEFORE/AFTER, OLD/NEW, auditoria, validação
- Functions: criação, RETURN, uso em SELECT, diferença com PROCEDURE
- Transações: ACID, BEGIN/COMMIT/ROLLBACK, níveis de isolamento
- Backup: mysqldump com opções corretas, restauração
- Segurança: CREATE USER, GRANT, REVOKE, mínimo privilégio
- Otimização: EXPLAIN, identificação de problemas, reescrita de queries

---

## 🗂️ Roteiro de Revisão

- [ ] Sei identificar qual ação ON DELETE usar para cada cenário de negócio?
- [ ] Sei criar índices compostos e entendo a regra do prefixo?
- [ ] Sei criar uma procedure com parâmetros IN/OUT, transação e tratamento de erro?
- [ ] Sei criar triggers BEFORE e AFTER com OLD e NEW corretamente?
- [ ] Sei criar uma function e usá-la dentro de um SELECT?
- [ ] Entendo as propriedades ACID e o que cada uma garante?
- [ ] Sei interpretar a saída do EXPLAIN e identificar problemas?
- [ ] Sei fazer backup com mysqldump com as opções corretas?
- [ ] Sei criar usuários com o mínimo de privilégios necessários?

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
