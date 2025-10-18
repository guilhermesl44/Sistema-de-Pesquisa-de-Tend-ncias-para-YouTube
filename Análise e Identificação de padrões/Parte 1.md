
---

# 🧠  Parte 1 — Identificação de Padrões de Títulos

## 📋 Visão Geral

Esta etapa da automação é responsável por identificar **padrões estruturais e linguísticos** nos títulos de vídeos do nicho analisado.
Ela funciona em **duas análises paralelas**:

1. **Análise Positiva** → detecta padrões **comuns entre os títulos de melhor desempenho**, revelando estruturas replicáveis.
2. **Análise Negativa** → identifica padrões **ineficientes ou recorrentes entre os títulos de pior desempenho**, mostrando o que deve ser evitado.

O objetivo é gerar uma base sólida de **aprendizado automático sobre o estilo do nicho**, entregando dois blocos JSON detalhados (`TituloPositivo` e `TituloNegativo`) que alimentarão os próximos módulos da automação (geração e avaliação de ideias).

---

## 🔄 Fluxo de Execução

O processo segue esta sequência lógica:

1. 🔁 **Trigger manual (`When clicking 'Execute workflow'`)** – dispara a execução da automação.
2. 📊 **Leitura de dados (Google Sheets)** – obtém os títulos brutos da aba `Dados ordenados`.
3. 🧩 **Normalização de campos (`Edit Fields`)** – corrige nomes e formatos de colunas (como `Titulo ` → `Titulo`).
4. ⚖️ **Divisão em dois ramos**:

   * **Ramo superior** → análise dos **melhores títulos** (top 50).
   * **Ramo inferior** → análise dos **piores títulos** (bottom 50).
5. 🧠 **Execução dos agentes LLM**:

   * `AI Agent` (títulos bons)
   * `AI Agent1` (títulos ruins)
6. 🪄 **Geração de JSONs estruturados**:

   * `Análise Positiva de Títulos`
   * `Análise Negativa de Títulos`
7. 🔗 **Merge das saídas** – combina as duas análises em um só fluxo.
8. 📤 **Update no Google Sheets (`Update row in sheet2`)** – grava os resultados finais nas colunas correspondentes da aba `Identificação de padrões`.

---

## ⚙️ Nós do Workflow

### 🟢 **Trigger**

**Nome:** `When clicking 'Execute workflow'`
**Função:** executa manualmente o processo de análise.

---

### 📗 **Get row(s) in sheet)**

* **Fonte:** `Teste Dev IA Pleno`
* **Aba:** `Dados ordenados`
* **Campos utilizados:**

  * `row_number`
  * `Titulo` (com limpeza de espaço extra via node posterior)
* **Credenciais:** `Google Sheets account`

> Lê os títulos brutos que servirão de base para a análise estatística e semântica dos agentes.

---

### 🧱 **Edit Fields**

Normaliza e define campos para os próximos nós:

```js
Titulo = {{$json["Titulo "]}} // remove o espaço no nome da coluna
row_number = {{$json.row_number}}
```

> Essa padronização é essencial para evitar erros de parsing e manter consistência na referência dos dados.

---

### ⚖️ **Divisão dos Fluxos**

A partir do `Edit Fields`, o pipeline se divide em dois ramos independentes:

* **Ramo superior** → títulos **positivos (bons)**
* **Ramo inferior** → títulos **negativos (ruins)**

Ambos seguem o mesmo padrão de pré-processamento, variando apenas a lógica de ordenação e o prompt usado no agente.

---

### 🔹 Ramo 1 — Títulos Positivos

#### ① Code in JavaScript3

Limita o conjunto analisado:

```js
const limit = 50;
return $input.all().slice(0, limit);
```

#### ② Aggregate

Compacta os dados e mantém apenas o campo `Titulo`.

#### ③ OpenAI Chat Model

* Modelo: `gpt-4.1-mini`
* Credenciais: `Guilherme`

#### ④ AI Agent — “Analisador de Títulos de Alta Performance”

Executa o seguinte prompt:

```markdown
## System
Você é um **Especialista em Engenharia de Conteúdo, Psicologia do Click e Modelagem de Estruturas Virais**.  
Seu papel é analisar **títulos de vídeos de alta performance** e **identificar padrões replicáveis** com base em evidências observáveis.  
Você deve combinar **análise qualitativa (estrutural)** e **quantitativa (estatística)**, sem inferir dados externos (CTR, watchtime etc.).  
Sua resposta deve ser **JSON válido e parseável**, pronto para uso automatizado em um pipeline de geração de conteúdo.

---

## User
Analise os **quantidade títulos de melhor performance** do nicho: `nicho`.

### 📋 Dados recebidos
Cada item contém:
- **ID**  
- **Título** (ou "Titulo ")  
- **outlierScore** (métrica de destaque)  
- **Score Final** (0–100)  
- **Flags** (metadados de oportunidade)

---

## 🎯 Objetivo
Gerar um **raio-x completo dos títulos vencedores**, revelando:
1. **Padrões estruturais** (fórmulas narrativas)
2. **Frequência e suporte estatístico**
3. **Power words e gatilhos emocionais**
4. **Elementos formais** (números, símbolos, formato)
5. **Diretrizes práticas replicáveis**

---

## Processo

1. **Selecione os TOP títulos**
   - Use os *quantidade* com maior outlierScore
2. **Analise cada título**, identificando:
   - Estrutura narrativa (gatilho, número, tema, promessa, especificador)
   - Gatilhos emocionais (curiosidade, medo, urgência, autoridade, prova social)
   - Elementos formais (números, parênteses, dois-pontos, caps lock, aspas, interrogação)
   - Power words (palavras recorrentes de impacto)
   - Tipo de tema dominante (alimentos, exercícios, sintomas, vitaminas)
3. **Agrupe títulos similares** por estrutura abstrata (ex: “Gatilho + Problema + Número + Solução”)
4. **Calcule métricas globais:**
   - Comprimento em caracteres e palavras (média, mediana, min, max)
   - Frequência e percentual de cada elemento estrutural
   - Ocorrências de power words
5. **Classifique padrões:**
   - Apenas inclua padrões com ≥ 6 ocorrências (≥ 12%)
   - Outros padrões menores entram em `oportunidades_fracas`
6. **Selecione as 3 melhores estruturas** (por outlierScore médio)
7. **Gere o insight geral**, resumindo os achados de maior valor.

```

📤 **Saída esperada (JSON estrito):**

```json
{
  "nicho": "string",
  "amostras": 50,
  "estatisticas_texto": {
    "caracteres": { "media": 0, "mediana": 0, "min": 0, "max": 0 },
    "palavras": { "media": 0, "mediana": 0, "min": 0, "max": 0 }
  },
  "padroes_estruturais": [
    {
      "ranking": 1,
      "padrao": "Gatilho emocional + Problema + Número + Solução",
      "descricao": "Usa gatilho de alerta seguido de número e promessa concreta.",
      "frequencia": 0,
      "percentual": 0,
      "outlierScoreMedio": 0,
      "scoreFinalMedio": 0,
      "exemplos": ["...", "..."],
      "elementosChave": ["gatilho", "numero", "beneficio"],
      "diretriz": "Abra com um alerta ('ATENÇÃO', 'WARNING'), adicione um número e conclua com benefício específico.",
      "por_que_funciona": "Combina urgência e clareza, ativando o cérebro de sobrevivência e oferecendo recompensa imediata."
    }
  ],
  "elementos_estruturais": {
    "usa_numeros": { "contagem": 0, "percentual": 0 },
    "usa_parenteses": { "contagem": 0, "percentual": 0 },
    "usa_dois_pontos": { "contagem": 0, "percentual": 0 },
    "usa_caps": { "contagem": 0, "percentual": 0 },
    "usa_aspas": { "contagem": 0, "percentual": 0 },
    "usa_interrogacao": { "contagem": 0, "percentual": 0 },
    "listas_topX": { "contagem": 0, "percentual": 0 }
  },
  "gatilhos": [
    { "nome": "curiosidade", "contagem": 0, "percentual": 0, "exemplos": ["..."] },
    { "nome": "medo", "contagem": 0, "percentual": 0, "exemplos": ["..."] },
    { "nome": "autoridade", "contagem": 0, "percentual": 0, "exemplos": ["..."] },
    { "nome": "urgência", "contagem": 0, "percentual": 0, "exemplos": ["..."] },
    { "nome": "prova_social", "contagem": 0, "percentual": 0, "exemplos": ["..."] }
  ],
  "power_words": [
    { "termo": "warning", "contagem": 0 },
    { "termo": "foods", "contagem": 0 },
    { "termo": "after", "contagem": 0 },
    { "termo": "fix", "contagem": 0 }
  ],
  "oportunidades_fracas": [
    { "padrao": "Perguntas retóricas sobre sintomas", "contagem": 0 }
  ],
  "insights_gerais": {
    "estrutura_mais_comum": "string",
    "outlierScore_mais_alto": 0,
    "elemento_mais_recorrente": "string",
    "power_word_top": "string",
    "resumo": "1–2 frases sobre por que esses padrões funcionam psicologicamente."
  }
}
```

