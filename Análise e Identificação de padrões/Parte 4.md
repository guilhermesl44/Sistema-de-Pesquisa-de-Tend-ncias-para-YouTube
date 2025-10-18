# 🚀 Parte 4 — Normalização de Oportunidades (n-gramas → lacunas temáticas)

## 📌 Objetivo

Consolidar **padrões raros-fortes de n-gramas** extraídos dos **títulos da própria base** em **lacunas temáticas acionáveis**, com contexto, métricas e **ranking por score estimado**.
O resultado final é gravado em **IDentificação de padrões** (linha **2**), coluna **Lacunas**.

---

## 🔄 Fluxo de Execução

```
Google Sheets Trigger (a cada 1 min)
→ Aggregate6 (pass-through)
→ Get row(s) in sheet6 (Dados ordenados)
→ Edit Fields8 (row_number, Titulo, outlierScore)
→ Code in JavaScript7 (extrai n-gramas raros-fortes e benchmarks da base)
→ Aggregate7 (aggregateAllItemData)
→ AI Agent6 (normaliza em lacunas temáticas)  [modelo: gpt-4o-mini]
→ Update row in sheet5 (IDentificação de padrões!Lacunas = output; row_number = 2)
```

---

## 📦 Nós do Workflow (o que de fato acontece)

### 1) **Google Sheets Trigger**

**Tipo:** `googleSheetsTrigger` • **Frequência:** `everyMinute`
**Documento:** *Teste Dev IA Pleno* (`1XlZT...RV1WkQ`)
**Aba:** **Dados ordenados** (`gid=304295346`)
Dispara a leitura da base e envia o lote adiante.

---

### 2) **Aggregate6**

**Tipo:** `aggregate`
Função: apenas encaminha os itens do gatilho para leitura.

---

### 3) **Get row(s) in sheet6** (Read)

**Tipo:** `googleSheets`
Lê as linhas de **Dados ordenados** (títulos + métricas).

---

### 4) **Edit Fields8** (normalização de campos)

**Tipo:** `set`
Cria/garante:

* `row_number` = `{{$json.row_number}}` (number)
* `Titulo` = `{{$json["Titulo "]}}` (string)
* `outlierScore` = `{{$json.outlierScore}}` (string)

> **Obs.:** `outlierScore` vem como *string* aqui, mas o próximo nó converte para **Number**.

---

### 5) **Code in JavaScript7** — *Extração de n-gramas raros-fortes*

**Tipo:** `code` (JS)
**Entrada:** lote de itens com `Titulo` (ou “Titulo ”) e `outlierScore`.
**Saída:** objeto JSON com:

* `resumo`: `{ totalVideosBase, topConsiderados, p50, p75, p90, criterios {...} }`
* `padroesRarosFortes`: lista de n-gramas raros e fortes com métricas

**O que o script faz (fiel ao código):**

* **Normaliza** títulos (lowercase, remove acentos, corta *sufixos* de canal/autor após separadores: `" | "`, `" - "`, `"—"`, `"–"`, etc.).
* **Tokeniza** removendo *stopwords* PT/EN e ruídos.
* **Âncoras de domínio**: mantém apenas n-gramas que contenham termos do nicho (ex.: `leg, weak, stairs, vitamin, perna, fortalecer...`).
* **Ruído/cauda**: filtra tokens de *noise* frequentes (ex.: `dr, motivation, channel, best ...`).
* **Gera n-gramas** de tamanho `2` e `3` (bigramas/trigramas).
* **Benchmarks próprios**: calcula `p50`, `p75`, `p90` do `outlierScore` da **própria base**.
* **Top da base**: define `topK = min(round(total * 0.10), 50)` e marca presença nos TOPs.
* **Raridade**: mantém apenas `RARE_MIN ≤ ocorrências ≤ RARE_MAX` (2 a 6, no código).
* **Força**: exige **≥ p75** de `outlierScoreMedio` (prioriza **p90+**).
* **Presença em TOPs**: quando `REQUIRE_TOP_HIT = true`, exige aparecer **≥1x em TOP**.
* **Ranking**: ordena por banda (`p90+` > `p75+`), depois `outlierScoreMedio`, depois *mais raro* (menor `count`) e, por fim, presença nos TOPs.
* **Score estimado** (60–95): combina raridade, força (banda) e *top boost*.
* **Retorno**: até **20** padrões (`RETURN_LIMIT = 20`), **sempre com exemplos reais** (títulos).

