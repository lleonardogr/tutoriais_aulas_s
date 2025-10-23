# Aula 8: Projeto Prático I - Banco de Dados Simples

## Introdução

Chegou a hora de colocar em prática tudo que você aprendeu até agora! Neste projeto, você criará um banco de dados completo para um sistema de **Biblioteca**, aplicando conceitos de modelagem, criação de tabelas, inserção de dados, consultas e manipulação de informações.

Este projeto consolida os seguintes conceitos:
- Modelagem de dados (conceitual, lógica e física)
- Criação de banco de dados e tabelas
- Tipos de dados apropriados
- Chaves primárias e estrangeiras
- Inserção, atualização e exclusão de dados
- Consultas com WHERE e operadores avançados
- Boas práticas de SQL

## Objetivos do Projeto

Ao final desta aula, você será capaz de:

1. ✅ Modelar um banco de dados a partir de requisitos
2. ✅ Criar estruturas completas com relacionamentos
3. ✅ Popular tabelas com dados realistas
4. ✅ Executar consultas complexas para análise
5. ✅ Manipular dados com segurança usando transações

## Cenário: Sistema de Biblioteca

Você foi contratado para criar um banco de dados para uma biblioteca municipal. O sistema deve gerenciar:

- **Livros**: Catálogo de livros disponíveis
- **Autores**: Informações sobre os autores
- **Membros**: Pessoas cadastradas na biblioteca
- **Empréstimos**: Controle de empréstimos e devoluções
- **Categorias**: Classificação dos livros por gênero

### Requisitos do Sistema

#### Requisitos Funcionais

1. A biblioteca deve poder cadastrar livros com informações como título, ISBN, ano de publicação, número de cópias
2. Cada livro pertence a uma categoria (Ficção, Não-ficção, Romance, etc.)
3. Livros podem ter um ou mais autores
4. Membros da biblioteca devem ser cadastrados com nome, email, telefone e endereço
5. O sistema deve registrar empréstimos, incluindo data de empréstimo e data de devolução prevista
6. Empréstimos devem ter status (Ativo, Devolvido, Atrasado)
7. Um membro pode ter múltiplos empréstimos, mas não pode emprestar o mesmo livro duas vezes simultaneamente

#### Regras de Negócio

- Prazo de empréstimo: 14 dias
- Um membro pode ter no máximo 5 empréstimos ativos
- Livros só podem ser emprestados se houver cópias disponíveis
- Membros com empréstimos atrasados não podem fazer novos empréstimos

## Fase 1: Modelagem

### Modelo Conceitual (Entidades e Relacionamentos)

Identifique as entidades principais e seus relacionamentos:

```
CATEGORIAS (1) ----< (N) LIVROS (N) >----< (N) AUTORES
                            |
                            | (N)
                            |
                            v
                      EMPRÉSTIMOS (N) >---- (1) MEMBROS
```

**Entidades:**
- **Categorias**: Classificação dos livros
- **Livros**: Catálogo de livros
- **Autores**: Escritores dos livros
- **Membros**: Usuários da biblioteca
- **Empréstimos**: Registros de empréstimos
- **LivroAutor**: Tabela associativa (relacionamento N:N entre Livros e Autores)

### Modelo Lógico (Atributos)

**Categorias**
- CategoriaID (PK)
- Nome
- Descrição

**Livros**
- LivroID (PK)
- Titulo
- ISBN
- AnoPublicacao
- NumCopias
- NumCopiasDisponiveis
- CategoriaID (FK)

**Autores**
- AutorID (PK)
- Nome
- DataNascimento
- Nacionalidade
- Biografia

**LivroAutor** (tabela associativa)
- LivroID (FK)
- AutorID (FK)
- (PK composta: LivroID + AutorID)

**Membros**
- MembroID (PK)
- Nome
- Email
- Telefone
- Endereco
- DataCadastro
- Ativo

**Empréstimos**
- EmprestimoID (PK)
- LivroID (FK)
- MembroID (FK)
- DataEmprestimo
- DataDevolucaoPrevista
- DataDevolucaoReal
- Status

