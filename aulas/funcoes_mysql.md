# Referência de Funções — MariaDB / MySQL

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> Material de apoio complementar às aulas · [Voltar ao README](./README.md)

---

## Sobre este documento

Este arquivo documenta as principais funções nativas do MariaDB/MySQL, organizadas por categoria. Para cada função são apresentados: descrição, restrições, tabela de parâmetros e três exemplos práticos usando o banco de dados **Sakila** — o banco de exemplo oficial da Oracle/MySQL, que simula o sistema de uma locadora de filmes.

As funções cobertas representam a curadoria das mais utilizadas no mercado e as que aparecem nas aulas da disciplina. O MariaDB possui mais de 300 funções nativas; as aqui documentadas cobrem aproximadamente 90% dos casos de uso em projetos reais.

> ⚠️ **Diferenças MariaDB vs MySQL** são sinalizadas com o ícone ⚠️ ao longo do documento. Para fins práticos nas aulas (XAMPP com MariaDB 10.4+), o comportamento padrão descrito é sempre o do MariaDB.

---

## Instalando o Sakila

```bash
# Baixe o banco Sakila em: https://dev.mysql.com/doc/index-other.html
# Ou diretamente:
# https://downloads.mysql.com/docs/sakila-db.zip

# Após extrair, importe no MariaDB via terminal:
mysql -u root -p < sakila-schema.sql
mysql -u root -p < sakila-data.sql

# Verifique:
mysql -u root -p -e "SHOW DATABASES LIKE 'sakila';"
```

```sql
-- Dentro do MariaDB, após importar:
USE sakila;
SHOW TABLES;
-- Principais tabelas usadas nos exemplos:
-- actor, film, film_actor, customer, rental, payment,
-- store, staff, category, film_category, address, city, country
```

---

## Índice de Categorias

