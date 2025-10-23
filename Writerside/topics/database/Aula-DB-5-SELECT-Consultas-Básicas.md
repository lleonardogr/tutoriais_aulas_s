# Aula 5: SELECT e Consultas Básicas

## Introdução

O comando **SELECT** é o comando mais utilizado em SQL e permite recuperar (consultar) dados armazenados nas tabelas. Dominar o SELECT é fundamental para qualquer profissional que trabalha com bancos de dados, pois é através dele que extraímos informações valiosas para análises, relatórios e aplicações.

Nesta aula, você aprenderá:
- A sintaxe completa do SELECT
- Como filtrar, ordenar e limitar resultados
- Operadores e expressões
- Funções básicas e agregadas
- Boas práticas em consultas

## Sintaxe Básica do SELECT

A sintaxe completa do SELECT segue esta ordem (nem todas as cláusulas são obrigatórias):

```sql
SELECT [DISTINCT] [TOP n] colunas
FROM tabela
WHERE condição
GROUP BY colunas
HAVING condição
ORDER BY colunas [ASC|DESC]
OFFSET n ROWS FETCH NEXT m ROWS ONLY;
```

### SELECT Simples

```sql
-- Selecionar todas as colunas (evite em produção!)
SELECT * FROM Alunos;

-- Melhor prática: especificar colunas
SELECT AlunoID, Nome, Email, DataNascimento
FROM Alunos;

-- Selecionar com alias (renomear colunas no resultado)
SELECT
    AlunoID AS ID,
    Nome AS NomeCompleto,
    Email AS EnderecoEmail
FROM Alunos;
```

## Criando Dados de Exemplo

Antes de prosseguir, vamos criar um banco de dados com dados de exemplo:

<procedure title="Criar banco de dados de exemplo" id="criar-banco-exemplo">
    <step>
        <p>Abra o SQL Server Management Studio (SSMS)</p>
    </step>
    <step>
        <p>Execute o script abaixo para criar o banco e popular com dados:</p>
        <code-block lang="sql">
-- Criar banco de dados
CREATE DATABASE LojaDB;
GO

USE LojaDB;
GO

-- Criar tabela de Clientes
CREATE TABLE Clientes (
    ClienteID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100) NOT NULL,
    Email VARCHAR(100),
    Cidade NVARCHAR(50),
    Estado CHAR(2),
    DataCadastro DATE DEFAULT GETDATE(),
    Ativo BIT DEFAULT 1
);

-- Criar tabela de Produtos
CREATE TABLE Produtos (
    ProdutoID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100) NOT NULL,
    Categoria NVARCHAR(50),
    Preco DECIMAL(10,2) NOT NULL,
    Estoque INT DEFAULT 0,
    Ativo BIT DEFAULT 1
);

-- Criar tabela de Pedidos
CREATE TABLE Pedidos (
    PedidoID INT PRIMARY KEY IDENTITY(1,1),
    ClienteID INT NOT NULL,
    DataPedido DATE DEFAULT GETDATE(),
    ValorTotal DECIMAL(10,2),
    Status NVARCHAR(20) DEFAULT 'Pendente',
    FOREIGN KEY (ClienteID) REFERENCES Clientes(ClienteID)
);

-- Inserir dados de exemplo em Clientes
INSERT INTO Clientes (Nome, Email, Cidade, Estado, DataCadastro, Ativo)
VALUES
    ('João Silva', 'joao@email.com', 'São Paulo', 'SP', '2023-01-15', 1),
    ('Maria Santos', 'maria@email.com', 'Rio de Janeiro', 'RJ', '2023-02-20', 1),
    ('Pedro Oliveira', 'pedro@email.com', 'Belo Horizonte', 'MG', '2023-03-10', 1),
    ('Ana Costa', 'ana@email.com', 'São Paulo', 'SP', '2023-04-05', 1),
    ('Carlos Lima', 'carlos@email.com', 'Curitiba', 'PR', '2023-05-12', 0),
    ('Juliana Ferreira', 'juliana@email.com', 'Porto Alegre', 'RS', '2023-06-18', 1),
    ('Roberto Alves', 'roberto@email.com', 'Salvador', 'BA', '2023-07-22', 1),
    ('Fernanda Rocha', 'fernanda@email.com', 'São Paulo', 'SP', '2023-08-30', 1),
    ('Lucas Martins', 'lucas@email.com', 'Brasília', 'DF', '2023-09-14', 1),
    ('Patricia Souza', 'patricia@email.com', 'Recife', 'PE', '2023-10-25', 0);

