# Aula 11: Subconsultas (Subqueries)

## Introdução

Subconsultas, também conhecidas como subqueries, são consultas SQL aninhadas dentro de outra consulta. Elas são ferramentas poderosas que permitem resolver problemas complexos dividindo-os em partes menores e mais gerenciáveis.

Nesta aula, você aprenderá:
- O que são subconsultas e quando usá-las
- Subconsultas em diferentes cláusulas (WHERE, FROM, SELECT, HAVING)
- Subconsultas escalares, de linha única e múltiplas linhas
- Subconsultas correlacionadas vs não-correlacionadas
- CTEs (Common Table Expressions) com WITH
- Performance e otimização de subconsultas
- Alternativas a subconsultas (JOINs)

## Preparação: Banco de Dados

Vamos usar o banco BibliotecaDB:

```sql
USE BibliotecaDB;
GO
```

## O que são Subconsultas?

Uma subconsulta é uma consulta SELECT dentro de outra consulta SQL. Ela é executada primeiro, e seu resultado é usado pela consulta externa.

### Estrutura Básica

```sql
-- Estrutura geral
SELECT coluna
FROM tabela
WHERE coluna = (SELECT coluna FROM outra_tabela WHERE condição);
```

### Quando Usar Subconsultas

✅ **Use subconsultas quando:**
- Você precisa filtrar baseado em resultados de outra consulta
- O resultado depende de agregações ou cálculos complexos
- Você quer dividir um problema complexo em partes menores
- Precisa comparar valores com agregações

❌ **Evite subconsultas quando:**
- Um JOIN simples resolve o problema
- A subconsulta retorna muitos registros (considere JOIN)
- Você está usando subconsultas correlacionadas em tabelas grandes

## Tipos de Subconsultas

### 1. Subconsulta Escalar (Retorna um único valor)

```sql
-- Livros mais caros que a média
SELECT
    Titulo,
    Preco
FROM Livros
WHERE Preco > (SELECT AVG(Preco) FROM Livros);

-- Comparar com total geral
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros,
    (SELECT COUNT(*) FROM Livros) AS TotalGeralLivros,
    CAST(COUNT(L.LivroID) * 100.0 / (SELECT COUNT(*) FROM Livros) AS DECIMAL(5,2)) AS PercentualTotal
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome;

-- Mostrar livro mais emprestado
SELECT
    Titulo,
    (SELECT COUNT(*) FROM Emprestimos WHERE LivroID = L.LivroID) AS TotalEmprestimos
FROM Livros L
WHERE (SELECT COUNT(*) FROM Emprestimos WHERE LivroID = L.LivroID) > 0
ORDER BY TotalEmprestimos DESC;
```

### 2. Subconsulta de Linha Única

```sql
-- Encontrar o livro mais recente
SELECT Titulo, AnoPublicacao
FROM Livros
WHERE AnoPublicacao = (SELECT MAX(AnoPublicacao) FROM Livros);

-- Cliente que fez o pedido mais recente
SELECT Nome, Email
FROM Membros
WHERE MembroID = (
    SELECT MembroID
    FROM Emprestimos
    ORDER BY DataEmprestimo DESC
    OFFSET 0 ROWS FETCH NEXT 1 ROW ONLY
);

-- Categoria com mais livros
SELECT Nome
FROM Categorias
WHERE CategoriaID = (
    SELECT TOP 1 CategoriaID
    FROM Livros
    GROUP BY CategoriaID
    ORDER BY COUNT(*) DESC
);
```

### 3. Subconsulta de Múltiplas Linhas

```sql
-- Livros das categorias mais populares (com IN)
SELECT Titulo, CategoriaID
FROM Livros
WHERE CategoriaID IN (
    SELECT TOP 3 CategoriaID
    FROM Livros
    GROUP BY CategoriaID
    ORDER BY COUNT(*) DESC
);

-- Membros que fizeram empréstimos
SELECT Nome, Email
FROM Membros
WHERE MembroID IN (
    SELECT DISTINCT MembroID FROM Emprestimos
);

-- Livros de autores brasileiros
SELECT L.Titulo
FROM Livros L
WHERE L.LivroID IN (
    SELECT LA.LivroID
    FROM LivroAutor LA
    INNER JOIN Autores A ON LA.AutorID = A.AutorID
    WHERE A.Nacionalidade = 'Brasileira'
);
```

