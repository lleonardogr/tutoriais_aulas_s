# Aula 9: JOINs e Relacionamentos

## Introdução

JOINs são um dos conceitos mais importantes e poderosos em bancos de dados relacionais. Eles permitem combinar dados de múltiplas tabelas em uma única consulta, explorando os relacionamentos entre elas. Dominar JOINs é essencial para trabalhar efetivamente com SQL.

Nesta aula, você aprenderá:
- Tipos de JOINs (INNER, LEFT, RIGHT, FULL, CROSS)
- Como e quando usar cada tipo de JOIN
- Sintaxes diferentes de JOIN
- JOINs com múltiplas tabelas
- Self-joins e relacionamentos hierárquicos
- Performance e otimização de JOINs
- Padrões comuns e boas práticas

## Preparação: Banco de Dados

Vamos usar o banco BibliotecaDB criado na aula anterior. Se você não o tem, execute o script da Aula 8 primeiro.

```sql
USE BibliotecaDB;
GO

-- Verificar se as tabelas existem
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY TABLE_NAME;
```

## Conceitos Fundamentais de Relacionamentos

### Tipos de Relacionamentos

**1. Um-para-Muitos (1:N)**
- Mais comum
- Exemplo: Uma Categoria tem muitos Livros
- Um Livro pertence a uma Categoria

**2. Muitos-para-Muitos (N:N)**
- Requer tabela associativa
- Exemplo: Livros ↔ Autores
- Um Livro pode ter vários Autores
- Um Autor pode ter vários Livros

**3. Um-para-Um (1:1)**
- Menos comum
- Exemplo: Membro ↔ CarteiraDigital
- Geralmente usado para separar dados sensíveis ou grandes

```sql
-- Visualizar relacionamentos existentes
SELECT
    fk.name AS ForeignKey,
    tp.name AS ParentTable,
    cp.name AS ParentColumn,
    tr.name AS ReferencedTable,
    cr.name AS ReferencedColumn
FROM sys.foreign_keys fk
INNER JOIN sys.tables tp ON fk.parent_object_id = tp.object_id
INNER JOIN sys.tables tr ON fk.referenced_object_id = tr.object_id
INNER JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
INNER JOIN sys.columns cp ON fkc.parent_column_id = cp.column_id AND fkc.parent_object_id = cp.object_id
INNER JOIN sys.columns cr ON fkc.referenced_column_id = cr.column_id AND fkc.referenced_object_id = cr.object_id
ORDER BY tp.name, fk.name;
```

## INNER JOIN

O INNER JOIN retorna apenas registros que têm correspondência em **ambas** as tabelas.

### Sintaxe Básica

```sql
-- Sintaxe explícita (recomendada)
SELECT colunas
FROM Tabela1
INNER JOIN Tabela2 ON Tabela1.Coluna = Tabela2.Coluna;

-- Sintaxe antiga (evite)
SELECT colunas
FROM Tabela1, Tabela2
WHERE Tabela1.Coluna = Tabela2.Coluna;
```

### Exemplos Práticos de INNER JOIN

```sql
-- 1. Livros com suas categorias
SELECT
    L.Titulo,
    L.ISBN,
    C.Nome AS Categoria,
    L.AnoPublicacao
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
ORDER BY C.Nome, L.Titulo;

-- 2. Empréstimos com informações de membro e livro
SELECT
    E.EmprestimoID,
    M.Nome AS Membro,
    L.Titulo AS Livro,
    E.DataEmprestimo,
    E.Status
FROM Emprestimos E
INNER JOIN Membros M ON E.MembroID = M.MembroID
INNER JOIN Livros L ON E.LivroID = L.LivroID
ORDER BY E.DataEmprestimo DESC;

-- 3. Livros com seus autores (através de tabela associativa)
SELECT
    L.Titulo,
    A.Nome AS Autor,
    A.Nacionalidade,
    C.Nome AS Categoria
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
ORDER BY L.Titulo;

-- 4. INNER JOIN com filtros
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    A.Nome AS Autor
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
WHERE C.Nome = 'Ficção'
  AND L.AnoPublicacao >= 1900
ORDER BY L.AnoPublicacao DESC;
```

