# Aula 7: WHERE e Operadores Avançados

## Introdução

Na Aula 5, você aprendeu os fundamentos da cláusula WHERE e operadores básicos. Nesta aula, vamos aprofundar nosso conhecimento explorando operadores avançados, técnicas de filtragem complexa e padrões de consulta que você usará frequentemente em situações reais.

Nesta aula, você aprenderá:
- Operadores avançados e suas aplicações práticas
- Combinação de múltiplas condições de forma eficiente
- Padrões de busca complexos com LIKE
- Operadores de conjuntos e lógica avançada
- Técnicas de otimização de filtros
- Armadilhas comuns e como evitá-las

## Revisão Rápida: Operadores Básicos

Antes de avançar, vamos revisar rapidamente os operadores que já conhecemos:

```sql
USE LojaDB;
GO

-- Operadores de comparação: =, <>, <, >, <=, >=
SELECT * FROM Produtos WHERE Preco > 500;

-- Operadores lógicos: AND, OR, NOT
SELECT * FROM Clientes WHERE Estado = 'SP' AND Ativo = 1;

-- Operadores de intervalo: BETWEEN, IN
SELECT * FROM Produtos WHERE Preco BETWEEN 100 AND 500;
SELECT * FROM Clientes WHERE Estado IN ('SP', 'RJ', 'MG');

-- Operador de padrão: LIKE
SELECT * FROM Clientes WHERE Nome LIKE 'M%';

-- Operador de NULL: IS NULL, IS NOT NULL
SELECT * FROM Clientes WHERE Email IS NOT NULL;
```

## Operadores Avançados de Comparação

### Operador de Negação (<> vs !=)

Ambos significam "diferente de", mas `<>` é o padrão SQL:

```sql
-- Ambos funcionam, mas <> é preferível (padrão SQL)
SELECT Nome, Categoria FROM Produtos WHERE Categoria <> 'Livros';
SELECT Nome, Categoria FROM Produtos WHERE Categoria != 'Livros';

-- Negação com IN
SELECT Nome, Estado FROM Clientes WHERE Estado NOT IN ('SP', 'RJ');

-- Equivalente a múltiplos <>
SELECT Nome, Estado FROM Clientes
WHERE Estado <> 'SP' AND Estado <> 'RJ';
```

### Comparações com Strings

```sql
-- Comparação alfabética (case-insensitive por padrão)
SELECT Nome FROM Clientes WHERE Nome > 'M' ORDER BY Nome;

-- Comparação case-sensitive usando COLLATE
SELECT Nome FROM Clientes
WHERE Nome = 'joão silva' COLLATE Latin1_General_CS_AS;

-- Comparação case-insensitive explícita
SELECT Nome FROM Clientes
WHERE LOWER(Nome) = 'joão silva';

-- Comparar primeiros caracteres
SELECT Nome FROM Clientes
WHERE LEFT(Nome, 1) = 'J';
```

### Comparações com Datas

```sql
-- Clientes cadastrados hoje
SELECT Nome, DataCadastro FROM Clientes
WHERE CAST(DataCadastro AS DATE) = CAST(GETDATE() AS DATE);

-- Clientes cadastrados nos últimos 30 dias
SELECT Nome, DataCadastro FROM Clientes
WHERE DataCadastro >= DATEADD(DAY, -30, GETDATE());

-- Clientes cadastrados em 2023
SELECT Nome, DataCadastro FROM Clientes
WHERE YEAR(DataCadastro) = 2023;

-- Melhor para performance (usa índices)
SELECT Nome, DataCadastro FROM Clientes
WHERE DataCadastro >= '2023-01-01' AND DataCadastro < '2024-01-01';

-- Pedidos de um mês específico
SELECT * FROM Pedidos
WHERE DataPedido >= '2024-03-01' AND DataPedido < '2024-04-01';

-- Pedidos do trimestre atual
SELECT * FROM Pedidos
WHERE DATEPART(QUARTER, DataPedido) = DATEPART(QUARTER, GETDATE())
  AND YEAR(DataPedido) = YEAR(GETDATE());
```

## LIKE Avançado: Padrões Complexos

### Wildcards Detalhados