## Fase 2: Criação do Banco de Dados

Vamos começar criando o banco de dados e as tabelas:

<procedure title="Criação da Estrutura" id="criar-estrutura-biblioteca">
    <step>
        <p>Abra o SQL Server Management Studio (SSMS)</p>
    </step>
    <step>
        <p>Crie o banco de dados</p>
    </step>
    <step>
        <code-block lang="sql">
            -- Criar banco de dados
            CREATE DATABASE BibliotecaDB;
            GO
            USE BibliotecaDB;
            GO
        </code-block>
    </step>
    <step>
        <p>Crie as tabelas na ordem correta (tabelas sem dependências primeiro)</p>
    </step>
    <step>
    <code-block lang="sql">
            -- Tabela Categorias
            CREATE TABLE Categorias (
                CategoriaID INT PRIMARY KEY IDENTITY(1,1),
                Nome NVARCHAR(50) NOT NULL UNIQUE,
                Descricao NVARCHAR(200)
            );
            -- Tabela Autores
            CREATE TABLE Autores (
                AutorID INT PRIMARY KEY IDENTITY(1,1),
                Nome NVARCHAR(100) NOT NULL,
                DataNascimento DATE,
                Nacionalidade NVARCHAR(50),
                Biografia NVARCHAR(MAX)
            );
            -- Tabela Membros
            CREATE TABLE Membros (
                MembroID INT PRIMARY KEY IDENTITY(1,1),
                Nome NVARCHAR(100) NOT NULL,
                Email VARCHAR(100) UNIQUE,
                Telefone VARCHAR(20),
                Endereco NVARCHAR(200),
                DataCadastro DATE DEFAULT GETDATE(),
                Ativo BIT DEFAULT 1
            );
            -- Tabela Livros
            CREATE TABLE Livros (
                LivroID INT PRIMARY KEY IDENTITY(1,1),
                Titulo NVARCHAR(200) NOT NULL,
                ISBN VARCHAR(20) UNIQUE,
                AnoPublicacao INT,
                NumCopias INT DEFAULT 1 CHECK (NumCopias >= 0),
                NumCopiasDisponiveis INT DEFAULT 1 CHECK (NumCopiasDisponiveis >= 0),
                CategoriaID INT NOT NULL,
                FOREIGN KEY (CategoriaID) REFERENCES Categorias(CategoriaID),
                CONSTRAINT CK_Livros_CopiasDisponiveis CHECK (NumCopiasDisponiveis = NumCopias)
            );
            -- Tabela associativa LivroAutor
            CREATE TABLE LivroAutor (
                LivroID INT NOT NULL,
                AutorID INT NOT NULL,
                PRIMARY KEY (LivroID, AutorID),
                FOREIGN KEY (LivroID) REFERENCES Livros(LivroID) ON DELETE CASCADE,
                FOREIGN KEY (AutorID) REFERENCES Autores(AutorID) ON DELETE CASCADE
            );
            -- Tabela Empréstimos
            CREATE TABLE Emprestimos (
                EmprestimoID INT PRIMARY KEY IDENTITY(1,1),
                LivroID INT NOT NULL,
                MembroID INT NOT NULL,
                DataEmprestimo DATE DEFAULT GETDATE(),
                DataDevolucaoPrevista DATE NOT NULL,
                DataDevolucaoReal DATE,
                Status NVARCHAR(20) DEFAULT 'Ativo' CHECK (Status IN ('Ativo', 'Devolvido', 'Atrasado')),
                FOREIGN KEY (LivroID) REFERENCES Livros(LivroID),
                FOREIGN KEY (MembroID) REFERENCES Membros(MembroID)
            );
            PRINT 'Estrutura do banco de dados criada com sucesso!';
        </code-block>
    </step>
    <step>
        <p>Verifique as tabelas criadas</p>
    </step>
    <step>
        <code-block lang="sql">
        -- Ver todas as tabelas
        SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
        WHERE TABLE_TYPE = 'BASE TABLE'
        ORDER BY TABLE_NAME;
        -- Ver estrutura de uma tabela específica
        EXEC sp_help 'Livros';
        </code-block>
    </step>
