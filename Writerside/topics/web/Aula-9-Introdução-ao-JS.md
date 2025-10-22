# Aula 9. Introdução ao JavaScript

## O que é JavaScript?

JavaScript é uma linguagem de programação de alto nível, interpretada e orientada a objetos. É principalmente conhecida como a linguagem de script para páginas web, mas também é utilizada em muitos ambientes fora do navegador, como Node.js, Apache CouchDB e Adobe Acrobat.

## História e Evolução

- Criada em 1995 por Brendan Eich na Netscape
- Inicialmente chamada de Mocha, depois LiveScript, e finalmente JavaScript
- Padronizada como ECMAScript em 1997
- Versões importantes: ES6 (2015), ES7, ES8, ES9, ES10, ES11, ES12

## Por que JavaScript?

- É a linguagem de programação da web
- Executa no lado do cliente (navegador)
- Permite páginas web interativas e dinâmicas
- Pode ser usada para desenvolvimento full-stack (Node.js)
- Grande comunidade e ecossistema

## Configuração do Ambiente

Para começar com JavaScript, você precisa apenas:
1. Um navegador web (Chrome, Firefox, etc.)
2. Um editor de texto (VS Code, Sublime Text, etc.)

## Sintaxe Básica

### Incluindo JavaScript em uma página HTML:

```html
<!-- Interno -->
<script>
    // Código JavaScript aqui
    console.log("Olá, mundo!");
</script>

<!-- Externo -->
<script src="script.js"></script>
```

### Variáveis e Tipos de Dados

JavaScript possui três formas de declarar variáveis:

```javascript
// var - escopo de função (não recomendado para novos códigos)
var nome = "João";

// let - escopo de bloco, pode ser reatribuído
let idade = 25;

// const - escopo de bloco, não pode ser reatribuído
const PI = 3.14159;
```

#### Tipos de Dados Primitivos:

```javascript
// String
let texto = "JavaScript é incrível";

// Number
let numero = 42;
let decimal = 3.14;

// Boolean
let verdadeiro = true;
let falso = false;

// Undefined
let indefinido;

// Null
let nulo = null;

// Symbol (ES6)
let simbolo = Symbol("descrição");

// BigInt (ES2020)
let numeroGrande = 1234567890123456789012345678901234567890n;
```

### Operadores

```javascript
// Aritméticos
let soma = 5 + 3;         // 8
let subtracao = 5 - 3;     // 2
let multiplicacao = 5 * 3; // 15
let divisao = 5 / 3;       // 1.6666...
let modulo = 5 % 3;        // 2
let exponenciacao = 5 ** 3; // 125

// Comparação
let igual = 5 == "5";      // true (compara valor)
let estritamenteIgual = 5 === "5"; // false (compara valor e tipo)
let maior = 5 > 3;         // true
let menorOuIgual = 5 <= 5; // true

// Lógicos
let e = true && false;     // false
let ou = true || false;    // true
let nao = !true;           // false
```

## Estruturas de Controle

### Condicionais

```javascript
// if, else if, else
let idade = 18;

if (idade < 13) {
    console.log("Criança");
} else if (idade < 18) {
    console.log("Adolescente");
} else {
    console.log("Adulto");
}

// switch
let dia = 3;
switch (dia) {
    case 1:
        console.log("Segunda-feira");
        break;
    case 2:
        console.log("Terça-feira");
        break;
    case 3:
        console.log("Quarta-feira");
        break;
    default:
        console.log("Outro dia");
}

// Operador ternário
let status = idade >= 18 ? "Maior de idade" : "Menor de idade";
```

### Loops

```javascript
// for
for (let i = 0; i < 5; i++) {
    console.log(i); // 0, 1, 2, 3, 4
}

// while
let contador = 0;
while (contador < 5) {
    console.log(contador); // 0, 1, 2, 3, 4
    contador++;
}

// do-while
let j = 0;
do {
    console.log(j); // 0, 1, 2, 3, 4
    j++;
} while (j < 5);

// for...of (para iteráveis como arrays)
const numeros = [1, 2, 3, 4, 5];
for (const num of numeros) {
    console.log(num); // 1, 2, 3, 4, 5
}

// for...in (para propriedades de objetos)
const pessoa = { nome: "Ana", idade: 30 };
for (const prop in pessoa) {
    console.log(`${prop}: ${pessoa[prop]}`); // nome: Ana, idade: 30
}
```

## Funções

```javascript
// Declaração de função
function somar(a, b) {
    return a + b;
}

// Expressão de função
const multiplicar = function(a, b) {
    return a * b;
};

// Arrow function (ES6)
const dividir = (a, b) => a / b;

// Parâmetros padrão
function saudar(nome = "Visitante") {
    return `Olá, ${nome}!`;
}

// Funções com número variável de argumentos
function somarTodos(...numeros) {
    return numeros.reduce((total, num) => total + num, 0);
}
```

## Arrays

