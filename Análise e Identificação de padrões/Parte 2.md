# 🎯 Parte 2 — Análise de Thumbnails & Padrões Visuais

## 📌 Objetivo

Esta parte do workflow lê **exatamente os 50 primeiros registros** da aba **“Dados ordenados”**, verifica se cada linha **já possui descrição de thumbnail**, gera a **descrição técnica** (quando ausente) e por fim **consolida padrões visuais** a partir dessas descrições, salvando o resultado na aba **“IDentificação de padrões”** (linha 2). 

---

## 🔄 Fluxo de Execução

```
Google Sheets Trigger (a cada 1 min)
→ Aggregate6
→ Get row(s) in sheet1 (Dados ordenados)
→ Edit Fields3 (mapeia URL da thumb em uma chave vazia "")
→ Code in JavaScript (limita a 50 itens)
→ Loop Over Items2 (iterate)
   ├─ If (DescriçãoThumb vazia?)
   │   └─ Analyze image (descreve a thumbnail) → Wait (20 s) → Update row in sheet (grava DescriçãoThumb)
   └─ (se já tiver) segue adiante
→ Aggregate2 (agrega items)
→ Get row(s) in sheet2 (Dados ordenados)
→ Edit Fields4 (seleciona DescriçãoThumb + row_number)
→ Code in JavaScript1 (limita a 50)
→ Aggregate3 (agrega apenas DescriçãoThumb)
→ AI Agent2 (extrai padrões das descrições)
→ Update row in sheet1 (IDentificação de padrões!Thumb = output do AI Agent2)
```



---

## 📦 Nós do Workflow (o que de fato acontece)

### 1) **Google Sheets Trigger**

**Tipo:** `googleSheetsTrigger` — **a cada 1 minuto**.
*Documento:* `Teste Dev IA Pleno` – `1XlZTABwHA456bYCFRiS8BFLxynypmo65pXeBeRV1WkQ`
*Aba:* **Dados ordenados** (`gid=304295346`). 

---

### 2) **Aggregate6**

Pass-through dos dados do gatilho para leitura. 

---

### 3) **Get row(s) in sheet1**

**Tipo:** `googleSheets (Read)` — lê linhas de **Dados ordenados**. 

---

### 4) **Edit Fields3**

**Tipo:** `set` — cria:

* `""` (chave vazia) = `{{$json.Thumb}}`
* `row_number` = `{{$json.row_number}}`

> A **chave vazia** guarda a URL usada pelo **Analyze image**. 

---

### 5) **Code in JavaScript**

**Limita a 50 itens**:

```js
const limit = 50;
return $input.all().slice(0, limit);
```



---

### 6) **Loop Over Items2**

Itera **registro a registro** sobre os 50 itens. 

---

### 7) **If**

Verifica se **`DescriçãoThumb` está vazia**:

* **Vazia →** executa **Analyze image**
* **Preenchida →** ignora e segue no loop


---

### 8) **Analyze image** — `gpt-4o-mini`

Gera **descrição técnica** da thumbnail (imagem da chave `""`).
**Saída:** JSON com `descricao_detalhada`, `tags_visuais`, `cores_predominantes`, `texto_detectado`, `composicao`, `gancho_visual`, `resumo_visual`. 

📍 **Prompt (preencher manualmente):**