### INNER JOIN com Alias

```sql
-- Usar alias torna consultas mais legíveis
SELECT
    l.Titulo AS 'Título do Livro',
    c.Nome AS 'Categoria',
    l.NumCopiasDisponiveis AS 'Disponíveis'
FROM Livros l
INNER JOIN Categorias c ON l.CategoriaID = c.CategoriaID
WHERE l.NumCopiasDisponiveis > 0;
```

## LEFT JOIN (LEFT OUTER JOIN)

O LEFT JOIN retorna **todos** os registros da tabela esquerda e os correspondentes da direita. Se não houver correspondência, retorna NULL para as colunas da tabela direita.

### Quando Usar LEFT JOIN

- Quando você quer todos os registros da tabela principal
- Para encontrar registros sem correspondência (órfãos)
- Para incluir informações opcionais

### Exemplos Práticos de LEFT JOIN

```sql
-- 1. Todos os livros, com ou sem empréstimos
SELECT
    L.Titulo,
    L.NumCopias,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Livros L
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo, L.NumCopias
ORDER BY TotalEmprestimos DESC;

-- 2. Livros que NUNCA foram emprestados
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    L.AnoPublicacao
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
WHERE E.EmprestimoID IS NULL
ORDER BY L.Titulo;

-- 3. Todos os membros e seus empréstimos (mesmo sem empréstimos)
SELECT
    M.Nome AS Membro,
    M.Email,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    SUM(CASE WHEN E.Status = 'Ativo' THEN 1 ELSE 0 END) AS EmprestimosAtivos
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
GROUP BY M.MembroID, M.Nome, M.Email
ORDER BY TotalEmprestimos DESC;

-- 4. Categorias com contagem de livros (incluindo categorias vazias)
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalLivros DESC;

-- 5. Autores e seus livros (incluindo autores sem livros no acervo)
SELECT
    A.Nome AS Autor,
    A.Nacionalidade,
    STRING_AGG(L.Titulo, ', ') AS Livros
FROM Autores A
LEFT JOIN LivroAutor LA ON A.AutorID = LA.AutorID
LEFT JOIN Livros L ON LA.LivroID = L.LivroID
GROUP BY A.AutorID, A.Nome, A.Nacionalidade
ORDER BY A.Nome;
```

### Diferença Visual: INNER vs LEFT JOIN

```sql
-- Comparação direta
-- INNER JOIN: Apenas livros que foram emprestados
SELECT L.Titulo, COUNT(E.EmprestimoID) AS Emprestimos
FROM Livros L
INNER JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo;

-- LEFT JOIN: Todos os livros, tenham sido emprestados ou não
SELECT L.Titulo, COUNT(E.EmprestimoID) AS Emprestimos
FROM Livros L
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo;
```

## RIGHT JOIN (RIGHT OUTER JOIN)

O RIGHT JOIN é o oposto do LEFT JOIN: retorna **todos** os registros da tabela direita e os correspondentes da esquerda.

> **Nota:** RIGHT JOIN é menos usado. Na maioria dos casos, você pode reescrever usando LEFT JOIN invertendo a ordem das tabelas.

```sql
-- RIGHT JOIN
SELECT
    L.Titulo,
    C.Nome AS Categoria
FROM Livros L
RIGHT JOIN Categorias C ON L.CategoriaID = C.CategoriaID;

-- Equivalente com LEFT JOIN (preferível)
SELECT
    L.Titulo,
    C.Nome AS Categoria
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID;
```

### Exemplo Prático

```sql
-- Todas as categorias, mesmo sem livros (RIGHT JOIN)
SELECT
    C.Nome AS Categoria,
    COUNT(L.LivroID) AS TotalLivros
FROM Livros L
RIGHT JOIN Categorias C ON L.CategoriaID = C.CategoriaID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalLivros DESC;
```

## FULL OUTER JOIN

Retorna **todos** os registros de ambas as tabelas, com NULL onde não há correspondência.

