# Aula 10: Funções Agregadas e GROUP BY

## Introdução

Funções agregadas são ferramentas poderosas que permitem realizar cálculos sobre conjuntos de dados, como somas, médias, contagens e valores máximos/mínimos. Combinadas com a cláusula GROUP BY, elas possibilitam análises e relatórios sofisticados sobre seus dados.

Nesta aula, você aprenderá:
- Funções agregadas principais (COUNT, SUM, AVG, MIN, MAX)
- Cláusula GROUP BY para agrupamento
- HAVING para filtrar grupos
- Agregações com JOINs
- Funções agregadas avançadas
- Window functions (ROW_NUMBER, RANK, PARTITION BY)
- Boas práticas e otimização

## Preparação: Banco de Dados

Vamos usar o banco BibliotecaDB. Certifique-se de que está usando este banco:

```sql
USE BibliotecaDB;
GO
```

## Funções Agregadas Básicas

Funções agregadas operam sobre múltiplas linhas e retornam um único valor.

### COUNT - Contar Registros

```sql
-- Contar total de registros
SELECT COUNT(*) AS TotalLivros FROM Livros;

-- Contar valores não-nulos em uma coluna
SELECT COUNT(Email) AS MembrosComEmail FROM Membros;

-- Contar valores distintos
SELECT COUNT(DISTINCT CategoriaID) AS TotalCategorias FROM Livros;
SELECT COUNT(DISTINCT Estado) AS TotalEstados FROM Membros;

-- COUNT com filtro
SELECT COUNT(*) AS LivrosDisponiveis
FROM Livros
WHERE NumCopiasDisponiveis > 0;

-- Diferença entre COUNT(*) e COUNT(coluna)
SELECT
    COUNT(*) AS TotalMembros,
    COUNT(Email) AS MembrosComEmail,
    COUNT(*) - COUNT(Email) AS MembrosSemEmail
FROM Membros;
```

### SUM - Somar Valores

```sql
-- Soma de valores numéricos
SELECT SUM(NumCopias) AS TotalCopiasAcervo FROM Livros;

-- Soma com cálculo
SELECT SUM(Preco * Estoque) AS ValorTotalEstoque FROM Produtos;

-- Soma condicional com CASE
SELECT
    SUM(CASE WHEN Ativo = 1 THEN 1 ELSE 0 END) AS MembrosAtivos,
    SUM(CASE WHEN Ativo = 0 THEN 1 ELSE 0 END) AS MembrosInativos
FROM Membros;

-- Soma de empréstimos por status
SELECT
    SUM(CASE WHEN Status = 'Ativo' THEN 1 ELSE 0 END) AS EmprestimosAtivos,
    SUM(CASE WHEN Status = 'Devolvido' THEN 1 ELSE 0 END) AS EmprestimosDevolvidos,
    SUM(CASE WHEN Status = 'Atrasado' THEN 1 ELSE 0 END) AS EmprestimosAtrasados
FROM Emprestimos;
```

### AVG - Calcular Média

```sql
-- Média simples
SELECT AVG(NumCopias) AS MediaCopiasLivro FROM Livros;

-- Média com filtro
SELECT AVG(AnoPublicacao) AS MediaAnoPublicacao
FROM Livros
WHERE AnoPublicacao > 1900;

-- Média de dias até devolução
SELECT AVG(DATEDIFF(DAY, DataEmprestimo, DataDevolucaoReal)) AS MediaDiasDevolucao
FROM Emprestimos
WHERE DataDevolucaoReal IS NOT NULL;

-- IMPORTANTE: AVG ignora NULL automaticamente
SELECT
    AVG(NumCopiasDisponiveis) AS MediaComNULL,
    AVG(ISNULL(NumCopiasDisponiveis, 0)) AS MediaSemNULL
FROM Livros;
```

### MIN e MAX - Valores Mínimo e Máximo

```sql
-- Menor e maior valor
SELECT
    MIN(AnoPublicacao) AS LivroMaisAntigo,
    MAX(AnoPublicacao) AS LivroMaisRecente
FROM Livros;

-- MIN/MAX com strings (ordem alfabética)
SELECT
    MIN(Nome) AS PrimeiroAutor,
    MAX(Nome) AS UltimoAutor
FROM Autores;

-- MIN/MAX com datas
SELECT
    MIN(DataCadastro) AS PrimeiroCadastro,
    MAX(DataCadastro) AS UltimoCadastro
FROM Membros;

-- Combinar múltiplas agregações
SELECT
    COUNT(*) AS TotalLivros,
    MIN(NumCopias) AS MenorQuantidade,
    MAX(NumCopias) AS MaiorQuantidade,
    AVG(NumCopias) AS MediaCopias,
    SUM(NumCopias) AS TotalCopias
FROM Livros;
```

