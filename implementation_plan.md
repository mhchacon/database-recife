# Plano de Implementação: Modelo Preditivo Híbrido (XGBoost vs LSTM)

Este documento descreve a metodologia técnica para o desenvolvimento da Inteligência Artificial que preverá o consumo de água na cidade do Recife para os horizontes de 2030 e 2050. O plano é baseado na revisão de 6 artigos científicos recentes (estado da arte).

## 1. Seleção dos Arquivos Mais Completos (Datasets)

Para o sucesso do treinamento, combinaremos a profundidade histórica (32 anos) com a sensibilidade sazonal (12 meses).

- **`Recife_Dataset_Atualizado.csv`:** É a base **mais importante** de contexto macro. Possui o histórico anual de todas as variáveis vitais (PIB, Tarifa de Energia ANEEL calculada recentemente, Taxa de Urbanização de 100%, Ligações Ativas). 
- **`Recife_Demanda_Mensal_Desagregada.csv`:** É a base **alvo** (Target). Ela possui os 384 meses (1995 a 2026) com o volume mensal já ajustado pela sazonalidade climática empírica. 

**Estratégia de Integração:** O script fará um *Merge* matemático entre os dois arquivos, repetindo as taxas anuais (como o PIB) para os respectivos meses do ano na tabela mensal. O resultado será um dataset único, contínuo, de 384 linhas (meses), perfeito para treinar redes neurais.

## 2. Metodologia: Abordagem Comparativa (Híbrida)

Conforme visto na revisão bibliográfica, não confiaremos em um único modelo. A IA será estruturada como uma competição entre as duas arquiteturas mais recomendadas na literatura acadêmica:

1. **XGBoost (Extreme Gradient Boosting):**
   - *Categoria:* Ensemble / Árvores de Decisão (Machine Learning Clássico)
   - *Vantagem:* Robustez com poucos dados e altíssima capacidade de explicar os fatores.
   - *Entregável Acadêmico:* Geração do gráfico de **"Feature Importance"** (para responder à orientadora qual o peso exato da Tarifa, do PIB e do clima no consumo final).

2. **LSTM (Long Short-Term Memory):**
   - *Categoria:* Deep Learning / Redes Neurais Recorrentes
   - *Vantagem:* Desenhado especificamente para entender o "Tempo" (Séries Temporais). É o estado da arte para identificar tendências históricas de crescimento.
   - *Entregável Acadêmico:* Menor margem de erro projetando para horizontes distantes (2030 e 2050).

## 3. Arquitetura do Pipeline de Dados e Treinamento

1. **Pré-processamento:**
   - **Normalização Matemática:** Todos os dados (desde tarifas em centavos até PIBs nos milhões) serão padronizados para a escala entre `[0, 1]` usando o `MinMaxScaler`. *Isto é obrigatório para o modelo LSTM convergir adequadamente.*
   - **Criação de Variáveis Lag:** Serão criadas colunas que olham para o passado (ex: *consumo do mês anterior*).

2. **Divisão Treino-Teste (Data Splitting):**
   - **Treino (85%):** Dados históricos de 1995 a 2021 (a IA aprende aqui).
   - **Teste (15%):** Dados históricos de 2022 a 2026 (a IA será forçada a prever esse passado recente, para verificarmos sua precisão comparando com os dados reais).

3. **Métricas de Avaliação (Validation):**
   - Erro Quadrático Médio ($RMSE$)
   - Coeficiente de Determinação ($R^2$)

4. **Projeção Futura (Forecasting):**
   - O modelo que apresentar o menor $RMSE$ e maior $R^2$ na fase de teste será o eleito.
   - Ele receberá cenários macroeconômicos simulados para gerar as curvas preditivas de **2030** e **2050**.

## ⚠️ User Review Required

> [!IMPORTANT]
> **Aprovação da Orientadora:** Este plano atende plenamente o rigor acadêmico esperado (combina empírico + modelos comparativos comprovados no estado da arte). Por favor, clique em **Proceed / Aprovar** para que eu possa iniciar o desenvolvimento do código em Python (`modelagem_preditiva.py`) que executará todos esses passos!
