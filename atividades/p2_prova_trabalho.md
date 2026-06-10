# 🎬 P2 — Prova Trabalho: Uma Jornada na Locadora Sakila

---

## 📌 Antes de tudo — leia com atenção

### Formato de entrega

Você vai gravar um vídeo da sua tela executando e explicando cada comando SQL da avaliação. Ao terminar, **poste o vídeo no YouTube (pode ser não listado) ou em algum Drive online** e cole o link diretamente na atividade do Google Classroom da sua turma.

> ⚠️ **Sem computador?** Se você não tiver acesso a um computador para gravar em casa, a avaliação poderá ser realizada **presencialmente no dia 15/06 às 8h**, no laboratório da Fatec. Avise o professor com antecedência.

---

### 🎙️ Como gravar

Não é necessário aparecer na câmera — **apenas sua voz é obrigatória**. O importante é que a tela esteja visível e a explicação esteja clara.

Abaixo estão algumas opções gratuitas de gravação de tela:

| Programa | Sistema | Observação |
|---|---|---|
| **OBS Studio** | Windows, Mac, Linux | Gratuito e completo. Recomendado. |
| **Gravador do Windows** | Windows 10/11 | Nativo. Atalho: `Win + G` |
| **Clipchamp** | Windows 11 | Já vem instalado no Windows 11 |
| **Loom** | Windows, Mac, Web | Gratuito. Já gera link para compartilhar |
| **QuickTime Player** | Mac | Nativo. Menu Arquivo → Nova Gravação de Tela |

---

### 🎯 Objetivos de aprendizagem

Ao concluir esta avaliação, você vai demonstrar que é capaz de:

1. Consultar e agregar dados usando `SELECT`, `GROUP BY`, `COUNT`, `SUM` e `AVG`
2. Navegar e compreender o schema de um banco de dados real para identificar relacionamentos entre tabelas
3. Combinar tabelas com `INNER JOIN` e `LEFT JOIN`, entendendo quando usar cada um
4. Inserir dados respeitando a integridade referencial (ordem de inserção entre tabelas pai e filha)
5. Atualizar registros com `UPDATE` de forma precisa e segura
6. Diferenciar exclusão física (`DELETE`) de exclusão lógica (soft delete), aplicando cada uma no contexto correto
7. Implementar paginação de resultados com `LIMIT` e `OFFSET`
8. Comunicar decisões técnicas de forma clara e objetiva — como faria com um colega ou cliente

---

### ⚠️ Critério de flexibilidade — não se trave!

Se você travar em algum desafio específico (por exemplo, no cadastro do Desafio 4), isso **não impede que você continue**! Você pode usar dados de um cliente ou endereço que já existe no banco para demonstrar os desafios seguintes. O que vale é o seu raciocínio e a qualidade das explicações.

Cada desafio resolvido e bem explicado soma pontos à sua nota.

---

## 📖 A história: um dia de cliente e de gerente

---

## Parte 1 — O Totem de Autoatendimento

Você acabou de entrar na famosa locadora Sakila. Logo na entrada, tem um totem de autoatendimento moderno esperando por você.

---

### Desafio 1 — Contagem de Categorias

Ao iniciar a navegação, a tela do totem exibe um resumo do acervo da loja. Escreva a consulta SQL que o sistema roda por baixo dos panos para **listar todas as categorias de filmes** disponíveis e a **quantidade total de filmes** cadastrados em cada uma delas.

---

### Desafio 2 — Listagem de Filmes

Você olha as categorias disponíveis, escolhe uma que te chama atenção e clica nela. Escreva a consulta SQL que o sistema executa para **listar o título e demais dados importantes de todos os filmes** que pertencem àquela categoria — filtrando pelo nome da categoria diretamente, sem usar IDs.

> 💡 Dica: use o resultado do Desafio 1 para saber quais categorias existem no banco e escolha uma delas para trabalhar a partir daqui.

---

### Desafio 3 — Filtro Familiar

Você lembra que vai assistir ao filme com a família inteira, incluindo as crianças. Então ativa o filtro de classificação indicativa no totem. Apresente o SQL que o sistema executa para **refinar a busca anterior**, mostrando apenas os filmes daquela categoria que tenham classificação `'G'`.

---

## Parte 2 — O Cadastro e a Atualização de Dados

Você escolheu um filme e, na hora de finalizar, percebe que ainda não tem conta cadastrada no sistema da loja física.

---

### Desafio 4 — Autocadastro Completo

Cadastre-se na **Loja 1**! Analise a estrutura do banco de dados Sakila e escreva as instruções SQL que o sistema precisa executar para salvar o seu o seu cadastro como cliente com seu endereço completo, você deve usar seus dados reais.

> 💡 Preste atenção na estrutura das tabelas envolvidas. Alguns campos são obrigatórios e não aceitam valores nulos — o próprio banco vai te avisar se algo estiver faltando.

---

### Desafio 5 — Atualização de E-mail

Você percebe que digitou seu email errado. Escreva a consulta que o sistema usa para **localizar o seu `id_cliente`** no banco e, em seguida, o comando SQL para **atualizar o e-mail** para o padrão da faculdade: `seu_nome@fatecjahu.edu.br`.

> 💡 Caso não tenha conseguido realizar o Desafio 4, aplique este comando em qualquer cliente já existente no banco para fins de demonstração.

---

## Parte 3 — A Perspectiva do Gerente

Enquanto você navega no totem, o gerente da loja está nos computadores administrativos analisando o banco de dados para organizar o sistema e liberar espaço em disco.