```SQL
-- % : Zero ou mais caracteres
SELECT Nome FROM Clientes WHERE Nome LIKE 'Ma%';       -- Começa com 'Ma'
SELECT Nome FROM Clientes WHERE Nome LIKE 'Silva%';   -- Contém 'Silva'
SELECT Nome FROM Clientes WHERE Nome LIKE '%Costa';    -- Termina com 'Costa'

-- _ : Exatamente um caractere
SELECT Nome FROM Clientes WHERE Nome LIKE '_arlos';    -- 'Carlos', 'Darlos', etc.
SELECT Nome FROM Clientes WHERE Nome LIKE 'J__o';      -- 'João', 'Jaco', etc.
SELECT Nome FROM Clientes WHERE Nome LIKE '____';      -- Exatamente 4 letras

-- [] : Qualquer caractere dentro dos colchetes
SELECT Nome FROM Produtos WHERE Nome LIKE '[MNP]%';    -- Começa com M, N ou P
SELECT Nome FROM Produtos WHERE Nome LIKE 'Mouse [LR]%'; -- 'Mouse Logitech' ou 'Mouse Razer'

-- [^] ou [!] : Qualquer caractere NÃO nos colchetes
SELECT Nome FROM Produtos WHERE Nome LIKE '[^A-C]%';   -- NÃO começa com A, B ou C
SELECT Nome FROM Clientes WHERE Nome LIKE '[!0-9]%';   -- NÃO começa com número

-- Intervalos com []
SELECT Nome FROM Produtos WHERE Nome LIKE '[A-F]%';    -- Começa com A até F
SELECT Email FROM Clientes WHERE Email LIKE '%[0-9]%'; -- Contém número
```

### Padrões Práticos com LIKE

```sql
-- Buscar emails por domínio
SELECT Nome, Email FROM Clientes WHERE Email LIKE '%@gmail.com';
SELECT Nome, Email FROM Clientes WHERE Email LIKE '%@hotmail%';

-- Buscar telefones com padrões específicos (assumindo coluna Telefone)
-- SELECT Nome, Telefone FROM Clientes WHERE Telefone LIKE '(11)%';  -- DDD 11

-- Buscar produtos com código específico
SELECT Nome FROM Produtos WHERE Nome LIKE 'NB-%';  -- Notebooks

-- Buscar nomes compostos (com espaço)
SELECT Nome FROM Clientes WHERE Nome LIKE '% %';

-- Buscar nomes com três palavras
SELECT Nome FROM Clientes WHERE Nome LIKE '% % %';

-- Escape de caracteres especiais
SELECT * FROM Produtos WHERE Nome LIKE '%[%]%';   -- Contém o caractere %
SELECT * FROM Produtos WHERE Nome LIKE '%[_]%';   -- Contém o caractere _

-- Usar ESCAPE para definir caractere de escape personalizado
SELECT * FROM Produtos WHERE Nome LIKE '%\%%' ESCAPE '\';  -- Contém %
SELECT * FROM Produtos WHERE Nome LIKE '%#_%' ESCAPE '#';  -- Contém _
```

### Alternativas ao LIKE para Performance

```sql
-- LIKE pode ser lento em tabelas grandes
-- Alternativa 1: CHARINDEX (mais rápido para substring simples)
SELECT Nome, Email FROM Clientes
WHERE CHARINDEX('silva', LOWER(Nome)) > 0;

-- Alternativa 2: PATINDEX (suporta wildcards)
SELECT Nome FROM Clientes
WHERE PATINDEX('%[Ss]ilva%', Nome) > 0;

-- Alternativa 3: Full-Text Search (para buscas complexas)
-- Requer configuração de índice full-text
-- SELECT * FROM Produtos WHERE CONTAINS(Nome, 'Mouse OR Teclado');
```

## Operadores Lógicos Avançados

### Precedência de Operadores

```sql
-- Sem parênteses: AND tem precedência sobre OR
-- Esta query pode não fazer o que você espera!
SELECT Nome, Categoria, Preco FROM Produtos
WHERE Categoria = 'Eletrônicos' OR Categoria = 'Acessórios' AND Preco < 100;
-- Interpreta como: (Eletrônicos) OU (Acessórios E Preço<100)

-- ✅ Use parênteses para clareza
SELECT Nome, Categoria, Preco FROM Produtos
WHERE (Categoria = 'Eletrônicos' OR Categoria = 'Acessórios') AND Preco < 100;
```

### Ordem de Precedência (do maior para o menor)