-- Inserir dados de exemplo em Produtos
INSERT INTO Produtos (Nome, Categoria, Preco, Estoque, Ativo)
VALUES
    ('Notebook Dell', 'Eletrônicos', 3500.00, 15, 1),
    ('Mouse Logitech', 'Eletrônicos', 85.00, 50, 1),
    ('Teclado Mecânico', 'Eletrônicos', 450.00, 30, 1),
    ('Monitor LG 24"', 'Eletrônicos', 899.00, 20, 1),
    ('Cadeira Gamer', 'Móveis', 1200.00, 10, 1),
    ('Mesa para Computador', 'Móveis', 550.00, 8, 1),
    ('Headset HyperX', 'Eletrônicos', 350.00, 25, 1),
    ('Webcam Logitech', 'Eletrônicos', 280.00, 12, 1),
    ('SSD 500GB', 'Eletrônicos', 400.00, 40, 1),
    ('Memória RAM 16GB', 'Eletrônicos', 320.00, 35, 1),
    ('Livro: SQL Descomplicado', 'Livros', 65.00, 100, 1),
    ('Livro: Python para Iniciantes', 'Livros', 58.00, 80, 1),
    ('Mousepad Grande', 'Acessórios', 45.00, 60, 1),
    ('Cabo HDMI 2m', 'Acessórios', 25.00, 150, 1),
    ('Hub USB 4 Portas', 'Acessórios', 75.00, 45, 1);

-- Inserir dados de exemplo em Pedidos
INSERT INTO Pedidos (ClienteID, DataPedido, ValorTotal, Status)
VALUES
    (1, '2024-01-10', 3585.00, 'Entregue'),
    (1, '2024-02-15', 450.00, 'Entregue'),
    (2, '2024-01-20', 899.00, 'Entregue'),
    (3, '2024-02-05', 1750.00, 'Em Transporte'),
    (4, '2024-02-18', 4500.00, 'Entregue'),
    (5, '2024-03-01', 280.00, 'Cancelado'),
    (6, '2024-03-10', 775.00, 'Entregue'),
    (7, '2024-03-15', 123.00, 'Pendente'),
    (8, '2024-03-20', 3935.00, 'Em Processamento'),
    (9, '2024-03-25', 720.00, 'Entregue');

PRINT 'Banco de dados criado e populado com sucesso!';
        </code-block>
    </step>
    <step>
        <p>Verifique se os dados foram inseridos:</p>
        <code-block lang="sql">
SELECT COUNT(*) AS TotalClientes FROM Clientes;
SELECT COUNT(*) AS TotalProdutos FROM Produtos;
SELECT COUNT(*) AS TotalPedidos FROM Pedidos;
        </code-block>
    </step>
</procedure>

## Cláusula WHERE - Filtrando Dados

A cláusula WHERE permite filtrar registros baseado em condições:

### Operadores de Comparação

```sql
-- Igual (=)
SELECT * FROM Produtos WHERE Preco = 450.00;

-- Diferente (<> ou !=)
SELECT * FROM Produtos WHERE Categoria <> 'Livros';

-- Maior que (>)
SELECT Nome, Preco FROM Produtos WHERE Preco > 500.00;

-- Menor que (<)
SELECT Nome, Estoque FROM Produtos WHERE Estoque < 20;

-- Maior ou igual (>=)
SELECT * FROM Produtos WHERE Preco >= 100.00 AND Preco <= 500.00;

-- Menor ou igual (<=)
SELECT Nome, Estoque FROM Produtos WHERE Estoque <= 15;
```

