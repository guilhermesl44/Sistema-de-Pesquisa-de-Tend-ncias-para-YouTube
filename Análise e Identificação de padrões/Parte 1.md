# Parte 1: Identificação de Padrões de Performance de Títulos

![Fluxo Completo](./Imagens/auromacao2pt1.jpg)
## 📋 Visão Geral

Essa automação realiza a **análise comparativa de títulos** (positivos e negativos) a partir de dados obtidos de uma planilha Google Sheets.
O objetivo é **identificar padrões estruturais, gatilhos emocionais e deficiências linguísticas**, consolidando resultados em formato JSON para uso em etapas posteriores do pipeline.

---

## 🔄 Fluxo de Execução

```
[Google Sheets Trigger] → [Get row(s) in sheet] → [Edit Fields] → 
↳ (Caminho 1) → Code (JS3) → Aggregate → AI Agent → Edit Fields1
↳ (Caminho 2) → Sort → Code (JS2) → Aggregate1 → AI Agent1 → Edit Fields2
[Merge] → [Update row in sheet2]
```

---

## 📦 Sequências da Análise e Identificação de padrões

> A automação se divide em **4 partes lógicas**, representando as sub-etapas da análise.

---

### 🔹 Parte 1 – Leitura e Preparação dos Dados

#### 1. Google Sheets Trigger

**Tipo:** `n8n-nodes-base.googleSheetsTrigger`
**Função:** Inicia o fluxo automaticamente a cada 1 minuto.
**Configuração:**

* Modo de polling: `everyMinute`
* Documento: Teste Dev IA Pleno
* Aba: *Dados ordenados*
  **Comportamento:** Detecta novas ou alteradas linhas e dispara o workflow.

#### 2. Aggregate6

**Tipo:** `n8n-nodes-base.aggregate`
**Função:** Agrupa dados brutos vindos do trigger antes da leitura detalhada.

#### 3. Get row(s) in sheet

**Tipo:** `n8n-nodes-base.googleSheets`
**Função:** Busca as linhas ativas na aba *Dados ordenados* do documento.
**Credenciais:** Google Sheets account.

#### 4. Edit Fields

**Tipo:** `n8n-nodes-base.set`
**Função:** Cria campos padronizados para uso nos ramos seguintes.
**Campos:**

* `Titulo` ← coluna "Titulo "
* `row_number` ← coluna de índice.

---

### 🔹 Parte 2 – Análise Positiva (Top Títulos)

#### 1. Code in JavaScript3

**Tipo:** `n8n-nodes-base.code`
**Função:** Limita os dados aos 50 primeiros itens.

```js
const limit = 50;
return $input.all().slice(0, limit);
```

#### 2. Aggregate

**Tipo:** `n8n-nodes-base.aggregate`
**Função:** Agrupa a coluna `Titulo` para envio à IA.

#### 3. AI Agent

**Tipo:** `@n8n/n8n-nodes-langchain.agent`
**Função:** Analisa os títulos de melhor performance com base no prompt de diagnóstico positivo.

📍 **Prompt:**

```
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

---

>
> **Como o prompt limita o retorno:**
>
> * Exige JSON estrito e parseável.
> * Delimita a análise a padrões observáveis (sem inferir métricas).
> * Garante padronização de campos para uso automatizado.

#### 4. OpenAI Chat Model

**Tipo:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
**Modelo:** `gpt-4.1-mini`
**Função:** Fornece a capacidade LLM para o AI Agent.
**Credenciais:** OpenAI API (Guilherme).

#### 5. Edit Fields1

**Tipo:** `n8n-nodes-base.set`
**Função:** Armazena o output JSON gerado pela análise positiva no campo `Análise Positiva de Títulos`.

---

### 🔹 Parte 3 – Análise Negativa (Títulos de Baixa Performance)

#### 1. Sort

**Tipo:** `n8n-nodes-base.sort`
**Função:** Ordena as linhas por `row_number` em ordem decrescente.

#### 2. Code in JavaScript2

**Tipo:** `n8n-nodes-base.code`
**Função:** Limita os dados aos 50 primeiros itens para análise.

```js
const limit = 50;
return $input.all().slice(0, limit);
```

#### 3. Aggregate1

**Tipo:** `n8n-nodes-base.aggregate`
**Função:** Agrupa a coluna `Titulo` para enviar ao agente de análise negativa.

#### 4. AI Agent1

**Tipo:** `@n8n/n8n-nodes-langchain.agent`
**Função:** Executa a análise dos títulos de pior performance.

📍 **Prompt:**

```
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

---

## 🧾 Saída esperada (JSON estrito)

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
>
> **Como o prompt limita o retorno:**
>
> * Foca em identificar erros estruturais sem métricas externas.
> * Exige saída JSON padronizada com padrões negativos.
> * Garante consistência com a análise positiva para comparação automática.

#### 5. OpenAI Chat Model1

**Tipo:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
**Modelo:** `gpt-4.1-mini`
**Função:** Modelo de linguagem conectado ao AI Agent1.

#### 6. Edit Fields2

**Tipo:** `n8n-nodes-base.set`
**Função:** Registra o output JSON da análise negativa no campo `Análise Negativa de Títulos`.

---

### 🔹 Parte 4 – Unificação e Escrita dos Resultados

#### 1. Merge

**Tipo:** `n8n-nodes-base.merge`
**Função:** Combina as duas análises (positiva e negativa) por posição para gerar um único objeto.
**Configuração:** `combineByPosition`.

#### 2. Update row in sheet2

**Tipo:** `n8n-nodes-base.googleSheets`
**Função:** Atualiza a aba *Identificação de padrões* com os campos:

* `TituloPositivo` ← Análise Positiva de Títulos
* `TituloNegativo` ← Análise Negativa de Títulos
* `row_number` para referência de linha.
  **Credenciais:** Google Sheets account.

---

## 📊 Estrutura Final / Outputs Gerados

| Campo          | Descrição                                                     | Origem         |
| -------------- | ------------------------------------------------------------- | -------------- |
| TituloPositivo | JSON com padrões estruturais e gatilhos de melhor performance | AI Agent       |
| TituloNegativo | JSON com erros recorrentes e ausência de gatilhos             | AI Agent1      |
| row_number     | Índice da linha na planilha original                          | Sheets Trigger |

---

## 🎯 Interpretação / Uso dos Resultados

* Permite comparar automaticamente títulos positivos vs negativos.
* Fornece bases para recomendações estruturais e criação de novos títulos.
* Resultados ficam armazenados no Google Sheets para consulta e integração posterior.

---

## ⚙️ Configurações Técnicas

* **Trigger:** execução automática a cada 1 minuto.
* **Modelo LLM:** GPT-4.1-mini (OpenAI Chat).
* **Credenciais:** Google Sheets OAuth2 e OpenAI API.
* **Limite de itens:** 50 por tipo de análise.
* **Formatação:** JSON parseável com estrutura padronizada.

---

## 🔄 Integração com Outras Partes

**Input (vindo da Parte 1):** linhas ordenadas com títulos e métricas.
**Output (para Parte 3):** planilha atualizada com padrões positivos e negativos para uso em modelagem de roteiros ou visualizações posteriores.

---

## ✅ Resultado Final da Parte 1

**Input:** Títulos extraídos da planilha *Dados ordenados*.
**Output:** Análises positiva e negativa em formato JSON armazenadas na aba *Identificação de padrões*.
**Próxima Etapa:** Consumir esses dados para gerar insights ou roteiros automatizados.

---



