# ⚙️ Parte 3 — Análise de Estrutura e Gatilhos dos Roteiros

## 📌 Objetivo

Ler **exatamente os 50 primeiros registros** da aba **“Dados ordenados”**, **verificar se a coluna `Transcrição` está vazia** e, quando vazia, **gerar uma análise estrutural do roteiro via IA** (com base na transcrição do vídeo). Em seguida, **agregar as análises** e produzir um **sumário de padrões narrativos** (via outro agente), gravando o resultado final em **“IDentificação de padrões” (linha 2, colunas `Roteiro` e `Thumb`= “=”)**. 

---

## 🔄 Fluxo de Execução (alto nível)

```
Google Sheets Trigger (1 min)
→ Aggregate6
→ Get row(s) in sheet3 (Dados ordenados)
→ Edit Fields5 (Link, row_number)
→ Code in JavaScript4 (limita a 50)
→ Loop Over Items
   ├─ If1 (Transcrição vazia?)
   │   └─ Get video transcript (Supadata: getTranscript)
   │       → AI Agent3 (gera análise do roteiro)
   │       → Wait1 (60s)
   │       → Update row in sheet3 (grava análise em Transcrição)
   └─ (saída do loop) → Aggregate4 (agrega itens)
→ Get row(s) in sheet4 (Dados ordenados)
→ Edit Fields6 (Transcrição, row_number)
→ Code in JavaScript5 (limita a 50)
→ Aggregate5 (agrega apenas Transcrição)
→ AI Agent4 (sumário de padrões narrativos)
→ Update row in sheet4 (IDentificação de padrões: Roteiro=row 2, Thumb="=")
```



---

## 📦 Nós do Workflow (passo a passo fiel ao código)

### 1) **Google Sheets Trigger**

* **Tipo:** `n8n-nodes-base.googleSheetsTrigger`
* **Frequência:** `everyMinute`
* **Documento:** `Teste Dev IA Pleno` (ID `1XlZTABwHA456bYCFRiS8BFLxynypmo65pXeBeRV1WkQ`)
* **Aba:** `Dados ordenados` (gid `304295346`)
  Dispara o fluxo e envia as linhas atuais para **Aggregate6**. 

---

### 2) **Aggregate6**

* **Tipo:** `aggregate` (pass-through)
  Encaminha o lote lido para **Get row(s) in sheet3**. 

---

### 3) **Get row(s) in sheet3** (Read)

* **Tipo:** `n8n-nodes-base.googleSheets` (operação “get rows”)
* **Doc/Aba:** mesmo do gatilho (`Dados ordenados`)
  Entrega registros para mapeamento. 

---

### 4) **Edit Fields5** (normalização)

* **Tipo:** `n8n-nodes-base.set`
* **Campos criados:**

  * `Link` = `{{$json.Link}}`
  * `row_number` = `{{$json.row_number}}`
    Padroniza dados mínimos para o loop e para atualização posterior por `row_number`. 

---

### 5) **Code in JavaScript4** (limite)

* **Tipo:** `n8n-nodes-base.code`
* **Função:** limita a **50** itens por execução:

```js
const limit = 50;
return $input.all().slice(0, limit);
```

Garante que **somente os 50 primeiros** sigam no fluxo. 

---

### 6) **Loop Over Items** (batches)

* **Tipo:** `n8n-nodes-base.splitInBatches`
  Cria iterações item a item.
  **Saídas:**
* Saída 1 → **Aggregate4** (para consolidar depois)
* Saída 2 → **If1** (checar `Transcrição` vazia) 

---

### 7) **If1** (condição de processamento)

* **Tipo:** `n8n-nodes-base.if`
* **Condição:** `Transcrição` **vazia** na linha atual?

  * **Se vazia:** segue para **Get video transcript**
  * **Se preenchida:** volta ao **Loop Over Items** (não reprocessa)
    Essa verificação evita custo com IA em itens já tratados. 

---

### 8) **Get video transcript** (Supadata)

* **Tipo:** `n8n-nodes-supadata.supadata`
* **Operação:** `getTranscript`
* **Parâmetros:**

  * `videoId`: `={{ $('Loop Over Items').item.json.Link }}`
  * `text: true` (retorna texto de transcript)
    Baixa a **transcrição completa** do vídeo do item atual. 

---

### 9) **AI Agent3** (análise estrutural por vídeo)

