# Aula 6: INSERT, UPDATE e DELETE

## Introdução

Após aprender a consultar dados com SELECT, agora vamos aprender os comandos DML (Data Manipulation Language) que permitem manipular os dados dentro das tabelas: **INSERT** para inserir novos registros, **UPDATE** para atualizar dados existentes e **DELETE** para remover registros.

Nesta aula, você aprenderá:
- Como inserir dados em tabelas (INSERT)
- Como atualizar registros existentes (UPDATE)
- Como deletar dados (DELETE)
- Diferença entre DELETE e TRUNCATE
- Uso de transações para segurança
- Boas práticas e precauções

## Preparação: Usando o Banco LojaDB

Vamos continuar usando o banco de dados LojaDB criado na aula anterior. Se você ainda não o criou, execute o script da Aula 5 primeiro.

```sql
USE LojaDB;
GO
```

## INSERT - Inserindo Dados

O comando INSERT adiciona novos registros em uma tabela.

### Sintaxe Básica do INSERT

```sql
-- Sintaxe 1: Especificando colunas
INSERT INTO Tabela (Coluna1, Coluna2, Coluna3)
VALUES (Valor1, Valor2, Valor3);

-- Sintaxe 2: Sem especificar colunas (deve seguir ordem da tabela)
INSERT INTO Tabela
VALUES (Valor1, Valor2, Valor3, ...);

-- Sintaxe 3: Múltiplos registros
INSERT INTO Tabela (Coluna1, Coluna2)
VALUES
    (Valor1A, Valor2A),
    (Valor1B, Valor2B),
    (Valor1C, Valor2C);
```

### INSERT Simples

```sql
-- Inserir um único cliente
INSERT INTO Clientes (Nome, Email, Cidade, Estado, Ativo)
VALUES ('Gustavo Pereira', 'gustavo@email.com', 'Campinas', 'SP', 1);

-- Verificar o que foi inserido
SELECT * FROM Clientes WHERE Nome = 'Gustavo Pereira';
```

### INSERT com Múltiplos Registros

```sql
-- Inserir vários produtos de uma vez
INSERT INTO Produtos (Nome, Categoria, Preco, Estoque, Ativo)
VALUES
    ('Teclado Sem Fio', 'Eletrônicos', 180.00, 45, 1),
    ('Mouse Gamer RGB', 'Eletrônicos', 150.00, 38, 1),
    ('Suporte para Notebook', 'Acessórios', 95.00, 22, 1),
    ('Luminária LED USB', 'Acessórios', 48.00, 55, 1),
    ('Fone Bluetooth', 'Eletrônicos', 220.00, 30, 1);

-- Verificar quantos foram inseridos
SELECT COUNT(*) AS TotalProdutos FROM Produtos;
```

### INSERT sem Especificar Colunas

⚠️ **Cuidado**: Deve-se fornecer valores para TODAS as colunas na ordem exata da tabela.

```sql
-- Ver estrutura da tabela
EXEC sp_help 'Clientes';

-- Inserir sem especificar colunas (NÃO RECOMENDADO)
-- Deve seguir a ordem: ClienteID, Nome, Email, Cidade, Estado, DataCadastro, Ativo
INSERT INTO Clientes
VALUES (NULL, 'Teste Nome', 'teste@email.com', 'São Paulo', 'SP', GETDATE(), 1);
-- Erro! ClienteID é IDENTITY, não pode ser NULL manualmente

-- ✅ MELHOR PRÁTICA: Sempre especifique as colunas
INSERT INTO Clientes (Nome, Email, Cidade, Estado)
VALUES ('Nome Correto', 'correto@email.com', 'Rio de Janeiro', 'RJ');
```

### INSERT com Valores Padrão

```sql
-- Usar DEFAULT para colunas com valor padrão
INSERT INTO Clientes (Nome, Email, Cidade, Estado, DataCadastro, Ativo)
VALUES ('Maria Luiza', 'maria.luiza@email.com', 'Fortaleza', 'CE', DEFAULT, DEFAULT);

-- Omitir colunas com valores padrão (mais comum)
INSERT INTO Clientes (Nome, Email, Cidade, Estado)
VALUES ('Paulo Henrique', 'paulo.h@email.com', 'Manaus', 'AM');
```