```sql
-- FULL OUTER JOIN (menos comum)
SELECT
    M.Nome AS Membro,
    L.Titulo AS Livro,
    E.DataEmprestimo
FROM Membros M
FULL OUTER JOIN Emprestimos E ON M.MembroID = E.MembroID
FULL OUTER JOIN Livros L ON E.LivroID = L.LivroID
ORDER BY M.Nome, E.DataEmprestimo;
```

### Quando Usar FULL OUTER JOIN

- Reconciliação de dados de duas fontes
- Encontrar diferenças entre dois conjuntos de dados
- Relatórios que precisam mostrar tudo de ambas as tabelas

## CROSS JOIN

Retorna o produto cartesiano: cada linha da tabela 1 combinada com cada linha da tabela 2.

⚠️ **Cuidado:** Pode gerar muitos registros! (Tabela1.linhas × Tabela2.linhas)

```sql
-- CROSS JOIN: Todas as combinações possíveis
SELECT
    C.Nome AS Categoria,
    A.Nome AS Autor
FROM Categorias C
CROSS JOIN Autores A
ORDER BY C.Nome, A.Nome;

-- Útil para gerar combinações ou matrizes
-- Exemplo: Calendário de disponibilidade
DECLARE @DataInicio DATE = '2024-01-01';
DECLARE @DataFim DATE = '2024-01-31';

WITH Datas AS (
    SELECT @DataInicio AS Data
    UNION ALL
    SELECT DATEADD(DAY, 1, Data)
    FROM Datas
    WHERE Data < @DataFim
)
SELECT
    D.Data,
    M.Nome AS Membro
FROM Datas D
CROSS JOIN Membros M
WHERE M.Ativo = 1
ORDER BY D.Data, M.Nome
OPTION (MAXRECURSION 31);
```

## JOINs com Múltiplas Tabelas

### Cadeia de JOINs

```sql
-- Consulta completa: Empréstimos com todas as informações
SELECT
    E.EmprestimoID,
    M.Nome AS Membro,
    M.Email,
    L.Titulo AS Livro,
    A.Nome AS Autor,
    C.Nome AS Categoria,
    E.DataEmprestimo,
    E.DataDevolucaoPrevista,
    E.Status,
    CASE
        WHEN E.Status = 'Ativo' THEN DATEDIFF(DAY, GETDATE(), E.DataDevolucaoPrevista)
        ELSE NULL
    END AS DiasRestantes
FROM Emprestimos E
INNER JOIN Membros M ON E.MembroID = M.MembroID
INNER JOIN Livros L ON E.LivroID = L.LivroID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
WHERE E.Status IN ('Ativo', 'Atrasado')
ORDER BY E.DataDevolucaoPrevista;
```

### Misturando Tipos de JOIN

```sql
-- INNER + LEFT JOIN: Todos os livros com suas categorias,
-- incluindo informações de empréstimo se existirem
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    L.NumCopiasDisponiveis AS Disponiveis,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    SUM(CASE WHEN E.Status = 'Ativo' THEN 1 ELSE 0 END) AS EmprestimosAtivos
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo, C.Nome, L.NumCopiasDisponiveis
HAVING L.NumCopiasDisponiveis < 2 OR COUNT(E.EmprestimoID) > 3
ORDER BY TotalEmprestimos DESC;
```

## SELF JOIN

Um self-join é quando uma tabela faz join consigo mesma. Útil para relacionamentos hierárquicos ou comparações dentro da mesma tabela.

### Exemplo: Adicionar Estrutura Hierárquica