</procedure>

## Fase 3: Inserção de Dados

Agora vamos popular o banco com dados realistas:

<procedure title="Popular o Banco de Dados" id="popular-dados">
    <step>
        <p>Inserir Categorias</p>
    </step>
    <step>
        <code-block lang="sql">
        INSERT INTO Categorias (Nome, Descricao) VALUES
            ('Ficção', 'Obras de ficção literária'),
            ('Não-ficção', 'Obras baseadas em fatos reais'),
            ('Romance', 'Histórias românticas'),
            ('Suspense', 'Thrillers e mistérios'),
            ('Ficção Científica', 'Obras de sci-fi'),
            ('Fantasia', 'Mundos fantásticos e magia'),
            ('Biografia', 'Histórias de vida reais'),
            ('História', 'Livros sobre história'),
            ('Tecnologia', 'Livros técnicos e de TI'),
            ('Autoajuda', 'Desenvolvimento pessoal');
        SELECT * FROM Categorias;
        </code-block>
        </step>
            <step>
                <p>Inserir Autores</p>
            </step>
        <step>
            <code-block lang="sql">
            INSERT INTO Autores (Nome, DataNascimento, Nacionalidade, Biografia) VALUES
            ('Machado de Assis', '1839-06-21', 'Brasileira', 'Maior escritor brasileiro, fundador da ABL'),
            ('Clarice Lispector', '1920-12-10', 'Brasileira', 'Escritora e jornalista ucraniana naturalizada brasileira'),
            ('J.K. Rowling', '1965-07-31', 'Britânica', 'Autora da série Harry Potter'),
            ('George Orwell', '1903-06-25', 'Britânica', 'Autor de 1984 e A Revolução dos Bichos'),
            ('Isaac Asimov', '1920-01-02', 'Americana', 'Mestre da ficção científica'),
            ('Agatha Christie', '1890-09-15', 'Britânica', 'Rainha do crime'),
            ('Stephen King', '1947-09-21', 'Americana', 'Mestre do terror'),
            ('Neil Gaiman', '1960-11-10', 'Britânica', 'Autor de fantasia moderna'),
            ('Paulo Coelho', '1947-08-24', 'Brasileira', 'Autor de O Alquimista'),
            ('Yuval Noah Harari', '1976-02-24', 'Israelense', 'Historiador e autor de Sapiens');
            SELECT * FROM Autores;
        </code-block>
    </step>
    <step>
        <p>Inserir Membros</p>
    </step>
    <step>
        <code-block lang="sql">
        INSERT INTO Membros (Nome, Email, Telefone, Endereco) VALUES
            ('Ana Silva', 'ana.silva@email.com', '(11) 98765-4321', 'Rua das Flores, 123 - São Paulo'),
            ('Bruno Costa', 'bruno.costa@email.com', '(21) 97654-3210', 'Av. Atlântica, 456 - Rio de Janeiro'),
            ('Carla Souza', 'carla.souza@email.com', '(31) 96543-2109', 'Rua Minas, 789 - Belo Horizonte'),
            ('Daniel Pereira', 'daniel.p@email.com', '(41) 95432-1098', 'Rua Paraná, 321 - Curitiba'),
            ('Elisa Rodrigues', 'elisa.r@email.com', '(51) 94321-0987', 'Av. Borges, 654 - Porto Alegre'),
            ('Fernando Lima', 'fernando.lima@email.com', '(61) 93210-9876', 'SQN 308 - Brasília'),
            ('Gabriela Santos', 'gabi.santos@email.com', '(71) 92109-8765', 'Rua Bahia, 987 - Salvador'),
            ('Henrique Alves', 'henrique.a@email.com', '(81) 91098-7654', 'Av. Recife, 147 - Recife'),
            ('Isabela Martins', 'isa.martins@email.com', '(85) 90987-6543', 'Rua Ceará, 258 - Fortaleza'),
            ('João Oliveira', 'joao.oli@email.com', '(92) 89876-5432', 'Av. Amazonas, 369 - Manaus');
        SELECT * FROM Membros;
        </code-block>
    </step>
    <step>
        <p>Inserir Livros</p>
    </step>
    <step>
        <code-block lang="sql">
        INSERT INTO Livros (Titulo, ISBN, AnoPublicacao, NumCopias, NumCopiasDisponiveis, CategoriaID) VALUES
            ('Dom Casmurro', '978-8535911664', 1899, 3, 3, 1),
            ('A Hora da Estrela', '978-8520925683', 1977, 2, 2, 1),
            ('Harry Potter e a Pedra Filosofal', '978-8532530788', 1997, 5, 4, 6),
            ('1984', '978-8535914849', 1949, 4, 3, 1),
            ('Fundação', '978-8576570646', 1951, 2, 2, 5),
            ('Assassinato no Expresso do Oriente', '978-8595084841', 1934, 3, 2, 4),
            ('O Iluminado', '978-8581050201', 1977, 3, 1, 4),
            ('Deuses Americanos', '978-8580573466', 2001, 2, 2, 6),
            ('O Alquimista', '978-8576654667', 1988, 4, 3, 1),
            ('Sapiens', '978-8525432629', 2011, 3, 2, 2),
            ('21 Lições para o Século 21', '978-8535930733', 2018, 2, 2, 2),
            ('A Revolução dos Bichos', '978-8535909555', 1945, 3, 3, 1),
            ('Memórias Póstumas de Brás Cubas', '978-8535911671', 1881, 2, 1, 1),
            ('A Paixão Segundo G.H.', '978-8520936610', 1964, 2, 2, 1),
            ('Harry Potter e a Câmara Secreta', '978-8532530795', 1998, 5, 5, 6);
        SELECT * FROM Livros;
        </code-block>
    </step>
    <step>
        <p>Associar Livros e Autores</p>
    </step>
    <step>
        <code-block lang="sql">
            INSERT INTO LivroAutor (LivroID, AutorID) VALUES
                (1, 1),   -- Dom Casmurro - Machado de Assis
                (2, 2),   -- A Hora da Estrela - Clarice Lispector
                (3, 3),   -- Harry Potter 1 - J.K. Rowling
                (4, 4),   -- 1984 - George Orwell
                (5, 5),   -- Fundação - Isaac Asimov
                (6, 6),   -- Assassinato no Expresso - Agatha Christie
                (7, 7),   -- O Iluminado - Stephen King
                (8, 8),   -- Deuses Americanos - Neil Gaiman
                (9, 9),   -- O Alquimista - Paulo Coelho
                (10, 10), -- Sapiens - Yuval Harari
                (11, 10), -- 21 Lições - Yuval Harari
                (12, 4),  -- Revolução dos Bichos - George Orwell
                (13, 1),  -- Memórias Póstumas - Machado de Assis
                (14, 2),  -- A Paixão - Clarice Lispector
                (15, 3);  -- Harry Potter 2 - J.K. Rowling
            -- Verificar livros com seus autores
            SELECT L.Titulo, A.Nome AS Autor
            FROM Livros L
            INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
            INNER JOIN Autores A ON LA.AutorID = A.AutorID
            ORDER BY L.Titulo;
        </code-block>
    </step>
    <step>
        <p>Inserir Empréstimos</p>
    </step>
    <step>
        <code-block lang="sql">
        -- Inserir empréstimos (alguns já devolvidos, outros ativos)
        INSERT INTO Emprestimos (LivroID, MembroID, DataEmprestimo, DataDevolucaoPrevista, DataDevolucaoReal, Status) VALUES
            -- Empréstimos devolvidos
            (1, 1, '2024-09-01', '2024-09-15', '2024-09-14', 'Devolvido'),
            (4, 2, '2024-09-05', '2024-09-19', '2024-09-18', 'Devolvido'),
            (9, 3, '2024-09-10', '2024-09-24', '2024-09-23', 'Devolvido'),
        -- Empréstimos ativos (dentro do prazo)
            (3, 1, '2024-10-10', '2024-10-24', NULL, 'Ativo'),
            (6, 4, '2024-10-12', '2024-10-26', NULL, 'Ativo'),
            (10, 5, '2024-10-13', '2024-10-27', NULL, 'Ativo'),
            (7, 6, '2024-10-14', '2024-10-28', NULL, 'Ativo'),
            (13, 7, '2024-10-15', '2024-10-29', NULL, 'Ativo'),
        -- Empréstimos atrasados
            (4, 8, '2024-09-20', '2024-10-04', NULL, 'Atrasado'),
            (7, 9, '2024-09-25', '2024-10-09', NULL, 'Atrasado');
        -- Atualizar número de cópias disponíveis
        UPDATE Livros SET NumCopiasDisponiveis = NumCopias - (
            SELECT COUNT(*) FROM Emprestimos
            WHERE LivroID = Livros.LivroID AND Status = 'Ativo'
        );
        SELECT * FROM Emprestimos;
        </code-block>
    </step>
