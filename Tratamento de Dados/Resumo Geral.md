# Resumo Workflow 

## 🔄 4 Partes

### **Automação 1: Raspagem de Dados**
Faz raspagem de vídeos do YouTube usando um actor do Apify. Recebe nicho, quantidade e filtro via formulário web, aguarda 10 minutos o processamento e salva ~50 vídeos com métricas calculadas (engagementRate, outlierScore, viewsPerDay, etc.) na planilha DADOSBRUTOS.

**Tempo:** ~12 minutos

---

### **Automação 2: Cálculo de Benchmarks**
Processa todos os vídeos e calcula 13 estatísticas (média, mediana, desvio padrão, percentis P10-P99, etc.) para cada uma das 12 métricas. Cria uma tabela de referências estatísticas do nicho na aba BenchMark.

**Tempo:** ~3 segundos

---

### **Automação 3: Níveis de Referência**
Transforma as estatísticas em 6 níveis de performance compreensíveis (Excepcional/Top 1%, Excelente/Top 5%, Muito Bom/Top 10%, Bom/Top 25%, Médio/Mediana, Abaixo da Média/Bottom 25%) e salva na aba ANÁLISE.

**Tempo:** <1 segundo

---

### **Automação 4: Ranking**
Compara cada vídeo com os benchmarks, calcula um score composto (0-100) usando pesos para 7 métricas, classifica e marca flags especiais (topPerformer, viralPotential, smallChannelWin). Remove duplicatas e ordena por score na aba "Dados ordenados".

**Pesos do Score:**
- Views: 25%, EngagementRate: 20%, OutlierScore: 20%, ViewsPerDay: 15%, LikeRate: 10%, CommentRate: 5%, TotalEngagementRate: 5%

**Tempo:** <1 segundo

---

## 🚧 Dificuldades

**Principal desafio:** Definir um rankeamento justo com pesos equilibrados que reflitam o que realmente importa no YouTube. Como balancear views (popularidade), engagement (qualidade), outlier score (viralização) e velocidade de crescimento em um único score?

---

## 🔧 Melhorias Futuras

### 1. **Substituir Apify por Scraper Próprio**
Actor do Apify é rápido e confiável, mas tem custo. Desenvolver scraper em Python eliminaria custos recorrentes.

### 2. **Multi-idioma Automático**
Atualmente busca só no idioma do termo pesquisado. Fazer requisições paralelas nos 5 idiomas mais falados (EN, ES, PT, FR, HI) e unificar na base enriqueceria os dados.

### 3. **Banco de Dados Real**
Google Sheets perde histórico a cada execução. Migrar para Supabase/Baserow permitiria múltiplas pesquisas simultâneas e análise temporal.

### 4. **Pesos Dinâmicos por Nicho**
Pesos fixos não se adaptam. Nichos diferentes têm prioridades diferentes (entretenimento = viralização, educação = engajamento).

### 5. **Detecção de Outliers com Desvio Padrão**
O código calcula desvio padrão mas não o usa para detectar outliers estatisticamente. Implementar Z-score ou IQR para identificação mais rigorosa.

---

## 💡 Vantagem Principal

**Benchmarks dinâmicos por nicho.** Cada nicho cria suas próprias referências de sucesso - o que é "viral" em um nicho pode ser mediano em outro. O sistema se adapta automaticamente ao contexto específico da pesquisa.