## Subconsultas em Diferentes Cláusulas

### Subconsultas na Cláusula WHERE

```sql
-- Livros acima da média de cópias da sua categoria
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    L.NumCopias
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE L.NumCopias > (
    SELECT AVG(NumCopias)
    FROM Livros L2
    WHERE L2.CategoriaID = L.CategoriaID
);

-- Membros sem empréstimos (usando NOT IN)
SELECT Nome, Email
FROM Membros
WHERE MembroID NOT IN (
    SELECT DISTINCT MembroID FROM Emprestimos
);

-- Usando NOT EXISTS (geralmente mais rápido)
SELECT Nome, Email
FROM Membros M
WHERE NOT EXISTS (
    SELECT 1 FROM Emprestimos E WHERE E.MembroID = M.MembroID
);

-- Membros com mais empréstimos que a média
SELECT M.Nome, COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
GROUP BY M.MembroID, M.Nome
HAVING COUNT(E.EmprestimoID) > (
    SELECT AVG(Total)
    FROM (
        SELECT COUNT(*) AS Total
        FROM Emprestimos
        GROUP BY MembroID
    ) AS MediaEmprestimos
);
```

### Subconsultas na Cláusula FROM (Tabela Derivada)

```sql
-- Usar resultado de agregação como tabela
SELECT
    Categoria,
    TotalLivros,
    CASE
        WHEN TotalLivros > 5 THEN 'Grande'
        WHEN TotalLivros >= 3 THEN 'Média'
        ELSE 'Pequena'
    END AS TamanhoCategoria
FROM (
    SELECT
        C.Nome AS Categoria,
        COUNT(L.LivroID) AS TotalLivros
    FROM Categorias C
    LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
    GROUP BY C.CategoriaID, C.Nome
) AS CategoriaStats;

-- Top 5 livros mais emprestados com informações extras
SELECT
    TopLivros.*,
    C.Nome AS Categoria
FROM (
    SELECT
        L.LivroID,
        L.Titulo,
        L.CategoriaID,
        COUNT(E.EmprestimoID) AS TotalEmprestimos
    FROM Livros L
    LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
    GROUP BY L.LivroID, L.Titulo, L.CategoriaID
) AS TopLivros
INNER JOIN Categorias C ON TopLivros.CategoriaID = C.CategoriaID
ORDER BY TopLivros.TotalEmprestimos DESC;

-- Análise de empréstimos por período
SELECT
    MesAno,
    TotalEmprestimos,
    AVG(TotalEmprestimos) OVER (ORDER BY MesAno ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS MediaMovel3Meses
FROM (
    SELECT
        FORMAT(DataEmprestimo, 'yyyy-MM') AS MesAno,
        COUNT(*) AS TotalEmprestimos
    FROM Emprestimos
    GROUP BY FORMAT(DataEmprestimo, 'yyyy-MM')
) AS EmprestimosPorMes
ORDER BY MesAno;
```

### Subconsultas na Cláusula SELECT

```sql
-- Adicionar contagens relacionadas
SELECT
    L.Titulo,
    L.AnoPublicacao,
    (SELECT COUNT(*) FROM Emprestimos E WHERE E.LivroID = L.LivroID) AS TotalEmprestimos,
    (SELECT COUNT(*) FROM Emprestimos E WHERE E.LivroID = L.LivroID AND E.Status = 'Ativo') AS EmprestimosAtivos,
    L.NumCopiasDisponiveis
FROM Livros L
ORDER BY TotalEmprestimos DESC;

-- Informações agregadas por membro
SELECT
    M.Nome,
    M.Email,
    (SELECT COUNT(*) FROM Emprestimos E WHERE E.MembroID = M.MembroID) AS TotalEmprestimos,
    (SELECT MAX(DataEmprestimo) FROM Emprestimos E WHERE E.MembroID = M.MembroID) AS UltimoEmprestimo,
    (SELECT COUNT(*) FROM Emprestimos E WHERE E.MembroID = M.MembroID AND E.Status = 'Atrasado') AS EmprestimosAtrasados
FROM Membros M
WHERE M.Ativo = 1
ORDER BY TotalEmprestimos DESC;

-- Comparar com média da categoria
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    L.NumCopias,
    (SELECT AVG(NumCopias) FROM Livros L2 WHERE L2.CategoriaID = L.CategoriaID) AS MediaCategoria,
    L.NumCopias - (SELECT AVG(NumCopias) FROM Livros L2 WHERE L2.CategoriaID = L.CategoriaID) AS DiferencaDaMedia
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID;
```