```javascript
// Criando arrays
const frutas = ["Maçã", "Banana", "Laranja"];

// Acessando elementos
console.log(frutas[0]); // Maçã

// Métodos úteis
frutas.push("Morango");        // Adiciona no final
frutas.unshift("Uva");         // Adiciona no início
const ultima = frutas.pop();   // Remove do final
const primeira = frutas.shift(); // Remove do início

// Iteração
frutas.forEach((fruta, index) => {
    console.log(`${index}: ${fruta}`);
});

// map, filter, reduce
const numeros = [1, 2, 3, 4, 5];

const dobrados = numeros.map(n => n * 2);         // [2, 4, 6, 8, 10]
const pares = numeros.filter(n => n % 2 === 0);   // [2, 4]
const soma = numeros.reduce((acc, n) => acc + n, 0); // 15
```

## Objetos

```javascript
// Criando objetos
const pessoa = {
    nome: "Carlos",
    idade: 28,
    profissao: "Desenvolvedor",
    saudar: function() {
        return `Olá, meu nome é ${this.nome}`;
    }
};

// Acessando propriedades
console.log(pessoa.nome);        // Carlos
console.log(pessoa["profissao"]); // Desenvolvedor

// Adicionando/modificando propriedades
pessoa.local = "Brasil";
pessoa.idade = 29;

// Object destructuring (ES6)
const { nome, idade } = pessoa;

// Métodos Object
const chaves = Object.keys(pessoa);   // ['nome', 'idade', 'profissao', 'saudar', 'local']
const valores = Object.values(pessoa); // ['Carlos', 29, 'Desenvolvedor', [Function], 'Brasil']
const entradas = Object.entries(pessoa); // [['nome', 'Carlos'], ['idade', 29], ...]
```

## DOM (Document Object Model)

```javascript
// Selecionando elementos
const titulo = document.getElementById("titulo");
const paragrafos = document.getElementsByTagName("p");
const elementos = document.getElementsByClassName("item");
const primeiraDiv = document.querySelector("div");
const todasDivs = document.querySelectorAll("div");

// Modificando elementos
titulo.textContent = "Novo Título";
titulo.innerHTML = "Título com <span>HTML</span>";
titulo.style.color = "blue";
titulo.style.fontSize = "24px";

// Adicionando/removendo classes
titulo.classList.add("destaque");
titulo.classList.remove("antigo");
titulo.classList.toggle("visivel");

// Criando e adicionando elementos
const novoParagrafo = document.createElement("p");
novoParagrafo.textContent = "Texto do novo parágrafo";
document.body.appendChild(novoParagrafo);

// Removendo elementos
const elementoParaRemover = document.querySelector(".temporario");
elementoParaRemover.remove();
```

## Eventos

```javascript
// Adicionando event listeners
const botao = document.getElementById("meuBotao");

botao.addEventListener("click", function(evento) {
    console.log("Botão clicado!");
    console.log(evento);
});

// Formas alternativas
botao.onclick = function() {
    console.log("Clicado via propriedade onclick");
};

// Eventos comuns:
// click, dblclick, mouseenter, mouseleave, submit, keydown, keyup, load, etc.

// Propagação de eventos
elemento.addEventListener("click", function(e) {
    e.stopPropagation(); // Impede a propagação para elementos pais
});

// Prevenindo comportamento padrão
form.addEventListener("submit", function(e) {
    e.preventDefault(); // Impede o envio padrão do formulário
    // Código para validação e envio personalizado
});
```

## Assincronia em JavaScript

### Callbacks

```javascript
function buscarDados(callback) {
    setTimeout(() => {
        const dados = { id: 1, nome: "Produto" };
        callback(dados);
    }, 1000);
}

buscarDados(function(resultado) {
    console.log(resultado);
});
```

### Promises

```javascript
function buscarDados() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const dados = { id: 1, nome: "Produto" };
            // resolve(dados); // Sucesso
            // ou
            // reject("Erro ao buscar dados"); // Erro
            resolve(dados);
        }, 1000);
    });
}

buscarDados()
    .then(dados => console.log(dados))
    .catch(erro => console.error(erro));
```

### Async/Await (ES8)

```javascript
async function buscarUsuarios() {
    try {
        const resposta = await fetch('https://api.exemplo.com/usuarios');
        const dados = await resposta.json();
        return dados;
    } catch (erro) {
        console.error('Erro:', erro);
    }
}

// Chamando a função async
buscarUsuarios().then(usuarios => console.log(usuarios));
```

## Recursos para Aprendizado

- [MDN Web Docs](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript)
- [JavaScript.info](https://javascript.info/)
- [W3Schools](https://www.w3schools.com/js/)
- [freeCodeCamp](https://www.freecodecamp.org/)
- [Eloquent JavaScript](https://eloquentjavascript.net/)

## Frameworks e Bibliotecas Populares

- **React**: Biblioteca para construção de interfaces
- **Angular**: Framework completo para desenvolvimento web
- **Vue.js**: Framework progressivo para interfaces
- **Node.js**: Ambiente de execução JavaScript server-side
- **Express**: Framework web para Node.js
- **jQuery**: Biblioteca que simplifica manipulação DOM e Ajax

## Conclusão

JavaScript é uma linguagem versátil e poderosa que evoluiu muito desde sua criação. É fundamental para o desenvolvimento web moderno e seu conhecimento abre portas para diversas áreas de desenvolvimento, desde front-end até back-end, aplicações móveis e desktop.

Comece com os fundamentos, pratique regularmente e explore o vasto ecossistema que JavaScript oferece!