</procedure>

## Fase 4: Consultas e Análises

Agora vamos praticar consultas úteis para o sistema:

### Consultas Básicas

```sql
-- 1. Listar todos os livros com suas categorias
SELECT
    L.Titulo,
    L.ISBN,
    C.Nome AS Categoria,
    L.AnoPublicacao,
    L.NumCopiasDisponiveis AS Disponiveis
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
ORDER BY L.Titulo;

-- 2. Listar todos os autores brasileiros
SELECT Nome, DataNascimento
FROM Autores
WHERE Nacionalidade = 'Brasileira'
ORDER BY Nome;

-- 3. Livros disponíveis para empréstimo (com cópias disponíveis)
SELECT Titulo, NumCopiasDisponiveis
FROM Livros
WHERE NumCopiasDisponiveis > 0
ORDER BY Titulo;

-- 4. Membros ativos da biblioteca
SELECT Nome, Email, Telefone, DataCadastro
FROM Membros
WHERE Ativo = 1
ORDER BY DataCadastro DESC;
```

### Consultas com Relacionamentos

```sql
-- 5. Livros com seus autores
SELECT
    L.Titulo,
    STRING_AGG(A.Nome, ', ') AS Autores,
    C.Nome AS Categoria
FROM Livros L
INNER JOIN LivroAutor LA ON L.LivroID = LA.LivroID
INNER JOIN Autores A ON LA.AutorID = A.AutorID
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
GROUP BY L.LivroID, L.Titulo, C.Nome
ORDER BY L.Titulo;

-- 6. Empréstimos ativos com informações completas
SELECT
    E.EmprestimoID,
    M.Nome AS Membro,
    L.Titulo AS Livro,
    E.DataEmprestimo,
    E.DataDevolucaoPrevista,
    DATEDIFF(DAY, GETDATE(), E.DataDevolucaoPrevista) AS DiasRestantes
FROM Emprestimos E
INNER JOIN Membros M ON E.MembroID = M.MembroID
INNER JOIN Livros L ON E.LivroID = L.LivroID
WHERE E.Status = 'Ativo'
ORDER BY E.DataDevolucaoPrevista;

-- 7. Histórico de empréstimos de um membro específico
SELECT
    L.Titulo,
    E.DataEmprestimo,
    E.DataDevolucaoPrevista,
    E.DataDevolucaoReal,
    E.Status
FROM Emprestimos E
INNER JOIN Livros L ON E.LivroID = L.LivroID
WHERE E.MembroID = 1  -- Ana Silva
ORDER BY E.DataEmprestimo DESC;
```