#### ⑤ Edit Fields1

Armazena a saída em um campo intermediário:

```js
Análise Positiva de Titulos = {{$json.output}}
```

---

### 🔸 Ramo 2 — Títulos Negativos

#### ① Sort

Ordena os dados em ordem **decrescente** de `row_number` (ou da métrica desejada).

#### ② Code in JavaScript2

Limita o conjunto a **50 itens**:

```js
const limit = 50;
return $input.all().slice(0, limit);
```

#### ③ Aggregate1

Reduz a entrada apenas ao campo `Titulo`.

#### ④ OpenAI Chat Model1

* Modelo: `gpt-4.1-mini`
* Credenciais: `Guilherme`

#### ⑤ AI Agent1 — “Diagnóstico de Títulos Ineficazes”

Executa o prompt:

```markdown
## System
Você é um **Especialista em Engenharia de Conteúdo e Psicologia do Click**, especializado em **diagnosticar títulos ineficazes**.  
Seu papel é identificar **erros estruturais**, **ausência de gatilhos** e **padrões que reduzem o desempenho** com base em evidências observáveis.  
Você trabalha **apenas com dados reais**, sem inferir métricas externas (CTR, watchtime etc.).  
Sua resposta deve ser **JSON válido e parseável**.

---

## User
Analise os **quantidade títulos de pior performance** do nicho: `nicho`.

### 📋 Dados recebidos
Cada item contém:
- **ID**  
- **Título** (ou "Titulo ")  
- **outlierScore** (métrica de destaque)  
- **Score Final** (0–100)  
- **Flags** (metadados de oportunidade)

---

## 🎯 Objetivo
Gerar um **raio-x dos erros recorrentes** nos títulos de baixa performance, revelando:
1. **Estruturas ineficazes ou genéricas**
2. **Ausência de gatilhos e power words**
3. **Problemas de formato e clareza**
4. **Padrões linguísticos associados a baixo desempenho**
5. **Oportunidades de reescrita e ajuste estrutural**

---

## Processo

1. **Selecione os títulos de pior performance**
   - Use os *quantidade* com menor outlierScore.
2. **Analise cada título** para identificar:
   - Falta de gatilho, número, promessa ou especificador.
   - Frases genéricas, vagas ou sem diferencial emocional.
   - Estruturas extensas, confusas ou desbalanceadas.
   - Uso excessivo de palavras fracas (ex: “coisa”, “importante”, “veja”).
   - Falta de foco (mistura de múltiplas ideias).
3. **Agrupe títulos por padrão negativo:**
   - Ex: “Sem gatilho + promessa vaga” ou “Informativo genérico sem emoção”.
4. **Calcule estatísticas globais:**
   - Comprimento médio em caracteres e palavras.
   - Frequência de ausência de elementos (sem número, sem gatilho, etc.).
   - Ocorrência de power words fracas.
5. **Classifique os padrões:**
   - Apenas inclua padrões negativos com ≥ 6 ocorrências (≥ 12%).
   - Os menos recorrentes entram em `anomalias`.
6. **Gere recomendações curtas e diretas** para corrigir os erros.
```

📤 **Saída esperada (JSON estrito):**

