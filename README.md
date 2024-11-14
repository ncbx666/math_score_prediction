f# Студенческая Успеваемость: Прогнозирование Оценок по Математике

Данный проект представляет собой нейронную сеть для прогнозирования оценки по математике на основе различных характеристик учеников. Нейронная сеть обучается на датасете, содержащем демографические и академические данные.

## Данные

Датасет состоит из 8 столбцов с такими характеристиками:

- **гендер** — пол ученика (метод энкодинга OHE).
- **раса** — расовая принадлежность (OHE).
- **образование родителей** — уровень образования родителей (LabelEncoding).
- **ланч** — тип ланча (LabelEncoding).
- **подготовка к экзамену** — проходил ли ученик курс подготовки к экзамену (0 — не проходил, 1 — проходил).
- **оценка по чтению** — баллы за чтение.
- **оценка по письму** — баллы за письмо.

Целевой признак: **оценка по математике** — значение, которое сеть пытается предсказать.

## Предобработка данных

Все строковые признаки преобразованы в численные:

- Признаки **гендер** и **раса** закодированы с помощью One-Hot Encoding (OHE).
- Признаки **образование родителей** и **ланч** преобразованы с помощью Label Encoding.
- Признак **подготовка к экзамену** представлен в бинарной форме (0 и 1).

## Архитектура модели

Модель построена на основе нейронной сети с использованием библиотеки `torch.nn`. Функция `create_model()` принимает три параметра:

- **hid_size1** — размер первого скрытого слоя (по умолчанию 32).
- **n_features** — количество входных признаков (по умолчанию совпадает с количеством признаков после предобработки).
- **n_out** — количество выходных признаков (один — оценка по математике).

```python
import torch.nn as nn

def create_model(hid_size1=32, n_features=NUM_FEATURES, n_out=NUM_OUT):
    model = nn.Sequential()
    model.add_module('l1', nn.Linear(n_features, hid_size1))
    model.add_module('activation1', nn.ReLU())
    model.add_module('l2', nn.Linear(hid_size1, n_out))
    return model
```
## Обучение модели

Модель обучается с использованием стохастического градиентного спуска `SGD` и функцией потерь `MSELoss`. Обучение проходит по эпохам, где каждую итерацию берутся случайные батчи данных для улучшения обобщаемости модели.

### Код функции обучения

```python
import torch
import numpy as np

def train_model(model, x_train, y_train):
    loss_function = nn.MSELoss()
    opt = torch.optim.SGD(model.parameters(), lr=0.00001)

    loss_history = []
    for num_of_epoch in range(NUM_EPOCH):
        opt.zero_grad()

        # Выбор случайного батча из обучающих данных
        random_indices = np.random.choice(np.arange(len(x_train)), size=32)
        x_train_batch = x_train[random_indices]
        y_train_batch = y_train[random_indices]

        # Прогноз и вычисление ошибки
        y_predicted = model(x_train_batch).view(-1)
        loss = loss_function(y_predicted, y_train_batch)

        loss_history.append(loss.item())

        # Обновление параметров модели
        loss.backward()
        opt.step()

  return model, loss_history
```
## Оценка модели

После обучения модели оценивается средняя квадратичная ошибка 'MSELoss' на обучающей и тестовой выборках. Это позволяет проверить, насколько хорошо модель обучилась на данных и насколько она обобщается на новые данные.

### Код для оценки MSE

```python
def calculate_mse(model, x_data, y_data):
    model.eval()  #перевели модель в режим оценки
    with torch.no_grad():  #отключаем вычисление градиентов
        y_predicted = model(x_data).view(-1)
        mse = F.mse_loss(y_predicted, y_data)
    return mse.item()

train_mse = calculate_mse(trained_model, x_train, y_train)
test_mse = calculate_mse(trained_model, x_test, y_test)

```

## Результаты
После запуска оценки на обучающей и тестовой выборках были получены следующие значения MSE:

**Training MSE:** 29.875001907348633

**Testing MSE:** 26.76482580292285

Низкое значение тестовой ошибки по сравнению с обучающей ошибкой указывает на хорошую обобщаемость модели и на то, что модель не переобучилась.

Также был построен график зависимости значения функции потерь и эпохи обучения:
![image](https://github.com/user-attachments/assets/b3824784-e0c4-48a1-a76d-eb9b5643bec8)

