# Aula 13: Stored Procedures

## Introdução

**Stored Procedures** (procedimentos armazenados) são blocos de código SQL reutilizáveis armazenados no banco de dados. Eles permitem encapsular lógica de negócio complexa, melhorar performance e segurança, e simplificar a manutenção de aplicações.

Nesta aula, você aprenderá:

- O que são stored procedures e suas vantagens
- Como criar, modificar e executar procedures
- Parâmetros de entrada e saída
- Controle de fluxo (IF, WHILE, TRY/CATCH)
- Variáveis e cursores
- Boas práticas e otimização

## O que são Stored Procedures?

Uma **stored procedure** é um conjunto de instruções SQL compiladas e armazenadas no servidor de banco de dados que pode ser executada repetidamente.

### Vantagens

1. **Performance**: Código pré-compilado e otimizado pelo SQL Server
2. **Segurança**: Usuários executam procedures sem acesso direto às tabelas
3. **Manutenção**: Lógica centralizada no banco de dados
4. **Reutilização**: Mesma procedure pode ser chamada de várias aplicações
5. **Redução de tráfego**: Menos dados enviados entre aplicação e servidor
6. **Transações**: Facilita controle de transações complexas

### Desvantagens

1. **Portabilidade**: Código específico do SQL Server
2. **Versionamento**: Mais difícil de versionar que código de aplicação
3. **Debugging**: Mais complexo que depurar código de aplicação
4. **Sobrecarga**: Pode ser excessivo para operações simples

## Criando Stored Procedures

### Sintaxe Básica

```sql
CREATE PROCEDURE nome_procedure
AS
BEGIN
    -- Instruções SQL
END;
GO
```

### Primeira Stored Procedure

Vamos criar procedures usando o banco LojaDB:

```sql
USE LojaDB;
GO

-- Procedure simples: Listar todos os clientes ativos
CREATE PROCEDURE sp_ListarClientesAtivos
AS
BEGIN
    SELECT
        ClienteID,
        Nome,
        Email,
        Cidade,
        Estado,
        DataCadastro
    FROM Clientes
    WHERE Ativo = 1
    ORDER BY Nome;
END;
GO

-- Executar a procedure
EXEC sp_ListarClientesAtivos;
-- ou
EXECUTE sp_ListarClientesAtivos;
```

### Convenção de Nomenclatura

**Importante**: Evite usar o prefixo `sp_` para procedures de usuário (é reservado para procedures do sistema). Use prefixos como `usp_`, `proc_`, ou simplesmente um nome descritivo.

```sql
-- ✅ Bom
CREATE PROCEDURE usp_ListarClientesAtivos
CREATE PROCEDURE proc_ListarClientesAtivos
CREATE PROCEDURE ListarClientesAtivos

-- ❌ Evite (sp_ é para procedures do sistema)
CREATE PROCEDURE sp_ListarClientesAtivos
```

## Parâmetros

### Parâmetros de Entrada

```sql
-- Procedure com parâmetro de entrada
CREATE PROCEDURE usp_BuscarClientePorEstado
    @Estado CHAR(2)
AS
BEGIN
    SELECT
        ClienteID,
        Nome,
        Email,
        Cidade,
        Estado
    FROM Clientes
    WHERE Estado = @Estado AND Ativo = 1
    ORDER BY Nome;
END;
GO

-- Executar com parâmetro
EXEC usp_BuscarClientePorEstado 'SP';
EXEC usp_BuscarClientePorEstado @Estado = 'RJ';
```

### Múltiplos Parâmetros

```sql
-- Procedure com múltiplos parâmetros
CREATE PROCEDURE usp_BuscarProdutos
    @Categoria NVARCHAR(50) = NULL,
    @PrecoMinimo DECIMAL(10,2) = 0,
    @PrecoMaximo DECIMAL(10,2) = 999999
AS
BEGIN
    SELECT
        ProdutoID,
        Nome,
        Categoria,
        Preco,
        Estoque
    FROM Produtos
    WHERE
        Ativo = 1
        AND (@Categoria IS NULL OR Categoria = @Categoria)
        AND Preco BETWEEN @PrecoMinimo AND @PrecoMaximo
    ORDER BY Preco;
END;
GO

-- Diferentes formas de executar
EXEC usp_BuscarProdutos;  -- Usa valores padrão
EXEC usp_BuscarProdutos 'Eletrônicos', 100, 1000;
EXEC usp_BuscarProdutos @Categoria = 'Livros';
EXEC usp_BuscarProdutos @PrecoMinimo = 500, @PrecoMaximo = 2000;
```

