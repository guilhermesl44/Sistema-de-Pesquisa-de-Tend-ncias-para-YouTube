# Parte 2: Cálculo de Benchmarks Estatísticos

![Fluxo Completo](./Imagens/automacao1pt2.jpg)

## 📋 Visão Geral

Esta segunda parte do workflow é responsável por **calcular benchmarks estatísticos dinâmicos** baseados nos dados coletados na Parte 1. Processa todos os vídeos do nicho e gera estatísticas completas (média, mediana, percentis, desvio padrão) para cada métrica.

### Objetivo
Criar uma base estatística do nicho que serve como referência para identificar vídeos outliers e oportunidades de conteúdo na análise com LLM.

---

## 🔄 Fluxo de Execução

```
Trigger (Planilha Atualizada) → Processar Benchmarks → Normalizar → Dividir Arrays → Limpar Aba → Salvar Benchmarks
```

---

## 📦 Nós do Workflow

### 1. **Google Sheets Trigger** (Trigger Automático)
**Tipo:** `n8n-nodes-base.googleSheetsTrigger`

**Função:** Monitora a planilha DADOSBRUTOS e dispara automaticamente quando novos dados são inseridos pela Parte 1.

**Configuração:**
- **Polling:** A cada minuto
- **Documento:** Teste Dev IA Pleno (`1XlZTABwHA456bYCFRiS8BFLxynypmo65pXeBeRV1WkQ`)
- **Aba:** DADOSBRUTOS (`gid=0`)
- **Credenciais:** Google Sheets Trigger OAuth2

**Comportamento:**
- Detecta quando a Parte 1 termina de inserir dados
- Captura **TODAS as linhas** da planilha
- Passa todos os itens para o próximo nó

**Output:**
Array com todos os vídeos da planilha:
```json
[
  {
    "Termo de Pesquisa": "weak legs",
    "Filtro": "views",
    "ID": "abc123",
    "Titulo ": "5 Exercícios Para Pernas Fracas",
    "View": 150000,
    "Likes": 5000,
    "Comentarios ": 320,
    // ... todos os campos da planilha
  },
  // ... mais vídeos
]
```

---

### 2. **Code in JavaScript2** (Motor de Cálculo de Benchmarks)
**Tipo:** `n8n-nodes-base.code`

**Função:** Núcleo do processamento. Calcula benchmarks estatísticos completos para todas as métricas do nicho.

#### **Estrutura do Código**

##### A. Extração e Normalização
```javascript
// Suporta múltiplos formatos de campo
const nicho = (
  data["Termo de Pesquisa"] ||
  data.termo_de_pesquisa ||
  data.nicho || 
  data.Nicho ||
  "sem_nicho"
).toString().toLowerCase().trim();
```

**Campos Extraídos:**
- **Identificação**: ID, título, link, thumb, canal
- **Contexto**: termo de pesquisa, filtro, formato, data
- **Métricas Brutas**: view, likes, comentarios, inscritos, diasPublicado, duracaoSegundos
- **Métricas Calculadas**: engagementRate, likeRate, commentRate, totalEngagementRate, viewsPerDay, outlierScore

##### B. Agrupamento por Nicho
Agrupa todos os vídeos pelo campo "Termo de Pesquisa" usando `Map()`.

##### C. Função de Cálculo Estatístico
```javascript
function calcularEstatisticas(valores) {
  // Ordena valores uma única vez
  const sorted = valores.slice().sort((a, b) => a - b);
  
  // Calcula:
  return {
    media,           // Média aritmética
    mediana,         // P50
    desvioPadrao,    // Desvio padrão
    min,             // Valor mínimo
    max,             // Valor máximo
    p10, p25, p50,   // Percentis baixos
    p75, p90, p95, p99, // Percentis altos
    totalAmostras    // Total de vídeos
  };
}
```

#### **Estatísticas Calculadas por Métrica:**

Para **cada uma** das 12 métricas abaixo:
1. **view** (visualizações)
2. **likes** (curtidas)
3. **comentarios** (comentários)
4. **inscritos** (inscritos do canal)
5. **diasPublicado** (dias desde publicação)
6. **duracaoSegundos** (duração em segundos)
7. **engagementRate** (taxa de engajamento)
8. **likeRate** (taxa de likes por 1000 views)
9. **commentRate** (taxa de comentários por 1000 views)
10. **totalEngagementRate** (taxa total de engajamento)
11. **viewsPerDay** (views por dia)
12. **outlierScore** (score de viralização)