1. Parênteses `()`
2. Operadores de comparação `=, <>, <, >, <=, >=`
3. `NOT`
4. `AND`
5. `OR`

```sql
-- Exemplo complexo com precedência
SELECT Nome, Categoria, Preco, Estoque FROM Produtos
WHERE (Categoria = 'Eletrônicos' AND Preco > 500)
   OR (Categoria = 'Móveis' AND Estoque > 5)
   OR (Ativo = 1 AND Preco < 100);

-- NOT com precedência
SELECT Nome FROM Clientes
WHERE NOT (Estado = 'SP' OR Estado = 'RJ');  -- Nenhum dos dois

-- Equivalente a:
SELECT Nome FROM Clientes
WHERE Estado <> 'SP' AND Estado <> 'RJ';
```

### Condições Complexas

```sql
-- Produtos baratos OU caros, mas não médios
SELECT Nome, Preco FROM Produtos
WHERE Preco < 100 OR Preco > 1000;

-- Produtos de eletrônicos caros OU móveis em estoque
SELECT Nome, Categoria, Preco, Estoque FROM Produtos
WHERE (Categoria = 'Eletrônicos' AND Preco > 500)
   OR (Categoria = 'Móveis' AND Estoque > 0);

-- Clientes ativos de SP/RJ OU inativos de qualquer lugar
SELECT Nome, Estado, Ativo FROM Clientes
WHERE (Ativo = 1 AND Estado IN ('SP', 'RJ'))
   OR (Ativo = 0);

-- Produtos com problemas (sem estoque OU preço zerado OU inativo)
SELECT Nome, Preco, Estoque, Ativo FROM Produtos
WHERE Estoque = 0 OR Preco = 0 OR Ativo = 0;
```

## Operadores de Conjuntos

### IN com Subquery

```sql
-- Clientes que fizeram pedidos
SELECT Nome, Email FROM Clientes
WHERE ClienteID IN (SELECT DISTINCT ClienteID FROM Pedidos);

-- Produtos que nunca foram vendidos (assumindo tabela PedidoItens)
-- SELECT Nome FROM Produtos
-- WHERE ProdutoID NOT IN (SELECT DISTINCT ProdutoID FROM PedidoItens);

-- IN com múltiplos valores
SELECT Nome, Estado FROM Clientes
WHERE Estado IN ('SP', 'RJ', 'MG', 'ES');  -- Sudeste

-- IN é mais legível que múltiplos OR
-- ❌ Menos legível:
SELECT Nome FROM Clientes WHERE Estado = 'SP' OR Estado = 'RJ' OR Estado = 'MG';

-- ✅ Mais legível:
SELECT Nome FROM Clientes WHERE Estado IN ('SP', 'RJ', 'MG');
```

### EXISTS vs IN

`EXISTS` geralmente é mais eficiente que `IN` para subqueries:

```sql
-- Usando IN (pode ser lento)
SELECT Nome FROM Clientes
WHERE ClienteID IN (SELECT ClienteID FROM Pedidos WHERE ValorTotal > 1000);

-- Usando EXISTS (geralmente mais rápido)
SELECT C.Nome FROM Clientes C
WHERE EXISTS (
    SELECT 1 FROM Pedidos P
    WHERE P.ClienteID = C.ClienteID AND P.ValorTotal > 1000
);

-- NOT EXISTS: Clientes sem pedidos
SELECT C.Nome, C.Email FROM Clientes C
WHERE NOT EXISTS (
    SELECT 1 FROM Pedidos P WHERE P.ClienteID = C.ClienteID
);
```

## Operadores de Comparação de Intervalo

### BETWEEN Detalhado

```sql
-- BETWEEN é inclusivo (inclui os limites)
SELECT Nome, Preco FROM Produtos
WHERE Preco BETWEEN 100 AND 500;  -- Inclui 100 e 500

-- Equivalente a:
SELECT Nome, Preco FROM Produtos
WHERE Preco >= 100 AND Preco <= 500;

-- NOT BETWEEN
SELECT Nome, Preco FROM Produtos
WHERE Preco NOT BETWEEN 100 AND 500;

-- BETWEEN com datas
SELECT * FROM Pedidos
WHERE DataPedido BETWEEN '2024-01-01' AND '2024-03-31';

-- ⚠️ Cuidado com DATETIME
-- Este NÃO pega o dia 31/03 completo:
SELECT * FROM Pedidos
WHERE DataPedido BETWEEN '2024-01-01' AND '2024-03-31';

-- ✅ Melhor para DATETIME:
SELECT * FROM Pedidos
WHERE DataPedido >= '2024-01-01' AND DataPedido < '2024-04-01';

-- Ou use CAST para remover a hora:
SELECT * FROM Pedidos
WHERE CAST(DataPedido AS DATE) BETWEEN '2024-01-01' AND '2024-03-31';
```

