# Projeto CMPCD Jaú: Levantamento de Requisitos


## 1. Visão Geral do Sistema

O **CMPCD Jaú** (Conselho Municipal dos Direitos da Pessoa com Deficiência de Jahu) é um órgão colegiado consultivo, deliberativo e fiscalizador, responsável por acompanhar e propor políticas públicas voltadas às pessoas com deficiência no município de Jaú/SP. O sistema proposto é um **portal institucional integrado a um sistema operacional de cadastro e gerenciamento de PCDs**, voltado à Secretaria Municipal de Assistência e Desenvolvimento Social.

### 1.1 Domínios do Sistema

O sistema possui **três domínios distintos** que devem ser claramente separados na modelagem:

1. **Domínio Institucional (Portal Público)**: histórico do conselho, presidentes, membros do conselho, mandatos, organograma. Conteúdo público, consultivo, sem dados sensíveis. Visível a qualquer visitante.

2. **Domínio Operacional (Cadastro PCD)**: cadastro de pessoas com deficiência, dados pessoais, socioeconômicos, de saúde, documentos digitalizados, responsáveis legais. Contém **dados sensíveis** segundo o Art. 5º, II da LGPD (dados de saúde). Acesso restrito a usuários autenticados com perfil autorizado.

3. **Domínio Administrativo (Operadores e Segurança)**: usuários do sistema (operadores), perfis de acesso, controle de permissões. Sustenta o funcionamento dos outros dois domínios.

**Atenção**: usuários do sistema (operadores que fazem login) são **entidades distintas** dos PCDs cadastrados. PCDs são registros, não fazem login. Operadores são quem usa o sistema para cadastrar e gerenciar PCDs. Misturar essas duas entidades é erro grave de modelagem.

---

## 2. Atores do Sistema

| Ator | Descrição | Faz Login? |
|------|-----------|:----------:|
| **Visitante do Portal** | Qualquer pessoa que acessa o site público (cidadão, pesquisador, etc.). | Não |
| **Assistente Social** | Profissional da Secretaria que cadastra e gerencia PCDs. | Sim |
| **Conselheiro CMPCD** | Membro do conselho que acessa relatórios e estatísticas. | Sim |
| **Administrador/Editor** | Gestor do sistema, com permissões totais (gerencia operadores, conteúdo institucional, configurações). | Sim |
| **Pessoa com Deficiência (PCD)** | Cidadão cadastrado no sistema (registro, não acessa o sistema). | Não |

---

## 3. Requisitos Funcionais (RF)

### 3.1 Portal Institucional (Domínio Público)

**RF01. História Institucional**
O sistema deve exibir a história do conselho, sua origem, evolução e marcos institucionais relevantes. Cada marco histórico possui, no mínimo:

- Título.
- Descrição.
- Data ou ano do marco.

**RF02. Galeria de Presidentes**
O sistema deve exibir os presidentes do conselho ao longo do tempo. Cada presidente possui:

- Nome completo.
- Foto.
- Data de início do mandato.
- Data de término do mandato (pode estar vazia para o mandato atual).
- Biografia ou descrição curta (opcional).

**RF03. Membros do Conselho**
O sistema deve apresentar os membros ativos do conselho. Cada membro possui:

- Nome completo.
- Função no conselho (ex.: titular, suplente, secretário).
- Foto (opcional).
- Entidade ou segmento representado (ex.: poder público, sociedade civil, entidade de PCD).
- Data de início e data de término da participação no conselho.

> **Dilema de modelagem #1 (Membros e Presidentes)**: presidentes e membros do conselho compartilham atributos (nome, foto, períodos de atuação). Modelar como entidades independentes ou usar generalização (um membro pode ou não ter sido presidente, presidência é um papel temporário)?
> Possíveis abordagens: (a) entidades independentes com possível duplicação de dados, (b) entidade única com indicação de papel/mandato, (c) entidade membros com tabela associativa para presidências exercidas. Você deve decidir e justificar.

**RF04. Organograma Institucional**
O sistema deve exibir a estrutura organizacional oficial do conselho. Cada cargo ou setor do organograma possui:

- Nome do cargo ou setor.
- Descrição das atribuições.
- Cargo superior (referência hierárquica para construir a estrutura em árvore).
- Ordem de exibição entre pares.

**RF05. Formulário de Contato**
O sistema deve possuir um formulário de contato para envio de mensagens por e-mail diretamente. Deve conter os seguintes campos: nome, e-mail, telefone, assunto e mensagem, além dos contatos institucionais.

**RF06. Integração com Redes Sociais**
O sistema deve possuir integração com Instagram e Facebook para exibição das últimas postagens.