### Parâmetros de Saída (OUTPUT)

```sql
-- Procedure com parâmetro de saída
CREATE PROCEDURE usp_ContarClientesPorEstado
    @Estado CHAR(2),
    @TotalClientes INT OUTPUT
AS
BEGIN
    SELECT @TotalClientes = COUNT(*)
    FROM Clientes
    WHERE Estado = @Estado AND Ativo = 1;
END;
GO

-- Executar com OUTPUT
DECLARE @Total INT;
EXEC usp_ContarClientesPorEstado 'SP', @Total OUTPUT;
SELECT @Total AS TotalClientesSP;
```

### Parâmetros com Valores Padrão

```sql
CREATE PROCEDURE usp_ListarPedidos
    @ClienteID INT = NULL,
    @Status NVARCHAR(20) = 'Todos',
    @DataInicio DATE = NULL,
    @DataFim DATE = NULL
AS
BEGIN
    SELECT
        P.PedidoID,
        P.DataPedido,
        P.ValorTotal,
        P.Status,
        C.Nome AS NomeCliente
    FROM Pedidos P
    INNER JOIN Clientes C ON P.ClienteID = C.ClienteID
    WHERE
        (@ClienteID IS NULL OR P.ClienteID = @ClienteID)
        AND (@Status = 'Todos' OR P.Status = @Status)
        AND (@DataInicio IS NULL OR P.DataPedido >= @DataInicio)
        AND (@DataFim IS NULL OR P.DataPedido <= @DataFim)
    ORDER BY P.DataPedido DESC;
END;
GO

-- Executar de várias formas
EXEC usp_ListarPedidos;
EXEC usp_ListarPedidos @Status = 'Entregue';
EXEC usp_ListarPedidos @ClienteID = 1, @Status = 'Todos';
```

## Variáveis

```sql
CREATE PROCEDURE usp_CalcularEstatisticasVendas
AS
BEGIN
    -- Declarar variáveis
    DECLARE @TotalPedidos INT;
    DECLARE @ValorTotal DECIMAL(10,2);
    DECLARE @TicketMedio DECIMAL(10,2);
    DECLARE @MaiorPedido DECIMAL(10,2);
    DECLARE @MenorPedido DECIMAL(10,2);

    -- Calcular estatísticas
    SELECT
        @TotalPedidos = COUNT(*),
        @ValorTotal = SUM(ValorTotal),
        @TicketMedio = AVG(ValorTotal),
        @MaiorPedido = MAX(ValorTotal),
        @MenorPedido = MIN(ValorTotal)
    FROM Pedidos
    WHERE Status <> 'Cancelado';

    -- Exibir resultados
    SELECT
        @TotalPedidos AS TotalPedidos,
        @ValorTotal AS ValorTotal,
        @TicketMedio AS TicketMedio,
        @MaiorPedido AS MaiorPedido,
        @MenorPedido AS MenorPedido;
END;
GO

EXEC usp_CalcularEstatisticasVendas;
```

## Controle de Fluxo

### IF...ELSE

```sql
CREATE PROCEDURE usp_AtualizarStatusPedido
    @PedidoID INT,
    @NovoStatus NVARCHAR(20)
AS
BEGIN
    DECLARE @StatusAtual NVARCHAR(20);

    -- Obter status atual
    SELECT @StatusAtual = Status
    FROM Pedidos
    WHERE PedidoID = @PedidoID;

    -- Validar mudança de status
    IF @StatusAtual IS NULL
    BEGIN
        PRINT 'Pedido não encontrado!';
        RETURN;
    END

    IF @StatusAtual = 'Cancelado'
    BEGIN
        PRINT 'Não é possível alterar status de pedido cancelado!';
        RETURN;
    END

    IF @StatusAtual = 'Entregue' AND @NovoStatus <> 'Entregue'
    BEGIN
        PRINT 'Não é possível alterar status de pedido já entregue!';
        RETURN;
    END

    -- Atualizar status
    UPDATE Pedidos
    SET Status = @NovoStatus
    WHERE PedidoID = @PedidoID;

    PRINT 'Status atualizado com sucesso!';
END;
GO

EXEC usp_AtualizarStatusPedido 1, 'Em Transporte';
```