### Comparações com ANY, ALL, SOME

```sql
-- ANY: Verdadeiro se a condição for verdadeira para QUALQUER valor
SELECT Nome, Preco FROM Produtos
WHERE Preco > ANY (SELECT Preco FROM Produtos WHERE Categoria = 'Livros');
-- Retorna produtos mais caros que QUALQUER livro (mesmo que o mais barato)

-- ALL: Verdadeiro se a condição for verdadeira para TODOS os valores
SELECT Nome, Preco FROM Produtos
WHERE Preco > ALL (SELECT Preco FROM Produtos WHERE Categoria = 'Livros');
-- Retorna produtos mais caros que TODOS os livros

-- SOME: Sinônimo de ANY
SELECT Nome, Preco FROM Produtos
WHERE Preco < SOME (SELECT Preco FROM Produtos WHERE Categoria = 'Eletrônicos');

-- Exemplo prático: Produtos mais caros que a média
SELECT Nome, Preco FROM Produtos
WHERE Preco > (SELECT AVG(Preco) FROM Produtos);
```

## Operadores Especiais

### Operador de String: +

```sql
-- Concatenação de strings
SELECT Nome + ' - ' + Cidade AS ClienteCompleto FROM Clientes;

-- Concatenação com NULL resulta em NULL
SELECT Nome + ' - ' + Email FROM Clientes;  -- NULL se Email for NULL

-- Use CONCAT ou COALESCE para evitar NULL
SELECT CONCAT(Nome, ' - ', COALESCE(Email, 'Sem email')) AS Info FROM Clientes;
```

### Operadores Aritméticos em WHERE

```sql
-- Filtrar por cálculos
SELECT Nome, Preco, Estoque FROM Produtos
WHERE Preco * Estoque > 5000;  -- Valor total em estoque > 5000

-- Calcular margem (assumindo coluna PrecoCusto)
-- SELECT Nome, Preco, PrecoCusto FROM Produtos
-- WHERE (Preco - PrecoCusto) / PrecoCusto > 0.3;  -- Margem > 30%

-- Módulo (resto da divisão)
SELECT Nome, ProdutoID FROM Produtos
WHERE ProdutoID % 2 = 0;  -- IDs pares

-- Arredondar para comparação
SELECT Nome, Preco FROM Produtos
WHERE ROUND(Preco, 0) = 100;  -- Preços próximos a 100
```

### Operadores Bitwise

```sql
-- Operadores bit a bit: &, |, ^, ~
-- Úteis para flags e permissões

-- Exemplo: Sistema de permissões (1=Leitura, 2=Escrita, 4=Exclusão)
DECLARE @Permissoes INT = 7;  -- 111 em binário (todas permissões)

-- Verificar se tem permissão de escrita (bit 2)
SELECT CASE WHEN @Permissoes & 2 = 2 THEN 'Tem permissão de escrita' ELSE 'Sem permissão' END;

-- Verificar múltiplas permissões
SELECT CASE WHEN @Permissoes & 5 = 5 THEN 'Tem leitura e exclusão' ELSE 'Falta permissão' END;
```

## Tratamento de NULL em Condições

### NULL é Especial

```sql
-- NULL não é igual a nada, nem a ele mesmo!
SELECT NULL = NULL;  -- Retorna NULL (não TRUE!)
SELECT NULL <> NULL; -- Retorna NULL
SELECT NULL > 0;     -- Retorna NULL

-- Use IS NULL ou IS NOT NULL
SELECT Nome, Email FROM Clientes WHERE Email IS NULL;
SELECT Nome, Email FROM Clientes WHERE Email IS NOT NULL;

-- ISNULL: Substituir NULL por um valor
SELECT Nome, ISNULL(Email, 'Não informado') AS Email FROM Clientes;

-- COALESCE: Retorna o primeiro valor não-NULL
SELECT Nome,
       COALESCE(Email, Telefone, 'Sem contato') AS Contato
FROM Clientes;

-- NULLIF: Retorna NULL se os valores forem iguais
SELECT Nome,
       NULLIF(Estoque, 0) AS EstoqueNaoZero
FROM Produtos;
-- Útil para evitar divisão por zero
```

