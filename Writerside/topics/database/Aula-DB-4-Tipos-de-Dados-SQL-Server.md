# Aula 4: Tipos de Dados no SQL Server

## Introdução

Escolher o tipo de dado correto para cada coluna é fundamental para criar bancos de dados eficientes, seguros e escaláveis. O SQL Server oferece uma ampla variedade de tipos de dados para diferentes necessidades. Nesta aula, exploraremos detalhadamente cada categoria de tipos de dados, suas características, vantagens e quando utilizá-los.

## Categorias de Tipos de Dados

O SQL Server organiza os tipos de dados em categorias:

1. **Numéricos Exatos:** INT, BIGINT, DECIMAL, NUMERIC
2. **Numéricos Aproximados:** FLOAT, REAL
3. **Data e Hora:** DATE, TIME, DATETIME, DATETIME2, DATETIMEOFFSET
4. **Caracteres:** CHAR, VARCHAR, TEXT
5. **Unicode:** NCHAR, NVARCHAR, NTEXT
6. **Binários:** BINARY, VARBINARY, IMAGE
7. **Especiais:** BIT, UNIQUEIDENTIFIER, XML, JSON, SPATIAL

## Tipos Numéricos Exatos

### Inteiros

| Tipo | Bytes | Intervalo | Uso Recomendado |
|------|-------|-----------|-----------------|
| **TINYINT** | 1 | 0 a 255 | Status, flags, idades |
| **SMALLINT** | 2 | -32,768 a 32,767 | Quantidades pequenas |
| **INT** | 4 | -2,147,483,648 a 2,147,483,647 | IDs, quantidades gerais |
| **BIGINT** | 8 | -9,223,372,036,854,775,808 a 9,223... | IDs muito grandes, estatísticas |

```sql
CREATE TABLE Exemplo_Inteiros (
    Status TINYINT,              -- 0-255: suficiente para status
    Quantidade SMALLINT,         -- Estoque de produtos
    UsuarioID INT,               -- ID comum
    VisualizacoesTotais BIGINT   -- Contadores muito grandes
);
```

**Escolhendo o tipo certo:**
- Use o menor tipo que comporta seus dados
- INT é o mais comum para IDs
- BIGINT quando espera crescimento massivo
- TINYINT para economizar espaço em tabelas grandes

### Decimais e Numéricos

| Tipo | Descrição | Uso |
|------|-----------|-----|
| **DECIMAL(p,s)** | Precisão e escala fixas | Valores monetários |
| **NUMERIC(p,s)** | Sinônimo de DECIMAL | Idêntico ao DECIMAL |
| **MONEY** | Moeda (8 bytes) | Legado, use DECIMAL |
| **SMALLMONEY** | Moeda (4 bytes) | Legado, use DECIMAL |

**Parâmetros:**
- **p (precisão):** Número total de dígitos (1-38)
- **s (escala):** Número de dígitos após a vírgula (0-p)

```sql
CREATE TABLE Financeiro (
    Preco DECIMAL(10,2),          -- Até 99,999,999.99
    Taxa DECIMAL(5,4),             -- 0.0000 a 9.9999 (taxas percentuais)
    Salario DECIMAL(12,2),         -- Salários até 9,999,999,999.99
    Imposto DECIMAL(18,2),         -- Valores muito grandes
    PrecoLegado MONEY              -- Evite usar (legado)
);
```

**Boas práticas:**
- SEMPRE use DECIMAL para valores monetários (nunca FLOAT)
- DECIMAL(18,2) é comum para preços
- DECIMAL(5,4) para taxas percentuais (ex: 0.0525 = 5.25%)

## Tipos Numéricos Aproximados

| Tipo | Bytes | Precisão | Uso |
|------|-------|----------|-----|
| **REAL** | 4 | 7 dígitos | Cálculos científicos simples |
| **FLOAT(n)** | 4 ou 8 | 15 dígitos | Cálculos científicos, coordenadas |

```sql
CREATE TABLE Cientif ico (
    Temperatura REAL,              -- Leituras de sensores
    Latitude FLOAT,                -- Coordenadas geográficas
    Longitude FLOAT,
    ValorCientifico FLOAT(53)      -- Máxima precisão
);
```

⚠️ **CUIDADO:** Tipos aproximados podem ter imprecisão em cálculos!

```sql
-- Exemplo de imprecisão com FLOAT
DECLARE @f FLOAT = 0.1;
SELECT @f + @f + @f;  -- Pode resultar em 0.30000000000000004

-- Use DECIMAL para precisão
DECLARE @d DECIMAL(10,2) = 0.1;
SELECT @d + @d + @d;  -- Sempre 0.30
```

## Tipos de Data e Hora