### Subconsultas na Cláusula HAVING

```sql
-- Categorias com mais livros que a média geral
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
HAVING COUNT(L.LivroID) > (
    SELECT AVG(Total)
    FROM (
        SELECT COUNT(*) AS Total
        FROM Livros
        GROUP BY CategoriaID
    ) AS MediaPorCategoria
);

-- Autores com mais empréstimos que o autor médio
SELECT
    A.Nome AS Autor,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Autores A
INNER JOIN LivroAutor LA ON A.AutorID = LA.AutorID
INNER JOIN Livros L ON LA.LivroID = L.LivroID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY A.AutorID, A.Nome
HAVING COUNT(E.EmprestimoID) > (
    SELECT AVG(TotalPorAutor)
    FROM (
        SELECT COUNT(E2.EmprestimoID) AS TotalPorAutor
        FROM Autores A2
        INNER JOIN LivroAutor LA2 ON A2.AutorID = LA2.AutorID
        INNER JOIN Livros L2 ON LA2.LivroID = L2.LivroID
        LEFT JOIN Emprestimos E2 ON L2.LivroID = E2.LivroID
        GROUP BY A2.AutorID
    ) AS MediaAutores
)
ORDER BY TotalEmprestimos DESC;
```

## Subconsultas Correlacionadas vs Não-Correlacionadas

### Subconsulta Não-Correlacionada

Executada **uma vez** e seu resultado é usado pela consulta externa:

```sql
-- NÃO-CORRELACIONADA: Executada uma vez
SELECT Titulo, AnoPublicacao
FROM Livros
WHERE AnoPublicacao > (SELECT AVG(AnoPublicacao) FROM Livros);

-- Livros de categorias populares
SELECT L.Titulo, C.Nome AS Categoria
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE L.CategoriaID IN (
    SELECT CategoriaID
    FROM Livros
    GROUP BY CategoriaID
    HAVING COUNT(*) >= 3
);
```

### Subconsulta Correlacionada

Executada **uma vez para cada linha** da consulta externa:

```sql
-- CORRELACIONADA: Executada para cada livro
SELECT
    L.Titulo,
    L.NumCopias,
    (SELECT AVG(NumCopias)
     FROM Livros L2
     WHERE L2.CategoriaID = L.CategoriaID) AS MediaCategoria
FROM Livros L;

-- Livros com mais cópias que a média da sua categoria
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    L.NumCopias
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE L.NumCopias > (
    SELECT AVG(L2.NumCopias)
    FROM Livros L2
    WHERE L2.CategoriaID = L.CategoriaID  -- Correlação aqui!
);

-- Membros com empréstimos acima da média do seu estado
SELECT
    M.Nome,
    M.Estado,
    (SELECT COUNT(*) FROM Emprestimos E WHERE E.MembroID = M.MembroID) AS TotalEmprestimos
FROM Membros M
WHERE (
    SELECT COUNT(*)
    FROM Emprestimos E
    WHERE E.MembroID = M.MembroID
) > (
    SELECT AVG(Total)
    FROM (
        SELECT COUNT(*) AS Total
        FROM Emprestimos E2
        INNER JOIN Membros M2 ON E2.MembroID = M2.MembroID
        WHERE M2.Estado = M.Estado
        GROUP BY M2.MembroID
    ) AS MediaEstado
);
```

### Performance: Correlacionadas são mais lentas

