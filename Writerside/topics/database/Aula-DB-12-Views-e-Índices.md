# Aula 12: Views e Índices

## Introdução

**Views** (visões) e **índices** são recursos fundamentais para otimização e organização de bancos de dados. Views permitem criar "tabelas virtuais" baseadas em consultas, enquanto índices aceleram significativamente a recuperação de dados. Nesta aula, você aprenderá a criar, gerenciar e otimizar esses componentes essenciais.

Nesta aula, você aprenderá:
- O que são views e quando utilizá-las
- Como criar, alterar e remover views
- Tipos de índices no SQL Server
- Como criar e gerenciar índices
- Estratégias de indexação
- Performance e monitoramento

## O que são Views?

Uma **view** é uma consulta SQL armazenada que funciona como uma tabela virtual. Ela não armazena dados fisicamente (com exceção de views materializadas), mas apresenta dados de uma ou mais tabelas base.

### Vantagens das Views

1. **Segurança**: Oculta colunas sensíveis e restringe acesso a dados
2. **Simplificação**: Encapsula consultas complexas
3. **Reutilização**: Centraliza lógica de negócio
4. **Abstração**: Protege aplicações de mudanças na estrutura das tabelas
5. **Performance**: Pode simplificar consultas complexas (quando bem projetadas)

### Desvantagens das Views

1. **Performance**: Views mal projetadas podem degradar performance
2. **Limitações DML**: Nem todas as views permitem INSERT, UPDATE, DELETE
3. **Dependências**: Mudanças nas tabelas base podem quebrar views

## Criando Views

### Sintaxe Básica

```sql
CREATE VIEW nome_view AS
SELECT colunas
FROM tabelas
WHERE condições;
```

### Exemplos Práticos

Vamos usar o banco de dados LojaDB criado nas aulas anteriores:

```sql
USE LojaDB;
GO

-- View simples: Clientes ativos
CREATE VIEW vw_ClientesAtivos AS
SELECT
    ClienteID,
    Nome,
    Email,
    Cidade,
    Estado,
    DataCadastro
FROM Clientes
WHERE Ativo = 1;
GO

-- Usar a view
SELECT * FROM vw_ClientesAtivos;
SELECT * FROM vw_ClientesAtivos WHERE Estado = 'SP';
```

### View com JOIN

```sql
-- View: Pedidos com informações do cliente
CREATE VIEW vw_PedidosCompletos AS
SELECT
    P.PedidoID,
    P.DataPedido,
    P.ValorTotal,
    P.Status,
    C.ClienteID,
    C.Nome AS NomeCliente,
    C.Email AS EmailCliente,
    C.Cidade,
    C.Estado
FROM Pedidos P
INNER JOIN Clientes C ON P.ClienteID = C.ClienteID;
GO

-- Consultar a view
SELECT *
FROM vw_PedidosCompletos
WHERE Estado = 'SP' AND Status = 'Entregue';

-- Filtrar e ordenar
SELECT
    NomeCliente,
    COUNT(*) AS TotalPedidos,
    SUM(ValorTotal) AS TotalGasto
FROM vw_PedidosCompletos
WHERE Status = 'Entregue'
GROUP BY ClienteID, NomeCliente
ORDER BY TotalGasto DESC;
```

### View com Agregações

```sql
-- View: Resumo de vendas por cliente
CREATE VIEW vw_ResumoVendasCliente AS
SELECT
    C.ClienteID,
    C.Nome,
    C.Email,
    C.Cidade,
    C.Estado,
    COUNT(P.PedidoID) AS TotalPedidos,
    ISNULL(SUM(P.ValorTotal), 0) AS TotalGasto,
    ISNULL(AVG(P.ValorTotal), 0) AS TicketMedio,
    MAX(P.DataPedido) AS UltimoPedido
FROM Clientes C
LEFT JOIN Pedidos P ON C.ClienteID = P.ClienteID
    AND P.Status <> 'Cancelado'
WHERE C.Ativo = 1
GROUP BY C.ClienteID, C.Nome, C.Email, C.Cidade, C.Estado;
GO

-- Usar a view agregada
SELECT * FROM vw_ResumoVendasCliente
WHERE TotalGasto > 1000
ORDER BY TotalGasto DESC;
```

### View com Cálculos

