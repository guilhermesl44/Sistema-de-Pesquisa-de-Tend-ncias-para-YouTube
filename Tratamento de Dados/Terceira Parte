# Parte 3: Análise de Performance e Níveis de Referência

## 📋 Visão Geral

Esta terceira parte do workflow é responsável por **transformar os benchmarks estatísticos em níveis de performance acionáveis** e **gerar insights do nicho**. Converte dados estatísticos brutos em faixas de referência que serão usadas pelo LLM para classificar vídeos e oportunidades.

### Objetivo
Criar uma tabela de níveis de performance (Top 1%, Top 5%, Top 10%, etc.) que serve como referência visual e contexto para análise qualitativa com LLM na próxima etapa.

---

## 🔄 Fluxo de Execução

```
Trigger (Benchmarks Atualizados) → Reorganizar Dados → Calcular Níveis → Normalizar → Dividir → Limpar Aba → Salvar Análise
```

---

## 📦 Nós do Workflow

### 1. **Google Sheets Trigger1** (Trigger Automático)
**Tipo:** `n8n-nodes-base.googleSheetsTrigger`

**Função:** Monitora a aba BenchMark e dispara automaticamente quando a Parte 2 termina de calcular os benchmarks.

**Configuração:**
- **Polling:** A cada 1 minuto
- **Documento:** Teste Dev IA Pleno (`1XlZTABwHA456bYCFRiS8BFLxynypmo65pXeBeRV1WkQ`)
- **Aba:** BenchMark (`gid=156227565`)
- **Credenciais:** Google Sheets Trigger OAuth2

**Comportamento:**
- Detecta quando a Parte 2 insere os benchmarks
- Captura todas as 13 linhas de estatísticas
- Delay típico: ~1-2 minutos após Parte 2

**Input Esperado:**
13 itens, cada um com:
```json
{
  "BenchMarks": "media", // ou "mediana", "p90", etc.
  "view": 150000,
  "likes": 5000,
  "comentarios": 320,
  "inscritos": 580000,
  "diasPublicado": 45,
  "duracaoSegundos": 624,
  "engagementRate": 3.33,
  "likeRate": 33.33,
  "commentRate": 2.13,
  "totalEngagementRate": 3.55,
  "viewsPerDay": 3333,
  "outlierScore": 25.86
}
```

---

### 2. **Code in JavaScript3** (Reorganização e Análise)
**Tipo:** `n8n-nodes-base.code`

**Função:** Núcleo da Parte 3. Reorganiza os benchmarks e cria níveis de performance e insights do nicho.

#### **Etapa 1: Reorganização dos Dados**

```javascript
// Recebe 13 itens (linhas) e transforma em objeto estruturado
const benchmarks = {
  view: {
    media: 150000,
    mediana: 125000,
    p90: 450000,
    p95: 600000,
    p99: 800000,
    // ... todas as estatísticas
  },
  likes: { ... },
  // ... todas as métricas
}
```

**Métricas Reorganizadas:**
- view
- likes
- comentarios
- inscritos
- diasPublicado
- duracaoSegundos
- engagementRate
- likeRate
- commentRate
- totalEngagementRate
- viewsPerDay
- outlierScore

#### **Etapa 2: Criação dos Níveis de Performance**

Define 6 níveis baseados em percentis:

```javascript
const niveisPerformance = {
  "excepcional - Top 1%": {
    view: benchmarks.view.p99,
    engagementRate: benchmarks.engagementRate.p99,
    // ... todas as métricas no p99
  },
  "excelente - Top 5%": {
    // métricas no p95
  },
  "muitoBom - Top 10%": {
    // métricas no p90
  },
  "bom - Top 25%": {
    // métricas no p75
  },
  "medio - 50% (Mediana)": {
    // métricas no p50
  },
  "abaixoMedia - Bottom 25%": {
    // métricas no p25
  }
}
```

**Tabela de Níveis:**

| Nível | Percentil | Interpretação |
|-------|-----------|---------------|
| Excepcional - Top 1% | P99 | Mega viral, extremo sucesso |
| Excelente - Top 5% | P95 | Muito viral, grande sucesso |
| Muito Bom - Top 10% | P90 | Viral, sucesso consistente |
| Bom - Top 25% | P75 | Acima da média, bom desempenho |
| Médio - 50% (Mediana) | P50 | Performance típica do nicho |
| Abaixo da Média - 25% Inferiores | P25 | Abaixo do esperado |

#### **Etapa 3: Transformação em Arrays**

Converte os níveis em arrays para inserção no Google Sheets:

```javascript
const niveisArray = {
  Nivel: [
    "Excepcional - Top 1%",
    "Excelente - Top 5%",
    "Muito Bom - Top 10%",
    "Bom - Top 25%",
    "Médio - 50% (Mediana)",
    "Abaixo da Média - 25% Inferiores"
  ],
  view: [
    "800000",  // p99
    "600000",  // p95
    "450000",  // p90
    "210000",  // p75
    "125000",  // p50
    "75000"    // p25
  ],
  engagementRate: [
    "5.2",     // p99
    "4.5",     // p95
    "4.0",     // p90
    "3.5",     // p75
    "3.0",     // p50
    "2.5"      // p25
  ],
  // ... arrays para todas as 8 métricas
}
```