## GROUP BY - Agrupamento de Dados

GROUP BY agrupa linhas que têm valores iguais em colunas especificadas e permite aplicar funções agregadas a cada grupo.

### Sintaxe Básica

```sql
SELECT coluna_agrupamento, FUNCAO_AGREGADA(coluna)
FROM tabela
GROUP BY coluna_agrupamento;
```

### GROUP BY Simples

```sql
-- Contar livros por categoria
SELECT
    CategoriaID,
    COUNT(*) AS TotalLivros
FROM Livros
GROUP BY CategoriaID
ORDER BY TotalLivros DESC;

-- Com JOIN para nomes legíveis
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalLivros DESC;

-- Empréstimos por status
SELECT
    Status,
    COUNT(*) AS Total
FROM Emprestimos
GROUP BY Status
ORDER BY Total DESC;

-- Membros por estado
SELECT
    Estado,
    COUNT(*) AS TotalMembros
FROM Membros
WHERE Ativo = 1
GROUP BY Estado
ORDER BY Estado;
```

### GROUP BY com Múltiplas Colunas

```sql
-- Livros por categoria e disponibilidade
SELECT
    C.Nome AS Categoria,
    CASE
        WHEN L.NumCopiasDisponiveis > 0 THEN 'Disponível'
        ELSE 'Indisponível'
    END AS Disponibilidade,
    COUNT(*) AS TotalLivros
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
GROUP BY C.Nome,
    CASE
        WHEN L.NumCopiasDisponiveis > 0 THEN 'Disponível'
        ELSE 'Indisponível'
    END
ORDER BY C.Nome, Disponibilidade;

-- Empréstimos por ano e mês
SELECT
    YEAR(DataEmprestimo) AS Ano,
    MONTH(DataEmprestimo) AS Mes,
    COUNT(*) AS TotalEmprestimos
FROM Emprestimos
GROUP BY YEAR(DataEmprestimo), MONTH(DataEmprestimo)
ORDER BY Ano DESC, Mes DESC;

-- Formato mais legível
SELECT
    FORMAT(DataEmprestimo, 'yyyy-MM') AS AnoMes,
    FORMAT(DataEmprestimo, 'MMMM yyyy', 'pt-BR') AS MesExtenso,
    COUNT(*) AS TotalEmprestimos
FROM Emprestimos
GROUP BY FORMAT(DataEmprestimo, 'yyyy-MM'), FORMAT(DataEmprestimo, 'MMMM yyyy', 'pt-BR')
ORDER BY AnoMes DESC;
```

### GROUP BY com Múltiplas Agregações

```sql
-- Estatísticas completas por categoria
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros,
    SUM(L.NumCopias) AS TotalCopias,
    SUM(L.NumCopiasDisponiveis) AS CopiasDisponiveis,
    AVG(CAST(L.NumCopiasDisponiveis AS FLOAT) / NULLIF(L.NumCopias, 0) * 100) AS PercentualDisponivel
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalLivros DESC;

-- Análise de empréstimos por membro
SELECT
    M.Nome AS Membro,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    SUM(CASE WHEN E.Status = 'Ativo' THEN 1 ELSE 0 END) AS Ativos,
    SUM(CASE WHEN E.Status = 'Devolvido' THEN 1 ELSE 0 END) AS Devolvidos,
    SUM(CASE WHEN E.Status = 'Atrasado' THEN 1 ELSE 0 END) AS Atrasados,
    MIN(E.DataEmprestimo) AS PrimeiroEmprestimo,
    MAX(E.DataEmprestimo) AS UltimoEmprestimo
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
GROUP BY M.MembroID, M.Nome
HAVING COUNT(E.EmprestimoID) > 0
ORDER BY TotalEmprestimos DESC;
```

## HAVING - Filtrar Grupos

HAVING é como WHERE, mas funciona com grupos (após GROUP BY). WHERE filtra linhas antes do agrupamento, HAVING filtra grupos depois.

### Diferença entre WHERE e HAVING