### Consultas de Análise

```sql
-- 8. Livros mais emprestados
SELECT TOP 5
    L.Titulo,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Livros L
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY L.LivroID, L.Titulo
ORDER BY TotalEmprestimos DESC;

-- 9. Membros com empréstimos atrasados
SELECT
    M.Nome,
    M.Email,
    L.Titulo,
    E.DataDevolucaoPrevista,
    DATEDIFF(DAY, E.DataDevolucaoPrevista, GETDATE()) AS DiasAtraso
FROM Emprestimos E
INNER JOIN Membros M ON E.MembroID = M.MembroID
INNER JOIN Livros L ON E.LivroID = L.LivroID
WHERE E.Status = 'Atrasado'
ORDER BY DiasAtraso DESC;

-- 10. Categorias mais populares
SELECT
    C.Nome AS Categoria,
    COUNT(E.EmprestimoID) AS TotalEmprestimos
FROM Categorias C
INNER JOIN Livros L ON C.CategoriaID = L.CategoriaID
LEFT JOIN Emprestimos E ON L.LivroID = E.LivroID
GROUP BY C.CategoriaID, C.Nome
ORDER BY TotalEmprestimos DESC;

-- 11. Livros que nunca foram emprestados
SELECT L.Titulo, C.Nome AS Categoria
FROM Livros L
INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
WHERE NOT EXISTS (
    SELECT 1 FROM Emprestimos E WHERE E.LivroID = L.LivroID
)
ORDER BY L.Titulo;

-- 12. Autores com mais livros no acervo
SELECT
    A.Nome AS Autor,
    COUNT(LA.LivroID) AS TotalLivros
FROM Autores A
INNER JOIN LivroAutor LA ON A.AutorID = LA.AutorID
GROUP BY A.AutorID, A.Nome
ORDER BY TotalLivros DESC;
```