### WHILE (Loops)

```sql
CREATE PROCEDURE usp_GerarRelatorioMensal
    @Ano INT,
    @Mes INT
AS
BEGIN
    DECLARE @DataInicio DATE;
    DECLARE @DataFim DATE;
    DECLARE @Contador INT = 1;

    -- Loop pelos dias do mês
    SET @DataInicio = DATEFROMPARTS(@Ano, @Mes, 1);
    SET @DataFim = EOMONTH(@DataInicio);

    CREATE TABLE #RelatorioTemp (
        Data DATE,
        TotalPedidos INT,
        ValorTotal DECIMAL(10,2)
    );

    WHILE @DataInicio <= @DataFim
    BEGIN
        INSERT INTO #RelatorioTemp (Data, TotalPedidos, ValorTotal)
        SELECT
            @DataInicio,
            COUNT(*),
            ISNULL(SUM(ValorTotal), 0)
        FROM Pedidos
        WHERE CAST(DataPedido AS DATE) = @DataInicio
            AND Status <> 'Cancelado';

        SET @DataInicio = DATEADD(DAY, 1, @DataInicio);
    END

    SELECT * FROM #RelatorioTemp
    ORDER BY Data;

    DROP TABLE #RelatorioTemp;
END;
GO

EXEC usp_GerarRelatorioMensal 2024, 3;
```

### CASE

```sql
CREATE PROCEDURE usp_ClassificarClientes
AS
BEGIN
    SELECT
        C.ClienteID,
        C.Nome,
        C.Email,
        ISNULL(SUM(P.ValorTotal), 0) AS TotalGasto,
        CASE
            WHEN ISNULL(SUM(P.ValorTotal), 0) = 0 THEN 'Sem Compras'
            WHEN ISNULL(SUM(P.ValorTotal), 0) < 1000 THEN 'Bronze'
            WHEN ISNULL(SUM(P.ValorTotal), 0) < 5000 THEN 'Prata'
            WHEN ISNULL(SUM(P.ValorTotal), 0) < 10000 THEN 'Ouro'
            ELSE 'Platina'
        END AS Categoria
    FROM Clientes C
    LEFT JOIN Pedidos P ON C.ClienteID = P.ClienteID
        AND P.Status = 'Entregue'
    WHERE C.Ativo = 1
    GROUP BY C.ClienteID, C.Nome, C.Email
    ORDER BY TotalGasto DESC;
END;
GO

EXEC usp_ClassificarClientes;
```

## Tratamento de Erros (TRY...CATCH)

```sql
CREATE PROCEDURE usp_InserirCliente
    @Nome NVARCHAR(100),
    @Email VARCHAR(100),
    @Cidade NVARCHAR(50),
    @Estado CHAR(2)
AS
BEGIN
    BEGIN TRY
        -- Validações
        IF @Nome IS NULL OR LEN(@Nome) < 3
        BEGIN
            RAISERROR('Nome deve ter pelo menos 3 caracteres', 16, 1);
            RETURN;
        END

        IF @Email IS NULL OR @Email NOT LIKE '%@%.%'
        BEGIN
            RAISERROR('Email inválido', 16, 1);
            RETURN;
        END

        -- Verificar se email já existe
        IF EXISTS (SELECT 1 FROM Clientes WHERE Email = @Email)
        BEGIN
            RAISERROR('Email já cadastrado', 16, 1);
            RETURN;
        END

        -- Inserir cliente
        INSERT INTO Clientes (Nome, Email, Cidade, Estado, Ativo)
        VALUES (@Nome, @Email, @Cidade, @Estado, 1);

        PRINT 'Cliente cadastrado com sucesso!';
        SELECT SCOPE_IDENTITY() AS NovoClienteID;

    END TRY
    BEGIN CATCH
        -- Capturar e exibir erro
        DECLARE @ErrorMessage NVARCHAR(4000) = ERROR_MESSAGE();
        DECLARE @ErrorSeverity INT = ERROR_SEVERITY();
        DECLARE @ErrorState INT = ERROR_STATE();

        PRINT 'Erro ao cadastrar cliente:';
        PRINT @ErrorMessage;

        -- Repassar erro
        RAISERROR(@ErrorMessage, @ErrorSeverity, @ErrorState);
    END CATCH
END;
GO

-- Testar
EXEC usp_InserirCliente 'Ana Silva', 'ana@exemplo.com', 'São Paulo', 'SP';
EXEC usp_InserirCliente 'A', 'invalido', 'São Paulo', 'SP';  -- Deve gerar erro
```

