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
- Модел за игра
- Модел за играч в конкретна игра
- Модел за рунд
- Основен game state
- Round lifecycle
- Ограничения за трима играчи
- Backend тестове за игрите, играчите и рундовете


## Основни понятия


- Domain model
- Foreign key
- One-to-many relationship
- Unique constraint
- Status field
- Choices
- Related name
- Data integrity
- Game lifecycle
- Round lifecycle


## Структура


```text
backend/
├── games/
│   ├── migrations/
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── test_games.py
│   │   ├── test_players.py
│   │   └── test_rounds.py
│   ├── admin.py
│   ├── apps.py
│   └── models.py
└── manage.py
```


## Основна идея


- Една игра има точно 3 играча
- Всеки играч участва чрез отделен `Player` запис
- Всеки играч има уникален цвят в рамките на играта
- Играта преминава през поредица от рундове
- Всеки рунд има номер, тип и status
- Победителят в рунда е `Player` от същата игра
- Играта приключва при изпълнение на условие за край
- Териториите и картата се добавят в M04


## Модели


### `Game`


| Поле | Тип | Изисквания |
|---|---|---|
| `created_at` | `DateTimeField` | `auto_now_add=True` |
| `status` | `CharField` | Задължително, `choices`, стойност по подразбиране `waiting` |


Играчите на играта се достъпват чрез обратната връзка:

```python
game.players.all()
```

Връзката се създава от `Player.game`.


### `Player`


| Поле | Тип | Изисквания |
|---|---|---|
| `user` | `ForeignKey` | Връзка към `settings.AUTH_USER_MODEL`, `on_delete=models.CASCADE` |
| `game` | `ForeignKey` | Връзка към `Game`, `on_delete=models.CASCADE`, `related_name="players"` |
| `score` | `IntegerField` | Стойност по подразбиране `0` |
| `color` | `CharField` | Задължително, `choices` |


### `Round`


| Поле | Тип | Изисквания |
|---|---|---|
| `game` | `ForeignKey` | Връзка към `Game`, `on_delete=models.CASCADE`, `related_name="rounds"` |
| `number` | `PositiveIntegerField` | Номер на рунда в рамките на играта |
| `type` | `CharField` | Задължително, `choices` |
| `status` | `CharField` | Задължително, `choices`, стойност по подразбиране `pending` |
| `winner` | `ForeignKey` | Връзка към `Player`, `null=True`, `blank=True`, `on_delete=models.SET_NULL` |
| `created_at` | `DateTimeField` | `auto_now_add=True` |
| `completed_at` | `DateTimeField` | `null=True`, `blank=True` |


## Choices


### `Game.status`


- `waiting`
- `active`
- `completed`


### `Player.color`


- `red`
- `green`
- `blue`


### `Round.type`


- `city_capture`
- `battle`
- `capital_attack`
- `bonus`


### `Round.status`


- `pending`
- `active`
- `completed`


## Методи


### `Game.get_current_round()`


```python
def get_current_round(self):
    return self.rounds.order_by("-number").first()
```


### `Game.is_active()`


```python
def is_active(self):
    return self.status == "active"
```


### `Game.is_completed()`


```python
def is_completed(self):
    return self.status == "completed"
```


## Model ограничения


### `Game`


- Начален status `waiting`
- Преход към `active` при точно 3 играча
- Преход към `completed` при край на играта


### `Player`


- Един user участва само веднъж в конкретна игра
- Всеки цвят се използва само веднъж в конкретна игра
- Score не може да бъде отрицателен


```python
class Meta:
    constraints = [
        models.UniqueConstraint(
            fields=["user", "game"],
            name="unique_user_per_game",
        ),
        models.UniqueConstraint(
            fields=["game", "color"],
            name="unique_color_per_game",
        ),
        models.CheckConstraint(
            condition=models.Q(score__gte=0),
            name="player_score_gte_0",
        ),
    ]
```


### `Round`


- Номерът е уникален в рамките на играта
- Winner принадлежи на същата игра
- Winner може да бъде празен при незавършен рунд
- Завършен рунд има `completed_at`
- В една игра има най-много един active рунд


```python
class Meta:
    constraints = [
        models.UniqueConstraint(
            fields=["game", "number"],
            name="unique_round_number_per_game",
        ),
    ]
```


## Game lifecycle


### `waiting`


- Създадена игра
- Добавяне на играчи
- Максимум 3 играча
- Уникални users
- Уникални цветове


### `active`


- Точно 3 играча
- Разрешено създаване и стартиране на рундове
- Един текущ active рунд


### `completed`


- Играта е приключила
- Не се създават нови рундове
- Не се добавят нови играчи


## Основен game flow