> **Importante:** aqui **não há corte fixo de 50 linhas de entrada**. O script processa *toda a base lida* e só limita **`topK`** (para marcar presença em TOP) e o **número de padrões retornados** (20).

---

### 6) **Aggregate7**

**Tipo:** `aggregate` • **Config:** `aggregateAllItemData`
Agrupa a saída do script anterior em um único item.

> **Estrutura esperada no próximo nó:** o **AI Agent6** lê `{{$json.data}}`. Com `aggregateAllItemData`, o campo resultante costuma ser `data` contendo os payloads anteriores.
> **Se seu ambiente retornar o objeto direto (sem `data`)**, ajuste o parâmetro `text` do **AI Agent6** para `={{ $json }}` ou mude o modo de agregação para garantir `data`.

---

### 7) **AI Agent6** — *Normalizador de Oportunidades*

**Tipo:** `@n8n/n8n-nodes-langchain.agent`
**Modelo:** `gpt-4o-mini` (via **OpenAI Chat Model6**)
**Entrada (text):** `={{ $json.data }}` → deve conter `{ resumo, padroesRarosFortes }` do passo anterior.
**Função:** agrupar n-gramas semelhantes, consolidar métricas por **tema**, calcular **concorrência** e **scoreEstimado**, e devolver **até 10 lacunas** ordenadas.

📍 **Prompt (preencher manualmente):**

```
<role> Você é um **Normalizador de Oportunidades**. Seu papel é transformar padrões de n-gramas detectados automaticamente em **lacunas temáticas consolidadas** — agrupadas, contextualizadas e pontuadas — para o agente gerador de ideias. </role>
<input> Você receberá um objeto JSON com duas partes:
{
  "resumo": {
    "totalVideosBase": number,
    "p75": number,
    "p90": number
  },
  "padroesRarosFortes": [
    {
      "ngrama": "string",
      "ocorrencias": number,
      "presentesNosTop": number,
      "outlierScoreMedio": number,
      "exemplos": ["string", "string", "string"]
    }
  ]
}


Esses dados vêm do analisador de benchmark e representam padrões reais observados em vídeos de um mesmo subnicho (ex: weak legs).
Os n-gramas podem conter combinações de termos em inglês ou português, e já foram filtrados por relevância.
</input>

<task>

Sua tarefa é consolidar esses n-gramas em temas acionáveis, extraindo o contexto e produzindo uma lista de lacunas temáticas de alto potencial.

Etapas:

Agrupar n-gramas semelhantes:

Use a semelhança semântica ou tokens-chave para unir expressões relacionadas.

Exemplo:

“vitamin combo”, “legs vitamin” → “vitaminas que fortalecem pernas fracas”

“climb stairs”, “stairs easily” → “subir escadas com facilidade”

Calcular métricas consolidadas:

videosExistentes: soma das ocorrências.

percentualDaBase: (videosExistentes / totalVideosBase) × 100.

outlierScoreMedio: média ponderada das pontuações do grupo.

concorrencia:

< 2% → “baixa”

2–5% → “media”

5% → “alta”

exemplos: até 3 títulos reais mais representativos.

Pontuar e ranquear:

Base = 50

+25 se outlierScoreMedio >= p90

+15 se outlierScoreMedio >= p75

+15 se percentualDaBase < 2

+10 se percentualDaBase < 5

Score final limitado entre 60 e 95.

Produzir 10 lacunas principais:

Ordenadas por scoreEstimado (descendente).

Usar nomes de tema claros e compreensíveis.

Não inventar termos não presentes nos dados.

Traduzir títulos ou termos para português quando fizer sentido.

</task>

<output_format>

{
  "nicho": "weak legs / idosos",
  "data_analise": "YYYY-MM-DD",
  "benchmarks": { "p75": number, "p90": number },
  "baseAnalisada": number,
  "lacunas": [
    {
      "ranking": number,
      "tema": "string",
      "videosExistentes": number,
      "percentualDaBase": number,
      "outlierScoreMedio": number,
      "concorrencia": "baixa|media|alta",
      "exemplos": ["string", "string", "string"],
      "scoreEstimado": number
    }
  ]
}


</output_format>

<regras_criticas>
✅ Use apenas dados fornecidos — não crie informações externas.
✅ Mantenha nomes de tema curtos e práticos.
✅ Sempre retorne 10 lacunas no máximo.
✅ Traduza termos técnicos em linguagem natural (ex: “weak shaky legs” → “pernas fracas e trêmulas”).
✅ Priorize temas que combinam alta performance (outlierScoreMedio) e baixa concorrência.
❌ Não misture subnichos diferentes.
❌ Não invente títulos de vídeo — use apenas exemplos reais.
</regras_criticas>
```

