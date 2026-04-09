# Guia Completo de HTML5 Semântico
### Para o seu projeto: Splash Page + Dashboard de Ferramentas

---

## Índice

1. [Tags Semânticas Essenciais](#1-tags-semânticas-essenciais)
2. [Arquitetura Lógica do Site](#2-arquitetura-lógica-do-site)
3. [Preparação para CSS e Comportamento](#3-preparação-para-css-e-comportamento)
4. [Recursos Fundamentais do HTML5](#4-recursos-fundamentais-do-html5)
5. [Exercício Prático Guiado](#5-exercício-prático-guiado)
6. [Boas Práticas e Erros Comuns](#6-boas-práticas-e-erros-comuns)

---

## 1. Tags Semânticas Essenciais

### O que é HTML semântico?

HTML semântico significa usar tags que **descrevem o significado do conteúdo**, não apenas sua aparência. A diferença entre `<div>` e `<main>` não é visual — é de **significado**. Isso beneficia:

- **Leitores de tela** (acessibilidade para pessoas com deficiência visual)
- **Motores de busca** (SEO)
- **Você mesmo** (código mais fácil de ler e manter)
- **JavaScript** (seletores mais expressivos)

---

### As 7 tags estruturais essenciais

#### `<header>` — Cabeçalho de uma seção ou da página

Contém elementos introdutórios: logotipo, título principal, subtítulo, barra de busca, avatar do usuário. **Pode aparecer mais de uma vez** na página (um para a `<main>`, outro para um `<article>`), mas normalmente há um por página.

```html
<!-- No seu splash: cabeçalho simples com logo -->
<header class="splash__header">
  <img src="logo.svg" alt="Meu Dashboard" class="logo">
  <h1>Meu Dashboard</h1>
</header>

<!-- No seu dashboard: cabeçalho com nome do usuário -->
<header class="app-header">
  <img src="logo.svg" alt="Meu Dashboard" class="logo">
  <h1>Meu Dashboard</h1>
  <div class="user-info">
    <img src="avatar.jpg" alt="Foto de perfil de João">
    <span>João Silva</span>
  </div>
</header>
```

---

#### `<nav>` — Menu de navegação

Use **apenas para conjuntos de links de navegação** (menu principal, abas, breadcrumbs). Não use para todos os grupos de links — só para os que ajudam o usuário a navegar entre seções.

```html
<!-- No dashboard: menu de atalhos por categoria -->
<nav class="topics-nav" aria-label="Categorias de ferramentas">
  <ul>
    <li><a href="#produtividade">Produtividade</a></li>
    <li><a href="#design">Design</a></li>
    <li><a href="#desenvolvimento">Desenvolvimento</a></li>
    <li><a href="#comunicacao">Comunicação</a></li>
  </ul>
</nav>
```

> ⚠️ O `aria-label` é importante quando há mais de um `<nav>` na página. Ele distingue "menu principal" de "menu de categorias", por exemplo.

---

#### `<main>` — Conteúdo principal

Deve haver **exatamente um `<main>` por página**. Contém o conteúdo único daquela tela — tudo que não é header, footer ou sidebar repetido. Leitores de tela usam esta tag para pular direto ao conteúdo.

```html
<!-- No splash: o conteúdo principal é o formulário/botão de login -->
<main class="splash__main">
  <section class="login-area">
    <!-- formulário ou botão de login -->
  </section>
</main>

<!-- No dashboard: o conteúdo principal são as seções de ferramentas -->
<main class="dashboard__main" id="conteudo-principal">
  <section id="produtividade"> ... </section>
  <section id="design"> ... </section>
</main>
```

---

#### `<section>` — Seção temática

Agrupa conteúdo com um **tema em comum**. Cada `<section>` normalmente tem seu próprio título (`<h2>`, `<h3>`, etc.). No seu dashboard, cada categoria de ferramentas é uma `<section>`.

```html
<section id="produtividade" class="tool-section" aria-labelledby="titulo-produtividade">
  <h2 id="titulo-produtividade">Produtividade</h2>
  <ul class="tools-list">
    <!-- links de ferramentas -->
  </ul>
</section>
```

> 💡 O par `aria-labelledby` + `id` no título vincula a seção ao seu rótulo — essencial para acessibilidade.

---

#### `<article>` — Conteúdo independente e autocontido

Use quando o conteúdo **faz sentido sozinho**, fora do contexto da página (um card de ferramenta, um post, uma notícia). No seu dashboard, cada card de ferramenta poderia ser um `<article>`.

```html
<article class="tool-card">
  <h3>Notion</h3>
  <p>Notas, wikis e gerenciamento de projetos.</p>
  <a href="https://notion.so" target="_blank" rel="noopener noreferrer">
    Abrir ferramenta
  </a>
</article>
```

> 🤔 **`<section>` vs `<article>`**: Se o bloco precisa do contexto da página para fazer sentido → `<section>`. Se pode ser "recortado e colado" em qualquer lugar e ainda fazer sentido → `<article>`.

---

#### `<aside>` — Conteúdo lateral ou complementar

Conteúdo relacionado, mas não essencial ao fluxo principal. No seu dashboard, pode ser uma barra de ferramentas recentes, atalhos rápidos ou um painel de anúncios.

```html
<aside class="sidebar" aria-label="Ferramentas recentes">
  <h2>Acessados Recentemente</h2>
  <ul>
    <li><a href="https://figma.com">Figma</a></li>
    <li><a href="https://github.com">GitHub</a></li>
  </ul>
</aside>
```

---

#### `<footer>` — Rodapé

Informações de encerramento da página ou de uma seção: copyright, links de suporte, créditos. Assim como `<header>`, pode aparecer dentro de `<article>` ou `<section>`.

```html
<footer class="app-footer">
  <p>
    <small>&copy; 2025 Meu Dashboard — 
      <a href="/privacidade">Política de Privacidade</a>
    </small>
  </p>
</footer>
```

---

### Tags semânticas complementares relevantes para o seu projeto

#### `<figure>` e `<figcaption>` — Imagem com legenda

```html
<!-- No splash: logo com descrição -->
<figure class="splash__logo-figure">
  <img src="logo.svg" alt="Logo do Meu Dashboard">
  <figcaption>Seu portal pessoal de ferramentas</figcaption>
</figure>
```

#### `<time>` — Datas e horários legíveis por máquinas

```html
<!-- Útil no dashboard para mostrar "última atualização" -->
<p>Última atualização: <time datetime="2025-06-15">15 de junho de 2025</time></p>
```

#### `<details>` e `<summary>` — Conteúdo expansível (sem JavaScript!)

Perfeito para categorias que podem ser colapsadas no dashboard, nativamente no HTML:

```html
<details class="tool-section--collapsible">
  <summary>Design (6 ferramentas)</summary>
  <ul class="tools-list">
    <li><a href="https://figma.com">Figma</a></li>
    <li><a href="https://coolors.co">Coolors</a></li>
  </ul>
</details>
```

#### `<dialog>` — Janela modal nativa

Para confirmações, alertas ou formulários popup — sem bibliotecas externas:

```html
<!-- Diálogo de logout ou confirmação -->
<dialog id="dialog-logout" aria-labelledby="dialog-titulo">
  <h2 id="dialog-titulo">Sair da conta?</h2>
  <p>Você tem certeza que deseja sair?</p>
  <menu>
    <button type="button" id="btn-cancelar">Cancelar</button>
    <button type="button" id="btn-confirmar">Sair</button>
  </menu>
</dialog>
```

> Em JavaScript: `document.getElementById('dialog-logout').showModal()` para abrir.

---

## 2. Arquitetura Lógica do Site

### Hierarquia de títulos (`<h1>` a `<h6>`)

A hierarquia de títulos deve seguir uma **estrutura de tópicos como um documento de texto** — nunca pule níveis por razões estéticas (use CSS para estilizar o tamanho).

#### Regra de ouro:
- **`<h1>`**: Um único por "estado" da página. É o título principal daquele contexto.
- **`<h2>`**: Seções de primeiro nível dentro do `<main>`.
- **`<h3>`**: Itens dentro de cada seção.
- **`<h4>` a `<h6>`**: Raramente necessários; use apenas se houver real sub-hierarquia.

#### Hierarquia na Splash Page:
```
<h1> Meu Dashboard              ← título principal da tela de login
  (sem h2 necessário — conteúdo simples)
```

#### Hierarquia na Dashboard Page:
```
<h1> Meu Dashboard              ← título do app (no header)
  <h2> Produtividade            ← cada seção de categoria
    <h3> Notion                 ← cada ferramenta (se usar cards <article>)
  <h2> Design
    <h3> Figma
  <h2> Desenvolvimento
    <h3> VS Code Web
```

---

### Estrutura da página principal por tópico

Existem três abordagens válidas — escolha conforme a complexidade:

#### Opção A — Lista simples (mais semântica, mais leve)
```html
<section id="produtividade">
  <h2>Produtividade</h2>
  <ul class="tools-list">
    <li>
      <a href="https://notion.so" target="_blank" rel="noopener noreferrer">
        Notion
      </a>
    </li>
    <li>
      <a href="https://todoist.com" target="_blank" rel="noopener noreferrer">
        Todoist
      </a>
    </li>
  </ul>
</section>
```
Quando usar: links simples, sem descrição extra.

#### Opção B — Cards com article (mais rica, permite ícone + descrição)
```html
<section id="design">
  <h2>Design</h2>
  <ul class="tools-grid" role="list">
    <li>
      <article class="tool-card">
        <img src="icons/figma.svg" alt="" aria-hidden="true">
        <h3><a href="https://figma.com" target="_blank" rel="noopener noreferrer">Figma</a></h3>
        <p>Design de interfaces e prototipação.</p>
      </article>
    </li>
  </ul>
</section>
```
Quando usar: cards visuais com ícone, nome e descrição.

#### Opção C — details/summary (expansível nativamente)
```html
<section id="desenvolvimento">
  <details>
    <summary><h2>Desenvolvimento</h2></summary>
    <ul class="tools-list">
      <li><a href="https://github.com">GitHub</a></li>
    </ul>
  </details>
</section>
```
Quando usar: muitas categorias, quer deixar algumas colapsadas por padrão.

> **Recomendação para o seu projeto**: Opção B (cards com `<article>`) oferece a melhor base para futuro CSS grid/flexbox e é visualmente mais interessante.

---

## 3. Preparação para CSS e Comportamento

### Estratégia de `class` e `id`

| Atributo | Quando usar | Exemplo |
|----------|-------------|---------|
| `id` | Identificador **único** na página. Âncoras de navegação, alvo de JavaScript. | `id="produtividade"` |
| `class` | Grupos de elementos com **estilos ou comportamento compartilhado**. | `class="tool-card"` |

#### Sistema de nomenclatura sugerido (BEM simplificado)

```
bloco__elemento--modificador
```

```html
<!-- Bloco: splash -->
<div class="splash">                          <!-- bloco -->
  <header class="splash__header">             <!-- elemento do bloco -->
  <main class="splash__main">                 <!-- elemento do bloco -->
    <section class="splash__login">           <!-- elemento do bloco -->
      <button class="btn btn--google">        <!-- bloco btn, modificador google -->
```

```html
<!-- Bloco: dashboard -->
<div class="dashboard">
  <header class="dashboard__header">
  <nav class="dashboard__nav">
  <main class="dashboard__main">
    <section class="tool-section" id="produtividade">
      <ul class="tools-grid">
        <li>
          <article class="tool-card tool-card--produtividade">
```

#### Por que isso importa para CSS?

```css
/* Estilizar TODA a tela de abertura */
.splash { display: flex; flex-direction: column; }

/* Estilizar SÓ os cards de Design */
.tool-card--design { border-color: purple; }

/* Estilizar todos os cards */
.tool-card { border: 1px solid #ccc; border-radius: 8px; }
```

#### Por que importa para JavaScript?

```javascript
// Mostrar/esconder telas
document.querySelector('.splash').classList.add('hidden');
document.querySelector('.dashboard').classList.remove('hidden');

// Trabalhar com todos os cards de uma categoria
document.querySelectorAll('.tool-card--desenvolvimento')
  .forEach(card => card.style.display = 'none');
```

---

### Atributos de acessibilidade essenciais (ARIA)

Você não precisa memorizar tudo — comece com estes 5:

| Atributo | Quando usar | Exemplo |
|----------|-------------|---------|
| `aria-label` | Descreve elemento sem texto visível | `<button aria-label="Fechar menu">✕</button>` |
| `aria-labelledby` | Aponta para o id do elemento que serve como rótulo | `<section aria-labelledby="titulo-design">` |
| `aria-hidden="true"` | Esconde da leitura de tela (ícones decorativos) | `<img src="icon.svg" aria-hidden="true">` |
| `role` | Define o papel semântico quando a tag não é suficiente | `<ul role="list">` (quando CSS reseta o estilo de lista) |
| `alt` | Texto alternativo de imagens (não é ARIA, mas é fundamental) | `<img src="logo.png" alt="Logo do Dashboard">` |

```html
<!-- Bom exemplo: nav com dois menus na mesma página -->
<nav aria-label="Menu principal">...</nav>
<nav aria-label="Categorias de ferramentas">...</nav>

<!-- Bom exemplo: botão ícone acessível -->
<button type="button" aria-label="Abrir menu de configurações">
  <img src="gear-icon.svg" alt="" aria-hidden="true">
</button>
```

---

## 4. Recursos Fundamentais do HTML5

### 4.1 Estrutura base obrigatória

```html
<!DOCTYPE html>
<!-- DOCTYPE: diz ao navegador que é HTML5. Sempre na primeira linha. -->

<html lang="pt-BR">
<!-- lang: essencial para leitores de tela e corretor ortográfico -->

<head>
  <!-- Tudo aqui não é visível, mas configura a página -->
  
  <meta charset="UTF-8">
  <!-- charset: garante que acentos e caracteres especiais apareçam correto -->
  
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- viewport: essencial para responsividade em celulares -->
  
  <meta name="description" content="Meu portal pessoal de ferramentas e links">
  <!-- description: aparece no resultado de buscas do Google -->
  
  <title>Meu Dashboard — Login</title>
  <!-- title: aparece na aba do navegador e nos favoritos -->
  
  <link rel="stylesheet" href="styles.css">
  <!-- link: conecta o arquivo CSS externo -->
  
  <link rel="icon" href="favicon.ico" type="image/x-icon">
  <!-- favicon: ícone da aba do navegador -->
</head>

<body>
  <!-- Todo o conteúdo visível fica aqui -->
</body>

</html>
```

---

### 4.2 Formulários e botões de ação

```html
<!-- Formulário de login manual (para aprender a estrutura) -->
<form action="/login" method="POST" class="login-form" novalidate>
  
  <div class="form-group">
    <label for="email">E-mail</label>
    <!-- label + for="X" + id="X" vinculam o rótulo ao campo -->
    <input 
      type="email" 
      id="email" 
      name="email" 
      placeholder="seu@email.com"
      autocomplete="email"
      required
    >
  </div>
  
  <div class="form-group">
    <label for="senha">Senha</label>
    <input 
      type="password" 
      id="senha" 
      name="senha" 
      autocomplete="current-password"
      required
    >
  </div>
  
  <button type="submit" class="btn btn--primary">Entrar</button>
</form>

<!-- Botão de login com Google (sem formulário, via JavaScript) -->
<button type="button" class="btn btn--google" id="btn-login-google">
  <img src="google-icon.svg" alt="" aria-hidden="true">
  Entrar com Google
</button>
```

> 💡 `type="button"` em botões que não enviam formulário é muito importante — evita comportamento padrão inesperado.

---

### 4.3 Listas para organizar ferramentas

```html
<!-- Lista não-ordenada: para links onde a ordem não importa -->
<ul class="tools-list">
  <li><a href="https://notion.so">Notion</a></li>
  <li><a href="https://todoist.com">Todoist</a></li>
  <li><a href="https://calendar.google.com">Google Calendar</a></li>
</ul>

<!-- Lista ordenada: para passos ou rankings (menos comum no seu caso) -->
<ol class="steps-list">
  <li>Faça login com sua conta Google</li>
  <li>Personalize suas categorias</li>
  <li>Adicione seus links favoritos</li>
</ol>
```

---

### 4.4 Links e menus de navegação

```html
<!-- Link externo: sempre com target e rel por segurança -->
<a href="https://github.com" target="_blank" rel="noopener noreferrer">
  GitHub
</a>
<!-- rel="noopener noreferrer": impede que a página aberta acesse window.opener (segurança) -->

<!-- Link interno (âncora): navega para seção na mesma página -->
<a href="#produtividade">Ir para Produtividade</a>

<!-- Link que aciona JavaScript (use button, não <a href="#">) -->
<button type="button" id="btn-logout">Sair</button>
<!-- Nunca use <a href="javascript:void(0)"> — use <button> -->

<!-- Menu de navegação completo -->
<nav aria-label="Categorias">
  <ul>
    <li><a href="#produtividade" class="nav-link">Produtividade</a></li>
    <li><a href="#design" class="nav-link nav-link--active">Design</a></li>
    <li><a href="#desenvolvimento" class="nav-link">Desenvolvimento</a></li>
  </ul>
</nav>
```

---

### 4.5 `<div>` e `<span>` — quando usar

| Tag | Tipo | Quando usar |
|-----|------|-------------|
| `<div>` | Bloco | Agrupamento **sem** significado semântico, apenas para layout/CSS |
| `<span>` | Inline | Destacar **parte** de um texto para estilização |

```html
<!-- div: agrupar dois elementos apenas para posicionamento CSS -->
<div class="user-info">
  <img src="avatar.jpg" alt="Foto de João">
  <span class="user-name">João Silva</span>
  <!-- span: estilizar só o nome dentro de um parágrafo ou div -->
</div>

<!-- Ruim: div onde deveria ser section ou article -->
<!-- <div class="categoria"> — use <section> em vez disso -->
```

---

## 5. Exercício Prático Guiado

### 5.1 Splash Page (Tela de Login)

```html
<!DOCTYPE html>
<!-- Declara HTML5 — sempre deve ser a primeira linha do arquivo -->

<html lang="pt-BR">
<!-- lang="pt-BR": indica idioma português do Brasil.
     Leitores de tela vão pronunciar corretamente.
     Importante para SEO também. -->

<head>
  <meta charset="UTF-8">
  <!-- Codificação UTF-8: suporta todos os caracteres do português -->
  
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- Responsividade: sem isso, o celular vai exibir a versão desktop minúscula -->
  
  <meta name="description" content="Acesse seu painel pessoal de ferramentas e links organizados por categoria.">
  <!-- Descrição para buscadores — não aparece na página, só no Google -->
  
  <title>Meu Dashboard — Entrar</title>
  <!-- Título da aba. No splash, indica a ação: "Entrar". 
       No dashboard, mudaria para "Meu Dashboard — Produtividade" etc. -->
  
  <link rel="stylesheet" href="css/styles.css">
  <!-- Folha de estilo externa. O arquivo não existe ainda,
       mas já preparamos o HTML para quando você criar o CSS. -->
       
  <link rel="icon" href="assets/favicon.ico" type="image/x-icon">
  <!-- Ícone da aba do navegador -->
</head>

<body class="page page--splash">
<!-- body: raiz de todo conteúdo visível.
     class="page": classe base para todos os estados da página.
     class="page--splash": modificador que identifica ESTE estado específico.
     Com isso, em CSS você pode fazer:
       .page--splash { background: blue; }
       .page--dashboard { background: white; }
     E em JS: document.body.classList.replace('page--splash', 'page--dashboard') -->

  <!-- ============================================================
       HEADER — Cabeçalho da tela de login
       Contém a identidade visual (logo + nome do app).
       Não contém navegação — o usuário ainda não está logado.
  ============================================================ -->
  <header class="splash__header" role="banner">
  <!-- role="banner": reforço de acessibilidade. Indica o cabeçalho principal.
       Tecnicamente redundante em <header> direto filho de <body>,
       mas é boa prática para compatibilidade com leitores mais antigos. -->

    <figure class="splash__logo">
    <!-- figure: agrupa a imagem e sua legenda como uma unidade semântica.
         Ideal quando a imagem tem uma descrição associada. -->
    
      <img 
        src="assets/logo.svg" 
        alt="Logo do Meu Dashboard"
        class="logo__image"
        width="80" 
        height="80"
      >
      <!-- alt: descrição da imagem para leitores de tela e quando a imagem falha.
           width e height: evita "layout shift" (o conteúdo pulando enquanto carrega). -->
      
      <figcaption class="logo__tagline">Seu portal de ferramentas</figcaption>
      <!-- figcaption: legenda da figura. Pode ser visualmente escondida com CSS,
           mas continua acessível para leitores de tela. -->
    </figure>

  </header>
  <!-- /splash__header -->


  <!-- ============================================================
       MAIN — Conteúdo principal da tela de login
       Deve haver UM único <main> por estado de página.
       É aqui que está a ação principal: fazer login.
  ============================================================ -->
  <main class="splash__main" id="conteudo-principal">
  <!-- id="conteudo-principal": alvo do "pular para conteúdo" (acessibilidade).
       Você pode adicionar um link invisível no topo:
       <a href="#conteudo-principal" class="skip-link">Pular para o conteúdo</a> -->

    <section class="login-section" aria-labelledby="login-titulo">
    <!-- section: agrupa o conteúdo de login como uma unidade temática.
         aria-labelledby: aponta para o id do título desta seção.
         Leitores de tela anunciam: "Seção: Bem-vindo de volta" -->

      <h1 id="login-titulo" class="login-section__title">Bem-vindo de volta</h1>
      <!-- h1: UM por estado de página. É o título mais importante desta tela.
           id="login-titulo": referenciado pelo aria-labelledby da section acima. -->

      <p class="login-section__subtitle">
        Entre com sua conta Google para acessar suas ferramentas.
      </p>
      <!-- p: parágrafo de apoio, sem tag semântica especial necessária. -->


      <!-- ---- Botão principal: Login com Google ---- -->
      <div class="login-actions">
      <!-- div: agrupador sem semântica. Usado aqui APENAS para layout CSS
           (por exemplo, centralizar o botão com flexbox). -->

        <button 
          type="button" 
          class="btn btn--google btn--large" 
          id="btn-login-google"
          aria-label="Entrar com conta Google"
        >
        <!-- type="button": crucial. Sem isso, dentro de um form o padrão é "submit".
             id="btn-login-google": alvo específico do JS para adicionar o evento de login.
             aria-label: descreve a ação completa para leitores de tela. -->
        
          <img 
            src="assets/google-icon.svg" 
            alt="" 
            aria-hidden="true"
            class="btn__icon"
            width="20"
            height="20"
          >
          <!-- alt="": imagem decorativa — o texto do botão já descreve a ação.
               aria-hidden="true": esconde explicitamente da leitura de tela. -->
          
          <span class="btn__text">Entrar com Google</span>
          <!-- span: envolve o texto do botão para permitir estilização separada do ícone. -->
        </button>

      </div>
      <!-- /login-actions -->


      <!-- ---- Separador visual + login manual (opcional) ---- -->
      <div class="login-divider" aria-hidden="true">
        <span>ou</span>
      </div>
      <!-- aria-hidden="true": o separador "ou" é puramente visual/decorativo.
           Leitores de tela podem pular isso. -->

      <!-- Formulário de login manual -->
      <form 
        class="login-form" 
        id="form-login-manual"
        novalidate
        aria-label="Formulário de login com e-mail e senha"
      >
      <!-- novalidate: desativa a validação nativa do browser.
           Você controlará a validação com JavaScript depois.
           aria-label: descreve o formulário para leitores de tela. -->

        <div class="form-group">
          <label for="campo-email" class="form-label">E-mail</label>
          <!-- label for="X": SEMPRE vincule ao id do input correspondente.
               Ao clicar no label, o foco vai para o input (usabilidade).
               Leitores de tela anunciam o label ao focar no campo. -->
          
          <input 
            type="email"
            id="campo-email"
            name="email"
            class="form-input"
            placeholder="seu@email.com"
            autocomplete="email"
            required
            aria-describedby="email-erro"
          >
          <!-- type="email": teclado numérico em mobile, validação básica.
               autocomplete="email": o browser pode sugerir e-mails salvos.
               aria-describedby="email-erro": aponta para onde vai a mensagem de erro (JS). -->
          
          <span 
            class="form-error" 
            id="email-erro" 
            role="alert" 
            aria-live="polite"
          ></span>
          <!-- role="alert" + aria-live="polite": quando JS inserir texto aqui,
               o leitor de tela anuncia automaticamente a mensagem de erro. -->
        </div>
        <!-- /form-group email -->

        <div class="form-group">
          <label for="campo-senha" class="form-label">Senha</label>
          <input 
            type="password"
            id="campo-senha"
            name="senha"
            class="form-input"
            autocomplete="current-password"
            required
            aria-describedby="senha-erro"
          >
          <span class="form-error" id="senha-erro" role="alert" aria-live="polite"></span>
        </div>
        <!-- /form-group senha -->

        <button type="submit" class="btn btn--primary btn--full">
          Entrar
        </button>
        <!-- type="submit": dentro de um form, este é o tipo correto para envio. -->

      </form>
      <!-- /form-login-manual -->

    </section>
    <!-- /login-section -->

  </main>
  <!-- /splash__main -->


  <!-- ============================================================
       FOOTER — Rodapé da tela de login
       Informações secundárias: copyright, links de ajuda.
  ============================================================ -->
  <footer class="splash__footer" role="contentinfo">
  <!-- role="contentinfo": reforço semântico. Identifica o rodapé principal.
       Tecnicamente redundante em <footer> filho de <body>, mas boa prática. -->

    <p class="footer__copy">
      <small>
        &copy; <time datetime="2025">2025</time> Meu Dashboard.
        <!-- time + datetime: a máquina lê "2025", o humano lê "2025".
             Importante para mecanismos de busca e bots de calendário. -->
        Todos os direitos reservados.
      </small>
    </p>

    <nav class="footer__nav" aria-label="Links do rodapé">
    <!-- nav: mesmo no rodapé, um conjunto de links de navegação merece <nav>.
         aria-label diferencia do nav principal. -->
      <ul>
        <li><a href="/privacidade">Privacidade</a></li>
        <li><a href="/termos">Termos de Uso</a></li>
        <li><a href="/suporte">Suporte</a></li>
      </ul>
    </nav>

  </footer>
  <!-- /splash__footer -->


  <!-- ============================================================
       SCRIPTS — Sempre no final do body, antes de </body>
       Colocar scripts no final garante que o HTML já foi carregado
       quando o JavaScript tentar encontrar os elementos.
  ============================================================ -->
  <script src="js/auth.js" defer></script>
  <!-- defer: o script baixa em paralelo mas executa após o HTML ser analisado.
       Equivalente a colocar no final do body, mas semânticamente mais correto. -->

</body>
</html>
```

---

### 5.2 Dashboard Page (Após Login)

```html
<!DOCTYPE html>
<html lang="pt-BR">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Meu Dashboard — Ferramentas</title>
  <link rel="stylesheet" href="css/styles.css">
  <link rel="icon" href="assets/favicon.ico" type="image/x-icon">
</head>

<body class="page page--dashboard">
<!-- Mesma estratégia de classe da splash:
     page: base comum.
     page--dashboard: estado específico. -->

  <!-- ============================================================
       LINK DE ACESSIBILIDADE — "Pular para conteúdo"
       Invisível por padrão, aparece ao pressionar Tab.
       Permite que usuários de teclado/leitores de tela pulem a navegação.
  ============================================================ -->
  <a href="#conteudo-principal" class="skip-to-content">
    Pular para o conteúdo principal
  </a>
  <!-- CSS para isso:
       .skip-to-content { position: absolute; top: -100%; }
       .skip-to-content:focus { top: 0; } -->


  <!-- ============================================================
       HEADER — Cabeçalho do app (sempre visível)
       Contém: logo, título, informações do usuário logado, botão de logout.
  ============================================================ -->
  <header class="app-header" role="banner">

    <!-- Identidade do app -->
    <div class="app-header__brand">
    <!-- div: agrupador puro para layout flexbox. Sem semântica própria. -->
      <img 
        src="assets/logo.svg" 
        alt="Meu Dashboard" 
        class="app-header__logo"
        width="40" 
        height="40"
      >
      <span class="app-header__app-name">Meu Dashboard</span>
      <!-- span: inline, sem semântica. 
           Nota: NÃO é <h1> pois o título do app no header não é o título da página.
           O <h1> semântico estará dentro do <main>. -->
    </div>

    <!-- Informações do usuário logado -->
    <div class="app-header__user" aria-label="Informações do usuário">
      <img 
        src="assets/user-avatar.jpg" 
        alt="Foto de perfil de João Silva"
        class="user-avatar"
        width="36" 
        height="36"
      >
      <span class="user-name">João Silva</span>
    </div>

    <!-- Botão de logout -->
    <button 
      type="button" 
      class="btn btn--ghost btn--small" 
      id="btn-logout"
      aria-label="Sair da conta de João Silva"
    >
      <img src="assets/logout-icon.svg" alt="" aria-hidden="true" width="16" height="16">
      <span class="btn__text">Sair</span>
    </button>

  </header>
  <!-- /app-header -->


  <!-- ============================================================
       LAYOUT WRAPPER — Container para sidebar + conteúdo principal
       div sem semântica: apenas para o layout de duas colunas no CSS.
  ============================================================ -->
  <div class="app-layout">


    <!-- ============================================================
         ASIDE — Barra lateral com ferramentas recentes e navegação rápida
         aside: conteúdo complementar ao main, mas não essencial ao fluxo.
    ============================================================ -->
    <aside class="app-sidebar" aria-label="Navegação e atalhos">

      <!-- Navegação por categoria -->
      <nav class="sidebar-nav" aria-label="Categorias de ferramentas">
        <h2 class="sidebar-nav__title">Categorias</h2>
        <!-- h2: primeiro nível de título dentro do aside.
             Hierarquia se mantém mesmo fora do <main>. -->
        
        <ul class="sidebar-nav__list">
          <li>
            <a href="#produtividade" class="sidebar-nav__link sidebar-nav__link--active">
              Produtividade
            </a>
          </li>
          <li>
            <a href="#design" class="sidebar-nav__link">Design</a>
          </li>
          <li>
            <a href="#desenvolvimento" class="sidebar-nav__link">Desenvolvimento</a>
          </li>
          <li>
            <a href="#comunicacao" class="sidebar-nav__link">Comunicação</a>
          </li>
          <li>
            <a href="#aprendizado" class="sidebar-nav__link">Aprendizado</a>
          </li>
        </ul>
      </nav>
      <!-- /sidebar-nav -->

      <!-- Ferramentas acessadas recentemente -->
      <section class="sidebar-recent" aria-labelledby="titulo-recentes">
      <!-- section dentro do aside: agrupa conteúdo temático com seu próprio título. -->
        
        <h2 id="titulo-recentes" class="sidebar-recent__title">Recentes</h2>
        
        <ul class="sidebar-recent__list">
          <li>
            <a href="https://figma.com" target="_blank" rel="noopener noreferrer" class="recent-link">
              <img src="assets/icons/figma.svg" alt="" aria-hidden="true" width="16" height="16">
              Figma
            </a>
          </li>
          <li>
            <a href="https://github.com" target="_blank" rel="noopener noreferrer" class="recent-link">
              <img src="assets/icons/github.svg" alt="" aria-hidden="true" width="16" height="16">
              GitHub
            </a>
          </li>
          <li>
            <a href="https://notion.so" target="_blank" rel="noopener noreferrer" class="recent-link">
              <img src="assets/icons/notion.svg" alt="" aria-hidden="true" width="16" height="16">
              Notion
            </a>
          </li>
        </ul>
      </section>
      <!-- /sidebar-recent -->

    </aside>
    <!-- /app-sidebar -->


    <!-- ============================================================
         MAIN — Todo o conteúdo principal do dashboard
         Um único <main> com id para o "skip link".
    ============================================================ -->
    <main class="dashboard-main" id="conteudo-principal">

      <!-- Cabeçalho interno do main -->
      <div class="dashboard-main__header">
        <h1 class="dashboard-main__title">Minhas Ferramentas</h1>
        <!-- h1: único na página. Descreve o CONTEÚDO do main,
             não o nome do app (que está no <header> como <span>). -->
        
        <p class="dashboard-main__subtitle">
          <span id="tools-count">24</span> ferramentas em 
          <span id="category-count">5</span> categorias
        </p>
        <!-- span com ids: JS poderá atualizar esses números dinamicamente. -->
      </div>


      <!-- ==================================================
           SEÇÃO: Produtividade
      ================================================== -->
      <section 
        class="tool-section" 
        id="produtividade"
        aria-labelledby="titulo-produtividade"
      >
        <header class="tool-section__header">
        <!-- header DENTRO de section: válido! Cada seção pode ter seu próprio cabeçalho. -->
          
          <h2 id="titulo-produtividade" class="tool-section__title">
            <img src="assets/icons/category-produtividade.svg" alt="" aria-hidden="true">
            Produtividade
          </h2>
          
          <span class="tool-section__count" aria-label="6 ferramentas nesta categoria">
            6 ferramentas
          </span>
        </header>
        <!-- /tool-section__header -->

        <ul class="tools-grid" role="list">
        <!-- role="list": quando CSS remove o estilo de lista (list-style: none),
             alguns leitores de tela param de anunciar como lista.
             role="list" restaura isso. -->

          <li class="tools-grid__item">
            <article class="tool-card tool-card--produtividade">
            <!-- article: cada ferramenta é autocontida — faz sentido sozinha.
                 Poderia ser extraída da página e ainda seria compreensível.
                 tool-card--produtividade: modificador de categoria para CSS/JS. -->
              
              <div class="tool-card__icon-wrap">
                <img 
                  src="assets/icons/notion.svg" 
                  alt="" 
                  aria-hidden="true"
                  class="tool-card__icon"
                  width="32" 
                  height="32"
                >
              </div>
              
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a 
                    href="https://notion.so" 
                    target="_blank" 
                    rel="noopener noreferrer"
                    class="tool-card__link"
                    aria-label="Abrir Notion em nova aba"
                  >
                    Notion
                  </a>
                </h3>
                <!-- h3: terceiro nível da hierarquia. main > section (h2) > article (h3). -->
                
                <p class="tool-card__description">
                  Notas, wikis e gerenciamento de projetos.
                </p>
              </div>

            </article>
            <!-- /tool-card Notion -->
          </li>

          <!-- Repita o padrão para cada ferramenta da categoria -->
          <li class="tools-grid__item">
            <article class="tool-card tool-card--produtividade">
              <div class="tool-card__icon-wrap">
                <img src="assets/icons/todoist.svg" alt="" aria-hidden="true" width="32" height="32">
              </div>
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a href="https://todoist.com" target="_blank" rel="noopener noreferrer" class="tool-card__link" aria-label="Abrir Todoist em nova aba">
                    Todoist
                  </a>
                </h3>
                <p class="tool-card__description">Listas de tarefas e projetos.</p>
              </div>
            </article>
          </li>

          <li class="tools-grid__item">
            <article class="tool-card tool-card--produtividade">
              <div class="tool-card__icon-wrap">
                <img src="assets/icons/gcalendar.svg" alt="" aria-hidden="true" width="32" height="32">
              </div>
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a href="https://calendar.google.com" target="_blank" rel="noopener noreferrer" class="tool-card__link" aria-label="Abrir Google Calendar em nova aba">
                    Google Calendar
                  </a>
                </h3>
                <p class="tool-card__description">Agenda e lembretes.</p>
              </div>
            </article>
          </li>

        </ul>
        <!-- /tools-grid produtividade -->

      </section>
      <!-- /section#produtividade -->


      <!-- ==================================================
           SEÇÃO: Design
      ================================================== -->
      <section 
        class="tool-section" 
        id="design"
        aria-labelledby="titulo-design"
      >
        <header class="tool-section__header">
          <h2 id="titulo-design" class="tool-section__title">
            <img src="assets/icons/category-design.svg" alt="" aria-hidden="true">
            Design
          </h2>
          <span class="tool-section__count" aria-label="5 ferramentas nesta categoria">5 ferramentas</span>
        </header>

        <ul class="tools-grid" role="list">
          
          <li class="tools-grid__item">
            <article class="tool-card tool-card--design">
              <div class="tool-card__icon-wrap">
                <img src="assets/icons/figma.svg" alt="" aria-hidden="true" width="32" height="32">
              </div>
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a href="https://figma.com" target="_blank" rel="noopener noreferrer" class="tool-card__link" aria-label="Abrir Figma em nova aba">
                    Figma
                  </a>
                </h3>
                <p class="tool-card__description">Design de interfaces e prototipação.</p>
              </div>
            </article>
          </li>

          <li class="tools-grid__item">
            <article class="tool-card tool-card--design">
              <div class="tool-card__icon-wrap">
                <img src="assets/icons/coolors.svg" alt="" aria-hidden="true" width="32" height="32">
              </div>
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a href="https://coolors.co" target="_blank" rel="noopener noreferrer" class="tool-card__link" aria-label="Abrir Coolors em nova aba">
                    Coolors
                  </a>
                </h3>
                <p class="tool-card__description">Gerador de paletas de cores.</p>
              </div>
            </article>
          </li>

        </ul>
        <!-- /tools-grid design -->

      </section>
      <!-- /section#design -->


      <!-- ==================================================
           SEÇÃO: Desenvolvimento
      ================================================== -->
      <section 
        class="tool-section" 
        id="desenvolvimento"
        aria-labelledby="titulo-desenvolvimento"
      >
        <header class="tool-section__header">
          <h2 id="titulo-desenvolvimento" class="tool-section__title">
            <img src="assets/icons/category-dev.svg" alt="" aria-hidden="true">
            Desenvolvimento
          </h2>
          <span class="tool-section__count" aria-label="6 ferramentas nesta categoria">6 ferramentas</span>
        </header>

        <ul class="tools-grid" role="list">

          <li class="tools-grid__item">
            <article class="tool-card tool-card--desenvolvimento">
              <div class="tool-card__icon-wrap">
                <img src="assets/icons/github.svg" alt="" aria-hidden="true" width="32" height="32">
              </div>
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a href="https://github.com" target="_blank" rel="noopener noreferrer" class="tool-card__link" aria-label="Abrir GitHub em nova aba">
                    GitHub
                  </a>
                </h3>
                <p class="tool-card__description">Repositórios e controle de versão.</p>
              </div>
            </article>
          </li>

          <li class="tools-grid__item">
            <article class="tool-card tool-card--desenvolvimento">
              <div class="tool-card__icon-wrap">
                <img src="assets/icons/mdn.svg" alt="" aria-hidden="true" width="32" height="32">
              </div>
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a href="https://developer.mozilla.org" target="_blank" rel="noopener noreferrer" class="tool-card__link" aria-label="Abrir MDN Web Docs em nova aba">
                    MDN Web Docs
                  </a>
                </h3>
                <p class="tool-card__description">Documentação oficial de HTML, CSS e JS.</p>
              </div>
            </article>
          </li>

          <li class="tools-grid__item">
            <article class="tool-card tool-card--desenvolvimento">
              <div class="tool-card__icon-wrap">
                <img src="assets/icons/codepen.svg" alt="" aria-hidden="true" width="32" height="32">
              </div>
              <div class="tool-card__body">
                <h3 class="tool-card__name">
                  <a href="https://codepen.io" target="_blank" rel="noopener noreferrer" class="tool-card__link" aria-label="Abrir CodePen em nova aba">
                    CodePen
                  </a>
                </h3>
                <p class="tool-card__description">Playground online para HTML, CSS e JS.</p>
              </div>
            </article>
          </li>

        </ul>
        <!-- /tools-grid desenvolvimento -->

      </section>
      <!-- /section#desenvolvimento -->


      <!-- Continue o padrão para Comunicação, Aprendizado, etc. -->

    </main>
    <!-- /dashboard-main -->


  </div>
  <!-- /app-layout -->


  <!-- ============================================================
       DIALOG — Modal de confirmação de logout
       Elemento nativo HTML5. Aparece como popup ao chamar .showModal() via JS.
  ============================================================ -->
  <dialog 
    class="modal" 
    id="dialog-logout"
    aria-labelledby="dialog-logout-titulo"
    aria-describedby="dialog-logout-desc"
  >
    <article class="modal__content">
    <!-- article dentro de dialog: o conteúdo do modal é autocontido. -->
      
      <header class="modal__header">
        <h2 id="dialog-logout-titulo" class="modal__title">Sair da conta?</h2>
        <button 
          type="button" 
          class="modal__close" 
          id="btn-fechar-dialog"
          aria-label="Fechar diálogo"
        >
          ✕
        </button>
      </header>

      <p id="dialog-logout-desc" class="modal__body">
        Você tem certeza que deseja sair? Precisará fazer login novamente para acessar suas ferramentas.
      </p>

      <footer class="modal__footer">
        <button type="button" class="btn btn--ghost" id="btn-cancelar-logout">
          Cancelar
        </button>
        <button type="button" class="btn btn--danger" id="btn-confirmar-logout">
          Sair
        </button>
      </footer>

    </article>
  </dialog>
  <!-- /dialog-logout -->


  <!-- ============================================================
       FOOTER — Rodapé da aplicação
  ============================================================ -->
  <footer class="app-footer" role="contentinfo">
    <div class="app-footer__inner">
      
      <p class="app-footer__copy">
        <small>
          &copy; <time datetime="2025">2025</time> Meu Dashboard
        </small>
      </p>

      <nav class="app-footer__nav" aria-label="Links do rodapé da aplicação">
        <ul>
          <li><a href="/privacidade">Privacidade</a></li>
          <li><a href="/suporte">Suporte</a></li>
          <li>
            Última atualização: 
            <time datetime="2025-06-15">15 jun. 2025</time>
          </li>
        </ul>
      </nav>

    </div>
  </footer>
  <!-- /app-footer -->

  <script src="js/dashboard.js" defer></script>

</body>
</html>
```

---

## 6. Boas Práticas e Erros Comuns

### 6.1 O que EVITAR

#### ❌ Divite: excesso de `<div>` sem semântica

```html
<!-- RUIM: tudo é div -->
<div class="header">
  <div class="nav">
    <div class="nav-item"><a href="#">Início</a></div>
  </div>
</div>

<!-- BOM: tags que descrevem o conteúdo -->
<header>
  <nav>
    <ul>
      <li><a href="#">Início</a></li>
    </ul>
  </nav>
</header>
```

---

#### ❌ `<h1>` em múltiplos lugares sem propósito

```html
<!-- RUIM: dois h1 na mesma tela -->
<header>
  <h1>Meu Dashboard</h1>  <!-- nome do app -->
</header>
<main>
  <h1>Produtividade</h1>  <!-- título da seção — deveria ser h2 -->
</main>

<!-- BOM: h1 único, hierarquia correta -->
<header>
  <span class="app-name">Meu Dashboard</span>  <!-- nome do app como span -->
</header>
<main>
  <h1>Minhas Ferramentas</h1>  <!-- título do conteúdo principal -->
  <section>
    <h2>Produtividade</h2>     <!-- seção = h2 -->
  </section>
</main>
```

---

#### ❌ IDs duplicados

```html
<!-- RUIM: id repetido -->
<article id="tool-card"> ... </article>
<article id="tool-card"> ... </article>  <!-- INVÁLIDO: IDs devem ser únicos -->

<!-- BOM: id único por elemento, classes para grupos -->
<article id="tool-notion" class="tool-card"> ... </article>
<article id="tool-figma" class="tool-card"> ... </article>
<!-- JavaScript consegue selecionar individualmente por id,
     e coletivamente por classe. -->
```

---

#### ❌ Pular níveis de título

```html
<!-- RUIM: pula do h2 para h4 -->
<h2>Design</h2>
  <h4>Figma</h4>  <!-- pulou h3 — confunde leitores de tela -->

<!-- BOM: hierarquia contínua -->
<h2>Design</h2>
  <h3>Figma</h3>  <!-- sequência lógica -->
```

---

#### ❌ `<a href="#">` para ações JavaScript

```html
<!-- RUIM: link sem destino real -->
<a href="#" onclick="logout()">Sair</a>

<!-- BOM: botão para ações, link para navegação -->
<button type="button" id="btn-logout">Sair</button>
<!-- Em JS: document.getElementById('btn-logout').addEventListener('click', logout) -->
```

---

#### ❌ Imagens sem `alt` ou com `alt` ruim

```html
<!-- RUIM: sem alt (leitor de tela lê o nome do arquivo) -->
<img src="logo-meu-dashboard-v2-final.svg">

<!-- RUIM: alt genérico -->
<img src="logo.svg" alt="imagem">

<!-- BOM: alt descritivo para imagens informativas -->
<img src="logo.svg" alt="Logo do Meu Dashboard">

<!-- BOM: alt vazio para imagens decorativas (leitor de tela ignora) -->
<img src="decorative-wave.svg" alt="" aria-hidden="true">
```

---

### 6.2 Como manter o código limpo

#### ✅ Recue o HTML consistentemente (2 ou 4 espaços)

```html
<main>
  <section>
    <h2>Design</h2>
    <ul>
      <li><a href="#">Figma</a></li>
    </ul>
  </section>
</main>
```

#### ✅ Comente o fechamento de tags longas

```html
<section id="produtividade">
  <!-- ... muito conteúdo ... -->
</section>
<!-- /section#produtividade -->
```

#### ✅ Use atributos na ordem lógica

```html
<!-- Convenção: type > id > name > class > href/src > outros atributos > aria- -->
<input type="email" id="campo-email" name="email" class="form-input" 
       autocomplete="email" required aria-describedby="email-erro">
```

#### ✅ Separe estrutura (HTML), estilo (CSS) e comportamento (JS)

```html
<!-- HTML: define a estrutura e o significado -->
<button type="button" class="btn btn--primary" id="btn-login-google">
  Entrar com Google
</button>

<!-- CSS (no arquivo .css): cuida da aparência -->
<!-- .btn--primary { background: #4285f4; color: white; } -->

<!-- JS (no arquivo .js): cuida do comportamento -->
<!-- document.getElementById('btn-login-google').addEventListener('click', handleLogin) -->
```

---

### 6.3 Checklist de qualidade antes de passar para CSS

Antes de abrir o editor de CSS, verifique:

- [ ] Há exatamente **um `<h1>`** na página (por estado)
- [ ] Todos os `id` são **únicos** no documento
- [ ] Cada `<label>` tem um `for` que corresponde a um `id` de `<input>`
- [ ] Toda `<img>` tem `alt` (vazio `""` se decorativa, descritivo se informativa)
- [ ] Links externos têm `rel="noopener noreferrer"` e `target="_blank"`
- [ ] Botões de ação têm `type="button"` (não confie no padrão)
- [ ] Há um único `<main>` por estado
- [ ] `<nav>` tem `aria-label` se houver mais de um na página
- [ ] A hierarquia de títulos não pula níveis (h1 → h2 → h3, nunca h1 → h3)
- [ ] Não há `<div>` onde deveria ter `<section>`, `<article>`, `<nav>`, etc.

---

## Resumo Visual da Estrutura

```
SPLASH PAGE
├── <body class="page page--splash">
│   ├── <header class="splash__header">   → logo + nome do app
│   ├── <main class="splash__main">
│   │   └── <section class="login-section">
│   │       ├── <h1> Bem-vindo
│   │       ├── <button> Login com Google
│   │       └── <form> Login manual
│   └── <footer class="splash__footer">   → copyright + links

DASHBOARD PAGE
├── <body class="page page--dashboard">
│   ├── <a href="#conteudo-principal">    → skip link (acessibilidade)
│   ├── <header class="app-header">       → logo + usuário + logout
│   ├── <div class="app-layout">          → container de layout
│   │   ├── <aside class="app-sidebar">
│   │   │   ├── <nav> Categorias
│   │   │   └── <section> Recentes
│   │   └── <main id="conteudo-principal">
│   │       ├── <h1> Minhas Ferramentas
│   │       ├── <section id="produtividade">
│   │       │   ├── <h2> Produtividade
│   │       │   └── <ul> <li> <article class="tool-card">
│   │       ├── <section id="design">
│   │       │   ├── <h2> Design
│   │       │   └── <ul> <li> <article class="tool-card">
│   │       └── <section id="desenvolvimento">
│   ├── <dialog id="dialog-logout">       → modal nativo HTML5
│   └── <footer class="app-footer">       → copyright + links
```

---

*Este guia foi elaborado especificamente para o seu projeto de Splash Page + Dashboard de Ferramentas, com foco em HTML5 semântico moderno, acessibilidade e preparação para CSS e JavaScript.*
