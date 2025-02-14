# Aula 5: Introdução ao CSS

Bem-vindo à Aula 5 do nosso curso de Desenvolvimento Web! Hoje, vamos explorar o CSS, a linguagem de estilos usada para definir a aparência visual das páginas web. Você aprenderá como adicionar cores, fontes e layouts aos seus projetos, separando o conteúdo (HTML) da apresentação (CSS).

## Visão Geral do Tópico

Nesta aula, você vai:
- Compreender o que é CSS e sua importância no desenvolvimento web.
- Aprender a sintaxe básica do CSS, incluindo seletores, propriedades e valores.
- Criar um arquivo CSS e vinculá-lo a um documento HTML.
- Aplicar estilos simples para melhorar a aparência da sua página.
- Entender conceitos básicos como a cascata e o Box Model.

## Exemplo: Crie uma Página com CSS Básico

Siga os passos abaixo para criar sua página `index.html` com estilos aplicados via CSS:

<procedure title="Criar seu arquivo conteudo.html" id="criar-conteudo-html">
  <step>
    <p>Abra o Visual Studio Code e crie uma nova pasta para este projeto.</p>
  </step>
  <step>
    <p>Dentro da pasta, crie um arquivo chamado <code>index.html</code>.</p>
  </step>
 <step>
    <p>Crie também um arquivo chamado <code>sytle.css</code>.</p>
  </step>
</procedure>

- No arquivo `index.html`, adicione o seguinte código:

   ```html
   <!DOCTYPE html>
   <html lang="pt-BR">
     <head>
       <meta charset="UTF-8">
       <title>Página com CSS</title>
       <link rel="stylesheet" href="style.css">
     </head>
     <body>
       <h1>Bem-vindo ao CSS!</h1>
       <p>Este é um exemplo de página estilizada com CSS.</p>
     </body>
   </html>
  
- No arquivo style.css, adicione o seguinte código:

```CSS
/* Define a cor de fundo da página e estilos básicos */
body {
  background-color: #f0f8ff;
  font-family: Arial, sans-serif;
  color: #333;
  margin: 20px;
}

/* Estiliza o cabeçalho principal */
h1 {
  color: #2e8b57;
  text-align: center;
}

/* Estiliza o parágrafo */
p {
  font-size: 18px;
  line-height: 1.5;
}
```

- Visualize o Resultado:
- •	Salve ambos os arquivos. 
- •	Abra o index.html no seu navegador para ver a página com os estilos aplicados.

## Explicação Detalhada dos Conceitos e Elementos do CSS

**`seletores`**
: Uso: Identificam quais elementos HTML serão estilizados. Exemplos incluem seletores de tag, classe e id.

**`Propriedades e Valores:`**
: Uso: Determinam quais estilos serão aplicados. Por exemplo, color: #333; define a cor do texto.

**`Arquivo Externo:`**
: Uso: Um arquivo CSS separado, que é vinculado ao HTML através da tag `<link>` no `<head>`, facilitando a manutenção e a reutilização dos estilos.

**`Cascata:`**
: Uso: Refere-se ao mecanismo pelo qual os estilos são aplicados. Regras mais específicas ou definidas posteriormente podem sobrescrever regras anteriores.

**`Box Model:`**
: Uso: É o modelo que define o layout dos elementos na página, incluindo margens, bordas, preenchimento (padding) e o conteúdo em si.

> **Dicas Importantes**
>
> **Experimente Diversos Elementos:**
Modifique, adicione e remova tags para ver como cada alteração afeta a estrutura da página.
>
> **Organize seu Código:**
Utilize identação e quebras de linha para manter o código limpo e legível.
>
> **Verifique a Sintaxe:**
Sempre feche as tags corretamente e confira se os atributos estão bem definidos.
>
> **Salve e Atualize:**
Após cada modificação, salve o arquivo e atualize o navegador para ver as mudanças.


<seealso>
    <category ref="wrs">
        <a href="https://www.w3schools.com/css/">W3Schools - CSS Tutorial</a>
        <a href="https://developer.mozilla.org/en-US/docs/Web/CSS">MDN Web Docs - CSS</a>
        <a href="https://css-tricks.com/">CSS-Tricks</a>
        <a href="https://www.codecademy.com/learn/learn-css">Codecademy - Learn CSS</a>
    </category>
</seealso>