## Fase 5: Operações de Manutenção

### Registrar um Novo Empréstimo

```sql
-- Procedimento para registrar empréstimo
DECLARE @LivroID INT = 3;  -- Harry Potter 1
DECLARE @MembroID INT = 5;  -- Elisa

BEGIN TRANSACTION;

    -- Verificar se o livro está disponível
    IF EXISTS (SELECT 1 FROM Livros WHERE LivroID = @LivroID AND NumCopiasDisponiveis > 0)
    BEGIN
        -- Registrar empréstimo
        INSERT INTO Emprestimos (LivroID, MembroID, DataEmprestimo, DataDevolucaoPrevista)
        VALUES (@LivroID, @MembroID, GETDATE(), DATEADD(DAY, 14, GETDATE()));

        -- Atualizar cópias disponíveis
        UPDATE Livros
        SET NumCopiasDisponiveis = NumCopiasDisponiveis - 1
        WHERE LivroID = @LivroID;

        COMMIT;
        PRINT 'Empréstimo registrado com sucesso!';
    END
    ELSE
    BEGIN
        ROLLBACK;
        PRINT 'Erro: Livro não disponível para empréstimo.';
    END
```

### Registrar Devolução

```sql
-- Procedimento para registrar devolução
DECLARE @EmprestimoID INT = 4;  -- ID do empréstimo

BEGIN TRANSACTION;

    -- Obter LivroID do empréstimo
    DECLARE @LivroIDDevolucao INT;
    SELECT @LivroIDDevolucao = LivroID FROM Emprestimos WHERE EmprestimoID = @EmprestimoID;

    -- Atualizar empréstimo
    UPDATE Emprestimos
    SET DataDevolucaoReal = GETDATE(),
        Status = 'Devolvido'
    WHERE EmprestimoID = @EmprestimoID;

    -- Atualizar cópias disponíveis
    UPDATE Livros
    SET NumCopiasDisponiveis = NumCopiasDisponiveis + 1
    WHERE LivroID = @LivroIDDevolucao;

COMMIT;
PRINT 'Devolução registrada com sucesso!';
```

### Atualizar Status de Empréstimos Atrasados

```sql
-- Atualizar status de empréstimos atrasados
UPDATE Emprestimos
SET Status = 'Atrasado'
WHERE Status = 'Ativo'
  AND DataDevolucaoPrevista < GETDATE()
  AND DataDevolucaoReal IS NULL;

-- Ver quantos foram atualizados
SELECT @@ROWCOUNT AS EmprestimosAtrasados;
```

