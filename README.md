# Pipeline Preditivo de Manutencao Industrial

## Problema Resolvido

Paradas nao planejadas em linhas de producao industrial geram custos elevados de reparo corretivo e perda de produtividade. Este projeto implementa um pipeline completo de Ciencia de Dados e Aprendizado de Maquina para prever falhas mecanicas (`falha_maquina`) em equipamentos industriais a partir de leituras de telemetria de sensores fisicos.

A base de dados analisada (`data/manutencao_preditiva.csv`) contem 10.000 registros historicos e 14 colunas. Conforme o mapeamento tecnico do Departamento de Engenharia:
- **Atributos preditivos utilizados:** `tipo` (categoria de qualidade L, M ou H), `temperatura_ar_k` (Kelvin), `temperatura_processo_k` (Kelvin), `velocidade_rotacao_rpm` (RPM), `torque_nm` (Newton-metro), `desgaste_ferramenta_min` (minutos) e a variavel derivada `potencia`.
- **Atributos descartados por regra de negocio:** `udi` e `id_produto` (identificadores unicos de linha e serie sem valor preditivo) e as colunas `falha_twf`, `falha_hdf`, `falha_pwf`, `falha_osf` e `falha_rnf`, que registram a causa especifica da quebra apos o evento e causariam vazamento de dados (Data Leakage) caso mantidas em `X`.

## Tecnologias Utilizadas

- **Linguagem:** Python 3.13
- **Manipulacao e Analise de Dados:** Pandas (`3.1.1`) e NumPy (`2.5.4`)
- **Visualizacao de Dados:** Matplotlib (`3.11.2`) e Seaborn (`0.13.2`)
- **Pre-processamento e Modelagem:** Scikit-Learn (`1.9.1`) e Imbalanced-Learn (`0.14.2`)
- **Ambiente Interativo:** Jupyter / NBConvert (`7.17.2`) e IPyKernel (`7.4.0`)
- **Controle de Versao:** Git (fluxo Gitflow com branches por etapa do pipeline)

## Arquitetura do Pipeline

O arquivo `pipeline_preditivo.ipynb` foi estruturado de forma incremental em sete fases:

1. **Fase 1 - Exploracao de Dados (EDA):**
   - Inspecao estrutural (`.shape`, `.info()`, `.describe()`), confirmando 10.000 linhas, 14 colunas e 500 valores ausentes em quatro sensores continuos.
   - Identificacao de desbalanceamento severo na variavel alvo `falha_maquina`: 9.661 registros normais (96,61%) contra 339 falhas (3,39%).
   - Analise de correlacao de Pearson, destacando a correlacao positiva de `0.8766` entre `temperatura_ar_k` e `temperatura_processo_k` e a correlacao negativa de `-0.8754` entre `velocidade_rotacao_rpm` e `torque_nm`.

2. **Fase 2 - Limpeza e Tratamento de Dados:**
   - Verificacao de duplicatas (`0` registros duplicados).
   - Imputacao estatistica dos 500 valores ausentes baseada na assimetria de cada coluna: imputacao pela **media** em `temperatura_ar_k` (`300.0022 K`, skew `0.1149`), `temperatura_processo_k` (`310.0009 K`, skew `0.0156`) e `torque_nm` (`39.9742 Nm`, skew `-0.0079`); imputacao pela **mediana** em `velocidade_rotacao_rpm` (`1504.0000 RPM`, skew `1.9946`) devido a forte assimetria positiva.
   - Diagnostico de outliers via boxplots e IQR (465 outliers em `velocidade_rotacao_rpm` e 102 em `torque_nm`), mantendo os registros extremos por representarem regimes fisicos reais de sobrecarga.

3. **Fase 3 - Engenharia de Recursos (Feature Engineering):**
   - Criacao da coluna combinada `potencia = velocidade_rotacao_rpm * torque_nm` apos a imputacao de nulos, validada com zero valores `NaN` (media de `59961.21` e amplitude entre `10966.80` e `99980.40`).