```sql
-- WHERE: Filtra ANTES de agrupar
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
WHERE L.AnoPublicacao >= 1900  -- Filtra livros individuais
GROUP BY C.CategoriaID, C.Nome;

-- HAVING: Filtra DEPOIS de agrupar
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
HAVING COUNT(L.LivroID) >= 3  -- Filtra grupos (categorias)
ORDER BY TotalLivros DESC;

-- Combinando WHERE e HAVING
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
WHERE L.AnoPublicacao >= 1900  -- Primeiro: filtra livros antigos
GROUP BY C.CategoriaID, C.Nome
HAVING COUNT(L.LivroID) >= 2   -- Depois: mantém categorias com 2+ livros
ORDER BY TotalLivros DESC;
```

### Exemplos Práticos com HAVING

```sql
-- Categorias com mais de 3 livros
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
HAVING COUNT(L.LivroID) > 3;

-- Membros com mais de 2 empréstimos
SELECT
    M.Nome,
    M.Email,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Membros M
INNER JOIN Emprestimos E ON M.MembroID = E.MembroID
GROUP BY M.MembroID, M.Nome, M.Email
HAVING COUNT(E.EmprestimoID) > 2
ORDER BY TotalEmprestimos DESC;

-- Autores com média de ano de publicação acima de 1980
SELECT
    A.Nome AS Autor,
    COUNT(L.LivroID) AS TotalLivros,
    AVG(L.AnoPublicacao) AS MediaAno
FROM Autores A
INNER JOIN LivroAutor LA ON A.AutorID = LA.AutorID
INNER JOIN Livros L ON LA.LivroID = L.LivroID
GROUP BY A.AutorID, A.Nome
HAVING AVG(L.AnoPublicacao) > 1980
ORDER BY MediaAno DESC;

-- HAVING com múltiplas condições
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros,
    SUM(L.NumCopias) AS TotalCopias
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
HAVING COUNT(L.LivroID) >= 2 AND SUM(L.NumCopias) >= 5
ORDER BY TotalLivros DESC;
```

## Agregações com JOINs

Combinar agregações com JOINs permite análises complexas:

```sql
-- Livros mais emprestados
SELECT TOP 10
    L.Titulo,
    A.Nome AS Autor,
    C.Nome AS Categoria,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    SUM(CASE WHEN E.Status = 'Ativo' THEN 1 ELSE 0 END) AS AtualmenteEmprestado
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo, A.Nome, C.Nome
ORDER BY TotalEmprestimos DESC;

-- Ranking de categorias por empréstimos
SELECT
    C.Nome AS Categoria,
    COUNT(DISTINCT L.LivroID) AS TotalLivros,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    CAST(COUNT(E.EmprestimoID) AS FLOAT) / NULLIF(COUNT(DISTINCT L.LivroID), 0) AS MediaEmprestimosPorLivro
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalEmprestimos DESC;

-- Autores mais populares
SELECT
    A.Nome AS Autor,
    A.Nacionalidade,
    COUNT(DISTINCT L.LivroID) AS LivrosNoBiblioteca,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    SUM(CASE WHEN E.Status = 'Ativo' THEN 1 ELSE 0 END) AS EmprestimosAtivos
FROM Autores A
INNER JOIN LivroAutor LA ON A.AutorID = LA.AutorID
INNER JOIN Livros L ON LA.LivroID = L.LivroID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY A.AutorID, A.Nome, A.Nacionalidade
HAVING COUNT(E.EmprestimoID) > 0
ORDER BY TotalEmprestimos DESC;
```

## Funções Agregadas Avançadas

### STRING_AGG - Concatenar Valores

```sql
-- Listar autores de cada livro (SQL Server 2017+)
SELECT
    L.Titulo,
    STRING_AGG(A.Nome, ', ') AS Autores
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
GROUP BY L.LivroID, L.Titulo
ORDER BY L.Titulo;

-- Com ORDER BY dentro do STRING_AGG
SELECT
    L.Titulo,
    STRING_AGG(A.Nome, ', ') WITHIN GROUP (ORDER BY A.Nome) AS Autores
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
GROUP BY L.LivroID, L.Titulo;

-- Listar livros por autor
SELECT
    A.Nome AS Autor,
    STRING_AGG(L.Titulo, ' | ') AS Livros,
    COUNT(L.LivroID) AS TotalLivros
FROM Autores A
INNER JOIN LivroAutor LA ON A.AutorID = LA.AutorID
INNER JOIN Livros L ON LA.LivroID = L.LivroID
GROUP BY A.AutorID, A.Nome;
```

### GROUPING SETS, ROLLUP e CUBE

