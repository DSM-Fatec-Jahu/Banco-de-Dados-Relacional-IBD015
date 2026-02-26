# Aula 14 — Backup, Restauração e Segurança

> **IBD015 — Banco de Dados Relacional** · Fatec Jahu · Prof. Ronan Adriel Zenatti
> [← Aula 13](./Aula_13_Functions_Transacoes.md) · [Voltar ao README](../README.md) · [Próxima Aula →](./Aula_15_Otimizacao_Indices.md)

---

## 📌 Objetivos da Aula

Ao final desta aula, você será capaz de realizar backup e restauração de bancos de dados no MariaDB usando `mysqldump`, entender as diferenças entre os tipos de backup, criar usuários com privilégios adequados e aplicar boas práticas de segurança em SGBDs.

---

## 🎥 Vídeos de Apoio

- 📺 [Backup e Restore com mysqldump](https://www.youtube.com/watch?v=8Ib1tRjX6EA) — Bóson Treinamentos
- 📺 [Usuários e Permissões no MySQL](https://www.youtube.com/watch?v=OFcpg-wfOzs) — CFBCursos

---

## 1. Estratégias de Backup

[prompt para nanobanana: "Timeline diagram showing three database backup strategies. Top row: 'Backup Completo' - full database cylinder icon on Sunday, labeled 'Copia tudo - Mais lento, arquivo maior'. Middle row: 'Backup Incremental' - small delta icons on Mon/Tue/Wed/Thu/Fri/Sat, labeled 'Apenas mudanças desde o último backup'. Bottom row: 'Backup Diferencial' - medium icons growing in size Mon through Sat, labeled 'Mudanças desde o último backup COMPLETO'. A restore scenario at the bottom shows: Completo needs Sunday only, Incremental needs Sunday+each day, Diferencial needs Sunday+latest differential. Clean educational flat design, blue/green/orange color coding, white background, Portuguese labels."]
![Estratégias de Backup](../imgs/Aula_14_img_01.png)

**Backup Completo:** copia todo o banco de dados. É a base de qualquer estratégia. Mais lento e maior, mas a restauração é simples — apenas um arquivo.

**Backup Incremental:** copia apenas os dados alterados desde o **último backup** (completo ou incremental). Mais rápido e menor, mas a restauração é complexa (precisa aplicar o completo + cada incremental em sequência).

**Backup Diferencial:** copia os dados alterados desde o **último backup completo**. Equilíbrio entre os dois: a restauração precisa do completo + apenas o diferencial mais recente.

---

## 2. Backup com mysqldump

O `mysqldump` é a ferramenta padrão para backup lógico no MariaDB/MySQL. Ele gera um arquivo SQL com os comandos para recriar o banco.

```bash
# Backup completo de um banco específico
mysqldump -u root -p ecommerce > backup_ecommerce_2026-03-15.sql

# Backup com compressão (recomendado para bancos grandes)
mysqldump -u root -p ecommerce | gzip > backup_ecommerce_2026-03-15.sql.gz

# Backup de múltiplos bancos
mysqldump -u root -p --databases ecommerce escola rh > backup_multiplos.sql

# Backup de TODOS os bancos do servidor
mysqldump -u root -p --all-databases > backup_completo_servidor.sql

# Backup apenas da estrutura (sem dados) — útil para documentação
mysqldump -u root -p --no-data ecommerce > estrutura_ecommerce.sql

# Backup apenas dos dados (sem DDL) — útil para migração
mysqldump -u root -p --no-create-info ecommerce > dados_ecommerce.sql

# Backup com opções recomendadas para produção:
mysqldump -u root -p \
    --single-transaction \   # garante consistência sem travar tabelas (InnoDB)
    --routines \             # inclui procedures e functions
    --triggers \             # inclui triggers
    --events \               # inclui events agendados
    --add-drop-database \    # adiciona DROP DATABASE antes do CREATE
    ecommerce > backup_completo.sql
```

> 💡 **`--single-transaction`:** esta opção inicia uma transação de leitura no InnoDB antes do dump, garantindo um snapshot consistente dos dados sem travar as tabelas. **Essencial em produção** para não bloquear usuários durante o backup.

---

## 3. Restauração

```bash
# Restaurar um backup SQL
mysql -u root -p ecommerce < backup_ecommerce_2026-03-15.sql

# Restaurar backup comprimido
gunzip -c backup_ecommerce_2026-03-15.sql.gz | mysql -u root -p ecommerce

# Restaurar backup de todos os bancos
mysql -u root -p < backup_completo_servidor.sql
```

```sql
-- Dentro do MariaDB: restaurar via SOURCE
SOURCE /caminho/completo/backup_ecommerce.sql;

-- Verificar após restauração
SHOW DATABASES;
USE ecommerce;
SHOW TABLES;
SELECT COUNT(*) FROM pedidos;
```

---

## 4. Controle de Acesso — Usuários e Privilégios

O princípio do **mínimo privilégio** diz que cada usuário deve ter apenas as permissões necessárias para executar suas funções — nem mais, nem menos.

### 4.1 Criando Usuários

```sql
-- Sintaxe básica
-- MariaDB e MySQL >= 5.7:
CREATE USER 'nome_usuario'@'host' IDENTIFIED BY 'senha_forte_aqui';

-- host define de onde o usuário pode se conectar:
-- 'localhost'  = apenas da mesma máquina
-- '127.0.0.1'  = apenas via IPv4 local
-- '%'          = de qualquer host (cuidado em produção!)
-- '192.168.1.%' = de qualquer IP na sub-rede 192.168.1.x

-- Exemplos:
CREATE USER 'app_ecommerce'@'localhost'  IDENTIFIED BY 'SenhaApp@2026!';
CREATE USER 'relatorios'@'192.168.1.%'  IDENTIFIED BY 'SenhaRel@2026!';
CREATE USER 'admin_bd'@'localhost'       IDENTIFIED BY 'SenhaAdm@2026!';
```

> ⚠️ **Diferença MariaDB vs MySQL:** no MariaDB 10.4+, o usuário `root` usa o plugin `unix_socket` por padrão (autenticação pelo SO), o que impede login com senha via terminal a menos que você conecte como root do sistema. No MySQL, o root usa autenticação por senha tradicional. Para criar usuários administrativos com senha no MariaDB, use `mysql -u root` (sem -p) ou crie via phpMyAdmin.

### 4.2 Concedendo Privilégios com GRANT

```sql
-- Privilégios disponíveis: SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX,
-- ALTER, CREATE ROUTINE, EXECUTE, TRIGGER, CREATE VIEW, SHOW VIEW, ALL PRIVILEGES

-- Usuário de aplicação: apenas leitura e escrita de dados (sem DDL)
GRANT SELECT, INSERT, UPDATE, DELETE
    ON ecommerce.*
    TO 'app_ecommerce'@'localhost';

-- Usuário de relatórios: apenas leitura
GRANT SELECT
    ON ecommerce.*
    TO 'relatorios'@'192.168.1.%';

-- Usuário administrador: tudo no banco ecommerce
GRANT ALL PRIVILEGES
    ON ecommerce.*
    TO 'admin_bd'@'localhost';

-- Permissão em tabela específica (mais restritivo ainda)
GRANT SELECT
    ON ecommerce.vw_produtos_catalogo
    TO 'relatorios'@'192.168.1.%';

-- Aplicar as permissões imediatamente
FLUSH PRIVILEGES;
```

### 4.3 Revogando Privilégios com REVOKE

```sql
-- Remover uma permissão específica
REVOKE DELETE
    ON ecommerce.*
    FROM 'app_ecommerce'@'localhost';

-- Verificar os privilégios de um usuário
SHOW GRANTS FOR 'app_ecommerce'@'localhost';
```

### 4.4 Removendo Usuários

```sql
-- Remove o usuário e todos os seus privilégios
DROP USER IF EXISTS 'relatorios'@'192.168.1.%';
```

---

## 5. Boas Práticas de Segurança

[prompt para nanobanana: "Security checklist infographic for database administration with checkboxes and icons. Items: 1) Senha forte para root (lock icon), 2) Remover usuário anonymous (user-x icon), 3) Princípio do mínimo privilégio (shield icon), 4) Backup regular testado (cloud-backup icon), 5) Conexão criptografada em produção (ssl lock icon), 6) Logs de auditoria ativados (log file icon), 7) Atualizar o SGBD regularmente (update arrow icon). Clean flat design, green checkmarks, white background, Portuguese labels."]
![Boas Práticas de Segurança em BD](../imgs/Aula_14_img_02.png)

```sql
-- 1. Verificar usuários sem senha (nunca deve existir em produção)
SELECT User, Host, authentication_string
FROM   mysql.user
WHERE  authentication_string = '' OR authentication_string IS NULL;

-- 2. Remover o usuário anônimo (criado por padrão em algumas instalações)
DELETE FROM mysql.user WHERE User = '';
FLUSH PRIVILEGES;

-- 3. Alterar a senha do root (se ainda não tiver):
ALTER USER 'root'@'localhost' IDENTIFIED BY 'SenhaRootForte@2026!';

-- 4. Nunca use 'root' para conexões de aplicação
-- Crie sempre usuários específicos com o mínimo de privilégios necessários

-- 5. Verificar conexões ativas
SHOW PROCESSLIST;

-- 6. Ver configurações de segurança atuais
SHOW VARIABLES LIKE '%password%';
SHOW VARIABLES LIKE 'validate_password%';
```

---

## 6. Automatizando Backups (Linux/XAMPP)

```bash
#!/bin/bash
# Script: backup_diario.sh
# Execute via cron para backups automáticos

DATA=$(date +%Y-%m-%d)
DIR_BACKUP="/backups/mariadb"
USUARIO="root"
SENHA="sua_senha"
BANCO="ecommerce"

# Cria o diretório se não existir
mkdir -p $DIR_BACKUP

# Realiza o backup com compressão
mysqldump -u $USUARIO -p$SENHA \
    --single-transaction \
    --routines \
    --triggers \
    $BANCO | gzip > $DIR_BACKUP/backup_${BANCO}_${DATA}.sql.gz

# Remove backups com mais de 30 dias
find $DIR_BACKUP -name "*.sql.gz" -mtime +30 -delete

echo "Backup concluído: backup_${BANCO}_${DATA}.sql.gz"
```

```bash
# Agendar via cron (todo dia às 02:00)
# crontab -e
# 0 2 * * * /scripts/backup_diario.sh >> /logs/backup.log 2>&1
```

---

## 7. Exercícios de Fixação

**Exercício 1:** faça um backup completo do banco `ecommerce` usando `mysqldump` com as opções `--single-transaction`, `--routines` e `--triggers`. Depois, crie um banco `ecommerce_teste` e restaure o backup nele.

**Exercício 2:** crie um usuário `dev_frontend` que tenha apenas permissão de `SELECT` nas views `vw_pedidos_completos` e `vw_produtos_catalogo` do banco `ecommerce`.

**Exercício 3:** qual a diferença entre backup lógico (mysqldump) e backup físico (cópia dos arquivos do datadir)? Quais são as vantagens e desvantagens de cada um?

---

## 📚 Referências

- SILBERSCHATZ, A.; KORTH, H. F. *Sistema de banco de dados*. 6 ed. Cap. 17 — Recuperação de Falhas. Rio de Janeiro: Elsevier, 2016.
- Documentação MariaDB — [mysqldump](https://mariadb.com/kb/en/mysqldump/)

---

> **Próxima aula:** [Aula 15 — Otimização de Consultas e Índices](./Aula_15_Otimizacao_Indices.md)

---

<div align="center">
  <sub>Fatec Jahu · IBD015 — Banco de Dados Relacional · Prof. Ronan Adriel Zenatti · 2026</sub>
</div>