### 3.2 Espaço de Transparência Pública

**RF07. Documentos de Transparência Pública**
O sistema deve disponibilizar área pública para visualização e download de documentos oficiais. Cada documento possui, no mínimo:

- Título.
- Tipo (ata, resolução, edital, ofício, relatório, outros).
- Data de publicação.
- Arquivo digitalizado (referência ao caminho do arquivo armazenado).
- Descrição ou ementa.
- Status (publicado, arquivado, etc).

> **Dilema de modelagem #2 (Estrutura de Documentos)**: o sistema lida com dois tipos distintos de documentos: documentos de transparência pública (RF07: atas, resoluções, editais) e documentos anexos ao cadastro PCD (RF08: RG, comprovante de residência, laudo médico). Como modelar?
> Possíveis abordagens:
> (a) **Entidades separadas**  Vantagem: separação semântica clara, atributos específicos de cada tipo, permissões diferentes (público vs sensível). Desvantagem: duplicação estrutural se os atributos forem semelhantes.
> (b) **Entidade única** com discriminador de tipo e referência. Vantagem: estrutura unificada, mais simples. Desvantagem: mistura semânticas radicalmente diferentes (público vs sensível LGPD), atributos opcionais conforme contexto.
> Você deve decidir e justificar, considerando que documentos públicos e documentos sensíveis têm requisitos de segurança e privacidade muito distintos.

### 3.3 Cadastro e Gestão de PCDs (Domínio Sensível LGPD)

**RF08. Cadastro de PCD**
O sistema deve permitir o cadastro completo de cidadãos com deficiência. O cadastro contempla:

#### 3.3.1. Dados Pessoais (obrigatórios)

- Nome completo.
- Nome social (atendimento à dignidade da pessoa, conforme Decreto Federal 8.727/2016).
- CPF (único, validado).
- RG.
- Data de nascimento.
- Sexo biológico.
- Gênero (autodeclarado).
- Estado civil.
- Raça/cor (padrão IBGE: branca, preta, parda, amarela, indígena, não declarado).
- Nacionalidade.

#### 3.3.2. Endereço (obrigatório)

- CEP.
- Logradouro.
- Número.
- Complemento (opcional).
- Bairro.
- Cidade.
- UF.

#### 3.3.3. Contato

- Telefone principal.
- Telefone secundário (opcional).
- E-mail (opcional).
- Nome e telefone de contato de emergência.

#### 3.3.4. Dados Socioeconômicos (obrigatórios)

- Renda familiar mensal.
- Número de dependentes.
- Situação habitacional (própria quitada, própria financiada, alugada, cedida, outra).
- Escolaridade (sem instrução, fundamental incompleto, fundamental completo, médio incompleto, médio completo, superior incompleto, superior completo).
- Ocupação atual.
- Recebe BPC/LOAS (benefício de prestação continuada)? (sim/não). Se sim, qual?

#### 3.3.5. Dados de Saúde e Deficiência