### NULL em Comparações Lógicas

```sql
-- AND com NULL
SELECT 1 AND NULL;   -- NULL
SELECT 0 AND NULL;   -- 0 (FALSE)
SELECT NULL AND NULL; -- NULL

-- OR com NULL
SELECT 1 OR NULL;    -- 1 (TRUE)
SELECT 0 OR NULL;    -- NULL
SELECT NULL OR NULL; -- NULL

-- Implicações práticas
SELECT Nome FROM Clientes
WHERE Ativo = 1 OR Email IS NULL;  -- Retorna ativos OU sem email

-- Comportamento com NOT
SELECT Nome FROM Clientes
WHERE NOT (Ativo = 0);  -- NÃO retorna registros onde Ativo é NULL!

-- Para incluir NULL, seja explícito:
SELECT Nome FROM Clientes
WHERE Ativo <> 0 OR Ativo IS NULL;
```

## Técnicas de Filtragem Avançada

### Busca de Texto Flexível

```SQL
    -- Busca case-insensitive (padrão no SQL Server)
    SELECT Nome FROM Clientes WHERE Nome LIKE 'silva';
    
    -- Busca ignorando acentos (depende da collation)
    SELECT Nome FROM Clientes
    WHERE Nome COLLATE Latin1_General_CI_AI LIKE 'jose';  -- Acha 'José'
    
    -- Busca por múltiplas palavras (qualquer uma)
    SELECT Nome FROM Clientes
    WHERE Nome LIKE 'Silva' OR Nome LIKE 'Santos' OR Nome LIKE 'Costa';
    
    -- Busca por múltiplas palavras (todas)
    SELECT Nome FROM Clientes
    WHERE Nome LIKE 'João' AND Nome LIKE 'Silva';
    
    -- Remover espaços extras antes de comparar
    SELECT Nome FROM Clientes
    WHERE LTRIM(RTRIM(Nome)) = 'João Silva';
```

### Filtros Dinâmicos

```sql
-- Filtro opcional: Se parâmetro for NULL, ignora o filtro
DECLARE @FiltroEstado CHAR(2) = NULL;  -- NULL = buscar todos
DECLARE @FiltroAtivo BIT = 1;

SELECT Nome, Estado, Ativo FROM Clientes
WHERE (Estado = @FiltroEstado OR @FiltroEstado IS NULL)
  AND (Ativo = @FiltroAtivo OR @FiltroAtivo IS NULL);

-- Filtro de intervalo de datas opcional
DECLARE @DataInicio DATE = NULL;
DECLARE @DataFim DATE = NULL;

SELECT * FROM Pedidos
WHERE (DataPedido >= @DataInicio OR @DataInicio IS NULL)
  AND (DataPedido <= @DataFim OR @DataFim IS NULL);
```

### Filtros com Ranking

```sql
-- Top N por grupo (usando window functions)
WITH ProdutosRankeados AS (
    SELECT
        Nome,
        Categoria,
        Preco,
        ROW_NUMBER() OVER (PARTITION BY Categoria ORDER BY Preco DESC) AS Ranking
    FROM Produtos
)
SELECT Nome, Categoria, Preco
FROM ProdutosRankeados
WHERE Ranking <= 3;  -- Top 3 mais caros de cada categoria
```

## Otimização de Consultas com WHERE

### Boas Práticas

```sql
-- ✅ Bom: Usar colunas indexadas
SELECT * FROM Clientes WHERE ClienteID = 10;

-- ❌ Evite: Funções em colunas indexadas (impede uso do índice)
SELECT * FROM Clientes WHERE YEAR(DataCadastro) = 2023;

-- ✅ Melhor: Reescrever sem função
SELECT * FROM Clientes
WHERE DataCadastro >= '2023-01-01' AND DataCadastro < '2024-01-01';

-- ❌ Evite: LIKE começando com %
SELECT * FROM Clientes WHERE Nome LIKE 'Silva%';  -- Scan completo

-- ✅ Melhor: LIKE começando com literal
SELECT * FROM Clientes WHERE Nome LIKE 'Silva%';  -- Pode usar índice

-- ❌ Evite: OR entre colunas diferentes (dificulta índices)
SELECT * FROM Produtos WHERE Nome LIKE 'Mouse%' OR Categoria = 'Acessórios';

-- ✅ Considere: UNION ALL se apropriado
SELECT * FROM Produtos WHERE Nome LIKE 'Mouse%'
UNION ALL
SELECT * FROM Produtos WHERE Categoria = 'Acessórios' AND Nome NOT LIKE 'Mouse%';
```