```sql
-- Adicionar uma coluna para hierarquia de categorias
ALTER TABLE Categorias ADD CategoriaPaiID INT NULL;
ALTER TABLE Categorias ADD FOREIGN KEY (CategoriaPaiID) REFERENCES Categorias(CategoriaID);

-- Criar subcategorias
UPDATE Categorias SET CategoriaPaiID = NULL WHERE Nome IN ('Ficção', 'Não-ficção');
UPDATE Categorias SET CategoriaPaiID = 1 WHERE Nome IN ('Romance', 'Suspense', 'Ficção Científica', 'Fantasia');
UPDATE Categorias SET CategoriaPaiID = 2 WHERE Nome IN ('Biografia', 'História', 'Tecnologia', 'Autoajuda');

-- Self-join para mostrar hierarquia
SELECT
    C1.Nome AS Categoria,
    C2.Nome AS CategoriaPai
FROM Categorias C1
LEFT JOIN Categorias C2 ON C1.CategoriaPaiID = C2.CategoriaID
ORDER BY C2.Nome, C1.Nome;

-- Hierarquia completa com níveis
SELECT
    C1.Nome AS Categoria,
    COALESCE(C2.Nome, 'Raiz') AS CategoriaPai,
    CASE
        WHEN C1.CategoriaPaiID IS NULL THEN 0
        ELSE 1
    END AS Nivel
FROM Categorias C1
LEFT JOIN Categorias C2 ON C1.CategoriaPaiID = C2.CategoriaID
ORDER BY Nivel, CategoriaPai, C1.Nome;
```

### Outro Exemplo: Encontrar Membros com Empréstimos Similares

```sql
-- Membros que emprestaram os mesmos livros
SELECT DISTINCT
    M1.Nome AS Membro1,
    M2.Nome AS Membro2,
    L.Titulo AS LivroComum
FROM Emprestimos E1
INNER JOIN Emprestimos E2 ON E1.LivroID = E2.LivroID AND E1.MembroID < E2.MembroID
INNER JOIN Membros M1 ON E1.MembroID = M1.MembroID
INNER JOIN Membros M2 ON E2.MembroID = M2.MembroID
INNER JOIN Livros L ON E1.LivroID = L.LivroID
ORDER BY L.Titulo, M1.Nome;
```

## Condições de JOIN Complexas

### JOIN com Múltiplas Condições

```sql
-- JOIN com múltiplas condições
SELECT
    L.Titulo,
    M.Nome AS Membro,
    E.DataEmprestimo
FROM Livros L
INNER JOIN Emprestimos E ON L.LivroID = E.LivroID
    AND E.Status = 'Ativo'
    AND E.DataEmprestimo >= '2024-10-01'
INNER JOIN Membros M ON E.MembroID = M.MembroID
    AND M.Ativo = 1
ORDER BY E.DataEmprestimo DESC;
```

### JOIN com Intervalos

```sql
-- Empréstimos que coincidiram no tempo (mesmo livro emprestado simultaneamente - erro!)
SELECT
    E1.EmprestimoID AS Emprestimo1,
    E2.EmprestimoID AS Emprestimo2,
    L.Titulo,
    E1.DataEmprestimo AS Data1,
    E2.DataEmprestimo AS Data2
FROM Emprestimos E1
INNER JOIN Emprestimos E2 ON E1.LivroID = E2.LivroID
    AND E1.EmprestimoID < E2.EmprestimoID
    AND E1.DataEmprestimo <= ISNULL(E2.DataDevolucaoReal, GETDATE())
    AND E2.DataEmprestimo <= ISNULL(E1.DataDevolucaoReal, GETDATE())
INNER JOIN Livros L ON E1.LivroID = L.LivroID
ORDER BY L.Titulo;
```

## Padrões Comuns de JOIN

### Padrão 1: Lista Mestre-Detalhe

```sql
-- Membros com seus empréstimos detalhados
SELECT
    M.Nome AS Membro,
    M.Email,
    L.Titulo AS Livro,
    E.DataEmprestimo,
    E.DataDevolucaoPrevista,
    E.Status
FROM Membros M
INNER JOIN Emprestimos E ON M.MembroID = E.MembroID
INNER JOIN Livros L ON E.LivroID = L.LivroID
WHERE M.MembroID = 1
ORDER BY E.DataEmprestimo DESC;
```

### Padrão 2: Agregação com JOIN

```sql
-- Estatísticas por categoria
SELECT
    C.Nome AS Categoria,
    COUNT(DISTINCT L.LivroID) AS TotalLivros,
    COUNT(DISTINCT LA.AutorID) AS TotalAutores,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    AVG(CAST(L.NumCopiasDisponiveis AS FLOAT) / NULLIF(L.NumCopias, 0)) AS PercentualDisponivel
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
LEFT JOIN LivroAutor LA ON L.LivroID = LA.LivroID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalEmprestimos DESC;
```

