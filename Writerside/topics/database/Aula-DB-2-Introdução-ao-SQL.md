# Aula 2: Introdução ao SQL

Bem-vindo à Aula 2 do nosso curso de Bancos de Dados! Hoje vamos explorar a linguagem SQL (Structured Query Language), a linguagem universal para trabalhar com bancos de dados relacionais. Nesta aula, você aprenderá os conceitos fundamentais do SQL e criará seu primeiro banco de dados.

## Visão Geral do Tópico

Nesta aula, você vai:
- **Entender o que é SQL:** A linguagem padrão para bancos de dados relacionais
- **Conhecer as categorias de comandos SQL:** DDL, DML, DCL, TCL e DQL
- **Criar seu primeiro banco de dados:** Usando comandos SQL no SQL Server
- **Aprender a sintaxe básica:** Regras e convenções da linguagem SQL
- **Explorar tipos de dados:** Principais tipos de dados no SQL Server

## O que é SQL?

**SQL (Structured Query Language)** é uma linguagem de programação declarativa projetada especificamente para gerenciar e manipular dados em sistemas de gerenciamento de bancos de dados relacionais (SGBDR). Criada nos anos 1970 pela IBM, tornou-se o padrão internacional (ISO/IEC 9075) para trabalhar com bancos de dados.

### Características do SQL

- **Declarativa:** Você especifica "o quê" deseja, não "como" fazer
- **Padronizada:** Funciona em diferentes SGBDs com pequenas variações
- **Poderosa:** Permite desde operações simples até análises complexas
- **Integrada:** Pode ser usada dentro de outras linguagens de programação

### SQL vs Outras Linguagens

Diferente de linguagens procedurais como Python ou Java, o SQL é declarativo:

```SQL
-- SQL (declarativo): você diz O QUÊ quer
SELECT nome, email
FROM clientes
WHERE cidade = 'São Paulo';

-- Em uma linguagem procedural, você diria COMO fazer:
-- for cada cliente na lista:
--     if cliente.cidade == 'São Paulo':
--         imprimir cliente.nome, cliente.email
```

## Categorias de Comandos SQL

O SQL é dividido em cinco categorias principais:

### 1. DDL (Data Definition Language) - Linguagem de Definição de Dados

Comandos para definir e modificar a estrutura do banco de dados:

- **CREATE:** Cria objetos (bancos, tabelas, views, etc.)
- **ALTER:** Modifica estrutura de objetos existentes
- **DROP:** Remove objetos do banco de dados
- **TRUNCATE:** Remove todos os dados de uma tabela rapidamente

```sql
-- Exemplos de DDL
CREATE DATABASE MeuBanco;
CREATE TABLE Clientes (ID INT, Nome VARCHAR(100));
ALTER TABLE Clientes ADD Email VARCHAR(100);
DROP TABLE Clientes;
```

### 2. DML (Data Manipulation Language) - Linguagem de Manipulação de Dados

Comandos para manipular os dados dentro das tabelas:

- **INSERT:** Insere novos registros
- **UPDATE:** Atualiza registros existentes
- **DELETE:** Remove registros
- **MERGE:** Combina INSERT, UPDATE e DELETE em uma operação

```sql
-- Exemplos de DML
INSERT INTO Clientes (ID, Nome) VALUES (1, 'João Silva');
UPDATE Clientes SET Email = 'joao@email.com' WHERE ID = 1;
DELETE FROM Clientes WHERE ID = 1;
```

### 3. DQL (Data Query Language) - Linguagem de Consulta de Dados

O comando mais utilizado no SQL:

- **SELECT:** Consulta e recupera dados

```sql
-- Exemplo de DQL
SELECT Nome, Email
FROM Clientes
WHERE Cidade = 'São Paulo'
ORDER BY Nome;
```

### 4. DCL (Data Control Language) - Linguagem de Controle de Dados

Comandos para controlar permissões e acesso:

- **GRANT:** Concede permissões
- **REVOKE:** Remove permissões
- **DENY:** Nega permissões explicitamente (específico do SQL Server)

```sql
-- Exemplos de DCL
GRANT SELECT ON Clientes TO usuario1;
REVOKE INSERT ON Clientes FROM usuario1;
```

### 5. TCL (Transaction Control Language) - Linguagem de Controle de Transações