1. [Funções de String](#1-funções-de-string)
2. [Funções Numéricas](#2-funções-numéricas)
3. [Funções de Data e Hora](#3-funções-de-data-e-hora)
4. [Funções de Agregação](#4-funções-de-agregação)
5. [Funções Condicionais](#5-funções-condicionais)
6. [Funções de Conversão de Tipo](#6-funções-de-conversão-de-tipo)
7. [Funções JSON](#7-funções-json)
8. [Funções de Criptografia e Hash](#8-funções-de-criptografia-e-hash)
9. [Funções de Informação do Sistema](#9-funções-de-informação-do-sistema)
10. [Window Functions](#10-window-functions)

---
## 1. Funções de String

---

### `CONCAT()`

**Descrição:** concatena dois ou mais valores em uma única string. Se qualquer argumento for `NULL`, o resultado é `NULL` (use `CONCAT_WS` para ignorar NULLs).

**Restrições:** mínimo de 2 argumentos. Valores numéricos são convertidos implicitamente para string.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str1` | ANY | ✅ | Primeiro valor a concatenar |
| `str2` | ANY | ✅ | Segundo valor a concatenar |
| `...strN` | ANY | ❌ | Valores adicionais (ilimitados) |

```sql
-- Exemplo 1: nome completo do ator
SELECT CONCAT(first_name, ' ', last_name) AS nome_completo
FROM   actor
ORDER BY last_name
LIMIT 5;

-- Exemplo 2: título do filme com seu rating entre parênteses
SELECT CONCAT(title, ' (', rating, ')') AS titulo_classificado
FROM   film
WHERE  rating IN ('G', 'PG')
ORDER BY title
LIMIT 5;

-- Exemplo 3: endereço completo do cliente
SELECT CONCAT(a.address, ', ', ci.city, ' - ', co.country) AS endereco_completo
FROM   customer c
JOIN   address  a  ON a.address_id  = c.address_id
JOIN   city     ci ON ci.city_id    = a.city_id
JOIN   country  co ON co.country_id = ci.country_id
LIMIT 5;
```

---

### `CONCAT_WS()`

**Descrição:** concatena valores usando um separador (*With Separator*). Diferente de `CONCAT()`, **ignora automaticamente valores NULL** — o separador não é duplicado para NULLs.

**Restrições:** o primeiro argumento obrigatoriamente é o separador. NULLs nos demais argumentos são silenciosamente ignorados.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `separator` | STRING | ✅ | Separador a ser inserido entre os valores |
| `str1` | ANY | ✅ | Primeiro valor |
| `str2` | ANY | ✅ | Segundo valor |
| `...strN` | ANY | ❌ | Valores adicionais |

```sql
-- Exemplo 1: nome completo com separador espaço (mais limpo que CONCAT para nomes)
SELECT CONCAT_WS(' ', first_name, last_name) AS nome
FROM   actor LIMIT 5;

-- Exemplo 2: endereço com separador ' | ', ignorando address2 quando NULL
SELECT CONCAT_WS(' | ', address, address2, district, postal_code) AS endereco
FROM   address
WHERE  address2 IS NOT NULL
LIMIT 5;

-- Exemplo 3: lista de categorias de um filme (pré-GROUP_CONCAT)
SELECT f.title,
       CONCAT_WS(', ', c1.name, c2.name) AS categorias
FROM   film f
JOIN   film_category fc ON fc.film_id   = f.film_id
JOIN   category      c1 ON c1.category_id = fc.category_id
LEFT JOIN category   c2 ON c2.category_id = fc.category_id AND c2.category_id <> c1.category_id
LIMIT 5;
```

---

### `LENGTH()` / `CHAR_LENGTH()`

**Descrição:** `LENGTH()` retorna o número de **bytes** de uma string; `CHAR_LENGTH()` (ou `CHARACTER_LENGTH()`) retorna o número de **caracteres**. Para strings ASCII puras, o resultado é igual. Para caracteres multibyte (acentos, emojis), diferem.

**Restrições:** retorna `NULL` se o argumento for `NULL`.

> ⚠️ **MariaDB vs MySQL:** em ambos, `LENGTH()` conta bytes e `CHAR_LENGTH()` conta caracteres. Para texto em português com acentos em `utf8mb4`, sempre prefira `CHAR_LENGTH()` para contar caracteres reais.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String a ser medida |

```sql
-- Exemplo 1: filmes com título mais longo que 20 caracteres
SELECT title, CHAR_LENGTH(title) AS tamanho
FROM   film
WHERE  CHAR_LENGTH(title) > 20
ORDER BY tamanho DESC
LIMIT 5;

-- Exemplo 2: comparar LENGTH e CHAR_LENGTH (igual para ASCII)
SELECT title,
       LENGTH(title)      AS bytes,
       CHAR_LENGTH(title) AS caracteres
FROM   film
ORDER BY title
LIMIT 5;

-- Exemplo 3: descrições com menos de 50 caracteres
SELECT film_id, title, CHAR_LENGTH(description) AS tamanho_desc
FROM   film
WHERE  description IS NOT NULL
ORDER BY tamanho_desc
LIMIT 5;
```

---

### `SUBSTRING()` / `SUBSTR()`

**Descrição:** extrai uma parte de uma string a partir de uma posição, com comprimento opcional. A posição começa em 1 (não em 0). Posições negativas contam a partir do fim da string.

**Restrições:** se `pos` exceder o tamanho da string, retorna string vazia. Se `len` for negativo, retorna string vazia.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String de origem |
| `pos` | INT | ✅ | Posição inicial (começa em 1; negativo = conta do fim) |
| `len` | INT | ❌ | Quantidade de caracteres a extrair (omitir = até o fim) |

```sql
-- Exemplo 1: iniciais dos atores (primeiro caractere de cada nome)
SELECT CONCAT(SUBSTRING(first_name, 1, 1), '.', SUBSTRING(last_name, 1, 1), '.') AS iniciais,
       first_name, last_name
FROM   actor
LIMIT 10;

-- Exemplo 2: ano de lançamento como string (primeiros 4 chars de release_year)
SELECT title, SUBSTRING(CAST(release_year AS CHAR), 1, 4) AS ano
FROM   film
LIMIT 5;

-- Exemplo 3: últimos 4 dígitos do postal_code
SELECT address, SUBSTRING(postal_code, -4) AS cep_final
FROM   address
WHERE  postal_code IS NOT NULL
LIMIT 5;
```

---

### `UPPER()` / `LOWER()`

**Descrição:** converte todos os caracteres de uma string para maiúsculas (`UPPER`) ou minúsculas (`LOWER`). Respeita o collation da coluna para caracteres especiais.

**Restrições:** retorna `NULL` se o argumento for `NULL`. Números e caracteres não-alfabéticos são retornados sem alteração.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String a converter |

```sql
-- Exemplo 1: nomes de atores em maiúsculas (já são maiúsculas no Sakila, mas ilustrativo)
SELECT UPPER(first_name) AS nome, LOWER(last_name) AS sobrenome
FROM   actor LIMIT 5;

-- Exemplo 2: busca case-insensitive manual (sem usar collation)
SELECT title FROM film
WHERE  UPPER(title) LIKE UPPER('%love%')
LIMIT 5;

-- Exemplo 3: padronizar emails de clientes para minúsculas
SELECT LOWER(CONCAT(first_name, '.', last_name, '@sakila.com')) AS email_padrao
FROM   customer LIMIT 5;
```

---

### `TRIM()` / `LTRIM()` / `RTRIM()`

**Descrição:** remove espaços (ou caracteres específicos) das extremidades de uma string. `LTRIM` remove apenas à esquerda, `RTRIM` apenas à direita, `TRIM` nos dois lados.

**Restrições:** por padrão remove espaços. `TRIM` aceita especificação de caractere e lado (`LEADING`, `TRAILING`, `BOTH`).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `[remchars FROM]` | STRING | ❌ | Caractere(s) a remover (padrão: espaço) |
| `str` | STRING | ✅ | String a processar |

```sql
-- Exemplo 1: remover espaços extras de endereços
SELECT TRIM(address) AS endereco_limpo FROM address LIMIT 5;

-- Exemplo 2: remover ponto final do título (se houver)
SELECT TRIM(TRAILING '.' FROM title) AS titulo_sem_ponto
FROM   film LIMIT 5;

-- Exemplo 3: remover zeros à esquerda de postal_code
SELECT TRIM(LEADING '0' FROM postal_code) AS cep_sem_zeros
FROM   address WHERE postal_code IS NOT NULL LIMIT 5;
```

---

### `REPLACE()`

**Descrição:** substitui todas as ocorrências de uma substring por outra dentro de uma string. A busca é **case-sensitive** (diferencia maiúsculas de minúsculas).

**Restrições:** substitui **todas** as ocorrências (não apenas a primeira). Retorna `NULL` se qualquer argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String original |
| `from_str` | STRING | ✅ | Substring a ser substituída |
| `to_str` | STRING | ✅ | Substring substituta (string vazia para remover) |

```sql
-- Exemplo 1: substituir 'ACADEMY' por 'ACADEMIA' nos títulos
SELECT REPLACE(title, 'ACADEMY', 'ACADEMIA') AS titulo_pt
FROM   film WHERE title LIKE '%ACADEMY%';

-- Exemplo 2: remover espaços do postal_code
SELECT REPLACE(postal_code, ' ', '') AS cep_sem_espacos
FROM   address WHERE postal_code IS NOT NULL LIMIT 5;

-- Exemplo 3: mascarar parte do email do cliente
SELECT REPLACE(email, SUBSTRING_INDEX(email, '@', 1), '****') AS email_mascarado
FROM   customer LIMIT 5;
```

---

### `INSTR()` / `LOCATE()` / `POSITION()`

**Descrição:** retorna a posição (a partir de 1) da primeira ocorrência de uma substring dentro de uma string. Retorna 0 se não encontrar. `LOCATE()` aceita uma posição de início para a busca; `INSTR()` não.

**Restrições:** a busca é **case-insensitive** por padrão no MariaDB (depende do collation). Retorna 0 (não NULL) quando não encontra.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `substr` | STRING | ✅ | Substring a localizar |
| `str` | STRING | ✅ | String onde buscar |
| `pos` | INT | ❌ | (apenas LOCATE) Posição de início da busca |

```sql
-- Exemplo 1: filmes que contêm 'LOVE' no título e a posição onde aparece
SELECT title, INSTR(title, 'LOVE') AS posicao_love
FROM   film WHERE INSTR(title, 'LOVE') > 0;

-- Exemplo 2: localizar '@' no email para extrair o domínio
SELECT email, LOCATE('@', email) AS pos_arroba,
       SUBSTRING(email, LOCATE('@', email) + 1) AS dominio
FROM   customer LIMIT 5;

-- Exemplo 3: filmes com 'THE' após a posição 5 do título
SELECT title FROM film
WHERE  LOCATE('THE', title, 5) > 0 LIMIT 5;
```

---

### `LEFT()` / `RIGHT()`

**Descrição:** extrai os primeiros (`LEFT`) ou últimos (`RIGHT`) N caracteres de uma string. Atalhos para `SUBSTRING(str, 1, n)` e `SUBSTRING(str, -n)`.

**Restrições:** se `n` for maior que o tamanho da string, retorna a string inteira. Retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String de origem |
| `len` | INT | ✅ | Número de caracteres a extrair |

```sql
-- Exemplo 1: primeiras 3 letras do título (abreviação)
SELECT title, LEFT(title, 3) AS abrev FROM film LIMIT 10;

-- Exemplo 2: últimos 2 caracteres do postal_code (sufixo)
SELECT postal_code, RIGHT(postal_code, 2) AS sufixo
FROM   address WHERE postal_code IS NOT NULL LIMIT 5;

-- Exemplo 3: iniciais da categoria (LEFT de cada palavra)
SELECT name, LEFT(name, 4) AS abrev_categoria FROM category;
```

---

### `LPAD()` / `RPAD()`

**Descrição:** preenche uma string à esquerda (`LPAD`) ou à direita (`RPAD`) com um caractere de preenchimento até atingir o comprimento total especificado. Se a string original for maior que `len`, ela é **truncada**.

**Restrições:** se `padstr` for vazio, retorna string vazia. Se `len` for menor que o tamanho atual, trunca a string.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String a preencher |
| `len` | INT | ✅ | Comprimento total desejado |
| `padstr` | STRING | ✅ | String de preenchimento (repetida se necessário) |

```sql
-- Exemplo 1: formatar film_id com zeros à esquerda (ex: 001, 042)
SELECT LPAD(film_id, 3, '0') AS id_formatado, title FROM film LIMIT 10;

-- Exemplo 2: alinhar preços à direita com espaços
SELECT title, LPAD(CAST(rental_rate AS CHAR), 5, ' ') AS preco_alinhado
FROM   film ORDER BY rental_rate DESC LIMIT 5;

-- Exemplo 3: criar código de barras fictício com RPAD
SELECT film_id, RPAD(CAST(film_id AS CHAR), 13, '0') AS codigo_barras
FROM   film LIMIT 5;
```

---

### `REPEAT()`

**Descrição:** repete uma string N vezes. Útil para gerar separadores, barras de progresso ou dados de teste.

**Restrições:** se `count` for menor ou igual a 0, retorna string vazia. Retorna `NULL` se qualquer argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String a repetir |
| `count` | INT | ✅ | Número de repetições |

```sql
-- Exemplo 1: barra de rating visual (★ por nível)
SELECT title, rating,
       CASE rating
           WHEN 'G'     THEN REPEAT('★', 1)
           WHEN 'PG'    THEN REPEAT('★', 2)
           WHEN 'PG-13' THEN REPEAT('★', 3)
           WHEN 'R'     THEN REPEAT('★', 4)
           WHEN 'NC-17' THEN REPEAT('★', 5)
       END AS nivel_visual
FROM film LIMIT 10;

-- Exemplo 2: separador de seção em relatório
SELECT REPEAT('-', 50) AS separador;

-- Exemplo 3: gerar espaçamento proporcional ao rental_rate
SELECT title, rental_rate, REPEAT('|', ROUND(rental_rate)) AS barra
FROM   film ORDER BY rental_rate LIMIT 10;
```

---

### `REVERSE()`

**Descrição:** inverte a ordem dos caracteres de uma string.

**Restrições:** retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String a inverter |

```sql
-- Exemplo 1: verificar palíndromos nos títulos (ilustrativo)
SELECT title FROM film WHERE title = REVERSE(title);

-- Exemplo 2: inverter os sobrenomes dos atores
SELECT last_name, REVERSE(last_name) AS invertido FROM actor LIMIT 10;

-- Exemplo 3: chave de hash simples invertendo o email
SELECT email, REVERSE(email) AS chave FROM customer LIMIT 5;
```

---

### `REGEXP_REPLACE()`

**Descrição:** substitui ocorrências de um padrão de expressão regular por uma string de substituição. Disponível no MariaDB 10.0.5+ e MySQL 8.0+.

**Restrições:** a expressão regular segue a sintaxe POSIX. No MariaDB, não é sensível a maiúsculas por padrão. Pode ser lento em tabelas grandes.

> ⚠️ **Disponibilidade:** `REGEXP_REPLACE()` não existe no MySQL 5.7 ou anterior. No MySQL 8.0+ e MariaDB 10.0.5+ está disponível.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String de entrada |
| `pattern` | STRING | ✅ | Expressão regular a localizar |
| `replace` | STRING | ✅ | String substituta |

```sql
-- Exemplo 1: remover todos os dígitos do título
SELECT title, REGEXP_REPLACE(title, '[0-9]', '') AS sem_numeros
FROM   film WHERE title REGEXP '[0-9]' LIMIT 5;

-- Exemplo 2: remover caracteres não numéricos do postal_code
SELECT postal_code, REGEXP_REPLACE(postal_code, '[^0-9]', '') AS apenas_numeros
FROM   address WHERE postal_code IS NOT NULL LIMIT 5;

-- Exemplo 3: substituir múltiplos espaços por um único espaço
SELECT REGEXP_REPLACE('ACADEMY  DINOSAUR', ' +', ' ') AS normalizado;
```

---

### `FORMAT()`

**Descrição:** formata um número com separadores de milhar e casas decimais, retornando uma **string** formatada. O separador de milhar e decimal segue o locale especificado.

**Restrições:** retorna uma **string**, não um número — não use para cálculos. Se `decimals` for 0, o decimal é omitido.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número a formatar |
| `D` | INT | ✅ | Número de casas decimais |
| `locale` | STRING | ❌ | Locale para formatação (ex: `'pt_BR'`). Padrão: `en_US` |

```sql
-- Exemplo 1: valor do pagamento formatado como moeda
SELECT payment_id, FORMAT(amount, 2) AS valor_formatado
FROM   payment ORDER BY amount DESC LIMIT 5;

-- Exemplo 2: receita total formatada
SELECT FORMAT(SUM(amount), 2, 'en_US') AS receita_total FROM payment;

-- Exemplo 3: rental_rate formatado sem decimais para filmes baratos
SELECT title, rental_rate, FORMAT(rental_rate, 0) AS preco_arredondado
FROM   film WHERE rental_rate < 1.0;
```

---

### `SUBSTRING_INDEX()`

**Descrição:** retorna a parte de uma string antes (count positivo) ou depois (count negativo) do N-ésimo delimitador encontrado.

**Restrições:** a busca pelo delimitador é **case-sensitive**. Muito útil para parsear strings com separadores fixos (emails, caminhos, CSVs simples).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String de entrada |
| `delim` | STRING | ✅ | Delimitador a procurar |
| `count` | INT | ✅ | N-ésima ocorrência (positivo = da esquerda; negativo = da direita) |

```sql
-- Exemplo 1: extrair nome de usuário do email (antes do @)
SELECT email, SUBSTRING_INDEX(email, '@', 1) AS usuario
FROM   customer LIMIT 5;

-- Exemplo 2: extrair domínio do email (depois do @)
SELECT email, SUBSTRING_INDEX(email, '@', -1) AS dominio
FROM   customer LIMIT 5;

-- Exemplo 3: primeiro nome do ator (antes do espaço) de um nome concatenado
SELECT CONCAT(first_name, ' ', last_name) AS nome_completo,
       SUBSTRING_INDEX(CONCAT(first_name, ' ', last_name), ' ', 1) AS primeiro_nome
FROM   actor LIMIT 5;
```

---

### `GROUP_CONCAT()`

**Descrição:** função de agregação que concatena valores de múltiplas linhas em uma única string, com separador configurável e possibilidade de ordenação. Extremamente útil para "pivotar" resultados sem criar tabelas intermediárias.

**Restrições:** o resultado é limitado pelo parâmetro `group_concat_max_len` (padrão: 1024 bytes). Ignorar `ORDER BY` dentro do `GROUP_CONCAT` pode gerar ordenação não determinística.

> ⚠️ **MariaDB vs MySQL:** em ambos o padrão do `group_concat_max_len` é 1024. Em MySQL 8.0+, o padrão foi aumentado. Para textos longos, ajuste: `SET SESSION group_concat_max_len = 65536;`

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `[DISTINCT] expr` | ANY | ✅ | Expressão/coluna a concatenar |
| `ORDER BY` | CLAUSE | ❌ | Ordena os valores antes de concatenar |
| `SEPARATOR str` | STRING | ❌ | Separador entre valores (padrão: `,`) |

```sql
-- Exemplo 1: lista de categorias de cada filme
SELECT f.film_id, f.title,
       GROUP_CONCAT(c.name ORDER BY c.name SEPARATOR ', ') AS categorias
FROM   film f
JOIN   film_category fc ON fc.film_id    = f.film_id
JOIN   category      c  ON c.category_id = fc.category_id
GROUP BY f.film_id, f.title
LIMIT 10;

-- Exemplo 2: atores de cada filme (separados por ' | ')
SELECT f.title,
       GROUP_CONCAT(CONCAT(a.first_name, ' ', a.last_name)
                    ORDER BY a.last_name
                    SEPARATOR ' | ') AS elenco
FROM   film f
JOIN   film_actor fa ON fa.film_id  = f.film_id
JOIN   actor      a  ON a.actor_id  = fa.actor_id
GROUP BY f.film_id, f.title
LIMIT 5;

-- Exemplo 3: IDs de filmes por categoria
SELECT c.name AS categoria,
       GROUP_CONCAT(fc.film_id ORDER BY fc.film_id) AS ids_filmes
FROM   category c
JOIN   film_category fc ON fc.category_id = c.category_id
GROUP BY c.category_id, c.name;
```

---
## 2. Funções Numéricas

---

### `ROUND()`

**Descrição:** arredonda um número para o número especificado de casas decimais. Com `decimals` negativo, arredonda à esquerda do ponto decimal (dezenas, centenas etc.).

**Restrições:** usa arredondamento bancário (*half-up* em MariaDB). Retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número a arredondar |
| `D` | INT | ❌ | Casas decimais (padrão: 0; negativo arredonda à esquerda) |

```sql
-- Exemplo 1: arredondar valor médio de pagamento para 2 casas
SELECT ROUND(AVG(amount), 2) AS media_pagamento FROM payment;

-- Exemplo 2: rental_rate arredondado para inteiro
SELECT title, rental_rate, ROUND(rental_rate) AS preco_inteiro
FROM   film ORDER BY rental_rate LIMIT 10;

-- Exemplo 3: receita total arredondada para a dezena mais próxima
SELECT ROUND(SUM(amount), -1) AS receita_aproximada FROM payment;
```

---

### `FLOOR()`

**Descrição:** retorna o maior inteiro **menor ou igual** ao número fornecido (arredondamento para baixo). Para negativos, arredonda em direção ao infinito negativo.

**Restrições:** retorna um valor `BIGINT`. Retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número a arredondar para baixo |

```sql
-- Exemplo 1: faixa de preço de cada filme (arredondado para baixo)
SELECT title, rental_rate, FLOOR(rental_rate) AS faixa_preco
FROM   film ORDER BY rental_rate;

-- Exemplo 2: quantidade de semanas completas de uma locação
SELECT rental_id,
       FLOOR(DATEDIFF(return_date, rental_date) / 7) AS semanas_completas
FROM   rental WHERE return_date IS NOT NULL LIMIT 10;

-- Exemplo 3: dividir clientes em grupos de 100
SELECT customer_id, first_name,
       FLOOR((customer_id - 1) / 100) + 1 AS grupo
FROM   customer LIMIT 10;
```

---

### `CEIL()` / `CEILING()`

**Descrição:** retorna o menor inteiro **maior ou igual** ao número fornecido (arredondamento para cima). `CEIL` e `CEILING` são sinônimos.

**Restrições:** retorna um valor `BIGINT`. Retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número a arredondar para cima |

```sql
-- Exemplo 1: número de páginas para exibir todos os filmes (10 por página)
SELECT CEIL(COUNT(*) / 10) AS total_paginas FROM film;

-- Exemplo 2: cobrança mínima (arredonda para cima o valor por dia)
SELECT rental_id,
       DATEDIFF(return_date, rental_date)              AS dias,
       CEIL(DATEDIFF(return_date, rental_date) * 0.99) AS cobranca_minima
FROM   rental WHERE return_date IS NOT NULL LIMIT 10;

-- Exemplo 3: número de atores por filme (arredondando grupos)
SELECT film_id, COUNT(*) AS atores,
       CEIL(COUNT(*) / 5.0) AS grupos_de_5
FROM   film_actor GROUP BY film_id LIMIT 10;
```

---

### `ABS()`

**Descrição:** retorna o valor absoluto (módulo) de um número — remove o sinal negativo se houver.

**Restrições:** retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número do qual calcular o valor absoluto |

```sql
-- Exemplo 1: diferença absoluta entre rental_rate e o preço médio
SELECT title, rental_rate,
       ABS(rental_rate - (SELECT AVG(rental_rate) FROM film)) AS distancia_media
FROM   film ORDER BY distancia_media LIMIT 10;

-- Exemplo 2: garantir que replacement_cost seja positivo (ilustrativo)
SELECT film_id, ABS(replacement_cost) AS custo_positivo FROM film LIMIT 5;

-- Exemplo 3: diferença em dias entre datas (sem importar a ordem)
SELECT rental_id, ABS(DATEDIFF(return_date, rental_date)) AS dias_absoluto
FROM   rental WHERE return_date IS NOT NULL LIMIT 10;
```

---

### `MOD()` / `%`

**Descrição:** retorna o resto da divisão inteira de X por Y. O operador `%` é equivalente.

**Restrições:** retorna `NULL` se Y for 0. Retorna `NULL` se qualquer argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Dividendo |
| `Y` | NUMERIC | ✅ | Divisor |

```sql
-- Exemplo 1: identificar filmes com film_id par
SELECT film_id, title FROM film WHERE MOD(film_id, 2) = 0 LIMIT 10;

-- Exemplo 2: distribuir clientes em 3 grupos (A, B, C)
SELECT customer_id, first_name,
       CASE MOD(customer_id, 3)
           WHEN 0 THEN 'Grupo A'
           WHEN 1 THEN 'Grupo B'
           WHEN 2 THEN 'Grupo C'
       END AS grupo
FROM customer LIMIT 10;

-- Exemplo 3: calcular quantidade de locações por semana (ímpares vs pares)
SELECT MOD(WEEK(rental_date), 2) AS semana_par_impar, COUNT(*) AS total
FROM   rental GROUP BY semana_par_impar;
```

---

### `POWER()` / `POW()`

**Descrição:** retorna X elevado à potência Y (X^Y). `POW` e `POWER` são sinônimos.

**Restrições:** retorna `NULL` se qualquer argumento for `NULL`. Resulta em erro de domínio se X=0 e Y<0.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Base |
| `Y` | NUMERIC | ✅ | Expoente |

```sql
-- Exemplo 1: calcular juros compostos sobre replacement_cost (10% ao ano por 3 anos)
SELECT title, replacement_cost,
       ROUND(replacement_cost * POWER(1.10, 3), 2) AS valor_futuro
FROM   film LIMIT 5;

-- Exemplo 2: converter para escala quadrática o rental_duration
SELECT title, rental_duration, POWER(rental_duration, 2) AS duracao_quadratica
FROM   film LIMIT 5;

-- Exemplo 3: demonstração de potências
SELECT POWER(2, 8) AS dois_elevado_oito,
       POWER(10, 3) AS mil;
```

---

### `SQRT()`

**Descrição:** retorna a raiz quadrada de um número. O resultado é do tipo `DOUBLE`.

**Restrições:** retorna `NULL` para números negativos. Retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número (deve ser >= 0) |

```sql
-- Exemplo 1: raiz quadrada do replacement_cost
SELECT title, replacement_cost, ROUND(SQRT(replacement_cost), 4) AS raiz
FROM   film ORDER BY replacement_cost DESC LIMIT 5;

-- Exemplo 2: desvio padrão manual usando variância
SELECT ROUND(SQRT(AVG(POW(amount - (SELECT AVG(amount) FROM payment), 2))), 2) AS desvio_padrao
FROM   payment;

-- Exemplo 3: distância geométrica fictícia entre film_ids
SELECT ABS(f1.film_id - f2.film_id) AS diferenca,
       ROUND(SQRT(POW(f1.film_id, 2) + POW(f2.film_id, 2)), 2) AS distancia
FROM   film f1, film f2
WHERE  f1.film_id = 1 AND f2.film_id = 5;
```

---

### `TRUNCATE()`

**Descrição:** trunca um número para o número especificado de casas decimais **sem arredondar** — simplesmente descarta as casas excedentes.

**Restrições:** diferente de `ROUND()`, **nunca arredonda**. Com `D` negativo, zera dígitos à esquerda do ponto.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número a truncar |
| `D` | INT | ✅ | Casas decimais a manter (negativo = zera à esquerda) |

```sql
-- Exemplo 1: truncar rental_rate (sem arredondar)
SELECT title, rental_rate,
       TRUNCATE(rental_rate, 1) AS truncado_1_casa
FROM   film LIMIT 5;

-- Exemplo 2: truncar para centena mais próxima (D=-2)
SELECT replacement_cost, TRUNCATE(replacement_cost, -2) AS arredondado_centena
FROM   film LIMIT 5;

-- Exemplo 3: comparar ROUND vs TRUNCATE
SELECT amount,
       ROUND(amount, 1)    AS arredondado,
       TRUNCATE(amount, 1) AS truncado
FROM   payment LIMIT 10;
```

---

### `RAND()`

**Descrição:** retorna um número de ponto flutuante aleatório no intervalo `[0, 1)`. Com uma semente (`seed`), a sequência é determinística (reproduzível).

**Restrições:** sem semente, o resultado é não-determinístico — cada chamada retorna um valor diferente. Use `seed` para resultados reproduzíveis.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `seed` | INT | ❌ | Semente para geração determinística |

```sql
-- Exemplo 1: selecionar 5 filmes aleatórios
SELECT film_id, title FROM film ORDER BY RAND() LIMIT 5;

-- Exemplo 2: gerar um preço aleatório entre 1.00 e 5.00
SELECT title, ROUND(1 + RAND() * 4, 2) AS preco_aleatorio
FROM   film LIMIT 5;

-- Exemplo 3: atribuir um número aleatório a cada cliente (com semente fixa)
SELECT customer_id, first_name, RAND(customer_id) AS numero_sorteio
FROM   customer ORDER BY numero_sorteio LIMIT 10;
```

---

### `SIGN()`

**Descrição:** retorna o sinal de um número: -1 (negativo), 0 (zero) ou 1 (positivo).

**Restrições:** retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `X` | NUMERIC | ✅ | Número a verificar |

```sql
-- Exemplo 1: classificar filmes pelo sinal da diferença do rental_rate em relação à média
SELECT title, rental_rate,
       SIGN(rental_rate - (SELECT AVG(rental_rate) FROM film)) AS sinal
FROM   film LIMIT 10;

-- Exemplo 2: verificar se pagamento está dentro do esperado
SELECT payment_id, amount,
       SIGN(amount - 4.00) AS acima_media
FROM   payment LIMIT 10;

-- Exemplo 3: contagem por sinal
SELECT SIGN(amount - 3) AS categoria, COUNT(*) AS total
FROM   payment GROUP BY categoria;
```

---

### `GREATEST()` / `LEAST()`

**Descrição:** retorna o maior (`GREATEST`) ou menor (`LEAST`) valor entre uma lista de argumentos. Aceita qualquer tipo comparável.

**Restrições:** se qualquer argumento for `NULL`, retorna `NULL`. Todos os argumentos são comparados usando o tipo do primeiro.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `val1` | ANY | ✅ | Primeiro valor |
| `val2` | ANY | ✅ | Segundo valor |
| `...valN` | ANY | ❌ | Valores adicionais |

```sql
-- Exemplo 1: maior valor entre rental_rate e replacement_cost / 30
SELECT title,
       rental_rate,
       ROUND(replacement_cost / 30, 2)                        AS custo_diario,
       GREATEST(rental_rate, ROUND(replacement_cost / 30, 2)) AS maior_valor
FROM   film LIMIT 5;

-- Exemplo 2: menor duração entre rental_duration e 5
SELECT title, rental_duration,
       LEAST(rental_duration, 5) AS duracao_ate_5
FROM   film LIMIT 10;

-- Exemplo 3: data mais recente entre duas colunas fictícias
SELECT rental_id, rental_date, return_date,
       GREATEST(rental_date, COALESCE(return_date, rental_date)) AS data_mais_recente
FROM   rental LIMIT 5;
```

---
## 3. Funções de Data e Hora

---

### `NOW()` / `SYSDATE()` / `CURRENT_TIMESTAMP()`

**Descrição:** retorna a data e hora atual do servidor. `NOW()` e `CURRENT_TIMESTAMP()` retornam o timestamp do **início da instrução** (consistente dentro de uma transação); `SYSDATE()` retorna o momento **exato da chamada**.

**Restrições:** `NOW()` é determinística dentro de uma query/transação — todas as chamadas retornam o mesmo valor. `SYSDATE()` não é, o que pode causar problemas com replicação.

> ⚠️ **MariaDB vs MySQL:** no MariaDB, `NOW()` aceita parâmetro de precisão de frações de segundo: `NOW(6)` retorna microsegundos. O mesmo vale para MySQL 5.6.4+.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `fsp` | INT (0–6) | ❌ | Precisão de frações de segundo |

```sql
-- Exemplo 1: data atual de cada locação em aberto
SELECT rental_id, rental_date, NOW() AS consultado_em,
       DATEDIFF(NOW(), rental_date) AS dias_em_aberto
FROM   rental WHERE return_date IS NULL;

-- Exemplo 2: usar NOW() como timestamp de auditoria
INSERT INTO rental (rental_date, inventory_id, customer_id, staff_id)
VALUES (NOW(), 1, 1, 1);

-- Exemplo 3: comparar NOW() com SYSDATE() (mostrar diferença de microsegundos)
SELECT NOW(6) AS agora_preciso, SYSDATE(6) AS sysdate_preciso;
```

---

### `CURDATE()` / `CURRENT_DATE()`

**Descrição:** retorna apenas a data atual (sem hora) no formato `YYYY-MM-DD`.

**Restrições:** sem parâmetros. Retorna o mesmo valor durante toda a execução da instrução.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| *(nenhum)* | — | — | — |

```sql
-- Exemplo 1: locações feitas hoje
SELECT * FROM rental WHERE DATE(rental_date) = CURDATE();

-- Exemplo 2: filmes cujo rental_duration ainda está vigente a partir de hoje
SELECT title, rental_duration,
       DATE_ADD(CURDATE(), INTERVAL rental_duration DAY) AS prazo_maximo
FROM   film LIMIT 10;

-- Exemplo 3: quantos dias desde a primeira locação registrada
SELECT DATEDIFF(CURDATE(), MIN(rental_date)) AS dias_desde_inicio
FROM   rental;
```

---

### `CURTIME()` / `CURRENT_TIME()`

**Descrição:** retorna apenas a hora atual no formato `HH:MM:SS`.

**Restrições:** sem parâmetros. Aceita precisão de frações de segundo (MariaDB/MySQL 5.6.4+).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `fsp` | INT (0–6) | ❌ | Precisão de frações de segundo |

```sql
-- Exemplo 1: hora atual do servidor
SELECT CURTIME() AS hora_servidor;

-- Exemplo 2: verificar se está dentro do horário comercial (8h-18h)
SELECT CURTIME() BETWEEN '08:00:00' AND '18:00:00' AS horario_comercial;

-- Exemplo 3: hora atual com precisão de microsegundos
SELECT CURTIME(6) AS hora_microsegundos;
```

---

### `DATE()` / `TIME()`

**Descrição:** extrai apenas a parte de data (`DATE`) ou de hora (`TIME`) de um valor `DATETIME` ou `TIMESTAMP`.

**Restrições:** retorna `NULL` se o argumento for `NULL` ou inválido.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | DATETIME | ✅ | Valor de data/hora de origem |

```sql
-- Exemplo 1: extrair apenas a data das locações
SELECT rental_id, DATE(rental_date) AS data_locacao FROM rental LIMIT 5;

-- Exemplo 2: extrair apenas a hora das locações
SELECT rental_id, TIME(rental_date) AS hora_locacao FROM rental LIMIT 5;

-- Exemplo 3: agrupar locações por data
SELECT DATE(rental_date) AS data, COUNT(*) AS total_locacoes
FROM   rental GROUP BY DATE(rental_date) ORDER BY data LIMIT 10;
```

---

### `YEAR()` / `MONTH()` / `DAY()` / `HOUR()` / `MINUTE()` / `SECOND()`

**Descrição:** extrai uma parte específica de um valor de data/hora. Equivalente a `EXTRACT(parte FROM expr)`.

**Restrições:** retornam `NULL` se o argumento for `NULL`. `MONTH()` retorna 1–12; `DAY()` retorna 1–31.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `date` | DATE/DATETIME | ✅ | Valor de data/hora |

```sql
-- Exemplo 1: locações por ano e mês
SELECT YEAR(rental_date) AS ano, MONTH(rental_date) AS mes, COUNT(*) AS total
FROM   rental GROUP BY ano, mes ORDER BY ano, mes;

-- Exemplo 2: filmes por ano de lançamento
SELECT YEAR(release_year) AS ano, COUNT(*) AS total
FROM   film GROUP BY ano ORDER BY ano;

-- Exemplo 3: horário de pico das locações (por hora do dia)
SELECT HOUR(rental_date) AS hora, COUNT(*) AS total
FROM   rental GROUP BY hora ORDER BY total DESC LIMIT 5;
```

---

### `DATE_FORMAT()`

**Descrição:** formata um valor de data/hora como string usando especificadores de formato. Altamente flexível para geração de relatórios.

**Restrições:** retorna `NULL` se `date` for `NULL`. A string de formato é sensível a maiúsculas nos especificadores.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `date` | DATE/DATETIME | ✅ | Valor a formatar |
| `format` | STRING | ✅ | String com especificadores de formato |

**Principais especificadores:**

| Código | Significado | Exemplo |
|---|---|---|
| `%d` | Dia (01–31) | 05 |
| `%m` | Mês numérico (01–12) | 03 |
| `%Y` | Ano com 4 dígitos | 2026 |
| `%y` | Ano com 2 dígitos | 26 |
| `%H` | Hora 24h (00–23) | 14 |
| `%i` | Minuto (00–59) | 30 |
| `%s` | Segundo (00–59) | 45 |
| `%W` | Nome do dia (Monday) | Monday |
| `%M` | Nome do mês (January) | January |
| `%p` | AM/PM | PM |

```sql
-- Exemplo 1: data da locação formatada no padrão brasileiro
SELECT rental_id,
       DATE_FORMAT(rental_date, '%d/%m/%Y %H:%i') AS data_br
FROM   rental LIMIT 10;

-- Exemplo 2: relatório mensal com mês por extenso
SELECT DATE_FORMAT(payment_date, '%M/%Y') AS periodo,
       SUM(amount) AS total
FROM   payment GROUP BY periodo ORDER BY MIN(payment_date);

-- Exemplo 3: dia da semana e hora de cada locação
SELECT rental_id,
       DATE_FORMAT(rental_date, '%W, %d de %M de %Y às %H:%i') AS descricao
FROM   rental LIMIT 5;
```

---

### `DATEDIFF()`

**Descrição:** retorna o número de dias entre duas datas (expr1 − expr2). Considera apenas a parte de data, ignorando a hora.

**Restrições:** retorna um inteiro (pode ser negativo se expr2 > expr1). Retorna `NULL` se qualquer argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr1` | DATE/DATETIME | ✅ | Data mais recente (minuendo) |
| `expr2` | DATE/DATETIME | ✅ | Data mais antiga (subtraendo) |

```sql
-- Exemplo 1: dias de duração de cada locação
SELECT rental_id,
       rental_date, return_date,
       DATEDIFF(return_date, rental_date) AS dias_locacao
FROM   rental WHERE return_date IS NOT NULL LIMIT 10;

-- Exemplo 2: locações com atraso (mais de 5 dias)
SELECT rental_id, customer_id,
       DATEDIFF(return_date, rental_date) AS dias
FROM   rental
WHERE  return_date IS NOT NULL
  AND  DATEDIFF(return_date, rental_date) > 5
ORDER BY dias DESC LIMIT 10;

-- Exemplo 3: tempo médio de locação por categoria
SELECT c.name AS categoria,
       ROUND(AVG(DATEDIFF(r.return_date, r.rental_date)), 1) AS media_dias
FROM   rental r
JOIN   inventory  i  ON i.inventory_id  = r.inventory_id
JOIN   film       f  ON f.film_id       = i.film_id
JOIN   film_category fc ON fc.film_id   = f.film_id
JOIN   category   c  ON c.category_id  = fc.category_id
WHERE  r.return_date IS NOT NULL
GROUP BY c.name ORDER BY media_dias DESC;
```

---

### `TIMESTAMPDIFF()`

**Descrição:** retorna a diferença entre dois timestamps na unidade especificada (YEAR, MONTH, DAY, HOUR, MINUTE, SECOND). Mais preciso que `DATEDIFF()` para unidades diferentes de dias.

**Restrições:** retorna um inteiro (parte inteira da diferença). Retorna `NULL` se qualquer argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `unit` | ENUM | ✅ | Unidade: YEAR, MONTH, DAY, HOUR, MINUTE, SECOND, MICROSECOND |
| `datetime_expr1` | DATETIME | ✅ | Data/hora inicial (mais antiga) |
| `datetime_expr2` | DATETIME | ✅ | Data/hora final (mais recente) |

```sql
-- Exemplo 1: horas de cada locação
SELECT rental_id,
       TIMESTAMPDIFF(HOUR, rental_date, return_date) AS horas_locacao
FROM   rental WHERE return_date IS NOT NULL LIMIT 10;

-- Exemplo 2: idade dos clientes em anos (usando last_update como proxy)
SELECT customer_id, first_name, last_name,
       TIMESTAMPDIFF(YEAR, create_date, NOW()) AS anos_de_cliente
FROM   customer LIMIT 10;

-- Exemplo 3: diferença em minutos entre locação e devolução
SELECT rental_id,
       TIMESTAMPDIFF(MINUTE, rental_date, return_date) AS minutos_total
FROM   rental WHERE return_date IS NOT NULL
ORDER BY minutos_total DESC LIMIT 5;
```

---

### `DATE_ADD()` / `DATE_SUB()` / `ADDDATE()` / `SUBDATE()`

**Descrição:** adiciona (`DATE_ADD`) ou subtrai (`DATE_SUB`) um intervalo de tempo a uma data. `ADDDATE` e `SUBDATE` são sinônimos. A cláusula `INTERVAL` define a quantidade e unidade.

**Restrições:** retorna `NULL` se o argumento for `NULL` ou se o resultado for uma data inválida.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `date` | DATE/DATETIME | ✅ | Data de origem |
| `INTERVAL expr unit` | INTERVAL | ✅ | Quantidade e unidade (DAY, MONTH, YEAR, HOUR...) |

```sql
-- Exemplo 1: prazo máximo de devolução (data + rental_duration)
SELECT r.rental_id,
       r.rental_date,
       DATE_ADD(r.rental_date, INTERVAL f.rental_duration DAY) AS prazo_devolucao,
       r.return_date
FROM   rental r
JOIN   inventory i ON i.inventory_id = r.inventory_id
JOIN   film      f ON f.film_id      = i.film_id
WHERE  r.return_date IS NOT NULL LIMIT 10;

-- Exemplo 2: locações dos últimos 30 dias
SELECT COUNT(*) AS locacoes_ultimo_mes
FROM   rental
WHERE  rental_date >= DATE_SUB(NOW(), INTERVAL 30 DAY);

-- Exemplo 3: projeção de vencimento daqui a 6 meses
SELECT title, rental_duration,
       DATE_ADD(CURDATE(), INTERVAL 6 MONTH) AS vencimento_promo
FROM   film LIMIT 5;
```

---

### `EXTRACT()`

**Descrição:** extrai uma parte específica de uma data/hora. Equivalente funcional às funções `YEAR()`, `MONTH()`, `DAY()` etc., mas com sintaxe padrão SQL.

**Restrições:** retorna `NULL` se o argumento for `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `unit` | ENUM | ✅ | YEAR, MONTH, DAY, HOUR, MINUTE, SECOND, QUARTER, WEEK |
| `FROM date` | DATE/DATETIME | ✅ | Valor de origem |

```sql
-- Exemplo 1: trimestre das locações
SELECT EXTRACT(QUARTER FROM rental_date) AS trimestre, COUNT(*) AS total
FROM   rental GROUP BY trimestre ORDER BY trimestre;

-- Exemplo 2: semana do ano
SELECT EXTRACT(WEEK FROM rental_date) AS semana, COUNT(*) AS total
FROM   rental GROUP BY semana ORDER BY semana LIMIT 10;

-- Exemplo 3: usar QUARTER para agrupamento de faturamento
SELECT EXTRACT(YEAR FROM payment_date)    AS ano,
       EXTRACT(QUARTER FROM payment_date) AS trimestre,
       SUM(amount)                        AS faturamento
FROM   payment
GROUP BY ano, trimestre ORDER BY ano, trimestre;
```

---

### `STR_TO_DATE()`

**Descrição:** converte uma string em um valor `DATE`, `TIME` ou `DATETIME` usando um formato especificado. Inverso de `DATE_FORMAT()`.

**Restrições:** retorna `NULL` se a string não corresponder ao formato. Útil para importar dados com formatos de data não-padrão.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String contendo a data |
| `format` | STRING | ✅ | Formato da string (mesmos especificadores de `DATE_FORMAT`) |

```sql
-- Exemplo 1: converter string no formato brasileiro para DATE
SELECT STR_TO_DATE('15/03/2026', '%d/%m/%Y') AS data_convertida;

-- Exemplo 2: comparar data em formato textual com coluna
SELECT rental_id, rental_date
FROM   rental
WHERE  rental_date > STR_TO_DATE('2005-06-01', '%Y-%m-%d')
LIMIT 5;

-- Exemplo 3: converter timestamp completo
SELECT STR_TO_DATE('15/03/2026 14:30:00', '%d/%m/%Y %H:%i:%s') AS datetime_convertido;
```

---

### `WEEK()` / `WEEKDAY()` / `DAYOFWEEK()` / `DAYOFYEAR()`

**Descrição:** funções para extrair informações de semana e dia. `WEEK()` retorna o número da semana (0–53); `WEEKDAY()` retorna 0=Segunda a 6=Domingo; `DAYOFWEEK()` retorna 1=Domingo a 7=Sábado (padrão ODBC); `DAYOFYEAR()` retorna 1–366.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `date` | DATE/DATETIME | ✅ | Data de origem |
| `mode` | INT (0–7) | ❌ | (apenas WEEK) Define o primeiro dia da semana e contagem |

```sql
-- Exemplo 1: locações por dia da semana (0=Segunda)
SELECT WEEKDAY(rental_date) AS dia_semana, COUNT(*) AS total
FROM   rental GROUP BY dia_semana ORDER BY dia_semana;

-- Exemplo 2: número da semana de cada locação
SELECT rental_id, rental_date, WEEK(rental_date) AS semana_do_ano
FROM   rental LIMIT 10;

-- Exemplo 3: locações no fim de semana (DAYOFWEEK 1=Dom, 7=Sab)
SELECT COUNT(*) AS locacoes_fds
FROM   rental
WHERE  DAYOFWEEK(rental_date) IN (1, 7);
```

---

### `LAST_DAY()`

**Descrição:** retorna o último dia do mês de uma data fornecida. Útil para calcular fins de mês dinamicamente.

**Restrições:** retorna `NULL` se o argumento for `NULL` ou inválido.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `date` | DATE/DATETIME | ✅ | Qualquer data do mês desejado |

```sql
-- Exemplo 1: último dia do mês de cada pagamento
SELECT payment_id, payment_date, LAST_DAY(payment_date) AS fim_do_mes
FROM   payment LIMIT 5;

-- Exemplo 2: dias restantes no mês de cada locação
SELECT rental_id, rental_date,
       DATEDIFF(LAST_DAY(rental_date), rental_date) AS dias_restantes_mes
FROM   rental LIMIT 10;

-- Exemplo 3: primeiro dia do próximo mês
SELECT DATE_ADD(LAST_DAY(CURDATE()), INTERVAL 1 DAY) AS primeiro_proximo_mes;
```

---

## 4. Funções de Agregação

---

### `COUNT()`

**Descrição:** conta o número de linhas ou valores não-NULL. `COUNT(*)` conta todas as linhas (incluindo NULLs); `COUNT(coluna)` conta apenas valores não-NULL; `COUNT(DISTINCT col)` conta valores únicos não-NULL.

**Restrições:** sempre retorna um inteiro >= 0. Nunca retorna NULL.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `* \| expr` | ANY | ✅ | `*` para todas as linhas, ou expressão/coluna específica |
| `DISTINCT` | KEYWORD | ❌ | Conta apenas valores únicos |

```sql
-- Exemplo 1: total de filmes, atores e clientes
SELECT
    (SELECT COUNT(*) FROM film)     AS total_filmes,
    (SELECT COUNT(*) FROM actor)    AS total_atores,
    (SELECT COUNT(*) FROM customer) AS total_clientes;

-- Exemplo 2: filmes por categoria com contagem
SELECT c.name AS categoria, COUNT(fc.film_id) AS total_filmes
FROM   category c
LEFT JOIN film_category fc ON fc.category_id = c.category_id
GROUP BY c.name ORDER BY total_filmes DESC;

-- Exemplo 3: clientes distintos que alugaram filmes de ação
SELECT COUNT(DISTINCT r.customer_id) AS clientes_unicos_acao
FROM   rental r
JOIN   inventory     i  ON i.inventory_id  = r.inventory_id
JOIN   film          f  ON f.film_id       = i.film_id
JOIN   film_category fc ON fc.film_id      = f.film_id
JOIN   category      c  ON c.category_id  = fc.category_id
WHERE  c.name = 'Action';
```

---

### `SUM()`

**Descrição:** retorna a soma de todos os valores não-NULL de uma expressão numérica. Com `DISTINCT`, soma apenas valores únicos.

**Restrições:** retorna `NULL` se não houver linhas ou se todos os valores forem `NULL`. Ignora NULLs automaticamente.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `DISTINCT` | KEYWORD | ❌ | Soma apenas valores distintos |
| `expr` | NUMERIC | ✅ | Expressão numérica a somar |

```sql
-- Exemplo 1: receita total por loja
SELECT s.store_id, SUM(p.amount) AS receita_total
FROM   payment p
JOIN   staff   st ON st.staff_id = p.staff_id
JOIN   store   s  ON s.store_id  = st.store_id
GROUP BY s.store_id;

-- Exemplo 2: total arrecadado por categoria de filme
SELECT c.name AS categoria, SUM(p.amount) AS receita
FROM   payment p
JOIN   rental        r  ON r.rental_id    = p.rental_id
JOIN   inventory     i  ON i.inventory_id = r.inventory_id
JOIN   film_category fc ON fc.film_id     = i.film_id
JOIN   category      c  ON c.category_id  = fc.category_id
GROUP BY c.name ORDER BY receita DESC;

-- Exemplo 3: custo total de reposição do inventário
SELECT SUM(f.replacement_cost * COUNT_inv.qtd) AS custo_total_reposicao
FROM   film f
JOIN   (SELECT film_id, COUNT(*) AS qtd FROM inventory GROUP BY film_id) AS COUNT_inv
       ON COUNT_inv.film_id = f.film_id;
```

---

### `AVG()`

**Descrição:** retorna a média aritmética de valores não-NULL. Com `DISTINCT`, calcula a média apenas dos valores únicos.

**Restrições:** retorna `NULL` se não houver linhas ou todos forem `NULL`. Ignora NULLs no cálculo.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `DISTINCT` | KEYWORD | ❌ | Média apenas de valores distintos |
| `expr` | NUMERIC | ✅ | Expressão numérica |

```sql
-- Exemplo 1: média de pagamento por cliente
SELECT customer_id, ROUND(AVG(amount), 2) AS media_pagamento
FROM   payment GROUP BY customer_id ORDER BY media_pagamento DESC LIMIT 10;

-- Exemplo 2: duração média de filmes por rating
SELECT rating, ROUND(AVG(length), 1) AS duracao_media_min
FROM   film GROUP BY rating ORDER BY duracao_media_min;

-- Exemplo 3: ticket médio por mês
SELECT DATE_FORMAT(payment_date, '%Y-%m') AS mes,
       ROUND(AVG(amount), 2)              AS ticket_medio
FROM   payment GROUP BY mes ORDER BY mes;
```

---

### `MIN()` / `MAX()`

**Descrição:** retornam o menor (`MIN`) ou maior (`MAX`) valor de um conjunto. Funcionam com números, strings e datas.

**Restrições:** ignoram `NULL`. Retornam `NULL` se todos os valores forem `NULL` ou não houver linhas.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | ANY | ✅ | Expressão a avaliar |

```sql
-- Exemplo 1: filme mais barato e mais caro de cada categoria
SELECT c.name AS categoria,
       MIN(f.rental_rate) AS menor_preco,
       MAX(f.rental_rate) AS maior_preco
FROM   film f
JOIN   film_category fc ON fc.film_id    = f.film_id
JOIN   category      c  ON c.category_id = fc.category_id
GROUP BY c.name;

-- Exemplo 2: primeira e última locação de cada cliente
SELECT customer_id,
       MIN(rental_date) AS primeira_locacao,
       MAX(rental_date) AS ultima_locacao
FROM   rental GROUP BY customer_id LIMIT 10;

-- Exemplo 3: pagamento mínimo e máximo por loja
SELECT st.store_id, MIN(p.amount) AS menor, MAX(p.amount) AS maior
FROM   payment p
JOIN   staff st ON st.staff_id = p.staff_id
GROUP BY st.store_id;
```

---

### `STD()` / `STDDEV()` / `STDDEV_POP()` / `STDDEV_SAMP()`

**Descrição:** calculam o desvio padrão de um conjunto de valores. `STD()`/`STDDEV()`/`STDDEV_POP()` calculam o desvio padrão **populacional**; `STDDEV_SAMP()` calcula o **amostral** (dividindo por N-1).

**Restrições:** retornam `NULL` com 0 linhas. `STDDEV_SAMP()` retorna `NULL` com apenas 1 linha.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | NUMERIC | ✅ | Expressão numérica |

```sql
-- Exemplo 1: variação do preço dos filmes
SELECT ROUND(AVG(rental_rate), 2)  AS media,
       ROUND(STD(rental_rate), 4)  AS desvio_padrao_pop,
       ROUND(STDDEV_SAMP(rental_rate), 4) AS desvio_padrao_amostral
FROM   film;

-- Exemplo 2: consistência de pagamentos por funcionário
SELECT staff_id, ROUND(STD(amount), 4) AS variacao_pagamentos
FROM   payment GROUP BY staff_id;

-- Exemplo 3: desvio padrão da duração de filmes por rating
SELECT rating, ROUND(STD(length), 2) AS desvio_duracao
FROM   film GROUP BY rating ORDER BY desvio_duracao DESC;
```

---

### `VARIANCE()` / `VAR_POP()` / `VAR_SAMP()`

**Descrição:** calculam a variância dos valores. `VAR_POP()` é populacional (N); `VAR_SAMP()` é amostral (N-1). `VARIANCE()` é alias de `VAR_POP()`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | NUMERIC | ✅ | Expressão numérica |

```sql
-- Exemplo 1: variância dos pagamentos
SELECT ROUND(VARIANCE(amount), 4)  AS variancia_pop,
       ROUND(VAR_SAMP(amount), 4)  AS variancia_amostral
FROM   payment;

-- Exemplo 2: variância do custo de reposição por categoria
SELECT c.name, ROUND(VARIANCE(f.replacement_cost), 2) AS variancia_custo
FROM   film f
JOIN   film_category fc ON fc.film_id    = f.film_id
JOIN   category      c  ON c.category_id = fc.category_id
GROUP BY c.name ORDER BY variancia_custo DESC;

-- Exemplo 3: comparação de variância entre filmes curtos e longos
SELECT CASE WHEN length < 90 THEN 'Curto' ELSE 'Longo' END AS tipo,
       ROUND(VAR_SAMP(rental_rate), 4) AS variancia_preco
FROM   film GROUP BY tipo;
```

---
## 5. Funções Condicionais

---

### `IF()`

**Descrição:** avalia uma condição e retorna o segundo argumento se verdadeira, ou o terceiro se falsa. Equivalente ao operador ternário de linguagens de programação.

**Restrições:** não é o mesmo que o `IF` de procedures (que é uma instrução de controle de fluxo). Aqui é uma função escalar usada em `SELECT`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | BOOLEAN | ✅ | Condição a avaliar |
| `if_true` | ANY | ✅ | Valor retornado se a condição for verdadeira |
| `if_false` | ANY | ✅ | Valor retornado se a condição for falsa |

```sql
-- Exemplo 1: classificar filme como 'Caro' ou 'Barato'
SELECT title, rental_rate,
       IF(rental_rate >= 3.99, 'Caro', 'Barato') AS faixa_preco
FROM   film ORDER BY rental_rate;

-- Exemplo 2: verificar se locação foi devolvida
SELECT rental_id,
       IF(return_date IS NULL, 'Em aberto', 'Devolvida') AS situacao
FROM   rental LIMIT 10;

-- Exemplo 3: contagem condicional (filmes por faixa de preço)
SELECT SUM(IF(rental_rate < 2, 1, 0))  AS baratos,
       SUM(IF(rental_rate >= 2 AND rental_rate < 4, 1, 0)) AS medios,
       SUM(IF(rental_rate >= 4, 1, 0)) AS caros
FROM   film;
```

---

### `IFNULL()`

**Descrição:** retorna o primeiro argumento se ele não for `NULL`; caso contrário, retorna o segundo. Caso de uso mais comum: substituir NULLs por um valor padrão.

**Restrições:** apenas dois argumentos. Para mais de uma alternativa, use `COALESCE()`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr1` | ANY | ✅ | Valor a verificar |
| `expr2` | ANY | ✅ | Valor substituto se expr1 for NULL |

```sql
-- Exemplo 1: exibir 'Não devolvido' quando return_date for NULL
SELECT rental_id, IFNULL(return_date, 'Não devolvido') AS devolucao
FROM   rental LIMIT 10;

-- Exemplo 2: usar 0 quando amount for NULL
SELECT payment_id, IFNULL(amount, 0) AS valor FROM payment LIMIT 5;

-- Exemplo 3: substituir address2 NULL por string vazia
SELECT address, IFNULL(address2, '') AS complemento FROM address LIMIT 10;
```

---

### `COALESCE()`

**Descrição:** retorna o **primeiro valor não-NULL** da lista de argumentos. Equivalente a múltiplos `IFNULL` aninhados. É a função padrão SQL para tratamento de NULLs com múltiplas alternativas.

**Restrições:** aceita qualquer número de argumentos. Retorna `NULL` apenas se **todos** os argumentos forem `NULL`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `val1` | ANY | ✅ | Primeiro valor a verificar |
| `val2` | ANY | ✅ | Segundo valor (alternativa) |
| `...valN` | ANY | ❌ | Alternativas adicionais |

```sql
-- Exemplo 1: endereço completo usando COALESCE para campos opcionais
SELECT COALESCE(address2, address, 'Sem endereço') AS melhor_endereco
FROM   address LIMIT 10;

-- Exemplo 2: nome de exibição: apelido ou nome completo
-- (Sakila não tem apelido, simulamos com last_name como fallback)
SELECT COALESCE(NULL, first_name, 'Cliente') AS nome_exibicao
FROM   customer LIMIT 5;

-- Exemplo 3: valor do pagamento ou 0 se NULL
SELECT rental_id, COALESCE(
    (SELECT SUM(amount) FROM payment p WHERE p.rental_id = r.rental_id),
    0
) AS total_pago
FROM rental r LIMIT 10;
```

---

### `NULLIF()`

**Descrição:** retorna `NULL` se os dois argumentos forem **iguais**; caso contrário, retorna o primeiro argumento. Útil para evitar divisão por zero.

**Restrições:** compara os dois argumentos com `=`. Retorna o tipo do primeiro argumento.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr1` | ANY | ✅ | Valor a retornar se diferente de expr2 |
| `expr2` | ANY | ✅ | Valor de comparação |

```sql
-- Exemplo 1: evitar divisão por zero ao calcular média
SELECT c.name AS categoria,
       COUNT(fc.film_id) AS total,
       SUM(f.rental_rate) / NULLIF(COUNT(fc.film_id), 0) AS media_segura
FROM   category c
LEFT JOIN film_category fc ON fc.category_id = c.category_id
LEFT JOIN film           f  ON f.film_id      = fc.film_id
GROUP BY c.name;

-- Exemplo 2: tratar string vazia como NULL
SELECT address, NULLIF(address2, '') AS complemento_nulo
FROM   address LIMIT 5;

-- Exemplo 3: retornar NULL quando rental_rate é igual ao mínimo
SELECT title, rental_rate,
       NULLIF(rental_rate, 0.99) AS preco_exceto_minimo
FROM   film ORDER BY rental_rate LIMIT 10;
```

---

### `CASE`

**Descrição:** expressão condicional que avalia condições e retorna um resultado. Existe em duas formas: **simples** (compara uma expressão com valores fixos) e **pesquisada** (avalia condições booleanas independentes).

**Restrições:** deve ter ao menos um `WHEN`. O `ELSE` é opcional — sem ele, retorna `NULL` quando nenhuma condição é satisfeita. Pode ser usada em `SELECT`, `WHERE`, `ORDER BY` e `GROUP BY`.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | ANY | ❌ | (forma simples) Valor a comparar com cada WHEN |
| `WHEN val THEN result` | — | ✅ | Ao menos um par WHEN/THEN |
| `ELSE result` | — | ❌ | Resultado padrão se nenhum WHEN satisfeito |

```sql
-- Exemplo 1: classificação de rating em português (forma simples)
SELECT title, rating,
       CASE rating
           WHEN 'G'     THEN 'Livre'
           WHEN 'PG'    THEN 'Indicado para Pais'
           WHEN 'PG-13' THEN 'Maiores de 13'
           WHEN 'R'     THEN 'Restrito'
           WHEN 'NC-17' THEN 'Adulto'
           ELSE 'Não classificado'
       END AS classificacao_br
FROM film LIMIT 10;

-- Exemplo 2: faixa de duração do filme (forma pesquisada)
SELECT title, length,
       CASE
           WHEN length < 60  THEN 'Curta-metragem'
           WHEN length < 90  THEN 'Filme curto'
           WHEN length < 120 THEN 'Duração padrão'
           ELSE 'Longa-metragem'
       END AS categoria_duracao
FROM film ORDER BY length LIMIT 10;

-- Exemplo 3: pivô de locações por dia da semana
SELECT
    SUM(CASE WHEN WEEKDAY(rental_date) = 0 THEN 1 ELSE 0 END) AS segunda,
    SUM(CASE WHEN WEEKDAY(rental_date) = 1 THEN 1 ELSE 0 END) AS terca,
    SUM(CASE WHEN WEEKDAY(rental_date) = 2 THEN 1 ELSE 0 END) AS quarta,
    SUM(CASE WHEN WEEKDAY(rental_date) = 3 THEN 1 ELSE 0 END) AS quinta,
    SUM(CASE WHEN WEEKDAY(rental_date) = 4 THEN 1 ELSE 0 END) AS sexta,
    SUM(CASE WHEN WEEKDAY(rental_date) = 5 THEN 1 ELSE 0 END) AS sabado,
    SUM(CASE WHEN WEEKDAY(rental_date) = 6 THEN 1 ELSE 0 END) AS domingo
FROM rental;
```

---

## 6. Funções de Conversão de Tipo

---

### `CAST()`

**Descrição:** converte um valor de um tipo para outro explicitamente. É o padrão SQL para conversão de tipos.

**Restrições:** nem toda conversão é possível — conversão de string para tipo numérico retorna 0 se a string não for um número válido, sem erro. Tipos suportados: `CHAR`, `DATE`, `DATETIME`, `DECIMAL`, `FLOAT`, `DOUBLE`, `SIGNED`, `UNSIGNED`, `TIME`, `JSON` (MariaDB 10.2.7+).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | ANY | ✅ | Valor a converter |
| `AS type` | TYPE | ✅ | Tipo de destino |

```sql
-- Exemplo 1: converter rental_duration para CHAR para concatenação
SELECT CONCAT('Duração: ', CAST(rental_duration AS CHAR), ' dias') AS info
FROM   film LIMIT 5;

-- Exemplo 2: converter string de data para DATE
SELECT CAST('2026-03-15' AS DATE) AS data_convertida;

-- Exemplo 3: converter amount para SIGNED para operações
SELECT payment_id, CAST(amount AS SIGNED) AS valor_inteiro FROM payment LIMIT 5;
```

---

### `CONVERT()`

**Descrição:** similar ao `CAST()`, mas com sintaxe alternativa. Também suporta conversão de charset quando usado com `USING`.

**Restrições:** mesmas restrições do `CAST()`. A forma com `USING` é específica para conversão de charset, não de tipo de dado.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | ANY | ✅ | Valor a converter |
| `type` | TYPE | ✅ (forma 1) | Tipo de destino (mesmos do CAST) |
| `USING charset` | STRING | ✅ (forma 2) | Charset de destino |

```sql
-- Exemplo 1: CONVERT de tipo (equivalente ao CAST)
SELECT CONVERT(rental_rate, CHAR) AS preco_texto FROM film LIMIT 5;

-- Exemplo 2: CONVERT de charset
SELECT CONVERT(title USING utf8mb4) AS titulo_utf8 FROM film LIMIT 5;

-- Exemplo 3: converter DECIMAL para UNSIGNED
SELECT CONVERT(ROUND(amount), UNSIGNED) AS valor_inteiro_positivo
FROM   payment LIMIT 5;
```

---

### `FORMAT()` (como conversão)

*(Ver Seção 1 — já documentada como função de string.)*

---

### `HEX()` / `UNHEX()`

**Descrição:** `HEX()` converte um número ou string para sua representação hexadecimal. `UNHEX()` faz o inverso — converte uma string hexadecimal para binário.

**Restrições:** `UNHEX()` retorna `NULL` se a string contiver caracteres não-hexadecimais.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str \| N` | STRING/INT | ✅ | Valor a converter |

```sql
-- Exemplo 1: representação hexadecimal dos film_ids
SELECT film_id, HEX(film_id) AS hex_id FROM film LIMIT 5;

-- Exemplo 2: converter título para hex (útil para debug de encoding)
SELECT title, HEX(title) AS hex_titulo FROM film LIMIT 3;

-- Exemplo 3: round-trip HEX/UNHEX
SELECT UNHEX(HEX(first_name)) AS nome_original FROM actor LIMIT 3;
```

---

### `BIN()` / `OCT()`

**Descrição:** `BIN()` converte um inteiro para sua representação binária (base 2) como string. `OCT()` converte para octal (base 8).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `N` | INT | ✅ | Número inteiro a converter |

```sql
-- Exemplo 1: representação binária dos film_ids
SELECT film_id, BIN(film_id) AS binario FROM film LIMIT 5;

-- Exemplo 2: representação octal
SELECT film_id, OCT(film_id) AS octal FROM film LIMIT 5;

-- Exemplo 3: comparar base 10, 2 e 16
SELECT film_id, BIN(film_id) AS bin, HEX(film_id) AS hex FROM film LIMIT 5;
```

---

## 7. Funções JSON

> ⚠️ **Disponibilidade:** as funções JSON requerem MariaDB 10.2.3+ ou MySQL 5.7.8+. O Sakila não tem colunas JSON nativas, então os exemplos criam estruturas JSON inline com `JSON_OBJECT()` e `JSON_ARRAY()`.

---

### `JSON_OBJECT()`

**Descrição:** cria um objeto JSON a partir de pares chave-valor alternados. As chaves devem ser strings; os valores podem ser de qualquer tipo.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `key1, val1, key2, val2, ...` | ANY | ✅ | Pares alternados de chave (string) e valor |

```sql
-- Exemplo 1: criar JSON de dados do filme
SELECT JSON_OBJECT(
    'id',      film_id,
    'titulo',  title,
    'preco',   rental_rate,
    'rating',  rating
) AS filme_json
FROM film LIMIT 3;

-- Exemplo 2: dados do cliente em JSON
SELECT JSON_OBJECT(
    'id',     customer_id,
    'nome',   CONCAT(first_name, ' ', last_name),
    'email',  email,
    'ativo',  IF(active = 1, true, false)
) AS cliente_json
FROM customer LIMIT 3;

-- Exemplo 3: sumarizar dados de locação em JSON
SELECT JSON_OBJECT(
    'rental_id',  rental_id,
    'data',       DATE_FORMAT(rental_date, '%Y-%m-%d'),
    'devolvido',  return_date IS NOT NULL
) AS locacao_json
FROM rental LIMIT 3;
```

---

### `JSON_ARRAY()`

**Descrição:** cria um array JSON a partir de uma lista de valores. Os valores podem ser de qualquer tipo.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `val1, val2, ...` | ANY | ✅ | Valores do array |

```sql
-- Exemplo 1: array de ratings disponíveis
SELECT JSON_ARRAY('G', 'PG', 'PG-13', 'R', 'NC-17') AS ratings;

-- Exemplo 2: array de film_ids de um ator
SELECT actor_id,
       JSON_ARRAYAGG(film_id) AS filmes_json
FROM   film_actor
GROUP BY actor_id
LIMIT 5;

-- Exemplo 3: combinar JSON_OBJECT e JSON_ARRAY
SELECT JSON_OBJECT(
    'ator',   CONCAT(first_name, ' ', last_name),
    'filmes', JSON_ARRAY(
        (SELECT film_id FROM film_actor WHERE actor_id = a.actor_id LIMIT 1),
        (SELECT film_id FROM film_actor WHERE actor_id = a.actor_id LIMIT 1 OFFSET 1)
    )
) AS ator_filmes
FROM actor a LIMIT 3;
```

---

### `JSON_EXTRACT()` / Operador `->`

**Descrição:** extrai um valor de um documento JSON usando um caminho JSONPath. O operador `->` é um atalho para `JSON_EXTRACT()` e retorna o valor com aspas se for string; `->>`  retorna sem aspas (equivale a `JSON_UNQUOTE(JSON_EXTRACT())`).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `json_doc` | JSON/STRING | ✅ | Documento JSON |
| `path, ...` | STRING | ✅ | Caminho JSONPath (ex: `$.chave`, `$.array[0]`) |

```sql
-- Exemplo 1: extrair valor de JSON inline
SELECT JSON_EXTRACT('{"titulo": "Pulp Fiction", "ano": 1994}', '$.titulo') AS titulo;

-- Exemplo 2: extrair múltiplos campos
SET @doc = '{"id": 1, "nome": "Ana Lima", "notas": [8.5, 9.0, 7.5]}';
SELECT JSON_EXTRACT(@doc, '$.nome')     AS nome,
       JSON_EXTRACT(@doc, '$.notas[0]') AS primeira_nota;

-- Exemplo 3: usar -> (requer coluna JSON real; aqui com variável)
SET @filme = JSON_OBJECT('title', 'ACADEMY DINOSAUR', 'rating', 'PG');
SELECT @filme -> '$.title'  AS titulo,
       @filme -> '$.rating' AS rating;
```

---

### `JSON_SET()` / `JSON_INSERT()` / `JSON_REPLACE()`

**Descrição:** modificam documentos JSON. `JSON_SET()` insere ou atualiza; `JSON_INSERT()` apenas insere (não atualiza se o caminho existir); `JSON_REPLACE()` apenas atualiza (não insere se não existir).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `json_doc` | JSON | ✅ | Documento JSON a modificar |
| `path, val [, path, val...]` | STRING, ANY | ✅ | Pares de caminho e novo valor |

```sql
-- Exemplo 1: JSON_SET — inserir ou atualizar campo
SET @doc = '{"titulo": "ACADEMY DINOSAUR"}';
SELECT JSON_SET(@doc, '$.rating', 'PG', '$.ano', 2006) AS doc_atualizado;

-- Exemplo 2: JSON_INSERT — apenas inserir (sem sobrescrever)
SET @doc = '{"titulo": "ADAPTATION HOLES", "rating": "NC-17"}';
SELECT JSON_INSERT(@doc, '$.rating', 'PG', '$.novo_campo', 'valor') AS resultado;
-- '$.rating' não é alterado pois já existe; '$.novo_campo' é inserido

-- Exemplo 3: JSON_REPLACE — apenas atualizar (sem inserir)
SET @doc = '{"titulo": "AFFAIR PREJUDICE", "rating": "G"}';
SELECT JSON_REPLACE(@doc, '$.rating', 'PG', '$.inexistente', 'X') AS resultado;
-- '$.rating' é atualizado; '$.inexistente' é ignorado (não existe)
```

---

### `JSON_ARRAYAGG()` / `JSON_OBJECTAGG()`

**Descrição:** funções de agregação que constroem arrays ou objetos JSON a partir de múltiplas linhas. `JSON_ARRAYAGG()` cria um array com os valores; `JSON_OBJECTAGG()` cria um objeto com chave-valor.

> ⚠️ **Disponibilidade:** `JSON_ARRAYAGG()` e `JSON_OBJECTAGG()` estão disponíveis no MariaDB 10.5.0+ e MySQL 5.7.22+.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | ANY | ✅ | (ARRAYAGG) Valor a incluir no array |
| `key, val` | STRING, ANY | ✅ | (OBJECTAGG) Par chave-valor |

```sql
-- Exemplo 1: array de títulos por categoria
SELECT c.name AS categoria,
       JSON_ARRAYAGG(f.title) AS filmes_json
FROM   category c
JOIN   film_category fc ON fc.category_id = c.category_id
JOIN   film          f  ON f.film_id      = fc.film_id
GROUP BY c.name LIMIT 3;

-- Exemplo 2: objeto com film_id como chave e título como valor
SELECT JSON_OBJECTAGG(film_id, title) AS catalogo
FROM   film WHERE film_id <= 5;

-- Exemplo 3: array de objetos de atores por filme
SELECT f.title,
       JSON_ARRAYAGG(
           JSON_OBJECT('id', a.actor_id, 'nome', CONCAT(a.first_name, ' ', a.last_name))
       ) AS elenco_json
FROM   film f
JOIN   film_actor fa ON fa.film_id  = f.film_id
JOIN   actor      a  ON a.actor_id  = fa.actor_id
GROUP BY f.film_id, f.title
LIMIT 3;
```

---

## 8. Funções de Criptografia e Hash

> ⚠️ **Importante:** funções como `MD5()` e `SHA1()` **não são criptografia** — são funções de hash unidirecionais. Não são adequadas para senhas sem salting e iteração. Para senhas em produção, use bcrypt ou Argon2 na camada de aplicação.

---

### `MD5()`

**Descrição:** retorna o hash MD5 de uma string como string hexadecimal de 32 caracteres. Algoritmo unidirecional — não é possível reverter.

**Restrições:** MD5 é considerado **inseguro para senhas** (colisões conhecidas). Use apenas para checksums ou identificadores não-críticos.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String a fazer o hash |

```sql
-- Exemplo 1: hash MD5 do email do cliente (mascaramento)
SELECT customer_id, MD5(email) AS email_hash FROM customer LIMIT 5;

-- Exemplo 2: gerar identificador único para o título
SELECT film_id, title, MD5(title) AS titulo_hash FROM film LIMIT 5;

-- Exemplo 3: checksum de dados concatenados
SELECT rental_id, MD5(CONCAT(rental_id, customer_id, rental_date)) AS checksum
FROM   rental LIMIT 5;
```

---

### `SHA1()` / `SHA2()`

**Descrição:** `SHA1()` retorna hash SHA-1 (40 hex chars). `SHA2()` retorna hash SHA-2 com comprimento configurável: 224, 256, 384 ou 512 bits. SHA-256 e SHA-512 são os mais usados e considerados seguros para checksums.

**Restrições:** `SHA2()` retorna `NULL` se o comprimento especificado não for válido (0, 224, 256, 384 ou 512).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | (SHA1/SHA2) String a fazer o hash |
| `hash_length` | INT | ✅ | (apenas SHA2) Comprimento: 224, 256, 384, 512 ou 0 (=256) |

```sql
-- Exemplo 1: hash SHA-256 do email
SELECT customer_id, SHA2(email, 256) AS email_sha256 FROM customer LIMIT 3;

-- Exemplo 2: comparar tamanhos de SHA
SELECT
    LENGTH(SHA1(title))    AS sha1_len,
    LENGTH(SHA2(title,256)) AS sha256_len,
    LENGTH(SHA2(title,512)) AS sha512_len
FROM film LIMIT 1;

-- Exemplo 3: integridade de dados — verificar se título mudou
SELECT film_id, title, SHA2(title, 256) AS hash_titulo
FROM   film WHERE film_id IN (1, 2, 3);
```

---

### `AES_ENCRYPT()` / `AES_DECRYPT()`

**Descrição:** criptografia simétrica AES (*Advanced Encryption Standard*). `AES_ENCRYPT()` retorna dados binários cifrados; `AES_DECRYPT()` reverte com a mesma chave. Diferente dos hashes, esta operação é **reversível**.

**Restrições:** retorna dados binários — armazene em coluna `VARBINARY` ou `BLOB`. A segurança depende inteiramente da chave. Por padrão usa AES-128-ECB (modo inseguro para dados sensíveis em produção — prefira modo CBC com IV via `block_encryption_mode`).

> ⚠️ **MariaDB vs MySQL:** o modo de criptografia padrão é `aes-128-ecb` em ambos. Para segurança real em produção, configure `block_encryption_mode = 'aes-256-cbc'` e use um IV (Initialization Vector).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `str` | STRING | ✅ | String a cifrar/decifrar |
| `key_str` | STRING | ✅ | Chave de criptografia |
| `init_vector` | STRING | ❌ | Vetor de inicialização (para modos CBC/OFB/CFB) |

```sql
-- Exemplo 1: criptografar e descriptografar email (modo padrão ECB)
SELECT AES_ENCRYPT('ana@sakila.com', 'chave_secreta_aula') AS cifrado;
SELECT CAST(AES_DECRYPT(AES_ENCRYPT('ana@sakila.com', 'chave_secreta_aula'), 'chave_secreta_aula') AS CHAR) AS original;

-- Exemplo 2: verificar tamanho do dado cifrado
SELECT LENGTH(AES_ENCRYPT(email, 'chave_teste')) AS tamanho_cifrado
FROM   customer LIMIT 3;

-- Exemplo 3: cifrar coluna e verificar com hex
SELECT HEX(AES_ENCRYPT(first_name, 'chave_demo')) AS nome_cifrado_hex
FROM   actor LIMIT 3;
```

---

### `PASSWORD()` (depreciada)

**Descrição:** gerava hashes de senha no formato interno do MySQL. **Depreciada no MySQL 5.7.6** e **removida no MySQL 8.0**. No MariaDB ainda está presente mas não recomendada para uso geral.

> ⚠️ **Não use `PASSWORD()` em aplicações.** Esta função foi criada para uso interno do MySQL no sistema de autenticação. Para hash de senhas de usuários de aplicação, use `SHA2()` com salt na aplicação, ou preferencialmente bcrypt/Argon2.

---

### `ENCRYPT()` (depreciada)

**Descrição:** usa a função `crypt()` do sistema operacional Unix para criptografia. **Depreciada e removida no MySQL 8.0**. Não disponível no Windows. Não recomendada.

---

## 9. Funções de Informação do Sistema

---

### `USER()` / `CURRENT_USER()`

**Descrição:** `USER()` retorna o usuário e host conectados como `'usuario@host'` (da conexão atual, pode incluir curingas). `CURRENT_USER()` retorna o usuário e host **conforme autenticado** (pode diferir se houver proxy user).

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| *(nenhum)* | — | — | — |

```sql
-- Exemplo 1: quem está conectado
SELECT USER() AS usuario_conectado, CURRENT_USER() AS usuario_autenticado;

-- Exemplo 2: registrar usuário em log de auditoria
INSERT INTO auditoria (acao, usuario, momento)
VALUES ('SELECT em filmes', USER(), NOW());

-- Exemplo 3: verificar permissões do usuário atual
SELECT USER() AS eu,
       (SELECT COUNT(*) FROM information_schema.USER_PRIVILEGES
        WHERE GRANTEE = CONCAT("'", SUBSTRING_INDEX(USER(),'@',1),
              "'@'", SUBSTRING_INDEX(USER(),'@',-1), "'")) AS total_privilegios;
```

---

### `DATABASE()` / `SCHEMA()`

**Descrição:** retorna o nome do banco de dados atualmente selecionado. `SCHEMA()` é sinônimo. Retorna `NULL` se nenhum banco estiver selecionado.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| *(nenhum)* | — | — | — |

```sql
-- Exemplo 1: banco atual
SELECT DATABASE() AS banco_selecionado;

-- Exemplo 2: usar em condições dinâmicas
SELECT table_name, table_rows
FROM   information_schema.tables
WHERE  table_schema = DATABASE()
ORDER BY table_rows DESC;

-- Exemplo 3: listar todas as tabelas do banco atual com tamanho
SELECT table_name,
       ROUND(data_length / 1024, 2) AS tamanho_kb
FROM   information_schema.tables
WHERE  table_schema = DATABASE()
ORDER BY data_length DESC;
```

---

### `VERSION()`

**Descrição:** retorna a versão do servidor MariaDB/MySQL como string. Útil para queries e scripts que precisam de comportamento condicional por versão.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| *(nenhum)* | — | — | — |

```sql
-- Exemplo 1: versão do servidor
SELECT VERSION() AS versao_servidor;

-- Exemplo 2: extrair número maior de versão
SELECT CAST(SUBSTRING_INDEX(VERSION(), '.', 1) AS UNSIGNED) AS versao_major;

-- Exemplo 3: identificar MariaDB vs MySQL
SELECT VERSION() AS versao,
       IF(VERSION() LIKE '%MariaDB%', 'MariaDB', 'MySQL') AS sgbd;
```

---

### `LAST_INSERT_ID()`

**Descrição:** retorna o ID gerado pelo último `INSERT AUTO_INCREMENT` na sessão atual. Específico da sessão — não é afetado por INSERTs de outras conexões.

**Restrições:** retorna 0 se nenhum INSERT com AUTO_INCREMENT foi executado na sessão. Em INSERT múltiplo (várias linhas), retorna o ID da **primeira** linha inserida.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| *(nenhum)* | — | — | Retorna o último AUTO_INCREMENT gerado na sessão |

```sql
-- Exemplo 1: inserir ator e recuperar o ID gerado
INSERT INTO actor (first_name, last_name, last_update)
VALUES ('RONAN', 'ZENATTI', NOW());
SELECT LAST_INSERT_ID() AS novo_actor_id;

-- Exemplo 2: usar LAST_INSERT_ID() em INSERT encadeado (FK)
-- (após inserir um filme, inserir na film_category)
-- SELECT LAST_INSERT_ID() AS id_novo_filme;

-- Exemplo 3: confirmar o último ID gerado sem novo INSERT
SELECT LAST_INSERT_ID() AS ultimo_id_sessao;
```

---

### `ROW_COUNT()`

**Descrição:** retorna o número de linhas afetadas pelo último comando DML (`INSERT`, `UPDATE`, `DELETE`). Para `SELECT`, retorna -1.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| *(nenhum)* | — | — | — |

```sql
-- Exemplo 1: quantas linhas foram atualizadas
UPDATE film SET rental_rate = rental_rate * 1.1 WHERE rating = 'NC-17';
SELECT ROW_COUNT() AS filmes_atualizados;

-- Exemplo 2: verificar sucesso de DELETE
DELETE FROM rental WHERE return_date < DATE_SUB(NOW(), INTERVAL 5 YEAR);
SELECT ROW_COUNT() AS locacoes_removidas;

-- Exemplo 3: usar ROW_COUNT() em procedure para verificar sucesso
-- (ilustrativo — em procedure real, usaríamos a variável em lógica IF)
SELECT ROW_COUNT() AS ultima_operacao_afetou;
```

---

### `FOUND_ROWS()`

**Descrição:** retorna o número de linhas que o último `SELECT` **teria** retornado sem a cláusula `LIMIT` — mas apenas se o `SELECT` usou `SQL_CALC_FOUND_ROWS`. Útil para paginação sem duas queries.

**Restrições:** requer `SQL_CALC_FOUND_ROWS` na query original. Tem custo de performance — o banco calcula o total mesmo ao aplicar o LIMIT.

> ⚠️ **MariaDB vs MySQL:** `SQL_CALC_FOUND_ROWS` e `FOUND_ROWS()` são depreciados no MySQL 8.0 — a alternativa recomendada é usar `COUNT()` numa subquery ou window function. No MariaDB 10.x, ainda são suportados.

```sql
-- Exemplo 1: paginação com total sem segunda query
SELECT SQL_CALC_FOUND_ROWS film_id, title
FROM   film
ORDER BY title
LIMIT 10 OFFSET 0;
SELECT FOUND_ROWS() AS total_sem_limit;

-- Exemplo 2: mesma técnica para clientes
SELECT SQL_CALC_FOUND_ROWS customer_id, first_name, last_name
FROM   customer WHERE active = 1
LIMIT 5;
SELECT FOUND_ROWS() AS total_ativos;

-- Exemplo 3: verificar total de resultados de busca
SELECT SQL_CALC_FOUND_ROWS title FROM film WHERE title LIKE '%LOVE%';
SELECT FOUND_ROWS() AS filmes_com_love;
```

---

### `CONNECTION_ID()`

**Descrição:** retorna o ID numérico único da conexão atual ao servidor.

```sql
-- Exemplo 1: identificar a conexão atual
SELECT CONNECTION_ID() AS minha_conexao;

-- Exemplo 2: ver todas as conexões ativas
SHOW PROCESSLIST;

-- Exemplo 3: registrar ID de conexão em log
SELECT CONNECTION_ID() AS conn_id, USER() AS usuario, DATABASE() AS banco, NOW() AS momento;
```

---

### `CHARSET()` / `COLLATION()`

**Descrição:** retornam o charset (`CHARSET`) ou collation (`COLLATION`) de uma string ou expressão.

```sql
-- Exemplo 1: charset atual das colunas
SELECT CHARSET(title) AS charset_titulo, COLLATION(title) AS collation_titulo
FROM   film LIMIT 1;

-- Exemplo 2: verificar charset do banco
SELECT CHARSET(DATABASE()) AS charset_banco;

-- Exemplo 3: verificar collation de uma string literal
SELECT COLLATION('teste') AS collation_literal;
```

---
## 10. Window Functions

> Window Functions (funções de janela) executam cálculos sobre um conjunto de linhas relacionadas à linha atual — sem colapsar as linhas em grupos como `GROUP BY`. Isso permite calcular rankings, médias móveis, totais acumulados e comparações entre linhas adjacentes, tudo em uma única query.

> ⚠️ **Disponibilidade:** Window Functions estão disponíveis no **MariaDB 10.2+** e **MySQL 8.0+**. **Não funcionam no MySQL 5.7 ou anterior.** Verifique com `SELECT VERSION()` antes de usar.

### Anatomia de uma Window Function

```sql
FUNÇÃO() OVER (
    [PARTITION BY coluna]   -- divide os dados em partições (grupos)
    [ORDER BY coluna]       -- ordena as linhas dentro de cada partição
    [frame_clause]          -- define a "janela" (ROWS/RANGE BETWEEN...)
)
```

[prompt para nanobanana: "Educational diagram explaining SQL Window Functions. Shows a table with columns: film_id, category, rental_rate. The rows are divided into colored partitions by category (Action=blue, Comedy=green, Drama=orange). Over each partition, an arrow labeled 'OVER (PARTITION BY category ORDER BY rental_rate)' shows ranking numbers being calculated per partition. The result table on the right shows the original rows preserved (not collapsed) with an added rank column. Label 'GROUP BY colapsa linhas' with X mark on left, 'Window Function preserva linhas' with check mark on right. Clean flat educational design, white background, Portuguese labels."]
![Como funcionam as Window Functions](../imgs/funcoes_mysql_img_01.png)

---

### `ROW_NUMBER()`

**Descrição:** atribui um número sequencial único a cada linha dentro de uma partição. Os números começam em 1 e nunca se repetem, mesmo com valores empate.

**Restrições:** requer `ORDER BY` dentro do `OVER` para resultado determinístico. Sem `ORDER BY`, a ordem é indefinida.

| Parâmetro do OVER | Obrigatório | Descrição |
|---|---|---|
| `PARTITION BY` | ❌ | Divide em grupos; a numeração reinicia em cada partição |
| `ORDER BY` | ✅ (recomendado) | Determina a ordem da numeração |

```sql
-- Exemplo 1: numerar filmes por ordem de preço dentro de cada categoria
SELECT f.title, c.name AS categoria, f.rental_rate,
       ROW_NUMBER() OVER (
           PARTITION BY c.category_id
           ORDER BY f.rental_rate DESC
       ) AS rank_preco
FROM   film f
JOIN   film_category fc ON fc.film_id    = f.film_id
JOIN   category      c  ON c.category_id = fc.category_id
LIMIT 20;

-- Exemplo 2: paginação segura — retornar a 2ª página (linhas 11-20)
SELECT * FROM (
    SELECT f.film_id, f.title, f.rental_rate,
           ROW_NUMBER() OVER (ORDER BY f.title) AS rn
    FROM   film f
) AS paginado
WHERE rn BETWEEN 11 AND 20;

-- Exemplo 3: numerar locações de cada cliente cronologicamente
SELECT customer_id, rental_date, return_date,
       ROW_NUMBER() OVER (
           PARTITION BY customer_id
           ORDER BY rental_date
       ) AS numero_locacao
FROM   rental
WHERE  customer_id IN (1, 2, 3)
ORDER BY customer_id, rental_date;
```

---

### `RANK()` / `DENSE_RANK()`

**Descrição:** atribuem rankings com tratamento de empates. `RANK()` pula números após empates (1, 1, 3, 4...); `DENSE_RANK()` não pula (1, 1, 2, 3...). Escolha entre eles dependendo se os "buracos" no ranking são aceitáveis.

| Parâmetro do OVER | Obrigatório | Descrição |
|---|---|---|
| `PARTITION BY` | ❌ | Reinicia o rank em cada partição |
| `ORDER BY` | ✅ | Determina a ordem do ranking |

```sql
-- Exemplo 1: ranking de atores por número de filmes (com empates)
SELECT actor_id, first_name, last_name, total_filmes,
       RANK()       OVER (ORDER BY total_filmes DESC) AS rank_com_pulo,
       DENSE_RANK() OVER (ORDER BY total_filmes DESC) AS rank_sem_pulo
FROM (
    SELECT a.actor_id, a.first_name, a.last_name, COUNT(fa.film_id) AS total_filmes
    FROM   actor a
    JOIN   film_actor fa ON fa.actor_id = a.actor_id
    GROUP BY a.actor_id, a.first_name, a.last_name
) AS contagem
ORDER BY total_filmes DESC LIMIT 15;

-- Exemplo 2: top 3 filmes mais caros por categoria
SELECT * FROM (
    SELECT f.title, c.name AS categoria, f.rental_rate,
           DENSE_RANK() OVER (
               PARTITION BY c.category_id
               ORDER BY f.rental_rate DESC
           ) AS posicao
    FROM   film f
    JOIN   film_category fc ON fc.film_id    = f.film_id
    JOIN   category      c  ON c.category_id = fc.category_id
) AS ranked
WHERE posicao <= 3
ORDER BY categoria, posicao;

-- Exemplo 3: percentil de cada cliente por total gasto
SELECT customer_id, total_gasto,
       RANK() OVER (ORDER BY total_gasto DESC) AS posicao,
       DENSE_RANK() OVER (ORDER BY total_gasto DESC) AS posicao_densa
FROM (
    SELECT customer_id, SUM(amount) AS total_gasto
    FROM   payment GROUP BY customer_id
) AS gastos
ORDER BY total_gasto DESC LIMIT 15;
```

---

### `NTILE()`

**Descrição:** divide as linhas em N grupos (buckets) aproximadamente iguais e retorna o número do grupo (1 a N) de cada linha. Útil para quartis, decis e percentis.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `N` | INT | ✅ | Número de grupos a criar |
| `PARTITION BY` (OVER) | ❌ | Divide em partições |
| `ORDER BY` (OVER) | ✅ | Define a ordem para distribuição |

```sql
-- Exemplo 1: dividir filmes em 4 quartis de preço
SELECT title, rental_rate,
       NTILE(4) OVER (ORDER BY rental_rate) AS quartil
FROM   film
ORDER BY rental_rate, quartil;

-- Exemplo 2: classificar clientes em decis por valor gasto
SELECT customer_id, total_gasto,
       NTILE(10) OVER (ORDER BY total_gasto) AS decil
FROM (
    SELECT customer_id, SUM(amount) AS total_gasto
    FROM   payment GROUP BY customer_id
) AS gastos
ORDER BY total_gasto DESC LIMIT 20;

-- Exemplo 3: média de preço por quartil
SELECT quartil, ROUND(AVG(rental_rate), 2) AS media_quartil,
       COUNT(*) AS filmes
FROM (
    SELECT rental_rate, NTILE(4) OVER (ORDER BY rental_rate) AS quartil FROM film
) AS q
GROUP BY quartil ORDER BY quartil;
```

---

### `PERCENT_RANK()` / `CUME_DIST()`

**Descrição:** calculam o ranking relativo de uma linha dentro de sua partição como um valor entre 0 e 1. `PERCENT_RANK()` calcula `(rank - 1) / (total_linhas - 1)`; `CUME_DIST()` calcula a distribuição cumulativa.

```sql
-- Exemplo 1: percent_rank dos filmes por preço
SELECT title, rental_rate,
       ROUND(PERCENT_RANK() OVER (ORDER BY rental_rate), 4) AS percent_rank,
       ROUND(CUME_DIST()    OVER (ORDER BY rental_rate), 4) AS cume_dist
FROM   film ORDER BY rental_rate LIMIT 15;

-- Exemplo 2: clientes no percentil 75+ de gastos
SELECT customer_id, total_gasto, pct
FROM (
    SELECT customer_id, total_gasto,
           ROUND(PERCENT_RANK() OVER (ORDER BY total_gasto), 4) AS pct
    FROM (
        SELECT customer_id, SUM(amount) AS total_gasto
        FROM   payment GROUP BY customer_id
    ) AS g
) AS ranked
WHERE pct >= 0.75
ORDER BY total_gasto DESC;

-- Exemplo 3: cume_dist dos pagamentos
SELECT amount, ROUND(CUME_DIST() OVER (ORDER BY amount), 4) AS cume
FROM   payment ORDER BY amount LIMIT 10;
```

---

### `LAG()` / `LEAD()`

**Descrição:** acessam o valor de uma linha **anterior** (`LAG`) ou **posterior** (`LEAD`) dentro da partição, sem precisar de auto-JOIN. Extremamente úteis para comparações entre linhas consecutivas.

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `expr` | ANY | ✅ | Coluna/expressão a acessar |
| `offset` | INT | ❌ | Quantas linhas atrás/frente (padrão: 1) |
| `default` | ANY | ❌ | Valor padrão quando não há linha no offset |

```sql
-- Exemplo 1: variação do pagamento em relação ao anterior do mesmo cliente
SELECT customer_id, payment_date, amount,
       LAG(amount, 1, 0) OVER (
           PARTITION BY customer_id
           ORDER BY payment_date
       ) AS pagamento_anterior,
       amount - LAG(amount, 1, 0) OVER (
           PARTITION BY customer_id
           ORDER BY payment_date
       ) AS variacao
FROM   payment
WHERE  customer_id IN (1, 2)
ORDER BY customer_id, payment_date;

-- Exemplo 2: próximo pagamento do cliente
SELECT customer_id, payment_date, amount,
       LEAD(amount, 1, NULL) OVER (
           PARTITION BY customer_id
           ORDER BY payment_date
       ) AS proximo_pagamento
FROM   payment
WHERE  customer_id = 1
ORDER BY payment_date;

-- Exemplo 3: datas consecutivas de locação do cliente
SELECT customer_id, rental_date,
       LAG(rental_date) OVER (PARTITION BY customer_id ORDER BY rental_date) AS locacao_anterior,
       DATEDIFF(rental_date,
           LAG(rental_date) OVER (PARTITION BY customer_id ORDER BY rental_date)
       ) AS dias_desde_ultima
FROM   rental
WHERE  customer_id IN (1, 2)
ORDER BY customer_id, rental_date;
```

---

### `FIRST_VALUE()` / `LAST_VALUE()` / `NTH_VALUE()`

**Descrição:** retornam o valor da **primeira** (`FIRST_VALUE`), **última** (`LAST_VALUE`) ou **N-ésima** linha (`NTH_VALUE`) dentro de uma janela.

**Restrições:** `LAST_VALUE()` por padrão usa a frame padrão (`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`), o que faz com que retorne o valor da linha atual, não o último da partição. Use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` para o último real.

```sql
-- Exemplo 1: menor preço de cada categoria ao lado de cada filme
SELECT f.title, c.name AS categoria, f.rental_rate,
       FIRST_VALUE(f.rental_rate) OVER (
           PARTITION BY c.category_id
           ORDER BY f.rental_rate
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS menor_preco_categoria
FROM   film f
JOIN   film_category fc ON fc.film_id    = f.film_id
JOIN   category      c  ON c.category_id = fc.category_id
LIMIT 15;

-- Exemplo 2: maior preço da categoria com LAST_VALUE corrigido
SELECT f.title, c.name AS categoria, f.rental_rate,
       LAST_VALUE(f.rental_rate) OVER (
           PARTITION BY c.category_id
           ORDER BY f.rental_rate
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS maior_preco_categoria
FROM   film f
JOIN   film_category fc ON fc.film_id    = f.film_id
JOIN   category      c  ON c.category_id = fc.category_id
LIMIT 15;

-- Exemplo 3: 2º menor preço de cada categoria com NTH_VALUE
SELECT f.title, c.name AS categoria, f.rental_rate,
       NTH_VALUE(f.rental_rate, 2) OVER (
           PARTITION BY c.category_id
           ORDER BY f.rental_rate
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS segundo_menor
FROM   film f
JOIN   film_category fc ON fc.film_id    = f.film_id
JOIN   category      c  ON c.category_id = fc.category_id
LIMIT 15;
```

---

### `SUM()` / `AVG()` / `COUNT()` como Window Functions

**Descrição:** as funções de agregação tradicionais podem ser usadas como window functions com `OVER()`. Isso permite calcular totais acumulados, médias móveis e porcentagens do total sem `GROUP BY`.

```sql
-- Exemplo 1: total acumulado de pagamentos por cliente
SELECT customer_id, payment_date, amount,
       SUM(amount) OVER (
           PARTITION BY customer_id
           ORDER BY payment_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS total_acumulado
FROM   payment
WHERE  customer_id = 1
ORDER BY payment_date;

-- Exemplo 2: percentual de cada pagamento sobre o total do cliente
SELECT customer_id, payment_date, amount,
       SUM(amount)   OVER (PARTITION BY customer_id) AS total_cliente,
       ROUND(amount / SUM(amount) OVER (PARTITION BY customer_id) * 100, 2) AS pct_do_total
FROM   payment
WHERE  customer_id IN (1, 2)
ORDER BY customer_id, payment_date;

-- Exemplo 3: média móvel de 3 pagamentos consecutivos
SELECT customer_id, payment_date, amount,
       ROUND(AVG(amount) OVER (
           PARTITION BY customer_id
           ORDER BY payment_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ), 2) AS media_movel_3
FROM   payment
WHERE  customer_id = 1
ORDER BY payment_date;
```

---

## Apêndice — Índice Rápido por Função

| Função | Categoria | Seção |
|---|---|---|
| `ABS()` | Numérica | 2 |
| `AES_DECRYPT()` | Criptografia | 8 |
| `AES_ENCRYPT()` | Criptografia | 8 |
| `AVG()` | Agregação | 4 |
| `BIN()` | Conversão | 6 |
| `CAST()` | Conversão | 6 |
| `CEIL()` / `CEILING()` | Numérica | 2 |
| `CHAR_LENGTH()` | String | 1 |
| `CHARSET()` | Sistema | 9 |
| `COALESCE()` | Condicional | 5 |
| `COLLATION()` | Sistema | 9 |
| `CONCAT()` | String | 1 |
| `CONCAT_WS()` | String | 1 |
| `CONNECTION_ID()` | Sistema | 9 |
| `CONVERT()` | Conversão | 6 |
| `COUNT()` | Agregação | 4 |
| `CUME_DIST()` | Window | 10 |
| `CURDATE()` | Data/Hora | 3 |
| `CURTIME()` | Data/Hora | 3 |
| `DATABASE()` | Sistema | 9 |
| `DATE()` | Data/Hora | 3 |
| `DATE_ADD()` | Data/Hora | 3 |
| `DATE_FORMAT()` | Data/Hora | 3 |
| `DATE_SUB()` | Data/Hora | 3 |
| `DATEDIFF()` | Data/Hora | 3 |
| `DAY()` | Data/Hora | 3 |
| `DAYOFWEEK()` | Data/Hora | 3 |
| `DAYOFYEAR()` | Data/Hora | 3 |
| `DENSE_RANK()` | Window | 10 |
| `ENCRYPT()` | Criptografia | 8 |
| `EXTRACT()` | Data/Hora | 3 |
| `FIRST_VALUE()` | Window | 10 |
| `FLOOR()` | Numérica | 2 |
| `FORMAT()` | String / Conversão | 1 |
| `FOUND_ROWS()` | Sistema | 9 |
| `GREATEST()` | Numérica | 2 |
| `GROUP_CONCAT()` | String/Agregação | 1 |
| `HEX()` | Conversão | 6 |
| `HOUR()` | Data/Hora | 3 |
| `IF()` | Condicional | 5 |
| `IFNULL()` | Condicional | 5 |
| `INSTR()` | String | 1 |
| `JSON_ARRAY()` | JSON | 7 |
| `JSON_ARRAYAGG()` | JSON | 7 |
| `JSON_EXTRACT()` | JSON | 7 |
| `JSON_INSERT()` | JSON | 7 |
| `JSON_OBJECT()` | JSON | 7 |
| `JSON_OBJECTAGG()` | JSON | 7 |
| `JSON_REPLACE()` | JSON | 7 |
| `JSON_SET()` | JSON | 7 |
| `LAG()` | Window | 10 |
| `LAST_DAY()` | Data/Hora | 3 |
| `LAST_INSERT_ID()` | Sistema | 9 |
| `LAST_VALUE()` | Window | 10 |
| `LEAD()` | Window | 10 |
| `LEAST()` | Numérica | 2 |
| `LEFT()` | String | 1 |
| `LENGTH()` | String | 1 |
| `LOCATE()` | String | 1 |
| `LOWER()` | String | 1 |
| `LPAD()` | String | 1 |
| `LTRIM()` | String | 1 |
| `MAX()` | Agregação | 4 |
| `MD5()` | Criptografia | 8 |
| `MIN()` | Agregação | 4 |
| `MINUTE()` | Data/Hora | 3 |
| `MOD()` | Numérica | 2 |
| `MONTH()` | Data/Hora | 3 |
| `NOW()` | Data/Hora | 3 |
| `NTH_VALUE()` | Window | 10 |
| `NTILE()` | Window | 10 |
| `NULLIF()` | Condicional | 5 |
| `OCT()` | Conversão | 6 |
| `PASSWORD()` | Criptografia | 8 |
| `PERCENT_RANK()` | Window | 10 |
| `POSITION()` | String | 1 |
| `POW()` / `POWER()` | Numérica | 2 |
| `RAND()` | Numérica | 2 |
| `RANK()` | Window | 10 |
| `REGEXP_REPLACE()` | String | 1 |
| `REPEAT()` | String | 1 |
| `REPLACE()` | String | 1 |
| `REVERSE()` | String | 1 |
| `RIGHT()` | String | 1 |
| `ROUND()` | Numérica | 2 |
| `ROW_COUNT()` | Sistema | 9 |
| `ROW_NUMBER()` | Window | 10 |
| `RPAD()` | String | 1 |
| `RTRIM()` | String | 1 |
| `SECOND()` | Data/Hora | 3 |
| `SHA1()` | Criptografia | 8 |
| `SHA2()` | Criptografia | 8 |
| `SIGN()` | Numérica | 2 |
| `SQRT()` | Numérica | 2 |
| `STD()` / `STDDEV()` | Agregação | 4 |
| `STR_TO_DATE()` | Data/Hora | 3 |
| `SUBSTR()` / `SUBSTRING()` | String | 1 |
| `SUBSTRING_INDEX()` | String | 1 |
| `SUM()` | Agregação / Window | 4, 10 |
| `SYSDATE()` | Data/Hora | 3 |
| `TIME()` | Data/Hora | 3 |
| `TIMESTAMPDIFF()` | Data/Hora | 3 |
| `TRIM()` | String | 1 |
| `TRUNCATE()` | Numérica | 2 |
| `UNHEX()` | Conversão | 6 |
| `UPPER()` | String | 1 |
| `USER()` | Sistema | 9 |
| `VARIANCE()` / `VAR_POP()` | Agregação | 4 |
| `VERSION()` | Sistema | 9 |
| `WEEK()` / `WEEKDAY()` | Data/Hora | 3 |
| `YEAR()` | Data/Hora | 3 |

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
