# MachGuard ML - Sistema Preditivo de Manutencao Industrial

## 1. Visao Geral do Sistema

Na Industria 4.0, paradas nao programadas em linhas de manufatura geram perdas financeiras expressivas, atrasos logisticos e custos elevados de manutencao corretiva emergencial. Ao mesmo tempo, substituir componentes preventivamente antes do fim de sua vida util desperdiça recursos e interrompe equipamentos que ainda operam em condicoes seguras.

O **MachGuard ML** foi desenvolvido para solucionar esse problema por meio de um pipeline completo de Ciencia de Dados e Aprendizado de Maquina Supervisionado. O sistema processa leituras historicas de sensores acoplados ao maquinario industrial — temperatura ambiente, temperatura do processo de usinagem, velocidade de rotacao do motor, torque aplicado e tempo acumulado de desgaste da ferramenta — para realizar uma **classificacao binaria** do estado operacional do equipamento:
- **`Falha = 0` (Operacao Normal):** O equipamento opera dentro dos parametros seguros de estabilidade termica e mecanica.
- **`Falha = 1` (Falha Mecanica):** O equipamento encontra-se em regime critico de quebra iminente, acionando a intervencao preditiva da equipe de engenharia antes da parada catastrofica da linha.

---

## 2. Tecnologias e Bibliotecas Utilizadas

O projeto foi construido em linguagem **Python (`3.13.15`)**, utilizando as seguintes bibliotecas cientificas e ferramentas de versionamento:

- **Pandas (`3.1.1`):** Leitura do dataset em formato CSV, inspecao estrutural, remocao de duplicatas, tratamento estatistico de valores ausentes, engenharia de atributos e tabulacao de metricas.
- **NumPy (`2.5.4`):** Operacoes vetoriais, calculos numericos de suporte e manipulacao de arranjos matriciais.
- **Matplotlib (`3.11.2`) e Seaborn (`0.13.2`):** Construcao das visualizacoes estatisticas da analise exploratoria (histogramas com curva de densidade, grafico de barras de proporcao de classes, mapa de calor da correlacao de Pearson, boxplots de inspecao de outliers e grafico comparativo de desempenho entre modelos).
- **Scikit-Learn (`1.9.1`):** Divisao estratificada da base (`train_test_split`), padronizacao de variaveis continuas (`StandardScaler`), treinamento dos algoritmos `KNeighborsClassifier` (KNN) e `DecisionTreeClassifier` (Arvore de Decisao) e afericao de metricas de validacao (`accuracy_score`, `recall_score` e `f1_score`).
- **Imbalanced-Learn (`0.14.2`):** Aplicacao da tecnica de superamostragem sintetica `SMOTE` (*Synthetic Minority Over-sampling Technique*) para correcao do desbalanceamento da classe alvo no conjunto de treinamento.
- **Jupyter / NBConvert (`7.17.2`) e IPyKernel (`7.4.0`):** Execucao sequencial automatizada do notebook `pipeline_preditivo.ipynb` com persistencia das saidas graficas e textuais.
- **Git e GitHub:** Controle de versao distribuido estruturado no modelo Gitflow com branches dedicadas para cada etapa do pipeline.

---

## 3. Como Executar o Sistema

Siga as instrucoes abaixo para reproduzir o ambiente e executar o pipeline localmente:

1. **Clonar o repositorio:**
   ```bash
   git clone https://github.com/codeCau/machguard-ml.git
   cd machguard-ml
   ```

