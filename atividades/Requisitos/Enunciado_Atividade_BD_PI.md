# Atividade Avaliativa: Modelagem e Implementação Física do Banco de Dados do Projeto Integrador

**Disciplina**: IBD015 (Banco de Dados Relacional)
**Professor**: Ronan Adriel Zenatti
**Modalidade**: Individual (cada aluno entrega sua própria solução)
**SGBD alvo**: MariaDB 10.4 (XAMPP)
**Entrega**: arquivo `.sql` único, via Google Classroom

---

## 1. Contexto

Cada grupo de Projeto Integrador (PI) produziu um levantamento de requisitos para um sistema. Esses levantamentos, em geral, apresentam **lacunas, ambiguidades e inconsistências**, o que é natural em um documento de fase inicial. Para esta atividade, você receberá uma **versão refinada** do levantamento do seu grupo, onde:

- Corrigi inconsistências evidentes.
- Completei lacunas de informação.
- Sinalizei explicitamente os **dilemas de modelagem** que você deverá decidir.

A partir desse documento, **cada aluno do grupo, individualmente**, deverá projetar e implementar o banco de dados relacional do projeto, tomando decisões fundamentadas sobre os pontos em aberto.

Selecione o seu projeto e acesse o respectivo arquivo de requisitos:

| Projeto | Arquivo de requisitos |
|---------|------------------------|
| CMPCD | [Requisitos_CMPCD.md](Requisitos_CMPCD.md) |
| Controla$eu | [Requisitos_ControlaSeu.md](Requisitos_ControlaSeu.md) |
| Singularys | [Requisitos_Singularys.md](Requisitos_Singularys.md) |

---

## 2. Objetivos de Aprendizagem

- Interpretar um levantamento de requisitos e traduzi-lo em um modelo relacional.
- Tomar decisões de modelagem fundamentadas (generalização/especialização, unificação de entidades semelhantes, inclusão ou exclusão de entidades opcionais).
- Aplicar convenções técnicas consistentes na implementação física do banco.
- Justificar tecnicamente as decisões tomadas.

---

## 3. Escopo da Entrega

O arquivo `.sql` entregue deve conter **exclusivamente DDL** (`DROP`,`CREATE `), nessa ordem:

1. Instrução de limpeza de banco de dados e tabelas que evitem erros de execução.
2. Instruções de criação de tabelas com garantias de integridade referencial, que evitem erros de execução, na ordem correta de dependência (tabelas referenciadas antes das que referenciam).
3. Comentários explicativos das decisões tomadas (ver seção 5).

---

## 4. Convenções Técnicas

O arquivo deve seguir as convenções definidas em aula e presentes no Github de conteudo do curso.

**Lembre-se de usar os campos de auditoria obrigatórios (`criado_em`, `atualizado_em`) nas tabelas!**

### 4.1 Decisão Opcional

- **Soft-delete** (exclusão lógica via campo `deletado_em` ou similar): Você decide se irá utilizar em seu projeto. Podendo ser aplicado em qualquer tabela, em todas, ou em nenhuma. Caso utilize, justifique no bloco de decisões.

--- 

## 5. Decisões de Modelagem (Análise Individual)

**Os dilemas de modelagem presentes no documento de requisitos do seu grupo devem ser decididos por você** com base no levantamento refinado do seu grupo. Para cada um, registre sua decisão e justificativa em **comentários no arquivo SQL**, no início do arquivo, em um bloco de comentário multilinha (`/* ... */`).

---

## 6. Critérios de Avaliação (2,0 pontos)

| Critério | Peso |
|----------|:----:|
| **Aderência às convenções obrigatórias** (nomes, tipos, charset e auditoria). Convenções recomendadas não pontuam, mas violações reduzem a nota neste critério. | 0,6 |
| **Cobertura dos requisitos funcionais** (todas as entidades necessárias estão presentes e completas). | 0,6 |
| **Correção da modelagem** (relacionamentos coerentes, cardinalidades corretas, integridade referencial via FK). | 0,6 |
| **Coerência das decisões com a justificativa** (decisões registradas no bloco de comentário batem com o que foi implementado no SQL). | 0,2 |

---

## 7. Entrega

- **Formato**: arquivo único `.sql` com codificação UTF-8.
- **Nomenclatura do arquivo**: `NomeCompleto_PI_BD.sql` (ex.: `JoaoSilvaSantos_PI_BD.sql`).

---

## 8. Material de Consulta Permitido

Durante a elaboração da atividade, **a única fonte de consulta permitida** é o repositório oficial da disciplina:

🔗 **https://github.com/DSM-Fatec-Jahu/Banco-de-Dados-Relacional-IBD015/**

O repositório contém todo o material das aulas, exemplos de DDL, convenções da disciplina e funções do MariaDB necessárias para a realização da atividade. Nenhuma outra fonte externa (sites, fóruns, IA generativa, etc.) deve ser utilizada.

---

## 9. Recomendações Práticas

1. **Leia o levantamento refinado mais de uma vez** antes de começar a modelar. Identifique todas as entidades, atributos e relacionamentos antes de escrever uma única linha de SQL.

2. **Esboce um DER antes do DDL**. Mesmo que não seja entregue, modelar em papel ou em ferramenta visual (dbdiagram.io, MySQL Workbench) reduz drasticamente erros de relacionamento.

3. **Teste no XAMPP/MariaDB 10.4 antes de entregar**. Crie um banco zerado, execute seu script inteiro e verifique se todas as tabelas foram criadas sem erros.

4. **Justifique de verdade**. As justificativas devem refletir o que foi efetivamente implementado no SQL. Decisões registradas que não correspondem ao código entregue reduzem a nota no critério de coerência.

---

*Atividade da disciplina IBD015 (Banco de Dados Relacional), Fatec Jahu, Prof. Ronan Adriel Zenatti.*
