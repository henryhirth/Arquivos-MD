# 🚀 Módulo 1 — Core JS (ESNext+ 2026)

**Mentor:** Engenharia de Software Sênior | **Aluno:** Henry Hirth | **Local:** São Mateus do Sul, PR  
**Projeto:** Portal de Ferramentas da Internet (Full-Stack)  
**Progresso do Roadmap:** `██░░░░░░░░` 5% — Módulo 1/8 iniciado

---

## Índice

1. [Por que JavaScript é a espinha dorsal do seu projeto](#1-por-que-javascript-é-a-espinha-dorsal-do-seu-projeto)
2. [Onde o JS vive: Browser vs. Node.js/Bun/Deno](#2-onde-o-js-vive-browser-vs-nodejsbundeno)
3. [Setup do Ambiente de Desenvolvimento](#3-setup-do-ambiente-de-desenvolvimento)
4. [Variáveis: var, let e const — e por que importa](#4-variáveis-var-let-e-const--e-por-que-importa)
5. [Tipos de Dados e Type Coercion](#5-tipos-de-dados-e-type-coercion)
6. [Funções: Declaração, Expressão e Arrow Functions](#6-funções-declaração-expressão-e-arrow-functions)
7. [Closures: O Conceito Mais Importante do JS](#7-closures-o-conceito-mais-importante-do-js)
8. [O DOM: Conectando JS ao seu HTML](#8-o-dom-conectando-js-ao-seu-html)
9. [Eventos: Fazendo o Botão de Login Funcionar](#9-eventos-fazendo-o-botão-de-login-funcionar)
10. [Modules (ESModules): Organizando o Código do Projeto](#10-modules-esmodules-organizando-o-código-do-projeto)
11. [Integração com seus Guias de HTML e CSS](#11-integração-com-seus-guias-de-html-e-css)
12. [Conexão com Autenticação: O que o JS faz no Login](#12-conexão-com-autenticação-o-que-o-js-faz-no-login)
13. [Código Ready-to-Run: Splash Page com JS](#13-código-ready-to-run-splash-page-com-js)
14. [Desafios Práticos](#14-desafios-práticos)
15. [Checklist e Próximo Módulo](#15-checklist-e-próximo-módulo)

---

## 1. Por que JavaScript é a espinha dorsal do seu projeto

> **Analogia:** Pense no seu portal de ferramentas como um carro. O HTML é a carroceria — estrutura e forma. O CSS é a pintura e o design interior. O **JavaScript é o motor, o painel, a direção e os freios** — tudo que faz o carro realmente funcionar quando você interage com ele.

Você já tem uma base sólida construída: seu **HTML semântico** define a estrutura (`<header>`, `<main>`, `<section>`, `<article>`, `<dialog>`), e seu **CSS moderno** define a aparência (unidades em `rem`, `dvh`, variáveis custom, sistema de botões escalável). Agora o JavaScript vai **dar vida** a essa estrutura.

No seu projeto específico, o JS vai fazer coisas concretas:

| Situação no site | O que o JS faz |
|---|---|
| Usuário clica "Entrar com Google" | Inicia o fluxo OAuth, redireciona para Google |
| Google retorna com token | JS envia o token para o backend, recebe sessão |
| Login bem-sucedido | Troca `page--splash` por `page--dashboard` no `<body>` |
| Usuário clica em logout | Envia `POST /logout`, limpa cookie, volta para splash |
| Usuário abre modal de logout | Chama `dialog.showModal()` no `<dialog id="dialog-logout">` |
| Contador de ferramentas | Atualiza `<span id="tools-count">` dinamicamente |

Cada uma dessas ações é JavaScript puro. Você vai entender e escrever todas elas neste módulo.

---

## 2. Onde o JS vive: Browser vs. Node.js/Bun/Deno

> **Analogia:** O mesmo ator pode interpretar papéis completamente diferentes no cinema e no teatro. O JavaScript é esse ator — a linguagem é a mesma, mas o "palco" (o ambiente de execução) muda radicalmente o que ele pode fazer.

### 2.1 No Navegador (Frontend)

O motor V8 (Chrome/Edge), SpiderMonkey (Firefox) ou JavaScriptCore (Safari) lê e executa seu JS. Nesse ambiente, o JS tem acesso a:

```
APIs do Browser disponíveis:
├── DOM API          → document.querySelector(), element.classList
├── Fetch API        → fetch('https://api.seusite.com/login')
├── Web Storage      → localStorage, sessionStorage (⚠️ NÃO use para tokens!)
├── Cookie API       → document.cookie (prefira HttpOnly cookies via backend)
├── Events API       → addEventListener('click', handler)
├── History API      → history.pushState() para navegação SPA
└── Web APIs         → setTimeout, setInterval, requestAnimationFrame
```

**O que o JS do browser NÃO pode fazer:**
- Acessar o sistema de arquivos diretamente
- Conectar ao banco de dados diretamente
- Guardar segredos (client_secret do Google, etc.) — esses SEMPRE ficam no backend

### 2.2 No Servidor (Backend com Node.js / Bun / Deno)

O mesmo V8, mas extraído do browser e empacotado com APIs de sistema operacional. Aqui o JS pode:

```
APIs do Node/Bun/Deno disponíveis:
├── File System      → fs.readFile(), fs.writeFile()
├── HTTP Server      → http.createServer(), Express, Fastify, Hono
├── Crypto           → crypto.randomBytes() ← para tokens de auth!
├── Database         → conectar a PostgreSQL, MongoDB, Redis
├── Environment      → process.env.GOOGLE_CLIENT_SECRET
└── OS               → process.platform, process.argv
```

**Comparação dos runtimes para o seu projeto:**

| Runtime | Velocidade | Segurança | TypeScript nativo | Indicado para |
|---|---|---|---|---|
| **Node.js** | Boa | Padrão | Via tsc/tsx | Ecossistema maduro, npm enorme |
| **Bun** | 🚀 Mais rápido | Padrão | ✅ Nativo | Cold starts, scripts rápidos |
| **Deno** | Boa | 🔒 Máxima (permissões) | ✅ Nativo | Segurança, Edge functions |

Para o seu projeto de aprendizado: **comece com Node.js** (ecossistema maior, mais tutoriais, você já vai encontrar tudo que precisa). Migrar para Bun depois é trivial — a maioria do código é idêntico.

---

## 3. Setup do Ambiente de Desenvolvimento

### 3.1 O que instalar

```bash
# 1. Node.js (inclui npm automaticamente)
# Acesse: https://nodejs.org → baixe a versão LTS (Long Term Support)
# Verifique a instalação:
node --version   # deve mostrar v20.x.x ou superior
npm --version    # deve mostrar 10.x.x ou superior

# 2. VS Code (editor recomendado)
# Acesse: https://code.visualstudio.com

# 3. Extensões VS Code recomendadas para o seu projeto:
# - "ESLint" (dbaeumer.vscode-eslint)
# - "Prettier" (esbenp.prettier-vscode)  
# - "px to rem" (cipchk.css-rem) ← você já viu no guia de CSS
# - "Auto Rename Tag" (formulahendry.auto-rename-tag)
# - "Thunder Client" (rangav.vscode-thunder-client) ← testar APIs
```

### 3.2 Estrutura de Pastas do Projeto

Baseada no seu HTML já estruturado, aqui está como organizar o projeto completo:

```
meu-portal/
├── index.html              ← Splash Page (seu arquivo atual)
├── dashboard.html          ← Dashboard Page (seu arquivo atual)
│
├── css/
│   └── styles.css          ← Seu CSS com rem, dvh, variáveis (já existe)
│
├── js/
│   ├── auth.js             ← Lógica de login/logout (este módulo!)
│   ├── dashboard.js        ← Lógica do dashboard
│   └── utils.js            ← Funções utilitárias compartilhadas
│
├── assets/
│   ├── logo.svg
│   ├── favicon.ico
│   ├── google-icon.svg
│   └── icons/
│       ├── notion.svg
│       ├── figma.svg
│       └── ...
│
└── backend/                ← Você vai criar isso no Módulo 4
    ├── package.json
    ├── server.js
    └── ...
```

### 3.3 Inicializando o projeto backend (prévia do Módulo 4)

```bash
# Crie a pasta do backend
mkdir meu-portal-backend
cd meu-portal-backend

# Inicializa o package.json (metadados do projeto Node)
npm init -y

# Você verá um arquivo package.json criado:
# {
#   "name": "meu-portal-backend",
#   "version": "1.0.0",
#   "main": "server.js",
#   ...
# }
```

---

## 4. Variáveis: var, let e const — e por que importa

> **Analogia:** `var` é um post-it colado em qualquer superfície da sala — qualquer um pode ler e apagar. `let` é uma lousa dentro de um cômodo específico — só quem está naquele cômodo acessa. `const` é uma placa de bronze parafusada na parede — não sai mais.

### 4.1 O problema histórico com `var`

```javascript
// ❌ var tem "hoisting" e escopo de função — comportamento confuso
function verificarLogin() {
  if (true) {
    var mensagem = "Bem-vindo!"; // declarado dentro do if
  }
  console.log(mensagem); // "Bem-vindo!" ← FUNCIONA! var "vaza" para fora do bloco
}

// Isso causa bugs silenciosos em código real
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // imprime 3, 3, 3 (não 0, 1, 2!)
  // Razão: var i é compartilhado entre todas as iterações
}
```

**Por que isso importa no seu projeto:**  
Se você usar `var` no código de autenticação, pode ter variáveis "vazando" de escopos e sobrescrevendo valores — em segurança, isso pode ser catastrófico.

### 4.2 `let` e `const` — o padrão moderno

```javascript
// ✅ let: escopo de bloco, pode ser reatribuída
let usuarioLogado = false;
usuarioLogado = true; // OK: reatribuição permitida

// ✅ const: escopo de bloco, NÃO pode ser reatribuída
const API_URL = "https://api.seuportal.com";
// API_URL = "outra url"; // ❌ TypeError: Assignment to constant variable

// ⚠️ Pegadinha importante: const com objetos/arrays
const usuario = { nome: "Henry", logado: false };
usuario.logado = true;  // ✅ FUNCIONA! A referência é constante, não o conteúdo
usuario = {};           // ❌ ERRO! Isso tenta reatribuir a referência
```

**Regra prática para o seu projeto:**

```javascript
// Use const por padrão para TUDO
const btnLogin = document.getElementById('btn-login-google');
const formLogin = document.getElementById('form-login-manual');
const dialogLogout = document.getElementById('dialog-logout');

// Use let APENAS quando precisar reatribuir
let tentativasLogin = 0;
let tokenTemporario = null;

// NUNCA use var em código novo
```

### 4.3 Diferença de comportamento no for loop (relevante para renderizar cards)

```javascript
// ❌ Com var: todas as funções compartilham o mesmo 'i'
const ferramentas = ['Notion', 'Figma', 'GitHub'];
for (var i = 0; i < ferramentas.length; i++) {
  setTimeout(() => console.log(ferramentas[i]), 0);
  // Imprime: undefined, undefined, undefined
  // Porque quando o timeout executa, i já é 3
}

// ✅ Com let: cada iteração tem seu próprio 'i'
for (let i = 0; i < ferramentas.length; i++) {
  setTimeout(() => console.log(ferramentas[i]), 0);
  // Imprime: Notion, Figma, GitHub ✓
}

// ✅ Ainda melhor: use forEach ou for...of
ferramentas.forEach(ferramenta => {
  setTimeout(() => console.log(ferramenta), 0);
  // Imprime: Notion, Figma, GitHub ✓
});
```

---

## 5. Tipos de Dados e Type Coercion

> **Analogia:** Type coercion é como um tradutor automático que às vezes faz traduções que parecem corretas mas têm um significado completamente diferente. Entendê-lo evita surpresas em validações de formulário.

### 5.1 Os 8 tipos primitivos do JS

```javascript
// 1. String — texto
const email = "henry@exemplo.com";
const senha = 'minhasenha123';     // aspas simples também funcionam
const template = `Olá, ${email}`; // template literal — concatena com ${}

// 2. Number — todos os números (inteiros e decimais)
const anoAtual = 2026;
const preco = 29.90;
const infinito = Infinity;
const naoENumero = NaN; // "Not a Number" — resultado de operações inválidas

// 3. Boolean — verdadeiro ou falso
const estaLogado = false;
const emailVerificado = true;

// 4. undefined — variável declarada mas sem valor
let tokenSessao; // undefined
console.log(tokenSessao); // undefined

// 5. null — ausência intencional de valor
const usuarioAtual = null; // "Não há usuário logado"

// 6. BigInt — números inteiros muito grandes
const idUsuario = 9007199254740993n; // note o 'n' no final

// 7. Symbol — identificador único (avançado, você usará no Módulo 7)
const chavePrivada = Symbol('auth_token');

// 8. Object — coleções de dados (arrays são objetos também!)
const usuario = { nome: "Henry", email: "henry@exemplo.com" };
const categorias = ["Produtividade", "Design", "Desenvolvimento"];
```

### 5.2 Type Coercion — o que não te contaram

```javascript
// ===  vs  == : use SEMPRE ===
console.log(0 == false);   // true  ← coerção! 0 vira false
console.log(0 === false);  // false ← sem coerção, tipos diferentes
console.log("" == false);  // true  ← coerção! string vazia vira false
console.log("" === false); // false ← correto

// Coerção em validação de formulário — armadilha real
const inputEmail = ""; // campo de email vazio

// ❌ Validação frágil
if (inputEmail == false) {
  console.log("Email vazio"); // funciona, mas por razão errada
}

// ✅ Validação explícita e clara
if (inputEmail === "" || inputEmail.length === 0) {
  console.log("Email vazio"); // semântica clara
}

// Valores "falsy" do JS (se comportam como false em condicionais):
// false, 0, "", null, undefined, NaN

// Valores "truthy" (todo o resto, incluindo):
// "0", [], {}, "false"  ← strings não-vazias são truthy!
console.log(Boolean("0"));    // true ← "0" é uma string não-vazia!
console.log(Boolean([]));     // true ← array vazio é truthy!
console.log(Boolean({}));     // true ← objeto vazio é truthy!
```

**Aplicação no seu projeto — validação do formulário de login:**

```javascript
// ✅ Validação correta de campos de formulário
function validarFormularioLogin(email, senha) {
  const erros = [];

  // typeof garante que estamos verificando string
  if (typeof email !== 'string' || email.trim() === '') {
    erros.push('Email é obrigatório');
  }

  // Verificação de formato básico
  if (!email.includes('@')) {
    erros.push('Email inválido');
  }

  if (typeof senha !== 'string' || senha.length < 8) {
    erros.push('Senha deve ter no mínimo 8 caracteres');
  }

  return erros; // array vazio = sem erros
}

// Uso:
const erros = validarFormularioLogin('henry@', '123');
if (erros.length > 0) {
  console.log(erros); // ['Email inválido', 'Senha deve ter no mínimo 8 caracteres']
}
```

---

## 6. Funções: Declaração, Expressão e Arrow Functions

> **Analogia:** Uma função é uma receita de bolo. Você escreve a receita uma vez (define a função) e pode fazê-la quantas vezes quiser (chama a função). Arrow functions são a versão pós-it da receita — mais curta, mais direta.

### 6.1 As três formas de criar funções

```javascript
// Forma 1: Function Declaration (hoisted — pode chamar antes de declarar)
function fazerLogin(email, senha) {
  return fetch('/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, senha })
  });
}

// Forma 2: Function Expression (não hoisted — variável recebe a função)
const fazerLogout = function() {
  return fetch('/api/logout', { method: 'POST' });
};

// Forma 3: Arrow Function — sintaxe moderna, mais concisa
const abrirDialogLogout = () => {
  document.getElementById('dialog-logout').showModal();
};

// Arrow function com parâmetros e retorno implícito (sem chaves)
const formatarNomeUsuario = (nome) => nome.trim().toUpperCase();
// Equivale a:
// function formatarNomeUsuario(nome) { return nome.trim().toUpperCase(); }
```

### 6.2 A diferença crucial: `this` em arrow functions

Esta diferença vai importar quando você trabalhar com eventos e autenticação:

```javascript
// Function Declaration/Expression: 'this' depende de COMO a função é chamada
const botaoLogin = {
  texto: "Entrar",
  
  // Function expression: 'this' = o objeto 'botaoLogin' quando chamado como método
  mostrarTexto: function() {
    console.log(this.texto); // "Entrar" ✓
  },
  
  // Arrow function: 'this' = contexto do escopo ONDE foi definida (escopo global aqui)
  mostrarTextoArrow: () => {
    console.log(this.texto); // undefined ← 'this' não é 'botaoLogin'!
  }
};

botaoLogin.mostrarTexto();       // "Entrar"
botaoLogin.mostrarTextoArrow();  // undefined
```

**Regra prática:**

```javascript
// Use arrow functions para:
// ✅ Callbacks de eventos (você não precisa do 'this' do elemento)
const btnLogin = document.getElementById('btn-login-google');
btnLogin.addEventListener('click', () => {
  iniciarLoginGoogle(); // arrow function aqui é perfeito
});

// ✅ Callbacks de arrays (map, filter, forEach)
const nomesFerrams = ferramentas.map(f => f.nome);

// Use function declaration/expression para:
// ✅ Funções principais do módulo
function iniciarLoginGoogle() { /* ... */ }
function validarFormulario(dados) { /* ... */ }

// ✅ Métodos de objetos que precisam do 'this'
const authService = {
  tentativas: 0,
  registrarTentativa: function() {
    this.tentativas++; // 'this' = authService ✓
  }
};
```

### 6.3 Default Parameters e Rest/Spread (ESNext)

```javascript
// Default parameters — parâmetros com valor padrão
function criarCard(nome, descricao = "Sem descrição", categoria = "Geral") {
  return { nome, descricao, categoria };
}

criarCard("Notion");
// { nome: "Notion", descricao: "Sem descrição", categoria: "Geral" }

criarCard("Figma", "Design de interfaces");
// { nome: "Figma", descricao: "Design de interfaces", categoria: "Geral" }

// Rest parameters — captura múltiplos argumentos em array
function logErros(titulo, ...mensagens) {
  console.log(`[${titulo}]`);
  mensagens.forEach(msg => console.log(`  - ${msg}`));
}

logErros("Login falhou", "Email inválido", "Senha muito curta");
// [Login falhou]
//   - Email inválido
//   - Senha muito curta

// Spread operator — "espalha" array/objeto
const configBase = { method: 'POST', mode: 'cors' };
const configLogin = { ...configBase, body: JSON.stringify({ email, senha }) };
// { method: 'POST', mode: 'cors', body: '{"email":...}' }
```

---

## 7. Closures: O Conceito Mais Importante do JS

> **Analogia:** Uma closure é como uma mochila que a função carrega para qualquer lugar. Mesmo que você saia da "sala" onde a mochila foi criada, a função ainda carrega tudo que estava nessa mochila — o conteúdo persiste.

### 7.1 O que é uma Closure

```javascript
// Exemplo fundamental: contador de tentativas de login
function criarContadorTentativas(maximo) {
  // 'tentativas' vive na "mochila" de qualquer função retornada daqui
  let tentativas = 0;

  return {
    incrementar: function() {
      tentativas++;
      return tentativas;
    },
    verificarBloqueio: function() {
      return tentativas >= maximo;
    },
    resetar: function() {
      tentativas = 0;
    }
  };
}

// Cada chamada cria um contador INDEPENDENTE
const contadorLogin = criarContadorTentativas(5);

contadorLogin.incrementar(); // 1
contadorLogin.incrementar(); // 2
contadorLogin.incrementar(); // 3
console.log(contadorLogin.verificarBloqueio()); // false

contadorLogin.incrementar(); // 4
contadorLogin.incrementar(); // 5
console.log(contadorLogin.verificarBloqueio()); // true — conta bloqueada!

// 'tentativas' não é acessível de fora — encapsulamento real!
console.log(contadorLogin.tentativas); // undefined
```

**Por que closures são fundamentais para autenticação:**

A variável `tentativas` está protegida dentro da closure. Código externo não pode alterá-la diretamente — só através das funções expostas. Isso é **encapsulamento** — um dos pilares do código seguro.

### 7.2 Closure no módulo de autenticação do seu projeto

```javascript
// js/auth.js — módulo real do seu projeto usando closure
const authModule = (function() {
  // Estado privado — inacessível de fora do módulo
  let _usuarioAtual = null;
  let _sessaoAtiva = false;
  const MAX_TENTATIVAS = 5;
  let _tentativas = 0;

  // API pública — o que o resto do código pode usar
  return {
    // Verifica se há usuário logado
    estaAutenticado() {
      return _sessaoAtiva && _usuarioAtual !== null;
    },

    // Retorna dados do usuário (sem expor o objeto completo)
    getNomeUsuario() {
      return _usuarioAtual?.nome ?? "Visitante";
    },

    // Registra tentativa de login malsucedida
    registrarFalha() {
      _tentativas++;
      return MAX_TENTATIVAS - _tentativas; // tentativas restantes
    },

    // Limpa o estado de autenticação
    fazerLogout() {
      _usuarioAtual = null;
      _sessaoAtiva = false;
      _tentativas = 0;
    }
  };
})(); // IIFE — executa imediatamente, criando o módulo

// Uso em outros arquivos:
console.log(authModule.estaAutenticado()); // false
console.log(authModule.getNomeUsuario());  // "Visitante"
// authModule._usuarioAtual  // undefined — privacidade garantida pela closure
```

> 💡 **Conexão com o guia de autenticação:** A closure aqui espelha exatamente o que acontece no backend com sessões. No servidor, o estado da sessão fica em Redis/PostgreSQL (inacessível diretamente pelo cliente). Na closure, o estado fica encapsulado (inacessível de fora do módulo). Mesmo princípio, contextos diferentes.

---

## 8. O DOM: Conectando JS ao seu HTML

> **Analogia:** O DOM (Document Object Model) é o mapa ao vivo do seu HTML. Quando você edita o DOM com JavaScript, é como redesenhar o mapa enquanto o usuário está olhando — e ele vê as mudanças em tempo real, sem recarregar a página.

### 8.1 Selecionando elementos do seu HTML

```javascript
// Selecionando elementos do seu HTML semântico
// (referências diretas ao HTML que você já construiu)

// Por ID — retorna UM elemento (mais rápido)
const btnLoginGoogle = document.getElementById('btn-login-google');
const formLogin = document.getElementById('form-login-manual');
const dialogLogout = document.getElementById('dialog-logout');
const campoEmail = document.getElementById('campo-email');
const campoSenha = document.getElementById('campo-senha');

// Por seletor CSS — retorna o PRIMEIRO que combina
const loginSection = document.querySelector('.login-section');
const paginaSplash = document.querySelector('.page--splash');

// Por seletor CSS — retorna TODOS que combinam (NodeList)
const todosCards = document.querySelectorAll('.tool-card');
const linksNavSidebar = document.querySelectorAll('.sidebar-nav__link');

// Percorrendo todos os cards (como você vai fazer no dashboard)
todosCards.forEach((card) => {
  card.addEventListener('mouseenter', () => {
    card.classList.add('tool-card--hover');
  });
});
```

### 8.2 Manipulando classes e atributos

```javascript
// classList — método mais importante para seu projeto
const body = document.body;

// Trocar a splash page para o dashboard após login
body.classList.remove('page--splash');
body.classList.add('page--dashboard');

// Ou de forma mais elegante (como o seu HTML sugere):
body.classList.replace('page--splash', 'page--dashboard');

// toggle — adiciona se não tem, remove se tem
const sidebar = document.querySelector('.app-sidebar');
sidebar.classList.toggle('app-sidebar--collapsed'); // para menu hambúrguer futuro

// Verificar se tem uma classe
if (body.classList.contains('page--dashboard')) {
  carregarDadosDashboard();
}

// Manipulando atributos — para acessibilidade
const btnLogin = document.getElementById('btn-login-google');

// Desabilitar botão durante carregamento
btnLogin.disabled = true;
btnLogin.setAttribute('aria-busy', 'true');

// Reabilitar após resposta
btnLogin.disabled = false;
btnLogin.removeAttribute('aria-busy');
```

### 8.3 Criando elementos dinamicamente

Quando você carregar as ferramentas da API no dashboard, vai criar cards dinamicamente:

```javascript
// Função que cria um tool-card a partir de dados (como virá da API)
function criarToolCard(ferramenta) {
  // { nome: "Notion", descricao: "Notas e wikis", url: "https://notion.so", icone: "notion.svg" }

  // Cria o <li>
  const li = document.createElement('li');
  li.className = 'tools-grid__item';

  // Usa innerHTML com template literal — eficiente para HTML complexo
  li.innerHTML = `
    <article class="tool-card tool-card--${ferramenta.categoria}">
      <div class="tool-card__icon-wrap">
        <img 
          src="assets/icons/${ferramenta.icone}" 
          alt="" 
          aria-hidden="true"
          class="tool-card__icon"
          width="32" 
          height="32"
          loading="lazy"
        >
      </div>
      <div class="tool-card__body">
        <h3 class="tool-card__name">
          <a 
            href="${ferramenta.url}" 
            target="_blank" 
            rel="noopener noreferrer"
            class="tool-card__link"
            aria-label="Abrir ${ferramenta.nome} em nova aba"
          >
            ${ferramenta.nome}
          </a>
        </h3>
        <p class="tool-card__description">
          ${ferramenta.descricao}
        </p>
      </div>
    </article>
  `;

  return li;
}

// Renderiza uma lista de ferramentas no grid
function renderizarFerramentas(ferramentas, containerSelector) {
  const grid = document.querySelector(containerSelector);
  if (!grid) return; // guard clause — sai se o elemento não existe

  // Limpa o grid antes de renderizar (evita duplicatas)
  grid.innerHTML = '';

  // Adiciona cada card ao grid
  ferramentas.forEach(ferramenta => {
    const card = criarToolCard(ferramenta);
    grid.appendChild(card);
  });

  // Atualiza o contador na seção
  const count = grid.closest('.tool-section')?.querySelector('.tool-section__count');
  if (count) {
    count.textContent = `${ferramentas.length} ferramentas`;
  }
}

// Uso:
const ferramentasProdutividade = [
  { nome: "Notion", descricao: "Notas e wikis", url: "https://notion.so", icone: "notion.svg", categoria: "produtividade" },
  { nome: "Todoist", descricao: "Listas de tarefas", url: "https://todoist.com", icone: "todoist.svg", categoria: "produtividade" },
];

renderizarFerramentas(ferramentasProdutividade, '#produtividade .tools-grid');
```

---

## 9. Eventos: Fazendo o Botão de Login Funcionar

> **Analogia:** `addEventListener` é como instalar uma campainha na porta. Você define qual "campainha" (tipo de evento), em qual "porta" (elemento), e o que acontece quando alguém "toca" (a função handler).

### 9.1 O padrão correto de event listeners

```javascript
// ❌ Forma antiga — difícil de remover, mistura HTML e JS
// <button onclick="fazerLogin()">Entrar</button>

// ✅ Forma moderna — separação de responsabilidades
const btn = document.getElementById('btn-login-google');
btn.addEventListener('click', fazerLogin);

// Para remover (necessário em SPAs para evitar memory leaks):
btn.removeEventListener('click', fazerLogin);
```

### 9.2 O Event Object — informações sobre o evento

```javascript
// O event listener recebe automaticamente o objeto de evento
document.getElementById('form-login-manual').addEventListener('submit', (event) => {
  // event.preventDefault() — ESSENCIAL para formulários!
  // Sem isso, o browser faz submit padrão (recarrega a página)
  event.preventDefault();

  // event.target — o elemento que disparou o evento (o form)
  const formulario = event.target;

  // FormData — forma moderna de pegar dados do formulário
  const dados = new FormData(formulario);
  const email = dados.get('email');
  const senha = dados.get('senha');

  // Agora você pode validar e enviar para o backend
  processarLogin({ email, senha });
});
```

### 9.3 Event Delegation — um padrão essencial para o dashboard

Em vez de adicionar um listener em cada card (200+ cards!), adicione um no pai e detecte qual filho foi clicado:

```javascript
// ❌ Problemático: adiciona 200+ listeners
document.querySelectorAll('.tool-card__link').forEach(link => {
  link.addEventListener('click', registrarAcesso);
});

// ✅ Eficiente: um único listener no pai
document.querySelector('.dashboard-main').addEventListener('click', (event) => {
  // Verifica se o clique foi em um link de tool card
  const link = event.target.closest('.tool-card__link');
  if (!link) return; // clique em outro elemento, ignora

  // Registra o acesso (para a seção "Recentes" da sidebar)
  const nomeFerramenta = link.textContent.trim();
  registrarAcessoRecente(nomeFerramenta);
  // Não previne o default aqui — queremos que o link abra normalmente
});
```

---

## 10. Modules (ESModules): Organizando o Código do Projeto

> **Analogia:** Modules são como gavetas organizadas em um escritório. Em vez de jogar todos os papéis na mesa (um arquivo JS gigante), você os organiza em gavetas temáticas (arquivos separados) e pega exatamente o que precisa de cada gaveta quando precisar.

### 10.1 Sintaxe de import/export

```javascript
// === js/utils.js ===
// Exportações nomeadas — pode ter várias por arquivo
export function formatarEmail(email) {
  return email.toLowerCase().trim();
}

export function validarEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

export const MAX_TENTATIVAS_LOGIN = 5;

// Exportação default — apenas uma por arquivo
export default function inicializarApp() {
  console.log('Portal de Ferramentas iniciado');
}
```

```javascript
// === js/auth.js ===
// Importação de exportações nomeadas (use as chaves)
import { formatarEmail, validarEmail, MAX_TENTATIVAS_LOGIN } from './utils.js';

// Importação do default (sem chaves, nome à sua escolha)
import inicializarApp from './utils.js';

// Importação mista
import inicializarApp, { formatarEmail } from './utils.js';

// Importar tudo como namespace
import * as Utils from './utils.js';
Utils.formatarEmail('HENRY@EXEMPLO.COM');
```

### 10.2 Como conectar ao seu HTML

No seu HTML, você já tem:

```html
<!-- No seu splash.html -->
<script src="js/auth.js" defer></script>
```

Para usar ESModules, mude para:

```html
<!-- ✅ type="module" habilita import/export -->
<script type="module" src="js/auth.js"></script>
<!-- Nota: com type="module", o defer é automático -->
```

### 10.3 Estrutura real dos arquivos JS do projeto

```javascript
// === js/utils.js ===
export const API_BASE_URL = 'https://api.seuportal.com';

export function mostrarErro(elementoId, mensagem) {
  const elemento = document.getElementById(elementoId);
  if (elemento) {
    elemento.textContent = mensagem;
    elemento.setAttribute('role', 'alert');
  }
}

export function limparErro(elementoId) {
  const elemento = document.getElementById(elementoId);
  if (elemento) {
    elemento.textContent = '';
    elemento.removeAttribute('role');
  }
}

export function validarEmail(email) {
  return typeof email === 'string' && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

export function validarSenha(senha) {
  return typeof senha === 'string' && senha.length >= 8;
}
```

---

## 11. Integração com seus Guias de HTML e CSS

Esta seção conecta o JS que você está aprendendo diretamente ao HTML e CSS que você já tem.

### 11.1 Controlando classes CSS com JS

```javascript
// === Controlar o estado das páginas (Splash ↔ Dashboard) ===
// Referência: body.classList no seu HTML

function mostrarDashboard() {
  document.body.classList.replace('page--splash', 'page--dashboard');
}

function mostrarSplash() {
  document.body.classList.replace('page--dashboard', 'page--splash');
}

// === Navegação da sidebar — estado ativo do link ===
// Referência: .sidebar-nav__link--active no seu CSS

function ativarLinkNav(href) {
  // Remove ativo de todos
  document.querySelectorAll('.sidebar-nav__link').forEach(link => {
    link.classList.remove('sidebar-nav__link--active');
  });

  // Adiciona no link correspondente
  const linkAtivo = document.querySelector(`.sidebar-nav__link[href="${href}"]`);
  linkAtivo?.classList.add('sidebar-nav__link--active');
}

// === Feedback visual no botão durante carregamento ===
// Referência: .btn, .btn--primary no seu CSS

function setBotaoCarregando(btnId, carregando) {
  const btn = document.getElementById(btnId);
  if (!btn) return;

  btn.disabled = carregando;
  
  if (carregando) {
    btn.dataset.textoOriginal = btn.textContent;
    btn.textContent = 'Carregando...';
    btn.style.opacity = '0.7';
  } else {
    btn.textContent = btn.dataset.textoOriginal;
    btn.style.opacity = '';
  }
}
```

### 11.2 O `<dialog>` nativo — conectando HTML com JS

Seu HTML já tem o `<dialog id="dialog-logout">` construído. Veja como o JS o controla:

```javascript
// Referências do seu HTML
const dialogLogout = document.getElementById('dialog-logout');
const btnAbrirDialog = document.getElementById('btn-logout');
const btnCancelar = document.getElementById('btn-cancelar-logout');
const btnConfirmar = document.getElementById('btn-confirmar-logout');
const btnFechar = document.getElementById('btn-fechar-dialog');

// Abrir o modal
btnAbrirDialog.addEventListener('click', () => {
  dialogLogout.showModal();
  // showModal() também adiciona backdrop automático e bloqueia foco no dialog
});

// Cancelar — fecha sem ação
btnCancelar.addEventListener('click', () => {
  dialogLogout.close(); // fecha o dialog nativamente
});

// Fechar pelo X
btnFechar.addEventListener('click', () => {
  dialogLogout.close();
});

// Confirmar logout
btnConfirmar.addEventListener('click', async () => {
  dialogLogout.close();
  await fazerLogout(); // você vai implementar isso no Módulo 4
});

// Fechar ao clicar no backdrop (fora do dialog)
dialogLogout.addEventListener('click', (event) => {
  // event.target === dialog = clicou no backdrop
  if (event.target === dialogLogout) {
    dialogLogout.close();
  }
});

// Fechar com Escape (já é nativo, mas você pode interceptar para analytics)
dialogLogout.addEventListener('cancel', (event) => {
  console.log('Dialog fechado com Escape');
  // event.preventDefault() se quiser impedir o fechar
});
```

---

## 12. Conexão com Autenticação: O que o JS faz no Login

Esta seção conecta o JS diretamente ao seu **Guia Completo de Autenticação**.

### 12.1 O fluxo completo pelo ângulo do JS

```
Usuário clica "Entrar com Google"
         │
         ▼ JS no Frontend
[1] Event listener captura o clique
[2] JS redireciona para: 
    https://accounts.google.com/o/oauth2/auth?
      client_id=SEU_CLIENT_ID
      &redirect_uri=https://seusite.com/auth/callback
      &scope=openid email profile
      &response_type=code
      &state=<valor aleatório gerado pelo backend>
         │
         ▼ Google autentica o usuário
[3] Google redireciona de volta:
    https://seusite.com/auth/callback?code=AUTH_CODE&state=<mesmo valor>
         │
         ▼ JS no Frontend (na página de callback)
[4] JS lê o `code` da URL
[5] JS envia o code para SEU BACKEND (não diretamente ao Google!)
         │
         ▼ Backend troca o code pelo token (com client_secret)
[6] Backend cria sessão no banco de dados
[7] Backend seta cookie HttpOnly na resposta
         │
         ▼ JS no Frontend
[8] JS recebe confirmação de sucesso
[9] JS chama mostrarDashboard() → troca página--splash por page--dashboard
[10] JS carrega as ferramentas do usuário
```

### 12.2 Código JS do fluxo de login

```javascript
// === js/auth.js ===
import { API_BASE_URL, mostrarErro, limparErro, validarEmail, validarSenha } from './utils.js';

// ─── LOGIN COM GOOGLE ────────────────────────────────────────────────────────

document.getElementById('btn-login-google')?.addEventListener('click', () => {
  // O backend fornece a URL correta com client_id e state já configurados
  // Você NUNCA constrói essa URL no frontend com seus segredos
  window.location.href = `${API_BASE_URL}/auth/google`;
  // O backend vai redirecionar para o Google com os parâmetros corretos
});

// ─── LOGIN COM EMAIL/SENHA ───────────────────────────────────────────────────

document.getElementById('form-login-manual')?.addEventListener('submit', async (event) => {
  event.preventDefault();

  const email = document.getElementById('campo-email').value;
  const senha = document.getElementById('campo-senha').value;

  // Limpa erros anteriores (referência ao seu HTML: #email-erro, #senha-erro)
  limparErro('email-erro');
  limparErro('senha-erro');

  // Validação no frontend (rápida, antes de bater no servidor)
  let temErro = false;
  if (!validarEmail(email)) {
    mostrarErro('email-erro', 'Digite um email válido');
    temErro = true;
  }
  if (!validarSenha(senha)) {
    mostrarErro('senha-erro', 'Senha deve ter ao menos 8 caracteres');
    temErro = true;
  }
  if (temErro) return; // não prossegue se há erros

  // Feedback visual — desabilita botão durante a requisição
  const btnSubmit = event.target.querySelector('[type="submit"]');
  btnSubmit.disabled = true;
  btnSubmit.textContent = 'Entrando...';

  try {
    // Envia para o backend (você vai construir essa rota no Módulo 4)
    const resposta = await fetch(`${API_BASE_URL}/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include', // ESSENCIAL: envia e recebe cookies HttpOnly
      body: JSON.stringify({ email, senha })
    });

    if (!resposta.ok) {
      // Backend retornou erro (401, 400, etc.)
      const dados = await resposta.json();
      // Sempre mensagem genérica — nunca diga qual campo está errado
      // (anti-enumeração, como você aprendeu no guia de auth)
      mostrarErro('email-erro', dados.mensagem || 'Email ou senha inválidos');
      return;
    }

    // Login bem-sucedido!
    const dados = await resposta.json();
    
    // Atualiza o nome do usuário na UI (referência: .user-name no dashboard)
    const spanNome = document.querySelector('.user-name');
    if (spanNome) spanNome.textContent = dados.usuario.nome;

    // Transição para o dashboard
    mostrarDashboard();
    
  } catch (erro) {
    // Erro de rede ou problema inesperado
    mostrarErro('email-erro', 'Erro de conexão. Tente novamente.');
    console.error('Erro no login:', erro);
  } finally {
    // Sempre reabilita o botão, mesmo que dê erro
    btnSubmit.disabled = false;
    btnSubmit.textContent = 'Entrar';
  }
});

// ─── LOGOUT ──────────────────────────────────────────────────────────────────

async function fazerLogout() {
  try {
    await fetch(`${API_BASE_URL}/auth/logout`, {
      method: 'POST',
      credentials: 'include' // envia o cookie de sessão para o backend invalidar
    });
  } catch (erro) {
    console.error('Erro ao fazer logout:', erro);
  } finally {
    // Limpa a UI independente do resultado
    mostrarSplash();
  }
}

// ─── VERIFICAR SESSÃO AO CARREGAR A PÁGINA ───────────────────────────────────

async function verificarSessaoAtiva() {
  try {
    const resposta = await fetch(`${API_BASE_URL}/auth/me`, {
      credentials: 'include' // envia o cookie de sessão
    });

    if (resposta.ok) {
      const dados = await resposta.json();
      // Já tem sessão ativa — vai direto pro dashboard
      const spanNome = document.querySelector('.user-name');
      if (spanNome) spanNome.textContent = dados.usuario.nome;
      mostrarDashboard();
    }
    // Se não ok (401), fica na splash — comportamento padrão
  } catch (erro) {
    // Erro de rede — fica na splash
  }
}

// Executa ao carregar a página
verificarSessaoAtiva();
```

---

## 13. Código Ready-to-Run: Splash Page com JS

Este é o código completo e funcional para conectar o JavaScript ao seu HTML atual. Copie, cole e rode.

### Pré-requisito: estrutura de arquivos

```
meu-portal/
├── index.html     ← seu HTML da splash (já existe)
├── css/
│   └── styles.css ← seu CSS (já existe)
└── js/
    ├── utils.js   ← crie este arquivo
    └── auth.js    ← crie este arquivo
```

### js/utils.js

```javascript
// ============================================================
// ARQUIVO: js/utils.js
// PROJETO: Portal de Ferramentas — Henry Hirth
// Funções utilitárias compartilhadas
// ============================================================

// URL base da API (quando o backend estiver pronto no Módulo 4)
// Por enquanto, simulamos localmente
export const API_BASE_URL = 'http://localhost:3000';

/**
 * Exibe uma mensagem de erro no elemento de feedback do formulário
 * @param {string} elementoId - ID do span de erro (ex: 'email-erro')
 * @param {string} mensagem - Texto a exibir
 */
export function mostrarErro(elementoId, mensagem) {
  const elemento = document.getElementById(elementoId);
  if (!elemento) return;
  elemento.textContent = mensagem;
}

/**
 * Remove a mensagem de erro de um campo
 */
export function limparErro(elementoId) {
  const elemento = document.getElementById(elementoId);
  if (!elemento) return;
  elemento.textContent = '';
}

/**
 * Valida formato básico de email
 */
export function validarEmail(email) {
  if (typeof email !== 'string') return false;
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.trim());
}

/**
 * Valida comprimento mínimo de senha
 */
export function validarSenha(senha) {
  return typeof senha === 'string' && senha.length >= 8;
}

/**
 * Troca a "tela" atual (splash ↔ dashboard)
 * Usa as classes BEM do seu HTML
 */
export function trocarTela(de, para) {
  document.body.classList.replace(`page--${de}`, `page--${para}`);
}
```

### js/auth.js

```javascript
// ============================================================
// ARQUIVO: js/auth.js
// PROJETO: Portal de Ferramentas — Henry Hirth
// Lógica de autenticação do frontend
// ============================================================

import { mostrarErro, limparErro, validarEmail, validarSenha, trocarTela } from './utils.js';

// ─── INICIALIZAÇÃO ────────────────────────────────────────────────────────────

// document.readyState garante que o DOM está carregado
// (equivalente ao DOMContentLoaded, mas mais robusto com modules)
document.addEventListener('DOMContentLoaded', inicializar);

function inicializar() {
  configurarFormularioLogin();
  configurarBotaoGoogle();
  configurarModalLogout();
  // Futuramente: verificarSessaoAtiva();
}

// ─── FORMULÁRIO DE LOGIN ──────────────────────────────────────────────────────

function configurarFormularioLogin() {
  const form = document.getElementById('form-login-manual');
  if (!form) return; // guard clause: sai se o elemento não existe na página

  form.addEventListener('submit', async (event) => {
    event.preventDefault(); // ESSENCIAL: impede reload da página

    const email = document.getElementById('campo-email')?.value ?? '';
    const senha = document.getElementById('campo-senha')?.value ?? '';
    const btnSubmit = form.querySelector('[type="submit"]');

    // 1. Limpa erros anteriores
    limparErro('email-erro');
    limparErro('senha-erro');

    // 2. Validação no frontend
    let formValido = true;

    if (!validarEmail(email)) {
      mostrarErro('email-erro', 'Digite um email válido');
      formValido = false;
    }

    if (!validarSenha(senha)) {
      mostrarErro('senha-erro', 'Senha deve ter ao menos 8 caracteres');
      formValido = false;
    }

    if (!formValido) return;

    // 3. Feedback: desabilita botão
    if (btnSubmit) {
      btnSubmit.disabled = true;
      btnSubmit.textContent = 'Entrando...';
    }

    try {
      // 4. Simula login (troque pela chamada real no Módulo 4)
      await simularDelayRede(1500);

      // SIMULAÇÃO: credenciais demo
      if (email === 'demo@portal.com' && senha === 'demo1234') {
        console.log('✅ Login simulado com sucesso!');
        trocarTela('splash', 'dashboard');
      } else {
        // Mensagem genérica — nunca diga qual campo está errado
        mostrarErro('email-erro', 'Email ou senha inválidos');
      }
    } catch (erro) {
      mostrarErro('email-erro', 'Erro de conexão. Tente novamente.');
      console.error('[auth.js] Erro no login:', erro);
    } finally {
      // 5. Sempre reabilita o botão
      if (btnSubmit) {
        btnSubmit.disabled = false;
        btnSubmit.textContent = 'Entrar';
      }
    }
  });
}

// ─── BOTÃO GOOGLE ─────────────────────────────────────────────────────────────

function configurarBotaoGoogle() {
  const btn = document.getElementById('btn-login-google');
  if (!btn) return;

  btn.addEventListener('click', async () => {
    btn.disabled = true;
    
    const textoOriginal = btn.querySelector('.btn__text')?.textContent;
    const spanTexto = btn.querySelector('.btn__text');
    if (spanTexto) spanTexto.textContent = 'Redirecionando...';

    try {
      // Simula redirecionamento para Google (Módulo 4 implementará de verdade)
      await simularDelayRede(800);
      alert('🔐 Login com Google será implementado no Módulo 4 com OAuth 2.0!\nPor agora, use: demo@portal.com / demo1234');
    } finally {
      btn.disabled = false;
      if (spanTexto && textoOriginal) spanTexto.textContent = textoOriginal;
    }
  });
}

// ─── MODAL DE LOGOUT ──────────────────────────────────────────────────────────

function configurarModalLogout() {
  const dialogLogout = document.getElementById('dialog-logout');
  const btnAbrirLogout = document.getElementById('btn-logout');
  const btnCancelar = document.getElementById('btn-cancelar-logout');
  const btnConfirmar = document.getElementById('btn-confirmar-logout');
  const btnFechar = document.getElementById('btn-fechar-dialog');

  // Abre o modal
  btnAbrirLogout?.addEventListener('click', () => {
    dialogLogout?.showModal();
  });

  // Cancela
  btnCancelar?.addEventListener('click', () => {
    dialogLogout?.close();
  });

  // Fecha pelo X
  btnFechar?.addEventListener('click', () => {
    dialogLogout?.close();
  });

  // Confirma logout
  btnConfirmar?.addEventListener('click', async () => {
    dialogLogout?.close();
    
    // Simula logout (Módulo 4 enviará POST /api/logout)
    await simularDelayRede(300);
    trocarTela('dashboard', 'splash');
    console.log('✅ Logout realizado');
  });

  // Fecha ao clicar no backdrop
  dialogLogout?.addEventListener('click', (event) => {
    if (event.target === dialogLogout) {
      dialogLogout.close();
    }
  });
}

// ─── UTILITÁRIO INTERNO ───────────────────────────────────────────────────────

/**
 * Simula delay de rede para testar feedback visual
 * REMOVA no código de produção — substitua pelas chamadas fetch reais
 */
function simularDelayRede(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### Atualização no seu index.html

Adicione no `<head>` (ou substitua o script existente):

```html
<!-- ANTES (remova): -->
<script src="js/auth.js" defer></script>

<!-- DEPOIS (adicione): -->
<script type="module" src="js/auth.js"></script>
```

### Como testar

```bash
# Opção 1: VS Code com Live Server (extensão recomendada)
# Clique direito no index.html → "Open with Live Server"

# Opção 2: linha de comando com npx
npx serve .
# Acesse: http://localhost:3000

# Credenciais de demo:
# Email: demo@portal.com
# Senha: demo1234
```

---

## 14. Desafios Práticos

### 🟢 Desafio 1 — Fácil: Validador de formulário

**Objetivo:** Criar uma função `validarFormularioCompleto` que valida todos os campos do formulário de login antes de enviar.

**Critérios de sucesso:**
- Função aceita `{ email, senha }` como parâmetro
- Retorna um objeto `{ valido: boolean, erros: { email?: string, senha?: string } }`
- Email inválido: `"Digite um email válido"`
- Senha < 8 chars: `"Senha deve ter ao menos 8 caracteres"`
- Ambos vazios: retorna os dois erros
- Tudo ok: retorna `{ valido: true, erros: {} }`

```javascript
// Teste esperado:
validarFormularioCompleto({ email: '', senha: '' })
// { valido: false, erros: { email: 'Digite um email válido', senha: 'Senha deve ter ao menos 8 caracteres' } }

validarFormularioCompleto({ email: 'henry@exemplo.com', senha: 'minhasenha' })
// { valido: true, erros: {} }
```

---

### 🟡 Desafio 2 — Médio: Contador de tentativas com closure

**Objetivo:** Implementar um sistema de bloqueio de login após 3 tentativas falhas, usando closure para proteger o estado.

**Critérios de sucesso:**
- Função `criarSistemaDeLogin(maxTentativas)` retorna um objeto com métodos
- Método `tentarLogin(email, senha)` — retorna `{ sucesso: boolean, mensagem: string, tentativasRestantes?: number }`
- Após `maxTentativas` falhas: retorna `{ bloqueado: true, mensagem: "Conta bloqueada por 5 minutos" }`
- Método `resetar()` — reseta o contador
- O estado interno (contagem) não pode ser acessado diretamente de fora

```javascript
// Teste esperado:
const loginSistema = criarSistemaDeLogin(3);

loginSistema.tentarLogin('x@x.com', 'senhaerrada');
// { sucesso: false, mensagem: "Email ou senha inválidos", tentativasRestantes: 2 }

loginSistema.tentarLogin('x@x.com', 'senhaerrada');
// { sucesso: false, mensagem: "Email ou senha inválidos", tentativasRestantes: 1 }

loginSistema.tentarLogin('x@x.com', 'senhaerrada');
// { bloqueado: true, mensagem: "Conta bloqueada por 5 minutos" }
```

---

### 🔴 Desafio 3 — Avançado: Renderizador dinâmico de cards

**Objetivo:** Criar a função `renderizarSecaoFerramentas(dadosSecao, containerSelector)` que gera uma seção completa do dashboard dinamicamente a partir de dados.

**Dados de entrada:**
```javascript
const secaoProdutividade = {
  id: 'produtividade',
  titulo: 'Produtividade',
  iconeCategoria: 'assets/icons/category-produtividade.svg',
  ferramentas: [
    { nome: 'Notion', descricao: 'Notas e wikis', url: 'https://notion.so', icone: 'notion.svg' },
    { nome: 'Todoist', descricao: 'Listas de tarefas', url: 'https://todoist.com', icone: 'todoist.svg' },
  ]
};
```

**Critérios de sucesso:**
- Gera HTML que corresponde exatamente ao padrão do seu arquivo `dashboard.html`
- Usa `document.createElement` e `innerHTML` (não `document.write`)
- Inclui os atributos de acessibilidade (`aria-labelledby`, `aria-label`, `aria-hidden`)
- Atualiza o `<span class="tool-section__count">` com a contagem correta
- A função é uma arrow function que retorna o elemento criado (não o insere no DOM — quem chama que insere)

---

## 15. Checklist e Próximo Módulo

### ✅ O que você aprendeu neste módulo

- [ ] Diferença entre JS no browser e no servidor (Node.js/Bun/Deno)
- [ ] `const` vs `let` vs `var` — escopo de bloco e por que `var` é problemático
- [ ] Os 8 tipos primitivos e o perigo da type coercion em validações
- [ ] Function declarations, expressions e arrow functions — quando usar cada uma
- [ ] `this` em arrow functions vs functions normais
- [ ] Closures — encapsulamento e proteção de estado privado
- [ ] DOM API — `querySelector`, `querySelectorAll`, `classList`, `createElement`
- [ ] `addEventListener` e Event Delegation para performance
- [ ] ESModules — `import`/`export` para organizar o projeto
- [ ] Como o JS conecta ao seu HTML semântico e ao sistema de classes CSS
- [ ] O papel do JS em cada etapa do fluxo de autenticação (OAuth, sessões, cookies)
- [ ] `fetch` com `credentials: 'include'` para cookies HttpOnly

### 📊 Progresso no Roadmap

```
Módulo 1: Core JS           ████░░░░░░  Em andamento
Módulo 2: Assincronismo     ░░░░░░░░░░  Próximo
Módulo 3: Frontend Mastery  ░░░░░░░░░░  Bloqueado
Módulo 4: Backend Mastery   ░░░░░░░░░░  Bloqueado
Módulo 5: Segurança         ░░░░░░░░░░  Bloqueado
Módulo 6: Performance       ░░░░░░░░░░  Bloqueado
Módulo 7: Arquitetura       ░░░░░░░░░░  Bloqueado
Módulo 8: Projeto Final     ░░░░░░░░░░  Bloqueado
```

### 🚀 O que vem no Módulo 2: Assincronismo Avançado

No próximo módulo você vai desvendar o mecanismo mais importante do JS moderno: o **Event Loop**. Você vai entender por que `fetch` retorna uma Promise, como `async/await` torna código assíncrono legível como código síncrono, e por que o JS consegue fazer várias coisas "ao mesmo tempo" sendo single-threaded.

Mais importante: você vai implementar o `fetch` real para o endpoint de login, lidando com todos os estados possíveis (loading, success, error, timeout). Essa é a ponte entre o JS que você aprendeu aqui e a autenticação real com backend.

---

> **Responda com seu código dos desafios ou 'PRÓXIMO' para avançar para o Módulo 2: Assincronismo Avançado.**