2. **Criar e ativar um ambiente virtual Python (opcional, mas recomendado):**
   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```

3. **Instalar as dependencias do projeto:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Verificar a estrutura da pasta de dados (`/data`):**
   Certifique-se de que o arquivo `manutencao_preditiva.csv` permanece dentro do diretorio `data/` na raiz do projeto. Todo o codigo do notebook foi implementado utilizando o caminho relativo `data/manutencao_preditiva.csv`, garantindo portabilidade entre diferentes sistemas operacionais sem necessidade de alterar caminhos absolutos no codigo-fonte.

5. **Executar o notebook sequencialmente:**
   Abra o arquivo `pipeline_preditivo.ipynb` no Jupyter Notebook, JupyterLab ou VS Code e execute todas as celulas em ordem sequencial (de cima para baixo):
   ```bash
   jupyter notebook pipeline_preditivo.ipynb
   ```
   Caso deseje reexecutar todo o pipeline via terminal atualizando as saidas diretamente no arquivo `.ipynb`, utilize:
   ```bash
   jupyter nbconvert --to notebook --execute --inplace pipeline_preditivo.ipynb
   ```

---

## 4. Organizacao das Tarefas e Fluxo de Branches (Git)

### Planejamento Arquitetural em 7 Fases Sequenciais

Antes de iniciar a codificacao dos algoritmos de Machine Learning, estruturei o projeto dividindo o trabalho em **7 fases estritamente sequenciais**. Em projetos de Ciencia de Dados, a ordem das transformacoes impacta diretamente a validade estatistica do modelo:
- Não e possivel criar a coluna de potencia na Engenharia de Recursos antes de limpar e imputar os valores ausentes das colunas de rotacao e torque, caso contrario operacoes aritmeticas propagariam valores `NaN`.
- Não e permitido aplicar balanceamento de classes (`SMOTE`) ou escalonamento (`StandardScaler`) antes de dividir a base entre treino e teste (`train_test_split`), pois isso contaminaria os dados de teste com informacoes do treino (*Data Leakage*) e invalidaria toda a avaliacao posterior.

O planejamento previo em 7 etapas isoladas evitou retrabalho, garantiu que cada pre-requisito matematico fosse cumprido no momento correto e permitiu auditar a evolucao do arquivo `pipeline_preditivo.ipynb` commit a commit.

### Estrutura de Branches do Repositorio

O versionamento adotou uma estrategia baseada em Gitflow, onde nenhuma funcionalidade foi desenvolvida diretamente na `main`. Cada fase do pipeline foi implementada em sua respectiva branch de funcionalidade criada a partir da `develop` e integrada via `git merge --no-ff` para preservar o historico completo na arvore do Git:

| Branch | Origem | Destino do Merge | Objetivo Especifico no Projeto |
| :--- | :--- | :--- | :--- |
| `main` | Raiz | — | Branch principal de producao contendo a estrutura base inicial e a release final consolidada do pipeline. |
| `develop` | `main` | `main` | Branch central de integracao onde todas as etapas do pipeline foram reunidas progressivamente. |
| `feature/eda` | `develop` | `develop` | Implementacao da Fase 1: inspecao estrutural (`.shape`, `.info()`, `.describe()`), plotagem dos 3 graficos obrigatorios e redacao da analise interpretativa dos dados brutos. |
| `feature/data-prep` | `develop` | `develop` | Implementacao da Fase 2: verificacao/remocao de duplicados, analise de assimetria (`skewness`), imputacao de valores ausentes por media/mediana e inspecao de outliers com boxplots e IQR. |
| `feature/feature-engineering` | `develop` | `develop` | Implementacao da Fase 3: criacao da variavel fisica combinada `potencia` apos a imputacao de nulos e validacao em codigo da ausencia de registros `NaN`. |
| `feature/split-balance` | `develop` | `develop` | Implementacao da Fase 4: descarte das colunas pos-evento/identificadoras, divisao estratificada 80/20 (`stratify=y`) e balanceamento com `SMOTE` restrito ao conjunto de treino. |
| `feature/scaling` | `develop` | `develop` | Implementacao da Fase 5: padronizacao das variaveis continuas via `StandardScaler` exclusiva para o KNN e documentacao da imunidade da Arvore de Decisao a escala. |
| `feature/modeling` | `develop` | `develop` | Implementacao das Fases 6 e 7: treinamento do KNN e da Arvore de Decisao variando hiperparametros, diagnostico de *overfitting*, comparacao na base de teste e execucao integral do notebook. |
| `docs/readme` | `develop` | `develop` | Geracao do arquivo `requirements.txt` enxuto e elaboracao da documentacao tecnica e defesa das escolhas arquiteturais no `README.md`. |

---

## 5. Defesa Tecnica e Escolhas de Projeto (Etapas 1 a 7)

### 5.1. Exploracao de Dados (EDA) e Limpeza / Tratamento (Data Prep)

A leitura inicial do arquivo `data/manutencao_preditiva.csv` revelou exatamente **`10.000` linhas e `14` colunas**. A partir das anotacoes tecnicas do Departamento de Engenharia, identifiquei que:
- As colunas `udi` (1 a 10.000) e `id_produto` (10.000 codigos de serie unicos) sao chaves identificadoras sem poder de generalizacao.
- As cinco colunas binarias `falha_twf`, `falha_hdf`, `falha_pwf`, `falha_osf` e `falha_rnf` registram o modo especifico da falha somente apos a ocorrencia da quebra; mante-las como variaveis explicativas configuraria vazamento direto da variavel alvo.
- A coluna categorica `tipo` classifica a especificacao de qualidade do maquinario em tres niveis: `L` com `6.000` unidades (`60,00%`), `M` com `2.997` unidades (`29,97%`) e `H` com `1.003` unidades (`10,03%`).

**Padroes identificados nos 3 graficos da EDA:**
1. **Histogramas das variaveis preditoras:** As variaveis `temperatura_ar_k` (media de `300,0022 K` e mediana de `300,1000 K`), `temperatura_processo_k` (media de `310,0009 K` e mediana de `310,1000 K`) e `torque_nm` (media de `39,9742 Nm` e mediana de `40,1000 Nm`) apresentam comportamento unimodal e simetrico, proximo a uma distribuicao normal. Já `desgaste_ferramenta_min` distribui-se de forma quase uniforme entre `0` e `253` minutos (media de `107,9510` min e mediana de `108,0000` min). Em contrapartida, `velocidade_rotacao_rpm` apresenta acentuada assimetria positiva (cauda longa a direita alcancando `2.886,0 RPM`), o que desloca sua media (`1.539,2453 RPM`) para cima de sua mediana (`1.504,0000 RPM`).
2. **Grafico de barras da variavel alvo (`falha_maquina`):** Confirmou um severo desbalanceamento entre as classes, contabilizando **`9.661` registros de operacao normal (`96,61%`)** contra apenas **`339` registros de falha mecanica (`3,39%`)**.
3. **Mapa de calor da correlacao de Pearson:** Evidenciou duas correlacoes lineares intensas explicadas pela fisica do equipamento: correlacao positiva forte de **`0,8766`** entre `temperatura_ar_k` e `temperatura_processo_k` (acoplamento termico entre o calor do ambiente e o calor gerado na usinagem) e correlacao negativa forte de **`-0,8754`** entre `velocidade_rotacao_rpm` e `torque_nm` (relacao inversa entre velocidade angular e carga de torcao no eixo do motor). Em relacao a `falha_maquina`, as maiores correlacoes diretas ocorreram com `torque_nm` (`0,1889`) e `desgaste_ferramenta_min` (`0,1054`).

**Remocao de duplicados e Imputacao estatistica de valores nulos:**
- A verificacao `df.duplicated().sum()` apontou **`0` registros integralmente duplicados**, preservando as `10.000` linhas apos a execucao de `df.drop_duplicates()`.
- O diagnostico de dados faltantes identificou exatamente **`500` valores nulos (`NaN`)** em cada uma das quatro variaveis continuas de sensores: `temperatura_ar_k`, `temperatura_processo_k`, `velocidade_rotacao_rpm` e `torque_nm` (todas com `9.500` valores preenchidos). As demais `10` colunas não apresentavam valores nulos.
- Para evitar o descarte de `5,00%` da base de dados, apliquei imputacao estatistica selecionando a medida de tendencia central de acordo com o coeficiente de assimetria (*skewness*) real de cada coluna:
  - **Imputacao pela Media** nas colunas `temperatura_ar_k` (skew = `0,1149`, preenchida com **`300,0022 K`**), `temperatura_processo_k` (skew = `0,0156`, preenchida com **`310,0009 K`**) e `torque_nm` (skew = `-0,0079`, preenchida com **`39,9742 Nm`**). Como essas tres distribuicoes sao simetricas (`|skew| < 0,15`), a media aritmetica representa fielmente o centro da distribuicao sem sofrer distorcao.
  - **Imputacao pela Mediana** na coluna `velocidade_rotacao_rpm` (skew = `1,9946`, preenchida com **`1.504,0000 RPM`**). Devido a forte cauda positiva ate `2.886,0 RPM`, a media (`1.539,2453 RPM`) e inflada pelos valores extremos; a mediana, por ser uma estatistica de ordem robusta a *outliers*, evita introduzir vies nos 500 registros imputados.

**Analise de Outliers via Boxplots:**
A inspecao visual pelos boxplots combinada ao calculo do intervalo interquartil ($\text{IQR} = Q_3 - Q_1$) apos a imputacao contabilizou:
- `velocidade_rotacao_rpm`: **`465` outliers** (acima de `1.874,50 RPM`, com $\text{IQR} = 179,00$).
- `torque_nm`: **`102` outliers** (fora dos limites `[14,40 Nm, 65,60 Nm]`, com $\text{IQR} = 12,80$).
- `temperatura_processo_k`: **`2` outliers** (abaixo de `305,75 K`, com $\text{IQR} = 2,10$).
- `temperatura_ar_k` ($\text{IQR} = 3,00$) e `desgaste_ferramenta_min` ($\text{IQR} = 109,00$): **`0` outliers**.

Decidi **manter todos os outliers na base**, pois no dominio de manutencao preditiva leituras extremas de torque (ate `76,60 Nm`) e de rotacao (ate `2.886,0 RPM`) não sao ruidos de coleta, mas sim os eventos fisicos reais de sobrecarga mecanica que causam a quebra do equipamento. Exclui-los impediria os modelos de aprenderem os padroes das falhas por tensao e potencia.

---

### 5.2. Engenharia de Recursos (Feature Engineering)

Na Fase 3, criei a coluna numerica combinada `potencia` a partir da formula exata:

```python
df["potencia"] = df["velocidade_rotacao_rpm"] * df["torque_nm"]
```

**Justificativa fisica e matematica:** Na dinamica de maquinas rotativas, a potencia mecanica desenvolvida no eixo do motor e diretamente proporcional ao produto entre a velocidade angular e o torque aplicado ($P \propto \omega \cdot \tau$). Como constatamos na EDA que `velocidade_rotacao_rpm` e `torque_nm` possuem correlacao negativa de `-0,8754`, observar cada coluna isoladamente dificulta a separacao dos regimes em que o produto das duas grandezas ultrapassa os limites operacionais do motor (causando `falha_pwf`). A nova variavel `potencia` apresentou media de **`59.961,21`**, mediana de **`60.121,15`**, desvio padrao de **`9.951,11`**, valor minimo de **`10.966,80`** e valor maximo de **`99.980,40`**.

Por ter sido gerada estritamente apos o tratamento de nulos da Fase 2, a validacao programatica `df["potencia"].isnull().sum()` confirmou **`0` valores nulos (`NaN`)** nos `10.000` registros.

---

### 5.3. Divisao Estratificada e Balanceamento de Classes

Para a modelagem, codifiquei a coluna categorica ordinal `tipo` (`L = 0`, `M = 1`, `H = 2`) e defini a matriz preditora `X` com 7 atributos (`tipo`, `temperatura_ar_k`, `temperatura_processo_k`, `velocidade_rotacao_rpm`, `torque_nm`, `desgaste_ferramenta_min` e `potencia`) e o vetor alvo `y` com `falha_maquina`, descartando `udi`, `id_produto` e as 5 colunas pos-evento (`falha_twf` a `falha_rnf`).

**Uso do `stratify=y` na divisao 80/20:**
A separacao via `train_test_split(X, y, test_size=0.20, stratify=y, random_state=42)` dividiu os `10.000` registros em:
- **Conjunto de Treino (`80%`):** **`8.000` registros**, contendo **`7.729` amostras da classe `0` (`96,61%`)** e **`271` amostras da classe `1` (`3,39%`)**.
- **Conjunto de Teste (`20%`):** **`2.000` registros**, contendo **`1.932` amostras da classe `0` (`96,60%`)** e **`68` amostras da classe `1` (`3,40%`)**.

O parametro `stratify=y` e indispensavel em bases desbalanceadas porque uma amostragem puramente aleatoria poderia concentrar acidentalmente a maior parte das `339` falhas apenas no treino ou apenas no teste, distorcendo o aprendizado ou o calculo das metricas finais.

**Prevencao de *Data Leakage* no Balanceamento com `SMOTE`:**
Apliquei o algoritmo `SMOTE(random_state=42)` **exclusivamente sobre os dados de treino (`X_train`, `y_train`)**:
- **Treino antes do `SMOTE`:** `8.000` registros (`7.729` da classe `0` e `271` da classe `1`).
- **Treino depois do `SMOTE` (`X_train_bal`, `y_train_bal`):** **`15.458` registros**, perfeitamente equilibrados com **`7.729` amostras da classe `0` (`50,00%`)** e **`7.729` amostras da classe `1` (`50,00%`)**.
- **Teste (`X_test`, `y_test`):** Mantido **intacto com os `2.000` registros originais** (`1.932` da classe `0` e `68` da classe `1`).

Caso o balanceamento fosse executado na base inteira antes do `train_test_split`, o `SMOTE` criaria exemplos sinteticos interpolando registros que posteriormente iriam para o treino com vizinhos que iriam para o teste. Isso causaria vazamento de dados (*Data Leakage*): o modelo teria acesso indireto a geometria dos dados de teste durante o treinamento, gerando uma acuracia artificialmente alta que falharia no mundo real.

---

### 5.4. Escalonamento de Variaveis e Imunidade da Arvore de Decisao

Na Fase 5, apliquei o `StandardScaler` apenas nas 6 variaveis continuas (`temperatura_ar_k`, `temperatura_processo_k`, `velocidade_rotacao_rpm`, `torque_nm`, `desgaste_ferramenta_min` e `potencia`) para o treinamento do KNN, utilizando `scaler.fit_transform()` no treino balanceado (`X_train_bal`) e apenas `scaler.transform()` no conjunto de teste (`X_test`). Apos a transformacao, todas as 6 colunas continuas no treino passaram a ter media **`0,0000`** e desvio padrao **`1,0000`**.

- **Por que o KNN exige escalonamento:** O algoritmo K-Nearest Neighbors classifica uma nova observacao calculando a distancia geometrica (distancia Euclidiana) entre pontos no espaco de atributos:
  $$d(p, q) = \sqrt{\sum_{j=1}^{m} (p_j - q_j)^2}$$
  Em nossa base, a variavel `potencia` atinge valores de ate `99.980,40`, enquanto `temperatura_ar_k` varia apenas entre `295,30 K` e `304,50 K` (amplitude de `9,20 K`). Sem padronizacao, a diferenca ao quadrado da coluna `potencia` seria da ordem de $10^9$, anulando completamente a influencia das temperaturas e do torque no calculo dos vizinhos mais proximos. O uso de `fit_transform` restrito ao treino e `transform` no teste garante ainda que a media e a variancia do conjunto de teste nao vazem para o escalonador.
- **Por que a Arvore de Decisao foi mantida sem escalonamento:** Algoritmos baseados em arvores de decisao sao matematicamente **imunes a escala das variaveis**. Em vez de calcular distancias entre multiplos atributos simultaneamente, a Arvore de Decisao avalia uma unica variavel por vez em cada no, procurando o ponto de corte $x_j \le t$ que maximize o ganho de informacao (reducao da impureza de Gini ou Entropia). Como a padronizacao $z = (x - \mu)/\sigma$ e uma transformacao linear estritamente monotona, a ordem relativa das amostras não muda: qualquer limiar na escala original divide exatamente os mesmos subconjuntos de linhas na escala padronizada, produzindo o mesmo indice de Gini e a mesma arvore. Preservar os dados sem escalonar para a Arvore de Decisao mantem os pontos de corte nas unidades fisicas originais (`K`, `RPM`, `Nm` e `min`), permitindo interpretacao imediata pelos engenheiros de manutencao.

---

### 5.5. Controle de Overfitting e Veredito Final

Na Fase 6, treinei o `KNeighborsClassifier` variando `n_neighbors` em 4 valores impares (`3`, `5`, `7` e `9`) nos dados escalonados e o `DecisionTreeClassifier` (`random_state=42`) variando `max_depth` em 4 configuracoes (`3`, `5`, `10` e `None`) nos dados sem escalonar. Os resultados exatos obtidos no notebook estao consolidados na tabela abaixo:

| Modelo | Hiperparametro | Acuracia Treino | Acuracia Teste | Diferenca (Treino - Teste) | Recall Teste (Classe 1) | F1-Score Teste |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **KNN** | `n_neighbors=3` | `96,47%` (`0,9647`) | `90,45%` (`0,9045`) | `+6,02%` (`0,0602`) | `64,71%` (`0,6471`) | `0,3154` |
| **KNN** | `n_neighbors=5` | `95,34%` (`0,9534`) | `89,85%` (`0,8985`) | `+5,49%` (`0,0549`) | `70,59%` (`0,7059`) | `0,3211` |
| **KNN** | `n_neighbors=7` | `94,59%` (`0,9459`) | `88,85%` (`0,8885`) | `+5,74%` (`0,0574`) | `69,12%` (`0,6912`) | `0,2965` |
| **KNN** | `n_neighbors=9` | `94,02%` (`0,9402`) | `88,50%` (`0,8850`) | `+5,52%` (`0,0552`) | `72,06%` (`0,7206`) | `0,2988` |
| **Arvore de Decisao** | `max_depth=3` | `85,14%` (`0,8514`) | `85,95%` (`0,8595`) | `-0,81%` (`-0,0081`) | `58,82%` (`0,5882`) | `0,2216` |
| **Arvore de Decisao** | `max_depth=5` | `89,94%` (`0,8994`) | `89,95%` (`0,8995`) | `-0,01%` (`-0,0001`) | `82,35%` (`0,8235`) | `0,3578` |
| **Arvore de Decisao** | `max_depth=10` | `94,09%` (`0,9409`) | `92,10%` (`0,9210`) | `+1,99%` (`0,0199`) | `67,65%` (`0,6765`) | `0,3680` |
| **Arvore de Decisao** | `max_depth=None` | `99,26%` (`0,9926`) | `92,80%` (`0,9280`) | `+6,46%` (`0,0646`) | `66,18%` (`0,6618`) | `0,3846` |

**Identificacao exata de onde ocorreu *Overfitting*:**
1. **Na Arvore de Decisao com `max_depth=None`:** Sem restricao de profundidade, a arvore cresceu ate **33 niveis de profundidade e 769 folhas**, memorizando as particularidades e pontos sinteticos do conjunto de treino com **`99,26%`** de acuracia, mas recuando para **`92,80%`** nos dados ineditos de teste. Esse distanciamento de **`6,46` pontos percentuais** caracteriza o principal episodio de *overfitting* do experimento. Em contraste, quando restringimos a profundidade para `max_depth=5` (`89,94%` no treino vs `89,95%` no teste) ou `max_depth=10` (`94,09%` no treino vs `92,10%` no teste, diferenca de apenas `1,99%`), o sobreajuste e controlado com sucesso.
2. **No KNN com `n_neighbors=3`:** Dentre as variacoes do KNN, o valor $k = 3$ apresentou o maior grau de *overfitting*, alcancando **`96,47%`** no treino contra **`90,45%`** no teste (queda de **`6,02` pontos percentuais**), pois uma vizinhanca muito pequena torna a fronteira de classificacao excessivamente sensivel a ruidos locais do conjunto balanceado. Conforme aumentamos $k$ para `5`, `7` e `9`, a acuracia de treino diminui gradualmente (`95,34%`, `94,59%` e `94,02%`), suavizando a superficie de decisao.

**Veredito Final e Modelo Vencedor:**
Comparando o melhor resultado do KNN (`n_neighbors=3`, com **`90,45%`** de acuracia de teste) com a **Arvore de Decisao**, constata-se que a **Arvore de Decisao e o modelo vencedor**, superando o KNN tanto na configuracao irrestrita (`max_depth=None` com **`92,80%`** de acuracia de teste) quanto na configuracao podada livre de sobreajuste severo (`max_depth=10` com **`92,10%`** de acuracia de teste).

Recomendo que a empresa adote em producao a **Arvore de Decisao** (preferencialmente com **`max_depth=10`** para aliar alta acuracia de teste de `92,10%` e robustez contra *overfitting*, ou `max_depth=None` caso o criterio unico seja a maior acuracia absoluta de `92,80%`), pelos seguintes fatores decisivos:
1. **Superioridade preditiva na base de teste:** Supera o melhor KNN em ate **`2,35` pontos percentuais** de acuracia global (`92,80%` / `92,10%` contra `90,45%`) e alcanca maior F1-Score (`0,3846` / `0,3680` contra `0,3154`).
2. **Desempenho computacional em tempo real:** O KNN exige armazenar na memoria RAM os `15.458` registros de treino e calcular `15.458` distancias euclidianas a cada leitura de sensor ($O(N \cdot d)$). A Arvore de Decisao realiza a predicao executando apenas algumas comparacoes condicionais (`if/else`) em tempo de ordem $O(\text{profundidade})$, viabilizando a implantacao direta em sistemas embarcados na linha de producao.
3. **Auditabilidade pela Engenharia:** Por operar sobre as grandezas originais sem `StandardScaler`, permite inspecionar transparentemente quais limiares fisicos de `potencia`, `torque_nm` e `desgaste_ferramenta_min` determinaram cada alerta de manutencao.

---

## 6. Limitacoes Atuais e Melhorias Futuras

Embora o pipeline implementado cumpra integralmente os requisitos avaliativos do projeto, uma analise critica voltada para um ambiente industrial de missao critica aponta as seguintes limitacoes atuais e evolucoes planejadas:

1. **Selecao de Modelos Orientada a Recall, Precisao e F1-Score (Custo de Falsos Negativos):**
   Neste projeto, a escolha final do modelo vencedor baseou-se na maior **Acuracia de Teste** conforme especificado no escopo da avaliacao. Entretanto, como a base de teste real possui `96,60%` de maquinas normais e apenas `3,40%` de falhas (`68` quebras em `2.000` registros), a acuracia global isolada pode mascarar erros na classe minoritaria. Na industria, um **Falso Negativo** (nao detectar uma maquina que vai quebrar) custa muito mais caro do que um **Falso Positivo** (realizar uma inspecao preventiva em uma maquina saudavel). Observando nossas metricas complementares na tabela da Fase 6, nota-se que a Arvore de Decisao com `max_depth=5`, apesar de ter acuracia global ligeiramente menor (`89,95%`), atingiu um **Recall de `82,35%`** na classe de falha, contra `67,65%` em `max_depth=10` e `66,18%` em `max_depth=None`. Em producao real, a selecao do modelo e o ajuste do limiar de probabilidade (*threshold tuning*) devem ser guiados pela maximizacao do **Recall** e do **F1-Score** (ou da metrica $F_\beta$ com $\beta = 2$).

2. **Validacao Cruzada Estratificada (*Stratified K-Fold Cross-Validation*):**
   O pipeline atual avalia os hiperparametros em uma unica divisao fixa `80/20` (`random_state=42`). A implementacao do `StratifiedKFold` (por exemplo, com $k = 5$ ou $k = 10$ dobras) integrado a um `Pipeline` do `imblearn` (aplicando `SMOTE` e `StandardScaler` separadamente dentro de cada dobra de treino) eliminaria a variancia decorrente de uma unica particao e permitiria otimizar hiperparametros de forma sistematica com `GridSearchCV`.

3. **Adoção de Modelos de *Ensemble* (*Random Forest* e *XGBoost*):**
   Uma unica Arvore de Decisao possui alta variancia e tende ao *overfitting* conforme cresce em profundidade (`99,26%` no treino em `max_depth=None`). A evolucao natural da modelagem consiste em testar algoritmos de comite de arvores, como **Random Forest** (*Bagging*, que reduz a variancia combinando centenas de arvores independentes) e **XGBoost / LightGBM** (*Gradient Boosting*, que corrige iterativamente os erros residuais das arvores anteriores e oferece suporte nativo a pesos de classe via `scale_pos_weight`).

4. **Novos Atributos Fisicos na Engenharia de Recursos:**
   Alem da coluna `potencia`, as anotacoes de engenharia indicam a existencia de falhas por dissipacao de calor (`falha_hdf`) e por sobretensao (`falha_osf`). A criacao do diferencial termico (`delta_temperatura_k = temperatura_processo_k - temperatura_ar_k`) e do indice de esforco acumulado (`esforco_acumulado = desgaste_ferramenta_min * torque_nm`) forneceria sinais preditivos ainda mais especificos para separar esses modos de falha.