## Desafios Adicionais

<procedure title="Desafios para Praticar" id="desafios-biblioteca">
    <step>
        <p><strong>Desafio 1:</strong> Adicione 5 novos livros ao acervo, incluindo seus autores</p>
    </step>
    <step>
        <p><strong>Desafio 2:</strong> Crie uma consulta que mostre os membros mais ativos (com mais empréstimos)</p>
    </step>
    <step>
        <p><strong>Desafio 3:</strong> Adicione uma tabela de Multas para registrar multas por atraso</p>
    </step>
    <step>
        <p><strong>Desafio 4:</strong> Crie uma consulta que calcule a multa de R$ 1,00 por dia de atraso</p>
    </step>
    <step>
        <p><strong>Desafio 5:</strong> Implemente uma validação que impeça membros com empréstimos atrasados de fazer novos empréstimos</p>
    </step>
    <step>
        <p><strong>Desafio 6:</strong> Crie um relatório mensal de empréstimos por categoria</p>
    </step>
    <step>
        <p><strong>Desafio 7:</strong> Adicione uma coluna de Editora na tabela Livros e popule com dados</p>
    </step>
    <step>
        <p><strong>Desafio 8:</strong> Crie uma consulta que identifique livros com baixa disponibilidade (menos de 2 cópias disponíveis)</p>
    </step>
</procedure>

## Soluções dos Desafios

<procedure title="Soluções Sugeridas" id="solucoes-desafios" collapsible="true">
    <step>
        <p><strong>Solução Desafio 2:</strong> Membros mais ativos</p>
        <code-block lang="sql">
        SELECT TOP 5
            M.Nome,
            M.Email,
            COUNT(E.EmprestimoID) AS TotalEmprestimos,
            SUM(CASE WHEN E.Status = 'Ativo' THEN 1 ELSE 0 END) AS EmprestimosAtivos
        FROM Membros M
        LEFT JOIN Emprestimos E ON M.MembroID = E.MembroID
        GROUP BY M.MembroID, M.Nome, M.Email
        ORDER BY TotalEmprestimos DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução Desafio 3:</strong> Tabela de Multas</p>
        <code-block lang="sql">
        CREATE TABLE Multas (
            MultaID INT PRIMARY KEY IDENTITY(1,1),
            EmprestimoID INT NOT NULL,
            ValorMulta DECIMAL(10,2) NOT NULL,
            DataGeracao DATE DEFAULT GETDATE(),
            Pago BIT DEFAULT 0,
            DataPagamento DATE,
            FOREIGN KEY (EmprestimoID) REFERENCES Emprestimos(EmprestimoID)
        );
        </code-block>
    </step>
    <step>
        <p><strong>Solução Desafio 4:</strong> Calcular multas por atraso</p>
        <code-block lang="sql">
        SELECT
            M.Nome AS Membro,
            L.Titulo AS Livro,
            E.DataDevolucaoPrevista,
            DATEDIFF(DAY, E.DataDevolucaoPrevista, GETDATE()) AS DiasAtraso,
            DATEDIFF(DAY, E.DataDevolucaoPrevista, GETDATE()) * 1.00 AS MultaDevida
        FROM Emprestimos E
        INNER JOIN Membros M ON E.MembroID = M.MembroID
        INNER JOIN Livros L ON E.LivroID = L.LivroID
        WHERE E.Status = 'Atrasado'
        ORDER BY DiasAtraso DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução Desafio 6:</strong> Relatório mensal por categoria</p>
        <code-block lang="sql">
        SELECT
            FORMAT(E.DataEmprestimo, 'yyyy-MM') AS MesAno,
            C.Nome AS Categoria,
            COUNT(E.EmprestimoID) AS TotalEmprestimos
        FROM Emprestimos E
        INNER JOIN Livros L ON E.LivroID = L.LivroID
        INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
        WHERE E.DataEmprestimo >= DATEADD(MONTH, -6, GETDATE())
        GROUP BY FORMAT(E.DataEmprestimo, 'yyyy-MM'), C.CategoriaID, C.Nome
        ORDER BY MesAno DESC, TotalEmprestimos DESC;
        </code-block>
    </step>
    <step>
        <p><strong>Solução Desafio 8:</strong> Livros com baixa disponibilidade</p>
        <code-block lang="sql">
        SELECT
            L.Titulo,
            L.NumCopias AS TotalCopias,
            L.NumCopiasDisponiveis AS Disponiveis,
            C.Nome AS Categoria
        FROM Livros L
        INNER JOIN Categorias C ON L.CategoriaID = C.CategoriaID
        WHERE L.NumCopiasDisponiveis = 2
        ORDER BY L.NumCopiasDisponiveis;
        </code-block>
    </step>
