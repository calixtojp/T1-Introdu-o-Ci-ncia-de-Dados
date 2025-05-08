# Classificação de Fumantes a partir de Bio-sinais

![Python](https://img.shields.io/badge/python-3.8%2B-blue) ![scikit-learn](https://img.shields.io/badge/scikit--learn-1.1-green) ![MIT License](https://img.shields.io/badge/license-MIT-lightgrey)

## Sumário

- [Contexto e Problema](#contexto-e-problema)  
- [Descrição do Dataset](#descrição-do-dataset)  
- [Metodologia](#metodologia)  
  - [Pré-processamento](#pré-processamento)  
  - [Modelagem](#modelagem)  
  - [Validação e Métricas](#validação-e-métricas)  
- [Resultados Principais](#resultados-principais)  
- [Estrutura do Projeto](#estrutura-do-projeto)  
- [Como Executar](#como-executar)  
- [Boas Práticas Adotadas](#boas-práticas-adotadas)  
- [Licença](#licença)  

---

## Contexto e Problema

Determinamos se um indivíduo é fumante ou não a partir de **bio-sinais** básicos de saúde (pressão arterial, colesterol, exames de visão, índice de massa corporal etc.).  
O desafio envolve:

1. **Classificação binária** (fumante vs. não-fumante).  
2. Alto volume de registros (55 692 linhas, 27 colunas).  
3. Desejo de identificar quais sinais têm maior poder preditivo.

---

## Descrição do Dataset

- **Origem**: Kaggle – “Basic Health Bio-Signals”  
- **Formato**: CSV (`smoking.csv`, 55 692 registros × 27 atributos)  
- **Particionamento**:  
  - Treino (`df_train`): 44 553 registros  
  - Teste (`df_test`): 11 139 registros  
- **Colunas principais**:  
  - `gender`, `age` (intervalos de 5 anos), `height`, `weight`, `waist`  
  - Sinais vitais: `systolic`, `relaxation`, `fasting_blood_sugar`, `cholesterol`, `triglyceride`, `HDL`, `LDL`  
  - Enzimas hepáticas: `AST`, `ALT`, `Gtp`  
  - Exames: `eyesight_left`, `eyesight_right`, `hearing_left`, `hearing_right`, `oral`, `dental_caries`, `tartar`  
  - **Target**: `smoking` (0 = não, 1 = sim)  

Alguns passos de **pós-processamento** e **filtragem** foram aplicados (remoção de outliers, preenchimento de dados faltantes).

---

## Metodologia

### Pré-processamento

1. **Tratamento de valores ausentes**  
   - Preenchimento com mediana (variáveis contínuas) ou moda (categóricas).
2. **Codificação de variáveis categóricas**  
   - `OneHotEncoder` para colunas de baixa cardinalidade.  
   - `OrdinalEncoder` em exames com ordem natural (“nenhum”, “leve”, “grave”).
3. **Escalonamento**  
   - `StandardScaler` para normalizar distribuições e facilitar convergência de algoritmos.
4. **Seleção de características**  
   - **Filtro univariado** (Teste qui-quadrado / ANOVA) para reduzir dimensionalidade.  
   - Experimentamos conjuntos de tamanhos entre 70 e 80 atributos selecionados via cross-validation.

### Modelagem

Testamos 10 algoritmos clássicos, todos dentro de pipelines para garantir reprodutibilidade:

| Modelo                              | Biblioteca           |
| ----------------------------------- | -------------------- |
| GaussianNB & BernoulliNB            | `sklearn.naive_bayes`|
| k-Nearest Neighbors (KNN)           | `sklearn.neighbors`  |
| Random Forest                       | `sklearn.ensemble`   |
| Extra Trees                         | `sklearn.ensemble`   |
| Gradient Boosting / HistGB          | `sklearn.ensemble`   |
| XGBoost                              | `xgboost`            |
| LightGBM (GBDT & DART)               | `lightgbm`           |
| CatBoost                             | `catboost`           |

- **Cross-validation estratificada** (10 folds, shuffle + seed fixo).  
- **Tuning** do número de features via variação entre 70–80.  
- Métricas calculadas em cada fold: **ROC AUC** e **Accuracy**.

### Validação e Métricas

- **ROC AUC**: Área sob curva ROC — principal métrica para classes desbalanceadas.  
- **Accuracy**: Percentual de acerto.  
- **Tempo de treinamento**: monitorado para cada experimento (`time.time()`).

---

## Resultados Principais

- **Melhor configuração**:  
  - **Modelo**: `CatBoostClassifier`  
  - **Nº de features**: 77  
- **Desempenho médio (10-fold CV)**:  
  - **ROC AUC**: 0.83  
  - **Accuracy**: 0.75  
- **Tempo médio de treino**: ~9 min

> **Insight**: variáveis relacionadas a exames hepáticos e pressão arterial foram as que mais contribuíram (importância > 0.05 em `feature_importances_`).

---

## Estrutura do Projeto
```
📁 projeto-smoke-classification/
│
├─ data/
│   ├─ smoking.csv
│   ├─ processed/
│
├─ notebooks/
│   └─ Trabalho\_ICD.ipynb
│
├─ src/
│   ├─ data\_preprocessing.py
│   ├─ feature\_selection.py
│   ├─ model\_training.py
│   └─ evaluation.py
│
├─ requirements.txt
├─ README.md
└─ LICENSE
```

## Como Executar

1. **Clonar o repositório**  
   ```bash
   git clone https://github.com/seu-usuario/projeto-smoke-classification.git
   cd projeto-smoke-classification
    ````

2. **Criar ambiente e instalar dependências**

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```
3. **Pré-processar dados**

   ```bash
   python src/data_preprocessing.py --input data/smoking.csv \
       --output data/processed/train.csv data/processed/test.csv
   ```
4. **Treinar e avaliar**

   ```bash
   python src/model_training.py --config configs/experiment.yml
   python src/evaluation.py --predictions results/preds.csv
   ```

---

## Boas Práticas Adotadas

* **Reprodutibilidade**

  * Uso de `random_state` fixo em todas as rotinas.
  * Pipelines de pré-processamento e modelagem no scikit-learn.
* **Organização de Código**

  * Separação clara entre ETL, seleção de features, treinamento e avaliação.
  * Scripts parametrizáveis via linha de comando (`argparse`).
* **Controle de Versão**

  * `.gitignore` configurado para ignorar dados brutos e ambientes virtuais.
* **Documentação**

  * Comentários e docstrings em todos os módulos.
  * Este `README.md` contém instruções de uso e visão geral.
* **Teste de Qualidade**

  * Linters: `flake8`, `black`.
  * Validação de esquemas (pandas-schema) para checar integridade dos dados.

---

## Licença

Este projeto está licenciado sob a [MIT License](LICENSE).

```
```