Ele roda a seguinte consulta para encontrar cidades que estão salvas no banco mas que **nunca foram vinculadas a nenhum endereço** de cliente ou funcionário:

```sql
SELECT c.id_cidade, c.cidade
FROM cidades c
LEFT JOIN enderecos e ON c.id_cidade = e.cidade_id
WHERE e.id_endereco IS NULL;
```

---

### Desafio 6 — Limpeza de Dados

O gerente analisa o resultado dessa consulta e decide **remover definitivamente** do banco a primeira cidade que apareceu na lista como não utilizada. Apresente o comando de exclusão física que o sistema vai executar para apagar essa cidade específica.

---

## Parte 4 — A Desistência e a Exclusão Lógica

Você volta ao totem animado, mas descobre que todos os filmes que queria já foram alugados por outros clientes. Desapontado, você decide ir embora e pede para cancelar a conta que acabou de criar.

---

### Desafio 7 — Soft Delete por LGPD

A locadora Sakila segue as regras da LGPD e as políticas de auditoria financeira da empresa. Por isso, ela **não permite a exclusão física de registros de clientes** — um `DELETE` quebraria o histórico de transações passadas e poderia gerar problemas legais e contábeis sérios.

Apresente o comando de **exclusão lógica (soft delete)** que o sistema executa para desativar o seu usuário: alterando o status de atividade e registrando o momento exato da remoção, sem apagar nenhum dado do banco.

> 💡 Caso não tenha o seu usuário cadastrado, aplique este comando em qualquer cliente existente para fins de demonstração.

---

## Parte 5 — Relatórios e Paginação Administrativa

No fim do expediente, o gerente precisa extrair dados operacionais e financeiros para apresentar à diretoria.

---

### Desafio 8 — Paginação de Clientes

O gerente acessa um painel web para visualizar todos os **clientes ativos da Loja 1** com nome completo e endereço estruturado (Logradouro, Bairro e Cidade). A lista é grande, então o sistema exibe os resultados em páginas de 10 em 10 registros, em ordem alfabética pelo primeiro nome.

Apresente os **dois comandos SQL sequenciais** que o sistema executa: um para carregar a **Página 1** (os primeiros 10 clientes) e outro para a **Página 2** (os próximos 10).

---

### Desafio 9 — Relatório Anual de Faturamento

Para fechar o balanço histórico, o gerente precisa de um relatório que cubra **todo o ano de 2005**, agrupado dia a dia. O relatório deve mostrar para cada dia: a data, a quantidade de pagamentos realizados, o total faturado e o valor médio cobrado por transação (ticket médio).

O gerente ainda deixou claro: ele quer os dados **organizados de forma que façam sentido para uma leitura cronológica**. Pense em como você entregaria esse resultado para o seu chefe.

---

### Desafio 10 — Ranking de Lucratividade

Por fim, para planejar o estoque do próximo mês, o painel do gerente exibe uma lista das categorias de filmes ordenada pela **receita total acumulada** — da que mais gerou dinheiro até a que menos gerou. Apresente o SQL que o sistema executa para montar esse ranking.

---

## 📹 Resumo das instruções do vídeo

- **Ferramenta:** abra o MySQL Workbench, DBeaver, ou qualquer gerenciador de banco de dados de sua preferência
- **Execute** cada comando em tempo real durante a gravação
- **Explique** o que você está fazendo — não basta rodar o SQL. O que é um `LEFT JOIN`? Por que usou `GROUP BY` aqui? O que muda entre `LIMIT` e `OFFSET` de uma página para outra? O que diferencia um `DELETE` físico de um soft delete?
- **Câmera:** não é obrigatória. Apenas a sua voz e a tela precisam aparecer
- **Entrega:** poste o vídeo no YouTube (não listado) ou em algum Drive e cole o link na atividade do Classroom

---

## 📊 Rubrica de Avaliação

A nota final é composta por dois blocos: **execução técnica** (70%) e **explicação oral** (30%).

### Bloco 1 — Execução Técnica (70%)

Cada desafio vale **7 pontos**, distribuídos da seguinte forma:

| Critério | Peso |
|---|---|
| A query executa sem erros | 40% |
| A query retorna os dados corretos e completos | 35% |
| A lógica está adequada ao contexto do desafio | 25% |

### Bloco 2 — Explicação Oral (30%)

Avaliada de forma global ao longo de todo o vídeo:

| Critério | Peso |
|---|---|
| Explica o papel de cada cláusula usada (`JOIN`, `WHERE`, `GROUP BY`, etc.) | 40% |
| Justifica as escolhas técnicas (por que `LEFT JOIN` e não `INNER JOIN`? Por que soft delete?) | 35% |
| Clareza e objetividade na comunicação | 25% |

### Tabela de desafios e pesos

| Desafio | Tema | Peso na nota |
|---|---|---|
| 1 | Contagem de categorias | 10% |
| 2 | Listagem de filmes por categoria | 10% |
| 3 | Filtro por classificação indicativa | 10% |
| 4 | Cadastro com integridade referencial | 10% |
| 5 | Localização e atualização de e-mail | 10% |
| 6 | Deleção física com base em análise | 10% |
| 7 | Soft delete (LGPD) | 10% |
| 8 | Paginação administrativa | 10% |
| 9 | Relatório diário de faturamento | 10% |
| 10 | Ranking de lucratividade por categoria | 10% |

> **Entrega parcial é válida.** Apresente o máximo de desafios que conseguir. Desafios pulados não zeram a nota — cada um resolvido e bem explicado conta.

---