### Operadores Lógicos

```sql
-- AND: Todas as condições devem ser verdadeiras
SELECT Nome, Preco, Categoria
FROM Produtos
WHERE Categoria = 'Eletrônicos' AND Preco > 300.00;

-- OR: Pelo menos uma condição deve ser verdadeira
SELECT Nome, Cidade, Estado
FROM Clientes
WHERE Estado = 'SP' OR Estado = 'RJ';

-- NOT: Negação
SELECT Nome, Email
FROM Clientes
WHERE NOT Ativo = 0;  -- Mesma coisa que: WHERE Ativo = 1

-- Combinando operadores (use parênteses para clareza)
SELECT Nome, Preco, Categoria
FROM Produtos
WHERE (Categoria = 'Eletrônicos' OR Categoria = 'Acessórios')
  AND Preco < 100.00
  AND Ativo = 1;
```

### Operador BETWEEN

Verifica se um valor está dentro de um intervalo (inclusivo):

```sql
-- Produtos com preço entre 100 e 500 reais
SELECT Nome, Preco
FROM Produtos
WHERE Preco BETWEEN 100.00 AND 500.00;

-- Equivalente a:
SELECT Nome, Preco
FROM Produtos
WHERE Preco >= 100.00 AND Preco <= 500.00;

-- Pedidos de fevereiro de 2024
SELECT PedidoID, ClienteID, DataPedido, ValorTotal
FROM Pedidos
WHERE DataPedido BETWEEN '2024-02-01' AND '2024-02-29';
```

### Operador IN

Verifica se um valor está em uma lista de valores:

```sql
-- Clientes de estados específicos
SELECT Nome, Cidade, Estado
FROM Clientes
WHERE Estado IN ('SP', 'RJ', 'MG');

-- Equivalente a:
SELECT Nome, Cidade, Estado
FROM Clientes
WHERE Estado = 'SP' OR Estado = 'RJ' OR Estado = 'MG';

-- Produtos de categorias específicas
SELECT Nome, Categoria, Preco
FROM Produtos
WHERE Categoria IN ('Eletrônicos', 'Acessórios')
ORDER BY Preco DESC;

-- NOT IN: Valores que NÃO estão na lista
SELECT Nome, Status
FROM Pedidos P
INNER JOIN Clientes C ON P.ClienteID = C.ClienteID
WHERE Status NOT IN ('Cancelado', 'Entregue');
```

### Operador LIKE - Busca por Padrões

Usado para buscar padrões em strings:

**Caracteres coringa:**
- `%`: Representa zero ou mais caracteres
- `_`: Representa exatamente um caractere
- `[]`: Representa qualquer caractere único dentro dos colchetes
- `[^]`: Representa qualquer caractere único NÃO nos colchetes

```sql
-- Nomes que começam com 'João'
SELECT Nome, Email FROM Clientes WHERE Nome LIKE 'João%';

-- Emails do Gmail
SELECT Nome, Email FROM Clientes WHERE Email LIKE '%@gmail.com';

-- Nomes que contêm 'Silva'
SELECT Nome FROM Clientes WHERE Nome LIKE '%Silva%';

-- Nomes que terminam com 'Santos'
SELECT Nome FROM Clientes WHERE Nome LIKE '%Santos';

-- Nomes com exatamente 4 caracteres (usando _)
SELECT Nome FROM Clientes WHERE Nome LIKE '____';

-- Produtos que começam com 'M' ou 'N'
SELECT Nome FROM Produtos WHERE Nome LIKE '[MN]%';

-- Produtos que NÃO começam com 'A' ou 'B'
SELECT Nome FROM Produtos WHERE Nome LIKE '[^AB]%';

-- Case-insensitive por padrão no SQL Server
-- Para case-sensitive, use COLLATE
SELECT Nome FROM Clientes
WHERE Nome LIKE '%silva%' COLLATE Latin1_General_CS_AS;
```

### Operador IS NULL / IS NOT NULL

Verifica valores nulos:

```sql
-- Clientes sem email cadastrado
SELECT Nome, Email
FROM Clientes
WHERE Email IS NULL;

-- Clientes com email cadastrado
SELECT Nome, Email
FROM Clientes
WHERE Email IS NOT NULL;

-- IMPORTANTE: Não use = NULL ou <> NULL
-- ❌ ERRADO:
SELECT * FROM Clientes WHERE Email = NULL;

-- ✅ CORRETO:
SELECT * FROM Clientes WHERE Email IS NULL;
```

## Cláusula ORDER BY - Ordenando Resultados

Ordena os resultados da consulta:

```sql
-- Ordem crescente (ASC é padrão)
SELECT Nome, Preco
FROM Produtos
ORDER BY Preco;  -- Mesmo que: ORDER BY Preco ASC

-- Ordem decrescente
SELECT Nome, Preco
FROM Produtos
ORDER BY Preco DESC;

-- Ordenar por múltiplas colunas
SELECT Nome, Cidade, Estado
FROM Clientes
ORDER BY Estado ASC, Cidade ASC, Nome ASC;

-- Ordenar por alias
SELECT
    Nome,
    Preco,
    (Preco * 1.10) AS PrecoComTaxa
FROM Produtos
ORDER BY PrecoComTaxa DESC;

-- Ordenar por número da coluna (evite, menos legível)
SELECT Nome, Preco, Estoque
FROM Produtos
ORDER BY 2 DESC;  -- Ordena pela 2ª coluna (Preco)

-- Ordenar com CASE para ordem personalizada
SELECT Nome, Status
FROM Pedidos P
INNER JOIN Clientes C ON P.ClienteID = C.ClienteID
ORDER BY
    CASE Status
        WHEN 'Pendente' THEN 1
        WHEN 'Em Processamento' THEN 2
        WHEN 'Em Transporte' THEN 3
        WHEN 'Entregue' THEN 4
        WHEN 'Cancelado' THEN 5
    END;
```

## DISTINCT - Removendo Duplicatas

Remove linhas duplicadas do resultado:

```sql
-- Listar todas as cidades (sem repetir)
SELECT DISTINCT Cidade
FROM Clientes
ORDER BY Cidade;

-- Listar todas as categorias de produtos
SELECT DISTINCT Categoria
FROM Produtos;

-- DISTINCT com múltiplas colunas
-- (remove apenas se TODAS as colunas forem iguais)
SELECT DISTINCT Cidade, Estado
FROM Clientes
ORDER BY Estado, Cidade;

-- Contar valores distintos
SELECT COUNT(DISTINCT Cidade) AS TotalCidades
FROM Clientes;
```

## TOP - Limitando Resultados

Retorna apenas as primeiras N linhas:

```sql
-- Primeiros 5 clientes
SELECT TOP 5 * FROM Clientes;

-- 5 produtos mais caros
SELECT TOP 5 Nome, Preco
FROM Produtos
ORDER BY Preco DESC;

-- TOP com percentual
SELECT TOP 20 PERCENT Nome, Preco
FROM Produtos
ORDER BY Preco DESC;

-- TOP com TIES (inclui empates)
SELECT TOP 3 WITH TIES Nome, Preco
FROM Produtos
ORDER BY Preco DESC;

-- TOP em DELETE ou UPDATE (cuidado!)
-- Deletar os 10 registros mais antigos
DELETE TOP (10) FROM Logs
WHERE DataLog < DATEADD(YEAR, -1, GETDATE());
```

## OFFSET e FETCH - Paginação

Método moderno para paginação de resultados (SQL Server 2012+):

