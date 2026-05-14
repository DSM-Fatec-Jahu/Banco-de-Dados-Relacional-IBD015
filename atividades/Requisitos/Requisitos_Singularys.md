# Projeto Singularys: Levantamento de Requisitos 

---

## 1. Visão Geral do Sistema

O **Singularys** é um portal web que vende **servidores virtuais** (VPS) por assinatura. Na prática, funciona assim: o cliente escolhe um plano, paga, e o sistema cria automaticamente uma máquina virtual (VM) pronta para uso, entregando os dados de acesso pelo painel do cliente. A criação da VM acontece em uma plataforma chamada **Proxmox** (um software que gerencia máquinas virtuais em um servidor), a partir de modelos pré-configurados.

Não se preocupe se os termos de infraestrutura (Proxmox, VM, template, cloud-init) ainda forem novos para você. O glossário no final do documento explica cada um, e para esta atividade o que importa é entender **quais informações o sistema precisa guardar**, não como o Proxmox funciona por dentro.

O modelo comercial é **assinatura recorrente mensal**: o cliente contrata um plano e paga mensalmente pelo serviço. A falta de pagamento implica suspensão e, posteriormente, encerramento do serviço.

### 1.1 Domínios do Sistema

O sistema possui **três domínios distintos** que devem ser claramente separados na modelagem:

1. **Domínio Comercial**: catálogo de planos, pedidos, assinaturas recorrentes, faturas mensais e pagamentos. Trata da relação contratual e financeira entre o cliente e o Singularys.

2. **Domínio Operacional (Provisionamento)**: as máquinas virtuais criadas no Proxmox, seus dados de acesso e o registro técnico de cada etapa da criação. Trata da entrega técnica do serviço. ("Provisionamento" é só o nome técnico para o processo de criar e configurar a VM.)

3. **Domínio Administrativo (Identidade e Acesso)**: usuários, autenticação e controle de permissões. Sustenta o funcionamento dos demais domínios.

**Atenção**: pedidos (Domínio Comercial) e máquinas virtuais (Domínio Operacional) são entidades distintas, ainda que estreitamente relacionadas. Um pedido pode existir sem VM (antes do provisionamento), e uma VM existe sempre como consequência de um pedido aprovado.

---

## 2. Atores do Sistema

| Ator | Descrição | Faz Login? |
|------|-----------|:----------:|
| **Cliente** | Pessoa física ou jurídica que contrata planos de VPS. | Sim |
| **Administrador** | Gestor do sistema, com acesso completo (planos, usuários, VMs, configurações). | Sim |
| **Operador** | Funcionário de suporte técnico, com acesso a eventos de provisionamento e diagnóstico. | Sim |

---

## 3. Requisitos Funcionais (RF)

### 3.1 Identidade e Acesso

**RF01. Cadastro e Autenticação de Clientes**
O sistema deve permitir o cadastro e autenticação de clientes e demais usuários do sistema. Cada usuário possui, no mínimo:

- Nome completo.
- CPF ou CNPJ.
- Data de nascimento ou abertura da empresa.
- E-mail (único, usado para login).
- Senha (armazenada em hash com salt).
- Telefone.
- Status (ativo, inativo, bloqueado).
- Data do último login (opcional).
- Endereço para cobrança (opcional).

O sistema deve validar e-mail do cliente antes da liberação do uso pleno (validação de e-mail é pré-requisito para qualquer pedido, conforme item 10 do levantamento original sobre segurança).

> **Dilema de modelagem #1 (Controle de Permissões / Papéis)**: como representar os perfis de acesso (cliente, administrador, operador)?
>
> Possíveis abordagens:
> (a) **ENUM ou coluna direta** na tabela de usuários. Vantagem: simplicidade, adequado para conjunto pequeno e fixo de papéis. Desvantagem: usuário tem um único papel, sem flexibilidade.
> (b) **Tabela** com relacionamento à vários usuários. Vantagem: papéis viram dados editáveis, ainda mantém um papel por usuário.
> (c) **Tabela** com usuários podendo ter vários papéis e cada papel podendo ser atribuído à vários usuários. Vantagem: flexibilidade máxima (um usuário pode acumular papéis). Desvantagem: pode ser overkill para 3 papéis fixos com escopo claro.
>
> Você deve decidir e justificar.