Permitem criar múltiplos níveis de agregação em uma única consulta:

```sql
-- ROLLUP: Cria subtotais hierárquicos
SELECT
    C.Nome AS Categoria,
    YEAR(E.DataEmprestimo) AS Ano,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Emprestimos E
INNER JOIN Livros L ON E.LivroID = L.LivroID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
GROUP BY ROLLUP (C.Nome, YEAR(E.DataEmprestimo))
ORDER BY C.Nome, Ano;

-- CUBE: Cria todas as combinações possíveis
SELECT
    C.Nome AS Categoria,
    E.Status,
    COUNT(E.EmprestimoID) AS Total
FROM Emprestimos E
INNER JOIN Livros L ON E.LivroID = L.LivroID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
GROUP BY CUBE (C.Nome, E.Status)
ORDER BY C.Nome, E.Status;

-- GROUPING SETS: Especificar exatamente quais agrupamentos criar
SELECT
    C.Nome AS Categoria,
    E.Status,
    COUNT(E.EmprestimoID) AS Total
FROM Emprestimos E
INNER JOIN Livros L ON E.LivroID = L.LivroID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
GROUP BY GROUPING SETS (
    (C.Nome, E.Status),  -- Por categoria e status
    (C.Nome),            -- Apenas por categoria
    ()                   -- Total geral
)
ORDER BY C.Nome, E.Status;
```

## Window Functions (Funções de Janela)

Window functions permitem cálculos sobre um conjunto de linhas relacionadas mantendo o detalhe individual:

### ROW_NUMBER - Numerar Linhas

```sql
-- Numerar livros por categoria
SELECT
    C.Nome AS Categoria,
    L.Titulo,
    L.AnoPublicacao,
    ROW_NUMBER() OVER (PARTITION BY C.CategoriaID ORDER BY L.AnoPublicacao DESC) AS NumeroLinha
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
ORDER BY C.Nome, NumeroLinha;

-- Top 3 livros mais emprestados por categoria
WITH LivrosRanked AS (
    SELECT
        C.Nome AS Categoria,
        L.Titulo,
        COUNT(E.EmprestimoID) AS TotalEmprestimos,
        ROW_NUMBER() OVER (PARTITION BY C.CategoriaID ORDER BY COUNT(E.EmprestimoID) DESC) AS Ranking
    FROM Livros L
    INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
    LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
    GROUP BY C.CategoriaID, C.Nome, L.LivroID, L.Titulo
)
SELECT Categoria, Titulo, TotalEmprestimos, Ranking
FROM LivrosRanked
WHERE Ranking <= 3
ORDER BY Categoria, Ranking;
```

### RANK e DENSE_RANK

```sql
-- Diferença entre ROW_NUMBER, RANK e DENSE_RANK
SELECT
    L.Titulo,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    ROW_NUMBER() OVER (ORDER BY COUNT(E.EmprestimoID) DESC) AS RowNum,
    RANK() OVER (ORDER BY COUNT(E.EmprestimoID) DESC) AS Rank,
    DENSE_RANK() OVER (ORDER BY COUNT(E.EmprestimoID) DESC) AS DenseRank
FROM Livros L
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo
ORDER BY TotalEmprestimos DESC;

-- Ranking de membros por total de empréstimos
SELECT
    M.Nome,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    DENSE_RANK() OVER (ORDER BY COUNT(E.EmprestimoID) DESC) AS Posicao
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
GROUP BY M.MembroID, M.Nome
HAVING COUNT(E.EmprestimoID) > 0
ORDER BY Posicao;
```

### SUM, AVG, COUNT com OVER

```sql
-- Cumulativo: Total acumulado de empréstimos
SELECT
    DataEmprestimo,
    COUNT(*) AS EmprestimosDia,
    SUM(COUNT(*)) OVER (ORDER BY DataEmprestimo) AS TotalAcumulado
FROM Emprestimos
GROUP BY DataEmprestimo
ORDER BY DataEmprestimo;

-- Comparar com média do grupo
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    AVG(COUNT(E.EmprestimoID)) OVER (PARTITION BY C.CategoriaID) AS MediaCategoria,
    COUNT(E.EmprestimoID) - AVG(COUNT(E.EmprestimoID)) OVER (PARTITION BY C.CategoriaID) AS DiferencaDaMedia
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo, C.CategoriaID, C.Nome
ORDER BY C.Nome, TotalEmprestimos DESC;
```

## Padrões Comuns de Agregação

### Padrão 1: Relatório de Resumo

