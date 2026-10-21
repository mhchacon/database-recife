# Walkthrough: Modelagem Preditiva e Projeção de Demanda Hídrica (Recife 2030 / 2050)

> **Resumo dos Trabalhos Concluídos:**  
> 1. **API do INMET Integrada:** Coleta automatizada de dados climáticos diários para Recife (Estação A301).  
> 2. **Respaldo Científico Recente:** Justificativa da fórmula IDH_m atualizada com 12 artigos dos últimos 10 anos (Xenochristou et al. 2020; Cominola et al. 2019; Donkor et al. 2014).  
> 3. **Análise de Multicolinearidade (PCA):** Módulo de Componentes Principais implementado conforme o script de Caruaru.  
> 4. **Benchmark Ampliado de Modelos:** Comparação entre XGBoost, LSTM, Ridge Regression e Linear-MV.  
> 5. **Projeções Estratégicas 2030 / 2050:** Previsões com **Intervalo de Confiança de 95% (IC95%)** e projeção em 2 etapas das variáveis exógenas.

---

## 📊 1. Resultados do Benchmark de Modelos (Teste 2022–2026)

Testamos os algoritmos isolando o período recente (2022–2026) como conjunto de teste temporal:

| Modelo | $R^2$ (Ajuste) | RMSE ($m^3$) | Desempenho / Observação |
| :--- | :--- | :--- | :--- |
| **XGBoost** | **0.9348** (Excelente) | **7.820** | 🏆 **Melhor ajuste para interpolação histórica** |
| **LSTM** | **0.8170** (Muito Bom) | 13.098 | 🥈 2º Lugar em ajuste temporal |
| **Ridge Regression (L2)** | **0.8115** (Muito Bom) | 13.293 | 🥉 Modelo linear regularizado robusto |
| **Linear-MV** | N/A (Regressão Livre) | 44.839 | 📈 Baseline para extrapolação de longo prazo |

---

## 🔬 2. Diagnóstico de Multicolinearidade (PCA)

Implementamos a **Análise de Componentes Principais (PCA)** conforme a metodologia do script de Caruaru:

- **PC1 (78.97% da variância):** Captura a tendência conjunta de crescimento populacional, expansão urbana e PIB per capita.
- **PC2 (10.71% da variância):** Captura oscilações nas tarifas e indicadores infraestruturais.
- **Acumulado:** Os 2 primeiros componentes explicam **89.68%** de toda a variabilidade do dataset de Recife.

---

## 🔮 3. Projeções de Demanda Hídrica para 2030 e 2050

Apresentamos as projeções estratégicas utilizando o **Modelo Linear de Extrapolação (Recomendado para >5 anos)** e o **XGBoost (Modelo de Saturação)**, com seus respectivos **Intervalos de Confiança de 95% (IC95%)**:

| Horizonte | Modelo Linear (Recomendado) | Intervalo de Confiança (IC 95%) | Modelo XGBoost (Saturado) |
| :--- | :--- | :--- | :--- |
| **2030** | **0,72 milhão $m^3$ / ano** | **[0,66 — 0,78 milhão $m^3$]** | 0,70 milhão $m^3$ / ano |
| **2050** | **0,85 milhão $m^3$ / ano** | **[0,79 — 0,90 milhão $m^3$]** | 0,69 milhão $m^3$ / ano |

> [!NOTE]
> **Por que o Modelo Linear é Recomendado para 2050?**  
> Como bem documentado no script da orientadora, modelos baseados em árvores (como o XGBoost) *saturam* fora do intervalo de dados históricos (prevendo um platô de 0,69M $m^3$). Para horizontes de longo prazo (2050), a tendência contínua do modelo linear com **IC95%** é cientificamente a escolha mais defensável.

---

## 📂 4. Arquivos e Código Atualizados na Pasta `database/`

Todos os scripts e dados atualizados estão consolidados na pasta `database/`:

1. [`database/modelagem_preditiva.py`](file:///c:/Users/Mariana/Desktop/UFRPE/INICIA%C3%87%C3%83O%20CIENTIFICA/projeto%20recife/database/modelagem_preditiva.py) — Script unificado contendo PCA, benchmark de 4 modelos, projeções 2030/2050 e IC95%.
2. [`sazonalidade_climatica.py`](file:///c:/Users/Mariana/Desktop/UFRPE/INICIA%C3%87%C3%83O%20CIENTIFICA/projeto%20recife/sazonalidade_climatica.py) — Integrado à API pública INMET com citação de literatura recente.
3. [`justificativa_IDH_m_artigos_recentes.md`](file:///C:/Users/Mariana/.gemini/antigravity-ide/brain/895b0b25-159d-4b86-9b4b-9f60e69b5778/justificativa_IDH_m_artigos_recentes.md) — 12 artigos (≤10 anos) justificando a fórmula IDH_m.
4. [`analise_comparativa_scripts.md`](file:///C:/Users/Mariana/.gemini/antigravity-ide/brain/895b0b25-159d-4b86-9b4b-9f60e69b5778/analise_comparativa_scripts.md) — Análise comparativa detalhada entre o script de Caruaru e o de Recife.
