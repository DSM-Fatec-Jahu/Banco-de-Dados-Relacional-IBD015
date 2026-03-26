# Aula 09 — Exercícios de SELECT: Banco de Dados Sakila

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 08](./Aula_08_Joins_Subconsultas_Views.md) · [Voltar ao README](../README.md)

---

## 📌 Sobre esta Lista

Esta lista utiliza o banco de dados **Sakila** (versão traduzida `sakila_pt`), que modela o sistema de uma locadora de filmes com filiais, clientes, aluguéis e pagamentos. Antes de começar, certifique-se de ter o banco populado e execute:

```sql
USE sakila_pt;
```

> O gabarito completo está em um arquivo separado: **`Aula_09_Gabarito_SELECT_Sakila.md`**

---

### Tabelas disponíveis

| Tabela | Descrição |
|---|---|
| `filmes` | Catálogo de filmes — título, duração, taxa de aluguel, classificação, idioma… |
| `atores` | Cadastro de atores |
| `filmes_atores` | Relacionamento N:M entre filmes e atores |
| `categorias` | Categorias de filme (Ação, Drama, Comédia…) |
| `filmes_categorias` | Relacionamento N:M entre filmes e categorias |
| `idiomas` | Idiomas disponíveis |
| `inventarios` | Cópias físicas dos filmes nas lojas |
| `lojas` | Filiais da locadora |
| `funcionarios` | Funcionários das lojas |
| `clientes` | Clientes cadastrados |
| `enderecos` | Endereços de clientes, funcionários e lojas |
| `cidades` | Cidades |
| `paises` | Países |
| `alugueis` | Registros de aluguel |
| `pagamentos` | Registros de pagamento |

### Mapa de relacionamentos

```
paises ──→ cidades ──→ enderecos ──→ clientes ──→ alugueis ──→ pagamentos
                                  └──→ funcionarios ──┘
idiomas ──→ filmes ──→ inventarios ──→ alugueis
         └──→ filmes_atores ──→ atores
         └──→ filmes_categorias ──→ categorias
lojas.funcionario_gerente_id ──→ funcionarios
```

---

## Parte 1 — SELECT Básico