```sql
-- ❌ LENTO: Correlacionada (executada N vezes)
SELECT
    L.Titulo,
    (SELECT COUNT(*) FROM Emprestimos E WHERE E.LivroID = L.LivroID) AS Total
FROM Livros L;

-- ✅ RÁPIDO: JOIN (executada uma vez)
SELECT
    L.Titulo,
    COUNT(E.EmprestimoID) AS Total
FROM Livros L
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo;
```

## Operadores com Subconsultas

### EXISTS e NOT EXISTS

Verifica se a subconsulta retorna algum resultado:

```sql
-- Membros que fizeram pelo menos um empréstimo
SELECT Nome, Email
FROM Membros M
WHERE EXISTS (
    SELECT 1 FROM Emprestimos E WHERE E.MembroID = M.MembroID
);

-- Membros que NUNCA fizeram empréstimos
SELECT Nome, Email, DataCadastro
FROM Membros M
WHERE NOT EXISTS (
    SELECT 1 FROM Emprestimos E WHERE E.MembroID = M.MembroID
);

-- Categorias sem livros
SELECT Nome
FROM Categorias C
WHERE NOT EXISTS (
    SELECT 1 FROM Livros L WHERE L.CategoriaID = C.CategoriaID
);

-- Livros emprestados em 2024
SELECT L.Titulo
FROM Livros L
WHERE EXISTS (
    SELECT 1
    FROM Emprestimos E
    WHERE E.LivroID = L.LivroID
      AND YEAR(E.DataEmprestimo) = 2024
);
```

### IN e NOT IN

```sql
-- Livros de categorias específicas
SELECT Titulo
FROM Livros
WHERE CategoriaID IN (
    SELECT CategoriaID FROM Categorias WHERE Nome IN ('Ficção', 'Romance')
);

-- Membros ativos que não têm empréstimos atrasados
SELECT Nome, Email
FROM Membros
WHERE Ativo = 1
  AND MembroID NOT IN (
    SELECT MembroID FROM Emprestimos WHERE Status = 'Atrasado'
);

-- ⚠️ CUIDADO com NULL em NOT IN
-- Se a subconsulta retornar NULL, NOT IN pode não funcionar como esperado
-- Use NOT EXISTS ou WHERE coluna IS NOT NULL na subquery
```

### ANY, ALL, SOME

```sql
-- ANY: Maior que QUALQUER valor (mesmo que o menor)
SELECT Titulo, NumCopias
FROM Livros
WHERE NumCopias > ANY (
    SELECT NumCopias FROM Livros WHERE CategoriaID = 1
);

-- ALL: Maior que TODOS os valores
SELECT Titulo, NumCopias
FROM Livros
WHERE NumCopias > ALL (
    SELECT NumCopias FROM Livros WHERE CategoriaID = 1
);

-- SOME: Sinônimo de ANY
SELECT Titulo, AnoPublicacao
FROM Livros
WHERE AnoPublicacao = SOME (
    SELECT AnoPublicacao FROM Livros WHERE CategoriaID = 5
);

-- Exemplo prático: Livros mais caros que todos os livros de uma categoria
SELECT L.Titulo, L.NumCopias, C.Nome AS Categoria
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE L.NumCopias >= ALL (
    SELECT L2.NumCopias
    FROM Livros L2
    WHERE L2.CategoriaID = L.CategoriaID
);
```

## CTEs (Common Table Expressions) com WITH

CTEs são uma forma mais legível e organizada de trabalhar com subconsultas complexas:

### CTE Básico

```sql
-- Sintaxe básica
WITH NomeCTE AS (
    SELECT coluna FROM tabela WHERE condição
)
SELECT * FROM NomeCTE;

-- Exemplo: Livros com estatísticas
WITH LivrosStats AS (
    SELECT
        L.LivroID,
        L.Titulo,
        L.CategoriaID,
        COUNT(E.EmprestimoID) AS TotalEmprestimos
    FROM Livros L
    LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
    GROUP BY L.LivroID, L.Titulo, L.CategoriaID
)
SELECT
    LS.Titulo,
    C.Nome AS Categoria,
    LS.TotalEmprestimos
FROM LivrosStats LS
INNER JOIN Categorias C ON LS.CategoriaID = C.CategoriaID
WHERE LS.TotalEmprestimos > 2
ORDER BY LS.TotalEmprestimos DESC;
```

