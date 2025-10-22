# Aula 8: CSS Grid Layout

Bem-vindo à Aula 8 do nosso curso de Desenvolvimento Web! Nesta aula, você vai aprender sobre o CSS Grid Layout, um sistema de layout bidimensional poderoso que permite criar estruturas complexas de forma simples e intuitiva. O Grid complementa o Flexbox que estudamos anteriormente, oferecendo controle preciso sobre linhas e colunas simultaneamente.

## Visão Geral do Tópico

Nesta aula, você vai:
- Compreender os conceitos fundamentais do CSS Grid Layout
- Aprender a criar grids com diferentes tamanhos de células
- Posicionar elementos em áreas específicas do grid
- Criar layouts responsivos usando grid
- Combinar Grid e Flexbox para layouts mais complexos

## Exemplo: Crie uma Página com CSS Grid Layout

Siga os passos abaixo para criar sua página `grid.html` com um exemplo prático de layout utilizando CSS Grid:

1. **Crie o Projeto:**
    - Abra o Visual Studio Code.
    - Crie uma nova pasta para este projeto.

2. **Crie os Arquivos:**
    - Dentro da pasta, crie um arquivo chamado `grid.html`.
    - Crie também um arquivo chamado `grid.css`.

3. **Insira o Código HTML:**
    - No arquivo `grid.html`, adicione o seguinte código:

   ```html
   <!DOCTYPE html>
   <html lang="pt-BR">
     <head>
       <meta charset="UTF-8">
       <title>CSS Grid Layout</title>
       <link rel="stylesheet" href="grid.css">
     </head>
     <body>
       <div class="grid-container">
         <header class="cabecalho">
           <h1>Meu Site com Grid</h1>
         </header>
         <nav class="menu">
           <ul>
             <li><a href="#">Início</a></li>
             <li><a href="#">Sobre</a></li>
             <li><a href="#">Serviços</a></li>
             <li><a href="#">Contato</a></li>
           </ul>
         </nav>
         <aside class="sidebar">
           <h2>Menu Lateral</h2>
           <ul>
             <li><a href="#">Categoria 1</a></li>
             <li><a href="#">Categoria 2</a></li>
             <li><a href="#">Categoria 3</a></li>
             <li><a href="#">Categoria 4</a></li>
           </ul>
         </aside>
         <main class="conteudo">
           <h2>Conteúdo Principal</h2>
           <p>Este é o conteúdo principal da página, organizado em um layout de grid.</p>
           <div class="artigos">
             <article class="artigo">
               <h3>Artigo 1</h3>
               <p>Conteúdo do primeiro artigo aqui.</p>
             </article>
             <article class="artigo">
               <h3>Artigo 2</h3>
               <p>Conteúdo do segundo artigo aqui.</p>
             </article>
             <article class="artigo">
               <h3>Artigo 3</h3>
               <p>Conteúdo do terceiro artigo aqui.</p>
             </article>
             <article class="artigo">
               <h3>Artigo 4</h3>
               <p>Conteúdo do quarto artigo aqui.</p>
             </article>
           </div>
         </main>
         <footer class="rodape">
           <p>&copy; 2024 Meu Site com Grid</p>
         </footer>
       </div>
     </body>
   </html># Aula 8: Grid System
    ```

1. Insira o Código CSS:
No arquivo grid.css, adicione o seguinte código:

```css
/* Estilos básicos e reset */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: Arial, sans-serif;
  background-color: #f4f4f4;
  color: #333;
  line-height: 1.6;
}

/* Grid Container Principal */
.grid-container {
  display: grid;
  grid-template-areas: 
    "cabecalho cabecalho cabecalho"
    "menu menu menu"
    "sidebar conteudo conteudo"
    "rodape rodape rodape";
  grid-template-columns: 1fr 3fr 1fr;
  grid-template-rows: auto auto 1fr auto;
  min-height: 100vh;
  gap: 10px;
  padding: 10px;
}

/* Cabeçalho */
.cabecalho {
  grid-area: cabecalho;
  background-color: #4CAF50;
  color: #fff;
  padding: 20px;
  text-align: center;
}

/* Menu de navegação */
.menu {
  grid-area: menu;
  background-color: #333;
  padding: 10px;
}

.menu ul {
  list-style: none;
  display: flex;
  justify-content: center;
}

.menu li {
  margin: 0 15px;
}

.menu a {
  color: #fff;
  text-decoration: none;
  padding: 10px 0;
  display: block;
}

/* Sidebar */
.sidebar {
  grid-area: sidebar;
  background-color: #ddd;
  padding: 15px;
}

.sidebar ul {
  list-style: none;
}

.sidebar li {
  margin-bottom: 10px;
}

.sidebar a {
  color: #333;
  text-decoration: none;
}

/* Conteúdo principal */
.conteudo {
  grid-area: conteudo;
  background-color: #fff;
  padding: 20px;
  border: 1px solid #ccc;
}

/* Grid para os artigos dentro do conteúdo */
.artigos {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  margin-top: 20px;
}

.artigo {
  background-color: #f9f9f9;
  padding: 15px;
  border: 1px solid #eee;
  border-radius: 5px;
}

/* Rodapé */
.rodape {
  grid-area: rodape;
  background-color: #333;
  color: #fff;
  text-align: center;
  padding: 10px;
}

/* Media Queries para Responsividade */
@media (max-width: 768px) {
  .grid-container {
    grid-template-areas: 
      "cabecalho"
      "menu"
      "sidebar"
      "conteudo"
      "rodape";
    grid-template-columns: 1fr;
  }

  .artigos {
    grid-template-columns: 1fr;
  }
}
```

Visualize o Resultado:

Salve ambos os arquivos.
Abra o grid.html no seu navegador para ver o layout em ação.

Explicação Detalhada dos Conceitos de CSS Grid

Grid Container: 
: Uso: Cria um contexto de grid para os elementos filhos.
: display: grid: Define um container como grid.
: grid-template-columns: Define o número e tamanho das colunas.
: grid-template-rows: Define o número e tamanho das linhas.
: grid-template-areas: Define as áreas nomeadas no grid.
: gap: Define o espaçamento entre as células do grid.
: Posicionamento de Itens: : Uso: Controla a posição dos elementos dentro do grid.
: grid-area: Associa um elemento a uma área nomeada.
: grid-column: Define em quais colunas o item aparece.
: grid-row: Define em quais linhas o item aparece.
: justify-items e align-items: Alinha os itens dentro de suas células.

Unidades do Grid: 
: Uso: Define tamanhos e proporções no grid.
: fr: Unidade fracionária que distribui o espaço disponível.
: auto: Determina o tamanho baseado no conteúdo.
: minmax(): Define um intervalo de tamanho (mínimo e máximo).
: repeat(): Repete um padrão de tamanhos várias vezes.

> **Dicas Importantes**
>
> Grid vs Flexbox: Use Grid para layouts bidimensionais (linhas e colunas simultaneamente) e Flexbox para layouts unidimensionais (apenas linha ou coluna).
>
> Nomeie Áreas do Grid: Use grid-template-areas para criar layouts visuais e semânticos, facilitando o entendimento do código.
>
> Utilize Grid para Layouts Complexos: Com CSS Grid, você pode criar layouts assimétricos que seriam difíceis com outros métodos.
>
> Combine Grid e Flexbox: Use Grid para o layout principal da página e Flexbox para alinhar e distribuir elementos dentro das células do grid.
>
> Responsividade com Grid: Aproveite as media queries para reorganizar o grid em diferentes tamanhos de tela, criando layouts verdadeiramente responsivos.