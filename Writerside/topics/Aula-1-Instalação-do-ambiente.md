# Aula 1: Instalação do Ambiente

Esta documentação explica como instalar o **Visual Studio Code** e configurar duas extensões essenciais para o desenvolvimento web: **Live Preview** (para visualizar a página em tempo real) e **vscode-icons** (para melhorar a visualização dos arquivos com ícones personalizados). Ao seguir este procedimento, você estará apto a criar, editar e visualizar seus projetos web de forma dinâmica e organizada.

> **Dica Importante**
>
> Certifique-se de ter uma conexão com a internet ativa e permissões administrativas para instalar softwares no seu sistema.
>
{style="note"}

## Antes de começar

Antes de iniciar, verifique se você atende aos seguintes pré-requisitos:
- **Um computador com acesso à internet.**
- **Permissão para instalar softwares.**
- **Um sistema operacional compatível (Windows, macOS ou Linux).**

## Visual Stúdio Code - Passos de instalação.

Siga os passos abaixo para instalar o Visual Studio Code e configurar as extensões necessárias:

1. **Instalar o Visual Studio Code**

   Acesse o [site oficial do Visual Studio Code](https://code.visualstudio.com) e baixe a versão apropriada para o seu sistema operacional. Após o download, execute o instalador e siga as instruções apresentadas na tela.

   ```bash
   # Exemplo para usuários do Ubuntu via Snap:
   sudo snap install --classic code

2.	Abrir o Visual Studio Code
      Após a instalação, abra o VS Code. Você pode encontrá-lo no menu de aplicativos ou iniciá-lo via linha de comando digitando:

   ```bash
   code
   ```

<procedure title="Instalar a Extensões" id="instalar-live-preview">
    <step>
        <p>No VS Code, clique no ícone de <strong>Extensões</strong> na barra lateral ou pressione <shortcut>Ctrl+Shift+X</shortcut> (no Windows/Linux) ou <shortcut>Cmd+Shift+X</shortcut> (no macOS).</p>
        <img src="live_preview_extension.png" alt="Acesso às extensões no VS Code" border-effect="line"/>
    </step>
    <step>
        <p>Digite <em>Live Preview</em> na caixa de pesquisa para localizar a extensão.</p>
    </step>
    <step>
        <p>Selecione a extensão recomendada (algumas alternativas populares incluem <em>Live Server</em> ou <em>Live Preview</em>) e clique em <strong>Instalar</strong>.</p>
    </step>
   <step>
        <p>Ainda na seção de Extensões, pesquise por vscode-icons.</p>
    </step>
   <step>
        <p>Selecione a extensão vscode-icons e clique em Instalar. Essa extensão adiciona ícones personalizados para arquivos e pastas, facilitando a organização visual do seu projeto. </p>
    </step>
   <step>
      <p>Após instalar as extensões, reinicie o VS Code para que todas as configurações sejam aplicadas corretamente.
         Abra um arquivo HTML e, utilizando a extensão de Live Preview, inicie a visualização da página para confirmar que as alterações estão sendo atualizadas em tempo real. Geralmente, há um botão ou comando específico (por exemplo, Ctrl+Shift+L ou um ícone na barra de status) para ativar essa funcionalidade.
      </p>
   </step>
</procedure>


## Explorar Configurações e Personalizações
   -	Acesse as configurações do VS Code (via File > Preferences > Settings ou Ctrl+,) para ajustar opções das extensões conforme sua preferência.
   -	Configure atalhos personalizados e verifique as opções de atualização automática do Live Preview para otimizar seu fluxo de trabalho.

Parabéns! Agora você possui um ambiente de desenvolvimento configurado com o Visual Studio Code e as extensões essenciais para desenvolvimento web. Aproveite a visualização em tempo real e a organização visual proporcionada pelos ícones para tornar seu trabalho mais produtivo e agradável.