São calculados **13 valores estatísticos**:
- Média
- Mediana
- Desvio Padrão
- Mínimo
- Máximo
- P10 (percentil 10%)
- P25 (percentil 25%)
- P50 (percentil 50% = mediana)
- P75 (percentil 75%)
- P90 (percentil 90%)
- P95 (percentil 95%)
- P99 (percentil 99%)
- Total de Amostras

#### **Output do Código:**

```json
{
  "metadata": {
    "dataProcessamento": "2025-01-18T15:30:00Z",
    "tempoProcessamento": "2.34s",
    "performance": "214 vídeos/s",
    "totalVideosProcessados": 50,
    "nicho": "weak legs"
  },
  
  "benchmarks": {
    "BenchMarks": [
      "media", "mediana", "desvioPadrao", "min", "max",
      "p10", "p25", "p50", "p75", "p90", "p95", "p99", "totalAmostras"
    ],
    
    "view": [150000, 125000, 45000, 5000, 850000, 35000, 75000, 125000, 210000, 450000, 600000, 800000, 50],
    
    "likes": [5000, 4200, 1800, 150, 32000, 1200, 2500, 4200, 7500, 15000, 20000, 30000, 50],
    
    // ... arrays para cada métrica
  },
  
  "nichoProcessado": "weak legs"
}
```

#### **Performance:**
- Otimizado para processar **grande volume** de dados
- Logs detalhados de performance
- Exemplo: ~200-300 vídeos/segundo

**Console Logs:**
```
🚀 Iniciando processamento de 50 linhas...
📊 Agrupado em 1 nichos diferentes
📈 Processando nicho: "weak legs" (50 vídeos)

✅ PROCESSAMENTO CONCLUÍDO!
⏱️  Tempo total: 2.34s
⚡ Performance: 214 vídeos/segundo
📊 Total de vídeos: 50
🎯 Nicho processado: weak legs

📈 BENCHMARKS DO NICHO "WEAK LEGS"
   Total de vídeos: 50
   Mediana de views: 125,000
   Top 10% views: 450,000
   Engagement médio: 3.5%
```

---

### 3. **Edit Fields1** (Normalização)
**Tipo:** `n8n-nodes-base.set`

**Função:** Extrai os arrays de benchmarks para campos individuais, preparando para divisão e inserção no Sheets.

**Campos Criados:**
- `benchmarks.BenchMarks` (cabeçalho)
- `benchmarks.view`
- `benchmarks.likes`
- `benchmarks.comentarios`
- `benchmarks.inscritos`
- `benchmarks.diasPublicado`
- `benchmarks.duracaoSegundos`
- `benchmarks.engagementRate`
- `benchmarks.likeRate`
- `benchmarks.commentRate`
- `benchmarks.totalEngagementRate`
- `benchmarks.viewsPerDay`
- `benchmarks.outlierScore`

---

### 4. **Split Out1** (Divisão de Arrays)
**Tipo:** `n8n-nodes-base.splitOut`

**Função:** Transforma os arrays de benchmarks em linhas individuais para inserção no Google Sheets.

**Configuração:**
- **Campos para dividir:** Todos os 13 campos de benchmarks
- **Incluir outros campos:** Não

**Comportamento:**
Cada índice do array vira uma linha:
```
Índice 0 → Linha "media"
Índice 1 → Linha "mediana"
Índice 2 → Linha "desvioPadrao"
...
Índice 12 → Linha "totalAmostras"
```

**Output:**
13 itens, cada um representando uma estatística:
```json
[
  {
    "benchmarks.BenchMarks": "media",
    "benchmarks.view": 150000,
    "benchmarks.likes": 5000,
    // ... todas as métricas
  },
  {
    "benchmarks.BenchMarks": "mediana",
    "benchmarks.view": 125000,
    "benchmarks.likes": 4200,
    // ...
  }
  // ... 11 linhas adicionais
]
```

---

### 5. **Clear sheet1** (Limpeza)
**Tipo:** `n8n-nodes-base.googleSheets`

**Função:** Limpa a aba BenchMark antes de inserir novos dados.

**Configuração:**
- **Operação:** Clear
- **Documento:** Teste Dev IA Pleno
- **Aba:** BenchMark (`gid=156227565`)
- **Manter cabeçalho:** Sim

