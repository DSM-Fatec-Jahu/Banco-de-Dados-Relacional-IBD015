# Projeto Controla$EU: Levantamento de Requisitos

## 1. Visão Geral do Sistema

O **Controla$EU** é uma plataforma web de controle financeiro pessoal e empresarial, voltada principalmente para jovens da Geração Z e pequenas empresas. O sistema permite que usuários (pessoas físicas ou jurídicas) cadastrem suas receitas, despesas, orçamentos e metas financeiras, recebendo notificações automáticas e visualizando o histórico de movimentações. A plataforma opera por **assinatura mensal de planos**, com diferentes níveis de benefícios.

### 1.1 Domínios do Sistema

O sistema possui **dois domínios distintos** que devem ser claramente separados na modelagem:

1. **Domínio do Negócio Controla$EU**: planos comerciais oferecidos pela plataforma, assinaturas pagas pelos usuários, pagamentos de assinatura. Trata da relação comercial entre a empresa Controla$EU e seus clientes.

2. **Domínio Financeiro do Usuário**: receitas, despesas, categorias, orçamentos, metas e notificações do próprio usuário. Trata do controle financeiro pessoal/empresarial que o usuário gerencia dentro da plataforma.

---

## 2. Atores do Sistema

| Ator | Descrição |
|------|-----------|
| **Usuário Pessoa Física (PF)** | Indivíduo que utiliza o sistema para controle financeiro pessoal. Identificado por CPF. |
| **Usuário Pessoa Jurídica (PJ)** | Empresa que utiliza o sistema para controle financeiro corporativo. Identificada por CNPJ. |
| **Administrador da plataforma** | Responsável pela gestão dos planos comerciais e suporte. |

---

## 3. Requisitos Funcionais (RF)

### 3.1 Cadastro e Acesso

**RF01. Cadastro de Usuário**
O sistema deve permitir o cadastro de usuários, que podem ser **Pessoa Física** ou **Pessoa Jurídica**.

- Atributos comuns: nome, e-mail, telefone, senha.
- Atributos exclusivos de PF: CPF, data de nascimento.
- Atributos exclusivos de PJ: CNPJ, razão social, data de fundação.
- Um usuário é exclusivamente PF **ou** PJ, nunca ambos simultaneamente.
- E-mail, CPF e CNPJ devem ser únicos no sistema.

> **Dilema de modelagem #1 (Generalização/Especialização)**: como representar PF e PJ no banco?
> Possíveis abordagens: (a) tabela única com campos opcionais, (b) tabela genérica com FK para entidades específicas, (c) duas tabelas independentes. Você deve decidir e justificar, considerando integridade, normalização e flexibilidade.

**RF02. Autenticação (Login)**
O sistema deve permitir login do usuário utilizando CPF ou CNPJ (conforme o tipo) e senha. A senha deve estar armazenada de forma segura (hash).

### 3.2 Domínio Comercial (Planos e Assinaturas)

**RF03. Planos de Assinatura**
A plataforma oferece planos de assinatura para os usuários. Cada plano possui:

- Nome (ex.: Básico, Premium, Empresarial).
- Descrição dos benefícios.
- Valor período.
- Periodicidade (mensal, anual).
- Status (ativo, inativo).

**RF04. Assinatura e Pagamento de Plano**
O sistema deve permitir que o usuário **assine um plano** e registre o pagamento correspondente.

- Cada assinatura vincula um usuário a um plano, com data de início e data de cancelamento (quando aplicável).
- Cada pagamento de assinatura registra: valor pago, data de pagamento, forma de pagamento (cartão de crédito, Pix, boleto), status (confirmado, pendente, recusado) e identificador externo de transação (para evitar duplicação por idempotência).
- Um usuário pode ter histórico de várias assinaturas ao longo do tempo, mas apenas uma assinatura ativa por vez.

### 3.3 Domínio Financeiro do Usuário

**RF05. Categorias de Movimentação**
O sistema deve permitir que o usuário cadastre categorias próprias para classificar suas movimentações financeiras. Cada categoria possui:

- Nome.
- Descrição.
- Tipo (receita, despesa).
- Cor ou ícone (opcional, para visualização).

Cada categoria pertence a um usuário específico (categorias de um usuário não são compartilhadas com outros).

