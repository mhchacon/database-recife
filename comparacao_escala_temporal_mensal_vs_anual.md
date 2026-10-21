# Relatório Acadêmico: Estudo Comparativo entre Modelagem ANUAL vs MENSAL

> **Autoras:** Mariana (IC/UFRPE) & Orientação Acadêmica  
> **Objetivo:** Avaliar o impacto da resolução temporal (dados ANUAIS brutas vs dados MENSAIS desagregados via IDH_m) na modelagem preditiva e projeção estratégica da demanda hídrica de Recife para 2030 e 2050.

---

## 1. Motivação e Abordagem Metodológica

O script de referência para Caruaru (`projecao_consumo_agua_v2.py`) trabalha em escala **anual** (23 a 32 observações do SNIS). Para verificar se esse mesmo enfoque é ideal para Recife, desenvolvemos dois pipelines paralelos:

1. **Pipeline Anual:** Treinado diretamente sobre a série histórica anual de 1995 a 2026 ($N = 32$ amostras), utilizando a mesma suíte de modelos (Linear-MV, Ridge, Random Forest, Gradient Boosting e XGBoost).
2. **Pipeline Mensal:** Treinado sobre a série histórica desagrega com base empírica climática do INMET via Índice IDH_m ($N = 384$ amostras), utilizando XGBoost, LSTM, Ridge e Linear-MV.

---

## 2. Tabela Comparativa de Desempenho no Teste (2022–2026)

| Métrica / Dimensão | **Modelo MENSAL** ($N=384$) | **Modelo ANUAL** ($N=32$) | Vantagem Metodológica |
|---|---|---|---|
| **Amostragem para Treino** | 384 meses (144 treino / 60 teste) | 32 anos (27 treino / 5 teste) | **Mensal** (12x mais dados) |
| **Melhor Modelo no Teste** | **XGBoost ($R^2 = 0,9348$)** | **Linear-MV / Ridge ($MAPE = 6,29\%$)** | Mensal no XGBoost / Anual no Linear |
| **XGBoost em Amostra Pequena** | Funciona perfeitamente ($R^2 = 93,5\%$) | Oferece overfitting / saturação ($R^2 \approx 0$) | **Mensal** |
| **Erro Médio Relativo (MAPE)** | **~3,8%** | **6,29%** | **Mensal** |
| **Captura de Picos Climáticos** | ✅ Sim (Sazonalidade verão/inverno) | ❌ Não (Média anual oculta picos) | **Mensal** |
| **Projeção 2030 (Linear)** | **0,72 milhão $m^3$/ano** | **0,74 milhão $m^3$/ano** | Convergência entre modelos |
| **Projeção 2050 (Linear)** | **0,85 milhão $m^3$/ano** | **0,92 milhão $m^3$/ano** | Diferença de ~8% (margem segura) |

---

## 3. Principais Descobertas e Discussão

### 3.1 🔴 Limitação do Modelo Anual com Machine Learning Complexo (XGBoost / Random Forest)
Com apenas 32 linhas de treino (anual), algoritmos não-lineares avançados (como XGBoost ou Random Forest) sofrem com a limitação de volume de amostragem. No modelo anual, o XGBoost saturou e apresentou desempenho inferior aos modelos lineares simples.

### 3.2 🟢 Superioridade do Modelo Mensal para Algoritmos de IA
Ao desagregar a série em 384 meses usando o índice climático $IDH_m$, o XGBoost passa a ter observações suficientes para aprender os padrões sazonais finos, atingindo um **$R^2$ de 93,48%** e **MAPE de apenas 3,8%**.

### 3.3 🤝 Convergência nas Projeções de Longo Prazo (2030 e 2050)
Apesar da diferença de resolução:
- Para **2030**, ambos os modelos convergem praticamente para a mesma previsão: **0,72 milhão $m^3$** (mensal) vs **0,74 milhão $m^3$** (anual).
- Para **2050**, o modelo mensal prevê **0,85 milhão $m^3$** [IC95%: 0,79–0,90M] e o anual prevê **0,92 milhão $m^3$** [IC95%: 0,89–0,95M], uma variação de apenas 8%, o que traz imensa segurança científica.

---

## 4. Recomendação Científica para o Artigo / Dissertação

> **Recomendação para a Orientadora:**
>
> 1. **Usar ambos no artigo!** Apresentar o estudo comparativo valoriza imensamente a pesquisa. Mostra que você testou as duas resoluções temporais.
> 2. **Usar o modelo MENSAL como modelo principal de IA (XGBoost):** Pois ele atinge $R^2 = 93,5\%$ e captura a dinâmica de picos de consumo no verão pernambucano.
> 3. **Usar o modelo ANUAL como validação de longo prazo (Baseline):** Demonstrando que a projeção para 2030/2050 no modelo anual valida os valores encontrados no modelo mensal.
