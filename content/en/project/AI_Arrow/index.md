---
title: AI_Arrow
date: 2024-07-05
external_link: 
tags:
  - AI
  - Python
  - AI_Arrow_Bootcamp
---

In the summer, I participated in the AI_Arrow Middle track machine learning intensive. Today I will talk about one project in which, according to available data, it is necessary to specify whether the user will close his bank account or not.

## The beginning

Let's look at metrics for three main machine learning tasks:

- Binary classification
- Multiclass classification
- Regression

For each of the tasks, we will consider a "toy example" from kaggle competitions, where we will build several models and compare them by basic metrics.

First, let's install the necessary libraries and import all the necessary modules.

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
## Binary classification

<img src="1.jpg" alt="drawing" width=75%/>

### Task

And the first task that we will consider is the binary classification problem (https://www.kaggle.com/competitions/playground-series-s4e1/overview ) - according to available data, predict whether the user will close his bank account or not.

Our plan will be as follows:

- Let's look at the data, calculate some statistics
- Let's build models:
- LogReg
- Random Forest
- Catoost

And let's compare the quality of these models.

### Uploading data

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

That is, our data contains the following information about users:

- Customer ID: a unique identifier for the user
- Last name: Amilia/I am a user
- Credit rating: credit score (score/oenca?)
- Geography: the user's country of origin
- Gender: male user
- Age: the age of the user
- Term of office: has been using banking services for several years
- Balance: User's balance
- Number of products: the number of products for the bank and users (as well as a separation account, credit card, etc.)
- Bank card: You are a credit card user
- Is an active participant: an active user of the bank
- Estimated salary: the user's salary

As a result, we want to predict:
- Completed: Has the client been shot down (1 = yes, 0 = no)

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
Let's look at each of the columns to see if there are gaps or Nuns:

```python
train_data.info()
```

```python
test_data.info()
```

Let's check if there are duplicates in the data:

```python
# Count duplicate rows in train_data
train_duplicates = train_data.duplicated().sum()

# Count duplicate rows in test_data
test_duplicates = test_data.duplicated().sum()

# Print the results
print(f"Количество дупликатов в тренировочных данных: {train_duplicates}")
print(f"Количество дупликатов в тестовых данных: {test_duplicates}")
```

Thus, we were lucky - the data does not contain omissions, duplicates and is new! We can work with them!

## Data visualization

Recall that it is often better to look at what is available before working with it. Using the example of a binary classification problem, we will remind you how to visualize the available data.

Before building models, let's visualize a little what we have.

### Our target

First, let's look at our target.

```python
palette = ['indigo', 'orangered']

sns.countplot(x="Exited", data=train_data, palette=palette)

plt.show()
```

It is worth remembering that we have an unbalanced sample, where the elements of one class are much larger than the other!

### Categorical features

Now we visualize the categorical features.

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

### Numerical signs

Now let's move on to the numerical features.

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

### Categorical features

Categorical features are processed using LabelEncoder, which encodes categorical features by assigning a numeric value to each unique category.

How does LabelEncoder work?

LabelEncoder assigns a numeric label to each unique category. The conversion process consists of two stages:

Training (fit):

LabelEncoder analyzes the data and identifies all unique categories by creating numeric labels for them.

For example, for the categories ['cat', 'dog', 'fish'], labels can be assigned as follows:

cat → 0

dog → 1

fish → 2

Transformation (transform):

After training, the LabelEncoder can convert the original categorical data into their numeric labels.

For example, for a list ['dog', 'cat', 'dog', 'fish'], it converts it to [1, 0, 1, 2].

Let's see how it works in our example.

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
### Numerical features

To process numerical features, we will use the Standard Scale r from the sklearn library.

StandardScaler calculates the mean and standard deviation for each feature in the training dataset, and then uses these values to transform the data. Each feature (column) in the dataset is centered (the average value becomes 0) and scaled (the standard deviation becomes 1).

```python
from sklearn.preprocessing import StandardScaler

for col in num_colums:
    sc = StandardScaler()
    train_data[col] = sc.fit_transform(train_data[[col]])
    test_data[col] = sc.transform(test_data[[col]])

train_data.head()
```

Now let's drop some columns as unnecessary at this stage :)

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

## Summary

Metrics in machine learning play a key role in evaluating the quality of models and their performance. Different metrics are used for different types of tasks, each of which has its own specifics and scope.

In this webinar, we discussed the issues of binary and multiclass classification, as well as regression. Let's briefly look at what metrics can be used in these tasks.

In the makings of binary classification, a larger number of metrics used include:

Accuracy: the proportion of correctly represented classes among all observations.

Error Rate: The percentage of incorrect assumptions.

Precision: the proportion of referentially complete representations among all submitted complete ones.

Recall (Completeness): the proportion of referentially complete prepositions among all referentially complete ones.

F1 Score: The metric teaches both Precision and Recall.

ROC-AUC: The area under the error curve showing the ratio between True Positive Rate and False Positive Rate.