### Comparação Completa

| Tipo | Bytes | Intervalo | Precisão | Uso |
|------|-------|-----------|----------|-----|
| **DATE** | 3 | 0001-01-01 a 9999-12-31 | Dia | Datas de nascimento, vencimentos |
| **TIME** | 3-5 | 00:00:00.0000000 a 23:59:59.9999999 | 100ns | Horários, durações |
| **DATETIME** | 8 | 1753-01-01 a 9999-12-31 | 3.33ms | Legado (use DATETIME2) |
| **DATETIME2** | 6-8 | 0001-01-01 a 9999-12-31 | 100ns | Timestamps modernos |
| **DATETIMEOFFSET** | 10 | 0001-01-01 a 9999-12-31 | 100ns | Sistemas globais com fuso |
| **SMALLDATETIME** | 4 | 1900-01-01 a 2079-06-06 | 1min | Legado, evite |

```sql
CREATE TABLE Agenda (
    -- Apenas data
    DataNascimento DATE,
    DataVencimento DATE,

    -- Apenas hora
    HoraAbertura TIME,
    HoraFechamento TIME,

    -- Data + hora (use DATETIME2, não DATETIME)
    DataCriacao DATETIME2 DEFAULT GETDATE(),
    UltimaAtualizacao DATETIME2,

    -- Com fuso horário (para aplicações globais)
    EventoGlobal DATETIMEOFFSET,

    -- Legado (evite)
    CampoAntigo DATETIME,
    CampoAntigoSimples SMALLDATETIME
);
```

### Funções de Data e Hora

```sql
-- Data/hora atual
SELECT GETDATE();           -- Retorna DATETIME
SELECT SYSDATETIME();       -- Retorna DATETIME2 (mais preciso)
SELECT GETUTCDATE();        -- UTC
SELECT SYSDATETIMEOFFSET(); -- Com fuso horário

-- Extrair partes
SELECT YEAR(GETDATE());     -- Ano
SELECT MONTH(GETDATE());    -- Mês
SELECT DAY(GETDATE());      -- Dia
SELECT DATEPART(HOUR, GETDATE());  -- Hora

-- Manipular datas
SELECT DATEADD(DAY, 7, GETDATE());      -- Adicionar 7 dias
SELECT DATEADD(MONTH, -1, GETDATE());   -- Subtrair 1 mês
SELECT DATEDIFF(DAY, '2020-01-01', GETDATE());  -- Diferença em dias

-- Formatar datas
SELECT FORMAT(GETDATE(), 'dd/MM/yyyy');
SELECT FORMAT(GETDATE(), 'dd/MM/yyyy HH:mm:ss');
SELECT CONVERT(VARCHAR, GETDATE(), 103);  -- Formato britânico
```

## Tipos de Texto (Caracteres)

### Comparação

| Tipo | Tamanho | Max | Codificação | Uso |
|------|---------|-----|-------------|-----|
| **CHAR(n)** | Fixo | 8,000 | ANSI | Códigos fixos (CPF, CEP) |
| **VARCHAR(n)** | Variável | 8,000 | ANSI | Textos curtos |
| **VARCHAR(MAX)** | Variável | 2GB | ANSI | Textos longos |
| **NCHAR(n)** | Fixo | 4,000 | Unicode | Códigos fixos multilíngues |
| **NVARCHAR(n)** | Variável | 4,000 | Unicode | Textos multilíngues |
| **NVARCHAR(MAX)** | Variável | 2GB | Unicode | Textos longos multilíngues |
| **TEXT** | Variável | 2GB | ANSI | Obsoleto! Use VARCHAR(MAX) |
| **NTEXT** | Variável | 2GB | Unicode | Obsoleto! Use NVARCHAR(MAX) |

```sql
CREATE TABLE Textos (
    -- CHAR: tamanho fixo (sempre usa n bytes)
    CPF CHAR(11),                -- Sempre 11 caracteres
    CEP CHAR(8),                 -- Sempre 8 caracteres
    Sigla CHAR(2),               -- Estado: "SP", "RJ"

    -- VARCHAR: tamanho variável (economiza espaço)
    Nome VARCHAR(100),           -- Até 100 caracteres
    Email VARCHAR(255),
    Descricao VARCHAR(MAX),      -- Textos longos

    -- NVARCHAR: suporta Unicode (chinês, árabe, emoji)
    NomeInternacional NVARCHAR(100),  -- 你好, مرحبا, 😀
    Observacoes NVARCHAR(MAX),

    -- NÃO USE ISSO (obsoleto)
    TextoAntigo TEXT             -- ❌ Use VARCHAR(MAX)
);
```

