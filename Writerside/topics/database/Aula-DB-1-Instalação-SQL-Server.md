# Aula 1: Instalação e Configuração do SQL Server

Esta documentação explica como instalar o **Microsoft SQL Server** e suas principais ferramentas de gerenciamento: **SQL Server Management Studio (SSMS)** e **Azure Data Studio**. Ao seguir este procedimento, você estará apto a criar, gerenciar e consultar bancos de dados de forma profissional.

> **Dica Importante**
>
> Certifique-se de ter uma conexão com a internet ativa e permissões administrativas para instalar softwares no seu sistema. O SQL Server requer recursos significativos de hardware, especialmente memória RAM.
>
{style="note"}

## Antes de começar

Antes de iniciar, verifique se você atende aos seguintes pré-requisitos:

### Requisitos Mínimos

- **Sistema Operacional:** Windows 10/11, Windows Server 2016+, ou Linux (Ubuntu, Red Hat, SUSE)
- **Processador:** x64 com pelo menos 1.4 GHz (recomendado: 2.0 GHz ou superior)
- **Memória RAM:** Mínimo 1 GB (recomendado: 4 GB ou mais)
- **Espaço em Disco:** Mínimo 6 GB de espaço disponível
- **Permissões:** Conta de administrador do sistema
- **Internet:** Conexão ativa para download dos instaladores

### Edições do SQL Server

O SQL Server oferece várias edições para diferentes necessidades:

- **Express Edition:** Gratuita, ideal para aprendizado e aplicações pequenas (limite de 10 GB por banco)
- **Developer Edition:** Gratuita, com todos os recursos da Enterprise, mas apenas para desenvolvimento/teste
- **Standard Edition:** Versão paga para uso comercial com recursos intermediários
- **Enterprise Edition:** Versão completa com todos os recursos avançados

> **Para este curso, utilizaremos a Developer Edition ou Express Edition**
>
{style="tip"}

## Instalação do SQL Server

Siga os passos abaixo para instalar o SQL Server no Windows:

<procedure title="Instalar SQL Server 2022" id="instalar-sql-server">
    <step>
        <p>Acesse o <a href="https://www.microsoft.com/pt-br/sql-server/sql-server-downloads">site oficial do SQL Server</a> e baixe a edição desejada:</p>
        <list>
            <li><strong>Developer Edition:</strong> Clique em "Download now" na seção Developer</li>
            <li><strong>Express Edition:</strong> Clique em "Download now" na seção Express</li>
        </list>
    </step>
    <step>
        <p>Execute o instalador baixado (<code>SQL2022-SSEI-Dev.exe</code> ou <code>SQL2022-SSEI-Expr.exe</code>)</p>
    </step>
    <step>
        <p>Na tela inicial, escolha o tipo de instalação:</p>
        <list>
            <li><strong>Básico:</strong> Instalação rápida com configurações padrão (recomendado para iniciantes)</li>
            <li><strong>Personalizada:</strong> Permite escolher recursos específicos</li>
            <li><strong>Baixar Mídia:</strong> Para instalação offline</li>
        </list>
        <p>Para este curso, selecione <strong>Básico</strong></p>
    </step>
    <step>
        <p>Aceite os termos de licença clicando em <strong>Aceitar</strong></p>
    </step>
    <step>
        <p>Escolha o diretório de instalação (ou mantenha o padrão) e clique em <strong>Instalar</strong></p>
    </step>
    <step>
        <p>Aguarde o download e instalação dos componentes. Este processo pode levar de 10 a 30 minutos dependendo da sua conexão</p>
    </step>
    <step>
        <p>Após a conclusão, você verá uma tela com informações importantes:</p>
        <list>
            <li><strong>Nome da Instância:</strong> Geralmente <code>MSSQLSERVER</code> ou <code>SQLEXPRESS</code></li>
            <li><strong>String de Conexão:</strong> Algo como <code>localhost</code> ou <code>.\SQLEXPRESS</code></li>
        </list>
        <p><strong>Anote essas informações!</strong> Você precisará delas para conectar ao servidor</p>
    </step>
    <step>
        <p>Clique em <strong>Fechar</strong> para finalizar a instalação</p>
    </step>
</procedure>

## Instalação do SQL Server Management Studio (SSMS)

O SSMS é a ferramenta gráfica oficial da Microsoft para gerenciar o SQL Server. É essencial para administradores e desenvolvedores.

