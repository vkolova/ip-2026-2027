# M02 — Question Bank


## Предпоставка


- Завършен M00 — Initial Project Setup
- Завършен M01 — Authentication and Users
- Работещ Django проект
- Работещо приложение `accounts`
- Успешно приложени migrations


## Цели


- Django приложение `questions`
- Категории въпроси
- Въпроси с избираем отговор
- Въпроси с числов отговор
- Отделни модели за двата вида въпроси
- Управление на въпросите през Django Admin
- Начална банка с въпроси чрез Django fixture
- Backend тестове за моделите и fixture данните


## Основни понятия


- Model inheritance
- Abstract base model
- Foreign key
- Related objects
- Model validation
- Django Admin
- Fixture
- `loaddata`


## Структура


```text
backend/
├── questions/
│   ├── fixtures/
│   │   └── questions/
│   │       └── question_bank.json
│   ├── migrations/
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── test_models.py
│   │   └── test_fixtures.py
│   ├── admin.py
│   ├── apps.py
│   └── models.py
└── manage.py
```


## Модели


### `Category`


| Поле | Тип | Изисквания |
|---|---|---|
| `name` | `CharField` | Задължително, уникално |


### `BaseQuestion`


Abstract модел с общите полета на двата вида въпроси


| Поле | Тип | Изисквания |
|---|---|---|
| `category` | `ForeignKey` | Връзка към `Category`, `on_delete=models.PROTECT` |
| `text` | `TextField` | Задължително |


```python
class BaseQuestion(models.Model):
    category = models.ForeignKey(
        Category,
        on_delete=models.PROTECT,
        related_name="%(class)ss",
    )
    text = models.TextField()


    class Meta:
        abstract = True
```


### `ChoiceQuestion`


- Самостоятелен concrete модел
- Наследяване на `category` и `text` от `BaseQuestion`
- Въпрос с четири възможни отговора
- Точно един правилен отговор


```python
class ChoiceQuestion(BaseQuestion):
    pass
```


### `NumericQuestion`


- Самостоятелен concrete модел
- Наследяване на `category` и `text` от `BaseQuestion`
- Въпрос с числов отговор


| Поле | Тип | Изисквания |
|---|---|---|
| `correct_answer` | `IntegerField` | Задължително |


```python
class NumericQuestion(BaseQuestion):
    correct_answer = models.IntegerField()
```


### `AnswerOption`


| Поле | Тип | Изисквания |
|---|---|---|
| `question` | `ForeignKey` | Връзка към `ChoiceQuestion`, `on_delete=models.CASCADE` |
| `text` | `CharField` | Задължително |
| `is_correct` | `BooleanField` | Стойност по подразбиране `False` |


## Database структура


- `BaseQuestion` е abstract модел и няма собствена database таблица
- `ChoiceQuestion` и `NumericQuestion` се съхраняват в отделни таблици
- `correct_answer` съществува само в `NumericQuestion`
- `AnswerOption` се свързва само с `ChoiceQuestion`


## Връзки


```text
Category
├── ChoiceQuestion
│   └── AnswerOption
└── NumericQuestion
```


- Една категория → много choice въпроси
- Една категория → много numeric въпроси
- Един choice въпрос → точно четири отговора
- Един choice въпрос → точно един правилен отговор
- Numeric въпрос → един задължителен `correct_answer`


## Model validation


### `ChoiceQuestion`


- Точно четири свързани `AnswerOption` обекта
- Точно един `AnswerOption` с `is_correct=True`


### `NumericQuestion`


- Задължителен целочислен `correct_answer`


### `Category`


- Уникално име


## Django Admin


### `CategoryAdmin`


- Списък с категории
- Търсене по име


### `ChoiceQuestionAdmin`


- Текст на въпроса
- Категория
- Филтър по категория
- Търсене по текст
- `AnswerOption` inline


### `NumericQuestionAdmin`


- Текст на въпроса
- Категория
- Правилен отговор
- Филтър по категория
- Търсене по текст


## Initial fixture


### Местоположение


```text
backend/questions/fixtures/questions/question_bank.json
```


### Съдържание


- 6 категории
- 12 choice въпроса
- 48 answer options
- 12 numeric въпроса
- Въпроси и отговори на български език


