---
title: AI_Arrow
date: 2024-07-05
external_link: 
tags:
  - AI
  - Python
  - AI_Arrow_Bootcamp
---

Летом я участвовал в интенсиве по машинному обучению AI_Arrow Middle track. Сегодня я расскажу об одном проекте, в котором по имеющимся данным нужно предсказать, закроет ли пользователь свой аккаунт в банке или нет.

## Начало

Рассмотрим метрики для трёх основных задач машинного обучения:

- Бинарная классификация
- Многоклассовая классификация
- Регрессия

Для каждой из задач мы рассмотрим "игрушечный пример" с kaggle соревнований, где мы построим несколько моделей и сравним их по основным метриками.

Для начала давайте установим необходимые библиотеки и импортируем все нужные модули.

```python
!pip install catboost
```

```python
# Импорт библиотек
import warnings
warnings.simplefilter(action='ignore', category=FutureWarning)

import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.preprocessing import LabelEncoder
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, roc_auc_score, confusion_matrix, roc_curve
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from catboost import CatBoostClassifier

from sklearn.linear_model import LinearRegression
from sklearn.ensemble import RandomForestRegressor
from catboost import CatBoostRegressor
```
## Бинарная классификация

<img src="1.jpg" alt="drawing" width=75%/>

### Задача