* **Tipo:** `@n8n/n8n-nodes-langchain.agent`
* **Modelo:** `gpt-4.1-mini` (via **OpenAI Chat Model3**)
* **Entrada efetiva:** **fluxo do nó anterior** (transcrição completa)
* **Observação importante:** no node, o campo `text` está configurado como `={{ $('Loop Over Items').item.json.Link }}`. Apesar disso, **a transcrição chega via conexão do nó anterior** e é o conteúdo analisado. Se você quiser “blindar” isso, altere o `text` para ler explicitamente a transcrição recebida. 

📍 **Prompt (preencher manualmente):**

```
# 🎯 Função
Você é um **Analista de Estrutura Narrativa de Vídeos do YouTube**.  
Sua tarefa é **ler a transcrição completa de um vídeo** e **mapear a estrutura narrativa e lógica do roteiro** — ou seja, como o conteúdo é organizado, dividido e comunicado.

⚠️ **Importante:**  
- NÃO gere título.  
- NÃO faça suposições.  
- Baseie-se **apenas** no conteúdo da transcrição.  
- O foco é **entender o roteiro** e como o vídeo se desenrola, não o tema em si.

---

## 📥 Entrada
Transcrição textual completa do vídeo (sem cortes).

---

## 🧩 Etapas de Análise

### 1️⃣ Identificação das Seções
- Divida o vídeo em blocos/seções coerentes.
- Nomeie cada bloco com um **título descritivo**.
- Informe o **tempo aproximado de início** (se disponível).
- Descreva:
  - **Tipo de conteúdo:** educacional, técnico, motivacional, promocional, storytelling, etc.
  - **Função:** introdução, explicação, demonstração, exemplo, CTA (chamada para ação), conclusão, etc.
  - **Descrição:** o que acontece e qual o propósito comunicativo da seção.

### 2️⃣ Extração de Conteúdo
- Liste os **tópicos principais** (conceitos, temas e ideias-chave).
- Liste os **conceitos técnicos** ou termos importantes mencionados.

### 3️⃣ Análise de Fluxo
- Descreva **como o vídeo progride**:
  - As transições são suaves ou abruptas?
  - Existe um padrão lógico (problema → solução → reforço → CTA)?
  - O apresentador reforça ideias, dá exemplos, faz pausas estratégicas?

### 4️⃣ Síntese Estrutural
- Resuma a **mensagem central** (em uma frase).
- Descreva:
  - **estratégiaNarrativa:** estrutura geral (ex: tutorial linear, storytelling, problema-solução, jornada, etc.)
  - **estiloDeEntrega:** (ex: técnico, didático, emocional, informal)
  - **estruturaResumo:** versão compacta do fluxo em uma linha (ex: Introdução > Explicação > Aplicação > Conclusão)

---

## 🧾 Saída esperada (em JSON estrito)

```json
{
  "mensagemCentral": "string (resumo em 1 frase)",
  "estrutura": [
    {
      "secao": "string (nome da seção)",
      "inicioAprox": "string (ex: '00:00')",
      "tipo": "string (ex: educacional, técnico, motivacional)",
      "funcao": "string (ex: introdução, explicação, CTA)",
      "descricao": "string (resumo do que ocorre na seção)"
    }
  ],
  "tópicosPrincipais": ["string"],
  "conceitosTecnicos": ["string"],
  "analiseFluxo": "string (descrição do progresso do vídeo e transições)",
  "estrategiaNarrativa": "string (modelo estrutural adotado)",
  "estiloDeEntrega": "string (tom geral da apresentação)",
  "estruturaResumo": "string (resumo linear da estrutura)"
}
⚙️ Regras e Validações
Proibido gerar “titulo” → se o modelo criar esse campo, ignore ou remova.

Evite redundância: se duas seções tiverem mesmo propósito, agrupe.

Se a transcrição tiver timestamps (00:00, 02:30, etc), use-os como delimitadores de seções.

Se não houver timestamps, divida por lógica de conteúdo (mudança de tópico ou tom).

Use frases curtas e linguagem técnica, sem floreios.

Sempre retorne um JSON válido e bem formatado.