**CHAR vs VARCHAR:**
```sql
-- CHAR(10): sempre 10 bytes
INSERT INTO Teste (Campo_CHAR) VALUES ('ABC');  -- Armazena 'ABC       ' (7 espaços)

-- VARCHAR(10): tamanho variável
INSERT INTO Teste (Campo_VARCHAR) VALUES ('ABC');  -- Armazena 'ABC' (3 bytes + overhead)
```

**VARCHAR vs NVARCHAR:**
```sql
-- VARCHAR: 1 byte por caractere (apenas caracteres ocidentais)
DECLARE @v VARCHAR(10) = 'Hello';  -- 5 bytes

-- NVARCHAR: 2 bytes por caractere (suporta qualquer idioma)
DECLARE @nv NVARCHAR(10) = 'こんにちは';  -- 10 bytes (5 caracteres japoneses)
```

**Quando usar cada um:**
- **CHAR:** Dados de tamanho fixo conhecido (CPF, CEP, códigos)
- **VARCHAR:** Textos de tamanho variável (nomes, endereços)
- **VARCHAR(MAX):** Textos muito longos (artigos, descrições)
- **NVARCHAR:** Quando precisa suportar múltiplos idiomas ou emoji

## Tipos Binários

| Tipo | Tamanho | Max | Uso |
|------|---------|-----|-----|
| **BINARY(n)** | Fixo | 8,000 | Hashes, dados binários fixos |
| **VARBINARY(n)** | Variável | 8,000 | Dados binários pequenos |
| **VARBINARY(MAX)** | Variável | 2GB | Imagens, arquivos, documentos |
| **IMAGE** | Variável | 2GB | Obsoleto! Use VARBINARY(MAX) |

```sql
CREATE TABLE Arquivos (
    -- Hash de senha (sempre 32 bytes para SHA-256)
    SenhaHash BINARY(32),

    -- Documentos e imagens
    FotoPerfil VARBINARY(MAX),
    Documento VARBINARY(MAX),

    -- Pequenos dados binários
    Token VARBINARY(50),

    -- NÃO USE (obsoleto)
    ImagemAntiga IMAGE  -- ❌ Use VARBINARY(MAX)
);

-- Exemplo: armazenando arquivo
INSERT INTO Arquivos (Documento)
SELECT BulkColumn
FROM OPENROWSET(BULK 'C:\arquivo.pdf', SINGLE_BLOB) AS DocumentoFile;
```

⚠️ **Importante:** Para arquivos grandes, considere armazenar apenas o caminho e guardar o arquivo em sistema de arquivos ou blob storage.

## Tipos Especiais

### BIT (Booleano)

```sql
CREATE TABLE Configuracoes (
    Ativo BIT DEFAULT 1,         -- True/False
    Publicado BIT,
    EmailVerificado BIT
);

-- Valores possíveis: 0, 1, ou NULL
INSERT INTO Configuracoes VALUES (1, 0, NULL);

-- Consulta
SELECT * FROM Configuracoes WHERE Ativo = 1;
```

### UNIQUEIDENTIFIER (GUID)

```sql
CREATE TABLE Documentos (
    DocumentoID UNIQUEIDENTIFIER DEFAULT NEWID(),
    ChavePublica UNIQUEIDENTIFIER
);

-- Gerar novo GUID
INSERT INTO Documentos (ChavePublica) VALUES (NEWID());

-- Formato: 6F9619FF-8B86-D011-B42D-00C04FC964FF
```

**Vantagens:**
- Globalmente único
- Útil em sistemas distribuídos
- Não precisa de servidor central para gerar

**Desvantagens:**
- Ocupa 16 bytes (vs 4 bytes do INT)
- Performance inferior ao INT como chave primária
- Não é sequencial (impacta índices)

### XML

```sql
CREATE TABLE Configuracoes (
    ConfigID INT PRIMARY KEY,
    ConfigXML XML
);

INSERT INTO Configuracoes VALUES (1, '<config><timeout>30</timeout></config>');

-- Consultar XML
SELECT ConfigXML.value('(/config/timeout)[1]', 'INT') AS Timeout
FROM Configuracoes;
```

### JSON (desde SQL Server 2016)

```sql
CREATE TABLE Logs (
    LogID INT PRIMARY KEY,
    DadosJSON NVARCHAR(MAX)  -- SQL Server armazena JSON como texto
);

INSERT INTO Logs VALUES (1, '{"usuario": "joao", "acao": "login", "ip": "192.168.1.1"}');

-- Consultar JSON
SELECT
    JSON_VALUE(DadosJSON, '$.usuario') AS Usuario,
    JSON_VALUE(DadosJSON, '$.acao') AS Acao
FROM Logs;

-- Validar JSON
SELECT * FROM Logs WHERE ISJSON(DadosJSON) = 1;
```

