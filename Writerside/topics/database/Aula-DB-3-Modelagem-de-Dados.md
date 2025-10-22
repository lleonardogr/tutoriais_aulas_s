# Aula 3: Modelagem de Dados

## Introdução

A modelagem de dados é o processo de criar uma representação visual e estruturada das informações que serão armazenadas em um banco de dados. É uma etapa fundamental no desenvolvimento de sistemas, pois define como os dados serão organizados, relacionados e acessados. Nesta aula, você aprenderá os conceitos de modelagem conceitual, lógica e física, além de técnicas de normalização.

## Por que Modelar Dados?

A modelagem de dados oferece diversos benefícios:

- **Organização:** Estrutura lógica e clara das informações
- **Comunicação:** Facilita o entendimento entre desenvolvedores, analistas e usuários
- **Integridade:** Garante consistência e confiabilidade dos dados
- **Performance:** Otimiza o acesso e armazenamento de informações
- **Manutenção:** Facilita futuras modificações e expansões
- **Documentação:** Serve como referência para o sistema

## Níveis de Modelagem

A modelagem de dados é dividida em três níveis progressivos:

### 1. Modelagem Conceitual

**Objetivo:** Representar a visão de alto nível do negócio, independente de tecnologia.

**Características:**
- Foco no "o quê", não no "como"
- Usa o Modelo Entidade-Relacionamento (MER)
- Diagrama Entidade-Relacionamento (DER)
- Independente de SGBD
- Linguagem próxima ao usuário de negócio

**Elementos:**
- **Entidades:** Objetos do mundo real (Aluno, Professor, Disciplina)
- **Atributos:** Características das entidades (Nome, CPF, Email)
- **Relacionamentos:** Conexões entre entidades (Aluno CURSA Disciplina)

**Exemplo Visual:**
```
[Aluno] ----< matriculado em >---- [Turma]
   |                                   |
   |                                   |
Atributos:                        Atributos:
- Matrícula                      - Código
- Nome                           - Nome
- CPF                            - Período
- Email                          - Sala
```

### 2. Modelagem Lógica

**Objetivo:** Traduzir o modelo conceitual para uma estrutura que pode ser implementada em um SGBD relacional.

**Características:**
- Define tabelas, colunas e tipos de dados
- Estabelece chaves primárias e estrangeiras
- Aplica regras de normalização
- Ainda independente do SGBD específico
- Mais técnico que o conceitual

**Elementos:**
- **Tabelas:** Representação de entidades
- **Colunas:** Atributos das entidades
- **Chaves Primárias (PK):** Identificam unicamente cada registro
- **Chaves Estrangeiras (FK):** Criam relacionamentos entre tabelas
- **Constraints:** Regras de integridade

**Exemplo:**

```
Tabela: Aluno
- AlunoID (PK)
- Nome
- CPF (UNIQUE)
- Email
- DataNascimento

Tabela: Turma
- TurmaID (PK)
- Nome
- Periodo
- Sala

Tabela: Matricula (tabela associativa)
- MatriculaID (PK)
- AlunoID (FK -> Aluno)
- TurmaID (FK -> Turma)
- DataMatricula
- Status
```

### 3. Modelagem Física

**Objetivo:** Implementar o modelo lógico em um SGBD específico (SQL Server, no nosso caso).

**Características:**
- Código SQL real (CREATE TABLE)
- Tipos de dados específicos do SGBD
- Índices, views, stored procedures
- Otimizações de performance
- Configurações de armazenamento

**Exemplo em SQL Server:**