### 3.2 Catálogo de Planos

**RF02. Catálogo de Planos de VPS**
O portal deve exibir os planos de VPS disponíveis para contratação. Cada plano possui:

- Nome (ex.: Basic, Standard, Performance, Pro).
- Descrição comercial.
- Quantidade de núcleos de CPU (vCPUs).
- Memória RAM (em megabytes ou gigabytes).
- Armazenamento (em gigabytes).
- Largura de banda mensal incluída (em gigabytes, opcional).
- Preço mensal.
- Status (ativo, descontinuado).

Planos descontinuados continuam existindo no histórico para preservar assinaturas antigas que ainda os utilizam.

### 3.3 Pedidos, Assinaturas e Pagamentos (Domínio Comercial)

#### 3.3.1. Pedido Inicial

**RF03. Pedido de Contratação**
O cliente deve poder selecionar um plano e gerar um pedido de contratação. Cada pedido registra, no mínimo:

- Cliente associado.
- Plano contratado.
- Data de criação.
- Valor total (corresponde a uma mensalidade do plano no momento da contratação, congelado para histórico).
- Status (criado, aguardando pagamento, pago, cancelado, provisionado, falhou).

O pedido é o ponto de entrada do fluxo comercial. Sua aprovação dá origem à **assinatura** do cliente e ao provisionamento da VM.

#### 3.3.2. Assinatura Recorrente

**RF04. Assinaturas Mensais (NOVO em relação ao levantamento original)**
O modelo comercial do Singularys é **recorrente mensal**. Após a confirmação do pagamento do pedido inicial, deve ser criada uma **assinatura** vinculando o cliente ao plano contratado de forma contínua. Cada assinatura possui:

- Cliente.
- Plano contratado.
- Pedido de origem (referência ao pedido inicial que criou a assinatura).
- Data de início.
- Data do próximo vencimento (calculada mensalmente).
- Data de cancelamento (preenchida quando a assinatura é encerrada).
- Status (ativa, suspensa, cancelada, encerrada).

Um cliente pode ter múltiplas assinaturas simultâneas (uma por VM contratada). A assinatura sustenta o ciclo de vida mensal da VM associada.

#### 3.3.3. Faturas Mensais

**RF05. Faturas Mensais (NOVO em relação ao levantamento original)**
A cada ciclo mensal de uma assinatura ativa, o sistema deve gerar automaticamente uma **fatura** para cobrança. Cada fatura possui:

- Assinatura de origem.
- Mês de competência (período coberto pela fatura).
- Data de emissão.
- Data de vencimento.
- Valor.
- Status (aberta, paga, vencida, cancelada).

Faturas vencidas há mais de um período de tolerância configurado (ex.: 5 dias) acionam suspensão automática da VM associada (status da VM = suspensa, status da assinatura = suspensa). Faturas pagas reativam o serviço.

> **Observação**: a relação entre **pedido inicial** (RF03) e **fatura recorrente** (RF05) é conceitualmente importante. O pedido inicial pode ser visto como a primeira fatura, ou como entidade separada (o pedido representa a intenção de contratação; a fatura representa a obrigação periódica de pagamento). A modelagem pode unificar ou separar essas entidades, com argumentos defensáveis dos dois lados. Você deve refletir sobre essa escolha.

#### 3.3.4. Pagamentos

**RF06. Integração com Gateway de Pagamento**
O sistema deve permitir que o cliente realize pagamento online (cartão de crédito, Pix, boleto) das faturas (ou do pedido inicial). Cada pagamento registra:

