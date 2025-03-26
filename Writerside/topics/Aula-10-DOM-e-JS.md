# Aula 10: DOM e JS

## O que é o DOM?

O Document Object Model (DOM) é uma interface de programação para documentos HTML e XML. Ele representa a página para que programas possam alterar a estrutura, estilo e conteúdo do documento. O DOM representa o documento como nós e objetos, permitindo que linguagens como JavaScript interajam com a página.

## Estrutura do DOM

O DOM é estruturado como uma árvore de objetos, conhecida como "árvore DOM":

```
document
└── html
    ├── head
    │   ├── title
    │   ├── meta
    │   └── link
    └── body
        ├── header
        ├── div
        │   ├── h1
        │   └── p
        ├── section
        └── footer
```

## Acessando Elementos do DOM

JavaScript oferece vários métodos para selecionar elementos do DOM:

```javascript
// Selecionar por ID
const elemento = document.getElementById('meuId');

// Selecionar por classe (retorna uma HTMLCollection)
const elementos = document.getElementsByClassName('minhaClasse');

// Selecionar por tag (retorna uma HTMLCollection)
const paragrafos = document.getElementsByTagName('p');

// Selecionar usando seletores CSS (retorna o primeiro elemento encontrado)
const primeiroElemento = document.querySelector('.classe-ou-#id');

// Selecionar todos os elementos que correspondam ao seletor (retorna NodeList)
const todosElementos = document.querySelectorAll('.classe');
```

## Manipulando o Conteúdo dos Elementos

### Modificando Texto e HTML

```javascript
// Alterando o conteúdo de texto
elemento.textContent = 'Novo texto';

// Alterando o HTML interno
elemento.innerHTML = '<span>Conteúdo HTML</span>';

// Obtendo ou modificando o valor de um input
const input = document.querySelector('input');
console.log(input.value); // Ler o valor
input.value = 'Novo valor'; // Definir o valor
```

### Manipulando Atributos

```javascript
// Obtendo um atributo
const imgSrc = elemento.getAttribute('src');

// Definindo um atributo
elemento.setAttribute('src', 'nova-imagem.jpg');

// Verificando se um atributo existe
const temAtributo = elemento.hasAttribute('disabled');

// Removendo um atributo
elemento.removeAttribute('disabled');

// Usando propriedades diretas para atributos comuns
elemento.id = 'novoId';
elemento.className = 'novaClasse';
elemento.href = 'https://exemplo.com';
```

### Manipulando Estilos

```javascript
// Alterando estilos inline
elemento.style.color = 'red';
elemento.style.backgroundColor = '#f0f0f0';
elemento.style.fontSize = '16px';

// Adicionando ou removendo classes
elemento.classList.add('destaque');
elemento.classList.remove('inativo');
elemento.classList.toggle('visivel'); // Adiciona se não existir, remove se existir
elemento.classList.contains('destaque'); // Verifica se contém a classe
```

## Criando e Removendo Elementos

### Criando Elementos

```javascript
// Criar um novo elemento
const novoParagrafo = document.createElement('p');

// Adicionar conteúdo ao elemento
novoParagrafo.textContent = 'Este é um novo parágrafo';

// Adicionar classe
novoParagrafo.className = 'texto-destacado';

// Inserir o elemento no DOM
document.body.appendChild(novoParagrafo); // Adiciona como último filho do body
```

### Removendo Elementos

```javascript
// Remover um elemento
elemento.remove();

// Remover um elemento filho
paiElemento.removeChild(filhoElemento);
```

## Navegando pelo DOM

```javascript
// Acessando o elemento pai
const pai = elemento.parentNode; // ou parentElement

// Acessando elementos filhos
const filhos = elemento.children; // Retorna uma HTMLCollection
const primeiroFilho = elemento.firstElementChild;
const ultimoFilho = elemento.lastElementChild;

// Acessando elementos irmãos
const irmaoPosterior = elemento.nextElementSibling;
const irmaoAnterior = elemento.previousElementSibling;
```

## Eventos do DOM

### Adicionando Event Listeners

