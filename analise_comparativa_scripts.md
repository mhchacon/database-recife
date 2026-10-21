# Análise Comparativa: Script Caruaru (orientadora) × Modelo Recife

> **Objetivo:** Identificar o que o script `projecao_consumo_agua_v2.py` (Caruaru)  
> tem de valioso que pode ser incorporado ao pipeline de Recife.

---

## 1. Visão Geral dos Dois Pipelines

| Característica | **Script Caruaru** (`projecao_consumo_agua_v2.py`) | **Modelo Recife** (`modelagem_preditiva.py`) |
|---|---|---|
| **Autora/Origem** | Carolina (mestrado) — orientadora | Mariana (IC/UFRPE) |
| **Caso de estudo** | Caruaru/PE | Recife/PE |
| **Horizonte de projeção** | 2023–2034 (12 anos) | 2030 e 2050 |
| **Variáveis-alvo** | Vol. produzido + Vol. micromedido (anuais) | Consumo mensal desagregado |
| **Tipo de modelos** | Lineares (Linear-MV, Ridge) + Ensemble (RF, GB) | XGBoost + LSTM |
| **Validação** | Hold-out temporal (≤2017 treino, 2018–2022 teste) | Hold-out temporal (2022–2026 teste) |
| **PCA/PCR** | ✅ Sim — diagnóstico de multicolinearidade | ❌ Não — ausente |
| **Intervalo de Confiança** | ✅ IC 95% calculado explicitamente | ❌ Não implementado |
| **Escala temporal dos dados** | Anual (SNIS) | Mensal (desagregado via IDH_m) |
| **Variáveis exógenas** | Pop. Total, PIB/capita, Área urbanizada | PIB, Pop., IDH, Tarifa ANEEL, Infraestrutura SNIS |
| **Projeção de exógenas** | ✅ Regressão auxiliar explícita (2 etapas) | ⚠️ Parcial — projeção manual |
| **Exportação** | ✅ .xlsx multi-abas (audível) | ✅ .csv |
| **Análise de Feature Importance** | ❌ Não (modelos lineares) | ✅ Sim (XGBoost) |
| **Visualizações** | 5 gráficos diagnósticos (Scree, Biplot, PC1) | Gráfico de Feature Importance + Comparação LSTM vs XGB |
| **Documentação do código** | ✅ Exemplar — docstrings pedagógicas em cada função | ⚠️ Básica |
| **Robustez a multicolinearidade** | ✅ Ridge + PCR resolvem explicitamente | ⚠️ XGBoost é robusto, mas não diagnóstico |

---

## 2. Pontos Fortes EXCLUSIVOS do Script de Caruaru

### 2.1 ✅ PCA — Diagnóstico de Multicolinearidade

O script de Caruaru inclui um módulo completo de **Análise de Componentes Principais (PCA)** que:
- Detecta multicolinearidade entre as preditoras (crítico em modelos de regressão)
- Gera um **Scree Plot** (variância explicada por componente)
- Gera um **Biplot** (relação entre variáveis e observações no espaço reduzido)
- Gera a **trajetória temporal do PC1** (diagnóstico de tendência)

**Por que importa para Recife?** O dataset de Recife tem variáveis como PIB, tarifa de energia e IDH que são altamente correlacionadas com o tempo — multicolinearidade clássica. Um PCA revelaria isso e validaria as escolhas metodológicas perante a banca.

### 2.2 ✅ PCR — Regressão sobre Componentes Principais

O script implementa **Principal Component Regression (PCR)** como modelo robusto alternativo à regressão linear multivariada clássica. Testa k=1, 2, 3 componentes e reporta métricas comparativas.

**Por que importa para Recife?** PCR seria um modelo de *baseline* robusto e academicamente respeitável para comparar com o XGBoost — especialmente por modelos lineares extrapolarem melhor para 2050 (conforme comentário explícito no próprio script da orientadora).

### 2.3 ✅ Regressão Ridge (Regularização L2)

O script inclui **Ridge Regression** (`α=10`) como alternativa regularizada ao modelo linear clássico. Isso reduz a variância dos coeficientes em situações de multicolinearidade.

### 2.4 ✅ Intervalo de Confiança de 95% (IC95%)

Para cada projeção futura, o script calcula um **IC 95%** usando o RMSE in-sample:
```
IC95_inf = ŷ - 1.96 × RMSE
IC95_sup = ŷ + 1.96 × RMSE
```
Esse intervalo é visualizado como uma banda no gráfico de projeção.