### Múltiplos CTEs

```sql
-- Definir múltiplos CTEs
WITH
    EstatisticasLivros AS (
        SELECT
            CategoriaID,
            COUNT(*) AS TotalLivros,
            AVG(NumCopias) AS MediaCopias
        FROM Livros
        GROUP BY CategoriaID
    ),
    EstatisticasEmprestimos AS (
        SELECT
            L.CategoriaID,
            COUNT(E.EmprestimoID) AS TotalEmprestimos
        FROM Livros L
        LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
        GROUP BY L.CategoriaID
    )
SELECT
    C.Nome AS Categoria,
    EL.TotalLivros,
    EL.MediaCopias,
    ISNULL(EE.TotalEmprestimos, 0) AS TotalEmprestimos
FROM Categorias C
LEFT JOIN EstatisticasLivros EL ON C.CategoriaID = EL.CategoriaID
LEFT JOIN EstatisticasEmprestimos EE ON C.CategoriaID = EE.CategoriaID
ORDER BY TotalEmprestimos DESC;
```

### CTEs Recursivos

```SQL
    -- Exemplo: Hierarquia de categorias (se houver relacionamento pai-filho)
    WITH CategoriasHierarquia AS (
        -- Anchor: Categorias raiz
        SELECT
            CategoriaID,
            Nome,
            CategoriaPaiID,
            0 AS Nivel,
            CAST(Nome AS NVARCHAR(MAX)) AS Caminho
        FROM Categorias
        WHERE CategoriaPaiID IS NULL
    
        UNION ALL
    
        -- Recursive: Subcategorias
        SELECT
            C.CategoriaID,
            C.Nome,
            C.CategoriaPaiID,
            CH.Nivel + 1,
            CAST(CH.Caminho + ' > ' + C.Nome AS NVARCHAR(MAX))
        FROM Categorias C
        INNER JOIN CategoriasHierarquia CH ON C.CategoriaPaiID = CH.CategoriaID
    )
    SELECT
        REPLICATE('  ', Nivel) + Nome AS NomeIndentado,
        Nivel,
        Caminho
    FROM CategoriasHierarquia
    ORDER BY Caminho;
    
    -- Exemplo: Sequência de números
    WITH Numeros AS (
        SELECT 1 AS N
        UNION ALL
        SELECT N + 1
        FROM Numeros
        WHERE N < 10
    )
    SELECT N FROM Numeros;
```

### Vantagens dos CTEs

```sql
-- ✅ Mais legível que subconsultas aninhadas
-- ✅ Pode ser referenciado múltiplas vezes
-- ✅ Suporta recursão
-- ✅ Mais fácil de debugar e manter

-- Exemplo complexo: Análise de empréstimos
WITH
    -- CTE 1: Empréstimos por livro
    EmprestimosPorLivro AS (
        SELECT
            LivroID,
            COUNT(*) AS TotalEmprestimos,
            SUM(CASE WHEN Status = 'Ativo' THEN 1 ELSE 0 END) AS Ativos,
            SUM(CASE WHEN Status = 'Atrasado' THEN 1 ELSE 0 END) AS Atrasados
        FROM Emprestimos
        GROUP BY LivroID
    ),
    -- CTE 2: Informações dos livros
    LivrosDetalhados AS (
        SELECT
            L.LivroID,
            L.Titulo,
            C.Nome AS Categoria,
            STRING_AGG(A.Nome, ', ') AS Autores
        FROM Livros L
        INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
        INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
        INNER JOIN Autores A ON LA.AutorID = A.AutorID
        GROUP BY L.LivroID, L.Titulo, C.Nome
    ),
    -- CTE 3: Ranking
    LivrosRankeados AS (
        SELECT
            LD.*,
            EPL.TotalEmprestimos,
            EPL.Ativos,
            EPL.Atrasados,
            DENSE_RANK() OVER (ORDER BY EPL.TotalEmprestimos DESC) AS Ranking
        FROM LivrosDetalhados LD
        LEFT JOIN EmprestimosPorLivro EPL ON LD.LivroID = EPL.LivroID
    )
-- Query final
SELECT
    Ranking,
    Titulo,
    Autores,
    Categoria,
    ISNULL(TotalEmprestimos, 0) AS TotalEmprestimos,
    ISNULL(Ativos, 0) AS Ativos,
    ISNULL(Atrasados, 0) AS Atrasados
FROM LivrosRankeados
WHERE Ranking <= 10
ORDER BY Ranking;
```