</procedure>

## Boas Práticas Aplicadas no Projeto

### 1. Modelagem Adequada
- ✅ Tabelas normalizadas (3ª forma normal)
- ✅ Relacionamentos bem definidos
- ✅ Uso apropriado de chaves primárias e estrangeiras

### 2. Integridade de Dados
- ✅ Constraints (CHECK, UNIQUE, NOT NULL)
- ✅ Foreign keys com ON DELETE CASCADE onde apropriado
- ✅ Valores padrão (DEFAULT)

### 3. Tipos de Dados Apropriados
- ✅ NVARCHAR para textos em português (suporta acentos)
- ✅ VARCHAR para dados sem acentos (ISBN, Email)
- ✅ DATE para datas
- ✅ DECIMAL para valores monetários

### 4. Segurança nas Operações
- ✅ Uso de transações para operações críticas
- ✅ Verificações antes de modificar dados
- ✅ Rollback em caso de erro

### 5. Consultas Eficientes
- ✅ Índices em chaves primárias (automático)
- ✅ JOINs apropriados
- ✅ Uso de agregações quando necessário

## Próximos Passos

Para aprimorar ainda mais este projeto, você pode:

1. **Adicionar Índices**: Criar índices em colunas frequentemente consultadas
2. **Criar Views**: Views para consultas complexas frequentes
3. **Stored Procedures**: Automatizar operações comuns
4. **Triggers**: Automatizar atualizações de disponibilidade
5. **Auditoria**: Tabela de log para rastrear mudanças
6. **Relatórios**: Criar relatórios gerenciais

## Glossary

**Modelo Conceitual**
: Representação abstrata das entidades e relacionamentos do sistema

**Modelo Lógico**
: Definição detalhada de atributos e tipos de dados

**Modelo Físico**
: Implementação real no banco de dados

**Chave Primária (PK)**
: Identificador único de cada registro em uma tabela

**Chave Estrangeira (FK)**
: Coluna que referencia a chave primária de outra tabela

**Constraint**
: Restrição que garante integridade dos dados

**Transação**
: Conjunto de operações tratadas como uma unidade indivisível

**Normalização**
: Processo de organizar dados para reduzir redundância

**Relacionamento N:N**
: Relacionamento muitos-para-muitos que requer tabela associativa

**CASCADE**
: Propagação automática de operações (DELETE/UPDATE) em registros relacionados

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/relational-databases/tables/primary-and-foreign-key-constraints">Microsoft Docs - Chaves Primárias e Estrangeiras</a>
        <a href="https://learn.microsoft.com/pt-br/sql/t-sql/statements/create-table-transact-sql">Microsoft Docs - CREATE TABLE</a>
        <a href="https://www.sqlshack.com/learn-sql-database-design/">SQL Shack - Database Design</a>
    </category>
</seealso>

> **Parabéns!**
>
> Você completou seu primeiro projeto prático de banco de dados! Continue praticando e experimentando com novas funcionalidades.
> Na próxima aula, aprenderemos sobre JOINs e relacionamentos mais avançados!
>
{style="tip"}
