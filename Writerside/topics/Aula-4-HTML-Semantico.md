# Aula 4: HTML Semântico e Acessibilidade

Bem-vindo à Aula 04 do nosso curso de Desenvolvimento Web! Nesta aula, vamos abordar o HTML semântico e a acessibilidade. Você aprenderá como utilizar tags semânticas para estruturar melhor o conteúdo, tornando suas páginas mais compreensíveis para navegadores e leitores de tela, e garantindo uma experiência mais inclusiva para todos os usuários.

## Visão Geral do Tópico

Nesta aula, você vai:
- Compreender o conceito de HTML semântico e sua importância para a organização do conteúdo.
- Conhecer as principais tags semânticas, como `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>` e `<footer>`.
- Aprender boas práticas de acessibilidade para tornar suas páginas web mais inclusivas.
- Criar uma página semântica e acessível utilizando um exemplo prático.

## Exemplo: Crie uma Página Semântica e Acessível


<procedure title="Siga os passos abaixo para criar sua página `semantic.html`:" id="semantic-html">
  <step>
    <p>Abra o Visual Studio Code e crie uma nova pasta para este projeto.</p>
  </step>
  <step>
    <p>Dentro da pasta, crie um arquivo chamado <code>semantic.html</code>.</p>
  </step>
</procedure>

**Insira o Código HTML:**
    - Copie e cole o exemplo de código abaixo no arquivo `semantic.html`:

   ```html
   <!DOCTYPE html>
   <html lang="pt-BR">
     <head>
       <meta charset="UTF-8">
       <title>Página Semântica e Acessível</title>
     </head>
     <body>
       <header>
         <h1>Meu Site Semântico</h1>
       </header>
       <nav>
         <ul>
           <li><a href="#home">Início</a></li>
           <li><a href="#sobre">Sobre</a></li>
           <li><a href="#contato">Contato</a></li>
         </ul>
       </nav>
       <main>
         <section id="home">
           <article>
             <h2>Bem-vindo!</h2>
             <p>Esta é a seção inicial do site, onde apresentamos o conteúdo principal.</p>
           </article>
         </section>
         <section id="sobre">
           <article>
             <h2>Sobre Nós</h2>
             <p>Informações sobre a empresa ou o site.</p>
           </article>
         </section>
         <section id="contato">
           <article>
             <h2>Contato</h2>
             <p>Entre em contato conosco por meio deste formulário.</p>
           </article>
         </section>
       </main>
       <aside>
         <p>Notícias e atualizações importantes podem aparecer aqui.</p>
       </aside>
       <footer>
         <p>&copy; 2024 Meu Site Semântico. Todos os direitos reservados.</p>
       </footer>
     </body>
   </html>
```

## Explicação Detalhada das Tags e Elementos

**`<header>`**  
: *Uso:* Define a seção de cabeçalho da página, geralmente contendo o logotipo, título e informações introdutórias.

**`<nav>`**  
: *Uso:* Representa a área de navegação, contendo links para as principais seções do site.

**`<main>`**  
: *Uso:* Abriga o conteúdo principal da página, que é exclusivo e central para a experiência do usuário.

**`<section>`**  
: *Uso:* Divide o conteúdo em seções temáticas. Cada seção deve conter um título que indique seu conteúdo.

**`<article>`**  
: *Uso:* Representa um conteúdo independente e autônomo, como um post de blog, notícia ou uma entrada de conteúdo que pode ser distribuída separadamente.

**`<aside>`**  
: *Uso:* Destina-se a conteúdos suplementares ou complementares, como links relacionados, anúncios ou informações adicionais que não fazem parte do conteúdo principal.

**`<footer>`**  
: *Uso:* Define o rodapé da página, onde são colocadas informações de copyright, links de contato e outras informações relevantes.

> **Dicas Importantes**
>
> **Utilize Tags Semânticas:**  
> Use tags semânticas para melhorar a estrutura e a legibilidade do seu código, facilitando a compreensão para navegadores e ferramentas de acessibilidade.
>
> **Acessibilidade:**  
> Adicione atributos descritivos (como `alt` em imagens) e textos claros em links para garantir que seu conteúdo seja acessível a todos os usuários, inclusive aqueles que utilizam leitores de tela.
>
> **Organize seu Código:**  
> Mantenha seu código bem indentado e organizado para facilitar a manutenção e a leitura.
>
> **Teste com Ferramentas de Acessibilidade:**  
> Se possível, utilize leitores de tela ou outras ferramentas de teste de acessibilidade para garantir que seu site esteja funcionando de forma inclusiva.

<seealso>
    <category ref="html-resources">
        <a href="https://www.w3schools.com/html/html5_semantic_elements.asp">W3Schools - HTML5 Semantic Elements</a>
        <a href="https://developer.mozilla.org/en-US/docs/Web/HTML/Element">MDN Web Docs - HTML Elements</a>
        <a href="https://www.w3.org/WAI/fundamentals/accessibility-intro/">W3C - Introduction to Web Accessibility</a>
        <a href="https://www.a11yproject.com/">The A11Y Project</a>
    </category>
</seealso>