## Transações em Stored Procedures

```sql
CREATE PROCEDURE usp_TransferirPedido
    @PedidoID INT,
    @NovoClienteID INT
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        DECLARE @ClienteAntigoID INT;

        -- Verificar se pedido existe
        SELECT @ClienteAntigoID = ClienteID
        FROM Pedidos
        WHERE PedidoID = @PedidoID;

        IF @ClienteAntigoID IS NULL
        BEGIN
            RAISERROR('Pedido não encontrado', 16, 1);
        END

        -- Verificar se novo cliente existe
        IF NOT EXISTS (SELECT 1 FROM Clientes WHERE ClienteID = @NovoClienteID)
        BEGIN
            RAISERROR('Cliente não encontrado', 16, 1);
        END

        -- Atualizar pedido
        UPDATE Pedidos
        SET ClienteID = @NovoClienteID
        WHERE PedidoID = @PedidoID;

        -- Log da transferência (exemplo)
        PRINT 'Pedido ' + CAST(@PedidoID AS VARCHAR) + ' transferido de cliente ' +
              CAST(@ClienteAntigoID AS VARCHAR) + ' para ' + CAST(@NovoClienteID AS VARCHAR);

        COMMIT TRANSACTION;
        PRINT 'Transferência realizada com sucesso!';

    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        DECLARE @ErrorMessage NVARCHAR(4000) = ERROR_MESSAGE();
        RAISERROR(@ErrorMessage, 16, 1);
    END CATCH
END;
GO

EXEC usp_TransferirPedido 1, 2;
```

## RETURN e Valores de Retorno

```sql
CREATE PROCEDURE usp_ValidarEstoque
    @ProdutoID INT,
    @QuantidadeDesejada INT
AS
BEGIN
    DECLARE @EstoqueAtual INT;

    SELECT @EstoqueAtual = Estoque
    FROM Produtos
    WHERE ProdutoID = @ProdutoID;

    IF @EstoqueAtual IS NULL
        RETURN -1;  -- Produto não encontrado

    IF @EstoqueAtual < @QuantidadeDesejada
        RETURN 0;   -- Estoque insuficiente

    RETURN 1;       -- Estoque suficiente
END;
GO

-- Usar valor de retorno
DECLARE @Resultado INT;
EXEC @Resultado = usp_ValidarEstoque 1, 10;

IF @Resultado = -1
    PRINT 'Produto não encontrado';
ELSE IF @Resultado = 0
    PRINT 'Estoque insuficiente';
ELSE IF @Resultado = 1
    PRINT 'Estoque disponível';
```

## Modificando Stored Procedures

```sql
-- Alterar procedure existente
ALTER PROCEDURE usp_ListarClientesAtivos
AS
BEGIN
    SELECT
        ClienteID,
        Nome,
        Email,
        Cidade,
        Estado,
        DataCadastro,
        DATEDIFF(DAY, DataCadastro, GETDATE()) AS DiasComoCliente
    FROM Clientes
    WHERE Ativo = 1
    ORDER BY Nome;
END;
GO

-- CREATE OR ALTER (SQL Server 2016+)
CREATE OR ALTER PROCEDURE usp_BuscarProduto
    @ProdutoID INT
AS
BEGIN
    SELECT * FROM Produtos WHERE ProdutoID = @ProdutoID;
END;
GO
```

## Removendo Stored Procedures

```sql
-- Remover procedure
DROP PROCEDURE IF EXISTS usp_BuscarProduto;
GO

-- Remover múltiplas procedures
DROP PROCEDURE IF EXISTS usp_Proc1, usp_Proc2, usp_Proc3;
GO
```

## Metadados de Procedures