### INSERT com SELECT (Copiar Dados)

```sql
-- Criar tabela temporária para clientes inativos
CREATE TABLE ClientesInativos (
    ClienteID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100),
    Email VARCHAR(100),
    Cidade NVARCHAR(50),
    Estado CHAR(2),
    DataCadastro DATE
);

-- Copiar clientes inativos
INSERT INTO ClientesInativos (Nome, Email, Cidade, Estado, DataCadastro)
SELECT Nome, Email, Cidade, Estado, DataCadastro
FROM Clientes
WHERE Ativo = 0;

-- Verificar
SELECT * FROM ClientesInativos;

-- Limpar (opcional)
DROP TABLE ClientesInativos;
```

### INSERT e IDENTITY

Colunas IDENTITY são auto-incrementadas. O SQL Server gerencia automaticamente.

```sql
-- Inserir produto (ProdutoID é IDENTITY, será gerado automaticamente)
INSERT INTO Produtos (Nome, Categoria, Preco, Estoque)
VALUES ('Novo Produto', 'Eletrônicos', 299.99, 10);

-- Descobrir o último ID gerado
SELECT SCOPE_IDENTITY() AS UltimoID;

-- Alternativas para obter o ID inserido
SELECT @@IDENTITY AS UltimoIDGlobal;  -- Pode incluir triggers
SELECT IDENT_CURRENT('Produtos') AS UltimoIDTabela;  -- Por tabela

-- ✅ MELHOR: Usar SCOPE_IDENTITY() para evitar problemas
```

### INSERT com OUTPUT

Ver dados inseridos imediatamente:

```sql
-- Inserir e retornar o registro completo
INSERT INTO Clientes (Nome, Email, Cidade, Estado)
OUTPUT INSERTED.ClienteID, INSERTED.Nome, INSERTED.Email, INSERTED.DataCadastro
VALUES ('Carla Souza', 'carla@email.com', 'Curitiba', 'PR');

-- Inserir múltiplos e retornar IDs
INSERT INTO Produtos (Nome, Categoria, Preco, Estoque)
OUTPUT INSERTED.ProdutoID, INSERTED.Nome, INSERTED.Preco
VALUES
    ('Produto A', 'Categoria X', 100.00, 50),
    ('Produto B', 'Categoria Y', 200.00, 30),
    ('Produto C', 'Categoria Z', 150.00, 40);
```

## UPDATE - Atualizando Dados

O comando UPDATE modifica dados existentes em uma tabela.

### Sintaxe Básica do UPDATE

```sql
UPDATE Tabela
SET Coluna1 = NovoValor1,
    Coluna2 = NovoValor2
WHERE Condição;
```

⚠️ **ATENÇÃO CRÍTICA**: Sempre use WHERE no UPDATE! Sem WHERE, TODOS os registros serão atualizados!

### UPDATE Simples

```sql
-- SEMPRE faça um SELECT primeiro para ver o que será atualizado
SELECT * FROM Clientes WHERE ClienteID = 1;

-- Atualizar email de um cliente específico
UPDATE Clientes
SET Email = 'joao.silva.novo@email.com'
WHERE ClienteID = 1;

-- Verificar a alteração
SELECT * FROM Clientes WHERE ClienteID = 1;
```

### UPDATE de Múltiplas Colunas

```sql
-- Atualizar várias colunas ao mesmo tempo
UPDATE Clientes
SET Email = 'maria.atualizado@email.com',
    Cidade = 'Niterói',
    Estado = 'RJ'
WHERE ClienteID = 2;
```

### UPDATE com Cálculos

```sql
-- Aumentar preço de todos os produtos em 10%
UPDATE Produtos
SET Preco = Preco * 1.10
WHERE Categoria = 'Eletrônicos';

-- Adicionar 5 unidades ao estoque de produtos específicos
UPDATE Produtos
SET Estoque = Estoque + 5
WHERE Estoque < 20;

-- Aplicar desconto baseado em condição
UPDATE Produtos
SET Preco = CASE
    WHEN Preco > 1000 THEN Preco * 0.85  -- 15% desconto
    WHEN Preco > 500 THEN Preco * 0.90   -- 10% desconto
    ELSE Preco * 0.95                     -- 5% desconto
END
WHERE Categoria = 'Móveis';
```