```sql
CREATE TABLE Aluno (
    AlunoID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100) NOT NULL,
    CPF CHAR(11) UNIQUE NOT NULL,
    Email VARCHAR(100),
    DataNascimento DATE,
    DataCadastro DATETIME2 DEFAULT GETDATE(),
    INDEX IX_Aluno_CPF (CPF)
);

CREATE TABLE Turma (
    TurmaID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100) NOT NULL,
    Periodo VARCHAR(20),
    Sala VARCHAR(10),
    Ativa BIT DEFAULT 1
);

CREATE TABLE Matricula (
    MatriculaID INT PRIMARY KEY IDENTITY(1,1),
    AlunoID INT NOT NULL,
    TurmaID INT NOT NULL,
    DataMatricula DATE DEFAULT GETDATE(),
    Status VARCHAR(20) DEFAULT 'Ativa',
    FOREIGN KEY (AlunoID) REFERENCES Aluno(AlunoID),
    FOREIGN KEY (TurmaID) REFERENCES Turma(TurmaID),
    CONSTRAINT UK_Matricula UNIQUE (AlunoID, TurmaID)
);
```

## Relacionamentos entre Entidades

Os relacionamentos definem como as entidades se conectam. Existem três tipos principais:

### 1. Um para Um (1:1)

Cada registro de uma tabela se relaciona com exatamente um registro de outra tabela.

**Exemplo:** Pessoa e Passaporte
- Uma pessoa possui um passaporte
- Um passaporte pertence a uma pessoa

```sql
CREATE TABLE Pessoa (
    PessoaID INT PRIMARY KEY,
    Nome VARCHAR(100)
);

CREATE TABLE Passaporte (
    PassaporteID INT PRIMARY KEY,
    Numero VARCHAR(20),
    PessoaID INT UNIQUE,  -- UNIQUE garante 1:1
    FOREIGN KEY (PessoaID) REFERENCES Pessoa(PessoaID)
);
```

### 2. Um para Muitos (1:N)

Um registro de uma tabela se relaciona com vários registros de outra tabela.

**Exemplo:** Cliente e Pedidos
- Um cliente pode fazer vários pedidos
- Cada pedido pertence a um cliente

```sql
CREATE TABLE Cliente (
    ClienteID INT PRIMARY KEY,
    Nome VARCHAR(100)
);

CREATE TABLE Pedido (
    PedidoID INT PRIMARY KEY,
    Data DATE,
    ClienteID INT,  -- Chave estrangeira
    FOREIGN KEY (ClienteID) REFERENCES Cliente(ClienteID)
);
```

### 3. Muitos para Muitos (N:M)

Vários registros de uma tabela se relacionam com vários registros de outra tabela.

**Exemplo:** Alunos e Disciplinas
- Um aluno cursa várias disciplinas
- Uma disciplina tem vários alunos

**Solução:** Tabela associativa (também chamada de tabela de junção)

```sql
CREATE TABLE Aluno (
    AlunoID INT PRIMARY KEY,
    Nome VARCHAR(100)
);

CREATE TABLE Disciplina (
    DisciplinaID INT PRIMARY KEY,
    Nome VARCHAR(100)
);

-- Tabela associativa resolve N:M
CREATE TABLE AlunoDisc iplina (
    AlunoID INT,
    DisciplinaID INT,
    Nota DECIMAL(5,2),
    Semestre VARCHAR(10),
    PRIMARY KEY (AlunoID, DisciplinaID),  -- Chave composta
    FOREIGN KEY (AlunoID) REFERENCES Aluno(AlunoID),
    FOREIGN KEY (DisciplinaID) REFERENCES Disciplina(DisciplinaID)
);
```

## Cardinalidade e Participação

**Cardinalidade:** Define quantos registros de uma entidade podem se relacionar com outra.
- **Mínima:** Quantos no mínimo (0 ou 1)
- **Máxima:** Quantos no máximo (1 ou N)

**Participação:**
- **Total (obrigatória):** Todo registro deve participar do relacionamento
- **Parcial (opcional):** Registro pode ou não participar

**Notações:**
```
(0,1) = Mínimo 0, Máximo 1 (opcional, no máximo um)
(1,1) = Mínimo 1, Máximo 1 (obrigatório, exatamente um)
(0,N) = Mínimo 0, Máximo N (opcional, vários)
(1,N) = Mínimo 1, Máximo N (obrigatório, vários)
```

