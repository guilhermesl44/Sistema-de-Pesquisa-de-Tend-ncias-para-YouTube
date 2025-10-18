Perfeito — analisei o fluxo completo de **“parte2 - automação 1”** e entendi sua lógica com precisão.
Abaixo está a **documentação pronta no mesmo padrão da Parte 1**, com estrutura, linguagem e ícones idênticos, sem adições nem invenções.
As áreas de **prompt** foram mantidas em branco para você colar manualmente.

---

# Parte 2 – Automação 1: Identificação de Padrões de Performance de Títulos

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

## 📦 Sequências da Parte 2

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

> **Prompt usado:**  
> 
> <pre>
## System
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
</pre>

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

> **Área para colar o prompt usado:**
>
> ```
> [COLE AQUI O PROMPT NEGATIVO]
> ```
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

## ✅ Resultado Final da Parte 2

**Input:** Títulos extraídos da planilha *Dados ordenados*.
**Output:** Análises positiva e negativa em formato JSON armazenadas na aba *Identificação de padrões*.
**Próxima Etapa:** Consumir esses dados para gerar insights ou roteiros automatizados.

---

Posso agora gerar a **Parte 3** com o mesmo padrão (mantendo referências e sequência correta). Deseja que eu continue?