```
<persona>
Você é um Especialista em Análise Visual e Engenharia de Clicks, com foco em entender thumbnails de vídeos e identificar os elementos visuais que mais atraem a atenção. 
Seu papel é descrever de forma objetiva e estruturada tudo que aparece na imagem, destacando os aspectos que contribuem para o desempenho e o engajamento visual. 
Sua análise é usada como insumo para IA de criação de conteúdo, então precisão e consistência são essenciais.
</persona>

<objetivo>
Gerar uma descrição detalhada e técnica da imagem (thumbnail) e, em seguida, um resumo padronizado com os elementos visuais mais relevantes para o engajamento.
</objetivo>

<regras_e_diretrizes>
✅ Descreva tudo que for visualmente identificável, de forma neutra e analítica.
✅ Detalhe composição/layout, cores predominantes e texto visível.
✅ Aponte elementos de destaque (setas, círculos, emojis, etc.).
✅ Gere tags visuais curtas e um resumo visual.
❌ Não invente ou interprete intenções.
</regras_e_diretrizes>

<formato_saida>
{
  "descricao_detalhada": "...",
  "tags_visuais": ["..."],
  "cores_predominantes": ["..."],
  "texto_detectado": "...",
  "composicao": {
    "enquadramento": "...",
    "layout": "...",
    "contraste": "...",
    "elementos_destaque": ["..."]
  },
  "gancho_visual": "...",
  "resumo_visual": "..."
}
</formato_saida>
```

---

### 9) **Wait (20 s)**

Intervalo para respeitar limites de API antes de gravar. 

---

### 10) **Update row in sheet**

Atualiza a linha atual (`row_number`) em **Dados ordenados** com:

* `DescriçãoThumb = {{$json.content}}`
* `Transcrição = "="`


---

### 11) **Aggregate2** → **Get row(s) in sheet2** → **Edit Fields4**

Relê e organiza `DescriçãoThumb` + `row_number` para consolidação. 

---

### 12) **Code in JavaScript1**

**Limita novamente a 50** antes de agregar.

```js
const limit = 50;
return $input.all().slice(0, limit);
```



---

### 13) **Aggregate3**

Agrega **somente `DescriçãoThumb`** para análise do agente. 

---

### 14) **AI Agent2** — Extração de Padrões

Executa análise **sobre as descrições geradas (TOP 50)** e produz **um relatório JSON de padrões visuais** (e anti-padrões, quando inferíveis do próprio conjunto), **sem comparar com “piores”**.
*Modelo:* `gpt-4.1-mini`. 

📍 **Prompt (preencher manualmente):**

```
<persona>
Você é um Analista de Padrões Visuais especializado em thumbnails de vídeos.
</persona>

<objetivo>
Extrair padrões recorrentes e diretrizes acionáveis a partir das descrições técnicas das thumbnails processadas.
</objetivo>

<regras_e_diretrizes>
✅ Baseado em evidências textuais das descrições.
✅ Organize por categoria: Cores, Composição, Texto, Elementos de Destaque, Expressões/Sujeitos.
✅ Traga diretrizes claras e por quê funcionam.
✅ Saída sempre em JSON padronizado.
</regras_e_diretrizes>

<formato_saida>
{
  "padrões_visuais": { ... },
  "anti_padrões": [ ... ],
  "insight_geral": "..."
}
</formato_saida>
```

> **Observação:** O nó recebe a lista agregada de `DescriçãoThumb` (dos **TOP 50**). Não há feed de “BOTTOM” no fluxo. 

---

### 15) **Update row in sheet1**

Grava o **JSON final de padrões** na aba **“IDentificação de padrões”**, **linha 2 → coluna `Thumb`**. 

---

## 🧪 O que é verificado / limitado

* Processa **no máximo 50 itens** por execução. 
* Só gera `DescriçãoThumb` quando **vazia**. 
* Resultado final vai para **IDentificação de padrões** (`gid=1109606750`), **linha 2, coluna `Thumb`**. 
* A imagem é lida de `Thumb` via **chave vazia `""`** usada no **Analyze image**. 

---

## 🗂️ Planilhas envolvidas

* **Dados ordenados** (`gid=304295346`) — origem dos registros e destino de `DescriçãoThumb`. 
* **IDentificação de padrões** (`gid=1109606750`) — recebe o **JSON de padrões** (coluna `Thumb`, linha 2). 

---

## ✅ Resultado

Até **50 thumbnails** são checadas; descrições faltantes são geradas e gravadas. Em seguida, o agente **compila padrões visuais** a partir **dessas descrições** e o JSON final é salvo em **IDentificação de padrões**, **linha 2 → coluna `Thumb`**. 