```javascript
// Método moderno de adicionar eventos
elemento.addEventListener('click', function(event) {
    console.log('Elemento clicado!');
    console.log(event); // Objeto do evento
});

// Removendo event listener
elemento.removeEventListener('click', minhaFuncao);
```

### Eventos Comuns

```javascript
// Clique
elemento.addEventListener('click', handleClick);

// Passar o mouse por cima
elemento.addEventListener('mouseover', handleMouseOver);
elemento.addEventListener('mouseout', handleMouseOut);

// Pressionar tecla
document.addEventListener('keydown', handleKeyDown);

// Envio de formulário
formulario.addEventListener('submit', handleSubmit);

// Carregamento da página
window.addEventListener('load', handleLoad);

// Quando o DOM está pronto
document.addEventListener('DOMContentLoaded', handleReady);
```

### Prevenindo Comportamento Padrão

```javascript
formulario.addEventListener('submit', function(event) {
    event.preventDefault(); // Impede o envio padrão do formulário
    // Lógica personalizada aqui
});

link.addEventListener('click', function(event) {
    event.preventDefault(); // Impede a navegação padrão
    // Lógica personalizada aqui
});
```

## Exemplo Prático

Abaixo está um exemplo completo que demonstra várias técnicas de manipulação do DOM:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Manipulação do DOM</title>
    <style>
        .destaque { 
            background-color: yellow; 
            font-weight: bold; 
        }
        .completa {
            text-decoration: line-through;
            color: gray;
        }
    </style>
</head>
<body>
    <h1>Lista de Tarefas</h1>
    
    <form id="tarefa-form">
        <input type="text" id="tarefa-input" placeholder="Adicione uma nova tarefa">
        <button type="submit">Adicionar</button>
    </form>
    
    <ul id="lista-tarefas">
        <li>Estudar JavaScript</li>
        <li>Praticar manipulação do DOM</li>
    </ul>
    
    <script>
        // Selecionar elementos
        const form = document.getElementById('tarefa-form');
        const input = document.getElementById('tarefa-input');
        const lista = document.getElementById('lista-tarefas');
        
        // Adicionar tarefa ao enviar o formulário
        form.addEventListener('submit', function(event) {
            event.preventDefault();
            
            // Verificar se o input não está vazio
            if (input.value.trim() !== '') {
                // Criar novo item de lista
                const novaTarefa = document.createElement('li');
                novaTarefa.textContent = input.value;
                
                // Adicionar evento de clique para marcar como concluída
                novaTarefa.addEventListener('click', function() {
                    this.classList.toggle('completa');
                });
                
                // Adicionar evento de mouseover para destacar
                novaTarefa.addEventListener('mouseover', function() {
                    this.classList.add('destaque');
                });
                
                // Remover destaque ao tirar o mouse
                novaTarefa.addEventListener('mouseout', function() {
                    this.classList.remove('destaque');
                });
                
                // Adicionar à lista
                lista.appendChild(novaTarefa);
                
                // Limpar o input
                input.value = '';
            }
        });
        
        // Adicionar os mesmos eventos aos itens existentes
        const itensExistentes = lista.getElementsByTagName('li');
        for (let i = 0; i < itensExistentes.length; i++) {
            itensExistentes[i].addEventListener('click', function() {
                this.classList.toggle('completa');
            });
            
            itensExistentes[i].addEventListener('mouseover', function() {
                this.classList.add('destaque');
            });
            
            itensExistentes[i].addEventListener('mouseout', function() {
                this.classList.remove('destaque');
            });
        }
    </script>
</body>
</html>
```

## Considerações Finais

A manipulação do DOM é uma habilidade fundamental para qualquer desenvolvedor front-end. Ela permite criar páginas web interativas e dinâmicas sem a necessidade de recarregar a página.

Pontos importantes a lembrar:
- O DOM é uma representação em memória do documento HTML
- JavaScript pode selecionar, criar, modificar e remover elementos do DOM
- Event listeners permitem responder a ações do usuário
- Manipular o DOM diretamente pode ser menos eficiente que usar frameworks como React ou Vue, que utilizam DOM virtual

Dominando essas técnicas, você estará preparado para criar aplicações web dinâmicas e responsivas!