### UPDATE com JOIN

Atualizar usando dados de outra tabela:

```sql
-- Exemplo: Atualizar status de pedidos baseado em data
UPDATE P
SET P.Status = 'Atrasado'
FROM Pedidos P
INNER JOIN Clientes C ON P.ClienteID = C.ClienteID
WHERE P.Status = 'Pendente'
  AND P.DataPedido < DATEADD(DAY, -7, GETDATE());

-- Atualizar preços baseado em categoria
UPDATE P
SET P.Preco = P.Preco * 1.15
FROM Produtos P
WHERE P.Categoria IN ('Livros', 'Acessórios');
```

### UPDATE com Subconsulta

```sql
-- Atualizar baseado em agregação
UPDATE Clientes
SET Email = 'vip@email.com'
WHERE ClienteID IN (
    SELECT ClienteID
    FROM Pedidos
    GROUP BY ClienteID
    HAVING SUM(ValorTotal) > 5000
);
```

### UPDATE com OUTPUT

Ver valores antes e depois da atualização:

```sql
-- Atualizar e ver mudanças
UPDATE Produtos
SET Preco = Preco * 1.10
OUTPUT
    DELETED.ProdutoID,
    DELETED.Nome,
    DELETED.Preco AS PrecoAntigo,
    INSERTED.Preco AS PrecoNovo,
    INSERTED.Preco - DELETED.Preco AS Diferenca
WHERE Categoria = 'Livros';
```

### UPDATE com TOP

Limitar quantidade de registros atualizados:

```sql
-- Atualizar apenas os 5 primeiros
UPDATE TOP (5) Produtos
SET Estoque = Estoque + 10
WHERE Estoque < 15;

-- Atualizar 10% dos registros
UPDATE TOP (10) PERCENT Produtos
SET Ativo = 0
WHERE Estoque = 0;
```

## DELETE - Removendo Dados

O comando DELETE remove registros de uma tabela.

### Sintaxe Básica do DELETE

```sql
DELETE FROM Tabela
WHERE Condição;
```

⚠️ **ATENÇÃO CRÍTICA**: Sempre use WHERE no DELETE! Sem WHERE, TODOS os registros serão removidos!

### DELETE Simples

```sql
-- SEMPRE faça um SELECT primeiro!
SELECT * FROM Clientes WHERE ClienteID = 100;

-- Se o registro existir e você tiver certeza, delete
DELETE FROM Clientes
WHERE ClienteID = 100;

-- Verificar se foi deletado
SELECT * FROM Clientes WHERE ClienteID = 100;
```

### DELETE com Condições

```sql
-- Deletar produtos sem estoque e inativos
SELECT * FROM Produtos WHERE Estoque = 0 AND Ativo = 0;  -- Verificar primeiro

DELETE FROM Produtos
WHERE Estoque = 0 AND Ativo = 0;

-- Deletar pedidos antigos
DELETE FROM Pedidos
WHERE Status = 'Cancelado'
  AND DataPedido < DATEADD(YEAR, -2, GETDATE());
```

### DELETE com JOIN

```sql
-- Deletar pedidos de clientes inativos
DELETE P
FROM Pedidos P
INNER JOIN Clientes C ON P.ClienteID = C.ClienteID
WHERE C.Ativo = 0;
```

### DELETE com Subconsulta

```sql
-- Deletar produtos que nunca foram pedidos
DELETE FROM Produtos
WHERE ProdutoID NOT IN (
    SELECT DISTINCT ProdutoID
    FROM PedidoItens  -- Assumindo que existe esta tabela
);
```

### DELETE com TOP

```sql
-- Deletar apenas os primeiros N registros
DELETE TOP (10) FROM Logs
WHERE DataLog < DATEADD(MONTH, -6, GETDATE());
```

### DELETE com OUTPUT

Ver o que foi deletado:

```sql
-- Deletar e ver registros removidos
DELETE FROM Produtos
OUTPUT
    DELETED.ProdutoID,
    DELETED.Nome,
    DELETED.Preco,
    DELETED.Estoque
WHERE Estoque = 0 AND Ativo = 0;
```