### Padrão 3: Encontrar Registros Órfãos

```sql
-- Verificar integridade: Livros sem categoria (não deve acontecer com FK)
SELECT L.*
FROM Livros L
LEFT JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE C.CategoriaID IS NULL;

-- Membros sem nenhum empréstimo
SELECT
    M.Nome,
    M.Email,
    M.DataCadastro,
    DATEDIFF(DAY, M.DataCadastro, GETDATE()) AS DiasCadastrado
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
WHERE E.EmprestimoID IS NULL
ORDER BY M.DataCadastro;
```

### Padrão 4: Top N por Grupo

```sql
-- Top 3 livros mais emprestados por categoria
WITH EmprestimosPorLivro AS (
    SELECT
        L.LivroID,
        L.Titulo,
        C.Nome AS Categoria,
        COUNT(E.EmprestimoID) AS TotalEmprestimos,
        ROW_NUMBER() OVER (PARTITION BY C.CategoriaID ORDER BY COUNT(E.EmprestimoID) DESC) AS Ranking
    FROM Livros L
    INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
    LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
    GROUP BY L.LivroID, L.Titulo, C.CategoriaID, C.Nome
)
SELECT Categoria, Titulo, TotalEmprestimos, Ranking
FROM EmprestimosPorLivro
WHERE Ranking <= 3
ORDER BY Categoria, Ranking;
```

## Performance e Otimização de JOINs

### Dicas de Performance

```sql
-- ✅ BOM: Filtrar antes do JOIN (WHERE na subquery)
SELECT
    M.Nome,
    RecentesEmprestimos.Total
FROM Membros M
INNER JOIN (
    SELECT MembroID, COUNT(*) AS Total
    FROM Emprestimos
    WHERE DataEmprestimo >= '2024-10-01'
    GROUP BY MembroID
) AS RecentesEmprestimos ON M.MembroID = RecentesEmprestimos.MembroID
WHERE M.Ativo = 1;

-- ❌ EVITE: Funções em condições de JOIN
SELECT L.Titulo, C.Nome
FROM Livros L
INNER JOIN Categorias C ON LOWER(L.Titulo) LIKE LOWER(C.Nome) + '%';

-- ✅ MELHOR: Usar colunas indexadas diretamente
SELECT L.Titulo, C.Nome
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID;
```

### Analisar Plano de Execução

```sql
-- Ativar estatísticas
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Sua consulta aqui
SELECT
    L.Titulo,
    C.Nome AS Categoria,
    COUNT(E.EmprestimoID) AS Total
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo, C.Nome;

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;

-- Ver plano de execução estimado: Ctrl+L (SSMS)
-- Ver plano de execução real: Ctrl+M antes de executar
```

### Índices para JOINs

```sql
-- Ver índices existentes
EXEC sp_helpindex 'Emprestimos';

-- Criar índice para melhorar JOINs (exemplo)
CREATE NONCLUSTERED INDEX IX_Emprestimos_MembroID
ON Emprestimos(MembroID)
INCLUDE (LivroID, DataEmprestimo, Status);

-- Índice composto para múltiplas colunas de JOIN
CREATE NONCLUSTERED INDEX IX_Emprestimos_LivroMembro
ON Emprestimos(LivroID, MembroID)
INCLUDE (DataEmprestimo, Status);
```

## Exercícios Práticos

