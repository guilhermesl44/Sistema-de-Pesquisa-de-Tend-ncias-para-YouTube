---

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
→ AI Agent2 (compara TOP vs BOTTOM e extrai padrões)
→ Update row in sheet1 (IDentificação de padrões!Thumb = output do AI Agent2)
```

---

## 📦 Nós do Workflow (o que de fato acontece)

### 1) **Google Sheets Trigger**

**Tipo:** `googleSheetsTrigger`
**Disparo automático (a cada 1 minuto)**

* **Documento:** `Teste Dev IA Pleno` (`1XlZTABwHA456bYCFRiS8BFLxynypmo65pXeBeRV1WkQ`)
* **Aba:** **Dados ordenados** (`gid=304295346`)
* Envia as linhas atuais para o nó Aggregate6.

---

### 2) **Aggregate6**

**Tipo:** `aggregate`
Função: pass-through — encaminha os dados do gatilho para *Get row(s) in sheet1*.

---

### 3) **Get row(s) in sheet1**

**Tipo:** `googleSheets (Read)`
Lê todas as linhas da aba **Dados ordenados** e envia ao próximo nó.

---

### 4) **Edit Fields3**

**Tipo:** `set`
Cria duas propriedades:

* `""` (chave vazia) = `{{$json.Thumb}}`
* `row_number` = `{{$json.row_number}}`

> Essa chave vazia serve para armazenar a URL da imagem usada em `Analyze image`.

---

### 5) **Code in JavaScript**

Limita a 50 itens:

```js
const limit = 50;
return $input.all().slice(0, limit);
```

---

### 6) **Loop Over Items2**

Itera registro por registro sobre os 50 itens.

---

### 7) **If**

Verifica se `DescriçãoThumb` está vazia.

* Vazia → executa **Analyze image**
* Preenchida → retorna ao loop sem reprocessar.

---

### 8) **Analyze image** — `gpt-4o-mini`

Descreve tecnicamente a thumbnail.

* **Entrada:** `={{ $json[""] }}` (URL da imagem).
* **Saída:** JSON com `descricao_detalhada`, `tags_visuais`, `cores_predominantes`, `texto_detectado`, `composicao`, `gancho_visual`, `resumo_visual`.

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
✅ Descreva **tudo que for visualmente identificável**, de forma neutra e analítica: pessoas, expressões, poses, objetos, cores, texto, símbolos, e composições.
✅ **Não interprete intenções** (ex: "parece triste"), apenas descreva o que é visível (ex: "expressão facial neutra, boca levemente curvada para baixo").
✅ Identifique **composição e layout**: enquadramento (close-up, plano médio, corpo inteiro), posicionamento (texto à esquerda, pessoa centralizada, elemento no canto inferior direito).
✅ Liste **cores predominantes** e contraste entre fundo e elementos.
✅ Se houver texto na imagem, transcreva exatamente o que aparece (mesmo se for parcial).
✅ Aponte **elementos visuais de destaque** usados para chamar atenção (setas, círculos, emojis, objetos apontando, brilho, bordas coloridas, etc.).
✅ Gere tags visuais curtas (1 a 3 palavras) para facilitar classificação.
✅ Estruture o output em formato JSON padronizado.
❌ Não invente elementos ou contextos que não estão visíveis.
❌ Não insira interpretações emocionais, previsões ou opiniões.

</regras_e_diretrizes>

<formato_saida>
Retorne a resposta neste formato JSON:

{
  "descricao_detalhada": "Descrição completa da cena, incluindo pessoas, objetos, texto, expressões, layout e composição.",
  "tags_visuais": ["close-up", "texto-grande", "seta-vermelha", "antes-depois"],
  "cores_predominantes": ["vermelho", "branco", "amarelo"],
  "texto_detectado": "WARNING! Weak Legs? Fix It Fast!",
  "composicao": {
    "enquadramento": "plano médio, sujeito à direita",
    "layout": "texto à esquerda, pessoa à direita, fundo desfocado",
    "contraste": "alto contraste (vermelho e branco)",
    "elementos_destaque": ["seta vermelha", "círculo", "emoji de alerta"]
  },
  "gancho_visual": "seta vermelha apontando para a perna de uma pessoa",
  "resumo_visual": "Thumbnail com fundo claro, pessoa à direita e texto em vermelho grande à esquerda, com setas e círculos destacando uma área específica do corpo."
}
</formato_saida>

```