### Uso de Índices

```sql
-- Ver índices de uma tabela
EXEC sp_helpindex 'Clientes';

-- Analisar plano de execução
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

SELECT * FROM Clientes WHERE Estado = 'SP';

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;
```

## Armadilhas Comuns e Soluções

### Armadilha 1: Comparação com NULL

```sql
-- ❌ ERRADO: Nunca use = NULL
SELECT * FROM Clientes WHERE Email = NULL;  -- Retorna 0 linhas!

-- ✅ CORRETO: Use IS NULL
SELECT * FROM Clientes WHERE Email IS NULL;
```

### Armadilha 2: Precedência de Operadores

```sql
-- ❌ CONFUSO: Sem parênteses
SELECT * FROM Produtos WHERE Categoria = 'Eletrônicos' OR Categoria = 'Acessórios' AND Preco < 100;

-- ✅ CLARO: Com parênteses
SELECT * FROM Produtos WHERE (Categoria = 'Eletrônicos' OR Categoria = 'Acessórios') AND Preco < 100;
```

### Armadilha 3: BETWEEN com DATETIME

```sql
-- ❌ PROBLEMA: Não pega o dia inteiro
SELECT * FROM Pedidos WHERE DataPedido BETWEEN '2024-03-01' AND '2024-03-31';

-- ✅ SOLUÇÃO: Use < no próximo dia
SELECT * FROM Pedidos WHERE DataPedido >= '2024-03-01' AND DataPedido < '2024-04-01';
```

### Armadilha 4: Divisão por Zero

```sql
-- ❌ ERRO: Pode dar divisão por zero
SELECT Nome, ValorTotal / Quantidade AS PrecoUnitario FROM Pedidos;

-- ✅ SOLUÇÃO: Verificar antes
SELECT Nome, CASE WHEN Quantidade > 0 THEN ValorTotal / Quantidade ELSE 0 END AS PrecoUnitario FROM Pedidos;

-- Ou usar NULLIF
SELECT Nome, ValorTotal / NULLIF(Quantidade, 0) AS PrecoUnitario FROM Pedidos;
```

### Armadilha 5: Comparação de Strings com Espaços

```sql
-- Pode não funcionar se houver espaços extras
SELECT * FROM Clientes WHERE Nome = 'João Silva ';  -- Espaço no final

-- ✅ Solução: Use TRIM
SELECT * FROM Clientes WHERE LTRIM(RTRIM(Nome)) = 'João Silva';

-- Ou use LIKE
SELECT * FROM Clientes WHERE Nome LIKE 'João Silva%';
```

## Exercícios Práticos

<procedure title="Exercícios para Praticar" id="exercicios-where-operadores">
    <step>
        <p><strong>Exercício 1:</strong> Encontre todos os produtos cujo nome contém 'Mouse' OU 'Teclado' (use LIKE)</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Liste clientes de SP, RJ ou MG que estão ativos (use IN e AND)</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Encontre produtos com preço entre 50 e 300 OU com estoque acima de 100</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Busque clientes cujo nome começa com 'A', 'B' ou 'C' (use LIKE com [])</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Liste produtos que NÃO são da categoria 'Livros' nem 'Acessórios' (use NOT IN)</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Encontre pedidos do primeiro trimestre de 2024 (janeiro a março)</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Liste produtos cujo valor em estoque (Preço × Estoque) seja maior que 10000</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Busque clientes que têm email informado E cadastrados nos últimos 6 meses</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Encontre produtos com IDs pares (use operador módulo %)</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Liste clientes cujo nome contém pelo menos duas palavras (dica: use LIKE '% %')</p>
    </step>
    <step>
        <p><strong>Exercício Desafio:</strong> Crie uma consulta que encontre produtos "problemáticos": sem estoque OU inativos OU com preço zerado, mas apenas se forem da categoria 'Eletrônicos' ou 'Móveis'</p>
    </step>