## Normalização de Dados

A normalização é o processo de organizar dados para reduzir redundância e melhorar integridade. Existem várias formas normais (FN), sendo as três primeiras as mais importantes:

### Primeira Forma Normal (1FN)

**Regra:** Eliminar grupos repetitivos e garantir atomicidade dos atributos.

**Problema:**
```
Pedido
- PedidoID
- Cliente
- Produtos: "Notebook, Mouse, Teclado"  ❌ Não atômico
```

**Solução:**
```
Pedido
- PedidoID
- Cliente

PedidoProduto
- PedidoID
- Produto  ✅ Cada produto em uma linha
```

### Segunda Forma Normal (2FN)

**Regra:** Estar em 1FN e eliminar dependências parciais (todo atributo não-chave deve depender da chave primária completa).

**Problema:**
```
MatriculaDisciplina (PK: AlunoID, DisciplinaID)
- AlunoID
- DisciplinaID
- NomeAluno  ❌ Depende apenas de AlunoID
- NomeDisciplina  ❌ Depende apenas de DisciplinaID
- Nota
```

**Solução:**
```
Aluno
- AlunoID
- NomeAluno  ✅

Disciplina
- DisciplinaID
- NomeDisciplina  ✅

Matricula
- AlunoID
- DisciplinaID
- Nota  ✅
```

### Terceira Forma Normal (3FN)

**Regra:** Estar em 2FN e eliminar dependências transitivas (atributos não-chave não podem depender de outros atributos não-chave).

**Problema:**
```
Funcionario
- FuncionarioID
- Nome
- DepartamentoID
- NomeDepartamento  ❌ Depende de DepartamentoID (transitivo)
- LocalDepartamento  ❌ Depende de DepartamentoID
```

**Solução:**
```
Funcionario
- FuncionarioID
- Nome
- DepartamentoID  ✅

Departamento
- DepartamentoID
- NomeDepartamento  ✅
- LocalDepartamento  ✅
```

### Quando Desnormalizar?

Às vezes, a desnormalização controlada é aceitável para:
- Melhorar performance de leitura
- Reduzir JOINs complexos
- Data warehouses e sistemas analíticos

**Exemplo:** Armazenar o nome do cliente na tabela de pedidos para evitar JOIN constante.

## Chaves e Constraints

### Tipos de Chaves

**Primary Key (Chave Primária)**
: Identifica unicamente cada registro. Não pode ser NULL e deve ser única.

```sql
CREATE TABLE Produto (
    ProdutoID INT PRIMARY KEY,
    Nome VARCHAR(100)
);
```

**Foreign Key (Chave Estrangeira)**
: Referencia a chave primária de outra tabela, criando relacionamentos.

```sql
CREATE TABLE Pedido (
    PedidoID INT PRIMARY KEY,
    ClienteID INT,
    FOREIGN KEY (ClienteID) REFERENCES Cliente(ClienteID)
);
```

**Unique Key (Chave Única)**
: Garante valores únicos, mas permite NULL (exceto no SQL Server onde permite apenas um NULL).

```sql
CREATE TABLE Usuario (
    UsuarioID INT PRIMARY KEY,
    Email VARCHAR(100) UNIQUE,
    CPF CHAR(11) UNIQUE
);
```

**Composite Key (Chave Composta)**
: Combinação de duas ou mais colunas que juntas identificam unicamente um registro.

```sql
CREATE TABLE ItemPedido (
    PedidoID INT,
    ProdutoID INT,
    Quantidade INT,
    PRIMARY KEY (PedidoID, ProdutoID)  -- Chave composta
);
```

### Constraints (Restrições)

**NOT NULL**
: Campo obrigatório

```sql
Nome VARCHAR(100) NOT NULL
```

**UNIQUE**
: Valores únicos na coluna

```sql
Email VARCHAR(100) UNIQUE
```

**CHECK**
: Valida valores baseado em condição

