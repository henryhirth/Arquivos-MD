# Base de Conhecimento: Engenharia de Prompt e Fluxo de Trabalho com IAs

> Documento sintetizado a partir de sessões de conversa com IA. Foco: uso estratégico de múltiplas ferramentas para pesquisa complexa, prompt engineering de nível sênior e automação de workflows.

---

## Sumário

1. [Conceitos Fundamentais](#1-conceitos-fundamentais)
2. [Anatomia de um Prompt de Nível Sênior](#2-anatomia-de-um-prompt-de-nível-sênior)
3. [Frameworks de Prompt](#3-frameworks-de-prompt)
4. [O Arquivo de Contexto Pessoal](#4-o-arquivo-de-contexto-pessoal)
5. [Divisão Tática de Ferramentas](#5-divisão-tática-de-ferramentas)
6. [Pipeline de Pesquisa: Do Zero ao Manual](#6-pipeline-de-pesquisa-do-zero-ao-manual)
7. [Técnicas Avançadas](#7-técnicas-avançadas)
8. [Prompts Prontos para Uso](#8-prompts-prontos-para-uso)
9. [Sugestões de Continuidade](#9-sugestões-de-continuidade)

---

## 1. Conceitos Fundamentais

### Chaining (Encadeamento de IAs)
A saída de uma IA alimenta a entrada de outra. É a base da engenharia de prompt avançada e o núcleo de todo fluxo de trabalho multimodelo eficiente.

### Ontologia do Problema
Antes de buscar respostas, mapeie o *universo conceitual* do tema: quais são os conceitos, regras e variáveis que o regem. Isso evita perguntas genéricas e respostas rasas.

### Desmembramento Atômico
Nunca tente resolver um problema complexo em um único prompt. Quebre-o em unidades mínimas processáveis e resolva cada uma separadamente.

### Filtro de Exclusão
Em vez de pesquisar todos os itens de uma lista, defina critérios de eliminação primeiro. Reduzir 50 opções para 8 antes de usar uma IA mais cara (ex: Claude) maximiza a profundidade por token.

---

## 2. Anatomia de um Prompt de Nível Sênior

O modelo **CILO** (Contexto-Input-Lógica-Output) é o padrão técnico para prompts de alta densidade. Trate cada prompt como uma função de software.

### A. Contexto e Restrição (*The Sandbox*)
Define o universo de busca e o que a IA **não** pode fazer.

```
"Aja como um consultor de imigração para profissionais de STEM.
Considere apenas países com IDH acima de 0.850 e leis de imigração
facilitadas para egressos de cursos de tecnologia."
```

### B. Variáveis de Entrada (*The Data*)
Injete dados específicos para eliminar respostas genéricas.

```
"Utilize os seguintes critérios:
1. Custo de vida mensal < $1.500 USD
2. Proximidade de hubs de tecnologia
3. Existência de vistos de busca de emprego pós-graduação."
```

### C. Lógica de Processamento (*The Algorithm*)
Dite **como** a IA deve pensar, não apenas o que produzir.

```
"Realize uma análise SWOT para cada país. Em seguida, aplique
peso 0.7 para 'Empregabilidade' e 0.3 para 'Custo de Vida'
para gerar um ranking final."
```

### D. Especificação de Saída (*The Schema*)
Defina a estrutura técnica exata do arquivo de retorno.

```
"Gere a resposta em Markdown estruturado, com tabelas comparativas,
blocos de código para fórmulas financeiras e uma seção de
'Referências de Busca' ao final."
```

---

## 3. Frameworks de Prompt

| Framework | Quando Usar | Estrutura |
| :--- | :--- | :--- |
| **RTF** (Role-Task-Format) | Tarefas diretas de baixa complexidade | "Atue como [Papel]. Execute [Tarefa]. Entregue em [Formato]." |
| **Chain of Thought (CoT)** | Lógica sequencial, decisões complexas, cálculos | Inclua: *"Pense passo a passo. Primeiro analise X, depois compare com Y, e só então conclua Z."* |
| **Few-Shot Prompting** | Tom de voz específico ou estrutura de dados proprietária | Forneça 2–3 pares de Input/Output como exemplo antes do input real. |
| **CILO** | Pesquisas complexas e multivariáveis | Veja Seção 2. |
| **Meta-Prompt** | Quando você não sabe por onde começar | Peça à IA para gerar as perguntas antes de gerar respostas. |

### Chain of Thought — Exemplo de Ativação
Inserir a instrução abaixo força o modelo a exibir o raciocínio antes de concluir, reduzindo drasticamente alucinações:

> *"Pense passo a passo antes de responder."*

### Few-Shot — Exemplo de Estrutura
```
Input: Engenheiro de Software
Output: Profissional de Tecnologia, foco em backend.

Input: Analista de Dados
Output: Especialista em extração e visualização de métricas.

Input: [seu novo dado]
Output:
```

---

## 4. O Arquivo de Contexto Pessoal

Crie um arquivo `CONTEXTO_SENIOR.md` e cole-o no início de sessões importantes. Ele elimina "rodadas de ajuste" e padroniza o comportamento da IA.

### Template Base

```markdown
**Role:** Atue como um Arquiteto de Soluções e Pesquisador Acadêmico Sênior.

**Público-alvo:** Sou um estudante de computação de alto desempenho,
focado em sistemas complexos e carreira internacional.

**Estilo de Resposta:**
- Nível de Abstração: Baixo (detalhes técnicos, sem generalizações).
- Formato: Markdown estruturado com tabelas e blocos de código.
- Critério de Verdade: Dados quantitativos, links oficiais e
  requisitos técnicos.
- Proibição: Não use introduções motivacionais, frases de efeito
  ou resumos óbvios. Vá direto ao ponto técnico.

**Objetivo:** [Descreva seu objetivo de pesquisa atual aqui.]
```

### Bloco de Tom de Voz (adicione ao arquivo)

```markdown
Se eu perguntar sobre [TEMA], estruture a resposta como uma
Documentação Técnica de Sistema. Use terminologia precisa (ex: em vez
de 'ajuda do governo', use 'subsídios governamentais e incentivos
fiscais para talentos STEM'). Se houver incerteza estatística,
apresente o intervalo de confiança ou a fonte da divergência.
```

---

## 5. Divisão Tática de Ferramentas

| Ferramenta | Função Principal | Prompt Ideal |
| :--- | :--- | :--- |
| **Gemini** | Estratégia, planejamento e quebra de problemas | "Crie um framework de 10 critérios para avaliar [TEMA]." |
| **Perplexity / Phind** | Mineração de dados em tempo real e extração de links | Buscas granulares com termos técnicos e ano (ex: "DAAD scholarships 2026 CS"). |
| **Claude** | Síntese final e redação de documentação densa | Contexto + dados brutos → "Gere um manual técnico em MDL." |
| **NotebookLM** | Repositório de longo prazo e cruzamento de insights | Suba os manuais gerados e pergunte sobre contradições entre fontes. |

### Regra de Ouro para o Claude (poucos tokens)
A **primeira mensagem** deve sempre conter: Arquivo de Contexto + dados brutos coletados nas outras ferramentas. Não desperdice rodadas com apresentações.

---

## 6. Pipeline de Pesquisa: Do Zero ao Manual

### Fase 1 — Descoberta / Ontologia (Gemini)
**Objetivo:** Mapear o domínio antes de executar.

```
"Aja como um Arquiteto de Sistemas de Informação. Analise o domínio
'[TEMA]'. Identifique os 5 vetores críticos que definem sucesso ou
falha nesse processo. Para cada vetor, liste 3 termos técnicos
obrigatórios para buscas em bases governamentais/acadêmicas."
```

### Fase 2 — Mineração de Dados Brutos (Perplexity)
**Objetivo:** Coletar matéria-prima com fontes verificáveis.

```
"Search for: '[TERMO TÉCNICO] [ANO]' in [PAÍS/INSTITUIÇÃO].
Provide direct links to official edicts or government sources."
```

### Fase 3 — Síntese e Arquitetura (Claude)
**Objetivo:** Transformar dados brutos em documentação de nível sênior.

```
"Suba este arquivo de contexto e estes dados brutos. Como especialista
em sistemas, processe essas informações e gere um manual de execução.
Identifique os gargalos críticos onde a maioria falha e proponha uma
solução técnica para cada um."
```

### Fase 4 — Retenção e Cruzamento (NotebookLM)
**Objetivo:** Conectar os manuais gerados e identificar contradições.

```
"Quais são as contradições entre [Sistema A] e [Sistema B]
nos documentos carregados?"
```

---

## 7. Técnicas Avançadas

### Agrupamento por Similaridade (para listas grandes)
Ao invés de pesquisar 50 itens individualmente:

1. **Triagem (Gemini):** Agrupe em 5 blocos por similaridade (ex: UE, Commonwealth, Tigres Asiáticos).
2. **Pesquisa em lote (Perplexity):** Pesquise o bloco, não o item.
3. **Refino (Claude):** Entregue apenas os 5–10 finalistas para análise profunda.

### Prompt de Extração de Parâmetros (Meta-Prompt)
Use quando não souber por onde começar:

```
"Quero pesquisar sobre [ASSUNTO], mas quero evitar respostas
genéricas e economizar interações.

Sua tarefa: Atue como um Engenheiro de Prompt Sênior. Entreviste-me
fazendo as 5 perguntas técnicas mais críticas que eu ainda não
respondi, para que você possa gerar na sequência um Mega-Prompt
de Camadas ajustado ao meu caso.

Não responda o assunto ainda. Apenas faça as perguntas."
```

### Iteração por Elevação de Nível
Se a resposta vier rasa, use este comando de refinamento:

```
"Isso foi genérico. Use terminologia de nível sênior e foque em
dados quantitativos. O que um consultor de elite me diria
que você ainda não disse?"
```

---

## 8. Prompts Prontos para Uso

### Prompt de Partida Ideal (Zero-Start)

```markdown
[IDENTIDADE E PERSONA]
Atue como um Arquiteto de Carreira Internacional e Consultor
especializado em [ÁREA]. Sou um profissional de alto desempenho
focado em [OBJETIVO]. Ignore introduções genéricas e linguagem
simplista; trate-me como um par sênior.

[CONTEXTO ATUAL]
Utilizo o fluxo: Gemini (estratégia) → Perplexity (dados) →
Claude (síntese MDL). Preciso de um framework para analisar
[N itens] e filtrá-los para um Top [X] de viabilidade real.

[REGRAS DE EXECUÇÃO]
1. Use o método de Filtragem por Barreira de Entrada.
2. Apresente respostas em Markdown estruturado com tabelas
   e árvores de decisão.
3. Explique a mecânica técnica por trás de termos especializados.

[DADOS PARA PROCESSAMENTO]
- Área/Stack: [INSERIR]
- Idioma: [INSERIR]
- Orçamento: [INSERIR]
- Prazo: [INSERIR]

[TAREFA]
Com base nessas informações, desenhe o pipeline de pesquisa e
me dê os 3 primeiros prompts que devo levar ao Perplexity
para minerar dados de 2026.
```

### Prompt de Análise Profunda por País/Tema

```markdown
"Baseado no arquivo de contexto enviado, analise os dados abaixo
sobre [TEMA/PAÍS]:

1. Explique [CONCEITO A] e como ele se traduz para [SEU CONTEXTO].
2. Detalhe as exigências de [VARIÁVEL CRÍTICA] para 2026 e
   estratégias de mitigação.
3. Liste 3 [ENTIDADES/EMPRESAS/INSTITUIÇÕES] que atendem
   ao critério [X].
4. Formate tudo como um capítulo de documentação técnica,
   incluindo uma tabela de 'Prós e Contras Técnicos'."
```

### Prompt de Filtragem em Lote

```markdown
"Atue como um analista de dados. Receba esta lista de [N] itens.
Aplique um filtro de 'Barreira de Entrada': remova os que não
atendem a [CRITÉRIO 1] ou [CRITÉRIO 2].
Justifique tecnicamente cada exclusão em uma tabela."
```

---

## 9. Sugestões de Continuidade

### Opção A — Aplicação Imediata: Construir seu Arquivo de Contexto
Pegue o template da Seção 4, preencha com seus dados reais (stack, objetivos, restrições) e salve como `CONTEXTO_SENIOR.md`. Teste-o em uma sessão nova do Claude ou Gemini e mensure a diferença na qualidade das respostas em relação a sessões sem o arquivo.

### Opção B — Aprofundamento Técnico: Dominar Chain of Thought
Escolha um problema real e complexo que você esteja enfrentando agora. Resolva-o duas vezes: uma com um prompt simples e outra com CoT explícito (detalhando os passos lógicos que a IA deve seguir). Compare os resultados e documente o delta de qualidade. Isso calibra sua intuição sobre quando cada framework se aplica.

### Opção C — Automação de Workflow: Montar seu Pipeline Completo
Execute o pipeline da Seção 6 de ponta a ponta em um tema de seu interesse, usando as quatro ferramentas na sequência correta (Gemini → Perplexity → Claude → NotebookLM). O objetivo não é o resultado final, mas internalizar o *ritmo* do processo: quando cada ferramenta entra, o que entra nela e o que sai dela. Após uma execução completa, você terá um template reutilizável para qualquer pesquisa futura.