</procedure>

## Padrões de Consulta Comuns

### Padrão 1: Busca de Texto Parcial

```sql
-- Buscar por nome parcial (case-insensitive)
SELECT Nome, Email FROM Clientes
WHERE Nome LIKE '%' + 'silva' + '%';

-- Com parâmetro
DECLARE @Busca NVARCHAR(100) = 'silva';
SELECT Nome, Email FROM Clientes
WHERE Nome LIKE '%' + @Busca + '%';
```

### Padrão 2: Filtro Multi-Critério

```sql
-- Busca com múltiplos filtros opcionais
DECLARE @Nome NVARCHAR(100) = NULL;
DECLARE @Estado CHAR(2) = 'SP';
DECLARE @Ativo BIT = NULL;

SELECT * FROM Clientes
WHERE (@Nome IS NULL OR Nome LIKE '%' + @Nome + '%')
  AND (@Estado IS NULL OR Estado = @Estado)
  AND (@Ativo IS NULL OR Ativo = @Ativo);
```

### Padrão 3: Exclusão de Registros Relacionados

```sql
-- Clientes sem pedidos
SELECT C.Nome, C.Email FROM Clientes C
WHERE NOT EXISTS (SELECT 1 FROM Pedidos P WHERE P.ClienteID = C.ClienteID);

-- Produtos nunca vendidos
-- SELECT P.Nome FROM Produtos P
-- WHERE NOT EXISTS (SELECT 1 FROM PedidoItens I WHERE I.ProdutoID = P.ProdutoID);
```

### Padrão 4: Intervalo de Valores com Outliers

```sql
-- Produtos com preço normal (excluindo extremos)
SELECT Nome, Preco FROM Produtos
WHERE Preco BETWEEN 10 AND 5000
  AND Ativo = 1;

-- Ou usando percentis (mais robusto)
WITH Percentis AS (
    SELECT
        PERCENTILE_CONT(0.05) WITHIN GROUP (ORDER BY Preco) OVER () AS P5,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY Preco) OVER () AS P95
    FROM Produtos
)
SELECT DISTINCT P.Nome, P.Preco
FROM Produtos P
CROSS JOIN Percentis
WHERE P.Preco BETWEEN Percentis.P5 AND Percentis.P95;
```

## Resumo da Aula

Nesta aula, você aprendeu:

- ✅ Operadores avançados de comparação e suas nuances
- ✅ Padrões complexos com LIKE e wildcards
- ✅ Precedência de operadores lógicos e uso de parênteses
- ✅ Diferenças entre IN, EXISTS, ANY e ALL
- ✅ Tratamento correto de NULL em condições
- ✅ Técnicas de otimização de consultas
- ✅ Armadilhas comuns e como evitá-las

## Glossary

**Wildcard**
: Caractere especial usado em padrões LIKE (%, _, [], [^])

**Precedência de Operadores**
: Ordem em que operadores são avaliados em uma expressão

**EXISTS**
: Operador que verifica se uma subquery retorna algum resultado

**ANY/ALL/SOME**
: Operadores de comparação com conjuntos de valores

**Collation**
: Configuração que define como strings são comparadas (case-sensitive, accent-sensitive, etc.)

**CHARINDEX**
: Função que retorna a posição de uma substring dentro de uma string

**COALESCE**
: Função que retorna o primeiro valor não-NULL de uma lista

**Bitwise**
: Operações que trabalham com bits individuais de um valor

**Percentil**
: Valor abaixo do qual uma porcentagem de dados cai

**Índice**
: Estrutura de dados que melhora a velocidade de consultas

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/language-elements/operators-transact-sql">Microsoft Docs - Operadores</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/language-elements/like-transact-sql">Microsoft Docs - LIKE</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/language-elements/comparison-operators-transact-sql">Microsoft Docs - Operadores de Comparação</a>
        <a href="https://www.sqlshack.com/learn-sql-sql-server-operators/">SQL Shack - SQL Server Operators</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique todos os padrões de consulta apresentados
> - Complete os exercícios para fixar o conteúdo
> - Experimente combinações diferentes de operadores
> - Na próxima aula, faremos um projeto prático aplicando tudo que aprendemos!
>
{style="tip"}