```json
{
  "nicho": "string",
  "amostras": 50,
  "estatisticas_texto": {
    "caracteres": { "media": 0, "mediana": 0, "min": 0, "max": 0 },
    "palavras": { "media": 0, "mediana": 0, "min": 0, "max": 0 }
  },
  "padroes_negativos": [
    {
      "ranking": 1,
      "padrao": "Sem gatilho + Promessa vaga",
      "descricao": "Títulos que não despertam emoção nem comunicam benefício concreto.",
      "frequencia": 0,
      "percentual": 0,
      "outlierScoreMedio": 0,
      "scoreFinalMedio": 0,
      "exemplos": ["...", "..."],
      "problemas_comuns": [
        "Falta de emoção",
        "Sem número ou benefício específico",
        "Palavras genéricas"
      ],
      "recomendacao": "Adicionar gatilho emocional e promessa clara de transformação.",
      "impacto_estimado": "reduz engajamento inicial por falta de estímulo visual e emocional"
    }
  ],
  "elementos_estruturais": {
    "sem_numero": { "contagem": 0, "percentual": 0 },
    "sem_gatilho": { "contagem": 0, "percentual": 0 },
    "sem_promessa": { "contagem": 0, "percentual": 0 },
    "muito_longo": { "contagem": 0, "percentual": 0 },
    "titulo_confuso": { "contagem": 0, "percentual": 0 },
    "usa_palavras_fracas": { "contagem": 0, "percentual": 0 }
  },
  "power_words_fracas": [
    { "termo": "coisa", "contagem": 0 },
    { "termo": "importante", "contagem": 0 },
    { "termo": "veja", "contagem": 0 },
    { "termo": "entenda", "contagem": 0 }
  ],
  "gatilhos_ausentes": [
    { "nome": "curiosidade", "faltando_em": 0, "percentual": 0 },
    { "nome": "urgência", "faltando_em": 0, "percentual": 0 },
    { "nome": "autoridade", "faltando_em": 0, "percentual": 0 },
    { "nome": "prova_social", "faltando_em": 0, "percentual": 0 }
  ],
  "anomalias": [
    { "padrao": "Título excessivamente técnico", "ocorrencias": 0 },
    { "padrao": "Mistura de dois temas sem conexão", "ocorrencias": 0 }
  ],
  "insights_gerais": {
    "erro_mais_comum": "string",
    "elemento_mais_ausente": "string",
    "estrutura_mais_ineficaz": "string",
    "resumo": "1–2 frases diretas sobre o que mais compromete a performance dos títulos."
  }
}

```

#### ⑥ Edit Fields2

Armazena a saída intermediária:

```js
Análise Negativa de Titulos = {{$json.output}}
```

---

### 🔗 **Merge**

Tipo: `combineByPosition`

> Junta a análise positiva (porta 0) e negativa (porta 1) lado a lado, mantendo correspondência posicional.

---

### 📤 **Update row in sheet2**

Atualiza a aba **IDentificação de padrões** com:

```js
TituloPositivo = {{$json["Análise Positiva de Titulos"]}}
TituloNegativo = {{$json["Análise Negativa de Titulos"]}}
row_number = 2
```

* **Documento:** `Teste Dev IA Pleno`
* **Aba destino:** `IDentificação de padrões`
* **Credenciais:** `Google Sheets account`

---

## 📊 Estrutura Final no Sheets

| row_number | TituloPositivo            | TituloNegativo        | Thumb | Roteiro | Lacunas |
| ---------- | ------------------------- | --------------------- | ----- | ------- | ------- |
| 2          | JSON (padrões vencedores) | JSON (padrões fracos) | —     | —       | —       |

> Essa etapa não preenche as colunas `Thumb`, `Roteiro` e `Lacunas`.
> Elas serão usadas nas partes seguintes (Bloco 2 e Bloco 3).

---

## 🎯 Interpretação e Benefícios

| Tipo de Análise | Propósito                                        | Resultado                                 |
| --------------- | ------------------------------------------------ | ----------------------------------------- |
| **Positiva**    | Descobrir o que torna os títulos virais          | Estruturas e gatilhos de alta performance |
| **Negativa**    | Identificar erros recorrentes e promessas fracas | Diagnóstico e recomendações práticas      |

Essas duas saídas são os **pilares analíticos** do pipeline — todos os agentes seguintes (gerador de ideias, avaliador, roteirista e designer de thumb) usam esses JSONs como insumo.

---

## ⚙️ Configurações Técnicas

* **Modelo LLM:** `gpt-4.1-mini`
* **Formato de saída:** JSON **parseável e sem Markdown**
* **Limite de itens por análise:** `50`
* **Método de merge:** `combineByPosition`
* **Atualização de linha:** `row_number = 2`
* **Campos sensíveis a nomes:** `"Titulo "` (com espaço) deve ser normalizado para `"Titulo"`

---

quer que eu siga agora com a **Parte 2 · Bloco 2 — Geração, Avaliação e Explicação de Ideias** no mesmo estilo (com os três agentes + benchmarks + integração de lacunas)?