```sql
-- Pular os primeiros 5 e pegar os próximos 5
SELECT Nome, Preco
FROM Produtos
ORDER BY Preco DESC
OFFSET 5 ROWS
FETCH NEXT 5 ROWS ONLY;

-- Paginação: Página 1 (primeiros 10)
SELECT Nome, Email
FROM Clientes
ORDER BY Nome
OFFSET 0 ROWS
FETCH NEXT 10 ROWS ONLY;

-- Paginação: Página 2 (registros 11-20)
SELECT Nome, Email
FROM Clientes
ORDER BY Nome
OFFSET 10 ROWS
FETCH NEXT 10 ROWS ONLY;

-- Paginação: Página 3 (registros 21-30)
SELECT Nome, Email
FROM Clientes
ORDER BY Nome
OFFSET 20 ROWS
FETCH NEXT 10 ROWS ONLY;

-- Função para calcular OFFSET baseado no número da página
DECLARE @PaginaAtual INT = 3;
DECLARE @ItensPorPagina INT = 10;
DECLARE @Offset INT = (@PaginaAtual - 1) * @ItensPorPagina;

SELECT Nome, Preco
FROM Produtos
ORDER BY Nome
OFFSET @Offset ROWS
FETCH NEXT @ItensPorPagina ROWS ONLY;
```

**OFFSET vs TOP:**
- OFFSET/FETCH: Padrão SQL, melhor para paginação
- TOP: Específico do SQL Server, mais simples para casos básicos

## Funções Básicas

### Funções de String

```sql
-- UPPER: Converter para maiúsculas
SELECT Nome, UPPER(Nome) AS NomeMaiusculo
FROM Clientes;

-- LOWER: Converter para minúsculas
SELECT Email, LOWER(Email) AS EmailMinusculo
FROM Clientes;

-- LEN: Comprimento da string
SELECT Nome, LEN(Nome) AS TamanhoNome
FROM Clientes;

-- LEFT / RIGHT: Pegar caracteres da esquerda/direita
SELECT Nome, LEFT(Nome, 5) AS Primeiros5Chars
FROM Clientes;

-- SUBSTRING: Extrair parte da string
SELECT Email, SUBSTRING(Email, 1, CHARINDEX('@', Email) - 1) AS Usuario
FROM Clientes
WHERE Email IS NOT NULL;

-- CONCAT: Concatenar strings
SELECT CONCAT(Nome, ' - ', Cidade, '/', Estado) AS ClienteCompleto
FROM Clientes;

-- TRIM, LTRIM, RTRIM: Remover espaços
SELECT TRIM('   Texto com espaços   ') AS TextoLimpo;

-- REPLACE: Substituir texto
SELECT Nome, REPLACE(Email, '@email.com', '@novoemail.com') AS NovoEmail
FROM Clientes;
```

### Funções Numéricas

```sql
-- ROUND: Arredondar
SELECT Nome, Preco, ROUND(Preco, 0) AS PrecoArredondado
FROM Produtos;

-- CEILING: Arredondar para cima
SELECT Preco, CEILING(Preco) AS PrecoTeto
FROM Produtos;

-- FLOOR: Arredondar para baixo
SELECT Preco, FLOOR(Preco) AS PrecoChao
FROM Produtos;

-- ABS: Valor absoluto
SELECT ABS(-100) AS ValorAbsoluto;

-- POWER: Potência
SELECT Preco, POWER(Preco, 2) AS PrecoAoQuadrado
FROM Produtos;

-- SQRT: Raiz quadrada
SELECT Preco, SQRT(Preco) AS RaizQuadrada
FROM Produtos
WHERE Preco > 0;
```

### Funções de Data

```sql
-- GETDATE: Data/hora atual
SELECT GETDATE() AS DataHoraAtual;

-- YEAR, MONTH, DAY: Extrair partes da data
SELECT
    DataCadastro,
    YEAR(DataCadastro) AS Ano,
    MONTH(DataCadastro) AS Mes,
    DAY(DataCadastro) AS Dia
FROM Clientes;

-- DATEADD: Adicionar/subtrair tempo
SELECT
    DataPedido,
    DATEADD(DAY, 7, DataPedido) AS DataEntregaPrevista,
    DATEADD(MONTH, -1, DataPedido) AS UmMesAntes
FROM Pedidos;

-- DATEDIFF: Diferença entre datas
SELECT
    Nome,
    DataCadastro,
    DATEDIFF(DAY, DataCadastro, GETDATE()) AS DiasDesdeCadastro
FROM Clientes;

-- FORMAT: Formatar data (SQL Server 2012+)
SELECT
    DataPedido,
    FORMAT(DataPedido, 'dd/MM/yyyy') AS DataFormatada,
    FORMAT(DataPedido, 'dd/MM/yyyy HH:mm:ss') AS DataHoraFormatada
FROM Pedidos;

-- EOMONTH: Último dia do mês
SELECT
    DataCadastro,
    EOMONTH(DataCadastro) AS UltimoDiaMes
FROM Clientes;
```