✅ Exemplo de Saída
json
Copiar código
{
  "mensagemCentral": "Manter pernas fortes e um estilo de vida saudável com nutrição e exercícios previne o declínio físico do envelhecimento.",
  "estrutura": [
    {
      "secao": "Introdução e Contextualização",
      "inicioAprox": "00:00",
      "tipo": "motivacional/educacional",
      "funcao": "Apresentar o problema do enfraquecimento das pernas e gerar engajamento.",
      "descricao": "O apresentador destaca o impacto da perda muscular e convida o público à ação."
    },
    {
      "secao": "Explicação Científica",
      "inicioAprox": "02:00",
      "tipo": "técnico",
      "funcao": "Apresentar fundamentos biológicos e fisiológicos.",
      "descricao": "Fala sobre sarcopenia e perda de densidade óssea, apoiado em estudos."
    },
    {
      "secao": "Aplicação Prática - Exercícios e Nutrição",
      "inicioAprox": "10:00",
      "tipo": "instrutivo",
      "funcao": "Ensinar como aplicar os conceitos em práticas cotidianas.",
      "descricao": "Explica exercícios, alimentação e hábitos diários para manter força muscular."
    },
    {
      "secao": "Conclusão e CTA",
      "inicioAprox": "25:00",
      "tipo": "motivacional",
      "funcao": "Encerrar com reforço e convite à ação.",
      "descricao": "Reforça a importância da consistência e convida à inscrição e comentários."
    }
  ],
  "tópicosPrincipais": ["sarcopenia", "força muscular", "nutrição", "exercícios", "longevidade"],
  "conceitosTecnicos": ["colágeno", "isometria", "vitamina D", "cálcio"],
  "analiseFluxo": "O vídeo segue de um gancho motivacional para uma explicação técnica e encerra com aplicações práticas e CTA. As transições são suaves e bem conectadas.",
  "estrategiaNarrativa": "Problema → Explicação → Solução prática → Reforço/CTA",
  "estiloDeEntrega": "Didático e técnico, com tom inspirador.",
  "estruturaResumo": "Introdução > Explicação > Aplicação > Conclusão"
}

```

> **Saída esperada no próprio prompt:** JSON com `mensagemCentral`, `estrutura` (lista de seções), `tópicosPrincipais`, `conceitosTecnicos`, `analiseFluxo`, `estrategiaNarrativa`, `estiloDeEntrega`, `estruturaResumo`. 

---

### 10) **Wait1** (cooldown)

* **Tipo:** `n8n-nodes-base.wait`
* **Tempo:** **60 segundos**
  Ajuda a controlar cadência entre a IA e o write no Sheets. *(No fluxo, está 60s — não 20s.)* 

---

### 11) **Update row in sheet3** (grava análise por vídeo)

* **Tipo:** `n8n-nodes-base.googleSheets` (operation: `update`)
* **Alvo:** **“Dados ordenados”**
* **Match:** `row_number` (da linha iterada)
* **Mapeamento gravado:**

  * `Transcrição` = `={{ $json.output }}`  ⟵ **grava a ANÁLISE do AI Agent3**, não a transcrição “crua”
  * `row_number` = `={{ $('Loop Over Items').item.json.row_number }}`

> **Nota:** O design atual **substitui** `Transcrição` pelo **JSON de análise** do AI Agent3. Se você quiser manter a transcrição original, crie outra coluna (ex.: `TranscriçãoOriginal`) para armazenar o texto puro, e deixe `Transcrição` para o resumo/estrutura. 

---

### 12) **Aggregate4** → **Get row(s) in sheet4** → **Edit Fields6** → **Code in JavaScript5**

* **Aggregate4:** “varre” o que saiu do loop.
* **Get row(s) in sheet4:** relê **“Dados ordenados”**.
* **Edit Fields6:** reduz cada item para:

  * `Transcrição` = `{{$json["Transcrição"]}}`
  * `row_number` = `{{$json.row_number}}`
* **Code in JavaScript5:** **limita novamente a 50** antes de agregar:

```js
const limit = 50;
return $input.all().slice(0, limit);
```

Esses passos preparam um **lote de até 50 análises** (já salvas em `Transcrição`) para o agente comparativo. 

---

### 13) **Aggregate5** (coleção de análises)

* **Tipo:** `n8n-nodes-base.aggregate`
* **Config:** agrega **apenas o campo `Transcrição`** de cada item
  Saída: **lista simples** com as análises (JSONs) por vídeo. 

---

### 14) **AI Agent4** (síntese de padrões narrativos)

* **Tipo:** `@n8n/n8n-nodes-langchain.agent` (modelo: `gpt-4.1-mini` via **OpenAI Chat Model4**)
* **Entrada real:** a **lista agregada** de `Transcrição` (ou seja, **as análises estruturais produzidas pelo AI Agent3**).
* **Importante (inconsistência atual do prompt vs entrada):** O prompt do AI Agent4 instrui a comparar **Top** vs **Bottom** e prevê um **formato de input** com duas listas (`top_roteiros` e `bottom_roteiros`). **O fluxo, como está, NÃO constrói nem fornece esse split**; ele passa **uma lista única** de `Transcrição` agregada. Se você quiser realmente um comparativo Top/Bottom, será preciso **criar ramos** ou **pré-agregar dois grupos** antes de chamar o AI Agent4. Documentação aqui reflete o que o **código faz de fato**. 

📍 **Prompt (preencher manualmente):**

```
<persona>
Você é um Analista de Padrões Narrativos especializado em roteiros de vídeos para YouTube, com foco em identificar os elementos estruturais e retóricos que geram alto ou baixo desempenho.  
Sua função é comparar as estruturas de vídeos bem-sucedidos (top vídeos) e os de baixo desempenho (bottom vídeos), detectando padrões recorrentes e anomalias de forma objetiva, baseada em dados observáveis.  
Você atua como um **“cientista de estrutura narrativa”**, não como roteirista criativo.
</persona>