## TRUNCATE vs DELETE

Ambos removem dados, mas têm diferenças importantes:

### DELETE

```sql
-- DELETE: Remove linha por linha
DELETE FROM TabelaTeste;

-- Características:
-- ✓ Pode usar WHERE
-- ✓ Dispara triggers
-- ✓ Registra cada linha no log
-- ✓ Pode ser revertido com ROLLBACK
-- ✗ Mais lento para tabelas grandes
-- ✗ Mantém o contador IDENTITY
```

### TRUNCATE

```sql
-- TRUNCATE: Remove todos os dados rapidamente
TRUNCATE TABLE TabelaTeste;

-- Características:
-- ✓ Muito mais rápido
-- ✓ Usa menos espaço no log
-- ✓ Reseta contador IDENTITY para 1
-- ✗ NÃO pode usar WHERE
-- ✗ NÃO dispara triggers
-- ✗ Requer permissão ALTER TABLE
-- ✗ NÃO funciona se há FK referenciando
```

### Comparação Prática

```sql
-- Criar tabela de teste
CREATE TABLE TesteDelete (
    ID INT IDENTITY(1,1) PRIMARY KEY,
    Nome VARCHAR(50)
);

-- Inserir dados
INSERT INTO TesteDelete (Nome) VALUES ('A'), ('B'), ('C');
SELECT * FROM TesteDelete;  -- IDs: 1, 2, 3

-- DELETE e inserir novamente
DELETE FROM TesteDelete;
INSERT INTO TesteDelete (Nome) VALUES ('D');
SELECT * FROM TesteDelete;  -- ID: 4 (IDENTITY continua)

-- Recriar tabela
DROP TABLE TesteDelete;
CREATE TABLE TesteDelete (
    ID INT IDENTITY(1,1) PRIMARY KEY,
    Nome VARCHAR(50)
);

INSERT INTO TesteDelete (Nome) VALUES ('A'), ('B'), ('C');
TRUNCATE TABLE TesteDelete;
INSERT INTO TesteDelete (Nome) VALUES ('D');
SELECT * FROM TesteDelete;  -- ID: 1 (IDENTITY resetado)

-- Limpar
DROP TABLE TesteDelete;
```

### Quando Usar Cada Um

| Situação | Use |
|----------|-----|
| Remover todos os dados de uma tabela grande | TRUNCATE |
| Remover dados específicos com WHERE | DELETE |
| Resetar IDENTITY para 1 | TRUNCATE |
| Manter histórico no log de transações | DELETE |
| Disparar triggers de exclusão | DELETE |
| Tabela tem FK sendo referenciada | DELETE |
| Performance é crítica (milhões de linhas) | TRUNCATE |

## Transações - Segurança em Modificações

Transações garantem que um conjunto de operações seja executado completamente ou não seja executado.

### Sintaxe Básica de Transações

```sql
BEGIN TRANSACTION;  -- ou BEGIN TRAN

    -- Comandos SQL aqui
    INSERT INTO ...
    UPDATE ...
    DELETE FROM ...

    -- Se tudo OK:
    COMMIT;

    -- Se algo deu errado:
    -- ROLLBACK;
```

### Exemplo Prático: Transferência Bancária

```sql
-- Criar tabela de contas para exemplo
CREATE TABLE Contas (
    ContaID INT PRIMARY KEY,
    Titular VARCHAR(100),
    Saldo DECIMAL(10,2)
);

INSERT INTO Contas VALUES
    (1, 'João Silva', 1000.00),
    (2, 'Maria Santos', 500.00);

-- Transferir 200 reais da conta 1 para conta 2
BEGIN TRANSACTION;

    -- Debitar da conta 1
    UPDATE Contas
    SET Saldo = Saldo - 200
    WHERE ContaID = 1;

    -- Creditar na conta 2
    UPDATE Contas
    SET Saldo = Saldo + 200
    WHERE ContaID = 2;

    -- Verificar se os saldos estão corretos
    IF (SELECT Saldo FROM Contas WHERE ContaID = 1) >= 0
    BEGIN
        COMMIT;  -- Confirmar transação
        PRINT 'Transferência realizada com sucesso!';
    END
    ELSE
    BEGIN
        ROLLBACK;  -- Desfazer tudo
        PRINT 'Saldo insuficiente! Transação cancelada.';
    END

-- Verificar resultado
SELECT * FROM Contas;

-- Limpar
DROP TABLE Contas;
```