## Funções Agregadas

Funções que operam em múltiplas linhas e retornam um único valor:

```sql
-- COUNT: Contar registros
SELECT COUNT(*) AS TotalClientes FROM Clientes;
SELECT COUNT(Email) AS ClientesComEmail FROM Clientes;
SELECT COUNT(DISTINCT Cidade) AS TotalCidades FROM Clientes;

-- SUM: Somar valores
SELECT SUM(ValorTotal) AS TotalVendas FROM Pedidos;
SELECT SUM(Preco * Estoque) AS ValorTotalEstoque FROM Produtos;

-- AVG: Média
SELECT AVG(Preco) AS PrecoMedio FROM Produtos;
SELECT AVG(ValorTotal) AS TicketMedio FROM Pedidos;

-- MIN / MAX: Menor e maior valor
SELECT
    MIN(Preco) AS PrecoMinimo,
    MAX(Preco) AS PrecoMaximo
FROM Produtos;

-- Combinando várias agregações
SELECT
    COUNT(*) AS TotalProdutos,
    SUM(Estoque) AS EstoqueTotal,
    AVG(Preco) AS PrecoMedio,
    MIN(Preco) AS PrecoMinimo,
    MAX(Preco) AS PrecoMaximo
FROM Produtos
WHERE Ativo = 1;
```

## Expressões e Cálculos

```sql
-- Cálculos aritméticos
SELECT
    Nome,
    Preco,
    Preco * 1.10 AS PrecoComTaxa,
    Preco * 0.90 AS PrecoComDesconto,
    Preco * Estoque AS ValorEstoque
FROM Produtos;

-- CASE: Lógica condicional
SELECT
    Nome,
    Preco,
    CASE
        WHEN Preco < 100 THEN 'Barato'
        WHEN Preco BETWEEN 100 AND 500 THEN 'Médio'
        WHEN Preco > 500 THEN 'Caro'
        ELSE 'Sem Classificação'
    END AS Classificacao
FROM Produtos;

-- CASE com múltiplas condições
SELECT
    Nome,
    Estoque,
    CASE
        WHEN Estoque = 0 THEN 'Sem Estoque'
        WHEN Estoque < 10 THEN 'Estoque Baixo'
        WHEN Estoque < 30 THEN 'Estoque Normal'
        ELSE 'Estoque Alto'
    END AS StatusEstoque
FROM Produtos;

-- IIF: Condicional simples (SQL Server 2012+)
SELECT
    Nome,
    Ativo,
    IIF(Ativo = 1, 'Sim', 'Não') AS AtivoTexto
FROM Clientes;

-- COALESCE: Retorna o primeiro valor não-nulo
SELECT
    Nome,
    COALESCE(Email, 'Sem email') AS EmailDisplay
FROM Clientes;

-- NULLIF: Retorna NULL se os valores forem iguais
SELECT
    Nome,
    Estoque,
    NULLIF(Estoque, 0) AS EstoqueNaoZero
FROM Produtos;
```

## Boas Práticas em Consultas SELECT

### 1. Sempre Especifique as Colunas

```sql
-- ❌ Evite:
SELECT * FROM Clientes;

-- ✅ Prefira:
SELECT ClienteID, Nome, Email, Cidade
FROM Clientes;
```

**Por quê?**
- Performance: Transfere apenas dados necessários
- Manutenibilidade: Mudanças na tabela não quebram a aplicação
- Clareza: Fica explícito quais dados você está usando