```sql
-- Listar todas as procedures
SELECT
    name AS NomeProcedure,
    create_date AS DataCriacao,
    modify_date AS DataModificacao
FROM sys.procedures
ORDER BY name;

-- Ver definição de uma procedure
SELECT OBJECT_DEFINITION(OBJECT_ID('usp_ListarClientesAtivos'));

-- Ou use sp_helptext
EXEC sp_helptext 'usp_ListarClientesAtivos';

-- Informações detalhadas
EXEC sp_help 'usp_ListarClientesAtivos';

-- Parâmetros de uma procedure
SELECT
    p.name AS ParameterName,
    TYPE_NAME(p.user_type_id) AS DataType,
    p.max_length,
    p.is_output
FROM sys.parameters p
WHERE p.object_id = OBJECT_ID('usp_BuscarProdutos')
ORDER BY p.parameter_id;
```

## Procedures com Tabelas Temporárias

```sql
CREATE PROCEDURE usp_AnalisarVendasDetalhada
    @DataInicio DATE,
    @DataFim DATE
AS
BEGIN
    -- Tabela temporária local (visível apenas nesta sessão)
    CREATE TABLE #VendasTemp (
        ClienteID INT,
        NomeCliente NVARCHAR(100),
        TotalPedidos INT,
        ValorTotal DECIMAL(10,2)
    );

    -- Popular tabela temporária
    INSERT INTO #VendasTemp
    SELECT
        C.ClienteID,
        C.Nome,
        COUNT(P.PedidoID),
        SUM(P.ValorTotal)
    FROM Clientes C
    INNER JOIN Pedidos P ON C.ClienteID = P.ClienteID
    WHERE P.DataPedido BETWEEN @DataInicio AND @DataFim
        AND P.Status = 'Entregue'
    GROUP BY C.ClienteID, C.Nome;

    -- Análise 1: Top 10 clientes
    SELECT TOP 10 *
    FROM #VendasTemp
    ORDER BY ValorTotal DESC;

    -- Análise 2: Estatísticas gerais
    SELECT
        COUNT(*) AS TotalClientes,
        SUM(TotalPedidos) AS TotalPedidos,
        SUM(ValorTotal) AS ValorTotalVendas,
        AVG(ValorTotal) AS MediaPorCliente
    FROM #VendasTemp;

    -- Limpar
    DROP TABLE #VendasTemp;
END;
GO

EXEC usp_AnalisarVendasDetalhada '2024-01-01', '2024-12-31';
```

## Procedures Dinâmicas (SQL Dinâmico)

```sql
CREATE PROCEDURE usp_ConsultaDinamica
    @NomeTabela NVARCHAR(128),
    @Coluna NVARCHAR(128) = '*',
    @Condicao NVARCHAR(MAX) = NULL
AS
BEGIN
    -- ATENÇÃO: SQL dinâmico pode ser vulnerável a SQL Injection
    -- Sempre valide e sanitize inputs!

    DECLARE @SQL NVARCHAR(MAX);

    -- Validar nome da tabela
    IF OBJECT_ID(@NomeTabela) IS NULL
    BEGIN
        RAISERROR('Tabela não existe', 16, 1);
        RETURN;
    END

    -- Construir query
    SET @SQL = 'SELECT ' + @Coluna + ' FROM ' + QUOTENAME(@NomeTabela);

    IF @Condicao IS NOT NULL
        SET @SQL = @SQL + ' WHERE ' + @Condicao;

    -- Executar
    PRINT @SQL;  -- Debug
    EXEC sp_executesql @SQL;
END;
GO

-- Executar
EXEC usp_ConsultaDinamica 'Clientes', 'Nome, Email', 'Estado = ''SP''';
```

## Boas Práticas

### 1. Use SET NOCOUNT ON

```sql
CREATE PROCEDURE usp_ExemploBP
AS
BEGIN
    SET NOCOUNT ON;  -- Evita mensagens "X rows affected"

    -- Seu código aqui
    SELECT * FROM Clientes;
END;
GO
```

### 2. Sempre Use BEGIN...END

```sql
-- ✅ Bom
CREATE PROCEDURE usp_Exemplo
AS
BEGIN
    SELECT * FROM Clientes;
END;

-- ❌ Evite (funciona, mas menos legível)
CREATE PROCEDURE usp_Exemplo
AS
SELECT * FROM Clientes;
```

### 3. Nomeie Parâmetros Claramente