### Transação com TRY-CATCH

Tratamento de erros robusto:

```sql
BEGIN TRY
    BEGIN TRANSACTION;

        -- Operações perigosas
        UPDATE Produtos SET Preco = Preco * 1.10;
        DELETE FROM Produtos WHERE Estoque < 0;
        INSERT INTO LogAtualizacoes (Descricao, Data) VALUES ('Atualização de preços', GETDATE());

    COMMIT TRANSACTION;
    PRINT 'Operações concluídas com sucesso!';

END TRY
BEGIN CATCH
    -- Se qualquer erro ocorrer, desfazer tudo
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    -- Mostrar informações do erro
    PRINT 'Erro! Transação revertida.';
    PRINT 'Mensagem: ' + ERROR_MESSAGE();
    PRINT 'Linha: ' + CAST(ERROR_LINE() AS VARCHAR);
END CATCH;
```

### SAVEPOINT - Pontos de Salvamento

Permite desfazer parcialmente uma transação:

```sql
BEGIN TRANSACTION;

    UPDATE Produtos SET Preco = Preco * 1.05;
    SAVE TRANSACTION PontoSeguro;  -- Ponto de salvamento

    UPDATE Produtos SET Estoque = 0 WHERE Categoria = 'Teste';

    -- Se algo deu errado, voltar ao ponto de salvamento
    IF @@ROWCOUNT > 100  -- Muitos registros afetados
    BEGIN
        ROLLBACK TRANSACTION PontoSeguro;  -- Volta ao ponto, mas mantém primeiro UPDATE
        PRINT 'Rollback parcial executado';
    END

COMMIT TRANSACTION;
```

## Boas Práticas e Precauções

### 1. Sempre Teste com SELECT Primeiro

```sql
-- ❌ NUNCA faça isso direto:
DELETE FROM Clientes WHERE Cidade = 'São Paulo';

-- ✅ SEMPRE faça isso:
-- Passo 1: Ver o que será afetado
SELECT * FROM Clientes WHERE Cidade = 'São Paulo';

-- Passo 2: Se estiver certo, executar o DELETE
DELETE FROM Clientes WHERE Cidade = 'São Paulo';
```

### 2. Use Transações em Operações Críticas

```sql
-- ✅ Bom: Com transação
BEGIN TRANSACTION;
    UPDATE Produtos SET Preco = Preco * 1.10;
    -- Verificar se está OK
    SELECT COUNT(*) AS Afetados FROM Produtos WHERE Preco > 10000;
    -- Se OK: COMMIT, senão: ROLLBACK
COMMIT;
```

### 3. Faça Backup Antes de Operações Destrutivas

```sql
-- Backup de segurança antes de UPDATE/DELETE em massa
SELECT * INTO Produtos_Backup FROM Produtos;

-- Executar operação
DELETE FROM Produtos WHERE Ativo = 0;

-- Se precisar restaurar:
-- DELETE FROM Produtos;
-- INSERT INTO Produtos SELECT * FROM Produtos_Backup;

-- Limpar backup quando confirmar que está OK
-- DROP TABLE Produtos_Backup;
```

### 4. Cuidado com Foreign Keys

```sql
-- Erro: Tentar deletar cliente com pedidos
DELETE FROM Clientes WHERE ClienteID = 1;
-- Erro! Viola FK em Pedidos

-- ✅ Correto: Deletar pedidos primeiro
BEGIN TRANSACTION;
    DELETE FROM Pedidos WHERE ClienteID = 1;
    DELETE FROM Clientes WHERE ClienteID = 1;
COMMIT;

-- Ou usar CASCADE (definido na criação da FK)
```

### 5. Use OUTPUT para Auditar Mudanças