## Subconsultas vs JOINs

### Quando Usar Cada Um

```sql
-- Subconsulta: Bom para existência e agregações simples
SELECT Nome
FROM Membros M
WHERE EXISTS (SELECT 1 FROM Emprestimos E WHERE E.MembroID = M.MembroID);

-- JOIN: Melhor para trazer dados relacionados
SELECT M.Nome, COUNT(E.EmprestimoID) AS Total
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
GROUP BY M.MembroID, M.Nome;

-- Subconsulta: Para filtros baseados em agregação
SELECT Titulo
FROM Livros
WHERE CategoriaID IN (
    SELECT CategoriaID
    FROM Livros
    GROUP BY CategoriaID
    HAVING COUNT(*) > 5
);

-- JOIN equivalente (mais complexo)
SELECT DISTINCT L.Titulo
FROM Livros L
INNER JOIN (
    SELECT CategoriaID
    FROM Livros
    GROUP BY CategoriaID
    HAVING COUNT(*) > 5
) AS CatPopulares ON L.CategoriaID = CatPopulares.CategoriaID;
```

### Reescrevendo Subconsultas como JOINs

```sql
-- ANTES: Subconsulta correlacionada (pode ser lenta)
SELECT
    L.Titulo,
    (SELECT COUNT(*) FROM Emprestimos E WHERE E.LivroID = L.LivroID) AS Total
FROM Livros L;

-- DEPOIS: JOIN com agregação (geralmente mais rápido)
SELECT
    L.Titulo,
    COUNT(E.EmprestimoID) AS Total
FROM Livros L
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo;

-- ANTES: NOT IN
SELECT Nome
FROM Membros
WHERE MembroID NOT IN (SELECT MembroID FROM Emprestimos);

-- DEPOIS: LEFT JOIN com NULL
SELECT M.Nome
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
WHERE E.MembroID IS NULL;
```

## Padrões Comuns de Subconsultas

### Padrão 1: Top N por Grupo

```sql
-- Usando CTE com ROW_NUMBER
WITH LivrosRankeados AS (
    SELECT
        L.Titulo,
        C.Nome AS Categoria,
        L.AnoPublicacao,
        ROW_NUMBER() OVER (PARTITION BY C.CategoriaID ORDER BY L.AnoPublicacao DESC) AS RN
    FROM Livros L
    INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
)
SELECT Categoria, Titulo, AnoPublicacao
FROM LivrosRankeados
WHERE RN <= 3
ORDER BY Categoria, AnoPublicacao DESC;
```

### Padrão 2: Comparação com Agregação

```sql
-- Livros acima da média
WITH MediaPorCategoria AS (
    SELECT
        CategoriaID,
        AVG(CAST(NumCopias AS FLOAT)) AS MediaCopias
    FROM Livros
    GROUP BY CategoriaID
)
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    L.NumCopias,
    M.MediaCopias,
    L.NumCopias - M.MediaCopias AS DiferencaDaMedia
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
INNER JOIN MediaPorCategoria M ON L.CategoriaID = M.CategoriaID
WHERE L.NumCopias > M.MediaCopias
ORDER BY DiferencaDaMedia DESC;
```

### Padrão 3: Dados Ausentes (Anti-Join)

```sql
-- Categorias sem livros
SELECT C.Nome
FROM Categorias C
WHERE NOT EXISTS (
    SELECT 1 FROM Livros L WHERE L.CategoriaID = C.CategoriaID
);

-- Membros sem empréstimos em 2024
SELECT M.Nome, M.Email
FROM Membros M
WHERE NOT EXISTS (
    SELECT 1
    FROM Emprestimos E
    WHERE E.MembroID = M.MembroID
      AND YEAR(E.DataEmprestimo) = 2024
);
```