<objetivo>
Analisar as estruturas de roteiro de vídeos fornecidas e identificar:
- Os **padrões estruturais, comunicativos e retóricos** que se repetem entre os vídeos de melhor desempenho;
- Os **anti-padrões** (erros estruturais, ausências ou excessos) observados nos vídeos de baixo desempenho;
- Gerar diretrizes práticas e replicáveis para a criação de roteiros mais eficazes.

</objetivo>

<regras_e_diretrizes>
✅ **Baseado em dados observáveis** — todos os padrões devem vir de repetições reais nas estruturas analisadas (ex: “80% dos vídeos de sucesso têm introduções com gancho emocional e CTA em até 30s”).  
✅ **Comparativo e contrastivo** — sempre destacar diferenças entre vídeos “Top” e “Bottom”.  
✅ **Classificação por categoria estrutural** — organize os padrões por tipo (Abertura, Desenvolvimento, Call to Action, Transições, Tom e Ritmo, etc.).  
✅ **Linguagem objetiva e acionável** — não diga “roteiro envolvente”, diga **por que**: (ex: “uso de perguntas no início aumenta a retenção”).  
✅ **Nada de achismos** — não infira intenção do criador, apenas o padrão estrutural observado.  
✅ **Formato padronizado e em JSON** — deve seguir a estrutura abaixo.

</regras_e_diretrizes>

<formato_input>
O agente receberá um JSON com duas listas:

{
  "top_roteiros": [
    { "mensagemCentral": "...", "estrutura": [...], "estrategiaNarrativa": "...", "estiloDeEntrega": "...", "estruturaResumo": "..." },
    ...
  ],
  "bottom_roteiros": [
    { "mensagemCentral": "...", "estrutura": [...], "estrategiaNarrativa": "...", "estiloDeEntrega": "...", "estruturaResumo": "..." },
    ...
  ]
}
</formato_input>

<formato_saida>
A resposta deve ser um JSON com a seguinte estrutura:

{
  "padrões_narrativos": {
    "abertura": {
      "top": ["gancho emocional em até 0:30s", "promessa clara de transformação", "uso de pergunta provocativa"],
      "bottom": ["introdução longa sem objetivo", "sem gancho ou conexão emocional"],
      "diretriz": "Começar com gancho claro e promessa de benefício dentro dos primeiros 30 segundos.",
      "por_que_funciona": "Os vídeos de sucesso retêm atenção inicial ao criar curiosidade imediata e alinhar expectativa."
    },
    "desenvolvimento": {
      "top": ["estrutura linear problema → explicação → solução", "transições suaves entre tópicos", "uso de exemplos práticos"],
      "bottom": ["saltos temáticos", "ausência de estrutura clara", "excesso de informação sem contexto"],
      "diretriz": "Manter progressão lógica com transições claras e exemplos concretos.",
      "por_que_funciona": "Mantém o espectador engajado e reduz frustração cognitiva."
    },
    "tom_e_ritmo": {
      "top": ["equilíbrio entre técnico e emocional", "uso de pausas e variação de energia"],
      "bottom": ["monotonia de fala", "linguagem excessivamente técnica"],
      "diretriz": "Equilibrar conteúdo técnico com emoção e ritmo de fala variado.",
      "por_que_funciona": "Variação de tom aumenta retenção e aproxima o público."
    },
    "transicoes": {
      "top": ["uso de frases conectoras (ex: 'agora que você entendeu…')", "retomadas curtas de contexto"],
      "bottom": ["mudanças bruscas de tema", "ausência de conectores lógicos"],
      "diretriz": "Usar transições verbais que conectem seções e reforcem o fluxo lógico.",
      "por_que_funciona": "Garante fluidez narrativa e compreensão contínua."
    },
    "call_to_action": {
      "top": ["CTA natural ao final com reforço positivo", "convite à interação contextualizado"],
      "bottom": ["CTA genérico ou ausente", "pedido de inscrição desconectado do tema"],
      "diretriz": "Conectar o CTA ao conteúdo do vídeo de forma contextual e positiva.",
      "por_que_funciona": "Transforma engajamento em ação sem quebrar o fluxo emocional."
    }
  },
  "anti_padrões": [
    "Evitar introduções longas e sem propósito",
    "Evitar explicações técnicas sem contextualização prática",
    "Evitar mudanças bruscas de assunto sem transição clara",
    "Evitar CTAs genéricos ou forçados"
  ],
  "insight_geral": "Os roteiros de sucesso seguem uma sequência clara de gancho inicial, desenvolvimento lógico e conclusão com CTA contextualizado. Mantêm equilíbrio entre técnica e emoção, com ritmo dinâmico e transições suaves. Os roteiros de baixo desempenho são lineares demais, frios e desorganizados, com introduções lentas e CTAs desconectados."
}
</formato_saida>
```

**Saída:** JSON consolidado com `padrões_narrativos` (por categoria), `anti_padrões`, `insight_geral`. *(No estado atual, será inferido **apenas a partir da lista única** de análises.)* 

---

### 15) **Update row in sheet4** (grava o consolidado final)

* **Tipo:** `n8n-nodes-base.googleSheets` (operation: `update`)
* **Alvo:** **“IDentificação de padrões”** (gid `1109606750`)
* **Match:** `row_number = 2` (fixo)
* **Mapeamento gravado:**

  * `Roteiro` = `={{ $json.output }}` (JSON do AI Agent4)
  * `Thumb` = `"="` (placeholder)
  * `row_number` = `2`
    O resultado final da **Parte 3** fica centralizado na **linha 2** da aba de identificação de padrões. 

---

## 🧪 O que é verificado / limitado

* **Limite de itens:** 50 (duas vezes: antes do loop e antes da agregação final). 
* **Condição de processamento:** só gera análise se `Transcrição` **estiver vazia** na origem. 
* **Cooldown:** `Wait1` de **60s** entre IA e escrita. 
* **Persistência:** `Transcrição` recebe **o JSON de análise do AI Agent3** (não o texto bruto). 
* **Consolidado final:** gravado em **IDentificação de padrões** (`row_number = 2`) na coluna **Roteiro**; **Thumb** recebe `"="`. 

---

## 🗂️ Planilhas envolvidas

* **Dados ordenados** (gid `304295346`) — leitura base; também recebe **Transcrição** com a **análise JSON** por vídeo. 
* **IDentificação de padrões** (gid `1109606750`) — recebe o **sumário consolidado** em `Roteiro` (linha **2**). 

---

## ⚠️ Observações críticas (pra doc e para o revisor do código)

1. **`text` do AI Agent3 aponta para `Link`**, mas a entrada real (via conexão) é a **transcrição**. Sugestão: ajustar `text` para ler explicitamente a transcrição do nó anterior e evitar ambiguidade. 
2. **Campo `Transcrição` é sobrescrito** com a **análise JSON**. Se você precisa guardar a transcrição “crua”, crie uma coluna dedicada (ex.: `TranscriçãoOriginal`). 
3. **AI Agent4 espera Top vs Bottom**, mas **não há split** sendo criado no fluxo. Se o objetivo é comparar, inclua um passo de **classificação** (ex.: por `outlierScore` ou `Classificação`) e gere o payload nos moldes do `<formato_input>` do prompt (duas listas). **Nesta doc, descrevi o que o código realmente faz** (consolidação a partir de lista única). 

---

## ✅ Resultado

Até **50 vídeos** são processados por execução; quando `Transcrição` está vazia, a **análise de roteiro** é gerada e gravada na própria coluna `Transcrição`. Depois, essas análises são **agregadas** e sintetizadas em **padrões narrativos** pelo **AI Agent4**, e o **JSON final** é escrito em **IDentificação de padrões** → **linha 2 / coluna `Roteiro`** (com `Thumb` definido como `"="`). 