```sql
-- Dashboard da biblioteca
SELECT
    'Livros' AS Entidade,
    COUNT(*) AS Total,
    SUM(CASE WHEN NumCopiasDisponiveis > 0 THEN 1 ELSE 0 END) AS Disponiveis
FROM Livros
UNION ALL
SELECT
    'Membros',
    COUNT(*),
    SUM(CASE WHEN Ativo = 1 THEN 1 ELSE 0 END)
FROM Membros
UNION ALL
SELECT
    'Empréstimos Ativos',
    COUNT(*),
    SUM(CASE WHEN Status = 'Atrasado' THEN 1 ELSE 0 END)
FROM Emprestimos
WHERE Status = 'Ativo' OR Status = 'Atrasado';
```

### Padrão 2: Análise de Tendências

```sql
-- Tendência de empréstimos por mês
SELECT
    FORMAT(DataEmprestimo, 'yyyy-MM') AS AnoMes,
    COUNT(*) AS TotalEmprestimos,
    AVG(COUNT(*)) OVER (ORDER BY FORMAT(DataEmprestimo, 'yyyy-MM') ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS MediaMovel3Meses
FROM Emprestimos
GROUP BY FORMAT(DataEmprestimo, 'yyyy-MM')
ORDER BY AnoMes;
```

### Padrão 3: Análise de Distribuição

```sql
-- Distribuição de livros por faixa de ano
SELECT
    CASE
        WHEN AnoPublicacao < 1900 THEN 'Antes de 1900'
        WHEN AnoPublicacao BETWEEN 1900 AND 1950 THEN '1900-1950'
        WHEN AnoPublicacao BETWEEN 1951 AND 1980 THEN '1951-1980'
        WHEN AnoPublicacao BETWEEN 1981 AND 2000 THEN '1981-2000'
        ELSE 'Após 2000'
    END AS Periodo,
    COUNT(*) AS TotalLivros,
    CAST(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () AS DECIMAL(5,2)) AS Percentual
FROM Livros
GROUP BY
    CASE
        WHEN AnoPublicacao < 1900 THEN 'Antes de 1900'
        WHEN AnoPublicacao BETWEEN 1900 AND 1950 THEN '1900-1950'
        WHEN AnoPublicacao BETWEEN 1951 AND 1980 THEN '1951-1980'
        WHEN AnoPublicacao BETWEEN 1981 AND 2000 THEN '1981-2000'
        ELSE 'Após 2000'
    END
ORDER BY Periodo;
```

## Performance e Otimização

### Boas Práticas

```sql
-- ✅ BOM: Filtrar antes de agregar (WHERE)
SELECT
    C.Nome,
    COUNT(L.LivroID) AS Total
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
WHERE L.AnoPublicacao >= 2000
GROUP BY C.Nome;

-- ❌ EVITE: Filtrar depois de agregar sem necessidade
SELECT
    C.Nome,
    COUNT(L.LivroID) AS Total
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.Nome
HAVING COUNT(CASE WHEN L.AnoPublicacao >= 2000 THEN 1 END) > 0;

-- ✅ BOM: Usar índices em colunas de agrupamento
CREATE INDEX IX_Livros_CategoriaID ON Livros(CategoriaID);

-- ✅ BOM: Evitar funções em GROUP BY
SELECT
    YEAR(DataEmprestimo) AS Ano,
    COUNT(*) AS Total
FROM Emprestimos
GROUP BY YEAR(DataEmprestimo);  -- Pode impedir uso de índice

-- ✅ MELHOR: Considere coluna computada para queries frequentes
ALTER TABLE Emprestimos ADD AnoEmprestimo AS YEAR(DataEmprestimo) PERSISTED;
CREATE INDEX IX_Emprestimos_Ano ON Emprestimos(AnoEmprestimo);
```

## Exercícios Práticos

<procedure title="Exercícios para Praticar" id="exercicios-agregacao">
    <step>
        <p><strong>Exercício 1:</strong> Conte quantos livros existem em cada categoria</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Calcule a média de cópias por livro em cada categoria</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Liste as categorias com mais de 5 livros</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Encontre o livro mais antigo e o mais recente por categoria</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Conte quantos empréstimos cada membro fez e mostre apenas os que fizeram mais de 2</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Calcule o total de empréstimos por mês em 2024</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Liste os 5 livros mais emprestados com suas informações completas</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Crie um ranking dos autores por total de empréstimos de seus livros</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Calcule o percentual de livros disponíveis por categoria</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Use ROW_NUMBER para numerar os livros de cada categoria ordenados por ano</p>
    </step>
