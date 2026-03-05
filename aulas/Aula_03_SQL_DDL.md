# Aula 03 — SQL e DDL: Definição de Estruturas

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 02](./Aula_02_Normalizacao.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_04_SQL_DML.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de criar e manipular estruturas de banco de dados usando os comandos DDL do SQL. Você entenderá as convenções de nomenclatura adotadas nesta disciplina, saberá escolher os tipos de dados mais adequados para cada situação, aplicará todas as principais constraints de integridade e compreenderá as diferenças práticas entre MariaDB (nosso ambiente principal via XAMPP) e PostgreSQL — o segundo SGBD mais adotado no mercado.

---

## 🧭 O que é DDL e por que é o primeiro passo?

Na Aula 02 você aprendeu a construir um modelo lógico — um conjunto de tabelas, colunas, chaves primárias e estrangeiras existindo ainda apenas no papel (ou no diagrama). A DDL, **Data Definition Language** (Linguagem de Definição de Dados), é o subconjunto do SQL responsável por transformar esse modelo em estruturas reais dentro do banco de dados.

Pense na metáfora da construção: se o modelo lógico é a planta arquitetônica, a DDL é o ato de erguer as paredes, instalar as portas e definir as divisões do espaço. Tudo o que vier depois — inserir dados, fazer consultas, criar lógica de negócio — depende de você ter executado a DDL corretamente primeiro.

Os três comandos centrais da DDL são `CREATE` (criar), `ALTER` (modificar) e `DROP` (remover). Eles operam sobre **objetos de banco de dados**: schemas (ou databases), tabelas, índices, views, procedures, entre outros. Nesta aula, o foco é na criação e manipulação de schemas e tabelas.

---

## 🎥 Vídeos de Apoio

- 📺 [SQL DDL — CREATE, ALTER e DROP explicados](https://www.youtube.com/watch?v=Ofwq7R8TxSI) — Bóson Treinamentos
- 📺 [Tipos de Dados no MySQL — Guia Completo](https://www.youtube.com/watch?v=M7TgHWpBvwQ) — Bóson Treinamentos
- 📺 [Constraints no MySQL — PK, FK, UNIQUE, CHECK](https://www.youtube.com/watch?v=jm7ZHZK0z7Q) — CFBCursos

---

## 1. Convenções de Nomenclatura desta Disciplina

Antes de escrever uma única linha de SQL, precisamos estabelecer um conjunto de convenções que seguiremos em toda a disciplina. Convenções não são caprichos estéticos — elas são acordos que tornam o código legível, previsível e manutenível por qualquer pessoa do time, inclusive você mesmo daqui a seis meses.

As regras a seguir estão alinhadas com as boas práticas da indústria e serão cobradas nas avaliações.

**Regra 1 — snake_case em tudo:** todas as palavras são separadas por underline, nunca por espaço, hífen ou camelCase. Exemplo: `data_nascimento`, não `DataNascimento` nem `data-nascimento`.

**Regra 2 — Sempre minúsculas para nomes criados pelo usuário:** tabelas, colunas, schemas, aliases — tudo em letras minúsculas. As únicas letras maiúsculas no seu código devem ser as palavras reservadas do SQL.

**Regra 3 — Palavras reservadas em MAIÚSCULAS:** `SELECT`, `CREATE`, `TABLE`, `INSERT`, `WHERE`, `NOT NULL`, `PRIMARY KEY` etc. Isso cria um contraste visual imediato entre o que é linguagem e o que é dado — fundamental para leitura rápida de código.

**Regra 4 — Nomes de tabelas sempre no plural:** a tabela armazena uma coleção de registros, então seu nome deve refletir isso. `clientes`, `pedidos`, `produtos` — nunca `cliente`, `pedido`, `produto`.

**Regra 5 — Chave primária no padrão `id_nome_tabela_singular`:** o nome da PK sempre usa o singular do nome da tabela. Exemplos: tabela `clientes` → PK `id_cliente`; tabela `pedidos` → PK `id_pedido`; tabela `categorias_produtos` → PK `id_categoria_produto`.

**Regra 6 - Chave estrangeira no padrão `tabela_referencia_id`:** o nome da FK preferencialmente usa o nome da tabela na qual se quer referênciar. Exemplo: na tabela `itens_pedidos` a chave estrangeira que referência a tabela `produtos` deve ser `produto_id`.

**Regra 7 — Chave estrangeira pelo papel semântico, não pelo nome da tabela:** este é o ponto mais sutil e importante. Quando uma FK referencia uma tabela cuja entidade pode exercer papéis diferentes, use o papel — não o nome da tabela. Veja o exemplo clássico:

```sql
-- ❌ ERRADO: não use pessoa1_id e pessoa2_id
-- Isso não comunica nada sobre o papel de cada pessoa

-- ✅ CORRETO: use o papel semântico de cada um
CREATE TABLE vendas (
    id_venda        INT          NOT NULL,
    cliente_id      INT          NOT NULL,  -- referencia pessoas (papel: cliente)
    funcionario_id  INT          NOT NULL,  -- referencia pessoas (papel: vendedor)
    data_venda      DATE         NOT NULL,
    PRIMARY KEY (id_venda),
    FOREIGN KEY (cliente_id)     REFERENCES pessoas (id_pessoa),
    FOREIGN KEY (funcionario_id) REFERENCES pessoas (id_pessoa)
);
```

O padrão de nomenclatura de FK neste caso, é, portanto, `papel_id` — onde `papel` descreve o que aquela entidade representa no contexto daquele relacionamento.

---

## 2. Acessando o MariaDB via Terminal (XAMPP)

Antes de criar qualquer estrutura, você precisa saber como se conectar ao banco. O XAMPP instala o MariaDB com configurações padrão que você deve conhecer.

**Configurações padrão do XAMPP:**

| Parâmetro | Valor padrão |
|---|---|
| Host | `127.0.0.1` ou `localhost` |
| Porta | `3306` |
| Usuário root | `root` |
| Senha root | *(vazia — sem senha)* |

**Conexão via terminal — sem especificar porta (usa a padrão 3306):**

```bash
# Forma mais comum — porta 3306 é assumida automaticamente
mysql -u root -p

# Se o MariaDB estiver no PATH do sistema (Windows: geralmente não está por padrão)
# No Windows com XAMPP, navegue até a pasta bin primeiro:
# C:\xampp\mysql\bin\mysql.exe -u root -p
```

**Conexão via terminal — especificando a porta explicitamente:**

```bash
# Útil quando você tem múltiplas instâncias ou a porta foi alterada
mysql -u root -p --port=3306

# Forma abreviada com flag -P (P maiúsculo para porta, p minúsculo para senha)
mysql -u root -p -P 3306 -h 127.0.0.1
```

**Conexão com usuário e senha já informados (não recomendado em produção, mas útil em aula):**

```bash
# A senha fica visível no histórico do terminal — evite em ambientes reais
mysql -u root -pSUASENHA

# Com porta e host explícitos — útil para scripts de automação em aula
mysql -u root -p123456 -P 3306 -h 127.0.0.1
```

> ⚠️ **Atenção:** no XAMPP padrão, a senha do root é **vazia**. Então você digita `mysql -u root -p` e pressiona Enter sem digitar nada quando solicitado. Em um ambiente real de produção, isso seria inadmissível — sempre defina uma senha forte para o usuário administrador.

**Verificando a versão após conectar:**

```sql
-- Confirma se você está no MariaDB e qual versão
SELECT VERSION();

-- Resultado esperado (exemplo): 10.4.32-MariaDB
```

---

## 3. MariaDB vs MySQL vs PostgreSQL — Entendendo o Cenário

Antes de mergulhar nos comandos, é importante entender o ecossistema em que estamos trabalhando, porque você encontrará esses três SGBDs em projetos reais e na literatura.

**MariaDB** nasceu como um *fork* (ramificação) do MySQL em 2009, criado pelos fundadores originais do MySQL após a aquisição deste pela Oracle. O objetivo foi manter um SGBD open source com desenvolvimento comunitário ativo. O MariaDB mantém compatibilidade quase total com o MySQL em nível de SQL, mas tem evoluído de forma independente — com otimizações de performance, novos mecanismos de armazenamento e funcionalidades exclusivas. É o SGBD instalado pelo XAMPP a partir da versão 5.7+.

**MySQL** é o SGBD open source mais usado no mundo historicamente. Desde a aquisição pela Oracle, seu desenvolvimento ficou dividido entre a versão Community (gratuita) e a Enterprise (paga). Para fins de aprendizado e para a maior parte do que faremos em DDL, DML e DQL, MySQL e MariaDB são praticamente intercambiáveis — as diferenças aparecem em funcionalidades avançadas.

**PostgreSQL** é o SGBD open source mais avançado tecnicamente. Segue o padrão SQL de forma mais rigorosa que MySQL/MariaDB, suporta tipos de dados mais ricos (como arrays, JSON nativo, tipos geométricos), tem um sistema de constraints mais poderoso e é amplamente preferido em aplicações que exigem integridade e consistência absolutas. É o banco padrão em muitas plataformas cloud (Heroku, Supabase, Railway).

### 3.1 Diferença estrutural: Databases vs Schemas

Esta é uma diferença fundamental que confunde muitos desenvolvedores ao migrar entre os dois sistemas.

**No MySQL/MariaDB**, os termos `DATABASE` e `SCHEMA` são **sinônimos** — comandos como `CREATE DATABASE` e `CREATE SCHEMA` fazem exatamente a mesma coisa. Um "banco de dados" é o nível mais alto de organização, e dentro dele ficam as tabelas. Você cria múltiplos databases para separar projetos diferentes.

```sql
-- No MariaDB/MySQL, estes dois comandos são idênticos:
CREATE DATABASE loja_virtual;
CREATE SCHEMA loja_virtual;  -- faz a mesma coisa
```

**No PostgreSQL**, a hierarquia é diferente e mais granular. Existe o **cluster** (instância do servidor) → **database** (banco) → **schema** (namespace lógico dentro do banco) → **tabelas**. Um database pode conter múltiplos schemas, e o schema padrão se chama `public`. Isso permite organizar tabelas de módulos diferentes dentro de um mesmo banco sem misturá-las.

```sql
-- No PostgreSQL, você cria o banco UMA VEZ e depois schemas dentro dele:
CREATE DATABASE loja_virtual;
\c loja_virtual  -- conecta ao banco (comando do psql)

CREATE SCHEMA financeiro;
CREATE SCHEMA estoque;
CREATE SCHEMA clientes;

-- As tabelas ficam dentro dos schemas:
CREATE TABLE financeiro.pedidos ( ... );
CREATE TABLE estoque.produtos ( ... );
```

![Compração MySQL / PostgreSQL](../imgs/Aula_03_IMG_01.png)

---

## 4. Criando um Database — Construção Gradual

Vamos construir o comando `CREATE DATABASE` de forma incremental, adicionando um atributo por vez e entendendo o motivo de cada um. Esta abordagem gradual é exatamente como você deve pensar ao escrever DDL em projetos reais.

### 4.1 Forma mínima

```sql
-- Versão mais simples: apenas cria o banco com as configurações padrão do servidor
CREATE DATABASE loja_virtual;
```

Funciona, mas delega as configurações de caracteres ao padrão do servidor — o que pode causar problemas se o servidor estiver configurado com um charset diferente do que você espera. Nunca use esta forma em projetos reais.

### 4.2 Adicionando proteção contra erro

```sql
-- IF NOT EXISTS evita erro se o banco já existir
-- Útil em scripts que podem ser executados mais de uma vez (idempotência)
CREATE DATABASE IF NOT EXISTS loja_virtual;
```

O `IF NOT EXISTS` torna o script **idempotente** — você pode rodá-lo múltiplas vezes sem quebrar nada. É uma boa prática em scripts de setup de ambiente.

### 4.3 Especificando o CHARACTER SET

```sql
-- Definindo explicitamente o conjunto de caracteres
CREATE DATABASE IF NOT EXISTS loja_virtual
    CHARACTER SET utf8mb4;
```

O `CHARACTER SET` (ou `CHARSET`) define **como os caracteres são armazenados em bytes** no disco. É a escolha mais impactante que você fará ao criar um banco, porque uma vez que há dados, mudar o charset é trabalhoso.

**Por que `utf8mb4` e não simplesmente `utf8`?** Esta é uma das armadilhas mais clássicas do MySQL/MariaDB. O tipo chamado `utf8` no MySQL é, na verdade, uma implementação **incompleta** do UTF-8 — ele suporta apenas caracteres de até 3 bytes, o que exclui emojis, caracteres chineses de planos suplementares e outros símbolos modernos. O `utf8mb4` é a implementação **completa e correta** do UTF-8, suportando todos os 1.114.112 pontos de código Unicode (incluindo emojis como 🎉, que usam 4 bytes). **Sempre use `utf8mb4`.**

A tabela abaixo resume os principais charsets disponíveis:

| Charset | Bytes por caractere | Suporte | Quando usar |
|---|---|---|---|
| `latin1` | 1 | Apenas caracteres ocidentais (ISO 8859-1) | Sistemas legados, nunca em projetos novos |
| `utf8` (MySQL) | 1–3 | Unicode básico, **sem emojis** | Nunca — use utf8mb4 |
| `utf8mb4` | 1–4 | Unicode completo, emojis incluídos | **Sempre — padrão recomendado** |
| `utf16` | 2–4 | Unicode completo | Casos especiais de integração |
| `ascii` | 1 | Apenas 128 caracteres ASCII | Colunas técnicas internas (ex: hashes) |

### 4.4 Especificando o COLLATION

```sql
-- Adicionando a collation — como os caracteres são COMPARADOS e ORDENADOS
CREATE DATABASE IF NOT EXISTS loja_virtual
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;
```

Se o `CHARACTER SET` define *como armazenar*, o `COLLATE` define *como comparar*. A collation determina as regras de ordenação e de igualdade entre strings — o que afeta diretamente operações de `WHERE`, `ORDER BY`, `GROUP BY` e índices em colunas de texto.

O sufixo `_ci` significa **Case Insensitive** (não diferencia maiúsculas de minúsculas): `'Ana'` = `'ana'` = `'ANA'`. O sufixo `_cs` significa **Case Sensitive**. O sufixo `_bin` compara byte a byte — é o mais estrito.

As collations mais usadas com `utf8mb4` no MariaDB são:

| Collation | Diferencia maiúsc./minúsc. | Diferencia acentos | Quando usar |
|---|---|---|---|
| `utf8mb4_general_ci` | Não | Não | Performance ligeiramente melhor, menos preciso |
| `utf8mb4_unicode_ci` | Não | Não (agrupa `e`, `é`, `ê`) | **Recomendado para a maioria dos casos** |
| `utf8mb4_unicode_520_ci` | Não | Parcialmente | Unicode 5.2.0 — mais atual que unicode_ci |
| `utf8mb4_0900_ai_ci` | Não | Não | MySQL 8.0+ / MariaDB 10.10+ — padrão moderno |
| `utf8mb4_bin` | Sim | Sim | Comparação byte a byte — campos técnicos |

> 💡 **Diferença importante entre `general_ci` e `unicode_ci`:** a `unicode_ci` usa as regras do algoritmo Unicode de comparação (UCA), o que a torna mais correta em situações como a comparação de letras com diacríticos em idiomas não-ingleses. Para um sistema em português, `utf8mb4_unicode_ci` é a escolha mais segura e semânticamente correta.

### 4.5 Comando final completo e comentado

```sql
-- Criação completa e idiomática do banco de dados
-- Pronto para uso em projetos reais com dados em português
CREATE DATABASE IF NOT EXISTS loja_virtual
    CHARACTER SET utf8mb4          -- UTF-8 completo: suporta todos os caracteres Unicode
    COLLATE utf8mb4_unicode_ci;    -- Comparação case-insensitive seguindo o padrão Unicode

-- Selecionar o banco para uso nas próximas operações
USE loja_virtual;

-- Confirmar configurações aplicadas
SHOW CREATE DATABASE loja_virtual;
```

**Equivalente no PostgreSQL** (para referência — a sintaxe é bem diferente):

```sql
-- No PostgreSQL, o encoding equivalente ao utf8mb4 é simplesmente UTF8
-- A collation é definida via locale do sistema operacional
CREATE DATABASE loja_virtual
    ENCODING    'UTF8'
    LC_COLLATE  'pt_BR.UTF-8'   -- locale para português do Brasil
    LC_CTYPE    'pt_BR.UTF-8'
    TEMPLATE    template0;       -- necessário quando LC_ difere do template padrão
```

---

## 5. Tipos de Dados — Guia Aprofundado

A escolha do tipo de dado correto é uma das decisões mais importantes do DDL. Um tipo errado desperdiça espaço em disco, compromete a performance de índices e pode causar bugs silenciosos — como truncamento de strings ou imprecisão em cálculos financeiros.

### 5.1 Tipos Numéricos Inteiros

Os tipos inteiros diferem apenas na faixa de valores que suportam e no espaço que ocupam. Escolha sempre o menor tipo que comporte os valores esperados — isso impacta diretamente o tamanho dos índices.

| Tipo (MariaDB/MySQL) | Equivalente PostgreSQL | Bytes | Faixa (sem sinal) | Caso de uso típico |
|---|---|---|---|---|
| `TINYINT` | `SMALLINT` | 1 | 0 a 255 | Flags, status com poucos valores |
| `SMALLINT` | `SMALLINT` | 2 | 0 a 65.535 | Quantidades pequenas, códigos |
| `MEDIUMINT` | *(sem equivalente)* | 3 | 0 a 16.777.215 | Contadores médios |
| `INT` / `INTEGER` | `INTEGER` | 4 | 0 a ~4,29 bilhões | **Chaves primárias — uso geral** |
| `BIGINT` | `BIGINT` | 8 | 0 a ~18,4 quintilhões | IDs de alto volume, timestamps Unix |

**Armadilha clássica — `INT(11)` não é o que parece:**  no MySQL/MariaDB, o número entre parênteses em `INT(11)` **não** define o tamanho de armazenamento nem a faixa de valores — ele apenas especifica a largura de exibição ao usar o flag `ZEROFILL`. Um `INT(1)` e um `INT(11)` ocupam exatamente 4 bytes e armazenam os mesmos valores. Essa confusão é tão comum que o MariaDB 10.7+ e o MySQL 8.0+ **depreciaram** a sintaxe de largura de exibição para tipos inteiros.

```sql
-- ❌ Confuso e depreciado no MariaDB 10.7+:
id_produto INT(11) NOT NULL

-- ✅ Correto e moderno:
id_produto INT NOT NULL
```

**`UNSIGNED` — quando usar:** o modificador `UNSIGNED` elimina os valores negativos e dobra o limite positivo. Chaves primárias auto-incrementadas **nunca** serão negativas, então faz sentido usá-lo — mas atenção: operações de subtração entre dois `UNSIGNED` podem gerar erro se o resultado for negativo.

```sql
-- Chave primária sem sinal: dobra o limite de ~2 bi para ~4,29 bi
id_cliente INT UNSIGNED NOT NULL AUTO_INCREMENT
```

### 5.2 Tipos Numéricos com Ponto Flutuante e Decimais

Esta é a área onde mais ocorrem bugs silenciosos. A diferença entre `FLOAT`/`DOUBLE` e `DECIMAL` é fundamental.

| Tipo (MariaDB/MySQL) | Equivalente PostgreSQL | Armazenamento | Precisão | Caso de uso |
|---|---|---|---|---|
| `FLOAT` | `REAL` | 4 bytes | ~7 dígitos decimais | Medições científicas — **nunca valores monetários** |
| `DOUBLE` | `DOUBLE PRECISION` | 8 bytes | ~15 dígitos decimais | Cálculos científicos — **nunca valores monetários** |
| `DECIMAL(p,s)` | `NUMERIC(p,s)` | Variável | Exata — até 65 dígitos | **Valores monetários, notas, medidas precisas** |

**Por que nunca usar FLOAT ou DOUBLE para dinheiro?** Porque esses tipos usam representação de ponto flutuante binário (IEEE 754), que não consegue representar exatamente todos os decimais. `0.1 + 0.2` em ponto flutuante resulta em `0.30000000000000004`, não em `0.3`. Isso é tolerável em cálculos científicos, mas catastrófico em sistemas financeiros.

```sql
-- ❌ ERRADO para valores monetários — pode causar erros de arredondamento:
preco FLOAT NOT NULL

-- ✅ CORRETO: DECIMAL(10, 2) armazena até 10 dígitos no total, 2 após a vírgula
-- Suporta valores de -99999999.99 até 99999999.99
preco DECIMAL(10, 2) NOT NULL

-- Para sistemas que exigem mais casas (câmbio, criptomoedas):
cotacao DECIMAL(18, 8) NOT NULL
```

O `DECIMAL(p, s)` onde `p` é a **precisão** (total de dígitos) e `s` é a **escala** (dígitos após o ponto decimal). Um `DECIMAL(10, 2)` suporta valores como `99999999.99`.

### 5.3 Tipos de Texto

| Tipo (MariaDB/MySQL) | Equivalente PostgreSQL | Tamanho máximo | Quando usar |
|---|---|---|---|
| `CHAR(n)` | `CHAR(n)` | 255 caracteres | Strings de tamanho **fixo** (CPF, CEP, siglas) |
| `VARCHAR(n)` | `VARCHAR(n)` | 65.535 bytes | Strings de tamanho **variável** — uso mais comum |
| `TINYTEXT` | `TEXT` | 255 bytes | Textos curtos (raramente necessário) |
| `TEXT` | `TEXT` | 65.535 bytes | Descrições, comentários |
| `MEDIUMTEXT` | `TEXT` | 16 MB | Artigos, conteúdo longo |
| `LONGTEXT` | `TEXT` | 4 GB | Logs, documentos muito longos |

**`CHAR` vs `VARCHAR` — a diferença que importa:** `CHAR(n)` sempre ocupa `n` bytes, preenchendo com espaços à direita quando o valor é menor. `VARCHAR(n)` ocupa apenas o espaço necessário mais 1–2 bytes de overhead para armazenar o comprimento. Use `CHAR` quando o dado sempre terá o mesmo tamanho — o acesso é ligeiramente mais rápido porque o banco sabe exatamente onde termina cada valor.

```sql
-- CHAR é ideal para campos de tamanho fixo e previsível:
cpf          CHAR(14)     NOT NULL,  -- '000.000.000-00' sempre 14 caracteres
cep          CHAR(9)      NOT NULL,  -- '00000-000' sempre 9 caracteres
sigla_estado CHAR(2)      NOT NULL,  -- 'SP', 'RJ', 'PR' sempre 2 caracteres

-- VARCHAR para texto de comprimento variável:
nome         VARCHAR(100) NOT NULL,
email        VARCHAR(255) NOT NULL,
descricao    VARCHAR(500)            -- nullable: sem NOT NULL
```

> ⚠️ **Armadilha do `VARCHAR` no MariaDB:** o limite de `VARCHAR(n)` é calculado em bytes, não em caracteres. Com `utf8mb4`, cada caractere pode ocupar até 4 bytes. Então `VARCHAR(255)` pode armazenar **no máximo 255 caracteres** apenas se todos forem ASCII — na prática, o limite real de bytes por linha impõe restrições. Para strings longas com caracteres especiais, prefira `TEXT`.

**PostgreSQL** não tem `TINYTEXT`, `MEDIUMTEXT` ou `LONGTEXT` — tudo é `TEXT` sem limite fixo (até o espaço disponível). O `VARCHAR` sem tamanho máximo em PostgreSQL é equivalente a `TEXT`.

### 5.4 Tipos de Data e Hora

| Tipo (MariaDB/MySQL) | Equivalente PostgreSQL | Formato | Faixa | Quando usar |
|---|---|---|---|---|
| `DATE` | `DATE` | `YYYY-MM-DD` | 1000-01-01 a 9999-12-31 | Datas sem hora (nascimento, vencimento) |
| `TIME` | `TIME` | `HHH:MM:SS` | -838:59:59 a 838:59:59 | Horários, durações |
| `DATETIME` | `TIMESTAMP` | `YYYY-MM-DD HH:MM:SS` | 1000 a 9999 | Timestamps sem fuso horário |
| `TIMESTAMP` | `TIMESTAMPTZ` | `YYYY-MM-DD HH:MM:SS` | 1970 a 2038 | Timestamps **com fuso** (armazena UTC) |
| `YEAR` | *(use INTEGER)* | `YYYY` | 1901 a 2155 | Apenas anos |

**`DATETIME` vs `TIMESTAMP` — a diferença crítica:** o `TIMESTAMP` armazena o valor convertido para UTC e o converte para o fuso do servidor ao retornar. Isso significa que o mesmo registro pode exibir horas diferentes dependendo da configuração de fuso do servidor — o que é ótimo para sistemas distribuídos, mas pode surpreender quem não sabe. O `DATETIME` armazena o valor exatamente como foi inserido, sem conversão de fuso.

**O problema do ano 2038 com `TIMESTAMP`:** o `TIMESTAMP` usa um inteiro de 32 bits contando segundos a partir de 1970-01-01 00:00:00 UTC. Esse contador estoura em 19 de janeiro de 2038. Para datas além disso, use `DATETIME` ou `BIGINT`.

```sql
-- Datas de eventos passados ou futuros distantes: use DATETIME
data_nascimento  DATE        NOT NULL,
data_vencimento  DATE        NOT NULL,

-- Registros automáticos de criação/atualização: TIMESTAMP é ideal
criado_em        TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP,
atualizado_em    TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP
                             ON UPDATE CURRENT_TIMESTAMP
```

### 5.5 Tipos Especiais

| Tipo (MariaDB/MySQL) | Equivalente PostgreSQL | Quando usar |
|---|---|---|
| `BOOLEAN` / `TINYINT(1)` | `BOOLEAN` | Verdadeiro/falso — no MariaDB, é apenas alias de TINYINT(1) |
| `ENUM('a','b','c')` | `CREATE TYPE ... AS ENUM` | Lista fechada de valores — use com moderação |
| `JSON` | `JSON` / `JSONB` | Dados semiestruturados — disponível no MariaDB 10.2+ |
| `BINARY(n)` / `VARBINARY(n)` | `BYTEA` | Dados binários, hashes |

> 💡 **Sobre `ENUM`:** embora conveniente, `ENUM` tem desvantagens sérias — adicionar um novo valor exige um `ALTER TABLE` (que pode travar a tabela em produção) e o valor não é portável entre SGBDs. Uma alternativa mais flexível é criar uma tabela de domínio (ex: `status_pedidos`) e usar uma FK.

---

## 6. Criando Tabelas — `CREATE TABLE`

Com o vocabulário de tipos de dados e convenções estabelecido, podemos agora criar tabelas de verdade. Vamos construir o schema de um sistema de e-commerce, seguindo todas as convenções desta disciplina.

### 6.1 Estrutura geral do `CREATE TABLE`

```sql
CREATE TABLE IF NOT EXISTS nome_tabela (
    -- Definição das colunas
    nome_coluna TIPO [constraints_inline],
    ...
    -- Constraints de tabela (PK, FK, UNIQUE compostos)
    [CONSTRAINT nome_constraint] TIPO_CONSTRAINT (coluna),
    ...
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

O `ENGINE=InnoDB` é o mecanismo de armazenamento que suporta **transações**, **chaves estrangeiras** e **bloqueios a nível de linha** — sem ele, as FKs são ignoradas silenciosamente. Sempre especifique `InnoDB` explicitamente.

> **Diferença MariaDB vs MySQL:** no MariaDB, o engine padrão também é InnoDB (ou Aria em alguns contextos), mas é boa prática sempre especificá-lo. No MySQL 8.0+, InnoDB é o padrão e a especificação também é considerada boa prática pela legibilidade.

### 6.2 Tabela `pessoas` — a base do sistema

Começamos pela tabela que vai servir de base para clientes e funcionários, demonstrando o padrão de auto-referência mencionado na regra 6:

```sql
-- Tabela base de pessoas físicas
-- Usada para clientes e funcionários via papéis (FKs semânticas)
CREATE TABLE IF NOT EXISTS pessoas (
    id_pessoa        INT UNSIGNED     NOT NULL AUTO_INCREMENT,
    nome             VARCHAR(100)     NOT NULL,
    cpf              CHAR(11)         NOT NULL,  -- apenas dígitos, sem pontuação
    email            VARCHAR(255)     NOT NULL,
    data_nascimento  DATE             NOT NULL,
    telefone         CHAR(11)             NULL,  -- nullable: nem todo cadastro tem
    criado_em        TIMESTAMP        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    atualizado_em    TIMESTAMP        NOT NULL DEFAULT CURRENT_TIMESTAMP
                                               ON UPDATE CURRENT_TIMESTAMP,

    -- Constraints de tabela: nomeadas para facilitar debugging
    CONSTRAINT pk_pessoa  PRIMARY KEY (id_pessoa),
    CONSTRAINT uq_cpf     UNIQUE      (cpf),
    CONSTRAINT uq_email   UNIQUE      (email)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci
  COMMENT='Cadastro base de pessoas físicas (clientes e funcionários)';
```

Observe algumas decisões de projeto aqui: o CPF é armazenado como `CHAR(11)` apenas com dígitos (sem pontos e traço), porque a formatação é responsabilidade da camada de apresentação, não do banco. O `COMMENT` na tabela documenta o propósito diretamente no schema — isso aparece em ferramentas como MySQL Workbench e DBeaver.

### 6.3 Tabela `categorias` — simples e autoexplicativa

```sql
CREATE TABLE IF NOT EXISTS categorias (
    id_categoria   INT UNSIGNED  NOT NULL AUTO_INCREMENT,
    nome           VARCHAR(80)   NOT NULL,
    descricao      TEXT              NULL,
    ativa          TINYINT(1)    NOT NULL DEFAULT 1,  -- 1=ativa, 0=inativa

    CONSTRAINT pk_categoria  PRIMARY KEY (id_categoria),
    CONSTRAINT uq_cat_nome   UNIQUE      (nome)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
```

### 6.4 Tabela `produtos` — com FK e CHECK

```sql
CREATE TABLE IF NOT EXISTS produtos (
    id_produto      INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    categoria_id    INT UNSIGNED    NOT NULL,             -- FK para categorias
    nome            VARCHAR(150)    NOT NULL,
    descricao       TEXT                NULL,
    preco           DECIMAL(10, 2)  NOT NULL,
    estoque         INT             NOT NULL DEFAULT 0,
    ativo           TINYINT(1)      NOT NULL DEFAULT 1,
    criado_em       TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT pk_produto         PRIMARY KEY (id_produto),
    CONSTRAINT fk_produto_categoria FOREIGN KEY (categoria_id)
                                  REFERENCES categorias (id_categoria)
                                  ON DELETE RESTRICT
                                  ON UPDATE CASCADE,
    CONSTRAINT ck_produto_preco   CHECK (preco >= 0),
    CONSTRAINT ck_produto_estoque CHECK (estoque >= 0)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
```

> ⚠️ **Diferença importante — `CHECK` no MariaDB vs MySQL:**
> - No **MySQL** até a versão 8.0.15, constraints `CHECK` eram **aceitas na sintaxe mas completamente ignoradas** — o banco não as validava. A partir do MySQL 8.0.16, passaram a funcionar.
> - No **MariaDB**, constraints `CHECK` funcionam corretamente **desde a versão 10.2.1** (lançada em 2016).
> - Como usamos MariaDB via XAMPP, as constraints `CHECK` funcionam normalmente — mas sempre verifique a versão com `SELECT VERSION()` se tiver dúvidas.

### 6.5 Tabela `pedidos` — com FKs semânticas duplas

Este é o exemplo que demonstra a Regra 6 das convenções — duas FKs que referenciam a mesma tabela `pessoas`, mas com papéis diferentes:

```sql
CREATE TABLE IF NOT EXISTS pedidos (
    id_pedido       INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    cliente_id      INT UNSIGNED    NOT NULL,  -- pessoa no papel de cliente
    funcionario_id  INT UNSIGNED        NULL,  -- pessoa no papel de atendente (pode ser nulo: venda online)
    data_pedido     TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status          ENUM(
                        'pendente',
                        'confirmado',
                        'em_separacao',
                        'enviado',
                        'entregue',
                        'cancelado'
                    )               NOT NULL DEFAULT 'pendente',
    valor_total     DECIMAL(12, 2)  NOT NULL DEFAULT 0.00,
    observacoes     TEXT                NULL,

    CONSTRAINT pk_pedido          PRIMARY KEY (id_pedido),

    -- FKs com nomes semânticos: indicam o PAPEL de cada pessoa
    CONSTRAINT fk_pedido_cliente      FOREIGN KEY (cliente_id)
                                      REFERENCES pessoas (id_pessoa)
                                      ON DELETE RESTRICT
                                      ON UPDATE CASCADE,

    CONSTRAINT fk_pedido_funcionario  FOREIGN KEY (funcionario_id)
                                      REFERENCES pessoas (id_pessoa)
                                      ON DELETE SET NULL
                                      ON UPDATE CASCADE,

    CONSTRAINT ck_pedido_valor CHECK (valor_total >= 0)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
```

### 6.6 Tabela `itens_pedidos` — chave composta e relacionamento N:M

```sql
-- Tabela que resolve o relacionamento N:M entre pedidos e produtos
-- Um pedido pode conter muitos produtos; um produto pode estar em muitos pedidos
CREATE TABLE IF NOT EXISTS itens_pedidos (
    pedido_id       INT UNSIGNED    NOT NULL,
    produto_id      INT UNSIGNED    NOT NULL,
    quantidade      INT UNSIGNED    NOT NULL,
    preco_unitario  DECIMAL(10, 2)  NOT NULL,  -- snapshot do preço no momento da compra
    desconto        DECIMAL(5, 2)   NOT NULL DEFAULT 0.00,

    -- Chave primária composta pelas duas FKs
    CONSTRAINT pk_item_pedido PRIMARY KEY (pedido_id, produto_id),

    CONSTRAINT fk_item_pedido   FOREIGN KEY (pedido_id)
                                REFERENCES pedidos (id_pedido)
                                ON DELETE CASCADE    -- excluir pedido exclui seus itens
                                ON UPDATE CASCADE,

    CONSTRAINT fk_item_produto  FOREIGN KEY (produto_id)
                                REFERENCES produtos (id_produto)
                                ON DELETE RESTRICT  -- não deixa excluir produto que está em pedido
                                ON UPDATE CASCADE,

    CONSTRAINT ck_item_qtd      CHECK (quantidade > 0),
    CONSTRAINT ck_item_preco    CHECK (preco_unitario >= 0),
    CONSTRAINT ck_item_desc     CHECK (desconto >= 0 AND desconto <= 100)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
```

> 💡 **Por que armazenar `preco_unitario` na tabela de itens?** Porque o preço do produto pode mudar depois que o pedido foi feito. Se você armazenar apenas a FK do produto e consultar o preço atual, o valor do pedido histórico mudaria retroativamente — um erro grave em qualquer sistema comercial. O `preco_unitario` é um *snapshot* (fotografia) do preço no momento da compra.

---

## 7. Constraints em Detalhe

### 7.1 PRIMARY KEY

A chave primária identifica unicamente cada linha da tabela. Ela implica automaticamente `NOT NULL` e `UNIQUE`. Pode ser simples (uma coluna) ou composta (múltiplas colunas).

```sql
-- PK simples — forma inline (para tabelas com PK de uma coluna):
id_cliente INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY

-- PK simples — forma de constraint nomeada (recomendada — facilita ALTER TABLE):
CONSTRAINT pk_cliente PRIMARY KEY (id_cliente)

-- PK composta — apenas como constraint de tabela:
CONSTRAINT pk_item PRIMARY KEY (id_pedido, id_produto)
```

**`AUTO_INCREMENT` no MariaDB vs PostgreSQL:** no MariaDB/MySQL, usa-se `AUTO_INCREMENT` diretamente no tipo. No PostgreSQL, o equivalente é o tipo `SERIAL` (ou `BIGSERIAL` para BIGINT), que internamente cria uma sequence:

```sql
-- MariaDB/MySQL:
id_cliente INT UNSIGNED NOT NULL AUTO_INCREMENT

-- PostgreSQL equivalente:
id_cliente SERIAL PRIMARY KEY
-- ou mais explicitamente no PostgreSQL moderno:
id_cliente INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

### 7.2 FOREIGN KEY com ON DELETE e ON UPDATE

A FK garante a **integridade referencial** — impede que exista um valor na coluna filho que não exista na coluna pai. As ações `ON DELETE` e `ON UPDATE` definem o comportamento quando o registro pai é excluído ou tem sua PK alterada.

| Ação | Comportamento | Quando usar |
|---|---|---|
| `RESTRICT` | Impede a operação no pai se houver filhos | Padrão mais seguro — use quando a exclusão deve ser explícita |
| `NO ACTION` | Similar ao RESTRICT (verificado ao final da transação) | Padrão do SQL — comportamento igual ao RESTRICT no MariaDB |
| `CASCADE` | Propaga a operação para os filhos | Itens de pedido, dependentes — quando o filho não faz sentido sem o pai |
| `SET NULL` | Define a FK como NULL nos filhos | Quando a associação é opcional (ex: funcionário demitido) |
| `SET DEFAULT` | Define a FK como seu valor DEFAULT | Raro — pouco suportado na prática |

```sql
-- Exemplos comentados de cada cenário:

-- CASCADE em DELETE: excluir um pedido remove seus itens automaticamente
CONSTRAINT fk_item_pedido FOREIGN KEY (id_pedido)
    REFERENCES pedidos (id_pedido)
    ON DELETE CASCADE
    ON UPDATE CASCADE,

-- RESTRICT em DELETE: não deixa excluir uma categoria que tem produtos
CONSTRAINT fk_produto_categoria FOREIGN KEY (id_categoria)
    REFERENCES categorias (id_categoria)
    ON DELETE RESTRICT
    ON UPDATE CASCADE,

-- SET NULL em DELETE: se o funcionário for desligado, os pedidos
-- que ele atendeu ficam sem atendente (funcionario_id = NULL)
CONSTRAINT fk_pedido_funcionario FOREIGN KEY (funcionario_id)
    REFERENCES pessoas (id_pessoa)
    ON DELETE SET NULL
    ON UPDATE CASCADE
```

> ⚠️ **Diferença MariaDB vs MySQL — verificação de FK:**
> No **MariaDB**, é possível temporariamente desabilitar a verificação de FK com `SET FOREIGN_KEY_CHECKS = 0` para operações de carga em massa ou scripts de setup — mas lembre-se de reabilitar com `SET FOREIGN_KEY_CHECKS = 1` depois. No **MySQL**, o comportamento é idêntico. Em ambos, desabilitar FKs durante carga e reabilitar sem verificar a consistência pode deixar o banco em estado inválido — use com cuidado.

### 7.3 NOT NULL e DEFAULT

`NOT NULL` impede que uma coluna armazene o valor `NULL`. `DEFAULT` define o valor aplicado quando nenhum é fornecido no `INSERT`.

```sql
-- NOT NULL sem DEFAULT: o INSERT deve sempre fornecer um valor
nome VARCHAR(100) NOT NULL,

-- NOT NULL com DEFAULT: se não fornecido, usa o valor padrão
ativo        TINYINT(1)  NOT NULL DEFAULT 1,
criado_em    TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP,
estoque      INT         NOT NULL DEFAULT 0,

-- NULL explícito: o campo é opcional
observacoes  TEXT NULL,
-- ou simplesmente:
observacoes  TEXT,  -- NULL é o padrão quando não especificado
```

> 💡 **Sobre `NULL` em bancos de dados:** `NULL` não significa zero, string vazia ou falso — significa *ausência de valor* ou *valor desconhecido*. Isso tem implicações em consultas: `NULL = NULL` é `FALSE` em SQL (use `IS NULL` ou `IS NOT NULL` para comparar). Colunas que permitem NULL aumentam a complexidade das queries, então use `NOT NULL` sempre que possível.

### 7.4 UNIQUE e CHECK

`UNIQUE` garante que não existam dois registros com o mesmo valor em uma coluna (ou combinação de colunas). Diferente da PK, uma coluna com `UNIQUE` pode conter `NULL` — e múltiplos `NULL`s são permitidos (porque `NULL ≠ NULL` em SQL).

```sql
-- UNIQUE em coluna única:
CONSTRAINT uq_pessoa_cpf   UNIQUE (cpf),
CONSTRAINT uq_pessoa_email UNIQUE (email),

-- UNIQUE composto: a combinação das duas colunas deve ser única
-- (cada aluno pode estar matriculado em cada disciplina apenas uma vez por semestre)
CONSTRAINT uq_matricula UNIQUE (id_aluno, id_disciplina, semestre)
```

`CHECK` valida se os valores inseridos satisfazem uma expressão booleana. Qualquer linha que viole o CHECK é rejeitada:

```sql
-- Restrições de domínio com CHECK:
CONSTRAINT ck_preco_positivo   CHECK (preco >= 0),
CONSTRAINT ck_desconto_valido  CHECK (desconto >= 0 AND desconto <= 100),
CONSTRAINT ck_nota_valida      CHECK (nota >= 0 AND nota <= 10),
CONSTRAINT ck_email_formato    CHECK (email LIKE '%@%.%'),  -- validação básica de formato
CONSTRAINT ck_data_valida      CHECK (data_fim >= data_inicio)
```

---

## 8. Modificando Tabelas — `ALTER TABLE`

O `ALTER TABLE` permite modificar a estrutura de uma tabela existente sem perder os dados. É o comando que você usará quando os requisitos do sistema mudarem depois que o banco já foi populado.

```sql
-- Adicionar uma nova coluna ao final da tabela:
ALTER TABLE produtos
    ADD COLUMN peso DECIMAL(8, 3) NULL COMMENT 'Peso em quilogramas';

-- Adicionar coluna em posição específica (MariaDB/MySQL — não existe no PostgreSQL):
ALTER TABLE produtos
    ADD COLUMN codigo_barras VARCHAR(13) NULL AFTER nome;

-- Adicionar coluna como a primeira da tabela:
ALTER TABLE produtos
    ADD COLUMN codigo_interno VARCHAR(20) NOT NULL FIRST;

-- Modificar o tipo ou constraints de uma coluna existente:
-- MODIFY (MariaDB/MySQL) — redefine a coluna inteira
ALTER TABLE pessoas
    MODIFY COLUMN telefone VARCHAR(20) NULL;

-- CHANGE (MariaDB/MySQL) — renomeia E redefine a coluna
-- Sintaxe: CHANGE nome_antigo novo_nome tipo_novo [constraints]
ALTER TABLE pessoas
    CHANGE COLUMN telefone celular CHAR(11) NULL;

-- Remover uma coluna:
ALTER TABLE produtos
    DROP COLUMN codigo_interno;

-- Adicionar uma constraint FK após a criação da tabela:
ALTER TABLE produtos
    ADD CONSTRAINT fk_produto_fornecedor
        FOREIGN KEY (fornecedor_id)
        REFERENCES fornecedores (id_fornecedor)
        ON DELETE RESTRICT
        ON UPDATE CASCADE;

-- Remover uma constraint pelo nome:
ALTER TABLE produtos
    DROP FOREIGN KEY fk_produto_fornecedor;

ALTER TABLE pessoas
    DROP INDEX uq_email;  -- UNIQUE é armazenado como índice no MariaDB/MySQL

-- Renomear a tabela:
ALTER TABLE itens_pedidos RENAME TO itens_pedido;

-- Equivalente no MariaDB (sintaxe alternativa):
-- RENAME TABLE itens_pedidos TO itens_pedido;
```

> ⚠️ **Cuidado com `ALTER TABLE` em tabelas com dados:** modificar o tipo de uma coluna que já tem dados pode causar truncamento ou conversão implícita. Por exemplo, reduzir um `VARCHAR(255)` para `VARCHAR(50)` truncará silenciosamente valores maiores que 50 caracteres em algumas versões. Sempre faça backup antes de executar `ALTER TABLE` em produção.

**Diferença MariaDB/MySQL vs PostgreSQL no `ALTER TABLE`:**

```sql
-- No MariaDB/MySQL, MODIFY e CHANGE são exclusivos:
ALTER TABLE pessoas MODIFY COLUMN nome VARCHAR(150) NOT NULL;

-- No PostgreSQL, usa-se ALTER COLUMN com subcláusulas específicas:
ALTER TABLE pessoas ALTER COLUMN nome TYPE VARCHAR(150);
ALTER TABLE pessoas ALTER COLUMN nome SET NOT NULL;
ALTER TABLE pessoas ALTER COLUMN nome SET DEFAULT 'Não informado';
ALTER TABLE pessoas RENAME COLUMN telefone TO celular;
```

---

## 9. Removendo Objetos — `DROP`

O `DROP` remove permanentemente um objeto do banco. **Não há desfazer** — sempre faça backup antes.

```sql
-- Remover uma tabela (e todos os seus dados e índices):
DROP TABLE IF EXISTS itens_pedidos;

-- Remover múltiplas tabelas de uma vez — atenção à ordem (filhos antes dos pais):
DROP TABLE IF EXISTS itens_pedidos, pedidos, produtos, categorias, pessoas;

-- Remover o banco de dados inteiro:
DROP DATABASE IF EXISTS loja_virtual;

-- Remover o schema (MariaDB/MySQL — equivalente a DROP DATABASE):
DROP SCHEMA IF EXISTS loja_virtual;
```

**O problema da ordem ao dropar:** se você tentar remover uma tabela que é referenciada por uma FK em outra tabela, o MariaDB/MySQL rejeitará a operação. Você deve sempre dropar os "filhos" antes dos "pais", ou desabilitar temporariamente a verificação de FK:

```sql
-- Sequência correta para dropar o schema de e-commerce:
SET FOREIGN_KEY_CHECKS = 0;  -- desabilita verificação temporariamente

DROP TABLE IF EXISTS itens_pedidos;  -- filhos primeiro
DROP TABLE IF EXISTS pedidos;
DROP TABLE IF EXISTS produtos;
DROP TABLE IF EXISTS categorias;
DROP TABLE IF EXISTS pessoas;

SET FOREIGN_KEY_CHECKS = 1;  -- sempre reabilite
```

---

## 10. Comandos Utilitários Essenciais

Estes comandos não são DDL propriamente ditos, mas você os usará constantemente durante o desenvolvimento:

```sql
-- Listar todos os databases disponíveis:
SHOW DATABASES;

-- Selecionar o banco para uso:
USE loja_virtual;

-- Verificar qual banco está selecionado:
SELECT DATABASE();

-- Listar tabelas do banco atual:
SHOW TABLES;

-- Ver a estrutura de uma tabela:
DESCRIBE produtos;
-- ou:
DESC produtos;

-- Ver o CREATE TABLE completo (com todas as constraints como foram definidas):
SHOW CREATE TABLE produtos;

-- Ver todos os índices de uma tabela (PKs, UNIQUEs, FKs geram índices):
SHOW INDEX FROM produtos;

-- Ver as constraints de FK de uma tabela específica (usando information_schema):
SELECT
    constraint_name,
    column_name,
    referenced_table_name,
    referenced_column_name
FROM
    information_schema.key_column_usage
WHERE
    table_schema    = 'loja_virtual'
    AND table_name  = 'produtos'
    AND referenced_table_name IS NOT NULL;
```

---

## 11. Script Completo — Sistema de E-commerce

O script abaixo reúne tudo que foi apresentado nesta aula em um arquivo coeso, seguindo todas as convenções. Este é o padrão esperado para as entregas da disciplina:

```sql
-- =============================================================================
-- Schema: loja_virtual
-- Disciplina: IBD015 — Banco de Dados Relacional
-- Professor: Ronan Adriel Zenatti — FATEC Jahu
-- Descrição: DDL completo de um sistema de e-commerce simplificado
-- SGBD: MariaDB 10.4+ (XAMPP)
-- =============================================================================

-- Remove o banco se existir e recria do zero (útil em desenvolvimento)
DROP DATABASE IF EXISTS loja_virtual;

CREATE DATABASE loja_virtual
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE loja_virtual;

-- -----------------------------------------------------------------------------
-- Tabela: pessoas
-- Cadastro base de pessoas físicas — clientes e funcionários compartilham esta tabela
-- -----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS pessoas (
    id_pessoa        INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    nome             VARCHAR(100)    NOT NULL,
    cpf              CHAR(11)        NOT NULL COMMENT 'Apenas dígitos, sem formatação',
    email            VARCHAR(255)    NOT NULL,
    data_nascimento  DATE            NOT NULL,
    telefone         CHAR(11)            NULL COMMENT 'Apenas dígitos',
    criado_em        TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    atualizado_em    TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP
                                             ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT pk_pessoa   PRIMARY KEY (id_pessoa),
    CONSTRAINT uq_cpf      UNIQUE (cpf),
    CONSTRAINT uq_email    UNIQUE (email)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci
  COMMENT='Cadastro base de pessoas físicas';

-- -----------------------------------------------------------------------------
-- Tabela: enderecos
-- Um relacionamento 1:N: uma pessoa pode ter múltiplos endereços
-- -----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS enderecos (
    id_endereco  INT UNSIGNED  NOT NULL AUTO_INCREMENT,
    pessoa_id    INT UNSIGNED  NOT NULL,
    logradouro   VARCHAR(150)  NOT NULL,
    numero       VARCHAR(10)   NOT NULL,
    complemento  VARCHAR(50)       NULL,
    bairro       VARCHAR(80)   NOT NULL,
    cidade       VARCHAR(80)   NOT NULL,
    estado       CHAR(2)       NOT NULL,
    cep          CHAR(8)       NOT NULL COMMENT 'Apenas dígitos',
    principal    TINYINT(1)    NOT NULL DEFAULT 0 COMMENT '1 = endereço principal',

    CONSTRAINT pk_endereco       PRIMARY KEY (id_endereco),
    CONSTRAINT fk_endereco_pessoa FOREIGN KEY (pessoa_id)
                                  REFERENCES pessoas (id_pessoa)
                                  ON DELETE CASCADE
                                  ON UPDATE CASCADE,
    CONSTRAINT ck_estado         CHECK (LENGTH(estado) = 2)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- Tabela: categorias
-- -----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS categorias (
    id_categoria  INT UNSIGNED  NOT NULL AUTO_INCREMENT,
    nome          VARCHAR(80)   NOT NULL,
    descricao     TEXT              NULL,
    ativa         TINYINT(1)    NOT NULL DEFAULT 1,

    CONSTRAINT pk_categoria  PRIMARY KEY (id_categoria),
    CONSTRAINT uq_cat_nome   UNIQUE (nome)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- Tabela: produtos
-- -----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS produtos (
    id_produto    INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    categoria_id  INT UNSIGNED    NOT NULL,
    nome          VARCHAR(150)    NOT NULL,
    descricao     TEXT                NULL,
    preco         DECIMAL(10, 2)  NOT NULL,
    estoque       INT             NOT NULL DEFAULT 0,
    ativo         TINYINT(1)      NOT NULL DEFAULT 1,
    criado_em     TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    atualizado_em TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP
                                           ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT pk_produto          PRIMARY KEY (id_produto),
    CONSTRAINT fk_produto_categoria FOREIGN KEY (categoria_id)
                                    REFERENCES categorias (id_categoria)
                                    ON DELETE RESTRICT
                                    ON UPDATE CASCADE,
    CONSTRAINT ck_preco            CHECK (preco >= 0),
    CONSTRAINT ck_estoque          CHECK (estoque >= 0)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- Tabela: pedidos
-- Demonstra FKs semânticas duplas referenciando a mesma tabela (pessoas)
-- -----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS pedidos (
    id_pedido      INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    cliente_id     INT UNSIGNED    NOT NULL,   -- pessoa no papel de cliente
    funcionario_id INT UNSIGNED        NULL,   -- pessoa no papel de atendente
    endereco_id    INT UNSIGNED        NULL,   -- endereço de entrega
    data_pedido    TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status         ENUM(
                       'pendente',
                       'confirmado',
                       'em_separacao',
                       'enviado',
                       'entregue',
                       'cancelado'
                   )               NOT NULL DEFAULT 'pendente',
    valor_total    DECIMAL(12, 2)  NOT NULL DEFAULT 0.00,
    observacoes    TEXT                NULL,

    CONSTRAINT pk_pedido             PRIMARY KEY (id_pedido),
    CONSTRAINT fk_pedido_cliente     FOREIGN KEY (cliente_id)
                                     REFERENCES pessoas (id_pessoa)
                                     ON DELETE RESTRICT
                                     ON UPDATE CASCADE,
    CONSTRAINT fk_pedido_funcionario FOREIGN KEY (funcionario_id)
                                     REFERENCES pessoas (id_pessoa)
                                     ON DELETE SET NULL
                                     ON UPDATE CASCADE,
    CONSTRAINT fk_pedido_endereco    FOREIGN KEY (endereco_id)
                                     REFERENCES enderecos (id_endereco)
                                     ON DELETE SET NULL
                                     ON UPDATE CASCADE,
    CONSTRAINT ck_valor_total        CHECK (valor_total >= 0)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;

-- -----------------------------------------------------------------------------
-- Tabela: itens_pedidos
-- Resolve o N:M entre pedidos e produtos
-- Armazena snapshot do preço no momento da compra
-- -----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS itens_pedidos (
    pedido_id      INT UNSIGNED    NOT NULL,
    produto_id     INT UNSIGNED    NOT NULL,
    quantidade     INT UNSIGNED    NOT NULL,
    preco_unitario DECIMAL(10, 2)  NOT NULL COMMENT 'Preço no momento da compra',
    desconto       DECIMAL(5, 2)   NOT NULL DEFAULT 0.00 COMMENT 'Percentual de desconto',

    CONSTRAINT pk_item_pedido   PRIMARY KEY (pedido_id, produto_id),
    CONSTRAINT fk_item_pedido   FOREIGN KEY (pedido_id)
                                REFERENCES pedidos (id_pedido)
                                ON DELETE CASCADE
                                ON UPDATE CASCADE,
    CONSTRAINT fk_item_produto  FOREIGN KEY (produto_id)
                                REFERENCES produtos (id_produto)
                                ON DELETE RESTRICT
                                ON UPDATE CASCADE,
    CONSTRAINT ck_item_qtd      CHECK (quantidade > 0),
    CONSTRAINT ck_item_preco    CHECK (preco_unitario >= 0),
    CONSTRAINT ck_item_desconto CHECK (desconto >= 0 AND desconto <= 100)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
```

---

## 12. Exercícios de Fixação com Gabarito

**Exercício 1 — Identifique os erros:** o trecho abaixo possui violações das convenções desta disciplina. Liste todos os erros encontrados e reescreva o código corretamente.

```sql
CREATE TABLE Produto (
    idProduto INT(11) PRIMARY KEY,
    NomeProduto varchar(100),
    Preco FLOAT,
    ID_CATEGORIA int,
    FOREIGN KEY (ID_CATEGORIA) REFERENCES Categoria(id)
);
```

**Gabarito:** os erros são: nome da tabela em singular e com inicial maiúscula (deve ser `produtos`); `idProduto` usa camelCase (deve ser `id_produto`); `INT(11)` está depreciado (use `INT`); `NomeProduto` mistura maiúsculas (deve ser `nome`); `Preco` com maiúscula (deve ser `preco`); `FLOAT` inapropriado para preço (use `DECIMAL(10,2)`); `ID_CATEGORIA` mistura maiúsculas (deve ser `categoria_id` conforme Regra 6); o nome da FK não segue o padrão semântico; faltam `NOT NULL` nas colunas obrigatórias; falta `ENGINE=InnoDB` e `DEFAULT CHARSET`.

Versão corrigida:

```sql
CREATE TABLE IF NOT EXISTS produtos (
    id_produto   INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    categoria_id INT UNSIGNED    NOT NULL,
    nome         VARCHAR(100)    NOT NULL,
    preco        DECIMAL(10, 2)  NOT NULL,

    CONSTRAINT pk_produto          PRIMARY KEY (id_produto),
    CONSTRAINT fk_produto_categoria FOREIGN KEY (categoria_id)
                                    REFERENCES categorias (id_categoria)
                                    ON DELETE RESTRICT
                                    ON UPDATE CASCADE,
    CONSTRAINT ck_preco            CHECK (preco >= 0)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
```

**Exercício 2 — Criação guiada:** crie o DDL completo para um sistema de biblioteca com as entidades: `autores`, `livros`, `usuarios`, `emprestimos`. Siga todas as convenções. Um livro pode ter múltiplos autores; um usuário pode ter múltiplos empréstimos; cada empréstimo é de um único livro.

**Exercício 3 — ALTER TABLE:** dada a tabela `produtos` criada nesta aula, escreva os comandos `ALTER TABLE` para: (a) adicionar a coluna `peso DECIMAL(8, 3) NULL`; (b) renomear a coluna `ativo` para `disponivel` mantendo o mesmo tipo; (c) adicionar uma constraint CHECK que garanta que `peso > 0` quando não for NULL.

---

## 📚 Referências desta Aula

- ELMASRI, R.; NAVATHE, S. B. *Sistemas de Banco de Dados*. 7 ed. Cap. 6 — SQL Básico. São Paulo: Pearson, 2018.
- Documentação oficial do MariaDB — [CREATE TABLE](https://mariadb.com/kb/en/create-table/)
- Documentação oficial do MariaDB — [Data Types](https://mariadb.com/kb/en/data-types/)
- Documentação oficial do PostgreSQL — [Data Types](https://www.postgresql.org/docs/current/datatype.html)

---

> **Próxima aula:** na [Aula 04 — SQL DML](./Aula_04_SQL_DML.md), vamos popular as tabelas que criamos aqui usando `INSERT`, `UPDATE` e `DELETE`, e aprender sobre o controle básico de transações com `BEGIN`, `COMMIT` e `ROLLBACK`.

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