#### **Etapa 4: Geração de Insights**

Calcula métricas agregadas do nicho:

```javascript
const insights = {
  viewsTotal: 7500000,           // Total estimado de views
  likesTotal: 250000,            // Total estimado de likes
  comentariosTotal: 16000,       // Total estimado de comentários
  duracaoMediaMinutos: 10.4,     // Duração média em minutos
  idadeMediaDias: 45,            // Idade média dos vídeos
  
  taxaEngagementGeral: 3.33,     // Taxa geral de engagement
  
  rangeEsperado: {
    views: "75,000 - 210,000",
    engagementRate: "2.5% - 3.5%",
    viewsPerDay: "1,500 - 4,500"
  }
}
```

#### **Output do Código:**

```json
{
  "niveisPerformance": {
    "Nivel": ["Excepcional - Top 1%", "Excelente - Top 5%", ...],
    "view": ["800000", "600000", "450000", ...],
    "engagementRate": ["5.2", "4.5", "4.0", ...],
    "likeRate": ["52", "45", "40", ...],
    "commentRate": ["3.5", "3.0", "2.5", ...],
    "totalEngagementRate": ["5.5", "4.8", "4.2", ...],
    "viewsPerDay": ["8000", "6500", "5000", ...],
    "outlierScore": ["120", "95", "75", ...]
  },
  
  "insights": {
    "viewsTotal": 7500000,
    "likesTotal": 250000,
    "comentariosTotal": 16000,
    "duracaoMediaMinutos": 10.4,
    "idadeMediaDias": 45,
    "taxaEngagementGeral": 3.33,
    "rangeEsperado": {
      "views": "75,000 - 210,000",
      "engagementRate": "2.5% - 3.5%",
      "viewsPerDay": "1,500 - 4,500"
    }
  },
  
  "dataAnalise": "2025-01-18T16:00:00Z"
}
```

**Console Logs:**
```
🔍 Recebido 13 itens de estatísticas...
✅ Benchmarks inicializado
✅ Dados preenchidos
✅ Análise concluída!
📊 Total de amostras: 50
👁️  Views totais (estimado): 7,500,000
💬 Taxa de engagement geral: 3.33%

🏆 NÍVEIS DE PERFORMANCE:
   Excepcional (Top 1%): 800,000 views
   Excelente (Top 5%): 600,000 views
   Muito Bom (Top 10%): 450,000 views
   Bom (Top 25%): 210,000 views
```

---

### 3. **Edit Fields2** (Normalização)
**Tipo:** `n8n-nodes-base.set`

**Função:** Extrai os arrays de níveis de performance para campos individuais.

**Campos Criados:**
- `niveisPerformance.Nivel`
- `niveisPerformance.view`
- `niveisPerformance.engagementRate`
- `niveisPerformance.likeRate`
- `niveisPerformance.commentRate`
- `niveisPerformance.totalEngagementRate`
- `niveisPerformance.viewsPerDay`
- `niveisPerformance.outlierScore`

---

### 4. **Split Out2** (Divisão de Arrays)
**Tipo:** `n8n-nodes-base.splitOut`

**Função:** Transforma os arrays de níveis em 6 linhas individuais (uma para cada nível de performance).

**Configuração:**
- **Campos para dividir:** Todos os 8 campos de níveis

**Comportamento:**
```
Array índice 0 → Linha "Excepcional - Top 1%"
Array índice 1 → Linha "Excelente - Top 5%"
Array índice 2 → Linha "Muito Bom - Top 10%"
Array índice 3 → Linha "Bom - Top 25%"
Array índice 4 → Linha "Médio - 50% (Mediana)"
Array índice 5 → Linha "Abaixo da Média - 25% Inferiores"
```

**Output:**
6 itens:
```json
[
  {
    "niveisPerformance.Nivel": "Excepcional - Top 1%",
    "niveisPerformance.view": "800000",
    "niveisPerformance.engagementRate": "5.2",
    "niveisPerformance.likeRate": "52",
    "niveisPerformance.commentRate": "3.5",
    "niveisPerformance.totalEngagementRate": "5.5",
    "niveisPerformance.viewsPerDay": "8000",
    "niveisPerformance.outlierScore": "120"
  },
  // ... 5 linhas adicionais
]
```

---

### 5. **Clear sheet2** (Limpeza)
**Tipo:** `n8n-nodes-base.googleSheets`

**Função:** Limpa a aba ANÁLISE antes de inserir os novos níveis.

**Configuração:**
- **Operação:** Clear
- **Documento:** Teste Dev IA Pleno
- **Aba:** ANÁLISE (`gid=78424668`)
- **Manter cabeçalho:** Sim

---

### 6. **Append or update row in sheet1** (Salvamento)
**Tipo:** `n8n-nodes-base.googleSheets`

**Função:** Insere os 6 níveis de performance na planilha.