---

### 6. **Append or update row in sheet** (Salvamento)
**Tipo:** `n8n-nodes-base.googleSheets`

**Função:** Insere os benchmarks calculados na planilha.

**Configuração:**
- **Operação:** Append or Update
- **Documento:** Teste Dev IA Pleno
- **Aba:** BenchMark (`gid=156227565`)
- **Coluna de matching:** view

**Colunas da Planilha:**
1. BenchMarks (nome da estatística)
2. view
3. likes
4. comentarios
5. inscritos
6. diasPublicado
7. duracaoSegundos
8. engagementRate
9. likeRate
10. commentRate
11. totalEngagementRate
12. viewsPerDay
13. outlierScore

---

## 📊 Estrutura Final no Google Sheets (Aba BenchMark)

| BenchMarks | view | likes | comentarios | inscritos | ... |
|------------|------|-------|-------------|-----------|-----|
| media | 150000 | 5000 | 320 | 580000 | ... |
| mediana | 125000 | 4200 | 280 | 450000 | ... |
| desvioPadrao | 45000 | 1800 | 120 | 280000 | ... |
| min | 5000 | 150 | 10 | 5000 | ... |
| max | 850000 | 32000 | 2100 | 2500000 | ... |
| p10 | 35000 | 1200 | 80 | 50000 | ... |
| p25 | 75000 | 2500 | 150 | 180000 | ... |
| p50 | 125000 | 4200 | 280 | 450000 | ... |
| p75 | 210000 | 7500 | 480 | 850000 | ... |
| p90 | 450000 | 15000 | 950 | 1500000 | ... |
| p95 | 600000 | 20000 | 1400 | 2000000 | ... |
| p99 | 800000 | 30000 | 2000 | 2400000 | ... |
| totalAmostras | 50 | 50 | 50 | 50 | ... |

---

## 🎯 Interpretação dos Benchmarks

### Exemplo Prático: Métrica "view"

```
media: 150000          → Média geral de views
mediana: 125000        → 50% dos vídeos têm mais que isso
p75: 210000           → Top 25% dos vídeos
p90: 450000           → Top 10% (vídeos de sucesso)
p95: 600000           → Top 5% (vídeos virais)
p99: 800000           → Top 1% (mega virais)
```

### Uso na Análise:
- **P90-P99**: Benchmarks de sucesso (outliers positivos)
- **P50-P75**: Performance média/boa
- **P10-P25**: Performance baixa
- **Desvio Padrão**: Indica variabilidade no nicho

---

## ⚙️ Configurações Técnicas

### Trigger Automático
- **Polling:** A cada 1 minuto
- **Gatilho:** Detecta alterações na aba DADOSBRUTOS
- **Delay após Parte 1:** ~1-2 minutos

### Performance
- **Processamento:** 200-300 vídeos/segundo
- **Tempo médio:** 2-5 segundos para 50 vídeos
- **Otimização:** Single-pass sorting, cálculos in-memory

### Credenciais
- **Google Sheets Trigger:** OAuth2 (`6Z6MEbvijNLMdskV`)
- **Google Sheets API:** OAuth2 (`KrBqg4twP2wKcI6z`)

---

## 📈 Benefícios dos Benchmarks

### Para Análise Quantitativa:
1. **Identificar Outliers**: Vídeos acima do P90 são candidatos a análise
2. **Baseline do Nicho**: Entender o que é "normal" vs "excepcional"
3. **Segmentação**: Agrupar vídeos por faixas de performance

### Para Análise com LLM (Parte 3):
1. **Contexto Estatístico**: LLM entende o que é "viral" no nicho
2. **Validação de Padrões**: Correlaciona títulos com faixas de performance
3. **Geração de Score**: Base matemática para scoring de oportunidades

---

## 🔄 Integração com Outras Partes

### Input (Parte 1):
- Dados brutos dos vídeos
- Métricas já calculadas

### Output (Para Parte 3):
- Benchmarks estatísticos completos
- Contexto do nicho
- Base para análise com LLM

---

## 🎯 Resultado Final da Parte 2

**Input:** Dados da aba DADOSBRUTOS  
**Output:** Aba BenchMark com 13 linhas × 13 colunas de estatísticas

**Próxima Etapa:** Os benchmarks serão usados na Parte 3 para contextualizar a análise com LLM e identificar padrões de sucesso.