---

### 9) **Wait** (20 s)

Aguarda antes da escrita no Sheets para respeitar limites de API.

---

### 10) **Update row in sheet**

Atualiza a linha atual (`row_number`) com:

* `DescriçãoThumb = {{$json.content}}`
* `Transcrição = "="`

---

### 11) **Aggregate2**

Agrupa os itens do loop para seguirem em bloco.

---

### 12) **Get row(s) in sheet2**

Relê a aba **Dados ordenados** com as descrições já atualizadas.

---

### 13) **Edit Fields4**

Mantém `DescriçãoThumb` e `row_number`.

---

### 14) **Code in JavaScript1**

Limita novamente a 50 itens:

```js
const limit = 50;
return $input.all().slice(0, limit);
```

---

### 15) **Aggregate3**

Agrega somente o campo `DescriçãoThumb` para alimentar o agente de padrões.

---

### 16) AI Agent2 — Análise de Padrões Visuais

Analisa as thumbnails de melhor desempenho (TOP) e gera um relatório em JSON com:

padroes_visuais

anti_padroes

insight_geral

📍 **Prompt (preencher manualmente):**

```
<persona>
Você é um Analista de Padrões Visuais especializado em thumbnails de vídeos, com foco em identificar os elementos visuais que geram alto ou baixo engajamento. 
Sua função é comparar descrições de imagens (thumbnails) e extrair padrões de forma objetiva, detectando quais elementos se repetem em thumbnails de alto desempenho e quais aparecem em thumbnails de baixo desempenho. 
Você atua como um "cientista de padrões visuais", não como um designer subjetivo.
</persona>

<objetivo>
Analisar as descrições detalhadas de thumbnails fornecidas, identificar os padrões visuais recorrentes entre as de alto desempenho e contrastá-los com as de baixo desempenho. 
O objetivo é gerar um relatório de padrões replicáveis e anti-padrões a evitar, com justificativas baseadas em observações diretas dos dados.
</objetivo>

<regras_e_diretrizes>
✅ **Baseado em dados reais**: Cada padrão identificado deve se basear em observações concretas nas descrições (ex: "80% das thumbnails vencedoras têm fundo claro com texto vermelho grande").  
✅ **Comparativo**: Sempre destaque o contraste entre o que funciona e o que falha.  
✅ **Estruturado**: Classifique os padrões por categoria visual (ex: Cores, Composição, Texto, Elementos de Destaque, Enquadramento, Emoções Visuais).  
✅ **Linguagem objetiva e acionável** — não diga “fica bonito” ou “atraente”, diga **por que** atrai: (ex: “contraste alto vermelho/branco facilita leitura rápida no feed”).  
✅ **Nada de achismos**: não infira intenção do criador, apenas descreva o impacto provável com base nas evidências.  
✅ **Aplique método comparativo**:  
- Para cada categoria, liste os padrões dos “Top Thumbnails” e os dos “Bottom Thumbnails”.  
- Em seguida, extraia uma **diretriz replicável** e uma **regra de omissão (o que evitar)**.  
✅ **Saída sempre em JSON** com estrutura padronizada.

</regras_e_diretrizes>

<formato_input>
Você receberá um JSON com duas listas de thumbnails:

{
  "top_thumbnails": [
    { "descricao_detalhada": "...", "tags_visuais": [...], "cores_predominantes": [...], "texto_detectado": "...", "gancho_visual": "...", "resumo_visual": "..." },
    ...
  ],
  "bottom_thumbnails": [
    { "descricao_detalhada": "...", "tags_visuais": [...], "cores_predominantes": [...], "texto_detectado": "...", "gancho_visual": "...", "resumo_visual": "..." },
    ...
  ]
}
</formato_input>

<formato_saida>
Responda em JSON seguindo exatamente esta estrutura:

{
  "padrões_visuais": {
    "cores": {
      "top": ["fundo claro com texto vermelho", "contraste alto"],
      "bottom": ["fundo escuro e sem contraste"],
      "diretriz": "Usar fundo claro e cores quentes de alto contraste aumenta a legibilidade no feed.",
      "por_que_funciona": "As thumbnails de sucesso apresentam contraste visual forte e cores vibrantes que destacam texto e rosto."
    },
    "composicao": {
      "top": ["pessoa centralizada", "texto grande à esquerda", "close-up"],
      "bottom": ["texto pequeno", "vários elementos dispersos"],
      "diretriz": "Usar composição simples com foco em 1 elemento principal.",
      "por_que_funciona": "Facilita a leitura e compreensão instantânea mesmo em miniaturas pequenas."
    },
    "texto": {
      "top": ["palavras curtas", "fontes grandes e legíveis", "uso de caps moderado"],
      "bottom": ["frases longas", "muitos símbolos ou fontes finas"],
      "diretriz": "Limitar o texto a 2–4 palavras grandes e legíveis.",
      "por_que_funciona": "Aumenta a clareza e impacto em dispositivos móveis."
    },
    "elementos_destaque": {
      "top": ["setas vermelhas", "círculos de foco", "emoji de alerta"],
      "bottom": ["sem foco visual claro", "elementos confusos ou genéricos"],
      "diretriz": "Usar um elemento de foco visual forte (seta, círculo ou contraste direcional).",
      "por_que_funciona": "Guiar o olhar do espectador aumenta a taxa de clique."
    },
    "expressões_ou_sujeitos": {
      "top": ["rosto com emoção visível (surpresa, curiosidade)", "expressão intensa"],
      "bottom": ["sem rosto ou expressão neutra"],
      "diretriz": "Usar expressões humanas claras e emoção perceptível.",
      "por_que_funciona": "Thumbnails com emoção humana facilitam empatia e aumentam cliques."
    }
  },
  "anti_padrões": [
    "Evitar textos longos ou ilegíveis",
    "Evitar fundos escuros com pouco contraste",
    "Evitar excesso de elementos que dificultam foco visual"
  ],
  "insight_geral": "As thumbnails de sucesso compartilham contraste alto, foco central em pessoas com emoção visível, uso moderado de texto grande e cores quentes. Já as de baixo desempenho são confusas, com textos longos e cores frias de baixo contraste."
}
</formato_saida>
```