**Por que importa para Recife?** Projeções para 2030 e 2050 **sem intervalos de confiança** são cientificamente incompletas. Qualquer revisor ou banca examinadora vai questionar isso.

### 2.5 ✅ Projeção em Duas Etapas das Variáveis Exógenas

O script implementa explicitamente a **projeção auxiliar das variáveis preditoras** (Pop, PIB, Área):
- Cada exógena é projetada com sua regressão individual (aproveitando máximo de dados)
- PIB é ajustado em escala logarítmica (crescimento multiplicativo)
- Os resultados servem de entrada para o modelo principal

**Por que importa para Recife?** O modelo de Recife precisará de PIB, população e tarifa para 2030 e 2050 — esse framework de "duas etapas" é a forma metodologicamente correta de fazê-lo.

### 2.6 ✅ Benchmark Sistemático de 5 Modelos

O script compara 5 algoritmos no **mesmo hold-out temporal**:
1. Linear-MV (multivariado)
2. Ridge (α=10)
3. Linear-Ano (baseline mínimo)
4. Random Forest
5. Gradient Boosting

E inclui a motivação estratégica: *"Modelos lineares extrapolam de forma contínua — desejável para horizontes > 5 anos. Modelos de árvore saturam fora do intervalo histórico."*

> ⚠️ **PONTO CRÍTICO para Recife:** Esta observação se aplica diretamente ao seu XGBoost. Para projeções até **2050**, um modelo linear calibrado pode ser **mais confiável** que o XGBoost para extrapolação de longo prazo. O XGBoost é melhor para interpolação/nowcasting.

### 2.7 ✅ Documentação Pedagógica Exemplar

Cada função tem docstrings com seções como "POR QUE PADRONIZAR ANTES?", "LIMITAÇÃO IMPORTANTE:", "A REGRA X". Esse nível de documentação é o padrão esperado em trabalhos acadêmicos e dissertações.

---

## 3. Pontos Fortes EXCLUSIVOS do Modelo Recife

| Vantagem | Descrição |
|---|---|
| **XGBoost + LSTM** | Modelos state-of-the-art com maior poder preditivo no período histórico |
| **Dados mensais (384 obs.)** | Muito mais observações que o dataset anual de Caruaru (23 obs.) |
| **Feature Importance** | Identifica quais variáveis realmente importam (tarifa ANEEL, IDH) |
| **Tarifa de Energia ANEEL** | Variável inovadora não presente em Caruaru |
| **Índice de Sazonalidade (IDH_m)** | Desagregação climaticamente fundamentada — diferencial metodológico |
| **Horizonte 2050** | Planejamento estratégico de longo prazo |
| **Dados SNIS de infraestrutura** | Perdas, extensão de rede, economias ativas |

---

## 4. Recomendações de Incorporação (O que levar de Caruaru para Recife)

### 🔴 PRIORITÁRIO — Implementar imediatamente

#### Rec. 1: Adicionar Intervalo de Confiança 95% nas projeções para 2030 e 2050

```python
# Adicionar ao modelagem_preditiva.py após gerar previsões
rmse_insample = float(np.sqrt(np.mean((y_train - modelo.predict(X_train))**2)))

df_projecao['IC95_inf'] = df_projecao['previsao'] - 1.96 * rmse_insample
df_projecao['IC95_sup'] = df_projecao['previsao'] + 1.96 * rmse_insample

# No gráfico:
ax.fill_between(anos_futuros, IC95_inf, IC95_sup, alpha=0.2, label='IC 95%')
```

**Justificativa:** Projeções para 2050 sem IC são inaceitáveis academicamente. Esta é a adição de menor esforço com maior impacto científico.

---

#### Rec. 2: Adicionar um Modelo Linear como Baseline para Projeções de Longo Prazo

O XGBoost tem alto R² no período histórico, mas **satura ao extrapolar**. Para 2030/2050, incluir também um modelo Linear-MV como baseline:

```python
from sklearn.linear_model import LinearRegression, Ridge

lr_baseline = LinearRegression()
lr_baseline.fit(X_train, y_train)
y_proj_linear = lr_baseline.predict(X_futuro)

ridge = Ridge(alpha=10, random_state=42)
ridge.fit(X_train, y_train)
y_proj_ridge = ridge.predict(X_futuro)
```

**Justificativa:** O script da orientadora documenta: *"Modelos baseados em árvore SATURAM ao extrapolar para fora do intervalo histórico... Modelos lineares extrapolam de forma contínua, o que é desejável para horizontes > 5 anos."*