```sql
-- View: Produtos com informações calculadas
CREATE VIEW vw_ProdutosDetalhados AS
SELECT
    ProdutoID,
    Nome,
    Categoria,
    Preco,
    Estoque,
    Ativo,
    (Preco * Estoque) AS ValorEstoque,
    CASE
        WHEN Estoque = 0 THEN 'Sem Estoque'
        WHEN Estoque < 10 THEN 'Estoque Baixo'
        WHEN Estoque < 30 THEN 'Estoque Normal'
        ELSE 'Estoque Alto'
    END AS StatusEstoque,
    CASE
        WHEN Preco < 100 THEN 'Econômico'
        WHEN Preco BETWEEN 100 AND 500 THEN 'Intermediário'
        ELSE 'Premium'
    END AS FaixaPreco
FROM Produtos
WHERE Ativo = 1;
GO

SELECT * FROM vw_ProdutosDetalhados
WHERE StatusEstoque = 'Estoque Baixo';
```

## Modificando Views

### Alterando uma View com ALTER VIEW

```sql
-- Modificar view existente
ALTER VIEW vw_ClientesAtivos AS
SELECT
    ClienteID,
    Nome,
    Email,
    Cidade,
    Estado,
    DataCadastro,
    DATEDIFF(DAY, DataCadastro, GETDATE()) AS DiasComoCliente
FROM Clientes
WHERE Ativo = 1;
GO
```

### CREATE OR ALTER (SQL Server 2016+)

```sql
-- Criar ou alterar view em um único comando
CREATE OR ALTER VIEW vw_ProdutosPremium AS
SELECT
    ProdutoID,
    Nome,
    Categoria,
    Preco,
    Estoque
FROM Produtos
WHERE Preco > 500 AND Ativo = 1;
GO
```

## Removendo Views

```sql
-- Remover uma view
DROP VIEW IF EXISTS vw_ProdutosPremium;
GO

-- Remover múltiplas views
DROP VIEW IF EXISTS vw_View1, vw_View2, vw_View3;
GO
```

## Views Atualizáveis

Algumas views permitem operações INSERT, UPDATE e DELETE, desde que atendam a certos requisitos:

### Requisitos para Views Atualizáveis

1. View baseada em **uma única tabela**
2. Inclui todas as colunas **NOT NULL** da tabela base
3. Não usa **DISTINCT, GROUP BY, HAVING, UNION**
4. Não usa **funções agregadas**
5. Não usa **TOP, OFFSET/FETCH**

```sql
-- View atualizável simples
CREATE VIEW vw_ClientesEdicao AS
SELECT
    ClienteID,
    Nome,
    Email,
    Cidade,
    Estado,
    Ativo
FROM Clientes;
GO

-- INSERT através da view
INSERT INTO vw_ClientesEdicao (Nome, Email, Cidade, Estado, Ativo)
VALUES ('Novo Cliente', 'novo@email.com', 'São Paulo', 'SP', 1);

-- UPDATE através da view
UPDATE vw_ClientesEdicao
SET Email = 'atualizado@email.com'
WHERE Nome = 'Novo Cliente';

-- DELETE através da view
DELETE FROM vw_ClientesEdicao
WHERE Nome = 'Novo Cliente';
```

### WITH CHECK OPTION

Garante que modificações atendam às condições da view:

```sql
CREATE VIEW vw_ClientesSP AS
SELECT ClienteID, Nome, Email, Cidade, Estado
FROM Clientes
WHERE Estado = 'SP'
WITH CHECK OPTION;
GO

-- ✅ Isso funciona (Estado = 'SP')
INSERT INTO vw_ClientesSP (Nome, Email, Cidade, Estado)
VALUES ('Cliente SP', 'cliente@sp.com', 'São Paulo', 'SP');

-- ❌ Isso falha (Estado <> 'SP')
INSERT INTO vw_ClientesSP (Nome, Email, Cidade, Estado)
VALUES ('Cliente RJ', 'cliente@rj.com', 'Rio', 'RJ');
-- Erro: tentativa de inserir registro que não atende WHERE da view
```

## Metadados de Views

```sql
-- Listar todas as views do banco
SELECT
    name AS NomeView,
    create_date AS DataCriacao,
    modify_date AS DataModificacao
FROM sys.views
ORDER BY name;

-- Ver definição de uma view
SELECT OBJECT_DEFINITION(OBJECT_ID('vw_ClientesAtivos')) AS DefinicaoView;

-- Ou use sp_helptext
EXEC sp_helptext 'vw_ClientesAtivos';

-- Informações sobre colunas da view
EXEC sp_help 'vw_PedidosCompletos';

-- Ver dependências da view
EXEC sp_depends 'vw_PedidosCompletos';
```

## Índices

