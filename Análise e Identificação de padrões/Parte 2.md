# 🎯 Parte 2  — Análise de Thumbnails & Padrões Visuais

## 📌 Objetivo

Esta parte do workflow lê **exatamente os 50 primeiros registros** da aba **“Dados ordenados”**, verifica se cada linha **já possui descrição de thumbnail**, gera a **descrição técnica** (quando ausente) e por fim **consolida padrões visuais** a partir dessas descrições, salvando o resultado na aba **“IDentificação de padrões”** (linha 2).

---

## 🔄 Fluxo de Execução

```
Manual Trigger
→ Get row(s) in sheet1 (Dados ordenados)
→ Edit Fields3 (mapeia URL da thumb em uma chave vazia "")
→ Code in JavaScript (limita a 50 itens)
→ Loop Over Items2 (iterate)
   ├─ If (DescriçãoThumb vazia?)
   │   └─ Analyze image (descreve a thumbnail) → Wait (20s) → Update row in sheet (grava DescriçãoThumb)
   └─ (se já tiver) segue adiante
→ Aggregate2 (agrega items)
→ Get row(s) in sheet2 (Dados ordenados)
→ Edit Fields4 (seleciona DescriçãoThumb + row_number)
→ Code in JavaScript1 (limita a 50)
→ Aggregate3 (agrega apenas DescriçãoThumb)
→ AI Agent2 (compara TOP vs BOTTOM e extrai padrões)
→ Update row in sheet1 (IDentificação de padrões!Thumb = output do AI Agent2)
```

---

## 📦 Nós do Workflow (o que **de fato** acontece)

### 1) **When clicking ‘Execute workflow’**

**Tipo:** `manualTrigger`
Disparo manual do fluxo.

---

### 2) **Get row(s) in sheet1**

**Tipo:** `googleSheets (Read)`

* **Documento:** `Teste Dev IA Pleno` (`1XlZTABwHA456bYCFRiS8BFLxynypmo65pXeBeRV1WkQ`)
* **Aba:** **Dados ordenados** (`gid=304295346`)
* Lê todas as linhas da aba e envia ao próximo nó.

---

### 3) **Edit Fields3**

**Tipo:** `set`
Cria duas propriedades no item:

* `""` (chave de **nome vazio**) recebendo `{{$json.Thumb}}`
* `row_number` recebendo `{{$json.row_number}}`

> Observação: essa chave vazia `""` é usada adiante como **fonte de URL** da imagem.

---

### 4) **Code in JavaScript**

**Tipo:** `code`

```js
const limit = 50;
return $input.all().slice(0, limit);
```

Limita o fluxo aos **50 primeiros itens**.

---

### 5) **Loop Over Items2**

**Tipo:** `splitInBatches`
Itera registro a registro (em lotes) sobre **esses 50 itens**.

---

### 6) **If**

**Tipo:** `if`
Condição: verifica se **`DescriçãoThumb` está vazia** (usa o valor vindo do *Get row(s) in sheet1*).

* **Verdadeiro (vazia)** → chama **Analyze image**
* **Falso (já preenchida)** → **pula** a análise e retorna ao loop

---

### 7) **Analyze image**

**Tipo:** `openAi (Vision / analyze)` — **Modelo:** `gpt-4o-mini`

* **Prompt:** persona de **Análise Visual** (descreve tecnicamente a thumbnail).
* **imageUrls:** `={{ $json[""] }}` → usa a URL gravada na **chave vazia** `""` criada no *Edit Fields3*.
* **Saída esperada:** JSON com `descricao_detalhada`, `tags_visuais`, `cores_predominantes`, `texto_detectado`, `composicao`, `gancho_visual`, `resumo_visual`.

> Importante: este nó **não reescreve** nada sozinho — apenas **gera** a descrição.

---

### 8) **Wait**

**Tipo:** `wait`
Espera **20 segundos** antes de escrever no Sheets (buffer para processamento/limites de API).