<procedure title="Instalar SSMS" id="instalar-ssms">
    <step>
        <p>Acesse a <a href="https://learn.microsoft.com/pt-br/sql/ssms/download-sql-server-management-studio-ssms">página de download do SSMS</a></p>
    </step>
    <step>
        <p>Clique em <strong>Download SQL Server Management Studio (SSMS)</strong></p>
    </step>
    <step>
        <p>Execute o instalador <code>SSMS-Setup-PTB.exe</code> (cerca de 600 MB)</p>
    </step>
    <step>
        <p>Na tela de boas-vindas, clique em <strong>Instalar</strong></p>
    </step>
    <step>
        <p>Aguarde a instalação (pode levar de 5 a 15 minutos)</p>
    </step>
    <step>
        <p>Clique em <strong>Fechar</strong> quando a instalação for concluída</p>
    </step>
    <step>
        <p>Reinicie o computador se solicitado</p>
    </step>
</procedure>

## Instalação do Azure Data Studio (Opcional)

O Azure Data Studio é uma ferramenta moderna, multi-plataforma e leve, ideal para trabalhar com SQL Server, PostgreSQL e Azure SQL.

<procedure title="Instalar Azure Data Studio" id="instalar-azure-data-studio">
    <step>
        <p>Acesse a <a href="https://learn.microsoft.com/pt-br/sql/azure-data-studio/download-azure-data-studio">página de download do Azure Data Studio</a></p>
    </step>
    <step>
        <p>Baixe a versão apropriada para seu sistema operacional:</p>
        <list>
            <li>Windows: Instalador (.exe) ou versão portátil (.zip)</li>
            <li>macOS: Arquivo .zip</li>
            <li>Linux: .deb (Debian/Ubuntu) ou .rpm (Red Hat)</li>
        </list>
    </step>
    <step>
        <p>Execute o instalador e siga as instruções na tela</p>
    </step>
    <step>
        <p>Após a instalação, abra o Azure Data Studio</p>
    </step>
</procedure>

## Primeira Conexão ao SQL Server

Agora vamos conectar ao SQL Server pela primeira vez usando o SSMS:

<procedure title="Conectar ao SQL Server" id="conectar-sql-server">
    <step>
        <p>Abra o <strong>SQL Server Management Studio (SSMS)</strong> no menu Iniciar</p>
    </step>
    <step>
        <p>Na janela "Conectar ao Servidor", preencha as informações:</p>
        <list>
            <li><strong>Tipo de servidor:</strong> Mecanismo de Banco de Dados</li>
            <li><strong>Nome do servidor:</strong>
                <code>localhost</code> ou <code>.\SQLEXPRESS</code> (se instalou Express Edition)
            </li>
            <li><strong>Autenticação:</strong> Autenticação do Windows (recomendado para ambiente local)</li>
        </list>
    </step>
    <step>
        <p>Clique em <strong>Conectar</strong></p>
    </step>
    <step>
        <p>Se conectado com sucesso, você verá o <strong>Pesquisador de Objetos</strong> com a árvore de bancos de dados</p>
    </step>
    <step>
        <p>Explore os <strong>Bancos de Dados do Sistema</strong>:</p>
        <list>
            <li><strong>master:</strong> Banco de dados principal com informações do sistema</li>
            <li><strong>model:</strong> Template para novos bancos</li>
            <li><strong>msdb:</strong> Utilizado pelo SQL Server Agent para jobs e alertas</li>
            <li><strong>tempdb:</strong> Armazena objetos temporários</li>
        </list>
    </step>
</procedure>

## Verificando a Instalação

Vamos executar alguns comandos para verificar se tudo está funcionando corretamente:

<procedure title="Testar a Instalação" id="testar-instalacao">
    <step>
        <p>No SSMS, clique em <strong>Nova Consulta</strong> na barra de ferramentas</p>
    </step>
    <step>
        <p>Digite o seguinte comando SQL para verificar a versão do SQL Server:</p>
        <code-block lang="sql">
SELECT @@VERSION AS 'Versão do SQL Server';
        </code-block>
    </step>
    <step>
        <p>Clique em <strong>Executar</strong> (ou pressione <shortcut>F5</shortcut>)</p>
    </step>
    <step>
        <p>Verifique o resultado na grade inferior - você deve ver informações sobre a versão instalada</p>
    </step>
    <step>
        <p>Execute mais alguns comandos de verificação:</p>
        <code-block lang="sql">
-- Verificar nome da instância
SELECT @@SERVERNAME AS 'Nome do Servidor';

-- Verificar bancos de dados disponíveis
SELECT name AS 'Bancos de Dados'
FROM sys.databases;

-- Verificar data/hora atual do servidor
SELECT GETDATE() AS 'Data/Hora Atual';
        </code-block>
    </step>