> **Conceitos:** `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, `DISTINCT`, `LIKE`, `BETWEEN`, `IN`, `IS NULL`, `IS NOT NULL`, funções de string e data, expressões aritméticas no `SELECT`.

---

### 🟢 Exercícios Fáceis

---

**Exercício 1**

Liste o título, a classificação indicativa e a taxa de aluguel de todos os filmes, ordenados pelo título em ordem alfabética.

> **Caso de uso:** tela de catálogo da locadora — o atendente ou o cliente precisa navegar pelo acervo completo ordenado por nome para localizar um título.

> 💡 *Tabela: `filmes`.*

---

**Exercício 2**

Exiba o primeiro nome, o último nome e o e-mail de todos os clientes que estão ativos (`ativo = 1`), ordenados pelo último nome.

> **Caso de uso:** envio de e-mail marketing — o sistema de CRM precisa da lista de clientes ativos com contato para uma campanha promocional.

> 💡 *Tabela: `clientes`.*

---

**Exercício 3**

Liste os títulos de todos os filmes com classificação `'PG'` ou `'G'`, ordenados pela taxa de aluguel do mais barato para o mais caro.

> **Caso de uso:** filtro de conteúdo familiar no site da locadora — o responsável quer ver apenas filmes adequados para crianças, ordenados pelo custo.

> 💡 *Tabela: `filmes`. Use o operador `IN` para filtrar as classificações.*

---

**Exercício 4**

Liste os 10 filmes com maior duração, exibindo título e duração em minutos. Em caso de empate na duração, ordene pelo título alfabeticamente.

> **Caso de uso:** relatório de filmes longos — a locadora quer identificar os títulos que exigem maior período de locação para ajustar as taxas de aluguel.

> 💡 *Tabela: `filmes`. Coluna `duracao`.*

---

**Exercício 5**

Liste os nomes distintos de todos os países cadastrados, em ordem alfabética.

> **Caso de uso:** campo de seleção de país em formulário de cadastro — o sistema precisa popular um menu `<select>` com os países disponíveis, sem repetição.

> 💡 *Tabela: `paises`. Use `DISTINCT`.*

---

### 🟡 Exercícios Intermediários

---

**Exercício 6**

Liste o título e a duração de todos os filmes cuja duração esteja entre 90 e 120 minutos (inclusive) **e** que tenham taxa de aluguel acima de R$ 3,00. Ordene pelo título.

> **Caso de uso:** curadoria de sessão especial — a locadora quer montar uma grade de filmes de duração média com preço premium para um pacote de assinatura.

> 💡 *Tabela: `filmes`. Combine `BETWEEN` com `AND` para adicionar a segunda condição.*

---

**Exercício 7**

Encontre todos os atores cujo **último nome** começa com a letra `'S'`. Exiba o primeiro nome, o último nome e uma coluna `nome_completo` no formato `"SOBRENOME, Nome"` — último nome todo em maiúsculas e primeiro nome como cadastrado.

> **Caso de uso:** busca de elenco por inicial de sobrenome — o sistema permite que clientes encontrem atores pelo sobrenome, exibindo o nome formatado no padrão de fichas de catálogo.

> 💡 *Tabela: `atores`. Use `CONCAT`, `UPPER` e `LIKE 'S%'`.*

---

**Exercício 8**

Liste os filmes que **não possuem** idioma original cadastrado (coluna `idioma_original_id` é `NULL`). Exiba o título e a classificação, ordenados pela classificação e depois pelo título.

> **Caso de uso:** auditoria de dados do catálogo — a equipe de conteúdo precisa identificar filmes com cadastro incompleto para solicitar o preenchimento do idioma original.

> 💡 *Tabela: `filmes`. Use `IS NULL`.*

---

**Exercício 9**

Exiba o título dos filmes, a taxa de aluguel e o custo de reposição. Adicione uma coluna calculada chamada `margem_estimada` que representa a diferença entre `custo_reposicao` e `taxa_aluguel`. Ordene pela margem em ordem decrescente, mostrando apenas os 15 filmes com maior margem.

> **Caso de uso:** análise de risco do estoque — o gestor financeiro quer saber quais filmes têm maior custo de reposição em relação à taxa cobrada, para priorizar o cuidado com essas cópias.

> 💡 *Tabela: `filmes`. Uma expressão aritmética no `SELECT` cria uma coluna calculada.*

---

**Exercício 10**

Liste todos os clientes cadastrados **antes de 2006** (coluna `criado_em`). Exiba o nome completo formatado como `"Primeiro Último"`, o e-mail e a data de criação no padrão brasileiro `DD/MM/AAAA HH:MM`.

> **Caso de uso:** relatório de clientes antigos — o setor de fidelização quer contatar os clientes mais antigos da base para uma campanha de retenção com benefícios exclusivos.

> 💡 *Tabela: `clientes`. Use `DATE_FORMAT` para formatar a data e `CONCAT` para o nome.*

---

### 🔴 Exercícios Difíceis

---

**Exercício 11**

Para cada filme, exiba o título e uma coluna chamada `faixa_preco` com os seguintes rótulos baseados na `taxa_aluguel`:
- `'Econômico'` → taxa até R$ 1,99
- `'Regular'` → taxa de R$ 2,00 até R$ 3,99
- `'Premium'` → taxa de R$ 4,00 ou mais

Ordene pela faixa de preço e depois pelo título.

> **Caso de uso:** filtros por faixa de preço no aplicativo da locadora — o front-end precisa agrupar os filmes em categorias de valor para o cliente filtrar pelo orçamento disponível.

> 💡 *Tabela: `filmes`. Use `CASE WHEN ... THEN ... ELSE ... END` no `SELECT`.*

---

**Exercício 12**

Liste os filmes cujo título contém exatamente **5 palavras**. Para efeito deste exercício, considere que palavras são separadas por um único espaço — ou seja, o título possui exatamente 4 espaços. Exiba o título e o número de caracteres como `tamanho_titulo`.

> **Caso de uso:** geração de banners padronizados — o sistema de arte da locadora precisa selecionar títulos com comprimento específico para encaixar em um template de cartaz sem quebra de linha.

> 💡 *Tabela: `filmes`. O número de espaços pode ser calculado como `LENGTH(titulo) - LENGTH(REPLACE(titulo, ' ', ''))`.*

---

**Exercício 13**

Identifique os filmes com `taxa_aluguel` **e** `duracao` ambos acima da média geral. Exiba o título, a taxa de aluguel, a duração e quanto cada filme supera a média de duração em uma coluna chamada `excedente_duracao`. Use subconsultas no `WHERE` para comparar com as médias. Ordene pelo excedente de duração em ordem decrescente.

> **Caso de uso:** seleção de títulos para o pacote "Experiência Completa" — a equipe comercial quer destacar filmes que são simultaneamente longos e caros, posicionando-os como os títulos mais premium do catálogo.

> 💡 *Tabela: `filmes`. Use `(SELECT AVG(...) FROM filmes)` como subconsulta no `WHERE` para cada condição. Para o `excedente_duracao`, calcule a diferença diretamente no `SELECT`.*

---

**Exercício 14**

Liste os clientes que têm e-mail cadastrado **e** cujo e-mail pertence a domínios terminados em `.org` ou `.net`. Exiba o nome completo, o e-mail e o domínio extraído (tudo após o `@`) em uma coluna chamada `dominio`. Ordene pelo domínio e depois pelo nome completo.

> **Caso de uso:** segmentação de clientes por tipo de e-mail — o setor de parcerias quer contatar especificamente usuários com e-mails institucionais (`.org`) ou de provedores alternativos (`.net`) para uma campanha direcionada.

> 💡 *Tabela: `clientes`. Use `SUBSTRING_INDEX(email, '@', -1)` para extrair o domínio.*

---

**Exercício 15**

Liste todos os filmes cujo título começa e termina com a mesma letra (sem diferenciar maiúsculas de minúsculas). Exiba o título, a primeira letra e o total de caracteres do título como `tamanho`. Ordene pelo título.

> **Caso de uso:** geração de curiosidades para o blog da locadora — a equipe de marketing quer criar um post temático sobre filmes com títulos que "se fecham" na mesma letra, como uma trivia para engajamento nas redes sociais.

> 💡 *Tabela: `filmes`. Use `LEFT(titulo, 1)` para a primeira letra e `RIGHT(titulo, 1)` para a última. Aplique `LOWER` em ambos para comparar sem distinção de maiúsculas.*

---

## Parte 2 — SELECT com Agregação

> **Conceitos:** `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP BY`, `HAVING`, combinação com expressões e subconsultas.

---

### 🟢 Exercícios Fáceis

---

**Exercício 1**

Quantos filmes existem no catálogo? Exiba o resultado com o alias `total_filmes`.

> **Caso de uso:** dashboard administrativo — o painel de controle exibe estatísticas gerais do acervo ao gerente ao fazer login.

> 💡 *Tabela: `filmes`.*

---

**Exercício 2**

Qual é a taxa de aluguel média, a menor e a maior de todos os filmes? Exiba os três valores com aliases descritivos, arredondados com duas casas decimais.

> **Caso de uso:** relatório de precificação — o setor financeiro analisa mensalmente os indicadores de preço para definir se as taxas estão alinhadas com o mercado.

> 💡 *Tabela: `filmes`. Use `AVG`, `MIN` e `MAX` sem `GROUP BY`.*

---

**Exercício 3**

Quantos filmes existem por classificação indicativa? Liste a classificação e a quantidade, ordenada da classificação com mais filmes para a com menos.

> **Caso de uso:** relatório de composição do acervo — o responsável pelo conteúdo precisa saber a distribuição de classificações para garantir variedade e conformidade com a política da locadora.

> 💡 *Tabela: `filmes`. Use `GROUP BY classificacao`.*

---

**Exercício 4**

Qual é o total arrecadado em pagamentos? Exiba o resultado como `total_arrecadado`.

> **Caso de uso:** fechamento de caixa — o sistema financeiro totaliza toda a receita registrada para o balanço contábil do período.

> 💡 *Tabela: `pagamentos`.*

---

**Exercício 5**

Quantos clientes estão cadastrados em cada loja? Exiba o `loja_id` e o total de clientes, ordenados pelo total de forma decrescente.

> **Caso de uso:** comparativo de performance entre filiais — a diretoria usa esse número para avaliar qual loja tem a maior base de clientes e orientar investimentos em expansão.

> 💡 *Tabela: `clientes`. Use `GROUP BY loja_id`.*

---

### 🟡 Exercícios Intermediários

---

**Exercício 6**

Para cada classificação indicativa, exiba a quantidade de filmes, a duração média (arredondada para inteiro), a menor e a maior duração. Ordene pela duração média decrescente.

> **Caso de uso:** análise de acervo por faixa etária — a curadoria quer entender se filmes adultos tendem a ser mais longos que filmes infantis para compor pacotes de assinatura diferenciados.

> 💡 *Tabela: `filmes`. Use `ROUND(AVG(duracao), 0)` e `GROUP BY classificacao`.*

---

**Exercício 7**

Quais categorias possuem **mais de 60 filmes** cadastrados? Liste o `categoria_id` e a contagem, ordenados pela contagem decrescente.

> **Caso de uso:** identificação das categorias dominantes do catálogo — a equipe de aquisição usa essa informação para decidir quais gêneros já têm oferta suficiente e quais precisam de novos títulos.

> 💡 *Tabela: `filmes_categorias`. Use `HAVING COUNT(*) > 60`.*

---

**Exercício 8**

Calcule o valor total pago e a quantidade de pagamentos realizados por cada cliente. Exiba apenas os clientes que pagaram um total acima de R$ 150,00, ordenados pelo total pago em ordem decrescente.

> **Caso de uso:** programa de fidelidade — o sistema identifica os clientes de maior valor para oferecer benefícios exclusivos, como upgrades ou descontos progressivos.

> 💡 *Tabela: `pagamentos`. Use `SUM(valor)` e `COUNT(*)` com `GROUP BY cliente_id` e `HAVING`.*

---

**Exercício 9**

Para cada loja, calcule o número total de cópias no inventário e o número de filmes distintos disponíveis. Exiba `loja_id`, `total_copias` e `filmes_distintos`. Ordene por `total_copias` decrescente.

> **Caso de uso:** relatório de estoque por filial — o gestor de operações compara o tamanho do acervo físico de cada loja para planejar remanejamento de cópias entre filiais.

> 💡 *Tabela: `inventarios`. Use `COUNT(*)` para total de cópias e `COUNT(DISTINCT filme_id)` para filmes distintos.*

---

**Exercício 10**

Identifique os anos de lançamento com **mais de 100 filmes** no catálogo. Liste o `ano_lancamento` e a quantidade de filmes, ordenados pelo ano.

> **Caso de uso:** análise de concentração do acervo por época — a equipe de compras quer saber se o catálogo é concentrado em determinados anos para diversificar as aquisições.

> 💡 *Tabela: `filmes`. Use `GROUP BY ano_lancamento` com `HAVING COUNT(*) > 100`.*

---

### 🔴 Exercícios Difíceis

---

**Exercício 11**

Para cada ator, calcule em quantos filmes ele participou. Exiba o `ator_id`, o total de filmes e classifique cada ator em uma coluna `perfil` com as seguintes categorias:
- `'Figurante'` → até 15 filmes
- `'Coadjuvante'` → de 16 a 25 filmes
- `'Protagonista'` → mais de 25 filmes

Liste apenas os atores com mais de 10 filmes, ordenados pelo total em ordem decrescente.

> **Caso de uso:** ranking do elenco para página de atores — o site quer exibir uma badge de destaque ao lado de cada ator, baseada em sua presença no catálogo.

> 💡 *Tabela: `filmes_atores`. Use `CASE WHEN` dentro do `SELECT`, `GROUP BY ator_id` e `HAVING`.*

---

**Exercício 12**

Para cada mês e ano presentes na tabela `pagamentos`, calcule: o número de pagamentos realizados, o valor total arrecadado, o valor médio por pagamento e o valor do maior pagamento individual. Exiba apenas os meses com valor total acima de R$ 5.000,00. Ordene pelo valor total decrescente.

> **Caso de uso:** relatório financeiro mensal — o setor de contabilidade precisa do consolidado de receitas por período para o fechamento mensal e projeções de fluxo de caixa.

> 💡 *Tabela: `pagamentos`. Use `MONTH(data_pagamento)` e `YEAR(data_pagamento)` juntos no `GROUP BY` para distinguir o mesmo mês em anos diferentes.*

---

**Exercício 13**

Calcule a receita total e o número de aluguéis para cada valor de `taxa_aluguel` existente nos filmes. Exiba a `taxa_aluguel`, o `total_alugueis`, a `receita_total` e a `receita_media_por_aluguel` (com 2 casas decimais). Ordene pela taxa.

> **Caso de uso:** análise de elasticidade de preço — o setor comercial quer comparar se filmes com taxa mais alta geram proporcionalmente mais ou menos receita, para embasar uma revisão da tabela de preços.

> 💡 *Relacionamento: `pagamentos` → `alugueis` → `inventarios` → `filmes`. A agregação final usa `GROUP BY f.taxa_aluguel`.*

---

**Exercício 14**

Liste os **10 filmes mais alugados** de todos os tempos. Para cada filme, exiba o título, o total de aluguéis e o percentual que esse filme representa sobre o total geral de aluguéis (coluna `percentual`), formatado com 2 casas decimais e o símbolo `%`. Use uma subconsulta para calcular o total geral.

> **Caso de uso:** vitrine de "Mais Alugados" — a página inicial do site exibe um ranking dos títulos mais populares para guiar a escolha dos clientes indecisos.

> 💡 *Relacionamento: `alugueis` → `inventarios` → `filmes`. Use `(SELECT COUNT(*) FROM alugueis)` como subconsulta para o divisor do percentual.*

---

**Exercício 15**

Para cada **par (classificação × categoria)**, calcule quantos filmes existem. Exiba classificação, `categoria_id` e a contagem. Filtre apenas os pares com **mais de 5 filmes** e ordene pela contagem decrescente. Limite a 20 resultados.

> **Caso de uso:** análise cruzada de acervo — a gerência de conteúdo quer entender quais combinações de faixa etária e gênero são mais representadas no catálogo para embasar decisões de compra de novos títulos.

> 💡 *Relacionamento: `filmes` → `filmes_categorias`. Use `GROUP BY f.classificacao, fc.categoria_id` — ambas as colunas devem aparecer no `SELECT`.*

---

## Parte 3 — SELECT com INNER JOIN

> **Conceitos:** `INNER JOIN` com 2, 3 e 4+ tabelas, alias de tabela, combinação com `WHERE`, `GROUP BY`, `HAVING` e subconsultas.

---

### 🟢 Exercícios Fáceis

---

**Exercício 1**

Liste o título de cada filme e o nome do seu idioma. Exiba as colunas `titulo` e `idioma`. Ordene pelo nome do idioma e depois pelo título do filme.

> **Caso de uso:** página de detalhes do filme — o sistema exibe o idioma de produção para ajudar o cliente a decidir se precisa de legenda.

> 💡 *Relacionamento: `filmes` → `idiomas`.*

---

**Exercício 2**

Liste o nome completo dos atores e os títulos dos filmes em que participaram. Exiba as colunas `ator` e `titulo`. Ordene pelo nome do ator e depois pelo título.

> **Caso de uso:** página de filmografia de ator — ao clicar em um ator no site, o sistema carrega todos os filmes daquele ator disponíveis no catálogo.

> 💡 *Relacionamento: `atores` → `filmes_atores` → `filmes`.*

---

**Exercício 3**

Liste o nome completo de cada cliente e o nome da cidade em que mora. Exiba `cliente` e `cidade`. Ordene pela cidade e depois pelo nome do cliente.

> **Caso de uso:** relatório de distribuição geográfica de clientes — o setor de expansão usa essa lista para identificar cidades com maior concentração de clientes e planejar novas filiais.

> 💡 *Relacionamento: `clientes` → `enderecos` → `cidades`.*

---

**Exercício 4**

Liste o título dos filmes e o nome da categoria a que pertencem. Exiba `titulo` e `categoria`. Ordene pela categoria e depois pelo título.

> **Caso de uso:** navegação por gênero — a listagem de filmes por categoria no site exige combinar o título com sua categoria para renderizar cada card corretamente.

> 💡 *Relacionamento: `filmes` → `filmes_categorias` → `categorias`.*

---

**Exercício 5**

Liste o nome completo de cada funcionário e a cidade onde fica a loja em que trabalha. Exiba `funcionario` e `cidade_loja`. Ordene pelo nome do funcionário.

> **Caso de uso:** diretório interno de funcionários — o sistema de RH exibe a lotação de cada colaborador por cidade para facilitar a comunicação entre filiais.

> 💡 *Relacionamento: `funcionarios` → `lojas` → `enderecos` → `cidades`.*

---

### 🟡 Exercícios Intermediários

---

**Exercício 6**

Para cada aluguel realizado, exiba: o nome completo do cliente, o título do filme alugado, a data do aluguel e a data de devolução. Ordene pela data de aluguel decrescente. Limite a 20 resultados.

> **Caso de uso:** painel de aluguéis recentes — o atendente consulta os últimos registros de aluguel para acompanhar o movimento da loja em tempo real.

> 💡 *Relacionamento: `alugueis` → `clientes` e `alugueis` → `inventarios` → `filmes`.*

---

**Exercício 7**

Liste o nome completo do cliente, o título do filme e o valor pago em cada pagamento. Exiba apenas os pagamentos com valor acima de R$ 5,00. Ordene pelo valor pago de forma decrescente.

> **Caso de uso:** identificação de transações de alto valor — o setor financeiro monitora os pagamentos mais expressivos para análise de inadimplência e conferência de cobranças.

> 💡 *Relacionamento: `pagamentos` → `clientes` e `pagamentos` → `alugueis` → `inventarios` → `filmes`.*

---

**Exercício 8**

Para cada país, quantos clientes estão cadastrados? Exiba o nome do país e a contagem como `total_clientes`. Ordene pelo total de clientes decrescente.

> **Caso de uso:** relatório de alcance internacional — a diretoria avalia em quais países a locadora tem maior presença para direcionar campanhas de marketing globais.

> 💡 *Relacionamento: `clientes` → `enderecos` → `cidades` → `paises`. Use `GROUP BY` após os JOINs.*

---

**Exercício 9**

Liste o nome completo de cada funcionário e o nome completo do gerente da loja em que trabalha. Exiba `funcionario` e `gerente`. Ordene pelo nome do gerente e depois pelo nome do funcionário.

> **Caso de uso:** organograma de equipes — o sistema de RH precisa exibir a relação hierárquica entre funcionários e seus gestores diretos por filial.

> 💡 *Relacionamento: `funcionarios` (como funcionário) → `lojas` → `funcionarios` (como gerente). Você precisará da tabela `funcionarios` duas vezes com aliases distintos.*

---

**Exercício 10**

Liste o título dos filmes, sua categoria e a quantidade total de cópias disponíveis no inventário (todas as lojas somadas). Exiba `titulo`, `categoria` e `total_copias`. Ordene pelo total de cópias decrescente. Limite a 15 resultados.

> **Caso de uso:** relatório de títulos mais replicados — o setor de compras verifica quais filmes têm mais cópias físicas para equilibrar o estoque e evitar ociosidade.

> 💡 *Relacionamento: `filmes` → `filmes_categorias` → `categorias` e `filmes` → `inventarios`.*

---

### 🔴 Exercícios Difíceis

---

**Exercício 11**

Para cada categoria, liste: o nome da categoria, o número de filmes distintos disponíveis, o total de cópias no inventário e a receita total gerada por aluguéis de filmes dessa categoria. Ordene pela receita total decrescente.

> **Caso de uso:** análise de rentabilidade por gênero — o gestor financeiro quer saber quais categorias geram mais receita para priorizar a aquisição de novos títulos nesses gêneros.

> 💡 *Relacionamento: `categorias` → `filmes_categorias` → `filmes` → `inventarios` → `alugueis` → `pagamentos`. Use `COUNT(DISTINCT f.id_filme)` para filmes distintos e `SUM(p.valor)` para receita.*

---

**Exercício 12**

Identifique os **5 clientes que mais realizaram aluguéis**. Para cada um, liste: nome completo, total de aluguéis, total pago e o título do **último filme** que alugaram. O título do último filme deve ser obtido por uma subconsulta correlacionada.

> **Caso de uso:** relatório VIP — a gerência quer enviar um presente personalizado para os clientes mais fiéis, mencionando no e-mail o último filme que cada um alugou como toque de personalização.

> 💡 *Relacionamento principal: `clientes` → `alugueis` e `clientes` → `pagamentos`. Para o último filme, use subconsulta correlacionada por `cliente_id` em `alugueis` → `inventarios` → `filmes`, ordenando por `data_aluguel DESC LIMIT 1`.*

---

**Exercício 13**

Gere um relatório de **aluguéis em aberto** (ainda não devolvidos). Exiba: o nome completo do cliente, o título do filme, a data do aluguel, o prazo permitido em dias (`filmes.duracao_aluguel`) e o número de dias decorridos desde o aluguel até hoje. Adicione uma coluna `dias_atraso` que mostra quantos dias o aluguel ultrapassou o prazo — se não estiver atrasado, exiba `0`. Ordene pelos dias em atraso decrescente.

> **Caso de uso:** cobrança de multas por atraso — o sistema gera diariamente essa lista para que o atendente entre em contato com clientes inadimplentes e aplique as taxas de atraso correspondentes.

> 💡 *Relacionamento: `alugueis` → `clientes` e `alugueis` → `inventarios` → `filmes`. Filtre por `data_devolucao IS NULL`. Use `DATEDIFF(NOW(), a.data_aluguel)` para dias decorridos e `CASE WHEN` para calcular o atraso.*

---

**Exercício 14**

Para cada **par de atores** que compartilharam o mesmo filme, liste os nomes dos dois atores e o título do filme em que atuaram juntos. Para não repetir pares invertidos, inclua apenas registros onde o `id_ator` do primeiro for **menor** que o do segundo. Ordene pelo título e depois pelos nomes dos atores. Limite a 30 resultados.

> **Caso de uso:** sugestão de filmes com duplas conhecidas — o algoritmo de recomendação do site identifica duplas de atores que atuam juntas com frequência para sugerir ao cliente outros filmes com a mesma parceria.

> 💡 *Relacionamento: `filmes_atores` (como `fa1`) com `filmes_atores` (como `fa2`) pelo mesmo `filme_id`, restringindo `fa1.ator_id < fa2.ator_id`. Em seguida, faça JOIN com `atores` duas vezes para buscar os nomes de cada um.*

---

**Exercício 15**

Gere uma análise de **rentabilidade por loja**: para cada loja, exiba a cidade onde está localizada, o nome do gerente, o número de clientes cadastrados, o número de filmes distintos no inventário, o total de aluguéis realizados, a receita total e a receita por cliente (`receita_total ÷ total_clientes`, com 2 casas decimais). Ordene pela receita total decrescente.

> **Caso de uso:** comparativo de performance entre filiais — a diretoria usa esse painel na reunião mensal para avaliar a saúde financeira de cada loja e tomar decisões sobre investimentos, remanejamento de pessoal e fechamento de unidades.

> 💡 *Ponto de partida: `lojas`. A partir daí, relacione: `lojas` → `funcionarios` (gerente via `funcionario_gerente_id`), `lojas` → `enderecos` → `cidades`, `lojas` → `clientes`, `lojas` → `inventarios` → `filmes` e `lojas` → `inventarios` → `alugueis` → `pagamentos`.*

---

## 📚 Referências

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. São Paulo: Pearson, 2018.
- FORTA, B. *SQL em 10 Minutos por Dia*. 5 ed. São Paulo: Novatec, 2021.
- Documentação MariaDB — [SELECT](https://mariadb.com/kb/en/select/), [Aggregate Functions](https://mariadb.com/kb/en/aggregate-functions/), [JOIN](https://mariadb.com/kb/en/joins/)

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