- Tipos de deficiência (relacionamento N:N com tabela, ver dilema #3).
- Grau ou nível da deficiência (leve, moderada, severa, profunda). Se sim, qual?
- CID (Código Internacional de Doenças, opcional).
- Data de início ou diagnóstico da deficiência.
- Uso de tecnologia assistiva (cadeira de rodas, prótese, órtese, aparelho auditivo, bengala, outros).
- Necessita de acompanhante? (sim/não).
- Usa medicação contínua? Quais (texto livre)?

#### 3.3.6. Origem do Cadastro

- Origem do cadastro: enumerado contendo as entidades parceiras (AMAE, APAE, CISC, Cadastro Direto, Importação).

> **Dilema de modelagem #3 (Tipos de Deficiência)**: como representar tipos de deficiência?
> Possíveis abordagens:
> (a) **Tabela** com relacionamento com a PCD. Vantagem: representa fielmente a realidade (deficiência múltipla = uma PCD com mais de um tipo de deficiência associado), permite estatísticas precisas por tipo.
> (b) **ENUM fixo** com tipos (física, visual, auditiva, intelectual, múltipla, psicossocial). Vantagem: simplicidade. Desvantagem: se "múltipla" for um valor do ENUM, perde-se a informação sobre **quais** deficiências compõem a múltipla.
> Você deve decidir e justificar.

#### 3.3.7. Documentos Anexos (obrigatórios no cadastro)

- RG digitalizado.
- Comprovante de residência digitalizado.
- Laudo médico digitalizado.
- Outros documentos (opcionais: cartão SUS, carteira de identificação PCD, etc.).

Cada anexo armazena referência ao caminho do arquivo, data de upload, e usuário que fez o upload.

### 3.4 Responsável Legal

**RF09. Responsável Legal**
PCDs juridicamente incapazes (menores de idade ou com curatela) **devem ter** responsável legal cadastrado. Cada responsável legal possui:

- Nome completo.
- CPF.
- RG.
- Data de nascimento.
- Parentesco com a PCD (mãe, pai, tutor, curador, outro).
- Telefone.
- E-mail (opcional).
- Indicação se é responsável legal formal (com documentação de curatela/tutela).

Uma PCD pode ter um ou mais responsáveis legais associados (ex.: pai e mãe). Um responsável legal pode estar associado a mais de uma PCD (ex.: mãe de irmãos com deficiência).

**RF10. Gestão de Cadastros**
O sistema deve permitir localizar, editar e atualizar registros de PCD existentes, mantendo a integridade dos dados ao longo do tempo.

### 3.5 Importação de Dados

**RF11. Importação de Dados Externos** (Verificar se essa funcionalidade irá continuar)
O sistema deve permitir a importação de dados de PCDs a partir de planilhas Excel/CSV ou fichas físicas digitalizadas. Cada importação registra:

- Arquivo de origem (nome).
- Data e hora da importação.
- Usuário responsável pela importação.
- Quantidade de registros importados com sucesso.
- Quantidade de registros com erro.
- Status (em processamento, concluída, falhou).

PCDs importadas mantêm o atributo `origem_cadastro` preenchido com a entidade parceira correspondente (AMAE, APAE, CISC).

### 3.6 Segurança e Controle de Acesso

**RF12. Usuários do Sistema e Perfis de Acesso**
O sistema deve gerenciar usuários operadores (que fazem login) e seus perfis de acesso. Cada usuário do sistema possui:

- Nome completo.
- CPF.
- Celular.
- E-mail (único, usado para login).
- Senha (armazenada em hash).
- Perfil de acesso.
- Status (ativo, inativo, bloqueado).
- Data do último login (opcional).

Perfis de acesso:

- **Assistente Social**: cadastra e edita PCDs, visualiza dados completos.
- **Conselheiro CMPCD**: visualiza estatísticas e relatórios, não acessa dados pessoais identificáveis.
- **Administrador/Editor**: acesso completo (gerencia usuários, conteúdo institucional, configurações).

A modelagem dos perfis pode ser feita com tabela ou com ENUM, conforme sua decisão (não está como dilema explícito por ser decisão técnica menor).

### 3.7 Consultas e Relatórios 

> **Aviso pedagógico**: os requisitos a seguir descrevem **consultas, dashboards e exportações** que o sistema deve oferecer. Eles **não** correspondem a novas entidades no banco de dados. Os dados consultados já existem nas entidades anteriores. Você **não deve criar** tabelas como `estatisticas`, `dashboards` ou `relatorios`.

**RF13. Painel Estatístico e de Índices**
O sistema deve exibir dashboards administrativos com gráficos, estatísticas e indicadores sobre a população PCD cadastrada (ex.: distribuição por tipo de deficiência, faixa etária, bairro, renda familiar).

**RF14. Exportação de Relatórios**
O sistema deve permitir exportação de relatórios em Excel, CSV e PDF, gerados a partir dos dados das entidades existentes.

---

## 4. Dilemas Adicionais (Sua Decisão)

Além dos dilemas numerados acima, os seguintes pontos são decisões legítimas de modelagem que cada aluno deve tomar:

> **Dilema de modelagem #4 (Atendimentos e Acompanhamento)**: o sistema deve incluir uma entidade de **atendimentos** registrando o acompanhamento social, encaminhamentos e observações sobre cada PCD ao longo do tempo?
>
> Argumentos a favor: um cadastro vivo de PCD em um conselho municipal não é "cadastrou e acabou". Ações reais de assistência social envolvem registros de atendimentos, encaminhamentos a serviços públicos, observações sociais e evolução do caso. Aumenta o valor do sistema para o objetivo declarado de "planejamento de políticas públicas".
>
> Argumentos contra: o levantamento original não menciona atendimentos. Pode ser considerado fora do escopo do MVP, ficando para uma futura iteração do sistema. Reduz a complexidade do modelo. **Atenção**: se você decidir incluir atendimentos, provavelmente precisará também modelar uma entidade de parceiros (como AMAE, APAE, CISC, etc.) para registrar quem realizou o atendimento, indo além do atributo simples de origem do cadastro.
>
> Você deve decidir e justificar.

> **Dilema de modelagem #5 (Auditoria Detalhada de Alterações LGPD)**: o sistema deve incluir uma entidade de logs completos, registrando **todas** as alterações feitas em dados sensíveis (cadastro PCD, responsáveis legais)?
>
> Cada log registraria: usuário que alterou, tabela e campo alterado, valor anterior, valor novo, data e hora da alteração, motivo (opcional).
>
> Argumentos a favor: a LGPD (Lei 13.709/2018), citada no próprio RNF01, classifica dados de saúde como **dados pessoais sensíveis** (Art. 5º, II). O Art. 37 exige que o controlador mantenha registro das operações de tratamento. Em sistema governamental real, log detalhado de alterações **não é boa prática**, é **exigência legal**. Auditoria padrão (`criado_em`, `atualizado_em`) não atende a esse nível de rastreabilidade.
>
> Argumentos contra: aumenta significativamente o volume de dados e a complexidade da implementação. Em escopo didático e MVP, pode ser considerado exagero. A auditoria padrão obrigatória pelo enunciado (`criado_em` e `atualizado_em`) já cobre o mínimo.
>
> Você deve decidir e justificar. Recomenda-se considerar seriamente o argumento da LGPD.

---

> **Decisão opcional (Soft-delete)**: além dos dilemas acima, você também deve decidir sobre a aplicação de **soft-delete** (exclusão lógica via campo `deletado_em` ou similar), conforme a Seção 4.1 do enunciado da atividade. Considere que cadastros de PCD em órgão público dificilmente devem ser excluídos fisicamente (preservação histórica, transparência, possíveis demandas judiciais). Registre sua decisão e justificativa no bloco de comentários do SQL.

---

## 5. Requisitos Não Funcionais (RNF)

Apresentados apenas como contexto. **Não são objeto de modelagem** no banco de dados, mas algumas decisões de modelagem podem ser influenciadas por eles (ex.: senha armazenada em hash, em atendimento ao RNF01).

- **RNF01**: LGPD e Privacidade. Dados individuais dos PCDs devem ser protegidos e acessíveis apenas por usuários autenticados.
- **RNF02**: Integridade dos Dados. O sistema deve validar campos obrigatórios.
- **RNF03**: Acessibilidade. WCAG, leitores de tela, alto contraste, redimensionamento.
- **RNF04**: Responsividade. Desktop, tablets e smartphones.
- **RNF05**: Desempenho. Carregamento em até 3 segundos.
- **RNF06**: Stack tecnológica. HTML5, CSS3 (Bootstrap), JavaScript, MariaDB 10.4.
- **RNF07**: Interoperabilidade. Exportação em Excel, CSV e PDF.

---

## 6. Resumo dos Dilemas de Modelagem

Cada aluno deverá tomar e justificar (em comentário no arquivo SQL) decisões sobre:

| # | Dilema | Decisão Esperada |
|---|--------|------------------|
| 1 | Membros e presidentes do conselho: separados, unificados ou com generalização? | Decisão livre, com justificativa. |
| 2 | Documentos públicos e documentos PCD: entidades separadas ou unificadas? | Decisão livre, com justificativa. |
| 3 | Tipos de deficiência: tabela N:N ou ENUM? | Decisão livre, abordagem N:N é tecnicamente recomendada. |
| 4 | Incluir entidade de atendimentos/acompanhamento? | Decisão livre, com justificativa. |
| 5 | Incluir entidade de log detalhado de alterações (LGPD)? | Decisão livre, LGPD é forte argumento a favor. |

> **Decisão opcional adicional**: a aplicação de **soft-delete** (campo `deletado_em` ou similar) é decisão livre, conforme a Seção 4.1 do enunciado da atividade. Não conta como dilema numerado, mas deve ser registrada e justificada no bloco de comentários do SQL.

---

## 7. Glossário Mínimo

- **PCD**: Pessoa com Deficiência.
- **CMPCD**: Conselho Municipal dos Direitos da Pessoa com Deficiência.
- **BPC/LOAS**: Benefício de Prestação Continuada, previsto na Lei Orgânica da Assistência Social.
- **CID**: Código Internacional de Doenças, classificação da OMS.
- **Dados pessoais sensíveis**: categoria definida no Art. 5º, II da LGPD, que inclui dados sobre saúde, origem racial, convicção religiosa, opinião política, dados genéticos ou biométricos.
- **AMAE / APAE / CISC**: entidades parceiras locais que atendem pessoas com deficiência e podem ser origem de dados importados para o sistema.
- **Responsável Legal**: pessoa formalmente designada como tutor ou curador de uma PCD juridicamente incapaz.
- **Tecnologia Assistiva**: produto, recurso ou serviço que promove autonomia e independência a pessoas com deficiência.