### Зареждане


```bash
python backend/manage.py loaddata questions/question_bank.json
```


### Проверка


```bash
python backend/manage.py shell
```


```python
from questions.models import Category, ChoiceQuestion, NumericQuestion


Category.objects.count()
ChoiceQuestion.objects.count()
NumericQuestion.objects.count()
```


Очаквани стойности:


```text
6
12
12
```


## Backend тестове


### Задължителни сценарии


- [ ] Създаване на категория
  - Валидно име
  - Уникално име


- [ ] Създаване на choice въпрос
  - Категория
  - Текст
  - Четири отговора
  - Точно един правилен отговор


- [ ] Невалиден choice въпрос
  - По-малко или повече от четири отговора
  - Няма правилен отговор
  - Повече от един правилен отговор


- [ ] Създаване на numeric въпрос
  - Категория
  - Текст
  - Правилен числов отговор


- [ ] Изтриване на choice въпрос
  - Изтриване на свързаните answer options


- [ ] Защитена категория
  - Категория с въпроси не може да бъде изтрита


- [ ] Зареждане на fixture
  - 6 категории
  - 12 choice въпроса
  - 48 answer options
  - 12 numeric въпроса


- [ ] Валидност на fixture данните
  - Всеки choice въпрос има четири отговора
  - Всеки choice въпрос има точно един правилен отговор
  - Всеки numeric въпрос има правилен числов отговор


## Acceptance criteria


### Models


- [ ] `Category` модел
- [ ] Abstract `BaseQuestion` модел
- [ ] `ChoiceQuestion` модел
- [ ] `NumericQuestion` модел
- [ ] `AnswerOption` модел
- [ ] Успешни migrations


### Question bank


- [ ] Поне 6 категории
- [ ] Поне 12 choice въпроса
- [ ] Поне 12 numeric въпроса
- [ ] Точно четири отговора за всеки choice въпрос
- [ ] Точно един правилен отговор за всеки choice въпрос
- [ ] Правилен числов отговор за всеки numeric въпрос


### Django Admin


- [ ] Управление на категориите
- [ ] Управление на choice въпросите
- [ ] Управление на numeric въпросите
- [ ] Answer options inline
- [ ] Филтриране по категория
- [ ] Търсене по текст


### Fixture


- [ ] `question_bank.json`
- [ ] Валиден JSON
- [ ] Зареждане без грешки чрез `loaddata`
- [ ] Въпроси на български език
- [ ] Коректни връзки между записите


### Tests


- [ ] Category tests
- [ ] Choice question tests
- [ ] Numeric question tests
- [ ] Answer option tests
- [ ] Fixture tests


## Definition of Done


- [ ] Всички модели са имплементирани
- [ ] Всички migrations са commit-нати
- [ ] Django Admin работи за всички модели
- [ ] Fixture файлът е commit-нат
- [ ] Fixture файлът се зарежда без грешки
- [ ] Всички backend тестове минават успешно
- [ ] README съдържа командата за зареждане на fixture


## Предаване


- Код в основното repository
- Milestone tag: `m2-question-bank`
- Commit-нати migrations
- Commit-нат fixture файл
- Успешни backend тестове


```bash
python backend/manage.py test questions
python backend/manage.py loaddata questions/question_bank.json
git tag -a m2-question-bank -m "Milestone 2: Question Bank"
git push origin m2-question-bank
```


## Ресурси


- [Django — Models](https://docs.djangoproject.com/en/5.2/topics/db/models/)
- [Django — Model inheritance](https://docs.djangoproject.com/en/5.2/topics/db/models/#model-inheritance)
- [Django — Model field reference](https://docs.djangoproject.com/en/5.2/ref/models/fields/)
- [Django — Model validation](https://docs.djangoproject.com/en/5.2/ref/models/instances/#validating-objects)
- [Django — Admin site](https://docs.djangoproject.com/en/5.2/ref/contrib/admin/)
- [Django — Initial data](https://docs.djangoproject.com/en/5.2/howto/initial-data/)
- [Django — Fixtures](https://docs.djangoproject.com/en/5.2/topics/db/fixtures/)
- [Django — Testing](https://docs.djangoproject.com/en/5.2/topics/testing/)
