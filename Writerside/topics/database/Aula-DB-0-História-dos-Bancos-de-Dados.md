# Aula 0: História dos Bancos de Dados

## Introdução

Os sistemas de gerenciamento de bancos de dados (SGBDs) revolucionaram a forma como armazenamos, organizamos e recuperamos informações. Desde os primeiros sistemas baseados em arquivos até os modernos bancos de dados distribuídos e em nuvem, a evolução dos bancos de dados acompanhou e impulsionou o desenvolvimento da computação. Neste artigo, exploraremos a história dos bancos de dados, os principais marcos de sua evolução e como chegamos aos sistemas relacionais modernos, como o SQL Server.

## A Origem dos Bancos de Dados

Antes dos bancos de dados computacionais, as informações eram armazenadas em sistemas de arquivos simples. Cada aplicação mantinha seus próprios arquivos, o que levava a problemas de redundância, inconsistência e dificuldade de acesso aos dados. Na década de 1960, surgiram os primeiros sistemas de gerenciamento de dados para resolver esses problemas.

### Principais Marcos Históricos

- **Anos 1960:** Surgimento dos primeiros SGBDs hierárquicos e em rede
  - **IMS (Information Management System)** da IBM para o programa Apollo
  - Modelo hierárquico: dados organizados em estruturas em árvore
  - Modelo em rede: permitia relacionamentos mais complexos

- **1970:** Edgar F. Codd publica o artigo revolucionário "A Relational Model of Data for Large Shared Data Banks"
  - Introdução do modelo relacional baseado em teoria matemática
  - Conceito de tabelas, linhas e colunas
  - Independência entre dados e aplicações

- **Anos 1970-1980:** Desenvolvimento dos primeiros SGBDs relacionais
  - **System R** da IBM (precursor do DB2)
  - Surgimento da linguagem **SQL (Structured Query Language)**
  - **Oracle** lança o primeiro SGBD relacional comercial (1979)
  - **Ingres** desenvolvido na Universidade de Berkeley

- **Anos 1980:** Consolidação do modelo relacional
  - **SQL** torna-se padrão ANSI (1986)
  - Surgimento de produtos comerciais robustos
  - Crescimento exponencial do mercado de bancos de dados

- **1989:** Microsoft lança o **SQL Server 1.0** em parceria com Sybase e Ashton-Tate
  - Primeira versão para OS/2
  - Evolução contínua até se tornar um dos principais SGBDs do mercado

- **Anos 1990-2000:** Era dos bancos de dados empresariais
  - Bancos de dados distribuídos
  - Data warehouses e business intelligence
  - Expansão dos recursos de SQL Server (versões 6.5, 7.0, 2000)

- **Anos 2000-2010:** Diversificação e especialização
  - Surgimento dos bancos NoSQL para casos específicos
  - Cloud computing e bancos de dados como serviço
  - SQL Server 2005, 2008 e 2008 R2 com recursos avançados

- **Anos 2010-Presente:** Era moderna
  - Bancos de dados em nuvem (Azure SQL Database)
  - Big Data e análise em tempo real
  - SQL Server para Linux (2017)
  - Integração com IA e machine learning
  - SQL Server 2019, 2022 com recursos híbridos

## O Modelo Relacional: A Base da Computação Moderna

### Conceitos Fundamentais de Edgar F. Codd

Edgar F. Codd, pesquisador da IBM, revolucionou o mundo dos dados ao propor o modelo relacional. Seus princípios fundamentais incluem:

- **Estrutura Tabular:** Dados organizados em tabelas (relações) com linhas (tuplas) e colunas (atributos)
- **Integridade Referencial:** Relacionamentos entre tabelas mantidos através de chaves
- **Normalização:** Técnicas para eliminar redundância e manter consistência
- **Independência de Dados:** Separação entre a estrutura lógica e física dos dados
- **Linguagem Declarativa:** SQL permite especificar "o quê" buscar, não "como" buscar

### Por que o Modelo Relacional Prevaleceu?

1. **Base Matemática Sólida:** Fundamentado na teoria dos conjuntos e álgebra relacional
2. **Flexibilidade:** Permite consultas ad-hoc sem programação complexa
3. **Integridade:** Garante consistência dos dados através de constraints
4. **Padrão SQL:** Linguagem universal adotada por todos os principais SGBDs
5. **ACID:** Propriedades que garantem confiabilidade nas transações
   - **Atomicity (Atomicidade):** Transações completas ou totalmente revertidas
   - **Consistency (Consistência):** Dados sempre em estado válido
   - **Isolation (Isolamento):** Transações não interferem umas nas outras
   - **Durability (Durabilidade):** Dados confirmados são permanentes

## SQL Server: Evolução e Relevância

### Breve História do SQL Server

O Microsoft SQL Server tem uma trajetória marcante na história dos bancos de dados:

**Versões Principais:**
- **1989:** SQL Server 1.0 (parceria Microsoft/Sybase)
- **1993:** SQL Server 4.2 (primeira versão exclusivamente Microsoft)
- **1996:** SQL Server 6.5 (grande sucesso comercial)
- **1998:** SQL Server 7.0 (reescrita completa do código)
- **2000:** SQL Server 2000 (estabilidade e performance)
- **2005:** SQL Server 2005 (integração com .NET, XML nativo)
- **2008:** SQL Server 2008 (recursos de BI, compressão de dados)
- **2012:** SQL Server 2012 (AlwaysOn, ColumnStore)
- **2014:** SQL Server 2014 (in-memory OLTP)
- **2016:** SQL Server 2016 (segurança avançada, JSON)
- **2017:** SQL Server 2017 (disponível para Linux)
- **2019:** SQL Server 2019 (Big Data Clusters, UTF-8)
- **2022:** SQL Server 2022 (integração com Azure, recursos híbridos)

### Por que Aprender SQL Server?

1. **Mercado:** Amplamente utilizado em empresas de todos os tamanhos
2. **Integração:** Excelente integração com ecossistema Microsoft (.NET, Azure, Power BI)
3. **Ferramentas:** SQL Server Management Studio (SSMS) e Azure Data Studio
4. **Performance:** Otimizações avançadas e recursos de alta disponibilidade
5. **Comunidade:** Vasta documentação e comunidade ativa
6. **Certificações:** Certificações Microsoft reconhecidas no mercado
7. **Gratuito para Aprender:** Edição Express e Developer gratuitas

## Bancos de Dados Hoje e no Futuro

### Aplicações Modernas

Os bancos de dados relacionais são utilizados em praticamente todos os setores:

- **E-commerce:** Catálogos de produtos, pedidos, clientes
- **Bancos e Finanças:** Transações, contas, histórico financeiro
- **Saúde:** Prontuários eletrônicos, agendamentos, exames
- **Educação:** Sistemas acadêmicos, notas, matrículas
- **Governo:** Sistemas de gestão pública, cidadãos, serviços
- **Entretenimento:** Streaming, jogos, redes sociais

### Tendências e Desafios

- **Cloud Computing:** Migração para bancos de dados em nuvem (Azure SQL, AWS RDS)
- **Big Data:** Processamento de volumes massivos de dados
- **Bancos Híbridos:** Combinação de SQL e NoSQL
- **Segurança:** Proteção contra ataques e conformidade com LGPD/GDPR
- **IA e ML:** Integração de inteligência artificial nos SGBDs
- **Edge Computing:** Bancos de dados distribuídos em dispositivos IoT
- **Real-time Analytics:** Análise de dados em tempo real
- **Autonomous Databases:** Bancos auto-gerenciados com IA

### O Papel do Profissional de Dados

Com a crescente importância dos dados, surgem novas profissões:

- **DBA (Database Administrator):** Gerencia e mantém bancos de dados
- **Data Engineer:** Constrói pipelines e infraestrutura de dados
- **Data Analyst:** Analisa dados para gerar insights de negócio
- **Data Scientist:** Utiliza estatística e ML para modelagem preditiva
- **SQL Developer:** Desenvolve aplicações e queries otimizadas

## Glossary

**SGBD (Sistema de Gerenciamento de Banco de Dados)**
: Software responsável por criar, manipular e administrar bancos de dados. Exemplos: SQL Server, Oracle, MySQL, PostgreSQL.

**Modelo Relacional**
: Paradigma de organização de dados baseado em tabelas relacionadas através de chaves, proposto por Edgar F. Codd em 1970.

**SQL (Structured Query Language)**
: Linguagem padrão para gerenciamento e consulta de bancos de dados relacionais.

**ACID**
: Conjunto de propriedades que garantem confiabilidade nas transações: Atomicity, Consistency, Isolation, Durability.

**Edgar F. Codd**
: Cientista da computação britânico que revolucionou a área ao propor o modelo relacional de dados.

**Normalização**
: Processo de organização de dados para reduzir redundância e melhorar integridade.

**NoSQL**
: Categoria de bancos de dados não-relacionais otimizados para casos específicos (documentos, grafos, chave-valor).

**OLTP (Online Transaction Processing)**
: Sistemas focados em processar transações rápidas e confiáveis (ex: sistema bancário).

**OLAP (Online Analytical Processing)**
: Sistemas focados em análise de grandes volumes de dados (ex: data warehouse).

**Cloud Database**
: Banco de dados hospedado e gerenciado em nuvem (Azure SQL Database, Amazon RDS).

**Big Data**
: Conjuntos de dados extremamente grandes que requerem técnicas especiais de processamento.

**Data Warehouse**
: Repositório centralizado de dados integrados de múltiplas fontes para análise e relatórios.
