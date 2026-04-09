# Avaliação P1 — Banco de Dados Relacional

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu — Centro Paula Souza
> Prof. Ronan Adriel Zenatti · ronan.zenatti@cps.sp.gov.br
> Desenvolvimento de Software Multiplataforma · 1º Semestre / 2026

---

## Instruções Gerais

- Valor total: **3,0 pontos**
- Entregue **um único arquivo `.sql`** nomeado no padrão `RA_NomeCompleto_P1.sql` — exemplo: `2300042_MariaOliveira_P1.sql`
- O arquivo deve executar do início ao fim **sem erros** em um banco MariaDB/MySQL limpo
- A entrega deve ser feita pela atividade criada no Google Classroom da turma
- Não envie por e-mail nem compacte o arquivo

---

## Contexto do Sistema

Você foi contratado como analista de sistemas pela **Funilaria do Célio**, uma oficina de funilaria e pintura automotiva localizada em Jahu. O Seu Célio quer informatizar o negócio, e após uma reunião de levantamento você documentou os seguintes requisitos:

### Requisitos Funcionais

- **RF-01:** O sistema deve permitir o cadastro de clientes com nome completo, CPF e telefone.
- **RF-02:** Um cliente pode possuir vários veículos cadastrados. Cada veículo possui placa (única no sistema), marca, modelo e ano de fabricação. Um veículo pertence a exatamente um cliente.
- **RF-03:** O sistema deve gerenciar funcionários da oficina. Todos os funcionários possuem nome completo, CPF e data de contratação. Existem dois tipos de funcionário: **funileiros** e **pintores**. O funileiro possui uma certificação técnica obtida no SENAI. O pintor possui uma especialidade (automotivo, industrial ou geral). Todo funcionário é de um desses dois tipos — não existe funcionário sem tipo definido.
- **RF-04:** O sistema deve registrar Ordens de Serviço (OS). Cada OS está vinculada a um veículo, possui descrição do problema relatado, data de entrada, data de saída (preenchida ao concluir) e status (Aberta, Em Andamento, Aguardando Peça, Concluída ou Cancelada).
- **RF-05:** Cada OS deve registrar dois vínculos distintos com funcionários: o funcionário que **abriu** a OS no sistema e o funcionário **responsável** pela execução do serviço. Esses dois podem ou não ser a mesma pessoa — ambos devem ser registrados explicitamente.
- **RF-06:** Uma OS pode conter vários serviços. Os tipos de serviço possuem nome e preço padrão. Ao incluir um serviço em uma OS, o sistema deve registrar o **valor efetivamente cobrado**, pois pode haver desconto em relação ao preço padrão.

### Requisitos Não Funcionais

- **RNF-01:** CPF de clientes e funcionários deve ser único no sistema.
- **RNF-02:** Placa de veículo deve ser única no sistema.
- **RNF-03:** O status da OS deve ser restrito aos valores definidos no RF-04.
- **RNF-04:** A especialidade do pintor deve ser restrita aos valores definidos no RF-03.
- **RNF-05:** O sistema deve suportar exclusão lógica de registros: toda tabela deve possuir os campos `criado_em`, `alterado_em` e `deletado_em`. Consultas que exibem dados ativos devem filtrar `deletado_em IS NULL`.

---

## Parte 1 — DDL: Criação do Banco de Dados

Crie o banco de dados `funilaria_celio` e todas as tabelas necessárias para atender aos requisitos acima, seguindo rigorosamente as convenções de nomenclatura da disciplina. Garanta que o script execute de forma limpa do início ao fim.

### 1.1 Ajuste tardio nos Requisitos (ALTER TABLE)

- **RNF-06:** Toda tabela deve registrar qual usuário do sistema realizou a última modificação em cada registro, por meio de um campo `alterado_por_id` que referencia a tabela `usuarios`.

---

## Parte 2 — DML: Manipulação de Dados

### 2.1 Inserção

Insira ao menos **3 registros** em cada tabela. Use `INSERT` com lista de colunas explícita.

Os dados devem ser **coerentes com o contexto** da funilaria: veículos vinculados a clientes reais cadastrados, OS vinculadas a veículos existentes, itens de OS referenciando tipos de serviço válidos. Certifique-se de que os dados permitam que todas as consultas da Parte 3 retornem ao menos um resultado.

### 2.2 Atualização

Execute ao menos **1 `UPDATE` por tabela**, preenchendo sempre o campo `alterado_por_id` com o `id_usuario` correspondente ao usuário que realizou a operação.

### 2.3 Exclusão física e lógica

Escolha duas OS que tenham itens registrados e execute os dois procedimentos abaixo:

**Passo 1 — Exclusão física:** para a primeira OS escolhida, remova-a com `DELETE`. Justifique em comentário no script qual ação referencial foi configurada na FK de `itens_os` para `ordens_de_servico` e por que ela garante a integridade dos dados nessa operação.