</procedure>

## Soluções dos Exercícios

<procedure title="Soluções Sugeridas" id="solucoes-agregacao" collapsible="true">
    <step>
        <p><strong>Solução 1:</strong> Livros por categoria</p>
        <code-block lang="sql">
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalLivros DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 2:</strong> Média de cópias</p>
        <code-block lang="sql">
SELECT
    C.Nome AS Categoria,
    AVG(CAST(L.NumCopias AS FLOAT)) AS MediaCopias,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY MediaCopias DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 3:</strong> Categorias com mais de 5 livros</p>
        <code-block lang="sql">
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
HAVING COUNT(L.LivroID) > 5
ORDER BY TotalLivros DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 4:</strong> Livro mais antigo e mais recente por categoria</p>
        <code-block lang="sql">
SELECT
    C.Nome AS Categoria,
    MIN(L.AnoPublicacao) AS MaisAntigo,
    MAX(L.AnoPublicacao) AS MaisRecente,
    MAX(L.AnoPublicacao) - MIN(L.AnoPublicacao) AS DiferencaAnos
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY C.Nome;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 7:</strong> Top 5 livros mais emprestados</p>
        <code-block lang="sql">
SELECT TOP 5
    L.Titulo,
    STRING_AGG(A.Nome, ', ') AS Autores,
    C.Nome AS Categoria,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo, C.Nome
ORDER BY TotalEmprestimos DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 9:</strong> Percentual de livros disponíveis</p>
        <code-block lang="sql">
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros,
    SUM(CASE WHEN L.NumCopiasDisponiveis > 0 THEN 1 ELSE 0 END) AS Disponiveis,
    CAST(SUM(CASE WHEN L.NumCopiasDisponiveis > 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(L.LivroID) AS DECIMAL(5,2)) AS PercentualDisponivel
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY PercentualDisponivel DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 10:</strong> ROW_NUMBER por categoria e ano</p>
        <code-block lang="sql">
SELECT
    C.Nome AS Categoria,
    L.Titulo,
    L.AnoPublicacao,
    ROW_NUMBER() OVER (PARTITION BY C.CategoriaID ORDER BY L.AnoPublicacao, L.Titulo) AS Numero
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = L.CategoriaID
ORDER BY C.Nome, Numero;
        </code-block>
    </step>
</procedure>

## Resumo da Aula

Nesta aula, você aprendeu:

- ✅ Funções agregadas básicas (COUNT, SUM, AVG, MIN, MAX)
- ✅ GROUP BY para agrupamento de dados
- ✅ HAVING para filtrar grupos
- ✅ Combinação de agregações com JOINs
- ✅ Funções agregadas avançadas (STRING_AGG, ROLLUP, CUBE)
- ✅ Window functions (ROW_NUMBER, RANK, PARTITION BY)
- ✅ Padrões comuns de análise de dados
- ✅ Otimização de consultas com agregações

## Glossary

**Função Agregada**
: Função que opera sobre múltiplas linhas e retorna um único valor

**GROUP BY**
: Cláusula que agrupa linhas com valores iguais em colunas especificadas

**HAVING**
: Filtra grupos após a agregação (diferente de WHERE que filtra antes)

**Window Function**
: Função que calcula sobre um conjunto de linhas mantendo o detalhe individual

**PARTITION BY**
: Divide o resultado em partições para cálculos de window functions

**ROW_NUMBER**
: Atribui um número sequencial único a cada linha

**RANK**
: Atribui ranking com gaps para empates

**DENSE_RANK**
: Atribui ranking sem gaps para empates

**STRING_AGG**
: Concatena valores de múltiplas linhas em uma string

**ROLLUP**
: Cria subtotais hierárquicos

**CUBE**
: Cria todas as combinações possíveis de agregações

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/functions/aggregate-functions-transact-sql">Microsoft Docs - Funções Agregadas</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/queries/select-group-by-transact-sql">Microsoft Docs - GROUP BY</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/queries/select-over-clause-transact-sql">Microsoft Docs - Window Functions</a>
        <a href="https://www.sqlshack.com/sql-aggregate-functions/">SQL Shack - Aggregate Functions</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique todos os tipos de agregações
> - Complete os exercícios propostos
> - Experimente criar relatórios complexos
> - Na próxima aula, aprenderemos sobre Subconsultas!
>
{style="tip"}
