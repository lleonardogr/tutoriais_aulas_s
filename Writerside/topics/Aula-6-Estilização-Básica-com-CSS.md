# Aula 6: Estilização Básica com CSS

Bem-vindo à Aula 6 do nosso curso de Desenvolvimento Web! Nesta aula, vamos aprender a estilizar sua página com CSS. Você aprenderá como aplicar cores, fontes, margens, paddings, bordas e entender o Box Model, além de utilizar classes e IDs para aplicar estilos específicos aos elementos.

## Visão Geral do Tópico

Nesta aula, você vai:
- Compreender os fundamentos do CSS para estilização básica.
- Aplicar propriedades de cores, fontes e fundos para dar vida à sua página.
- Trabalhar com margens, paddings e bordas para ajustar o espaçamento e a disposição dos elementos.
- Entender o conceito do Box Model e como ele afeta o layout.
- Aprender a usar classes e IDs para aplicar estilos específicos em diferentes elementos.

## Exemplo: Crie uma Página com Estilização Básica

Siga os passos abaixo para criar sua página `estilos.html` com CSS básico:

1. **Crie o Projeto:**
    - Abra o Visual Studio Code.
    - Crie uma nova pasta para este projeto.

2. **Crie os Arquivos:**
    - Dentro da pasta, crie um arquivo chamado `estilos.html`.
    - Crie também um arquivo chamado `estilos.css`.

3. **Insira o Código HTML:**
    - No arquivo `estilos.html`, adicione o seguinte código:

   ```html
   <!DOCTYPE html>
   <html lang="pt-BR">
     <head>
       <meta charset="UTF-8">
       <title>Página com Estilização Básica</title>
       <link rel="stylesheet" href="estilos.css">
     </head>
     <body>
       <header class="cabecalho">
         <h1>Minha Página Estilizada</h1>
       </header>
       <main>
         <section class="conteudo">
           <h2>Bem-vindo!</h2>
           <p>Este é um exemplo de página com estilização básica 
   utilizando CSS.</p>
         </section>
         <section class="detalhes">
           <h2>Detalhes da Estilização</h2>
           <p>Observe como as cores, margens, paddings e bordas são 
   aplicadas para melhorar o layout.</p>
         </section>
       </main>
       <footer class="rodape">
         <p>&copy; 2024 Meu Site Estilizado</p>
       </footer>
     </body>
   </html>
   ```

4.	Insira o Código CSS:
      •	No arquivo estilos.css, adicione o seguinte código:

```CSS
/* Reset básico e definição de fonte */
body {
font-family: Arial, sans-serif;
margin: 0;
padding: 0;
background-color: #f9f9f9;
color: #333;
}

/* Estilização do cabeçalho */
.cabecalho {
background-color: #4CAF50;
color: #fff;
padding: 20px;
text-align: center;
}

/* Estilização do conteúdo principal */
.conteudo {
padding: 20px;
margin: 20px;
background-color: #fff;
border: 1px solid #ddd;
border-radius: 4px;
}

/* Estilização dos detalhes */
.detalhes {
padding: 20px;
margin: 20px;
background-color: #eef;
border: 1px solid #99c;
border-radius: 4px;
}

/* Estilização do rodapé */
.rodape {
background-color: #333;
color: #fff;
text-align: center;
padding: 10px;
position: fixed;
bottom: 0;
width: 100%;
}

/* Exemplo de Box Model: Margem, Padding, e Borda */
.box-model {
margin: 20px;
padding: 15px;
border: 2px solid #4CAF50;
}
```

5.	Visualize o Resultado:
      •	Salve os arquivos.
      •	Abra o estilos.html no seu navegador para ver a página com os estilos aplicados.

Explicação Detalhada dos Conceitos e Elementos do CSS

Propriedades de Cores e Fontes:
: Uso: Define a cor do texto, o fundo e a família de fontes utilizadas na página. Exemplos: color, background-color e font-family.

Margens, Paddings e Bordas:
: Uso: Controlam o espaçamento dos elementos.
•	Margem: Espaço externo ao elemento.
•	Padding: Espaço interno entre a borda e o conteúdo.
•	Borda: Linha que envolve o elemento, definindo sua área.

Box Model:
: Uso: Modelo que descreve como os elementos são renderizados, considerando margem, borda, padding e o conteúdo. Esse conceito é essencial para ajustar o layout e o espaçamento na página.

Classes e IDs:
: Uso: Permitem aplicar estilos específicos a grupos de elementos (classes) ou a um único elemento (IDs).
: Exemplo: .cabecalho, .conteudo e .rodape são classes utilizadas para estilizar partes distintas da página.

Dicas Importantes

> Dicas Importantes
>
> **Pratique Regularmente:**
Experimente modificar os valores das propriedades (cores, margens, paddings, bordas) e veja como cada alteração afeta o layout.
>
> **Organize seu CSS:**
> Utilize comentários e mantenha o código bem estruturado para facilitar futuras manutenções.
>
> **Experimente com Classes e IDs:**
> Crie diferentes estilos para testar a aplicação de classes e IDs em elementos específicos.
>
> **Entenda o Box Model:**
> Use as ferramentas de desenvolvimento do navegador para inspecionar elementos e compreender melhor o Box Model.