**Índices** são estruturas de dados que aceleram a recuperação de informações de uma tabela, funcionando como o índice de um livro.

### Tipos de Índices no SQL Server

1. **Clustered Index** (Índice Agrupado)
   - Determina a ordem física dos dados na tabela
   - Apenas **um** por tabela
   - Geralmente criado na chave primária

2. **Non-Clustered Index** (Índice Não-Agrupado)
   - Estrutura separada que aponta para os dados
   - Até **999** por tabela (SQL Server 2016+)
   - Ideal para colunas frequentemente consultadas

3. **Unique Index** (Índice Único)
   - Garante valores únicos
   - Pode ser clustered ou non-clustered

4. **Filtered Index** (Índice Filtrado)
   - Índice em um subconjunto dos dados
   - Economiza espaço e melhora performance

5. **Columnstore Index** (Índice Colunar)
   - Otimizado para data warehousing e analytics
   - Armazena dados por coluna

### Quando Criar Índices

✅ **Criar índices quando:**
- Coluna usada frequentemente em WHERE, JOIN, ORDER BY
- Coluna com alta cardinalidade (muitos valores distintos)
- Consultas retornam pequena % dos dados
- Tabela tem muitas leituras e poucas escritas

❌ **Evitar índices quando:**
- Tabela muito pequena (< 1000 linhas)
- Coluna raramente consultada
- Coluna com baixa cardinalidade (ex: booleanos)
- Tabela com muitas escritas (INSERT/UPDATE/DELETE)

## Criando Índices

### Índice Non-Clustered Simples

```sql
-- Criar índice na coluna Email de Clientes
CREATE INDEX IX_Clientes_Email
ON Clientes(Email);

-- Índice composto (múltiplas colunas)
CREATE INDEX IX_Clientes_Cidade_Estado
ON Clientes(Cidade, Estado);

-- Testar performance
-- Sem índice (Scan)
SELECT * FROM Clientes WHERE Email = 'joao@email.com';

-- Com índice (Seek) - muito mais rápido!
```

### Índice Único

```sql
-- Garantir que emails sejam únicos
CREATE UNIQUE INDEX IX_Clientes_Email_Unique
ON Clientes(Email)
WHERE Email IS NOT NULL;  -- Permite múltiplos NULLs
```

### Índice Filtrado

```sql
-- Índice apenas para clientes ativos
CREATE INDEX IX_Clientes_Ativos
ON Clientes(Nome, Cidade)
WHERE Ativo = 1;

-- Índice para produtos em estoque
CREATE INDEX IX_Produtos_EmEstoque
ON Produtos(Nome, Categoria, Preco)
WHERE Estoque > 0 AND Ativo = 1;
```

### Índice com Colunas Incluídas

Colunas incluídas (INCLUDE) não fazem parte da chave, mas são armazenadas no índice para evitar lookups:

```sql
-- Índice com colunas incluídas
CREATE INDEX IX_Pedidos_ClienteID_Includes
ON Pedidos(ClienteID)
INCLUDE (DataPedido, ValorTotal, Status);

-- Consulta coberta pelo índice (index covering)
SELECT ClienteID, DataPedido, ValorTotal, Status
FROM Pedidos
WHERE ClienteID = 5;
-- Não precisa acessar a tabela, tudo está no índice!
```

### Índice Descendente

```sql
-- Índice para ordenação descendente
CREATE INDEX IX_Pedidos_DataDesc
ON Pedidos(DataPedido DESC);

-- Útil para consultas como:
SELECT TOP 10 * FROM Pedidos
ORDER BY DataPedido DESC;
```

## Gerenciando Índices

### Listar Índices

```sql
-- Índices de uma tabela específica
SELECT
    i.name AS NomeIndice,
    i.type_desc AS TipoIndice,
    i.is_unique AS Unico,
    i.is_primary_key AS ChavePrimaria,
    COL_NAME(ic.object_id, ic.column_id) AS Coluna
FROM sys.indexes i
INNER JOIN sys.index_columns ic ON i.object_id = ic.object_id
    AND i.index_id = ic.index_id
WHERE i.object_id = OBJECT_ID('Clientes')
ORDER BY i.name, ic.key_ordinal;

-- Índices de todo o banco
SELECT
    OBJECT_NAME(i.object_id) AS Tabela,
    i.name AS NomeIndice,
    i.type_desc AS TipoIndice
FROM sys.indexes i
WHERE i.object_id > 100  -- Exclui tabelas do sistema
ORDER BY Tabela, NomeIndice;
```

### Remover Índices