### Padrão 4: Running Totals e Acumulados

```sql
-- Total acumulado de empréstimos
WITH EmprestimosDiarios AS (
    SELECT
        CAST(DataEmprestimo AS DATE) AS Data,
        COUNT(*) AS Total
    FROM Emprestimos
    GROUP BY CAST(DataEmprestimo AS DATE)
)
SELECT
    Data,
    Total,
    (SELECT SUM(Total)
     FROM EmprestimosDiarios ED2
     WHERE ED2.Data <= ED.Data) AS TotalAcumulado
FROM EmprestimosDiarios ED
ORDER BY Data;
```

## Performance e Otimização

### Dicas de Performance

```sql
-- ✅ BOM: EXISTS para verificar existência
SELECT Nome FROM Membros M
WHERE EXISTS (SELECT 1 FROM Emprestimos E WHERE E.MembroID = M.MembroID);

-- ❌ EVITE: COUNT para verificar existência
SELECT Nome FROM Membros M
WHERE (SELECT COUNT(*) FROM Emprestimos E WHERE E.MembroID = M.MembroID) > 0;

-- ✅ BOM: Subconsulta não-correlacionada
SELECT Titulo FROM Livros WHERE CategoriaID IN (SELECT CategoriaID FROM Categorias WHERE Nome = 'Ficção');

-- ❌ EVITE: Subconsulta correlacionada desnecessária
SELECT L.Titulo FROM Livros L
WHERE EXISTS (SELECT 1 FROM Categorias C WHERE C.CategoriaID = L.CategoriaID AND C.Nome = 'Ficção');

-- ✅ MELHOR: JOIN simples
SELECT L.Titulo FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE C.Nome = 'Ficção';
```

### Analisar Performance

```sql
-- Comparar planos de execução
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Versão 1: Subconsulta
SELECT Titulo FROM Livros
WHERE CategoriaID IN (SELECT CategoriaID FROM Categorias WHERE Nome = 'Ficção');

-- Versão 2: JOIN
SELECT L.Titulo FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE C.Nome = 'Ficção';

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;
```

## Exercícios Práticos

<procedure title="Exercícios para Praticar" id="exercicios-subconsultas">
    <step>
        <p><strong>Exercício 1:</strong> Liste livros com mais cópias que a média geral</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Encontre membros que nunca fizeram empréstimos (use NOT EXISTS)</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Mostre livros da categoria com mais livros no acervo</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Liste autores que têm pelo menos um livro emprestado atualmente</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Encontre categorias com mais livros que a média de livros por categoria</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Crie um CTE que liste os 3 livros mais emprestados de cada categoria</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Mostre membros com mais empréstimos que a média do seu estado</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Liste livros publicados no mesmo ano que o livro mais recente</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Use subconsulta para calcular o percentual de empréstimos de cada livro sobre o total</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Crie um CTE recursivo que gere uma sequência de 1 a 100</p>
    </step>
</procedure>

## Soluções dos Exercícios

<procedure title="Soluções Sugeridas" id="solucoes-subconsultas" collapsible="true">
    <step>
        <p><strong>Solução 1:</strong> Livros acima da média</p>
        <code-block lang="sql">
SELECT Titulo, NumCopias
FROM Livros
WHERE NumCopias > (SELECT AVG(NumCopias) FROM Livros)
ORDER BY NumCopias DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 2:</strong> Membros sem empréstimos</p>
        <code-block lang="sql">
SELECT Nome, Email, DataCadastro
FROM Membros M
WHERE NOT EXISTS (
    SELECT 1 FROM Emprestimos E WHERE E.MembroID = M.MembroID
);
        </code-block>
    </step>
    <step>
        <p><strong>Solução 3:</strong> Livros da maior categoria</p>
        <code-block lang="sql">