<procedure title="Exercícios para Praticar" id="exercicios-joins">
    <step>
        <p><strong>Exercício 1:</strong> Liste todos os livros com seus autores e categorias (use INNER JOIN)</p>
    </step>
    <step>
        <p><strong>Exercício 2:</strong> Encontre todos os membros que nunca fizeram um empréstimo (use LEFT JOIN)</p>
    </step>
    <step>
        <p><strong>Exercício 3:</strong> Mostre o total de empréstimos por categoria, incluindo categorias sem empréstimos</p>
    </step>
    <step>
        <p><strong>Exercício 4:</strong> Liste os autores brasileiros e seus livros disponíveis na biblioteca</p>
    </step>
    <step>
        <p><strong>Exercício 5:</strong> Crie uma consulta que mostre empréstimos atrasados com informações completas do membro, livro e autor</p>
    </step>
    <step>
        <p><strong>Exercício 6:</strong> Encontre pares de livros do mesmo autor (self-join em LivroAutor)</p>
    </step>
    <step>
        <p><strong>Exercício 7:</strong> Liste categorias que têm pelo menos 3 livros no acervo</p>
    </step>
    <step>
        <p><strong>Exercício 8:</strong> Mostre membros com mais de 2 empréstimos ativos</p>
    </step>
    <step>
        <p><strong>Exercício 9:</strong> Crie um relatório de livros: título, autor, categoria, total de empréstimos e cópias disponíveis</p>
    </step>
    <step>
        <p><strong>Exercício 10:</strong> Liste todos os livros emprestados em outubro de 2024 com informações do membro</p>
    </step>
</procedure>

## Soluções dos Exercícios

<procedure title="Soluções Sugeridas" id="solucoes-joins" collapsible="true">
    <step>
        <p><strong>Solução 1:</strong> Livros com autores e categorias</p>
        <code-block lang="sql">
SELECT
    L.Titulo,
    A.Nome AS Autor,
    C.Nome AS Categoria,
    L.AnoPublicacao
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
ORDER BY L.Titulo;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 2:</strong> Membros sem empréstimos</p>
        <code-block lang="sql">
SELECT
    M.Nome,
    M.Email,
    M.DataCadastro
FROM Membros M
LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
WHERE E.EmprestimoID IS NULL;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 3:</strong> Total de empréstimos por categoria</p>
        <code-block lang="sql">
SELECT
    C.Nome AS Categoria,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Categorias C
LEFT JOIN Livros L ON C.CategoriaID = L.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalEmprestimos DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 5:</strong> Empréstimos atrasados detalhados</p>
        <code-block lang="sql">
SELECT
    M.Nome AS Membro,
    M.Email,
    M.Telefone,
    L.Titulo AS Livro,
    A.Nome AS Autor,
    E.DataEmprestimo,
    E.DataDevolucaoPrevista,
    DATEDIFF(DAY, E.DataDevolucaoPrevista, GETDATE()) AS DiasAtraso
FROM Emprestimos E
INNER JOIN Membros M ON E.MembroID = M.MembroID
INNER JOIN Livros L ON E.LivroID = L.LivroID
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
WHERE E.Status = 'Atrasado'
ORDER BY DiasAtraso DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 6:</strong> Pares de livros do mesmo autor</p>
        <code-block lang="sql">
SELECT
    A.Nome AS Autor,
    L1.Titulo AS Livro1,
    L2.Titulo AS Livro2
FROM Autores A
INNER JOIN LivroAutor LA1 ON A.AutorID = LA1.AutorID
INNER JOIN LivroAutor LA2 ON A.AutorID = LA2.AutorID AND LA1.LivroID < LA2.LivroID
INNER JOIN Livros L1 ON LA1.LivroID = L1.LivroID
INNER JOIN Livros L2 ON LA2.LivroID = L2.LivroID
ORDER BY A.Nome, L1.Titulo;
        </code-block>
    </step>
    <step>
        <p><strong>Solução 9:</strong> Relatório completo de livros</p>
        <code-block lang="sql">
SELECT
    L.Titulo,
    STRING_AGG(A.Nome, ', ') AS Autores,
    C.Nome AS Categoria,
    COUNT(E.EmprestimoID) AS TotalEmprestimos,
    L.NumCopiasDisponiveis AS Disponiveis
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo, C.Nome, L.NumCopiasDisponiveis
ORDER BY TotalEmprestimos DESC;
        </code-block>
    </step>
</procedure>

## Boas Práticas com JOINs

### 1. Sempre Use Alias para Tabelas

```sql
-- ✅ Bom: Com alias
SELECT l.Titulo, c.Nome
FROM Livros l
INNER JOIN Categorias c ON l.CategoriaID = c.CategoriaID;

-- ❌ Ruim: Sem alias
SELECT Livros.Titulo, Categorias.Nome
FROM Livros
INNER JOIN Categorias ON Livros.CategoriaID = Categorias.CategoriaID;
```