А первая задача, которую мы рассмотрим - это задача бинарной классификации (https://www.kaggle.com/competitions/playground-series-s4e1/overview) - по имеющимся данным предсказать, закроет ли пользователь свой аккаунт в банке или нет.

Наш план будет следующим:

- Посмотрим на данные, посчитаем некоторые статистики
- Построим модели:
    - LogReg
    - Random Forest
    - Catoost

И сравним качество работы данных моделей.

### Загрузка данных

```python
# загрузим наши данные
train_data = pd.read_csv('train.csv')    # train data
test_data = pd.read_csv('test.csv')      # test data

# посмотрим на train данные
train_data.head()
```

```python
# посмотрим на test данные
test_data.head()
```

То есть наши данные содержат следующую инфорацию о пользователях:

- Customer ID: уникальный ID для пользователя
- Surname: Фамилия/имя пользователя
- Credit Score: кредитный счёт (балл/оценка?)
- Geography: страна резиденства пользователя
- Gender: гендер пользователя
- Age: возраст пользователя
- Tenure: сколько лет пользователь пользуется банковскими услугами
- Balance: баланс пользователя
- NumOfProducts: количество разных продуктов банка у пользователя (таких как сберегательный счёт, кредитная карта и др.)
- HasCrCard: есть ли у пользователя кредитная карта
- IsActiveMember: активный ли пользователь банка
- EstimatedSalary: зарплата пользователя

В результате мы хотим предсказать:
- Exited: Whether the customer has churned (1 = yes, 0 = no)

```python
# Посмотрим, сколько у нас данных

num_train_rows, num_train_columns = train_data.shape

num_test_rows, num_test_columns = test_data.shape

print("Training Data:")
print(f"Количество строк: {num_train_rows}")
print(f"Количество столбцов: {num_train_columns}\n")

print("Test Data:")
print(f"Количество строк: {num_test_rows}")
print(f"Количество столбцов: {num_test_columns}\n")
```
Посмотрим на каждую из колонок, есть ли там пропуски или Nans:

```python
train_data.info()
```

```python
test_data.info()
```

Проверим, есть ли в данных дупликаты:

```python
# Count duplicate rows in train_data
train_duplicates = train_data.duplicated().sum()

# Count duplicate rows in test_data
test_duplicates = test_data.duplicated().sum()

# Print the results
print(f"Количество дупликатов в тренировочных данных: {train_duplicates}")
print(f"Количество дупликатов в тестовых данных: {test_duplicates}")
```

Таким образом, нам повезло - данные не содержат пропусков, дупликатов и нанов! Можем с ними работать!

## Визуализация данных

Напомним, что чаще всего лучше посмотреть на то, что имеется, прежде чем с этим работать. На примере задачи бинарной классификации мы напомним, как можно визуализировать имеющиеся данные.

Прежде чем строить модели, немного повизуализируем то, что у нас есть.

### Наш таргет

Сначала посмотрим на наш таргет.

```python
palette = ['indigo', 'orangered']

sns.countplot(x="Exited", data=train_data, palette=palette)

plt.show()
```

Стоит запомнить, что у нас несбалансированная выборка, где элементов одного класса значительно больше другого!

### Категориальные признаки

Теперь визуализируем категориальные признаки.

```python
import seaborn as sns

cat_columns = ['Gender', 'Geography', 'HasCrCard', 'IsActiveMember' ]
palette = ['indigo', 'orangered', "deeppink", "gold"]

fig = plt.figure(figsize=(14, 14))

for i in range(4):
    plt.subplot(4 // 2 + 4 % 2, 2, i+1)
    sns.countplot(x=cat_columns[i], data=train_data, palette=palette)


plt.tight_layout()
plt.show()
```

```python
sns.countplot(x="Tenure", data=train_data, palette=palette)


plt.tight_layout()
plt.show()
```

### Численные признаки

Теперь перейдём к численным признакам.

```python
num_columns = []
num_columns.append("Balance")


fig= plt.figure(figsize=(10, 6))


histplot = sns.histplot(data=train_data, x="Balance", bins=20, color='indigo', kde=True)

# Calculate mean and median
mean_value = train_data["Balance"].mean()
median_value = train_data["Balance"].median()

# Add mean and median lines
plt.axvline(mean_value, color='red', linestyle='dashed', linewidth=3, label=f'Mean: {mean_value:.2f}')
plt.axvline(median_value, color='blue', linestyle='dashed', linewidth=3, label=f'Median: {median_value:.2f}')

# Set labels and title
plt.title("Distribution of Balance in df_train with Mean and Median")
plt.xlabel("Balance")
plt.ylabel("Count")

# Show legend
plt.legend()
```

```python
num_columns.append("EstimatedSalary")


fig= plt.figure(figsize=(10, 6))


histplot = sns.histplot(data=train_data, x="EstimatedSalary", bins=20, color='indigo', kde=True)

# Calculate mean and median
mean_value = train_data["EstimatedSalary"].mean()
median_value = train_data["EstimatedSalary"].median()

# Add mean and median lines
plt.axvline(mean_value, color='red', linestyle='dashed', linewidth=3, label=f'Mean: {mean_value:.2f}')
plt.axvline(median_value, color='blue', linestyle='dashed', linewidth=3, label=f'Median: {median_value:.2f}')

# Set labels and title
plt.title("Distribution of Balance in df_train with Mean and Median")
plt.xlabel("EstimatedSalary")
plt.ylabel("Count")

# Show legend
plt.legend()
```

```python
num_columns.append("CreditScore")


fig= plt.figure(figsize=(10, 6))


histplot = sns.histplot(data=train_data, x="CreditScore", bins=20, color='indigo', kde=True)

# Calculate mean and median
mean_value = train_data["CreditScore"].mean()
median_value = train_data["CreditScore"].median()

# Add mean and median lines
plt.axvline(mean_value, color='red', linestyle='dashed', linewidth=3, label=f'Mean: {mean_value:.2f}')
plt.axvline(median_value, color='blue', linestyle='dashed', linewidth=3, label=f'Median: {median_value:.2f}')

# Set labels and title
plt.title("Distribution of Balance in df_train with Mean and Median")
plt.xlabel("CreditScore")
plt.ylabel("Count")

# Show legend
plt.legend()
```

```python
num_columns.append("Age")


fig= plt.figure(figsize=(10, 6))


histplot = sns.histplot(data=train_data, x="Age", bins=20, color='indigo', kde=True)

# Calculate mean and median
mean_value = train_data["Age"].mean()
median_value = train_data["Age"].median()

# Add mean and median lines
plt.axvline(mean_value, color='red', linestyle='dashed', linewidth=3, label=f'Mean: {mean_value:.2f}')
plt.axvline(median_value, color='blue', linestyle='dashed', linewidth=3, label=f'Median: {median_value:.2f}')

# Set labels and title
plt.title("Distribution of Balance in df_train with Mean and Median")
plt.xlabel("Age")
plt.ylabel("Count")

# Show legend
plt.legend()
```

```python
train_data.head()
```

```python
num_colums = ['CreditScore','Balance','EstimatedSalary','Age','NumOfProducts']
cat_columns = ['Geography', 'Gender', "Tenure", "HasCrCard", "IsActiveMember" ]
```

### Категориальные фичи

Категориальные фичи обрабатывааются с помощью LabelEncoder, который кодирует категориальные признаки, присваивая каждой уникальной категории числовое значение.

Как работает LabelEncoder?

LabelEncoder присваивает каждой уникальной категории числовую метку (label). Процесс преобразования состоит из двух этапов:

Обучение (fit):

LabelEncoder анализирует данные и выявляет все уникальные категории, создавая для них числовые метки.

Например, для категорий ['cat', 'dog', 'fish'] метки могут быть присвоены следующим образом:

cat → 0

dog → 1

fish → 2

Преобразование (transform):

После обучения LabelEncoder может преобразовать исходные категориальные данные в их числовые метки.

Например, для списка ['dog', 'cat', 'dog', 'fish'] он преобразует его в [1, 0, 1, 2].

Давайте посмотрим, как он работает на нашем примере.

```python
# рассмотрим, например, работу нашего энкодера на колонке гендер.

gender_column = train_data['Gender']

print(f"В данной колонке находятся значения {gender_column.unique()}")

# теперь закодируем данную колонку с помощью LabelEncoder
enc = LabelEncoder()
gender_column_encoded = enc.fit_transform(gender_column)

print(f"В данной колонке находятся значения {np.unique(gender_column_encoded)}")
```

```python
# from sklearn.preprocessing import LabelEncoder

enc = LabelEncoder()
for cat_feat in cat_columns:
    train_data[cat_feat] = enc.fit_transform(train_data[cat_feat])
    test_data[cat_feat] = enc.transform(test_data[cat_feat])


train_data.head()
```
### Численные фичи

Для обработки численных признаков мы будем использовать StandardScaler из библиотеки sklearn.

StandardScaler вычисляет среднее значение и стандартное отклонение по каждому признаку в обучающем наборе данных, а затем использует эти значения для преобразования данных. Каждый признак (столбец) в наборе данных центрируется (среднее значение становится равным 0) и масштабируется (стандартное отклонение становится равным 1).

```python
from sklearn.preprocessing import StandardScaler

for col in num_colums:
    sc = StandardScaler()
    train_data[col] = sc.fit_transform(train_data[[col]])
    test_data[col] = sc.transform(test_data[[col]])

train_data.head()
```

Теперь дропнем некоторые колоночки за ненадобностю на данном этапе :)

