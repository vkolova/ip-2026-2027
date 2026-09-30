# M03 — Games and Rounds


## Предпоставка


- Завършен M00 — Initial Project Setup
- Завършен M01 — Authentication and Users
- Завършен M02 — Question Bank
- Работещ Django проект
- Работещи приложения `accounts` и `questions`
- Успешно приложени migrations


## Цели


- Django приложение `games`
- Модели за игра
- Модели за участници в игра
- Модели за рундове
- Модели за отговори в рунд
- Структура за бъдеща game logic
- Управление през Django Admin
- Backend тестове за моделите и връзките


## Основни понятия


- Domain model
- Foreign key
- One-to-many relationship
- Unique constraint
- Status field
- Ordering
- Related name
- Data integrity
- Round lifecycle


## Структура


```text
backend/
├── games/
│   ├── migrations/
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── test_models.py
│   │   └── test_constraints.py
│   ├── admin.py
│   ├── apps.py
│   └── models.py
└── manage.py
```


## Модели


### `Game`


| Поле | Тип | Изисквания |
|---|---|---|
| `created_by` | `ForeignKey` | Връзка към `settings.AUTH_USER_MODEL`, `on_delete=models.PROTECT` |
| `status` | `CharField` | Задължително, choices |
| `created_at` | `DateTimeField` | `auto_now_add=True` |
| `started_at` | `DateTimeField` | `null=True`, `blank=True` |
| `finished_at` | `DateTimeField` | `null=True`, `blank=True` |


### `Game status`


```python
WAITING = "waiting"
IN_PROGRESS = "in_progress"
FINISHED = "finished"
CANCELLED = "cancelled"
```


```python
STATUS_CHOICES = [
    (WAITING, "Waiting"),
    (IN_PROGRESS, "In Progress"),
    (FINISHED, "Finished"),
    (CANCELLED, "Cancelled"),
]
```


### `GamePlayer`


Модел за участник в конкретна игра


| Поле | Тип | Изисквания |
|---|---|---|
| `game` | `ForeignKey` | Връзка към `Game`, `on_delete=models.CASCADE` |
| `user` | `ForeignKey` | Връзка към `settings.AUTH_USER_MODEL`, `on_delete=models.PROTECT` |
| `player_order` | `PositiveSmallIntegerField` | Задължително |
| `score` | `IntegerField` | Стойност по подразбиране `0` |
| `is_active` | `BooleanField` | Стойност по подразбиране `True` |
| `joined_at` | `DateTimeField` | `auto_now_add=True` |


### `Round`


Модел за един рунд от конкретна игра


| Поле | Тип | Изисквания |
|---|---|---|
| `game` | `ForeignKey` | Връзка към `Game`, `on_delete=models.CASCADE` |
| `number` | `PositiveIntegerField` | Задължително |
| `status` | `CharField` | Задължително, choices |
| `question_type` | `CharField` | Задължително, choices |
| `choice_question` | `ForeignKey` | Връзка към `ChoiceQuestion`, `null=True`, `blank=True`, `on_delete=models.PROTECT` |
| `numeric_question` | `ForeignKey` | Връзка към `NumericQuestion`, `null=True`, `blank=True`, `on_delete=models.PROTECT` |
| `started_at` | `DateTimeField` | `null=True`, `blank=True` |
| `finished_at` | `DateTimeField` | `null=True`, `blank=True` |


### `Round status`


```python
PENDING = "pending"
OPEN = "open"
CLOSED = "closed"
EVALUATED = "evaluated"
```


```python
ROUND_STATUS_CHOICES = [
    (PENDING, "Pending"),
    (OPEN, "Open"),
    (CLOSED, "Closed"),
    (EVALUATED, "Evaluated"),
]
```


### `Question type`


```python
CHOICE = "choice"
NUMERIC = "numeric"
```


```python
QUESTION_TYPE_CHOICES = [
    (CHOICE, "Choice"),
    (NUMERIC, "Numeric"),
]
```


### `RoundAnswer`


Модел за отговор на конкретен играч в конкретен рунд


| Поле | Тип | Изисквания |
|---|---|---|
| `round` | `ForeignKey` | Връзка към `Round`, `on_delete=models.CASCADE` |
| `player` | `ForeignKey` | Връзка към `GamePlayer`, `on_delete=models.CASCADE` |
| `selected_option` | `ForeignKey` | Връзка към `AnswerOption`, `null=True`, `blank=True`, `on_delete=models.PROTECT` |
| `numeric_value` | `IntegerField` | `null=True`, `blank=True` |
| `is_correct` | `BooleanField` | `null=True`, `blank=True` |
| `points_awarded` | `IntegerField` | Стойност по подразбиране `0` |
| `submitted_at` | `DateTimeField` | `null=True`, `blank=True` |


## Database структура


- Един `Game` има много `GamePlayer`
- Един `Game` има много `Round`
- Един `Round` има много `RoundAnswer`
- Един `GamePlayer` има много `RoundAnswer`
- Един `Round` използва или `ChoiceQuestion`, или `NumericQuestion`
- Един `RoundAnswer` съдържа или `selected_option`, или `numeric_value`


## Връзки


```text
Game
├── GamePlayer
├── Round
│   ├── ChoiceQuestion or NumericQuestion
│   └── RoundAnswer
│       └── GamePlayer
```


- Една игра → много участници
- Една игра → много рундове
- Един рунд → много отговори
- Един участник → много отговори в различни рундове
- Един рунд → точно един въпрос
- Един отговор → принадлежи на един участник и един рунд