Entrada: Recebe as descrições detalhadas das thumbnails do TOP (campo DescriçãoThumb) geradas anteriormente pelo nó Analyze Image.
Essas descrições incluem elementos como cores predominantes, composição, texto detectado e expressões.

Processamento: O agente executa uma análise textual para identificar:

padrões recorrentes de cor, composição e texto,

elementos visuais que se destacam nas thumbnails de melhor desempenho,

e possíveis antipadrões (características menos frequentes ou visualmente desfavoráveis).

Saída: Gera um relatório em formato JSON estruturado com os campos padroes_visuais, anti_padroes e insight_geral.

---

### 17) **Update row in sheet1**

Grava o JSON final de padrões na aba **IDentificação de padrões** (`gid = 1109606750`) → `row_number = 2`, coluna **Thumb**.

---

## 🧪 O que é verificado / limitado

* Processa no máximo **50 itens**.
* Só gera `DescriçãoThumb` quando vazia.
* Resultado final vai sempre para **linha 2 / coluna Thumb** em *IDentificação de padrões*.
* Fonte da imagem vem de `Thumb` → chave vazia `""`.

---

## 🗂️ Planilhas envolvidas

* **Dados ordenados** (gid 304295346) → origem dos registros e destino de `DescriçãoThumb`.
* **IDentificação de padrões** (gid 1109606750) → recebe o JSON de padrões (“Thumb”, linha 2).

---

## ✅ Resultado

Até 50 thumbnails são checadas; se faltarem descrições, elas são geradas e gravadas.
Em seguida, o agente compila padrões visuais e o JSON final é gravado em **IDentificação de padrões**, **linha 2 → coluna Thumb**.