**Configuração:**
- **Operação:** Append or Update
- **Documento:** Teste Dev IA Pleno
- **Aba:** ANÁLISE (`gid=78424668`)
- **Coluna de matching:** Nível

**Colunas da Planilha:**
1. Nível
2. view
3. engagementRate
4. likeRate
5. commentRate
6. totalEngagementRate
7. viewsPerDay
8. outlierScore

---

## 📊 Estrutura Final no Google Sheets (Aba ANÁLISE)

| Nível | view | engagementRate | likeRate | commentRate | totalEngagementRate | viewsPerDay | outlierScore |
|-------|------|----------------|----------|-------------|---------------------|-------------|--------------|
| Excepcional - Top 1% | 800000 | 5.2 | 52 | 3.5 | 5.5 | 8000 | 120 |
| Excelente - Top 5% | 600000 | 4.5 | 45 | 3.0 | 4.8 | 6500 | 95 |
| Muito Bom - Top 10% | 450000 | 4.0 | 40 | 2.5 | 4.2 | 5000 | 75 |
| Bom - Top 25% | 210000 | 3.5 | 35 | 2.0 | 3.7 | 3500 | 50 |
| Médio - 50% (Mediana) | 125000 | 3.0 | 30 | 1.8 | 3.2 | 2500 | 35 |
| Abaixo da Média - 25% Inferiores | 75000 | 2.5 | 25 | 1.5 | 2.8 | 1500 | 20 |

---

## 🎯 Interpretação dos Níveis

### Como Usar esta Tabela:

#### Para Identificar Vídeos de Sucesso:
- **Top 1%**: Vídeos com 800k+ views são excecionais
- **Top 5%**: Vídeos com 600k+ views são excelentes
- **Top 10%**: Vídeos com 450k+ views são muito bons

#### Para Definir Metas:
- **Meta Conservadora**: Alcançar P50 (mediana)
- **Meta Realista**: Alcançar P75 (Top 25%)
- **Meta Ambiciosa**: Alcançar P90 (Top 10%)

#### Para Análise de Oportunidades:
Um tema com potencial deve visar **pelo menos P75** em todas as métricas:
- Views: 210k+
- EngagementRate: 3.5%+
- ViewsPerDay: 3500+

---

## 💡 Insights Gerados

### Métricas Agregadas do Nicho:

```json
{
  "viewsTotal": 7500000,
  "likesTotal": 250000,
  "comentariosTotal": 16000,
  "duracaoMediaMinutos": 10.4,
  "idadeMediaDias": 45,
  "taxaEngagementGeral": 3.33,
  "rangeEsperado": {
    "views": "75,000 - 210,000",
    "engagementRate": "2.5% - 3.5%",
    "viewsPerDay": "1,500 - 4,500"
  }
}
```

### Interpretação:

**Duração Ideal**: ~10 minutos  
**Taxa de Engagement Típica**: 3.33%  
**Range Esperado de Views**: 75k - 210k (P25 a P75)

Vídeos que performam **significativamente acima** deste range são outliers e merecem análise de padrões.

---

## ⚙️ Configurações Técnicas

### Trigger Automático
- **Polling:** A cada 1 minuto
- **Gatilho:** Detecta alterações na aba BenchMark
- **Delay após Parte 2:** ~1-2 minutos

### Performance
- **Processamento:** Instantâneo (~100ms)
- **Transformações:** Reorganização + 6 níveis
- **Output:** 6 linhas na planilha

### Credenciais
- **Google Sheets Trigger:** OAuth2 (`6Z6MEbvijNLMdskV`)
- **Google Sheets API:** OAuth2 (`KrBqg4twP2wKcI6z`)

---

## 🔄 Integração com Outras Partes

### Input (Parte 2):
- 13 linhas de benchmarks estatísticos
- Uma linha para cada estatística (média, mediana, percentis)

### Output (Para Parte 4):
- **Aba ANÁLISE**: Níveis de performance visual
- **Insights do nicho**: Contexto agregado
- **Referência para LLM**: Definição clara de "sucesso" no nicho

---

## 📈 Uso na Análise com LLM (Parte 4)

Esta tabela será usada pelo LLM para:

### 1. Classificar Vídeos
```
Se view >= 450k → "Viral (Top 10%)"
Se 210k <= view < 450k → "Acima da média (Top 25%)"
Se view < 125k → "Abaixo da mediana"
```

### 2. Validar Oportunidades
```
Tema sugerido deve ter potencial de:
- Views: >= P75 (210k)
- EngagementRate: >= P75 (3.5%)
- ViewsPerDay: >= P75 (3500)
```

### 3. Contextualizar Análise
```
"Este título gerou 600k views, colocando-o no Top 5% do nicho.
Padrões identificados: [análise qualitativa]"
```

---

## 🎯 Resultado Final da Parte 3

**Input:** 13 linhas de benchmarks (Parte 2)  
**Output:** 
- Aba ANÁLISE com 6 níveis de performance
- Insights agregados do nicho
- Referência visual para classificação

**Próxima Etapa:** A Parte 4 usará estes níveis + dados brutos para análise com LLM e geração das 15 sugestões finais.