```sql
-- ✅ Bom
CREATE PROCEDURE usp_BuscarCliente
    @ClienteID INT,
    @IncluirInativos BIT = 0
AS BEGIN...

-- ❌ Evite nomes vagos
CREATE PROCEDURE usp_BuscarCliente
    @ID INT,
    @Flag BIT = 0
AS BEGIN...
```

### 4. Documente Procedures

```sql
/*
Procedure: usp_ProcessarPedido
Descrição: Processa um novo pedido, validando estoque e atualizando tabelas
Autor: Seu Nome
Data: 2024-03-19
Parâmetros:
    @ClienteID: ID do cliente
    @ValorTotal: Valor total do pedido
Retorna: PedidoID do pedido criado
*/
CREATE PROCEDURE usp_ProcessarPedido
    @ClienteID INT,
    @ValorTotal DECIMAL(10,2)
AS
BEGIN
    -- Implementação
END;
GO
```

### 5. Use TRY...CATCH

```sql
CREATE PROCEDURE usp_OperacaoCritica
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        -- Operações

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        THROW;  -- Repassa o erro
    END CATCH
END;
GO
```

### 6. Evite SELECT *

```sql
-- ✅ Bom
SELECT ClienteID, Nome, Email FROM Clientes;

-- ❌ Evite
SELECT * FROM Clientes;
```

### 7. Use EXISTS ao invés de COUNT

```sql
-- ✅ Melhor performance
IF EXISTS (SELECT 1 FROM Clientes WHERE Email = @Email)
BEGIN...

-- ❌ Menos eficiente
IF (SELECT COUNT(*) FROM Clientes WHERE Email = @Email) > 0
BEGIN...
```

## Exercícios Práticos

<procedure title="Exercícios de Stored Procedures" id="exercicios-procedures">
    <step>
        <p><strong>Exercício 1:</strong> Crie uma procedure que receba um ClienteID e retorne todos os pedidos desse cliente com valor total</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Crie uma procedure para cadastrar um novo produto com validação de campos obrigatórios</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Crie uma procedure que calcule o faturamento total de um período (data início e fim) por categoria de produto</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Crie uma procedure com parâmetro OUTPUT que retorne quantos clientes estão inativos</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Crie uma procedure que atualize o estoque de um produto, com validação de quantidade mínima</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Crie uma procedure com TRY/CATCH que insira um pedido e atualize o estoque dos produtos em uma transação</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Crie uma procedure que use WHILE para processar todos os pedidos pendentes em lote</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Crie uma procedure que gere um relatório de clientes por faixa de valor gasto (0-100, 100-500, 500+)</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Crie uma procedure que receba um estado e retorne estatísticas de vendas daquele estado</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Crie uma procedure com tabela temporária que analise produtos com estoque baixo e sugira quantidade de reposição</p>
    </step>
</procedure>

## Glossary

**Stored Procedure**
: Conjunto de instruções SQL compiladas e armazenadas no servidor de banco de dados

**Parâmetro de Entrada**
: Valor passado para a procedure quando ela é executada

**Parâmetro de Saída (OUTPUT)**
: Valor retornado pela procedure através de um parâmetro

**RETURN**
: Comando que encerra a execução da procedure e opcionalmente retorna um valor inteiro

**TRY...CATCH**
: Estrutura para tratamento de erros em SQL Server

**@@TRANCOUNT**
: Variável global que indica o número de transações ativas

**SCOPE_IDENTITY()**
: Retorna o último valor de identidade inserido no escopo atual

**SET NOCOUNT ON**
: Suprime mensagens de "X rows affected"

**SQL Dinâmico**
: SQL construído como string e executado em tempo de execução

**sp_executesql**
: Procedure do sistema para executar SQL dinâmico

**RAISERROR**
: Gera uma mensagem de erro customizada

**THROW**
: SQL Server 2012+, alternativa moderna ao RAISERROR

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/relational-databases/stored-procedures/stored-procedures-database-engine">Microsoft Docs - Stored Procedures</a>
        <a href="https://www.sqlservertutorial.net/sql-server-stored-procedures/">SQL Server Tutorial - Stored Procedures</a>
        <a href="https://www.sqlshack.com/learn-sql-stored-procedures/">SQL Shack - Learn Stored Procedures</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique criando procedures para operações comuns
> - Experimente diferentes tipos de parâmetros
> - Implemente tratamento de erros robusto
> - Na próxima aula, aprenderemos sobre Triggers e Transações!
>
{style="tip"}