```python
!pip install optuna
```

```python
from catboost import Pool, cv
import optuna
from catboost import CatBoostClassifier
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, log_loss

# Загрузка данных
X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Определение целевой функции
def objective(trial):
    params = {
        'iterations': trial.suggest_int('iterations', 100, 500),
        'depth': trial.suggest_int('depth', 1, 5),
        'learning_rate': trial.suggest_loguniform('learning_rate', 1e-4, 1e-2),
        'l2_leaf_reg': trial.suggest_loguniform('l2_leaf_reg', 1, 10),
        'task_type': 'CPU',
        "loss_function":'MultiClass',
        "bootstrap_type":'Bernoulli',  # Использование Bagging
        "subsample":0.8  # Доля выборки для каждого дерева
    }

    model = CatBoostClassifier(**params)
    model.fit(X_train, y_train, eval_set=(X_test, y_test), early_stopping_rounds=50, verbose=False)
    y_pred = model.predict(X_test)
    accuracy = accuracy_score(y_test, y_pred)
    logloss = log_loss(y_test, model.predict_proba(X_test))

    return logloss

# Создание и запуск исследования
study = optuna.create_study(direction='minimize')
study.optimize(objective, n_trials=70)

# Вывод лучших параметров
print(f"Лучшие параметры: {study.best_params}")
print(f"Лучший результат: {study.best_value}")
```

```python
from sklearn.metrics import roc_auc_score
model = CatBoostClassifier(iterations= 497,
                           depth= 5,
                           learning_rate= 0.008135970021604337,
                           l2_leaf_reg= 1.0083902808198197,
                           early_stopping_rounds=50,
                           verbose=False,
                           loss_function ='MultiClass',
                           bootstrap_type='Bernoulli',
                           subsample=0.8 )
model.fit(X_train,y_train)
y_pred = model.predict(X_test)
print(roc_auc_score(y_test,y_pred,multi_class='ovr'))
```

```python
train_data.drop(['id', 'CustomerId', 'Surname'], inplace=True, axis=1)
```

```python
train_data.head()
```

```python
X_train = train_data.iloc[:, :-1].values
y_train = train_data.iloc[:, -1].values

# Разделение данных на обучающую и тестовую выборки
X_train, X_val, y_train, y_val = train_test_split(X_train, y_train, test_size=0.2, random_state=42)
```

## Выводы

Метрики в машинном обучении играют ключевую роль в оценке качества моделей и их производительности. Для разных типов задач применяются различные метрики, каждая из которых имеет свои особенности и области применения.

В данном вебинаре мы с вами рассмотрели задачи бинарной и многоклассовой классификации, а также регрессии. Давайте коротко вспомним, какие метрики можно использовать в данных задачах.

В задачах бинарной классификации наиболее часто используемые метрики включают:

Accuracy (Точность): доля правильно предсказанных классов среди всех наблюдений.

Error Rate (Ошибка): доля неправильных предсказаний.

Precision (Точность): доля истинно положительных предсказаний среди всех предсказанных положительных.

Recall (Полнота): доля истинно положительных предсказаний среди всех истинных положительных.

F1 Score: метрика учитывает и Precision и Recall.

ROC-AUC: площадь под кривой ошибок, показывающая соотношение между True Positive Rate и False Positive Rate.