- Fatura ou pedido associado (ver dilema #2 abaixo).
- Gateway utilizado (ex.: Stripe, Mercado Pago).
- Identificador da transação no gateway (único).
- Chave de idempotência (um código único que evita que o mesmo pagamento seja registrado duas vezes por engano).
- Status (pendente, confirmado, recusado, estornado).
- Valor pago.
- Data de pagamento.
- Forma de pagamento (cartão, Pix, boleto).

A confirmação do pagamento chega por um *webhook* do gateway (uma mensagem automática que o gateway envia ao sistema avisando que o pagamento mudou de status), com validação e controle para não processar a mesma mensagem duas vezes.

> **Dilema de modelagem #2 (Pagamento vinculado a Fatura ou Pedido)**: no modelo original, cada pedido tinha exatamente um pagamento. Mas agora que existem faturas mensais, cada cliente passa a ter vários pagamentos ao longo do tempo (o do pedido inicial e o de cada fatura). A pergunta é: a quem o pagamento deve se ligar?
>
> Possíveis abordagens:
> (a) **Pagamento ligado ao Pedido** (modelo original): o pedido inicial gera um pagamento, e cada fatura mensal também gera um pagamento. Vantagem: ideia simples de entender. Desvantagem: o pagamento de uma fatura e o pagamento de um pedido são coisas conceitualmente diferentes, mas ficariam na mesma tabela.
> (b) **Pagamento ligado à Fatura**, tratando o pedido inicial como se fosse a primeira fatura. Vantagem: todo pagamento segue o mesmo caminho. Desvantagem: pode acabar misturando pedido e fatura, que são conceitos distintos.
>
> Você deve decidir e justificar.

### 3.4 Provisionamento e Configuração de Máquinas Virtuais (Domínio Operacional)

**RF07. Provisionamento Automático da VM**
Após a confirmação do pagamento do pedido inicial, o sistema deve criar automaticamente a VM no Proxmox. Essa criação parte de um *template* (um modelo de máquina pronto, com o sistema operacional já instalado) e usa o *cloud-init* (uma ferramenta que aplica as configurações iniciais da VM, como usuário, senha e rede, na primeira vez que ela liga).

**RF08. Configuração da VM**
O sistema deve aplicar automaticamente os recursos do plano contratado (CPU, memória, armazenamento, rede, hostname) à VM provisionada. Cada VM persiste, no mínimo:

- Cliente proprietário.
- Pedido/assinatura de origem.
- Nó Proxmox (o servidor físico onde a VM está rodando).
- VMID (o número que identifica a VM dentro do Proxmox).
- Hostname.
- Endereço IPv4 atribuído.
- Recursos aplicados (vCPUs, RAM, disco), copiados do plano no momento da criação (assim o histórico é preservado mesmo se o plano for alterado depois).
- Template utilizado (ver dilema #3).
- Status (provisionando, ativa, suspensa, encerrada, falha).
- Data de provisionamento.

> **Dilema de modelagem #3 (Templates de Sistema Operacional)**: o sistema provisiona VMs a partir de templates do Proxmox, com diferentes sistemas operacionais (Ubuntu 22.04, Debian 12, AlmaLinux 9, etc). Como modelar?
>
> Possíveis abordagens:
> (a) **Atributo texto livre** ou ENUM na VM. Vantagem: simplicidade. Desvantagem: dificulta gestão centralizada e relatórios; mudanças de catálogo exigem alteração no SQL.
> (b) **Tabela** com atributos (nome, sistema operacional, versão, identificador no Proxmox, status). Vantagem: catálogo gerenciável, permite associar planos a templates compatíveis, facilita auditoria. Desvantagem: complexidade adicional.
>
> Atributos relevantes para templates (caso modelado): nome amigável, sistema operacional, versão, identificador (ou nome) do template no Proxmox, status (disponível, descontinuado).
>
> Você deve decidir e justificar.

**RF09. Credenciais de Acesso da VM**
O sistema deve registrar e disponibilizar ao cliente as credenciais de acesso à sua VM no painel do portal. Cada conjunto de credenciais possui:

- VM associada.
- Usuário inicial (ex.: `ciuser`, `root`, `ubuntu`).
- Senha (ver observação técnica abaixo).
- Chave SSH pública cadastrada (opcional).

> **Observação técnica importante (não é um dilema)**: guardar a senha de uma VM no banco de dados, mesmo "criptografada", é considerado uma **má prática de segurança** em sistemas reais. O recomendado é mostrar a senha ao cliente **uma única vez** (por e-mail seguro ou em uma tela que só aparece uma vez) e não guardá-la, dando preferência ao acesso por chave SSH. Para esta atividade, pense se você vai mesmo incluir o campo de senha que estava no levantamento original do grupo, e escreva no bloco de justificativas a sua reflexão sobre isso. **Essa reflexão deve constar nas suas decisões.**

**RF10. Eventos de Provisionamento**
O sistema deve registrar todas as etapas do processo de provisionamento de cada VM. Cada evento possui:

- VM associada (ou referência ao pedido/assinatura quando a VM ainda não existe).
- Tipo do evento (início de provisionamento, clone de template, configuração de rede, aplicação de recursos, inicialização, conclusão, erro, reprocessamento).
- Status (em execução, sucesso, falha).
- Mensagem (descrição textual do evento ou da falha).
- Data e hora do evento.

Esta entidade serve à **auditoria técnica e diagnóstico de falhas**, não à auditoria LGPD (que, se for o caso, é outra entidade).

### 3.5 Processamento Assíncrono

> **Dilema de modelagem #4 (Fila de Tarefas)**: algumas operações do sistema (criar a VM, falar com o gateway de pagamento) são demoradas, então não acontecem na hora: elas entram em uma **fila** e são processadas aos poucos. A arquitetura do projeto prevê usar ferramentas específicas para isso (Redis ou RabbitMQ, que são programas feitos para gerenciar filas). A pergunta é: além dessas ferramentas, vale a pena ter também uma tabela de fila no banco de dados?
>
> Possíveis abordagens:
> (a) **Criar uma tabela de fila no banco**, como o levantamento original propõe. Vantagem: as tarefas ficam guardadas mesmo se a ferramenta de fila falhar, e dá para consultar o estado da fila com SQL. Desvantagem: vira informação repetida (a fila já existe no Redis/RabbitMQ), e banco de dados não foi feito para esse uso.
> (b) **Não criar a tabela** (deixar a fila só no Redis/RabbitMQ). Vantagem: cada ferramenta cuida do que faz melhor. Desvantagem: você perde a possibilidade de consultar a fila pelo SQL.
> (c) **Criar uma tabela só com o histórico** (tarefas já concluídas ou que falharam), sem controlar a fila em tempo real. Vantagem: você tem o registro para auditoria sem competir com a ferramenta de fila. Desvantagem: exige um passo a mais para salvar o resultado de cada tarefa.
>
> Você deve decidir e justificar.

### 3.6 Backups e Snapshots

> **Dilema de modelagem #5 (Backups e Snapshots da VM)**: o item 10 do levantamento original menciona "backups de metadados e, conforme plano, snapshot/backup da VM". O sistema deve modelar essa funcionalidade?
>
> Possíveis abordagens:
> (a) **Modelar entidade de snapshots/backups** com tipo (snapshot/backup completo), data, tamanho, status, localização de armazenamento, referência à VM. Permite catálogo de backups com retenção configurável por plano. Vantagem: atende a um diferencial comercial real.
> (b) **Não modelar** (fora do escopo do MVP). Vantagem: simplicidade. Desvantagem: perde funcionalidade significativa.
>
> Se modelado, considerar se planos diferentes têm políticas de backup diferentes (ex.: plano básico = sem backup, plano pro = 7 snapshots retidos). Isso pode levar a um atributo no plano (`politica_backup`) ou uma entidade de associação.
>
> Você deve decidir e justificar.

### 3.7 Consultas e Notificações


> - **Painel do cliente** (visualização de VMs, faturas, status): consulta sobre entidades existentes.
> - **Notificações por e-mail** (provisionamento concluído, fatura próxima do vencimento, suspensão): envio externo, sem persistência no banco (a menos que você decida modelar como entidade adicional, o que é decisão livre).
> - **Dashboards administrativos**: consultas agregadas sobre entidades existentes.
---

## 4. Requisitos Não Funcionais (RNF)

Apresentados apenas como contexto. **Não são objeto de modelagem** no banco de dados, mas algumas decisões de modelagem podem ser influenciadas por eles (ex.: idempotência em pagamentos, em atendimento ao RNF05).

- **RNF01**: Disponibilidade SLA de 99,5%.
- **RNF02**: Tempo de resposta inferior a 3 segundos para operações comuns.
- **RNF03**: Escalabilidade para crescimento simultâneo de clientes e VMs.
- **RNF04**: Comunicações via HTTPS (TLS 1.2+), armazenamento seguro de credenciais.
- **RNF05**: Idempotência no provisionamento de VMs e na confirmação de pagamentos.
- **RNF06**: Usabilidade e responsividade da interface.
- **RNF07**: Conformidade com a LGPD.
- **RNF08**: Observabilidade (logs, eventos, métricas).
- **RNF09**: Processamento assíncrono para operações críticas.

---

## 5. Resumo dos Dilemas de Modelagem

Cada aluno deverá tomar e justificar (em comentário no arquivo SQL) decisões sobre:

| # | Dilema | Decisão Esperada |
|---|--------|------------------|
| 1 | Papéis de acesso: ENUM, 1:N ou N:N? | Decisão livre, com justificativa. |
| 2 | Pagamento vinculado a Pedido ou Fatura? | Decisão livre, com justificativa. |
| 3 | Templates de SO: atributo simples ou tabela? | Decisão livre, com justificativa. |
| 4 | Fila de tarefas: modelar em banco, não modelar, ou apenas histórico? | Decisão livre, com justificativa. |
| 5 | Backups e snapshots: entidade modelada ou fora do escopo? | Decisão livre, com justificativa. |

> **Decisão opcional adicional**: a aplicação de **soft-delete** (campo `deletado_em` ou similar) é decisão livre, conforme a Seção 4.1 do enunciado da atividade. Não conta como dilema numerado, mas deve ser registrada e justificada no bloco de comentários do SQL.

### 5.1 Observação Obrigatória de Reflexão

Além dos dilemas, você deve registrar uma **reflexão obrigatória** sobre o RF09 (armazenamento de credenciais), considerando que guardar senhas de VM no banco é uma má prática de segurança.

---

## 6. Glossário Mínimo

- **VPS** (Virtual Private Server): servidor virtual privado, máquina virtual dedicada a um cliente.
- **VM** (Virtual Machine): máquina virtual, instância de servidor isolada provisionada em um hypervisor.
- **Proxmox**: hypervisor open source para virtualização de servidores, usado como plataforma de infraestrutura.
- **VMID**: identificador único de uma VM dentro do cluster Proxmox.
- **Cloud-init**: ferramenta de configuração inicial de VMs em primeira inicialização (definição de usuário, chaves SSH, hostname, rede).
- **Template**: imagem base pré-configurada (geralmente um sistema operacional) usada para clonar e gerar novas VMs.
- **Webhook**: notificação HTTP enviada pelo gateway de pagamento ao sistema para informar mudança de status de transação.
- **Idempotência**: propriedade que garante que uma operação repetida produz o mesmo efeito da execução única (essencial para pagamentos e provisionamento).
- **Assinatura recorrente**: vínculo contínuo entre cliente e plano, gerando faturas mensais até cancelamento.
- **Fatura**: documento mensal de cobrança gerado a partir de uma assinatura ativa.
- **Snapshot**: cópia pontual do estado de uma VM, usada para restauração rápida ou backup leve.