```sql
-- Remover índice
DROP INDEX IF EXISTS IX_Clientes_Email ON Clientes;

-- Remover múltiplos índices
DROP INDEX IF EXISTS IX_Clientes_Cidade_Estado ON Clientes;
DROP INDEX IF EXISTS IX_Clientes_Ativos ON Clientes;
```

### Reconstruir e Reorganizar Índices

Com o tempo, índices ficam fragmentados. Reconstruir ou reorganizar melhora a performance:

```sql
-- Reconstruir índice (offline - bloqueia tabela)
ALTER INDEX IX_Clientes_Email ON Clientes REBUILD;

-- Reconstruir todos os índices de uma tabela
ALTER INDEX ALL ON Clientes REBUILD;

-- Reorganizar índice (online - menos impacto)
ALTER INDEX IX_Clientes_Email ON Clientes REORGANIZE;

-- Reconstruir com opções
ALTER INDEX IX_Clientes_Email ON Clientes
REBUILD WITH (
    ONLINE = ON,           -- SQL Server Enterprise
    FILLFACTOR = 80,       -- Deixa 20% de espaço livre
    SORT_IN_TEMPDB = ON    -- Usa tempdb para sort
);
```

### Verificar Fragmentação

```sql
-- Verificar fragmentação de índices
SELECT
    OBJECT_NAME(ips.object_id) AS Tabela,
    i.name AS Indice,
    ips.index_type_desc AS TipoIndice,
    ips.avg_fragmentation_in_percent AS Fragmentacao,
    ips.page_count AS Paginas
FROM sys.dm_db_index_physical_stats(
    DB_ID(), NULL, NULL, NULL, 'DETAILED'
) ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id
    AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
    AND ips.page_count > 100
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- Regra geral:
-- Fragmentação 5-30%: REORGANIZE
-- Fragmentação > 30%: REBUILD
```

## Desabilitando Índices

```sql
-- Desabilitar índice (mantém metadados, mas não é usado)
ALTER INDEX IX_Clientes_Email ON Clientes DISABLE;

-- Reabilitar (precisa REBUILD)
ALTER INDEX IX_Clientes_Email ON Clientes REBUILD;
```

## Índices e Performance

### Analisando Planos de Execução

```sql
-- Ativar plano de execução no SSMS
-- Query > Include Actual Execution Plan (Ctrl+M)

-- Consulta sem índice (Table Scan)
SELECT * FROM Clientes WHERE Email = 'joao@email.com';

-- Criar índice
CREATE INDEX IX_Clientes_Email ON Clientes(Email);

-- Mesma consulta (Index Seek) - muito mais rápido!
SELECT * FROM Clientes WHERE Email = 'joao@email.com';
```

### Estatísticas de Uso de Índices

```sql
-- Índices não utilizados (candidatos para remoção)
SELECT
    OBJECT_NAME(s.object_id) AS Tabela,
    i.name AS Indice,
    s.user_seeks AS Buscas,
    s.user_scans AS Scans,
    s.user_lookups AS Lookups,
    s.user_updates AS Updates
FROM sys.dm_db_index_usage_stats s
INNER JOIN sys.indexes i ON s.object_id = i.object_id
    AND s.index_id = i.index_id
WHERE s.database_id = DB_ID()
    AND OBJECT_NAME(s.object_id) NOT LIKE 'sys%'
    AND i.name IS NOT NULL
ORDER BY (s.user_seeks + s.user_scans + s.user_lookups) ASC;

-- Índices com muitas atualizações e poucas leituras
-- (podem estar prejudicando performance de INSERT/UPDATE)
```

### Índices Faltantes (Sugeridos pelo SQL Server)

```sql
-- SQL Server sugere índices que podem melhorar consultas
SELECT
    mid.statement AS Tabela,
    migs.avg_user_impact AS ImpactoMedio,
    migs.avg_total_user_cost AS CustoMedio,
    migs.user_seeks AS Buscas,
    migs.user_scans AS Scans,
    'CREATE INDEX IX_' + OBJECT_NAME(mid.object_id) + '_Sugerido ON ' +
    mid.statement + ' (' + ISNULL(mid.equality_columns, '') +
    CASE WHEN mid.inequality_columns IS NOT NULL
        THEN CASE WHEN mid.equality_columns IS NOT NULL THEN ', ' ELSE '' END +
        mid.inequality_columns ELSE '' END + ')' +
    CASE WHEN mid.included_columns IS NOT NULL
        THEN ' INCLUDE (' + mid.included_columns + ')' ELSE '' END
    AS ComandoCriarIndice
FROM sys.dm_db_missing_index_details mid
INNER JOIN sys.dm_db_missing_index_groups mig ON mid.index_handle = mig.index_handle
INNER JOIN sys.dm_db_missing_index_group_stats migs ON mig.index_group_handle = migs.group_handle
WHERE mid.database_id = DB_ID()
ORDER BY migs.avg_user_impact DESC;
```