```sql
Idade INT CHECK (Idade >= 18),
Status VARCHAR(20) CHECK (Status IN ('Ativo', 'Inativo', 'Pendente'))
```

**DEFAULT**
: Valor padrão quando não especificado

```sql
DataCadastro DATETIME2 DEFAULT GETDATE(),
Ativo BIT DEFAULT 1
```

## Exemplo Prático Completo: Sistema de Biblioteca

Vamos modelar um sistema de biblioteca seguindo todos os conceitos:

```sql
-- Criação do banco de dados
CREATE DATABASE BibliotecaDB;
GO

USE BibliotecaDB;
GO

-- Tabela de Autores
CREATE TABLE Autor (
    AutorID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100) NOT NULL,
    Nacionalidade NVARCHAR(50),
    DataNascimento DATE,
    Biografia NVARCHAR(MAX)
);

-- Tabela de Categorias
CREATE TABLE Categoria (
    CategoriaID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(50) NOT NULL UNIQUE,
    Descricao NVARCHAR(200)
);

-- Tabela de Livros
CREATE TABLE Livro (
    LivroID INT PRIMARY KEY IDENTITY(1,1),
    Titulo NVARCHAR(200) NOT NULL,
    ISBN CHAR(13) UNIQUE,
    AnoPublicacao SMALLINT CHECK (AnoPublicacao >= 1000 AND AnoPublicacao <= YEAR(GETDATE())),
    NumeroPaginas INT CHECK (NumeroPaginas > 0),
    CategoriaID INT NOT NULL,
    Sinopse NVARCHAR(MAX),
    FOREIGN KEY (CategoriaID) REFERENCES Categoria(CategoriaID)
);

-- Tabela associativa Livro-Autor (N:M)
CREATE TABLE LivroAutor (
    LivroID INT,
    AutorID INT,
    OrdemAutor TINYINT DEFAULT 1,  -- Para co-autores
    PRIMARY KEY (LivroID, AutorID),
    FOREIGN KEY (LivroID) REFERENCES Livro(LivroID),
    FOREIGN KEY (AutorID) REFERENCES Autor(AutorID)
);

-- Tabela de Exemplares (um livro pode ter várias cópias físicas)
CREATE TABLE Exemplar (
    ExemplarID INT PRIMARY KEY IDENTITY(1,1),
    LivroID INT NOT NULL,
    CodigoBarras VARCHAR(50) UNIQUE,
    Localizacao VARCHAR(50),
    Estado VARCHAR(20) CHECK (Estado IN ('Novo', 'Bom', 'Regular', 'Ruim')) DEFAULT 'Bom',
    Disponivel BIT DEFAULT 1,
    FOREIGN KEY (LivroID) REFERENCES Livro(LivroID)
);

-- Tabela de Usuários
CREATE TABLE Usuario (
    UsuarioID INT PRIMARY KEY IDENTITY(1,1),
    Nome NVARCHAR(100) NOT NULL,
    CPF CHAR(11) UNIQUE NOT NULL,
    Email VARCHAR(100),
    Telefone VARCHAR(20),
    DataCadastro DATE DEFAULT GETDATE(),
    Ativo BIT DEFAULT 1
);

-- Tabela de Empréstimos
CREATE TABLE Emprestimo (
    EmprestimoID INT PRIMARY KEY IDENTITY(1,1),
    UsuarioID INT NOT NULL,
    ExemplarID INT NOT NULL,
    DataEmprestimo DATE DEFAULT GETDATE(),
    DataDevolucaoPrevista DATE NOT NULL,
    DataDevolucaoReal DATE,
    Status VARCHAR(20) CHECK (Status IN ('Ativo', 'Devolvido', 'Atrasado')) DEFAULT 'Ativo',
    FOREIGN KEY (UsuarioID) REFERENCES Usuario(UsuarioID),
    FOREIGN KEY (ExemplarID) REFERENCES Exemplar(ExemplarID),
    -- Constraint: data de devolução prevista deve ser após empréstimo
    CONSTRAINT CHK_DataDevolucao CHECK (DataDevolucaoPrevista > DataEmprestimo)
);

-- Índices para melhorar performance de consultas comuns
CREATE INDEX IX_Livro_Titulo ON Livro(Titulo);
CREATE INDEX IX_Usuario_Nome ON Usuario(Nome);
CREATE INDEX IX_Emprestimo_Status ON Emprestimo(Status);
```