**Passo 2 — Exclusão lógica:** para a segunda OS escolhida, preencha `deletado_em`, atualize o status para `'Cancelada'` e registre o responsável em `alterado_por_id`.

---

## Parte 3 — DQL: Consultas

---

### 🟢 Consultas Simples

**CS-1:** liste o nome completo e o telefone de todos os clientes ativos, ordenados pelo nome em ordem alfabética.

**CS-2:** liste a placa, a marca, o modelo e o ano de fabricação de todos os veículos ativos, ordenados pelo ano de fabricação de forma decrescente.

---

### 🟡 Consultas Intermediárias

**CI-1:** liste o nome completo de cada cliente junto com a placa, marca e modelo de cada veículo ativo que ele possui. Considere apenas clientes ativos. Ordene pelo nome do cliente e depois pela placa do veículo.

**CI-2:** para cada funcionário ativo, exiba o nome completo e a **quantidade de Ordens de Serviço ativas** das quais ele é o responsável pela execução. Ordene do funcionário com mais OS para o com menos. Justifique em comentário no script por que funcionários sem OS não aparecem nesta consulta, e o que seria necessário para incluí-los.

---

### 🔴 Consultas Complexas

**CC-1 — Resumo de OS por veículo:** para cada veículo ativo, exiba a placa, a marca, o modelo, o nome do cliente proprietário e a **quantidade de OS ativas** vinculadas a esse veículo. Exiba apenas veículos com ao menos 1 OS ativa. Ordene pela quantidade de OS de forma decrescente.

**CC-2 — Faturamento por tipo de serviço:** para cada tipo de serviço ativo, exiba o nome do serviço, o preço padrão, a **quantidade de vezes que foi incluído em OS ativas**, o **total efetivamente faturado** (soma dos valores cobrados) e a **média do valor cobrado** com 2 casas decimais. Exiba apenas serviços que tenham sido utilizados ao menos 2 vezes. Ordene pelo total faturado de forma decrescente.

**CC-3 — Desempenho de funileiros:** para cada funileiro ativo, exiba o nome completo, a certificação técnica, a **quantidade de OS ativas** sob sua responsabilidade de execução, o **valor total cobrado** nos itens dessas OS e a **média do valor cobrado** arredondada com `ROUND` para 2 casas decimais. Filtre apenas funileiros cuja soma de valores cobrados seja superior a R$ 500,00. Ordene pelo valor total de forma decrescente.

---

## ✅ Checklist Final

- [ ] O script executa do zero sem erros no MariaDB/MySQL
- [ ] Todas as tabelas seguem as convenções de nomenclatura da disciplina
- [ ] Campos `criado_em`, `alterado_em` e `deletado_em` em todas as tabelas com os valores padrão corretos
- [ ] Campo `alterado_por_id` adicionado via `ALTER TABLE` ao final, com FK para `usuarios` em todas as tabelas
- [ ] Os dois vínculos de funcionário na OS possuem nomes semânticos distintos
- [ ] Ao menos 3 registros inseridos em cada tabela com dados coerentes com o contexto
- [ ] 1 `UPDATE` por tabela com `alterado_por_id` preenchido
- [ ] Exclusão física com comentário justificando a ação referencial configurada
- [ ] Exclusão lógica com `deletado_em`, status atualizado e `alterado_por_id` preenchido
- [ ] Todas as 7 consultas retornam resultados sem erros
- [ ] Consultas filtram `deletado_em IS NULL` em todas as tabelas envolvidas
- [ ] Consultas complexas usam `COUNT`, `SUM`, `AVG`, `ROUND`, `GROUP BY` e `HAVING`
- [ ] Nenhuma consulta usa `CASE WHEN`
- [ ] Nenhuma consulta usa subconsultas
- [ ] Todas as consultas com múltiplas tabelas usam alias e `INNER JOIN` explícito

---

## 📐 Critérios de Avaliação

| Critério | Descrição | Pontos |
|---|---|---|
| **DDL — Estrutura e modelagem** | Banco criado com charset e collation corretos; tabelas com PKs, FKs nomeadas, generalização e especialização, N:M e campos de auditoria corretos; `alterado_por_id` via `ALTER TABLE`; convenções de nomenclatura seguidas rigorosamente | 1,20 |
| **DML — Manipulação de dados** | Mínimo de 3 registros por tabela com dados coerentes e na ordem correta de dependência; `UPDATE` com `alterado_por_id` em todas as tabelas; exclusão física e lógica realizadas corretamente com comentário justificando a ação referencial | 0,80 |
| **DQL — Consultas** | Todas as 7 consultas executam sem erros, retornam resultados, usam alias e `INNER JOIN` explícito, filtram `deletado_em IS NULL` em todas as tabelas, e as consultas complexas utilizam corretamente `GROUP BY`, `HAVING` e funções de agregação | 1,00 |
| **Descontos automáticos** | Arquivo com nome fora do padrão: −0,10 · Arquivo que não executa sem erros do início ao fim: −0,50 | — |
| **Total** | | **3,00** |

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