**Como o prompt limita o retorno (de acordo com as regras do seu projeto):**

* Exige **apenas dados fornecidos** (sem inventar).
* **Agrupa** por semelhança/termos-chave e **calcula métricas** (videosExistentes, percentualDaBase, outlierScoreMedio, concorrencia).
* **Pontua** e **ranqueia** (base 50 + bônus por p90/p75 + baixa concorrência, clamp 60–95).
* **Retorna no formato JSON** do *output_format* (máx. 10 lacunas).
* **Traduz termos técnicos** para português quando fizer sentido.

> **Atenção:** Como o `text` está `={{ $json.data }}`, garanta que o **Aggregate7** realmente produza a chave `data` contendo `{resumo, padroesRarosFortes}`. Se não existir, ajuste (`={{ $json }}`) para evitar payload vazio.

---

### 8) **OpenAI Chat Model6**

**Tipo:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
**Modelo:** `gpt-4o-mini` • **Credenciais:** *OpenAI API (Guilherme)*
Fornece o LLM ao **AI Agent6**.

---

### 9) **Update row in sheet5** (Write)

**Tipo:** `googleSheets` (operation: `update`)
**Destino:** **IDentificação de padrões** (`gid=1109606750`)
**Match:** `row_number = 2`
**Mapeamento:**

* `Lacunas` = `={{ $json.output }}`  ← **JSON final das lacunas**
* `row_number` = `2`

---

## 🧪 O que é verificado / limitado

* **TOP vs base:** o script calcula `topK = min(round(total*10%), 50)` **apenas para marcar presença nos TOPs**, não para limitar a base lida.
* **Raridade:** n-gramas com **2–6** ocorrências.
* **Força:** **≥ p75** (prioriza **p90+** pelo `outlierScoreMedio`).
* **Âncora de domínio + sem ruído:** só entra n-grama com **termos do nicho** e **sem tokens de ruído**.
* **Saída do script:** **até 20** padrões; cada padrão traz **exemplos reais** (máx. 3 títulos).
* **Saída final (AI):** **até 10 lacunas** no JSON padronizado.
* **Persistência:** grava em **IDentificação de padrões → linha 2 → coluna Lacunas**.

---

## 🗂️ Planilhas envolvidas

* **Dados ordenados** (`gid=304295346`) — **origem** dos títulos/outlierScore.
* **IDentificação de padrões** (`gid=1109606750`) — **destino** do JSON final de **Lacunas** (linha **2**).

---

## ⚠️ Observações importantes (do jeito que está no código)

1. **`outlierScore` como string em `Edit Fields8`**: o JS converte para número; está ok, mas vale padronizar a coluna na planilha para número (evita “NaN” silencioso).
2. **`Aggregate7 → AI Agent6 (text = {{$json.data}})`**: confirme que a saída do *aggregate* expõe `data`. Se não, ajuste o *binding* (ou o modo de agregação) para não quebrar o prompt.
3. **Não há corte fixo a 50 entradas** nesta parte (diferente das Partes 2/3). O limite de “50” aparece **apenas** no cálculo de `topK`. Se quiser uniformizar, adicione um `Code` antes do JS para `slice(0, 50)`.

---

## ✅ Resultado

A partir da própria base, a automação **extrai n-gramas raros-fortes**, calcula **benchmarks internos** (p75/p90) e **marca presença em TOPs**; em seguida, o agente **normaliza** em **lacunas temáticas** (máx. 10) com **score estimado** e **exemplos reais**, gravando o JSON final em **IDentificação de padrões → linha 2 → Lacunas**.