</procedure>

## Configurações Importantes

### Habilitar Autenticação Mista (Opcional)

Por padrão, o SQL Server usa apenas Autenticação do Windows. Para permitir logins SQL:

<procedure title="Habilitar Autenticação Mista" id="autenticacao-mista">
    <step>
        <p>No Pesquisador de Objetos, clique com botão direito no servidor e selecione <strong>Propriedades</strong></p>
    </step>
    <step>
        <p>Vá para a página <strong>Segurança</strong></p>
    </step>
    <step>
        <p>Em "Autenticação do Servidor", selecione <strong>Modo de Autenticação do SQL Server e do Windows</strong></p>
    </step>
    <step>
        <p>Clique em <strong>OK</strong></p>
    </step>
    <step>
        <p>Reinicie o serviço do SQL Server para aplicar as mudanças</p>
    </step>
</procedure>

### Configurar Firewall (se necessário)

Se você planeja acessar o SQL Server de outras máquinas na rede:

```bash
# Abrir porta 1433 no Firewall do Windows (executar como Administrador)
netsh advfirewall firewall add rule name="SQL Server" dir=in action=allow protocol=TCP localport=1433
```

## Explorando o SQL Server Management Studio

O SSMS possui várias áreas importantes:

- **Pesquisador de Objetos:** Navega pela estrutura do servidor e bancos de dados
- **Editor de Consultas:** Escreve e executa comandos SQL
- **Janela de Resultados:** Exibe os resultados das consultas
- **Janela de Mensagens:** Mostra erros, avisos e informações de execução
- **Modelo de Objeto:** Permite arrastar e soltar objetos no editor

### Atalhos Úteis do SSMS

| Atalho | Função |
|--------|--------|
| `Ctrl + N` | Nova consulta |
| `F5` ou `Ctrl + E` | Executar consulta |
| `Ctrl + R` | Mostrar/ocultar painel de resultados |
| `Ctrl + L` | Exibir plano de execução estimado |
| `Ctrl + K, Ctrl + C` | Comentar linhas selecionadas |
| `Ctrl + K, Ctrl + U` | Descomentar linhas selecionadas |
| `Ctrl + Shift + V` | Navegar histórico da área de transferência |

## Recursos Adicionais

### Documentação Oficial

- [Documentação do SQL Server](https://learn.microsoft.com/pt-br/sql/sql-server/)
- [Tutoriais do SQL Server](https://learn.microsoft.com/pt-br/sql/sql-server/tutorials-for-sql-server-2016)
- [Guia do SSMS](https://learn.microsoft.com/pt-br/sql/ssms/sql-server-management-studio-ssms)

### Comunidade e Suporte

- [Fórum do SQL Server](https://social.msdn.microsoft.com/Forums/pt-BR/home?category=sqlserver)
- [Stack Overflow - SQL Server](https://stackoverflow.com/questions/tagged/sql-server)
- [SQL Server Central](https://www.sqlservercentral.com/)

## Solução de Problemas Comuns

### Não consigo conectar ao servidor

**Problema:** Erro "Não é possível conectar ao servidor"

**Soluções:**
1. Verifique se o serviço está rodando:
   - Abra "Serviços" do Windows (`services.msc`)
   - Procure por "SQL Server (MSSQLSERVER)" ou "SQL Server (SQLEXPRESS)"
   - Certifique-se de que está "Em execução"
2. Verifique o nome do servidor:
   - Use `localhost` ou `.\SQLEXPRESS` para instância Express
   - Use `localhost` ou `.` para instância padrão
3. Verifique as configurações de firewall

### SQL Server usando muita memória

**Problema:** SQL Server consome muita RAM

**Solução:** O SQL Server gerencia memória dinamicamente. Para limitar:
```sql
-- Limitar memória máxima para 2 GB (2048 MB)
EXEC sp_configure 'max server memory (MB)', 2048;
RECONFIGURE;
```

### Erro ao executar comandos

**Problema:** "Não há permissão para executar essa ação"

**Solução:** Verifique se seu usuário tem as permissões necessárias ou conecte com usuário administrador

## Conclusão

Parabéns! Agora você possui um ambiente completo de desenvolvimento com SQL Server configurado e pronto para uso. Nas próximas aulas, aprenderemos a criar bancos de dados, tabelas e realizar consultas SQL.

> **Próximos Passos**
>
> - Familiarize-se com a interface do SSMS
> - Explore os bancos de dados do sistema
> - Experimente executar comandos simples de consulta
> - Prepare-se para criar seu primeiro banco de dados!
>
{style="tip"}