Comandos para gerenciar transações:

- **BEGIN TRANSACTION:** Inicia uma transação
- **COMMIT:** Confirma as alterações
- **ROLLBACK:** Desfaz as alterações
- **SAVEPOINT:** Define pontos de salvamento dentro de uma transação

```sql
-- Exemplo de TCL
BEGIN TRANSACTION;
    UPDATE Contas SET Saldo = Saldo - 100 WHERE ID = 1;
    UPDATE Contas SET Saldo = Saldo + 100 WHERE ID = 2;
COMMIT;
```

## Criando Seu Primeiro Banco de Dados

Vamos criar um banco de dados simples para praticar:

<procedure title="Criar seu primeiro banco de dados" id="criar-primeiro-banco">
    <step>
        <p>Abra o SQL Server Management Studio (SSMS)</p>
    </step>
    <step>
        <p>Conecte-se ao servidor SQL</p>
    </step>
    <step>
        <p>Clique em <strong>Nova Consulta</strong> na barra de ferramentas</p>
    </step>
    <step>
        <p>Digite o seguinte código SQL</p>
    </step>
    <step>
        <code-block lang="sql">
        -- Criar um novo banco de dados
        SQL
        CREATE DATABASE EscolaDB;
        GO
        -- Usar o banco de dados criado
        USE EscolaDB;
        GO
        -- Criar uma tabela simples
        CREATE TABLE Alunos (
        AlunoID INT PRIMARY KEY IDENTITY(1,1),
        Nome VARCHAR(100) NOT NULL,
        Email VARCHAR(100),
        DataNascimento DATE,
        Ativo BIT DEFAULT 1
        );
        GO
        </code-block>
    </step>
    <step>
        <p>Execute o código pressionando <shortcut>F5</shortcut> ou clicando em <strong>Executar</strong></p>
    </step>
    <step>
        <p>Verifique no Pesquisador de Objetos: expanda <strong>Bancos de Dados</strong> e você verá <strong>EscolaDB</strong></p>
    </step>
</procedure>

### Explicação do Código

**`CREATE DATABASE EscolaDB;`**
: Cria um novo banco de dados chamado "EscolaDB"

**`GO`**
: Comando específico do SQL Server que separa lotes de comandos. Não faz parte do SQL padrão.

**`USE EscolaDB;`**
: Define EscolaDB como o banco de dados ativo para as próximas operações

**`CREATE TABLE Alunos (...)`**
: Cria uma tabela chamada "Alunos" com colunas específicas

**`AlunoID INT PRIMARY KEY IDENTITY(1,1)`**
: Coluna de ID inteiro, chave primária, com auto-incremento começando em 1

**`Nome VARCHAR(100) NOT NULL`**
: Coluna de texto com até 100 caracteres, obrigatória (não pode ser NULL)

**`Email VARCHAR(100)`**
: Coluna de texto opcional (pode ser NULL)

**`DataNascimento DATE`**
: Coluna para armazenar datas

**`Ativo BIT DEFAULT 1`**
: Coluna booleana (0 ou 1) com valor padrão 1 (verdadeiro)

## Sintaxe e Convenções SQL

### Regras Básicas

1. **Comandos SQL não são case-sensitive** (mas é boa prática usar MAIÚSCULAS para palavras-chave)
   ```sql
   SELECT * FROM Alunos;  -- Recomendado
   select * from alunos;  -- Funciona, mas menos legível
   ```

2. **Ponto e vírgula (;) separa comandos**
   ```sql
   SELECT * FROM Alunos;
   SELECT * FROM Professores;
   ```

3. **Espaços em branco e quebras de linha são ignorados**
   ```sql
   -- Estas duas consultas são equivalentes:
   SELECT Nome, Email FROM Alunos WHERE Ativo = 1;

   SELECT Nome, Email
   FROM Alunos
   WHERE Ativo = 1;
   ```

4. **Comentários**
   ```sql
   -- Comentário de linha única

   /* Comentário
      de múltiplas
      linhas */
   ```

5. **Strings usam aspas simples**
   ```sql
   SELECT * FROM Alunos WHERE Nome = 'João Silva';
   ```