## Conversão entre Tipos

### CAST e CONVERT

```sql
-- CAST (padrão SQL)
SELECT CAST('123' AS INT);
SELECT CAST(123.45 AS INT);  -- Trunca: 123
SELECT CAST(GETDATE() AS DATE);

-- CONVERT (específico SQL Server, permite formato)
SELECT CONVERT(INT, '123');
SELECT CONVERT(VARCHAR, GETDATE(), 103);  -- Formato: dd/mm/yyyy
SELECT CONVERT(VARCHAR, GETDATE(), 120);  -- Formato: yyyy-mm-dd HH:mm:ss
```

### TRY_CAST e TRY_CONVERT (seguro)

```sql
-- Retorna NULL se conversão falhar (ao invés de erro)
SELECT TRY_CAST('abc' AS INT);  -- Retorna NULL
SELECT TRY_CONVERT(INT, 'xyz'); -- Retorna NULL

-- Útil para validação
SELECT
    Valor,
    CASE WHEN TRY_CAST(Valor AS INT) IS NOT NULL
         THEN 'Válido'
         ELSE 'Inválido'
    END AS Status
FROM TabelaComTexto;
```

## Boas Práticas

### Escolhendo Tipos de Dados

1. **Use o tipo mais específico**
   ```sql
   -- ❌ Ruim
   DataNascimento VARCHAR(50)

   -- ✅ Bom
   DataNascimento DATE
   ```

2. **Use o menor tipo que atende**
   ```sql
   -- ❌ Desperdiça espaço
   Idade BIGINT

   -- ✅ Suficiente
   Idade TINYINT
   ```

3. **DECIMAL para dinheiro, nunca FLOAT**
   ```sql
   -- ❌ NUNCA faça isso
   Preco FLOAT

   -- ✅ Sempre use DECIMAL
   Preco DECIMAL(18,2)
   ```

4. **DATETIME2 ao invés de DATETIME**
   ```sql
   -- ❌ Legado
   DataCriacao DATETIME

   -- ✅ Moderno, mais preciso
   DataCriacao DATETIME2
   ```

5. **VARCHAR ao invés de CHAR para textos variáveis**
   ```sql
   -- ❌ Desperdiça espaço
   Nome CHAR(100)  -- Sempre 100 bytes

   -- ✅ Economiza espaço
   Nome VARCHAR(100)  -- Usa apenas o necessário
   ```

6. **NVARCHAR para suporte multilíngue**
   ```sql
   -- ❌ Não suporta Unicode
   Descricao VARCHAR(MAX)

   -- ✅ Suporta qualquer idioma
   Descricao NVARCHAR(MAX)
   ```

### Impacto no Armazenamento

```sql
-- Exemplo: tabela com 1 milhão de registros

-- Opção 1: Tipos grandes
CREATE TABLE Ineficiente (
    ID BIGINT,               -- 8 bytes
    Nome NVARCHAR(500),      -- Até 1000 bytes
    Idade INT,               -- 4 bytes
    Ativo INT                -- 4 bytes
);
-- Tamanho mínimo por linha: ~1016 bytes
-- 1M registros: ~1 GB

-- Opção 2: Tipos otimizados
CREATE TABLE Otimizada (
    ID INT,                  -- 4 bytes
    Nome VARCHAR(100),       -- Até 100 bytes
    Idade TINYINT,           -- 1 byte
    Ativo BIT                -- 1 byte
);
-- Tamanho mínimo por linha: ~106 bytes
-- 1M registros: ~100 MB
-- ECONOMIA: 90%!
```

## Glossary

**Tipo de Dado**
: Define que tipo de informação pode ser armazenada em uma coluna

**Precisão**
: Número total de dígitos em um número decimal

**Escala**
: Número de dígitos após a vírgula em um número decimal

**ANSI**
: Codificação de caracteres padrão (1 byte por caractere)

**Unicode**
: Codificação que suporta caracteres de qualquer idioma (2 bytes por caractere)

**GUID (Globally Unique Identifier)**
: Identificador único de 128 bits gerado algoritmicamente

**Conversão Implícita**
: SQL Server converte automaticamente entre tipos compatíveis

**Conversão Explícita**
: Desenvolvedor especifica conversão usando CAST ou CONVERT

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/data-types/data-types-transact-sql">Microsoft Docs - Tipos de Dados</a>
        <a href="https://www.sqlshack.com/sql-server-data-types/">SQL Server Data Types Guide</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique criar tabelas com diferentes tipos de dados
> - Experimente conversões entre tipos
> - Analise o impacto de tipos no tamanho do banco
> - Compare performance de diferentes tipos
>
{style="tip"}