### 2. Use Alias para Melhor Legibilidade

```sql
-- ✅ Bom:
SELECT
    C.Nome AS NomeCliente,
    C.Cidade,
    P.DataPedido,
    P.ValorTotal
FROM Clientes C
INNER JOIN Pedidos P ON C.ClienteID = P.ClienteID;
```

### 3. Formate Consultas de Forma Legível

```sql
-- ✅ Bem formatado:
SELECT
    C.Nome,
    C.Email,
    C.Cidade,
    P.ValorTotal
FROM
    Clientes C
    INNER JOIN Pedidos P ON C.ClienteID = P.ClienteID
WHERE
    C.Ativo = 1
    AND P.Status = 'Entregue'
    AND P.DataPedido >= '2024-01-01'
ORDER BY
    P.ValorTotal DESC;
```

### 4. Use Comentários

```sql
-- Buscar clientes ativos de SP com pedidos acima de 1000 reais
SELECT
    C.Nome,
    C.Email,
    SUM(P.ValorTotal) AS TotalCompras
FROM Clientes C
INNER JOIN Pedidos P ON C.ClienteID = P.ClienteID
WHERE
    C.Estado = 'SP'
    AND C.Ativo = 1
    AND P.Status = 'Entregue'
GROUP BY C.ClienteID, C.Nome, C.Email
HAVING SUM(P.ValorTotal) > 1000
ORDER BY TotalCompras DESC;
```

### 5. Evite SELECT * em Produção

### 6. Use EXISTS ao invés de IN para Subqueries Grandes

### 7. Teste Consultas em Ambiente de Desenvolvimento

## Exercícios Práticos

<procedure title="Exercícios para Praticar" id="exercicios-select">
    <step>
        <p><strong>Exercício 1:</strong> Liste todos os produtos da categoria 'Eletrônicos' ordenados por preço (do menor para o maior)</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Encontre todos os clientes de São Paulo que estão ativos</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Mostre os 3 produtos mais caros do estoque</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Liste todos os produtos com estoque abaixo de 20 unidades</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Calcule o valor total em estoque (Preço × Estoque) de cada produto</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Encontre todos os clientes cujo nome começa com 'M'</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Liste os pedidos feitos em março de 2024</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Mostre produtos com preço entre 50 e 500 reais</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Calcule quantos dias se passaram desde o cadastro de cada cliente</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Crie uma consulta com paginação mostrando 5 produtos por página (teste página 1, 2 e 3)</p>
    </step>
</procedure>

## Glossary

**SELECT**
: Comando SQL usado para consultar e recuperar dados de uma ou mais tabelas

**WHERE**
: Cláusula que filtra registros baseado em condições

**ORDER BY**
: Cláusula que ordena os resultados da consulta

**DISTINCT**
: Remove linhas duplicadas do resultado

**TOP**
: Limita o número de linhas retornadas

**OFFSET/FETCH**
: Método moderno para paginação de resultados

**LIKE**
: Operador usado para buscar padrões em strings

**BETWEEN**
: Operador que verifica se um valor está dentro de um intervalo

**IN**
: Operador que verifica se um valor está em uma lista

**IS NULL**
: Verifica se um valor é nulo

**Função Agregada**
: Função que opera em múltiplas linhas e retorna um único valor (COUNT, SUM, AVG, MIN, MAX)

**Alias**
: Nome alternativo dado a uma coluna ou tabela no resultado da consulta

**Wildcard (%)**
: Caractere coringa que representa zero ou mais caracteres em padrões LIKE

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/queries/select-transact-sql">Microsoft Docs - SELECT</a>
        <a href="https://www.w3schools.com/sql/sql_select.asp">W3Schools - SQL SELECT</a>
        <a href="https://sqlservertutorial.net/sql-server-basics/sql-server-select/">SQL Server Tutorial - SELECT</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique todos os exemplos desta aula
> - Complete os exercícios propostos
> - Experimente criar suas próprias consultas
> - Na próxima aula, aprenderemos INSERT, UPDATE e DELETE!
>
{style="tip"}