```sql
-- Criar tabela de auditoria
CREATE TABLE AuditoriaPrecos (
    AuditoriaID INT IDENTITY(1,1) PRIMARY KEY,
    ProdutoID INT,
    PrecoAntigo DECIMAL(10,2),
    PrecoNovo DECIMAL(10,2),
    DataAlteracao DATETIME2 DEFAULT GETDATE()
);

-- Update com auditoria
UPDATE Produtos
SET Preco = Preco * 1.10
OUTPUT
    DELETED.ProdutoID,
    DELETED.Preco,
    INSERTED.Preco,
    GETDATE()
INTO AuditoriaPrecos (ProdutoID, PrecoAntigo, PrecoNovo, DataAlteracao)
WHERE Categoria = 'Eletrônicos';

-- Ver auditoria
SELECT * FROM AuditoriaPrecos;
```

### 6. Evite SELECT * em INSERT/UPDATE de Produção

```sql
-- ❌ Ruim:
INSERT INTO TabelaNova SELECT * FROM TabelaVelha;

-- ✅ Melhor:
INSERT INTO TabelaNova (Col1, Col2, Col3)
SELECT Col1, Col2, Col3 FROM TabelaVelha;
```

### 7. Use Comentários em Scripts Complexos

```sql
-- Script de limpeza de dados
-- Objetivo: Remover registros antigos e limpar estoque zerado
-- Data: 2024-10-17
-- Autor: Leonardo

BEGIN TRANSACTION;

    -- Remover pedidos cancelados há mais de 1 ano
    DELETE FROM Pedidos
    WHERE Status = 'Cancelado'
      AND DataPedido < DATEADD(YEAR, -1, GETDATE());

    -- Inativar produtos sem estoque há mais de 6 meses
    UPDATE Produtos
    SET Ativo = 0
    WHERE Estoque = 0
      AND DataUltimaMovimentacao < DATEADD(MONTH, -6, GETDATE());

COMMIT;
```

## Exercícios Práticos

<procedure title="Exercícios para Praticar" id="exercicios-insert-update-delete">
    <step>
        <p><strong>Exercício 1:</strong> Insira 3 novos clientes na tabela Clientes</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Atualize o email de um cliente específico</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Aumente o preço de todos os produtos da categoria 'Livros' em 5%</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Delete todos os produtos inativos com estoque zerado</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Insira 5 novos produtos de uma vez usando múltiplos VALUES</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Crie uma transação que transfira estoque de um produto para outro</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Use OUTPUT para ver quais produtos tiveram o preço alterado</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Atualize o status de pedidos antigos para 'Arquivado'</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Delete pedidos de mais de 2 anos atrás (use WHERE com data)</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Crie uma cópia de segurança da tabela Produtos, faça alterações e depois restaure</p>
    </step>
</procedure>

## Glossary

**INSERT**
: Comando SQL para adicionar novos registros em uma tabela

**UPDATE**
: Comando SQL para modificar dados existentes em uma tabela

**DELETE**
: Comando SQL para remover registros de uma tabela

**TRUNCATE**
: Comando DDL que remove todos os dados de uma tabela rapidamente e reseta IDENTITY

**IDENTITY**
: Propriedade de coluna que gera valores auto-incrementais

**SCOPE_IDENTITY()**
: Função que retorna o último valor IDENTITY inserido no escopo atual

**OUTPUT**
: Cláusula que retorna dados afetados por INSERT, UPDATE ou DELETE

**Transaction (Transação)**
: Conjunto de operações tratadas como uma unidade única (tudo ou nada)

**COMMIT**
: Confirma e torna permanentes as mudanças de uma transação

**ROLLBACK**
: Desfaz todas as mudanças de uma transação

**SAVEPOINT**
: Ponto de salvamento dentro de uma transação que permite rollback parcial

**Foreign Key (Chave Estrangeira)**
: Restrição que cria relacionamento entre tabelas

**TRY-CATCH**
: Estrutura de tratamento de erros no SQL Server

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/statements/insert-transact-sql">Microsoft Docs - INSERT</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/statements/update-transact-sql">Microsoft Docs - UPDATE</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/statements/delete-transact-sql">Microsoft Docs - DELETE</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/language-elements/transactions-transact-sql">Microsoft Docs - Transações</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique todos os comandos desta aula em um ambiente seguro
> - SEMPRE use transações em operações importantes
> - Complete os exercícios propostos
> - Na próxima aula, aprofundaremos em WHERE e operadores avançados!
>
{style="tip"}