1. Създаване на игра със status `waiting`
2. Добавяне на трима играчи
3. Проверка на users и colors
4. Инициализиране на картата в M04
5. Преход към `active`
6. Създаване и провеждане на рундове
7. Обновяване на резултатите
8. Проверка за край на играта
9. Преход към `completed`


## Round lifecycle


### `pending`


- Рундът е създаден
- Няма победител
- Няма `completed_at`


### `active`


- Рундът е текущ
- Приемат се действия от играчите


### `completed`


- Рундът е приключил
- Записан winner
- Записан `completed_at`
- Следващият рунд може да бъде създаден


## Round типове


### `city_capture`


- Рунд за завземане на свободна територия
- Териториалната логика се добавя в следващите milestones


### `battle`


- Битка между двама играчи


### `capital_attack`


- Атака срещу столица


### `bonus`


- Допълнителен рунд
- Бонус резултат или игрово предимство


## Scoring


- Всеки `Player` има score
- Score започва от `0`
- Score се променя според резултатите от рундовете
- Конкретните правила за точкуване се добавят с gameplay логиката


## Django Admin


### `GameAdmin`


- ID
- Дата на създаване
- Status
- Брой играчи
- Филтър по status


### `PlayerAdmin`


- User
- Game
- Color
- Score
- Филтър по game и color
- Търсене по username


### `RoundAdmin`


- Game
- Number
- Type
- Status
- Winner
- Created at
- Completed at
- Филтър по game, type и status


## Backend тестове


### Задължителни сценарии


- [ ] Създаване на игра
  - Начален status `waiting`
  - Записан `created_at`


- [ ] Добавяне на играчи
  - Трима различни users
  - Три различни цвята


- [ ] Невалиден player
  - Повторен user в същата игра
  - Повторен цвят в същата игра
  - Отрицателен score


- [ ] Стартиране на игра
  - Разрешено при точно 3 играча
  - Status се променя на `active`


- [ ] Създаване на рунд
  - Game
  - Number
  - Type
  - Начален status `pending`


- [ ] Завършване на рунд
  - Winner от същата игра
  - Status `completed`
  - Записан `completed_at`


- [ ] Невалиден winner
  - Player от друга игра не може да бъде winner


- [ ] Текущ рунд
  - `get_current_round()` връща рунда с най-голям number


## Acceptance criteria


### Models


- [ ] `Game` модел
- [ ] `Player` модел
- [ ] `Round` модел
- [ ] Успешни migrations


### Game state


- [ ] Status `waiting`
- [ ] Status `active`
- [ ] Status `completed`
- [ ] Точно 3 играча при стартиране
- [ ] Уникален user в рамките на игра
- [ ] Уникален цвят в рамките на игра
- [ ] Цветове `red`, `green` и `blue`


### Rounds


- [ ] Уникален номер в рамките на играта
- [ ] Type choices
- [ ] Status choices
- [ ] Winner е `Player`
- [ ] Winner принадлежи на същата игра
- [ ] Работещ `get_current_round()`


### Django Admin


- [ ] Управление на игрите
- [ ] Управление на играчите
- [ ] Управление на рундовете
- [ ] Филтриране по status и type
- [ ] Търсене по username


### Tests


- [ ] Game tests
- [ ] Player tests
- [ ] Round tests
- [ ] Constraint tests


## Definition of Done


- [ ] Всички модели са имплементирани
- [ ] Всички migrations са commit-нати
- [ ] Django Admin работи за всички модели
- [ ] Game lifecycle е имплементиран
- [ ] Round lifecycle е имплементиран
- [ ] Model ограниченията работят
- [ ] Всички backend тестове минават успешно


## Предаване


- Код в основното repository
- Milestone tag: `m3-games-and-rounds`
- Commit-нати migrations
- Успешни backend тестове


```bash
python backend/manage.py test games
git tag -a m3-games-and-rounds -m "Milestone 3: Games and Rounds"
git push origin m3-games-and-rounds
```


## Ресурси


- [Django — Models](https://docs.djangoproject.com/en/5.2/topics/db/models/)
- [Django — Model field reference](https://docs.djangoproject.com/en/5.2/ref/models/fields/)
- [Django — Model Meta options](https://docs.djangoproject.com/en/5.2/ref/models/options/)
- [Django — Constraints](https://docs.djangoproject.com/en/5.2/ref/models/constraints/)
- [Django — QuerySet API](https://docs.djangoproject.com/en/5.2/ref/models/querysets/)
- [Django — Admin site](https://docs.djangoproject.com/en/5.2/ref/contrib/admin/)
- [Django — Testing](https://docs.djangoproject.com/en/5.2/topics/testing/)
