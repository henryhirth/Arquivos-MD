# 🎯 CSS Moderno — Tópico 1: Unidades de Medida
**Mentor:** Sênior Frontend | **Aluno:** Henry Hirth | **Projeto:** Portal de Ferramentas da Internet  
**Nível:** Core Fundamentals | **Tempo estimado:** 3–4 horas

---

## Índice
1. [Por que unidades importam para o seu negócio](#1-por-que-unidades-importam-para-o-seu-negócio)
2. [O problema com `px` — e a história por trás disso](#2-o-problema-com-px--e-a-história-por-trás-disso)
3. [rem — A unidade mais importante do seu projeto](#3-rem--a-unidade-mais-importante-do-seu-projeto)
4. [em — Relativa ao pai (use com propósito)](#4-em--relativa-ao-pai-use-com-propósito)
5. [Unidades de Viewport: vw, vh e as novas sv\*/lv\*/dv\*](#5-unidades-de-viewport-vw-vh-e-as-novas-svlvdv)
6. [% — Para larguras de layout](#6---para-larguras-de-layout)
7. [ch e lh — Unidades tipográficas modernas](#7-ch-e-lh--unidades-tipográficas-modernas)
8. [Tabela de Decisão Rápida](#8-tabela-de-decisão-rápida)
9. [Código Comentado — Aplicado ao seu HTML](#9-código-comentado--aplicado-ao-seu-html)
10. [Curiosidades e Hacks](#10-curiosidades-e-hacks)
11. [Exercício Prático](#11-exercício-prático)
12. [Checklist e Próximo Passo](#12-checklist-e-próximo-passo)

---

## 1. Por que unidades importam para o seu negócio

Antes de qualquer código: entender unidades de medida não é detalhe técnico. É estratégia de negócio.

Seu site terá **200+ ferramentas** organizadas em cards — isso significa dezenas de componentes repetidos. Se você definir tamanhos em unidades erradas:

- **Acessibilidade quebrada** → usuários com baixa visão aumentam a fonte do navegador e seu layout ignora isso → eles vão embora
- **Mobile com layout torto** → barras do Chrome mobile "engolindo" parte da splash page → primeira impressão destruída
- **Retrabalho em cascata** → mudar um tamanho base obriga alterar 50 lugares no CSS em vez de 1

Escolher a unidade certa **desde o início** poupa semanas de refatoração.

---

## 2. O problema com `px` — e a história por trás disso

### Histórico evolutivo

No CSS2 (anos 90–2000), `px` dominava porque os monitores tinham resolução fixa e os sites não precisavam ser responsivos. Quando o iPhone surgiu em 2007, tudo mudou — e o CSS3 começou a empurrar unidades relativas como padrão.

### O que é px exatamente?

`px` é uma unidade **absoluta** no CSS — ela **não se adapta a nada**: nem ao tamanho da tela, nem à preferência do usuário, nem ao elemento pai.

```css
/* ❌ Exemplo problemático — tudo em px */
.tool-card__name { font-size: 14px; }
.tool-section__title { font-size: 24px; }
.btn { padding: 12px 24px; }
```

### O cenário real que destrói conversões

```
1. Usuário com 60 anos acessa seu portal de ferramentas no Chrome
2. Ele foi em Configurações → Tamanho de fonte → "Grande" (equivale a 20px de base)
3. Seu CSS tem font-size: 14px em todo lugar
4. Resultado: absolutamente NADA muda. O navegador tenta ajudar, você ignora.
5. O usuário não consegue ler → fecha a aba → você perdeu um cliente
```

> **Impacto no negócio:** Cerca de 26% da população brasileira tem alguma deficiência visual. `px` no `font-size` exclui esse público diretamente.

### Quando px ainda é válido

`px` não é vilão em tudo. Use-o quando o valor **não deve** escalar:

```css
/* ✅ px faz sentido aqui */
.tool-card { border: 1px solid #e2e8f0; }   /* bordas finas sempre 1px */
.tool-card { border-radius: 8px; }           /* radius visual fixo é ok */
.app-header { box-shadow: 0 2px 4px rgba(0,0,0,0.1); } /* sombras fixas */
```

---

## 3. `rem` — A unidade mais importante do seu projeto

### O que é

**rem** = *root em* = relativo ao `font-size` do elemento `<html>` (a raiz, *root*, do documento).

Por padrão, **todos os navegadores** definem `html { font-size: 16px }`.

```
1rem   = 16px   (padrão do navegador)
1.5rem = 24px
2rem   = 32px
0.875rem = 14px
0.75rem  = 12px
```

### Por que é a mais importante

Quando o usuário muda a fonte do navegador para 20px:
- `1rem` passa a ser `20px`
- `1.5rem` passa a ser `30px`  
- **Todo o seu layout escala proporcionalmente e automaticamente**

### A regra do :root

```css
/* ✅ Padrão correto moderno */
:root {
  font-size: 100%; /* herda o padrão do navegador — NÃO trave em px */
}
```

> ⚠️ **Armadilha clássica:** Muitos tutoriais antigos ensinam `html { font-size: 62.5% }` para fazer `1rem = 10px` (matemática mais fácil). **Não faça isso.** Isso reduz a base para 10px e quebra a acessibilidade — usuários que aumentaram a fonte do navegador têm o efeito parcialmente cancelado.

### rem aplicado ao seu projeto

No seu HTML você já tem classes como `.tool-card__name`, `.tool-section__title`, `.btn--small`, `.btn--large`. Veja como rem torna tudo consistente:

```css
/* Sistema tipográfico em rem — escalável e acessível */
:root {
  --font-size-xs:   0.75rem;   /* 12px — labels, badges */
  --font-size-sm:   0.875rem;  /* 14px — descrições de card */
  --font-size-base: 1rem;      /* 16px — texto padrão */
  --font-size-lg:   1.125rem;  /* 18px — destaque leve */
  --font-size-xl:   1.25rem;   /* 20px — nome do app no header */
  --font-size-2xl:  1.5rem;    /* 24px — títulos de seção (h2) */
  --font-size-3xl:  2rem;      /* 32px — título principal (h1) */
  --font-size-4xl:  2.5rem;    /* 40px — hero headlines */
}

/* Aplicando ao seu HTML */
.dashboard-main__title   { font-size: var(--font-size-3xl); }  /* h1 */
.tool-section__title     { font-size: var(--font-size-2xl); }  /* h2 */
.tool-card__name         { font-size: var(--font-size-base); } /* h3 */
.tool-card__description  { font-size: var(--font-size-sm); }   /* p */
.tool-section__count     { font-size: var(--font-size-xs); }   /* span */
.app-header__app-name    { font-size: var(--font-size-xl); }   /* span */
```

> 💡 **Nota sobre Custom Properties (`var()`):** Você vai aprender profundamente sobre isso no Tópico 4. Por agora, entenda que `--font-size-base: 1rem` é uma variável CSS e `var(--font-size-base)` a usa. É como uma constante no seu CSS.

### rem para espaçamentos

```css
:root {
  --space-1:  0.25rem;  /* 4px  */
  --space-2:  0.5rem;   /* 8px  */
  --space-3:  0.75rem;  /* 12px */
  --space-4:  1rem;     /* 16px */
  --space-6:  1.5rem;   /* 24px */
  --space-8:  2rem;     /* 32px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
}

/* Nos seus componentes */
.tool-card         { padding: var(--space-4); gap: var(--space-3); }
.tool-section      { margin-block-end: var(--space-12); }
.tools-grid        { gap: var(--space-4); }
.app-header        { padding-inline: var(--space-6); }
```

> 💡 `margin-block-end` e `padding-inline` são **Logical Properties** — você vai aprender isso no Tópico 2 (Layout). Por agora, saiba que são equivalentes modernos de `margin-bottom` e `padding-left + padding-right`, mas que funcionam corretamente para idiomas da direita para esquerda.

---

## 4. `em` — Relativa ao pai (use com propósito)

### O que é

**em** = relativo ao `font-size` do **elemento pai imediato** (ou do próprio elemento, se usado em propriedades como `padding`).

```css
.tool-section { font-size: 1rem; }     /* 16px */
.tool-section p { font-size: 0.875em; } /* 0.875 × 16 = 14px */
```

### O perigo: efeito compounding (cascata de multiplicação)

```css
/* Cenário problemático */
.sidebar-nav            { font-size: 0.9em; }  /* 0.9 × 16 = 14.4px */
.sidebar-nav ul         { font-size: 0.9em; }  /* 0.9 × 14.4 = 12.96px */
.sidebar-nav ul li      { font-size: 0.9em; }  /* 0.9 × 12.96 = 11.66px */
/* Texto fica minúsculo sem você perceber! */
```

### Quando `em` é a escolha certa

O caso de uso mais valioso de `em` é em **componentes que precisam escalar proporcionalmente ao seu próprio texto** — especialmente botões:

```css
/* Seus botões do HTML: .btn, .btn--small, .btn--large */
.btn {
  font-size: 1rem;         /* base em rem — seguro */
  padding: 0.75em 1.5em;  /* em = relativo ao font-size DO PRÓPRIO .btn */
  border-radius: 0.375em;
}

/* Modificadores só mudam o font-size */
.btn--small { font-size: 0.875rem; }
/* padding automaticamente: 0.75 × 14px = 10.5px 1.5 × 14px = 21px */

.btn--large { font-size: 1.125rem; }
/* padding automaticamente: 0.75 × 18px = 13.5px 1.5 × 18px = 27px */
```

Resultado: `.btn--small`, `.btn--large` do seu HTML ficam **proporcionais** sem reescrever padding. Isso é elegância de arquitetura CSS.

### Regra prática definitiva

| Propriedade | Unidade recomendada | Motivo |
|---|---|---|
| `font-size` | **rem** | Evita compounding, respeita preferência do usuário |
| `padding` / `margin` em botões | **em** | Escala com o tamanho do texto do botão |
| `padding` / `margin` em layouts | **rem** | Consistência global |
| `border-radius` em componentes | **em** | Proporcional ao tamanho do componente |
| `line-height` | **sem unidade** (ex: `1.5`) | Relativo ao próprio `font-size` sem cascata |

---

## 5. Unidades de Viewport: `vw`, `vh` e as novas `sv*/lv*/dv*`

### O que é viewport

**Viewport** (área de visualização) é a área visível da janela do navegador — excluindo barras de rolagem, barra de endereços, etc.

```
vw = viewport width  → 1vw = 1% da LARGURA da viewport
vh = viewport height → 1vh = 1% da ALTURA da viewport
```

### O problema clássico com `vh` no mobile

Este é um dos bugs mais frustantes do CSS mobile, e afeta diretamente a sua splash page:

```css
/* ❌ Problema na sua splash page */
.page--splash {
  height: 100vh;
  /* No iPhone/Android com Chrome:
     A barra de endereços "encolhe" quando você rola para baixo.
     100vh é calculado SEM a barra → conteúdo fica maior que a tela visível
     → barra de scroll aparece na splash → experiência horrível */
}
```

**Visualizando o problema:**
```
Estado inicial (barra visível):
┌─────────────────┐
│ URL: exemplo.com│  ← barra de endereços (56px)
├─────────────────┤
│                 │
│  Sua splash     │  ← 100vh foi calculado aqui (sem a barra)
│   (cortada)     │
│                 │
└─────────────────┘
  ↕ scroll aparece ← problema!
```

### As novas unidades dinâmicas (2022–2026, suporte >92%)

Para resolver isso de vez, o CSS introduziu três famílias de unidades:

#### Família `sv*` — Small Viewport (menor área possível)
Calcula considerando que **todas as barras do browser estão visíveis**.

```css
svh  /* small viewport height */
svw  /* small viewport width  */
svi  /* small viewport inline (horizontal em pt-BR) */
svb  /* small viewport block  (vertical em pt-BR)   */
svmin, svmax
```

**Use quando:** o conteúdo não pode ser cortado em hipótese alguma (ex: um modal, um botão CTA crítico).

#### Família `lv*` — Large Viewport (maior área possível)
Calcula considerando que **todas as barras estão escondidas** (usuário rolou e elas sumiram).

```css
lvh, lvw, lvi, lvb, lvmin, lvmax
```

**Use quando:** backgrounds, imagens de cobertura, elementos decorativos que podem ser parcialmente ocultos sem problema.

#### Família `dv*` — Dynamic Viewport (muda em tempo real!)
**Atualiza em tempo real** conforme as barras aparecem/desaparecem.

```css
dvh, dvw, dvi, dvb, dvmin, dvmax
```

**Use quando:** altura de tela cheia que precisa ser sempre exata — especialmente `min-height` em páginas de entrada.

### Aplicando na sua splash page

```css
/* ✅ Solução completa para .page--splash */
.page--splash {
  min-height: 100vh;    /* fallback: navegadores sem suporte a dv* */
  min-height: 100dvh;   /* modern: se adapta dinamicamente à barra do browser */
  
  /* A segunda declaração sobrescreve a primeira em browsers modernos.
     Browsers antigos leem só a primeira (cascade normal). */
}
```

```css
/* ✅ Uso avançado: background que cobre tudo */
.page--splash::before {
  /* Imagem/gradiente de fundo que sempre preenche o máximo */
  min-height: 100lvh;   /* large: usa o máximo disponível para o background */
}

/* Conteúdo centralizado que nunca fica escondido */
.login-section {
  min-height: 100svh;   /* small: garante que o form sempre cabe na tela */
}
```

### vw para tipografia fluida (prévia do Tópico 3)

`vw` pode ser usado para criar textos que escalam com a largura da tela. Você vai aprender isso em profundidade no **Tópico 3** com `clamp()`, mas aqui vai um gostinho:

```css
/* Título da splash que cresce com a tela — sem media queries */
.login-section__title {
  /* min: 1.75rem (mobile), max: 3rem (desktop), meio: escala com a tela */
  font-size: clamp(1.75rem, 4vw, 3rem);
}
```

---

## 6. `%` — Para larguras de layout

**%** é relativo ao elemento **pai** — mas o comportamento varia por propriedade:

```css
/* width/height %: relativo à dimensão correspondente do pai */
.app-layout { width: 100%; }           /* 100% da largura do pai */
.app-sidebar { width: 260px; }         /* fixo em px — exceção válida para sidebar */
.dashboard-main { width: calc(100% - 260px); } /* você aprenderá calc() no Tópico 4 */

/* padding/margin %: SEMPRE relativo à LARGURA do pai (mesmo padding-top!) */
/* Isso é uma pegadinha clássica */
.hero { padding-top: 10%; }
/* Se o pai tem 1200px de largura → padding-top = 120px (não 10% da altura!) */
```

### Regra prática

```css
/* ✅ Use % para larguras de layout fluidas */
.tools-grid__item { width: 100%; }        /* ocupa toda a célula do grid */

/* ❌ Evite % para font-size — use rem */
.tool-card__name { font-size: 87.5%; }    /* confuso e frágil */
.tool-card__name { font-size: 0.875rem; } /* claro e previsível */
```

---

## 7. `ch` e `lh` — Unidades tipográficas modernas

### `ch` — Largura do caractere "0"

`1ch` = largura do caractere `0` na fonte atual. Ideal para **limitar comprimento de linha** para legibilidade ótima.

Pesquisas de tipografia indicam que **45–75 caracteres por linha** é o ideal para leitura confortável em telas.

```css
/* Nos cards do seu dashboard */
.tool-card__description {
  max-width: 55ch; /* ~55 caracteres → leitura confortável */
  /* Sem isso, em telas largas o texto se estica até 800px+ → ilegível */
}

/* Para o subtítulo da splash */
.login-section__subtitle {
  max-width: 40ch;
}
```

### `lh` — Line Height atual (novo! suporte ~88% em 2026)

`1lh` = valor atual do `line-height` do elemento. Útil para espaçamentos que precisam se alinhar ao ritmo tipográfico:

```css
/* Espaçamento entre seções alinhado ao grid tipográfico */
.tool-section {
  margin-block-end: 3lh; /* 3 alturas de linha — mantém ritmo vertical */
}

/* Ícones que precisam ter exatamente a altura de uma linha de texto */
.tool-card__icon {
  block-size: 1lh; /* se ajusta ao line-height do contexto */
}
```

---

## 8. Tabela de Decisão Rápida

Use esta tabela enquanto escreve CSS — você vai consultá-la muito no início:

| O que estou estilizando? | Unidade ideal | Motivo |
|---|---|---|
| `font-size` de qualquer elemento | `rem` | Respeita preferência do usuário |
| `padding`/`margin` em layouts | `rem` | Consistência global |
| `padding`/`margin` em botões | `em` | Proporcional ao texto do botão |
| `border`, `outline` | `px` | Detalhes fixos visuais |
| `border-radius` de botões | `em` | Proporcional ao tamanho |
| `border-radius` de cards | `px` ou `rem` | Fixo ou proporcional ao layout |
| `width`/`height` de layout | `%` ou `fr` (Grid) | Fluido |
| `width` de imagens/ícones | `px` ou `rem` | Fixo ou proporcional |
| Altura de tela cheia | `100dvh` + fallback `100vh` | Mobile-safe |
| Largura máxima de texto | `ch` | Legibilidade tipográfica |
| `line-height` | sem unidade (ex: `1.5`) | Relativo ao font-size sem cascata |
| Efeitos decorativos (`box-shadow`) | `px` | Valores visuais fixos |

---

## 9. Código Comentado — Aplicado ao seu HTML

Abaixo está o CSS inicial completo para o seu projeto, integrando tudo que vimos. Cada linha tem comentário explicando a decisão.

### 9.1 — Reset, Base e Variáveis

```css
/* ============================================================
   ARQUIVO: css/styles.css
   PROJETO: Portal de Ferramentas — Henry Hirth
   TÓPICO 1: Unidades de Medida aplicadas
   ============================================================ */


/* ------------------------------------------------------------
   1. BOX-SIZING GLOBAL
   A regra mais importante do CSS moderno.
   
   Comportamento padrão (sem isso):
     width: 200px + padding: 20px = 240px real ← confuso!
   
   Com border-box:
     width: 200px + padding: 20px = 200px real ← o padding "come" por dentro
   
   O seletor * + *::before + *::after garante que TODOS os elementos
   (incluindo pseudo-elementos ::before/::after) sigam essa regra.
   ------------------------------------------------------------ */
*,
*::before,
*::after {
  box-sizing: border-box;
}


/* ------------------------------------------------------------
   2. VARIÁVEIS GLOBAIS (Custom Properties)
   Definidas em :root para ficarem disponíveis em TODO o documento.
   
   :root é o elemento <html> com especificidade de pseudo-classe,
   o que garante que essas variáveis sobrescrevam qualquer outra.
   
   Você aprenderá Custom Properties em profundidade no Tópico 4.
   Por agora: pense nelas como constantes do seu design system.
   ------------------------------------------------------------ */
:root {

  /* --- TIPOGRAFIA em rem ---
     Base: 1rem = fonte padrão do navegador (geralmente 16px)
     NÃO defina font-size em px aqui — isso quebraria acessibilidade! */
  --font-size-xs:   0.75rem;    /* 12px — tool-section__count, badges */
  --font-size-sm:   0.875rem;   /* 14px — tool-card__description, labels */
  --font-size-base: 1rem;       /* 16px — texto padrão do corpo */
  --font-size-lg:   1.125rem;   /* 18px — destaques leves */
  --font-size-xl:   1.25rem;    /* 20px — app-header__app-name */
  --font-size-2xl:  1.5rem;     /* 24px — tool-section__title (h2) */
  --font-size-3xl:  2rem;       /* 32px — dashboard-main__title (h1) */
  --font-size-4xl:  2.5rem;     /* 40px — login-section__title (h1 splash) */

  /* --- ESPAÇAMENTOS em rem ---
     Sistema de 8pt grid: múltiplos de 0.5rem (8px na base).
     Isso cria ritmo visual consistente em todo o site. */
  --space-1:  0.25rem;  /* 4px  — micro-espaços */
  --space-2:  0.5rem;   /* 8px  — gap entre ícone e texto em botões */
  --space-3:  0.75rem;  /* 12px — padding interno de badges */
  --space-4:  1rem;     /* 16px — padding de cards, gap padrão */
  --space-6:  1.5rem;   /* 24px — padding do header, seções */
  --space-8:  2rem;     /* 32px — margem entre seções menores */
  --space-12: 3rem;     /* 48px — margem entre seções maiores */
  --space-16: 4rem;     /* 64px — margem hero, espaços generosos */
  --space-24: 6rem;     /* 96px — espaçamentos de página */

  /* --- CORES (você expandirá isso depois) --- */
  --color-primary:    #2563eb;  /* azul principal */
  --color-primary-hover: #1d4ed8;
  --color-surface:    #ffffff;
  --color-surface-2:  #f8fafc;  /* fundo de cards */
  --color-border:     #e2e8f0;
  --color-text:       #0f172a;
  --color-text-muted: #64748b;

  /* --- LARGURA MÁXIMA DE CONTEÚDO --- */
  --max-width-content: 75rem;   /* 1200px — container principal */
  --max-width-narrow: 40rem;    /* 640px — formulários, textos estreitos */

  /* --- BORDER RADIUS --- */
  --radius-sm:  0.25rem;  /* 4px */
  --radius-md:  0.5rem;   /* 8px — cards */
  --radius-lg:  0.75rem;  /* 12px */
  --radius-xl:  1rem;     /* 16px */
  --radius-full: 9999px;  /* pílula/círculo */
}


/* ------------------------------------------------------------
   3. BASE DO DOCUMENTO
   ------------------------------------------------------------ */
html {
  font-size: 100%;
  /* 100% = herda o padrão do navegador (normalmente 16px).
     NUNCA use 62.5% aqui — é um hack antigo que prejudica acessibilidade.
     Se o usuário mudou para 20px, 100% mantém isso.
     Se o usuário mudou para 20px, 62.5% reduziria para 12.5px — ruim! */
}

body {
  font-family: system-ui, -apple-system, 'Segoe UI', sans-serif;
  /* system-ui: usa a fonte padrão do sistema operacional.
     Sem downloads, sem FOUT (Flash of Unstyled Text), performance máxima.
     Você aprenderá a trocar por fontes do Google/custom no Tópico 7. */

  font-size: var(--font-size-base);
  /* 1rem — estabelece o tamanho base para todo o documento */

  line-height: 1.5;
  /* SEM UNIDADE = o melhor para line-height!
     Relativo ao font-size do elemento atual.
     1.5 × 16px = 24px de altura de linha (confortável para leitura).
     Se um filho tiver font-size: 24px, line-height = 36px automaticamente.
     Com px seria fixo e filhos com fonte maior ficariam esmagados. */

  color: var(--color-text);
  background-color: var(--color-surface);

  -webkit-font-smoothing: antialiased;
  /* Melhora a renderização de fontes no macOS/iOS.
     Sutil mas perceptível em textos finos. */
}
```

### 9.2 — Splash Page

```css
/* ============================================================
   SPLASH PAGE — Tela de Login
   Referência no seu HTML: <body class="page page--splash">
   ============================================================ */

.page--splash {
  min-height: 100vh;    /* fallback para browsers sem suporte a dvh */
  min-height: 100dvh;
  /* dvh = dynamic viewport height.
     Resolve o problema clássico do mobile onde 100vh é maior
     que a tela visível por conta da barra de endereços do Chrome.
     
     Suporte em 2026: >92% dos navegadores globais.
     A linha acima é o fallback — browsers antigos a usam.
     Browsers modernos sobrescrevem com a segunda declaração.
     Isso é chamado de "progressive enhancement" — ensinamento importante! */

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  /* Flexbox: você aprenderá isso em detalhe no Tópico 2.
     Por agora: centraliza o conteúdo verticalmente e horizontalmente. */

  padding: var(--space-4);
  /* rem via variável: 1rem = 16px de padding em todos os lados.
     Garante que em mobile o conteúdo não toque as bordas da tela. */

  background-color: var(--color-surface-2);
}


/* --- Header da splash: .splash__header --- */
.splash__header {
  margin-block-end: var(--space-8);
  /* margin-block-end = margin-bottom em idiomas ocidentais.
     É uma Logical Property — você aprenderá no Tópico 2.
     2rem = 32px de espaço entre o logo e o formulário. */

  text-align: center;
}

/* --- Logo: .splash__logo (figure do seu HTML) --- */
.splash__logo {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: var(--space-2);  /* 8px entre a imagem e a legenda */
  margin: 0;            /* reset do margin padrão do figure */
}

.logo__image {
  width: 5rem;    /* 80px — em rem, escala se o usuário mudar a fonte */
  height: 5rem;   /* preserva proporção — você pode usar aspect-ratio depois */
  /* Por que rem e não px aqui?
     Em um cenário extremo (usuário com 32px de base), o logo ficaria
     160px — proporcionalmente maior com o resto do site. Consistente! */
}

.logo__tagline {
  font-size: var(--font-size-sm);   /* 14px */
  color: var(--color-text-muted);
  line-height: 1.4;
}


/* --- Área principal do login: .splash__main --- */
.splash__main {
  width: 100%;
  max-width: var(--max-width-narrow);
  /* max-width: 40rem = 640px.
     Em rem: se o usuário tem base 20px, max-width = 800px.
     O formulário escala junto com o texto — nunca fica pequeno demais. */
}


/* --- Seção de login: .login-section --- */
.login-section {
  background: var(--color-surface);
  border: 1px solid var(--color-border);  /* px para borda fina — correto */
  border-radius: var(--radius-xl);         /* 1rem = 16px */
  padding: var(--space-8) var(--space-6);  /* 2rem 1.5rem = 32px 24px */
  width: 100%;
}

.login-section__title {
  font-size: var(--font-size-4xl);  /* 2.5rem = 40px */
  line-height: 1.2;
  /* line-height menor para títulos grandes — evita espaço excessivo */
  font-weight: 700;
  margin-block-end: var(--space-2);
  text-align: center;
}

.login-section__subtitle {
  font-size: var(--font-size-base);  /* 1rem */
  color: var(--color-text-muted);
  text-align: center;
  margin-block-end: var(--space-8);  /* 2rem espaço antes dos botões */
  max-width: 35ch;
  /* ch = ~35 caracteres de largura.
     Mantém o subtitle estreito e legível mesmo em telas largas. */
  margin-inline: auto;  /* centraliza horizontalmente */
}
```

### 9.3 — Botões (usando `em` para padding proporcional)

```css
/* ============================================================
   BOTÕES — Sistema escalável com em para padding
   Referência no HTML: .btn, .btn--google, .btn--primary,
                       .btn--ghost, .btn--danger,
                       .btn--small, .btn--large, .btn--full
   ============================================================ */

.btn {
  /* --- TIPOGRAFIA --- */
  font-size: 1rem;          /* rem: base segura */
  font-weight: 500;
  font-family: inherit;     /* herda a fonte do body */
  line-height: 1;           /* sem unidade: 1 × font-size atual */

  /* --- ESPAÇAMENTO com em ---
     em aqui = relativo ao font-size DO PRÓPRIO .btn (1rem = 16px).
     padding: 0.75em × 16px = 12px top/bottom
              1.5em × 16px  = 24px left/right
     
     MÁGICA: quando .btn--small muda font-size para 0.875rem (14px),
     o padding automaticamente vira: 0.75 × 14px = 10.5px, 1.5 × 14px = 21px.
     Proporcional sem reescrever nada! */
  padding: 0.75em 1.5em;

  /* --- FORMA --- */
  border-radius: 0.5em;     /* em: proporcional ao tamanho do botão */
  border: 1px solid transparent;  /* px para borda — correto */

  /* --- COMPORTAMENTO --- */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5em;               /* em: gap proporcional entre ícone e texto */
  cursor: pointer;
  text-decoration: none;    /* caso o botão seja um <a> */
  white-space: nowrap;
  transition: background-color 0.15s ease, border-color 0.15s ease;
  /* Transições: você aprenderá em detalhes no Tópico 6. */
}

/* --- Variante primária: .btn--primary --- */
.btn--primary {
  background-color: var(--color-primary);
  color: #ffffff;
  border-color: var(--color-primary);
}
.btn--primary:hover {
  background-color: var(--color-primary-hover);
  border-color: var(--color-primary-hover);
}

/* --- Variante Google: .btn--google --- */
.btn--google {
  background-color: #ffffff;
  color: #3c4043;
  border: 1px solid #dadce0;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);  /* px para sombra — correto */
}
.btn--google:hover {
  background-color: #f8f9fa;
  box-shadow: 0 2px 6px rgba(0,0,0,0.12);
}

/* --- Variante ghost: .btn--ghost --- */
.btn--ghost {
  background-color: transparent;
  color: var(--color-text);
  border-color: var(--color-border);
}
.btn--ghost:hover {
  background-color: var(--color-surface-2);
}

/* --- Variante danger: .btn--danger --- */
.btn--danger {
  background-color: #dc2626;
  color: #ffffff;
  border-color: #dc2626;
}

/* --- Modificadores de tamanho ---
   Só mudamos font-size. O padding (em) escala automaticamente! */
.btn--small { font-size: 0.875rem; }   /* 14px → padding: ~10.5px 21px */
.btn--large { font-size: 1.125rem; }   /* 18px → padding: ~13.5px 27px */

/* --- Modificador full width: .btn--full --- */
.btn--full {
  width: 100%;               /* % do pai — correto para layout */
  justify-content: center;
}

/* --- Ícone dentro do botão: .btn__icon --- */
.btn__icon {
  width: 1.25em;   /* em: escala com o font-size do botão */
  height: 1.25em;
  flex-shrink: 0;  /* não deixa o ícone encolher em flex */
}
```

### 9.4 — Cards de Ferramentas

```css
/* ============================================================
   TOOL CARDS — Os componentes mais repetidos do seu site
   Referência: .tool-card, .tool-card--produtividade, etc.
   ============================================================ */

.tools-grid {
  display: grid;
  /* Grid: você aprenderá em profundidade no Tópico 2.
     Por agora: grid responsivo automático */
  grid-template-columns: repeat(auto-fill, minmax(16rem, 1fr));
  /* 16rem = 256px mínimo por card — em rem para escalar com texto */
  gap: var(--space-4);  /* 1rem = 16px entre cards */
  
  list-style: none;  /* remove bullets da <ul> */
  padding: 0;
  margin: 0;
}

.tool-card {
  background: var(--color-surface);
  border: 1px solid var(--color-border);  /* 1px fixo — correto */
  border-radius: var(--radius-md);        /* 0.5rem = 8px */
  padding: var(--space-4);               /* 1rem = 16px */

  display: flex;
  align-items: flex-start;
  gap: var(--space-3);  /* 0.75rem = 12px entre ícone e corpo */

  transition: box-shadow 0.2s ease, border-color 0.2s ease;
}

.tool-card:hover {
  border-color: var(--color-primary);
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.1);  /* px para sombra */
}

.tool-card__icon-wrap {
  flex-shrink: 0;
}

.tool-card__icon {
  width: 2rem;    /* 32px em rem */
  height: 2rem;
  border-radius: var(--radius-sm);  /* 0.25rem = 4px */
}

.tool-card__body {
  min-width: 0;   /* fix: evita overflow em flex (você aprenderá no Tópico 2) */
}

.tool-card__name {
  font-size: var(--font-size-base);  /* 1rem */
  font-weight: 600;
  margin: 0 0 var(--space-1);       /* 0 0 4px */
  line-height: 1.3;
}

.tool-card__link {
  color: var(--color-text);
  text-decoration: none;
}
.tool-card__link:hover {
  color: var(--color-primary);
}

.tool-card__description {
  font-size: var(--font-size-sm);   /* 0.875rem = 14px */
  color: var(--color-text-muted);
  margin: 0;
  line-height: 1.5;
  max-width: 40ch;  /* ch: ~40 caracteres máximo para legibilidade */

  /* Truncar com ellipsis se muito longo */
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  /* Você aprenderá técnicas modernas de truncamento no Tópico 5. */
}
```

### 9.5 — Layout do Dashboard

```css
/* ============================================================
   DASHBOARD LAYOUT — Sidebar + Main
   Referência: .app-layout, .app-sidebar, .dashboard-main
   ============================================================ */

.app-header {
  display: flex;
  align-items: center;
  gap: var(--space-4);           /* 1rem entre elementos */
  padding: var(--space-3) var(--space-6);  /* 12px 24px */

  background: var(--color-surface);
  border-block-end: 1px solid var(--color-border);
  /* border-block-end = border-bottom (Logical Property) */

  position: sticky;
  top: 0;
  z-index: 100;  /* px não se aplica aqui — z-index é sem unidade */
  /* position sticky: mantém o header visível ao rolar */
}

.app-header__brand {
  display: flex;
  align-items: center;
  gap: var(--space-2);  /* 8px */
  margin-inline-end: auto;  /* empurra os outros elementos para a direita */
}

.app-header__logo {
  width: 2.5rem;   /* 40px em rem */
  height: 2.5rem;
}

.app-header__app-name {
  font-size: var(--font-size-xl);  /* 1.25rem = 20px */
  font-weight: 700;
}

/* --- Layout de duas colunas: .app-layout --- */
.app-layout {
  display: grid;
  grid-template-columns: 16rem 1fr;
  /* 16rem = 256px para a sidebar (rem: escala com preferência do usuário).
     1fr = todo o espaço restante para o conteúdo principal.
     Você aprenderá Grid em profundidade no Tópico 2. */

  min-height: calc(100dvh - 3.5rem);
  /* calc(): você aprenderá no Tópico 4.
     Aqui: altura total da viewport MENOS a altura do header (3.5rem ≈ 56px).
     Garante que o layout ocupe toda a tela sem scroll desnecessário.
     dvh = dynamic viewport height — seguro para mobile! */
}

.app-sidebar {
  border-inline-end: 1px solid var(--color-border);
  /* border-inline-end = border-right (Logical Property) */
  padding: var(--space-6) var(--space-4);  /* 24px 16px */
  
  position: sticky;
  top: 3.5rem;  /* abaixo do header fixo */
  height: calc(100dvh - 3.5rem);
  overflow-y: auto;  /* scroll interno na sidebar */
}

.dashboard-main {
  padding: var(--space-6) var(--space-8);  /* 24px 32px */
  max-width: var(--max-width-content);     /* 75rem = 1200px */
}

/* --- Seções de ferramentas --- */
.tool-section {
  margin-block-end: var(--space-12);  /* 3rem = 48px entre seções */
}

.tool-section__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-block-end: var(--space-4);   /* 1rem = 16px */
}

.tool-section__title {
  font-size: var(--font-size-2xl);  /* 1.5rem = 24px */
  font-weight: 700;
  margin: 0;
  display: flex;
  align-items: center;
  gap: var(--space-2);  /* 8px entre ícone e texto */
}

.tool-section__count {
  font-size: var(--font-size-xs);  /* 0.75rem = 12px */
  color: var(--color-text-muted);
  background: var(--color-surface-2);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-full);  /* pílula */
  padding: var(--space-1) var(--space-3);  /* 4px 12px */
}
```

---

## 10. Curiosidades e Hacks

### 💡 Hack 1: Converter px para rem mentalmente

Enquanto você não decorar os valores, use esta fórmula mental simples:

```
rem = px desejado ÷ 16
```

```
14px = 14 ÷ 16 = 0.875rem  ✓
24px = 24 ÷ 16 = 1.5rem    ✓
48px = 48 ÷ 16 = 3rem      ✓
```

No VS Code, a extensão **px to rem** (rsms) faz isso automaticamente — você digita `14px`, aperta um atalho e converte para `0.875rem`.

### 💡 Hack 2: O `dvh` pode tremer em alguns browsers antigos

Em versões antigas do Safari (pré-iOS 15.4), o `dvh` atualizava causando um "pulo" visual enquanto a barra de endereços aparecia/desaparecia. A solução é sempre manter o fallback:

```css
/* ✅ Estratégia à prova de balas */
.page--splash {
  min-height: 100vh;   /* 1º: fallback universal */
  min-height: 100svh;  /* 2º: Safari iOS 15.4+ (barra sempre visível) */
  min-height: 100dvh;  /* 3º: browsers modernos (dinâmico) */
}
```

Browsers leem todas as três mas aplicam **só a última que entendem**. Isso se chama **progressive enhancement** — a técnica de escrever CSS que funciona em qualquer browser, melhorando conforme o browser tem mais capacidade.

### 💡 Hack 3: `1lh` para alinhar ícones perfeitamente

Quer um ícone que fique EXATAMENTE alinhado com uma linha de texto, independente do font-size? `lh` resolve isso de forma elegante:

```css
/* Sem lh: você chuta o tamanho do ícone */
.recent-link img { width: 16px; height: 16px; }

/* Com lh: o ícone sempre tem a altura exata de uma linha de texto */
.recent-link img { 
  width: 1lh;
  height: 1lh;
  /* Perfeito! Muda o font-size do .recent-link e o ícone acompanha. */
}
```

### 🔗 Recursos para explorar

- **MDN — Unidades CSS:** https://developer.mozilla.org/pt-BR/docs/Learn/CSS/Building_blocks/Values_and_units
- **CanIUse — dvh/svh/lvh:** https://caniuse.com/viewport-unit-variants
- **CanIUse — lh unit:** https://caniuse.com/mdn-css_types_length_lh
- **Calculadora px→rem online:** https://nekocalc.com/px-to-rem-converter

---

## 11. Exercício Prático

### 🎯 Desafio: Estilize a Splash Page do zero

**Objetivo:** Criar o arquivo `css/styles.css` para a sua splash page usando exclusivamente as unidades aprendidas nesta aula.

**O que você precisa fazer:**

1. Crie a pasta `css/` e o arquivo `css/styles.css`
2. Cole o reset, as variáveis e as bases do item 9.1 deste guia
3. Estilize os seguintes elementos usando as unidades corretas:
   - `.page--splash` — altura de tela cheia (mobile-safe)
   - `.login-section` — card centralizado com padding em rem
   - `.login-section__title` — fonte em rem, tamanho grande
   - `.login-section__subtitle` — fonte menor, largura em `ch`
   - `.btn--google` — padding usando `em`
   - `.btn--primary` — padding usando `em`

**HTML de referência** (já está no seu guia de HTML5):
```html
<body class="page page--splash">
  <header class="splash__header">...</header>
  <main class="splash__main">
    <section class="login-section">
      <h1 class="login-section__title">Bem-vindo de volta</h1>
      <p class="login-section__subtitle">Entre com sua conta Google...</p>
      <button class="btn btn--google btn--large">Entrar com Google</button>
      <button class="btn btn--primary btn--full">Entrar com e-mail</button>
    </section>
  </main>
</body>
```

### ✅ Critérios de sucesso

Marque cada item ao completar:

- [ ] Nenhum `font-size` usa `px` (todos usam `rem` ou variáveis `rem`)
- [ ] O `.page--splash` usa `min-height: 100dvh` com fallback `100vh`
- [ ] Os botões usam `padding` com `em`
- [ ] O subtítulo tem `max-width` em `ch`
- [ ] A splash ocupa exatamente a tela no celular sem scroll indesejado (teste no DevTools: F12 → ícone de celular)
- [ ] Ao aumentar o zoom do navegador para 150% (Ctrl+Plus), **todo o layout escala proporcionalmente** sem quebrar

### 🧪 Como testar

1. Abra o arquivo HTML no Chrome
2. Pressione **F12** → clique no ícone de celular (Toggle device toolbar)
3. Selecione "iPhone 14" nas dimensões
4. Veja se a splash ocupa a tela toda sem scroll
5. Mude para "Pixel 7" e verifique novamente
6. Feche o DevTools e pressione **Ctrl+Plus** duas vezes → veja se tudo escala

---

## 12. Checklist e Próximo Passo

### ✅ O que você aprendeu hoje

- [ ] Por que `px` em `font-size` quebra acessibilidade e afeta conversões
- [ ] `rem`: relativo ao `<html>`, a unidade base para tipografia e espaçamento
- [ ] `em`: relativo ao pai, ideal para padding/border-radius de botões
- [ ] O problema do `100vh` no mobile e como `dvh/svh/lvh` resolvem isso
- [ ] `%`: para larguras de layout fluidas
- [ ] `ch`: para limitar comprimento de linha e melhorar legibilidade
- [ ] `lh`: para alinhar ícones ao ritmo tipográfico
- [ ] Sistema de variáveis CSS para centralizar toda a escala do projeto
- [ ] Progressive enhancement com fallbacks

### 🚀 O que vem a seguir — Tópico 2: Box Model Avançado

No próximo tópico você vai entender o mecanismo que controla **como cada elemento ocupa espaço** na tela: o Box Model. Você vai descobrir que `border-radius` tem poderes escondidos (dá para fazer formas orgânicas e ovais), que `outline` se comporta de forma completamente diferente de `border`, e vai dominar `margin-collapse` — o comportamento mais confuso para iniciantes.

---

> **O que achou desta aula, Henry? Ficou com alguma dúvida sobre alguma unidade específica? Conseguiu fazer o exercício e a splash apareceu corretamente no mobile?**
> 
> Me manda o resultado do teste de zoom — quero ver se tudo escalou direitinho. Quando você confirmar que está ok, avançamos para o **Tópico 2: Box Model Avançado**. 💪
