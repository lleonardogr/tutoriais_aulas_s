# Aula 3: Conteúdo e Estrutura em HTML

Bem-vindo à Aula 03 do nosso curso de Desenvolvimento Web! Nesta aula, vamos explorar como estruturar e organizar o conteúdo de uma página web usando HTML. Você aprenderá a utilizar tags para formatação de texto, criar listas e inserir imagens, links e vídeos. Esses conceitos são fundamentais para criar páginas web bem estruturadas e atraentes.

## Visão Geral do Tópico

Nesta aula, você vai:
- Aprender sobre elementos de formatação, como parágrafos, cabeçalhos e ênfases.
- Criar listas ordenadas e não ordenadas para organizar informações de forma clara.
- Inserir imagens e links para tornar o conteúdo mais interativo.
- Incorporar vídeos para enriquecer a experiência do usuário com conteúdos multimídia.

## Exemplo: Crie uma Página com Conteúdo Estruturado

Siga os passos abaixo para criar sua página `conteudo.html`:

<procedure title="Criar seu arquivo conteudo.html" id="criar-conteudo-html">
  <step>
    <p>Abra o Visual Studio Code e crie uma nova pasta para este projeto.</p>
  </step>
  <step>
    <p>Dentro da pasta, crie um arquivo chamado <code>conteudo.html</code>.</p>
  </step>
</procedure>

Copie e cole o exemplo de código abaixo no arquivo:
```html
   <!DOCTYPE html>
   <html lang="pt-BR">
     <head>
       <meta charset="UTF-8">
       <title>Conteúdo e Estrutura em HTML</title>
     </head>
     <body>
       <h1>Bem-vindo à Minha Página</h1>
       <p>Este é um exemplo de parágrafo. Aqui você pode adicionar textos explicativos, imagens e links.</p>
       
       <h2>Listas Ordenadas e Não Ordenadas</h2>
       <p>Exemplo de lista ordenada:</p>
       <ol>
         <li>Primeiro item</li>
         <li>Segundo item</li>
         <li>Terceiro item</li>
       </ol>
       
       <p>Exemplo de lista não ordenada:</p>
       <ul>
         <li>Item A</li>
         <li>Item B</li>
         <li>Item C</li>
       </ul>
       
       <h2>Utilizando a Tag &lt;div&gt;</h2>
       <p>A tag <code>&lt;div&gt;</code> é usada para agrupar e estruturar outros elementos em blocos. Isso facilita a organização e a aplicação de estilos específicos. Veja o exemplo abaixo:</p>
       <div style="border: 1px solid #ccc; padding: 10px;">
         <h3>Seção Agrupada com &lt;div&gt;</h3>
         <p>Este parágrafo está dentro de um <code>&lt;div&gt;</code>, que pode ser estilizado como um bloco separado.</p>
       </div>
       
       <h2>Utilizando a Tag &lt;table&gt;</h2>
       <p>A tag <code>&lt;table&gt;</code> permite criar tabelas para organizar dados em linhas e colunas. Por exemplo:</p>
       <table border="1" cellpadding="5">
         <tr>
           <th>Nome</th>
           <th>Idade</th>
           <th>Cidade</th>
         </tr>
         <tr>
           <td>Ana</td>
           <td>25</td>
           <td>São Paulo</td>
         </tr>
         <tr>
           <td>Bruno</td>
           <td>30</td>
           <td>Rio de Janeiro</td>
         </tr>
       </table>
       
       <h2>Inserindo Imagens e Links</h2>
       <p>A seguir, um exemplo de como inserir uma imagem:</p>
       <img src="https://via.placeholder.com/150" alt="Imagem de exemplo">
       
       <p>E aqui um exemplo de link:</p>
       <a href="https://www.example.com">Visite o Example.com</a>
       
       <h2>Incorporando Vídeos</h2>
       <p>Você pode incorporar vídeos do YouTube, por exemplo:</p>
       <iframe width="560" height="315" src="https://www.youtube.com/embed/dQw4w9WgXcQ" 
         title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
       </iframe>
     </body>
   </html>
```

## Explicação Detalhada das Tags e Elementos

**`<h1>` e `<h2>`**  
: *Uso:* Definem cabeçalhos. `<h1>` é o título principal e `<h2>` é usado para subtítulos ou seções.
  
**`<p>`**
: *Uso:* Define parágrafos, que organizam o texto em blocos legíveis.

**`<ol>` e `<ul>`**
: *Uso:* Criam listas ordenadas (`<ol>`) e não ordenadas (`<ul>`).  
: *Itens:* Cada item da lista é definido com `<li>`.

**`<img>`**  
: *Uso:* Insere imagens na página.  
: *Atributos:*  
    - `src`: define o caminho para a imagem.  
    - `alt`: fornece um texto alternativo que descreve a imagem.

**`<div>`**  
: Uso: Utilizada para agrupar e estruturar outros elementos em blocos, permitindo melhor organização e aplicação de estilos.
: Benefícios: Facilita a criação de layouts e a separação lógica do conteúdo.

**`<table>`**  
: Uso: Cria tabelas para organizar dados em linhas e colunas.
: Elementos Comuns:
	•	**`<tr>`**  para definir uma linha.
	•	**`<th>`**  para definir um cabeçalho de coluna.
	•	**`<td>`**   para definir uma célula de dados.

**`<a>`**  
: *Uso:* Cria links que permitem navegar para outras páginas ou sites.  
: *Atributo:*  
    - `href`: especifica o destino do link.

**`<iframe>`**  
: *Uso:* Incorpora conteúdos externos, como vídeos do YouTube, diretamente na página.  
: *Atributos:*  
    - `src`: o URL do conteúdo a ser incorporado.
    - `width` e `height`: definem o tamanho do frame.

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

---

<seealso>
    <category ref="wrs">
        <a href="https://www.w3schools.com/html/">W3Schools - HTML Tutorial</a>
        <a href="https://developer.mozilla.org/en-US/docs/Web/HTML">MDN Web Docs - HTML</a>
        <a href="https://html.com/">HTML.com</a>
        <a href="https://www.codecademy.com/learn/learn-html">Codecademy - Learn HTML</a>
    </category>
</seealso>