## Boas Práticas de Modelagem

1. **Identifique corretamente as entidades:** Substantivos do negócio
2. **Normalize adequadamente:** Até 3FN na maioria dos casos
3. **Use nomes significativos:** Auto-explicativos e consistentes
4. **Documente relacionamentos:** Especifique cardinalidades claramente
5. **Defina constraints apropriadas:** NOT NULL, UNIQUE, CHECK, etc.
6. **Considere performance:** Índices em colunas frequentemente consultadas
7. **Planeje para crescimento:** Escalabilidade futura
8. **Valide com stakeholders:** Confirme que o modelo atende ao negócio

## Ferramentas de Modelagem

### Ferramentas Visuais

- **Microsoft Visio:** Diagramas profissionais
- **MySQL Workbench:** Gratuito, multiplataforma
- **ERwin Data Modeler:** Ferramenta enterprise
- **dbdiagram.io:** Online, colaborativo
- **Lucidchart:** Online, intuitivo
- **Draw.io:** Gratuito, open-source

### No SQL Server Management Studio

O SSMS possui diagramas de banco de dados integrados:

<procedure title="Criar Diagrama no SSMS" id="criar-diagrama-ssms">
    <step>
        <p>No Pesquisador de Objetos, expanda seu banco de dados</p>
    </step>
    <step>
        <p>Clique com botão direito em <strong>Diagramas de Banco de Dados</strong></p>
    </step>
    <step>
        <p>Selecione <strong>Novo Diagrama de Banco de Dados</strong></p>
    </step>
    <step>
        <p>Adicione as tabelas que deseja visualizar</p>
    </step>
    <step>
        <p>O SSMS gerará automaticamente o diagrama com relacionamentos</p>
    </step>
</procedure>

## Glossary

**Entidade**
: Objeto do mundo real que pode ser identificado e armazenado no banco (Cliente, Produto, Pedido)

**Atributo**
: Característica ou propriedade de uma entidade (Nome, CPF, Preço)

**Relacionamento**
: Associação entre duas ou mais entidades

**Cardinalidade**
: Número de ocorrências de uma entidade que podem se relacionar com outra

**Normalização**
: Processo de organizar dados para reduzir redundância e melhorar integridade

**Chave Primária (PK)**
: Atributo ou conjunto de atributos que identificam unicamente um registro

**Chave Estrangeira (FK)**
: Atributo que referencia a chave primária de outra tabela

**Chave Composta**
: Chave primária formada por duas ou mais colunas

**Tabela Associativa**
: Tabela que resolve relacionamentos N:M

**Constraint**
: Regra que garante integridade e validade dos dados

**DER (Diagrama Entidade-Relacionamento)**
: Representação gráfica do modelo conceitual

**1FN, 2FN, 3FN**
: Primeira, Segunda e Terceira Formas Normais

## Recursos Adicionais

<seealso>
    <category ref="external">
        <a href="https://learn.microsoft.com/pt-br/sql/relational-databases/tables/tables">Microsoft Docs - Tabelas</a>
        <a href="https://www.guru99.com/database-normalization.html">Database Normalization Tutorial</a>
        <a href="https://www.lucidchart.com/pages/pt/o-que-e-diagrama-entidade-relacionamento">Guia de DER</a>
    </category>
</seealso>

> **Próximos Passos**
>
> - Pratique criar modelos conceituais para diferentes sistemas
> - Implemente modelos lógicos em SQL Server
> - Experimente normalizar bases de dados desnormalizadas
> - Crie diagramas usando ferramentas visuais
>
{style="tip"}
