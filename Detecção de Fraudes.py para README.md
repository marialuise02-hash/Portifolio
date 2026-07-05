# 🛡️ Detecção de Fraude em Cartões de Crédito com IA

Este projeto foi desenvolvido como parte do meu aprendizado no **Bootcamp Afya - Automação de Dados com IA**, na plataforma DIO. O objetivo principal é construir e avaliar modelos de Machine Learning capazes de identificar transações fraudulentas em cartões de crédito, lidando com o desafio de bases de dados altamente desbalanceadas. O código foi feito 100% pelas aulas da tutora.

---

## 📊 1. Carga dos Dados e Pré-processamento
Nesta primeira etapa, o projeto faz o download da base de dados pública do TensorFlow e realiza o tratamento inicial. Como os valores das transações (`Amount`) variam muito, aplicamos uma transformação logística e normalização para que o modelo consiga aprender melhor sem se confundir com valores discrepantes.

```python

import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

url = '[https://storage.googleapis.com/download.tensorflow.org/data/creditcard.csv](https://storage.googleapis.com/download.tensorflow.org/data/creditcard.csv)'
df = pd.read_csv(url)

df.head()
df['Class'].value_counts(normalize=True)

df['Amount_log'] = np.log1p(df['Amount'])
scaler = StandardScaler()
df['Amount_scaled'] = scaler.fit_transform(df[['Amount']])

x = df.drop('Class', axis=1)
y = df['Class']

x_train, x_test, y_train, y_test = train_test_split(
    x, y, stratify=y, test_size=0.3, random_state=42
)

from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, roc_curve, roc_auc_score, precision_recall_curve
import matplotlib.pyplot as plt

model = LogisticRegression(max_iter=1000)
model.fit(x_train, y_train)
y_pred = model.predict(x_test)

print(classification_report(y_test, y_pred))

y_probs = model.predict_proba(x_test)[:,1]
fpr, tpr, _ = roc_curve(y_test, y_probs)
plt.plot(fpr, tpr)
plt.title('ROC Curve')
plt.xlabel('False positive rate')
plt.ylabel('True positive rate')
plt.show()
print('AUC:', roc_auc_score(y_test, y_probs))

precision, recall, _ = precision_recall_curve(y_test, y_probs)
plt.plot(recall, precision)
plt.title('Precision-Recall Curve')
plt.xlabel('Recall')
plt.ylabel('Precision')
plt.show()

```
### 2. Balanceamento de Dados (SMOTE)
Nesta segunda etapa, o projeto aborda o problema do desbalanceamento extremo na base de dados, onde o número de fraudes é muito menor que o de transações normais. Utilizamos a técnica SMOTE para gerar dados sintéticos da classe minoritária, equalizando as categorias para que os modelos não fiquem tendenciosos e consigam aprender a detectar os padrões de fraude de forma eficaz.

```python
from imblearn.over_sampling import SMOTE

smote = SMOTE()
x_res, y_res = smote.fit_resample(x, y)

```
### 3. Treinamento dos Modelos (Baseline vs Avançados)
Nesta terceira etapa, o projeto realiza o treinamento e a avaliação dos modelos preditivos para identificar qual algoritmo entrega o melhor desempenho. Criamos uma Regressão Logística como nosso modelo inicial (Baseline) e analisamos seu comportamento através de relatórios de métricas e curvas de performance, avançando em seguida para modelos mais robustos baseados em árvores de decisão, como Random Forest e XGBoost, além de aplicar pipelines de automação e ajustes personalizados de limiar de decisão.

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.pipeline import Pipeline
from xgboost import XGBClassifier

rf = RandomForestClassifier(
    n_estimators=50,
    max_depth=10,
    class_weight='balanced',
    n_jobs=-1,
    random_state=42
)
rf.fit(x_train, y_train)
y_pred_rf = rf.predict(x_test)
print(classification_report(y_test, y_pred_rf))

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression(max_iter=1000))
])
pipeline.fit(x_train, y_train)

threshold = 0.3
y_pred_custom = (y_probs > threshold).astype(int)
print(classification_report(y_test, y_pred_custom))

xgb = XGBClassifier(
    scale_pos_weight=10,
    use_label_encoder=False,
    eval_metric='logloss'
)
xgb.fit(x_train, y_train)
y_pred_xgb = xgb.predict(x_test)
print(classification_report(y_test, y_pred_xgb))

```

##🛠️ Tecnologias Utilizadas

- Python
- Pandas e NumPy (Manipulação de dados)
- Scikit-Learn (Machine Learning e Métricas)
- Imbalanced-Learn (SMOTE) (Tratamento de dados)
- XGBoost (Algoritmo de alta performance)
- Matplotlib (Visualização de dados)