**RF06. Registro de Movimentações Financeiras (Receitas e Despesas)**
O sistema deve permitir o registro de **movimentações financeiras** do usuário, sejam elas receitas ou despesas. Cada movimentação possui:

- Descrição.
- Valor.
- Data da movimentação.
- Categoria associada.
- Tipo (receita ou despesa).
- Status (paga/recebida, pendente).

> **Dilema de modelagem #2 (Unificação de Movimentações)**: como representar receitas e despesas no banco?
> Possíveis abordagens:
> (a) **Tabela única** com campo `tipo` indicando receita/despesa. Vantagem: evita duplicação estrutural, facilita consultas de saldo (basta somar com sinal). Desvantagem: pode misturar semânticas diferentes em uma única entidade.
> (b) **Tabelas separadas**. Vantagem: separação semântica clara, permite atributos exclusivos de cada tipo no futuro. Desvantagem: duplicação estrutural, consultas de saldo exigem UNION ou subconsultas.
> Você deve decidir e justificar.

**RF06.1. Movimentações Recorrentes**
O sistema deve permitir que o usuário marque uma movimentação como **recorrente** (ex.: salário mensal, aluguel, assinatura de streaming). Cada recorrência possui:

- Periodicidade (diária, semanal, mensal, anual).
- Data de início.
- Data de término (opcional; vazio significa recorrência indefinida).
- Dia do mês ou da semana para geração automática.

Movimentações geradas automaticamente a partir de uma recorrência mantêm vínculo com a recorrência original, permitindo edição em massa ou cancelamento.

### 3.4 Contas e Carteiras (Dilema Pedagógico)

> **Dilema de modelagem #3 (Contas Bancárias/Carteiras)**: o sistema deve incluir uma entidade de contas representando contas bancárias, carteiras digitais ou meios de pagamento do usuário?
>
> Argumentos a favor: aplicativos financeiros reais (Organizze, Mobills) suportam múltiplas contas por usuário (corrente, poupança, carteira física, cartões de crédito), permitindo saldos segregados e melhor rastreabilidade. Cada movimentação seria vinculada a uma conta de origem ou destino.
>
> Argumentos contra: aumenta complexidade do modelo e da interface. Para um MVP focado em controle financeiro simples para a Geração Z, pode ser dispensável. O saldo passa a ser calculado pela soma algébrica das movimentações sem segregação por conta.
>
> Decida se esta entidade será incluída no seu modelo e justifique com base no perfil do público-alvo e nos requisitos do sistema.

### 3.5 Orçamentos

**RF07. Orçamentos Mensais**
O sistema deve permitir que o usuário crie orçamentos mensais. Cada orçamento possui, no mínimo:

- Nome.
- Valor limite definido.
- Data de início e data final (geralmente um mês).
- Status (ativo, encerrado).

> **Dilema de modelagem #4 (Escopo do Orçamento)**: o orçamento deve ser **geral** (limitando o total de despesas do usuário no período) ou **por categoria** (limitando os gastos em uma categoria específica de despesa)?
>
> Argumentos a favor de orçamento **por categoria** (abordagem profissional e recomendada): é o padrão adotado por aplicativos financeiros consolidados (Organizze, Mobills, GuiaBolso), pois permite controle granular dos hábitos de consumo (ex.: limite de R$ 400 para alimentação, R$ 200 para lazer). O usuário identifica com clareza qual categoria está estourando, gerando notificações mais úteis e relatórios mais ricos. Um usuário pode ter múltiplos orçamentos simultâneos, um para cada categoria que deseja controlar.
>
> Argumentos a favor de orçamento **geral**: modelo mais simples para o usuário iniciante, sem necessidade de planejamento por categoria. Adequado a um MVP voltado para a Geração Z, que pode não ter maturidade financeira para definir limites por categoria. Reduz complexidade do banco e da interface.
>
> Você deve decidir e justificar.

### 3.6 Metas Financeiras

**RF08. Metas de Economia/Investimento**
O sistema deve permitir que o usuário defina metas financeiras pessoais. Cada meta possui:

- Nome (ex.: "Viagem para Europa", "Reserva de emergência").
- Valor desejado (objetivo a atingir).
- Valor atual acumulado.
- Data limite para atingir a meta.
- Status (em andamento, concluída, cancelada).

**RF08.1. Aportes em Metas**
O usuário deve poder registrar **aportes** (depósitos manuais) em suas metas. Cada aporte registra:

- Valor aportado.
- Data do aporte.
- Meta destino.
- Observação (opcional).

O `valor_atual` da meta é a soma de todos os seus aportes (não é atualizado automaticamente pelo saldo geral do usuário, como o levantamento original sugeria erroneamente).

### 3.7 Consultas

**RF09. Consulta de Saldo por Período**
O sistema deve calcular e exibir o saldo do usuário (total de receitas menos total de despesas) em um período informado, indicando se o saldo é positivo ou negativo.

**RF10. Consulta de Contas a Pagar (Despesas Futuras/Pendentes)**
O sistema deve exibir as despesas pendentes com vencimento dentro de um intervalo de datas informado.

**RF11. Consulta de Contas a Receber (Receitas Futuras/Pendentes)**
O sistema deve exibir as receitas previstas com data de recebimento dentro de um intervalo de datas informado.

**RF12. Histórico com Filtros**
O sistema deve permitir que o usuário visualize o histórico completo de receitas, despesas, orçamentos e metas, com filtros por período, categoria e status.

### 3.8 Notificações

**RF13. Notificações (Entidade Persistente)**
O sistema deve **persistir** todas as notificações emitidas ao usuário em uma entidade dedicada. Cada notificação possui:

- Usuário destinatário.
- Tipo (meta atingida, conta a pagar, orçamento estourado, outros).
- Título e mensagem.
- Data de criação.
- Status (não lida, lida, arquivada).
- Referência opcional à entidade relacionada (ex.: id da meta, id da despesa, id do orçamento).

As notificações são geradas pelas seguintes regras de negócio:

- **RF13.1**: notificação de **meta atingida** quando o valor acumulado de aportes de uma meta atinge ou supera o valor desejado.
- **RF13.2**: notificação de **conta a pagar** com um dia de antecedência ao vencimento de uma despesa pendente.
- **RF13.3**: notificação de **orçamento estourado** quando o total de despesas no escopo do orçamento (geral ou por categoria, conforme sua decisão no dilema #4) ultrapassa o valor limite definido, dentro do período do orçamento.

---

## 4. Requisitos Não Funcionais (RNF)

Apresentados apenas como contexto. **Não são objeto de modelagem** no banco de dados, mas algumas decisões de modelagem podem ser influenciadas por eles (ex.: campos de senha como hash, em atendimento ao RNF02).

- **RNF01**: autenticação de dois fatores (2FA) no login e em transações financeiras.
- **RNF02**: criptografia de dados pessoais e bancários, em uso e em repouso.
- **RNF03**: precisão na exibição de saldos e valores financeiros.
- **RNF04**: suporte a múltiplos usuários simultâneos.
- **RNF05**: compatibilidade com Firefox, Chrome, Edge e demais navegadores modernos.
- **RNF06**: navegação intuitiva e curva de aprendizado baixa.
- **RNF07**: boa performance no carregamento de páginas e na persistência de dados.

---

## 5. Resumo dos Dilemas de Modelagem

Cada aluno deverá tomar e justificar (em comentário no arquivo SQL) decisões sobre:

| # | Dilema | Decisão Esperada |
|---|--------|------------------|
| 1 | Como representar PF e PJ? | Generalização/especialização ou tabela única ou tabelas independentes. |
| 2 | Receitas e despesas: unificadas ou separadas? | Tabela única com tipo ou duas tabelas. |
| 3 | Incluir entidade de contas bancárias/carteiras? | Sim ou não, com justificativa. |
| 4 | Orçamento geral ou por categoria? | Decisão livre, abordagem por categoria é considerada profissionalmente superior. |

> **Decisão opcional adicional**: a aplicação de **soft-delete** (campo `deletado_em` ou similar) é decisão livre, conforme a Seção 4.1 do enunciado da atividade. Não conta como dilema numerado, mas deve ser registrada e justificada no bloco de comentários do SQL.

---

## 6. Glossário Mínimo

- **Movimentação**: termo genérico para receita ou despesa registrada pelo usuário.
- **Aporte**: depósito manual feito pelo usuário em uma meta financeira.
- **Recorrência**: padrão de repetição automática de uma movimentação ao longo do tempo.
- **Saldo**: diferença entre receitas e despesas em um período. Calculado, não armazenado.
- **Orçamento estourado**: situação em que o total de despesas de uma categoria, no período do orçamento, ultrapassa o valor limite definido.