## Boas Práticas

### Views

1. **Nomeação**: Use prefixo `vw_` para identificar views
2. **Documentação**: Comente o propósito da view
3. **Simplicidade**: Evite views muito complexas
4. **Seletividade**: Não use SELECT * em views
5. **Segurança**: Use views para controlar acesso a dados sensíveis
6. **Evite cascata**: Não crie views baseadas em outras views (afeta performance)
7. **WITH SCHEMABINDING**: Use para views críticas (evita mudanças nas tabelas base)

```sql
-- View com SCHEMABINDING (protege estrutura)
CREATE VIEW vw_ClientesProtegida
WITH SCHEMABINDING
AS
SELECT
    ClienteID,
    Nome,
    Email
FROM dbo.Clientes  -- Precisa especificar schema
WHERE Ativo = 1;
GO

-- Agora não é possível alterar ou remover colunas usadas na view
```

### Boas Práticas para Índices

1. **Não exagere**: Cada índice consome espaço e afeta INSERT/UPDATE/DELETE
2. **Ordem importa**: Em índices compostos, coluna mais seletiva primeiro
3. **Monitore**: Remova índices não utilizados
4. **Mantenha**: Reconstrua índices fragmentados regularmente
5. **Teste**: Sempre teste impacto antes de criar índices em produção
6. **Filtre**: Use índices filtrados quando possível
7. **Inclua colunas**: Use INCLUDE para cobrir consultas frequentes

## Exercícios Práticos

<procedure title="Exercícios de Views e Índices" id="exercicios-views-indices">
    <step>
        <p><strong>Exercício 1:</strong> Crie uma view que mostre todos os produtos com estoque abaixo de 20 unidades, incluindo o valor total em estoque</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Crie uma view que liste os top 10 clientes por valor total gasto (apenas pedidos entregues)</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Crie um índice na coluna Estado da tabela Clientes e compare o plano de execução antes e depois</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Crie um índice composto em Pedidos(ClienteID, DataPedido) com INCLUDE (ValorTotal, Status)</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Crie um índice filtrado para produtos ativos da categoria 'Eletrônicos'</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Crie uma view atualizável para a tabela Produtos que permita editar apenas Nome, Preco e Estoque</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Liste todos os índices da tabela Pedidos com suas colunas</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Verifique a fragmentação dos índices do banco e identifique quais precisam de manutenção</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Consulte os índices sugeridos pelo SQL Server e crie um deles</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Crie uma view que combine dados de Clientes, Pedidos e mostre estatísticas por estado</p>
    </step>
</procedure>

## Glossary

**View (Visão)**
: Consulta SQL armazenada que funciona como uma tabela virtual

**Índice**
: Estrutura de dados que acelera a recuperação de informações

**Clustered Index**
: Índice que determina a ordem física dos dados na tabela (apenas um por tabela)

**Non-Clustered Index**
: Índice separado que aponta para os dados (até 999 por tabela)

**Index Seek**
: Operação eficiente que usa o índice para encontrar dados rapidamente

**Index Scan / Table Scan**
: Operação que lê todos os registros (lenta para tabelas grandes)

**Fragmentação**
: Desordenação física dos dados que degrada performance

**INCLUDE**
: Cláusula que adiciona colunas ao índice sem incluí-las na chave

**Filtered Index**
: Índice criado apenas para um subconjunto dos dados

**WITH SCHEMABINDING**
: Opção que protege a estrutura das tabelas base de uma view

**Index Covering**
: Quando todas as colunas necessárias estão no índice (não precisa acessar a tabela)

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/relational-databases/views/views">Microsoft Docs - Views</a>
        <a href="https://learn.microsoft.com/pt-br/sql/relational-databases/indexes/indexes">Microsoft Docs - Índices</a>
        <a href="https://www.sqlshack.com/sql-server-index-design-basics-and-guidelines/">SQL Shack - Index Design</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique criando views para consultas complexas
> - Experimente diferentes tipos de índices
> - Analise planos de execução antes e depois de criar índices
> - Na próxima aula, aprenderemos sobre Stored Procedures!
>
{style="tip"}