6. **Colchetes [ ] identificam nomes com caracteres especiais ou palavras reservadas**
   ```sql
   SELECT [Order], [Date] FROM [Sales Data];
   ```

### Convenções de Nomenclatura

**Boa prática:**
- **Tabelas:** PascalCase, plural (Alunos, Professores, MateriasAlunos)
- **Colunas:** PascalCase, singular (AlunoID, Nome, DataNascimento)
- **Chaves primárias:** NomeTabela + ID (AlunoID, ProfessorID)
- **Chaves estrangeiras:** NomeTabela + ID (AlunoID em Matriculas)

**Evite:**
- Espaços nos nomes (use "DataNascimento", não "Data Nascimento")
- Caracteres especiais (@, #, %, etc.)
- Palavras reservadas do SQL (SELECT, TABLE, etc.)
- Nomes muito longos ou muito curtos

## Principais Tipos de Dados no SQL Server

### Tipos Numéricos

| Tipo | Descrição | Intervalo | Uso |
|------|-----------|-----------|-----|
| **INT** | Inteiro de 4 bytes | -2.147.483.648 a 2.147.483.647 | IDs, quantidades |
| **BIGINT** | Inteiro de 8 bytes | -9.223.372.036.854.775.808 a 9.223... | Números muito grandes |
| **SMALLINT** | Inteiro de 2 bytes | -32.768 a 32.767 | Números pequenos |
| **TINYINT** | Inteiro de 1 byte | 0 a 255 | Flags, status |
| **DECIMAL(p,s)** | Número decimal preciso | Até 38 dígitos | Valores monetários |
| **FLOAT** | Número de ponto flutuante | Aproximado | Cálculos científicos |

```sql
CREATE TABLE Produtos (
    ProdutoID INT PRIMARY KEY,
    Quantidade SMALLINT,
    Preco DECIMAL(10,2),  -- 10 dígitos, 2 decimais
    Peso FLOAT
);
```

### Tipos de Texto

| Tipo | Descrição | Tamanho Máximo | Uso |
|------|-----------|----------------|-----|
| **CHAR(n)** | Texto de tamanho fixo | 8.000 caracteres | Códigos fixos (CPF, CEP) |
| **VARCHAR(n)** | Texto de tamanho variável | 8.000 caracteres | Textos curtos |
| **VARCHAR(MAX)** | Texto de tamanho variável | 2 GB | Textos longos |
| **NVARCHAR(n)** | Unicode variável | 4.000 caracteres | Textos multilíngues |
| **TEXT** | Texto longo (obsoleto) | 2 GB | Use VARCHAR(MAX) |

```sql
CREATE TABLE Pessoas (
    CPF CHAR(11),              -- Tamanho fixo
    Nome VARCHAR(100),         -- Tamanho variável
    Biografia VARCHAR(MAX),    -- Texto longo
    NomeInternacional NVARCHAR(100)  -- Unicode (支持中文)
);
```

### Tipos de Data e Hora

| Tipo | Descrição | Formato | Uso |
|------|-----------|---------|-----|
| **DATE** | Apenas data | YYYY-MM-DD | Datas de nascimento |
| **TIME** | Apenas hora | HH:MM:SS.nnnnnnn | Horários |
| **DATETIME** | Data e hora | YYYY-MM-DD HH:MM:SS | Timestamps legados |
| **DATETIME2** | Data e hora (melhor) | YYYY-MM-DD HH:MM:SS.nnnnnnn | Timestamps modernos |
| **DATETIMEOFFSET** | Com fuso horário | +/- timezone | Sistemas globais |

```sql
CREATE TABLE Eventos (
    EventoID INT PRIMARY KEY,
    DataEvento DATE,
    HoraInicio TIME,
    DataHoraCriacao DATETIME2 DEFAULT GETDATE(),
    DataHoraGlobal DATETIMEOFFSET
);
```

### Outros Tipos Importantes

| Tipo | Descrição | Uso |
|------|-----------|-----|
| **BIT** | Booleano (0 ou 1) | Flags (Ativo, Publicado) |
| **UNIQUEIDENTIFIER** | GUID (UUID) | Identificadores únicos globais |
| **XML** | Dados XML | Documentos XML |
| **JSON** | Dados JSON | Desde SQL Server 2016 (via VARCHAR) |
| **VARBINARY(MAX)** | Dados binários | Imagens, arquivos |

```sql
CREATE TABLE Configuracoes (
    Ativo BIT DEFAULT 1,
    ChavePublica UNIQUEIDENTIFIER DEFAULT NEWID(),
    DadosXML XML,
    DadosJSON VARCHAR(MAX),
    LogoEmpresa VARBINARY(MAX)
);
```

## Explorando o Banco de Dados Criado

Vamos executar alguns comandos para entender nosso banco:

```sql
-- Ver todas as tabelas do banco
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE';

-- Ver estrutura da tabela Alunos
EXEC sp_help 'Alunos';

-- Ou usar este comando para ver as colunas
SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH, IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Alunos';
```

## Boas Práticas em SQL

1. **Use nomes descritivos e consistentes**
2. **Sempre especifique as colunas** (evite SELECT *)
3. **Comente código complexo**
4. **Use indentação para facilitar leitura**
5. **Teste em ambiente de desenvolvimento primeiro**
6. **Faça backup antes de operações destrutivas**
7. **Use transações para operações críticas**

### Exemplo de Código Bem Formatado

```sql
-- Buscar alunos ativos matriculados em 2024
-- Ordenados por nome, mostrando apenas informações essenciais
SELECT
    A.AlunoID,
    A.Nome,
    A.Email,
    A.DataNascimento,
    DATEDIFF(YEAR, A.DataNascimento, GETDATE()) AS Idade
FROM
    Alunos A
WHERE
    A.Ativo = 1
    AND YEAR(A.DataNascimento) >= 2000
ORDER BY
    A.Nome;
```

## Exercícios Práticos

Pratique criando essas estruturas:

<procedure title="Exercícios de Prática" id="exercicios-pratica">
    <step>
        <p>Crie um banco de dados chamado "BibliotecaDB"</p>
    </step>
    <step>
        <p>Dentro dele, crie uma tabela "Livros" com as colunas:</p>
        <list>
            <li>LivroID (INT, chave primária, auto-incremento)</li>
            <li>Titulo (VARCHAR(200), obrigatório)</li>
            <li>Autor (VARCHAR(100))</li>
            <li>ISBN (CHAR(13))</li>
            <li>AnoPublicacao (SMALLINT)</li>
            <li>Preco (DECIMAL(10,2))</li>
            <li>Disponivel (BIT, padrão 1)</li>
        </list>
    </step>
    <step>
        <p>Crie outra tabela "Autores" com:</p>
        <list>
            <li>AutorID (INT, chave primária, auto-incremento)</li>
            <li>Nome (VARCHAR(100), obrigatório)</li>
            <li>Nacionalidade (VARCHAR(50))</li>
            <li>DataNascimento (DATE)</li>
        </list>
    </step>
</procedure>

## Glossary

**SQL (Structured Query Language)**
: Linguagem padrão para gerenciar e manipular bancos de dados relacionais

**DDL (Data Definition Language)**
: Comandos que definem a estrutura do banco (CREATE, ALTER, DROP)

**DML (Data Manipulation Language)**
: Comandos que manipulam os dados (INSERT, UPDATE, DELETE)

**DQL (Data Query Language)**
: Comando para consultar dados (SELECT)

**VARCHAR**
: Tipo de dado para texto de tamanho variável

**PRIMARY KEY**
: Restrição que identifica unicamente cada registro na tabela

**IDENTITY**
: Propriedade que gera valores auto-incrementais para uma coluna

**NULL**
: Ausência de valor em uma coluna

**GO**
: Delimitador de lote específico do SQL Server e SSMS

**Schema**
: Organização lógica de objetos em um banco de dados

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/language-reference">Referência T-SQL Microsoft</a>
        <a href="https://www.w3schools.com/sql/">W3Schools - SQL Tutorial</a>
        <a href="https://sqlservertutorial.net/">SQL Server Tutorial</a>
        <a href="https://docs.microsoft.com/pt-br/sql/t-sql/data-types/data-types-transact-sql">Tipos de Dados SQL Server</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique criar diferentes bancos e tabelas
> - Experimente todos os tipos de dados apresentados
> - Explore as propriedades das tabelas no SSMS
> - Prepare-se para aprender sobre modelagem de dados!
>
{style="tip"}