### 2. Use INNER JOIN Explícito

```sql
-- ✅ Bom: INNER JOIN explícito
SELECT l.Titulo, c.Nome
FROM Livros l
INNER JOIN Categorias c ON l.CategoriaID = c.CategoriaID;

-- ❌ Evite: Sintaxe antiga
SELECT l.Titulo, c.Nome
FROM Livros l, Categorias c
WHERE l.CategoriaID = c.CategoriaID;
```

### 3. Qualifique Todas as Colunas

```sql
-- ✅ Bom: Colunas qualificadas
SELECT
    l.Titulo,
    c.Nome AS Categoria,
    l.AnoPublicacao
FROM Livros l
INNER JOIN Categorias c ON l.CategoriaID = c.CategoriaID;

-- ❌ Evite: Colunas ambíguas
SELECT Titulo, Nome, AnoPublicacao  -- Qual tabela?
FROM Livros l
INNER JOIN Categorias c ON l.CategoriaID = c.CategoriaID;
```

### 4. Formate para Legibilidade

```sql
-- ✅ Bem formatado
SELECT
    m.Nome AS Membro,
    l.Titulo AS Livro,
    e.DataEmprestimo
FROM Emprestimos e
INNER JOIN Membros m ON e.MembroID = m.MembroID
INNER JOIN Livros l ON e.LivroID = l.LivroID
WHERE e.Status = 'Ativo'
ORDER BY e.DataEmprestimo DESC;
```

### 5. Considere a Ordem dos JOINs

```sql
-- Melhor performance: comece com a tabela menor ou mais filtrada
SELECT l.Titulo, e.DataEmprestimo
FROM (
    SELECT * FROM Emprestimos WHERE Status = 'Ativo'
) e
INNER JOIN Livros l ON e.LivroID = l.LivroID;
```

## Resumo da Aula

Nesta aula, você aprendeu:

- ✅ Tipos de JOINs (INNER, LEFT, RIGHT, FULL, CROSS)
- ✅ Quando usar cada tipo de JOIN
- ✅ JOINs com múltiplas tabelas
- ✅ Self-joins para relacionamentos hierárquicos
- ✅ Padrões comuns de consultas com JOIN
- ✅ Otimização e performance de JOINs
- ✅ Boas práticas de escrita de JOINs

## Glossary

**INNER JOIN**
: Retorna apenas registros com correspondência em ambas as tabelas

**LEFT JOIN (LEFT OUTER JOIN)**
: Retorna todos os registros da tabela esquerda e correspondentes da direita

**RIGHT JOIN (RIGHT OUTER JOIN)**
: Retorna todos os registros da tabela direita e correspondentes da esquerda

**FULL OUTER JOIN**
: Retorna todos os registros de ambas as tabelas

**CROSS JOIN**
: Retorna o produto cartesiano de duas tabelas

**Self-Join**
: Join de uma tabela consigo mesma

**Tabela Associativa**
: Tabela que implementa relacionamento muitos-para-muitos

**Produto Cartesiano**
: Combinação de cada linha de uma tabela com cada linha de outra

**Plano de Execução**
: Representação visual de como o SQL Server executa uma consulta

**Índice**
: Estrutura que acelera buscas e JOINs em colunas específicas

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/relational-databases/performance/joins">Microsoft Docs - JOINs</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/queries/from-transact-sql">Microsoft Docs - FROM Clause</a>
        <a href="https://www.sqlshack.com/learn-sql-join-multiple-tables/">SQL Shack - Joining Multiple Tables</a>
        <a href="https://www.red-gate.com/simple-talk/databases/sql-server/t-sql-programming-sql-server/sql-server-join-types-poster/">Red Gate - JOIN Types Poster</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique todos os tipos de JOINs com o banco BibliotecaDB
> - Complete os exercícios propostos
> - Experimente criar consultas com 4+ tabelas
> - Na próxima aula, aprenderemos sobre Funções Agregadas e GROUP BY!
>
{style="tip"}
