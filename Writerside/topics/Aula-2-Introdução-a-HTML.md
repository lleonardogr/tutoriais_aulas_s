# Aula 2: Fundamentos de HTML

Bem-vindo à Aula 2 do nosso curso de Desenvolvimento Web! Hoje vamos explorar os fundamentos do HTML, a linguagem usada para estruturar o conteúdo das páginas web. Nesta aula, você aprenderá como criar sua primeira página chamada `index.html`, entenderá o papel de cada tag e conhecerá detalhes que facilitarão seu aprendizado.

## Visão Geral do Tópico

Nesta aula, você vai:
- **Criar sua primeira página `index.html`**: Um exemplo prático para você colocar em prática o que aprendeu.
- **Entender a estrutura básica de um documento HTML**: Descobrindo o que cada tag faz.
- **Explorar os conceitos-chave do HTML**: Com explicações detalhadas e dicas úteis.
- **Aprender passo a passo**: Utilizando procedimentos detalhados para ajudar no processo.

## Exemplo: Crie sua Primeira Página `index.html`

Para começar, vamos criar um arquivo HTML simples. Siga o procedimento abaixo:

<procedure title="Criar seu primeiro arquivo index.html" id="criar-index-html">
  <step>
    <p>Abra o Visual Studio Code e crie uma nova pasta para o seu projeto.</p>
  </step>
  <step>
    <p>Dentro da pasta do projeto, crie um novo arquivo chamado <code>index.html</code>.</p>
  </step>
</procedure>

Copie o seguin

```HTML
<!DOCTYPE html>
<html lang="pt-BR">
    <head>
        <meta charset="UTF-8">
        <title>Minha Primeira Página</title>
    </head>
    <body>
        <h1>Isso é um título</h1>
        <p>Isso é um paragrafo</p>
    </body>
</html>
```

## Explicação Detalhada das Tags

A seguir, explicamos cada parte do código para que você entenda o que cada tag faz:

**`<!DOCTYPE html>`**
: *O que é:* Uma declaração que informa ao navegador que o documento está usando HTML5.
: *Por que é importante:* Garante que o navegador interprete o código corretamente.

**`<html lang="pt-BR">`**
: *O que é:* A tag raiz que envolve todo o conteúdo da página.
: *Detalhes:* O atributo `lang="pt-BR"` especifica que o idioma do documento é o português do Brasil.

**`<head>`**
: *O que é:* Uma seção que contém metadados e informações que não são exibidas diretamente na página.
: *O que inclui:* Pode conter links para folhas de estilo, scripts, definições de caracteres, entre outros.

**`<meta charset="UTF-8">`**
: *O que é:* Define a codificação de caracteres do documento.
: *Por que é importante:* Garante que caracteres especiais (como acentos) sejam exibidos corretamente.

**`<title>`**
: *O que é:* Define o título da página, que aparece na aba do navegador.
: *Dica:* Escolha um título curto e descritivo.

**`<body>`**
: *O que é:* Contém o conteúdo visível da página, como textos, imagens, links, etc.
: *Função:* Tudo que você deseja que o usuário veja e interaja deve estar dentro desta tag.

**`<h1>`**
: *O que é:* Define o cabeçalho principal da página.
: *Dica:* Use `<h1>` para títulos importantes e utilize `<h2>`, `<h3>`, etc., para seções subsequentes.

**`<p>`**
: *O que é:* Define um parágrafo de texto.
:*Uso:* Utilize para organizar o texto e criar espaçamentos que facilitem a leitura.

## Passo a Passo para Explorar o Código

Para ajudar você a entender e modificar o código, siga este procedimento adicional:

<procedure title="Explorar e Modificar o Código HTML" id="explorar-html">
  <step>
    <p>Abra o arquivo <code>index.html</code> no Visual Studio Code.</p>
  </step>
  <step>
    <p>Localize a tag <code>&lt;h1&gt;</code> e modifique o texto para personalizá-lo, por exemplo, altere para <code>"Olá, Mundo!"</code>.</p>
  </step>
  <step>
    <p>Altere o conteúdo da tag <code>&lt;p&gt;</code> para descrever algo sobre você ou sobre o site.</p>
  </step>
  <step>
    <p>Salve o arquivo e atualize a página no navegador para ver as mudanças.</p>
  </step>
  <step>
    <p>Experimente adicionar uma nova tag, como um parágrafo extra (<code>&lt;p&gt;</code>), e observe como ele aparece na página.</p>
  </step>
</procedure>

> **Dicas Importantes**
> 
> **Pratique Regularmente:** A melhor forma de aprender HTML é praticando. Experimente modificar o código e veja como cada alteração afeta a página.
>
> **Não Tenha Medo de Errar:** Cada erro é uma oportunidade para aprender. Se algo não funcionar, revise seu código e tente identificar o problema.
>
> **Pesquise e Experimente:** Se tiver dúvidas, busque exemplos online e consulte a documentação oficial. A prática constante ajudará a fixar os conceitos.
>

## Comentários
Use comentários no seu código se necessário, por exemplo:
```HTML
<!-- este é um comentário -->
<h1>Isto é um texto</h1>
```


<seealso>
    <category ref="wrs">
        <a href="https://www.w3schools.com/html/">W3Schools - HTML Tutorial</a>
        <a href="https://developer.mozilla.org/en-US/docs/Web/HTML">MDN Web Docs - HTML</a>
        <a href="https://html.com/">HTML.com</a>
        <a href="https://www.codecademy.com/learn/learn-html">Codecademy - Learn HTML</a>
    </category>
</seealso>