---

### 🟡 IMPORTANTE — Incorporar na próxima versão

#### Rec. 3: Adicionar PCA como Seção Diagnóstica

Adaptando o código do script da orientadora diretamente para o dataset de Recife:

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

feats_pca = ['PIB_per_capita', 'Pop_Total', 'IDH_Municipal', 
             'Tarifa_Energia_Kwh', 'Perdas_Distribuicao_Pct']

scaler = StandardScaler().fit(df_treino[feats_pca])
Xz = scaler.transform(df_treino[feats_pca])
pca = PCA(n_components=len(feats_pca)).fit(Xz)

# Scree plot + Biplot → diagnóstico de multicolinearidade
```

**Justificativa:** Demonstra rigor metodológico e responde à pergunta "essas variáveis não são todas correlacionadas com o tempo?".

#### Rec. 4: Adicionar PCR como Modelo Robusto Alternativo

O PCR seria um terceiro modelo no benchmark (além de XGBoost e LSTM), especialmente útil para demonstrar que os resultados não são dependentes do algoritmo escolhido.

#### Rec. 5: Formalizar a Projeção das Variáveis Exógenas (2 Etapas Explícitas)

Criar funções explícitas (no estilo do script de Caruaru) para projetar PIB, população e tarifa ANEEL até 2050, com:
- Regressão linear temporal para cada variável
- PIB com ajuste em escala log
- Diagnósticos de R² por variável
- Exportação dos valores projetados

---

### 🟢 OPCIONAL — Melhorias de qualidade do código

#### Rec. 6: Adotar a Estrutura de Funções Documentadas

Refatorar `modelagem_preditiva.py` seguindo o padrão do script de Caruaru:
- `carregar_dados()` — centraliza leitura e padronização
- `projetar_exogenas()` — projeta variáveis preditoras
- `comparar_modelos()` — benchmark sistemático
- `projetar_alvo()` — projeção final com IC

#### Rec. 7: Exportar para .xlsx Multi-Abas (no estilo da orientadora)

```python
with pd.ExcelWriter('Resultados_Recife_Final.xlsx') as writer:
    df_treino.to_excel(writer, sheet_name='Dados_Hist', index=False)
    df_projecao.to_excel(writer, sheet_name='Projecao_2030_2050', index=False)
    df_metricas.to_excel(writer, sheet_name='Metricas_Modelos', index=False)
    df_pca_loadings.to_excel(writer, sheet_name='PCA_Loadings')
    df_feature_importance.to_excel(writer, sheet_name='Feature_Importance')
```

---

## 5. Diferença-Chave de Escopo: Por Que os Modelos São Complementares, Não Concorrentes

```
SCRIPT CARUARU                       MODELO RECIFE
─────────────                        ─────────────
Dados anuais (23 obs.)               Dados mensais (384 obs.)
Horizonte: 2023–2034 (curto-médio)  Horizonte: 2030–2050 (longo prazo)
Modelos lineares (melhor p/ extrap.) Modelos ensemble (melhor p/ ajuste)
Foco: volume produzido + micromedido Foco: consumo mensal desagregado
Cidade de médio porte (Caruaru)      Capital metropolitana (Recife)
```

> **Conclusão:** Os dois scripts são complementares. Para Recife, a estratégia ideal é:
> - Usar **XGBoost/LSTM** para validação histórica e análise de variáveis (Feature Importance)
> - Usar **Linear-MV com IC95%** para as projeções finais de 2030 e 2050 (extrapolação mais segura)
> - Usar **PCA** como ferramenta diagnóstica na seção de metodologia

---

## 6. Plano de Ação Proposto

| # | Ação | Prioridade | Esforço |
|---|---|---|---|
| 1 | Adicionar IC95% nas projeções existentes | 🔴 Alta | 🟢 Baixo (30 min) |
| 2 | Adicionar modelo Linear-MV como baseline para 2050 | 🔴 Alta | 🟢 Baixo (1h) |
| 3 | Adicionar seção PCA com Scree Plot e Biplot | 🟡 Média | 🟡 Médio (2h) |
| 4 | Formalizar projeção 2-etapas das exógenas | 🟡 Média | 🟡 Médio (2h) |
| 5 | Refatorar código com funções documentadas | 🟢 Baixa | 🔴 Alto (4h) |
| 6 | PCR como terceiro modelo no benchmark | 🟢 Baixa | 🟡 Médio (1h) |