4. **Fase 4 - Divisao e Balanceamento:**
   - Divisao estratificada (`train_test_split` com `test_size=0.20`, `stratify=y` e `random_state=42`), gerando 8.000 registros de treino (7.729 normais e 271 falhas) e 2.000 registros de teste (1.932 normais e 68 falhas).
   - Aplicacao do `SMOTE` exclusivamente sobre o conjunto de treino (`X_train`, `y_train`), equilibrando as classes em 7.729 amostras cada (`15.458` registros totais) e mantendo o teste intacto para impedir Data Leakage.

5. **Fase 5 - Escalonamento:**
   - Padronizacao das variaveis continuas via `StandardScaler` (`fit_transform` no treino balanceado e `transform` no teste) direcionada ao KNN.
   - Manutencao das escalas fisicas originais para a Arvore de Decisao, dado que particoes baseadas em impureza de Gini dependem apenas da ordenacao de cada variavel individual.

6. **Fase 6 e Fase 7 - Modelagem, Analise de Overfitting e Veredito:**
   - Treinamento comparativo do `KNeighborsClassifier` (`n_neighbors` = 3, 5, 7 e 9) nos dados escalonados e do `DecisionTreeClassifier` (`max_depth` = 3, 5, 10 e `None`) nos dados sem escalonar.

### Resumo dos Resultados Experimentais

| Modelo | Hiperparametro | Acuracia Treino | Acuracia Teste | Diferenca Treino-Teste | Recall Teste | F1 Teste |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| KNN | `n_neighbors=3` | 96,47% | 90,45% | 6,02% | 64,71% | 0,3154 |
| KNN | `n_neighbors=5` | 95,34% | 89,85% | 5,49% | 70,59% | 0,3211 |
| KNN | `n_neighbors=7` | 94,59% | 88,85% | 5,74% | 69,12% | 0,2965 |
| KNN | `n_neighbors=9` | 94,02% | 88,50% | 5,52% | 72,06% | 0,2988 |
| Arvore de Decisao | `max_depth=3` | 85,14% | 85,95% | -0,81% | 58,82% | 0,2216 |
| Arvore de Decisao | `max_depth=5` | 89,94% | 89,95% | -0,01% | 82,35% | 0,3578 |
| Arvore de Decisao | `max_depth=10` | 94,09% | 92,10% | 1,99% | 67,65% | 0,3680 |
| Arvore de Decisao | `max_depth=None` | 99,26% | 92,80% | 6,46% | 66,18% | 0,3846 |

**Modelo recomendado para adocao:** Arvore de Decisao com profundidade controlada (`max_depth=10` para equilibrio entre acuracia global de 92,10% e controle de overfitting, ou `max_depth=5` quando a meta operacional priorizar recall de 82,35% na captura de falhas), superando o melhor KNN (`90,45%`) em acuracia, F1-score, custo computacional de inferencia e interpretabilidade fisica.

## Como Executar o Projeto

1. Clone o repositorio e acesse a pasta do projeto:
   ```bash
   git clone <url-do-repositorio>
   cd analise-preditiva
   ```

2. Crie e ative um ambiente virtual Python:
   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```

3. Instale as dependencias listadas no `requirements.txt`:
   ```bash
   pip install -r requirements.txt
   ```

4. Abra ou execute o notebook principal:
   ```bash
   jupyter notebook pipeline_preditivo.ipynb
   ```
   Para reexecutar todas as celulas via terminal salvando as saidas:
   ```bash
   jupyter nbconvert --to notebook --execute --inplace pipeline_preditivo.ipynb
   ```

## Melhorias Futuras

- Criacao da variavel de diferencial termico (`delta_temperatura = temperatura_processo_k - temperatura_ar_k`) para capturar falhas especificas de dissipacao de calor (`falha_hdf`).
- Avaliacao de comites de arvores (Random Forest e Gradient Boosting / XGBoost) com validacao cruzada estratificada (`StratifiedKFold`).
- Ajuste do limiar de probabilidade de decisao (*threshold tuning*) orientado a matriz de custo financeiro entre falsos positivos (inspecao preventiva desnecessaria) e falsos negativos (quebra inesperada em linha).
- Empacotamento do modelo vencedor com `joblib` e exposicao via API REST (FastAPI) para consumo em tempo real pelos sensores da planta industrial.