---

### 9) **Update row in sheet**

**Tipo:** `googleSheets (Update)` — **Aba:** **Dados ordenados**

* **Matching:** `row_number`
* **Escreve:**

  * `DescriçãoThumb = {{$json.content}}` (conteúdo do Analyze image)
  * `Transcrição = "="` (literal, conforme configurado)

> Resultado: cada linha que **não tinha** descrição passa a ter **DescriçãoThumb** preenchida.

---

### 10) **Aggregate2**

**Tipo:** `aggregate (aggregateAllItemData)`
Agrega os itens do loop para seguir em bloco.

---

### 11) **Get row(s) in sheet2**

**Tipo:** `googleSheets (Read)` — **Aba:** **Dados ordenados**
Lê novamente a aba com os **dados já atualizados** (incluindo as descrições recém-escritas).

---

### 12) **Edit Fields4**

**Tipo:** `set`
Seleciona apenas:

* `DescriçãoThumb = {{$json["DescriçãoThumb"]}}`
* `row_number = {{$json.row_number}}`

---

### 13) **Code in JavaScript1**

**Tipo:** `code`

```js
const limit = 50;
return $input.all().slice(0, limit);
```

Mantém apenas **50** itens para a etapa de padrões.

---

### 14) **Aggregate3**

**Tipo:** `aggregate`
Agrega **somente** o campo `DescriçãoThumb` para compor o input do próximo agente.

---

### 15) **AI Agent2**

**Tipo:** `agent` + **OpenAI Chat Model2** (`gpt-4.1-mini`)

* Recebe **apenas** a coleção de `DescriçãoThumb` (agregada).
* **System Message:** instruções para **comparar descrições** e **extrair padrões visuais** (cores, composição, texto, elementos de destaque, expressões/sujeitos), **anti-padrões** e **insight geral**.
* **Saída:** JSON estruturado em `padrões_visuais`, `anti_padrões`, `insight_geral`.

---

### 16) **Update row in sheet1**

**Tipo:** `googleSheets (Update)` — **Aba:** **IDentificação de padrões** (`gid=1109606750`)

* **Matching:** `row_number = 2`
* **Escreve:** `Thumb = {{$json.output}}`

  * Ou seja, salva **o JSON final de padrões visuais** retornado pelo *AI Agent2* **na linha 2**, coluna **Thumb** da aba **IDentificação de padrões**.

---

## 🧪 O que é verificado/limitado (sem invenção)

* **Teto de processados:** 50 itens (dois nós `code` com `slice(0, 50)` garantem isso).
* **Condição de criação de descrição:** só gera **DescriçãoThumb** se **estiver vazia**.
* **Onde vai o resultado do agente de padrões:** **apenas** na **linha 2** da aba **IDentificação de padrões**, coluna **Thumb**.
* **Fonte da URL de imagem:** vem do campo **Thumb** original, mapeado para a **chave vazia** `""` (truque proposital), lida por `Analyze image` em `{{$json[""]}}`.

---

## 🗂️ Planilhas envolvidas

* **Dados ordenados (gid=304295346)**

  * Fonte dos 50 primeiros registros
  * Recebe **DescriçãoThumb** (e “Transcrição” como `"="`)
* **IDentificação de padrões (gid=1109606750)**

  * Recebe **o JSON de padrões visuais** (coluna **Thumb**, **linha 2**)

---

## ✅ Resultado

* As **50 thumbnails** (no máximo) são checadas.
* Se a descrição estiver vazia, o fluxo **gera e grava** `DescriçãoThumb`.
* Em seguida, o agente consolida **padrões visuais top vs bottom** com base nas **descrições existentes**, e o **JSON final** é **salvo** na aba **IDentificação de padrões**, **linha 2 → coluna “Thumb”**.

—

*Observação:* este documento mantém o mesmo padrão de layout/explicação da parte anterior e descreve **somente o que o código faz de fato**, sem extrapolações. 


