# Предобработка данных: Student Performance

Ноутбук с пайплайном предобработки табличных данных для датасета об успеваемости студентов. Цель: подготовить признаки (импутация, масштабирование, кодирование) так, чтобы их можно было подать в модель регрессии, и не допустить утечки данных из тестовой выборки.

## Данные

- Файл: `data/student_performance.csv`
- Размер: 10 000 строк, 6 столбцов, пропусков нет
- Целевая переменная: `Performance Index` (число, регрессия)

| Признак | Тип | Описание |
|---|---|---|
| `Hours Studied` | числовой (int) | часы учёбы |
| `Previous Scores` | числовой (int) | предыдущие оценки |
| `Sleep Hours` | числовой (int) | часы сна |
| `Sample Question Papers Practiced` | числовой (int) | число решённых пробных работ |
| `Extracurricular Activities` | категориальный (Yes / No) | внеучебная активность |
| `Performance Index` | числовой (float) | **целевая переменная** |

## Структура проекта

```
.
├── README.md
├── data/
│   └── student_performance.csv
├── notebooks/
│   └── preprocessing.ipynb      # ноутбук читает данные по пути ../data/
└── venv/                        # виртуальное окружение (не коммитить)
```

## Установка и запуск

Использовалась версия Python 3.14.7.

```bash
python -m venv venv
venv\Scripts\activate            # Windows
pip install pandas numpy scikit-learn jupyter
jupyter notebook
```

Ноутбук нужно запускать из папки `notebooks/`, иначе путь `../data/student_performance.csv` не найдётся. Ячейки выполняются сверху вниз.

## Что делает пайплайн

1. **Загрузка и проверка:** `shape`, `info()`, типы, подсчёт пропусков.
2. **Разделение на X и y:** `y = Performance Index`, остальные 5 столбцов идут в `X`.
3. **Определение типов признаков:** 4 числовых и 1 категориальный (по `select_dtypes`).
4. **Разбиение train/test:** 80/20, `random_state=42` (8000 / 2000 строк).
5. **Числовые признаки:** `SimpleImputer(strategy="median")` и `StandardScaler`.
6. **Категориальный признак:** `SimpleImputer(strategy="constant", fill_value="missing")` и `OneHotEncoder(handle_unknown="ignore")`.
7. **Объединение:** `ColumnTransformer` с ветками `num` и `cat`.
8. **Применение:** `fit_transform` на train, только `transform` на test.
9. **Проверка результата:** размерности, имена признаков, `describe()` преобразованных данных (среднее ≈ 0, std ≈ 1 у числовых).

## Результаты

| Этап | Train | Test |
|---|---|---|
| До предобработки | 8000 × 5 | 2000 × 5 |
| После предобработки | 8000 × 6 | 2000 × 6 |

Число признаков выросло с 5 до 6, потому что `Extracurricular Activities` превратился в два one-hot столбца.

Итоговые признаки:

```
num__Hours Studied
num__Previous Scores
num__Sleep Hours
num__Sample Question Papers Practiced
cat__Extracurricular Activities_No
cat__Extracurricular Activities_Yes
```

## Важные замечания

- **Утечка данных исключена.** Препроцессор обучается только на `X_train`, а на `X_test` применяется `transform`. Не вызывайте `fit_transform` на тестовой выборке.
- **Пропусков в данных нет**, поэтому импутеры сейчас ничего не меняют. Они оставлены, чтобы пайплайн работал на новых данных с пропусками.
- `handle_unknown="ignore"` защищает от ошибки, если в новых данных встретится неизвестная категория.
- Модель в ноутбуке не обучается: работа заканчивается подготовкой признаков.

## Частые ошибки

| Ошибка | Причина и решение |
|---|---|
| `ModuleNotFoundError: No module named 'pandas'` | Библиотеки не установлены в активное окружение. Выполните `pip install pandas numpy scikit-learn`. |
| `KeyError: "['Class'] not found in axis"` | В этом датасете нет столбца `Class`. Целевая переменная называется `Performance Index`. |
| `FileNotFoundError` при `read_csv` | Ноутбук запущен не из папки `notebooks/`. Проверьте путь `../data/student_performance.csv`. |

## Возможное продолжение

- Обучить модель регрессии (например, `LinearRegression` или `RandomForestRegressor`), подключив препроцессор в `Pipeline`.
- Оценить качество по MAE, RMSE и R² на тестовой выборке.
- Подобрать параметры через кросс-валидацию (`GridSearchCV`).