SELECT L.Titulo, C.Nome AS Categoria
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE L.CategoriaID = (
    SELECT TOP 1 CategoriaID
    FROM Livros
    GROUP BY CategoriaID
    ORDER BY COUNT(*) DESC
);
        </code-block>
    </step>
    <step>
        <p><strong>Solução 5:</strong> Categorias acima da média</p>
        <code-block lang="sql">
SELECT C.Nome AS Categoria, COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
HAVING COUNT(L.LivroID) > (
    SELECT AVG(Total)
    FROM (
        SELECT COUNT(*) AS Total
        FROM Livros
        GROUP BY CategoriaID
    ) AS MediaCategorias
);
        </code-block>
    </step>
    <step>
        <p><strong>Solução 6:</strong> Top 3 por categoria com CTE</p>
        <code-block lang="sql">
WITH LivrosRankeados AS (
    SELECT
        L.Titulo,
        C.Nome AS Categoria,
        COUNT(E.EmprestimoID) AS TotalEmprestimos,
        ROW_NUMBER() OVER (PARTITION BY C.CategoriaID ORDER BY COUNT(E.EmprestimoID) DESC) AS Ranking
    FROM Livros L
    INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
    LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
    GROUP BY L.LivroID, L.Titulo, C.CategoriaID, C.Nome
)
SELECT Categoria, Titulo, TotalEmprestimos
FROM LivrosRankeados
WHERE Ranking <= 3
ORDER BY Categoria, Ranking;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 9:</strong> Percentual de empréstimos</p>
        <code-block lang="sql">
SELECT
    L.Titulo,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    (SELECT COUNT(*) FROM Emprestimos) AS TotalGeral,
    CAST(COUNT(E.EmprestimoID) * 100.0 / (SELECT COUNT(*) FROM Emprestimos) AS DECIMAL(5,2)) AS Percentual
FROM Livros L
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo
HAVING COUNT(E.EmprestimoID) > 0
ORDER BY Percentual DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 10:</strong> CTE recursivo 1-100</p>
        <code-block lang="sql">
WITH Numeros AS (
    SELECT 1 AS N
    UNION ALL
    SELECT N + 1
    FROM Numeros
    WHERE N < 100
)
SELECT N FROM Numeros
OPTION (MAXRECURSION 100);
        </code-block>
    </step>
</procedure>

## Resumo da Aula

Nesta aula, você aprendeu:

- ✅ O que são subconsultas e quando usá-las
- ✅ Tipos de subconsultas (escalar, linha única, múltiplas linhas)
- ✅ Subconsultas em WHERE, FROM, SELECT e HAVING
- ✅ Diferença entre correlacionadas e não-correlacionadas
- ✅ Operadores EXISTS, IN, ANY, ALL
- ✅ CTEs (WITH) e CTEs recursivos
- ✅ Quando usar subconsultas vs JOINs
- ✅ Padrões comuns e otimização

## Glossary

**Subconsulta (Subquery)**
: Consulta SQL aninhada dentro de outra consulta

**Subconsulta Escalar**
: Retorna um único valor (uma linha, uma coluna)

**Subconsulta Correlacionada**
: Referencia colunas da consulta externa e é executada para cada linha

**Subconsulta Não-Correlacionada**
: Independente da consulta externa, executada uma vez

**CTE (Common Table Expression)**
: Conjunto de resultados temporário definido com WITH

**Tabela Derivada**
: Subconsulta na cláusula FROM que age como uma tabela temporária

**EXISTS**
: Retorna TRUE se a subconsulta retornar pelo menos uma linha

**Anti-Join**
: Padrão que retorna linhas que NÃO têm correspondência (NOT EXISTS, NOT IN)

**CTE Recursivo**
: CTE que referencia a si mesmo para processar dados hierárquicos

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/queries/select-transact-sql">Microsoft Docs - SELECT (Subqueries)</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/queries/with-common-table-expression-transact-sql">Microsoft Docs - WITH (CTEs)</a>
        <a href="https://www.sqlshack.com/sql-subquery-basics/">SQL Shack - Subquery Basics</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique todos os tipos de subconsultas
> - Complete os exercícios propostos
> - Compare performance de subconsultas vs JOINs
> - Na próxima aula, aprenderemos sobre Views e Índices!
>
{style="tip"}
