# M01 — Authentication and Users

## Предпоставка

- Завършен M00
- Tag `m0-setup`
- Работещ Django backend
- Инсталиран Django REST Framework
- Работещ React + Vite frontend
- Без изпълнени Django migrations

## Цел

- Custom Django user model
- Потребителски профил
- Регистрация
- Вход
- Изход
- Session authentication
- Преглед на собствен профил
- Редактиране на собствен профил
- Permissions
- CSRF защита
- Backend тестове
- Самостоятелно разработен UI

## Извън M01

- Social login
- JWT authentication
- Email verification
- Password reset
- Avatar upload
- Потребителски роли
- Игрови permissions
- Lobby
- Multiplayer функционалност
- WebSockets

## Основни понятия

### Authentication

- Установяване на самоличност
- Credentials
- Login
- Logout
- Session
- `request.user`
- `is_authenticated`

### Authorization

- Проверка на права
- Permissions
- Достъп до защитени endpoints
- Достъп само до собствени данни

## Django application

Създаване от `backend/`:

```bash
python manage.py startapp accounts
```

Очаквана структура:

```text
backend/
├── accounts/
│   ├── migrations/
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── test_models.py
│   │   ├── test_serializers.py
│   │   └── test_api.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── serializers.py
│   ├── urls.py
│   └── views.py
├── config/
└── manage.py
```

Добавяне на `accounts` в `INSTALLED_APPS`.

## User модел

Изисквания:

- Наследяване от `AbstractUser`
- Уникален `username`
- Уникален `email`
- Django password management
- Регистрация в Django admin

Минимална структура:

```python
from django.contrib.auth.models import AbstractUser
from django.db import models


class User(AbstractUser):
    email = models.EmailField(unique=True)
```

`backend/config/settings.py`:

```python
AUTH_USER_MODEL = "accounts.User"
```

Правила:

- `AUTH_USER_MODEL` преди първата миграция
- `settings.AUTH_USER_MODEL` при model relations
- `get_user_model()` в runtime код
- Без директен import на стандартния Django `User`
- Без директно присвояване на `password`
- `create_user()` или `set_password()` за пароли

## Profile модел

| Поле | Тип | Правила |
|---|---|---|
| `user` | `OneToOneField` | `on_delete=CASCADE`, `related_name="profile"` |
| `nickname` | `CharField` | максимум 30 символа, уникално |
| `avatar_key` | `CharField` | максимум 30 символа, стойност по подразбиране |

Допустими начални стойности за `avatar_key`:

```text
knight-1
knight-2
knight-3
knight-4
```

## Първа миграция

Предварителна проверка:

- [ ] `accounts` в `INSTALLED_APPS`
- [ ] Custom `User`
- [ ] `AUTH_USER_MODEL = "accounts.User"`
- [ ] `Profile`

Команди:

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py check
```

## API

| Method | Endpoint | Достъп | Успех |
|---|---|---|---|
| `GET` | `/api/auth/csrf/` | Public | `204 No Content` |
| `POST` | `/api/auth/register/` | Public | `201 Created` |
| `POST` | `/api/auth/login/` | Public | `200 OK` |
| `POST` | `/api/auth/logout/` | Authenticated | `204 No Content` |
| `GET` | `/api/auth/me/` | Authenticated | `200 OK` |
| `PATCH` | `/api/auth/me/` | Authenticated | `200 OK` |

## Регистрация

### Request

```http
POST /api/auth/register/
Content-Type: application/json
X-CSRFToken: <token>
```

```json
{
  "username": "player_one",
  "email": "player@example.com",
  "nickname": "MountainKnight",
  "password": "example-password",
  "password_confirm": "example-password"
}
```

### Response

```http
201 Created
```

```json
{
  "id": 1,
  "username": "player_one",
  "email": "player@example.com",
  "profile": {
    "nickname": "MountainKnight",
    "avatar_key": "knight-1"
  }
}
```

### Валидация

- Уникален username
- Уникален email
- Уникален nickname
- Еднакви `password` и `password_confirm`
- `password` като `write_only`
- `password_confirm` като `write_only`
- Създаване чрез `create_user()`
- Създаване на `User` и `Profile` в една database transaction

## Вход

### Request

```http
POST /api/auth/login/
Content-Type: application/json
X-CSRFToken: <token>
```

```json
{
  "username": "player_one",
  "password": "example-password"
}
```

### Response

```http
200 OK
Set-Cookie: sessionid=...
```

```json
{
  "id": 1,
  "username": "player_one",
  "email": "player@example.com",
  "profile": {
    "nickname": "MountainKnight",
    "avatar_key": "knight-1"
  }
}
```

## Изход

### Request

```http
POST /api/auth/logout/
Cookie: sessionid=...
X-CSRFToken: <token>
```

### Response

```http
204 No Content
```

## Текущ потребител

### Request

```http
GET /api/auth/me/
Cookie: sessionid=...
```

### Response

```http
200 OK
```

```json
{
  "id": 1,
  "username": "player_one",
  "email": "player@example.com",
  "profile": {
    "nickname": "MountainKnight",
    "avatar_key": "knight-1"
  }
}
```

## Редактиране на профил

### Request

```http
PATCH /api/auth/me/
Content-Type: application/json
Cookie: sessionid=...
X-CSRFToken: <token>
```

```json
{
  "nickname": "NewKnight",
  "avatar_key": "knight-3"
}
```

### Позволени полета

- `nickname`
- `avatar_key`

### Забранени полета

- `id`
- `username`
- `email`
- `password`
- `is_staff`
- `is_superuser`
- `groups`
- `user_permissions`

## Грешки

Примерен формат:

```json
{
  "errors": {
    "email": ["User with this email already exists."],
    "password_confirm": ["Passwords do not match."]
  }
}
```

| Ситуация | Статус |
|---|---:|
| Невалидни входни данни | `400 Bad Request` |
| Невалидни credentials | `400 Bad Request` |
| Липсваща authentication | `401 Unauthorized` или `403 Forbidden` |
| Отказан достъп | `403 Forbidden` |
| Липсващ или невалиден CSRF token | `403 Forbidden` |

Еднакво поведение и еднакъв формат във всички endpoints.

## Permissions

| Операция | Anonymous | Authenticated |
|---|---:|---:|
| Получаване на CSRF token | Да | Да |
| Регистрация | Да | Да |
| Вход | Да | Да |
| Изход | Не | Да |
| Преглед на `/me` | Не | Само собствени данни |
| Редактиране на `/me` | Не | Само собствен профил |

Правила за `/me`:

- Потребител от `request.user`
- Без user ID в URL
- Без user ID в request body
- Без достъп до чужд профил

## Session и CSRF

- Django session authentication
- Session cookie след успешен login
- `request.user` при следващи заявки
- CSRF token за unsafe HTTP methods
- `X-CSRFToken` при `POST`, `PATCH`, `PUT` и `DELETE`
- Без `@csrf_exempt`
- Без JWT
- Без password в браузъра
- Без password или password hash в API responses

## Работа в час

### Час 1 — User и Profile

- Authentication и authorization
- Django authentication system
- `AbstractUser`
- `AUTH_USER_MODEL`
- `User` и `Profile`
- `OneToOneField`
- Django admin
- Първа миграция

### Час 2 — Serializers

- Serialization и deserialization
- Field validation
- Object validation
- `write_only`
- `validated_data`
- `create_user()`
- Database transaction

### Час 3 — Authentication API

- Register
- Login
- Logout
- `/me`
- Session authentication
- Permissions
- CSRF
- HTTP status codes

### Час 4 — Backend тестове

- DRF `APIClient`
- Registration scenarios
- Session scenarios
- Permission scenarios
- Profile update scenario
- Logout scenario
- CSRF integration scenario

## Backend тестове

### 1. Успешна регистрация

- `201 Created`
- Създаден `User`
- Създаден `Profile`
- Правилен `nickname`
- Работеща hash-ната парола чрез `check_password()`
- Без password в response

### 2. Невалидна регистрация

Проверки чрез parameterized test, `subTest()` или отделни кратки случаи:

- Зает `username`
- Зает `email`
- Зает `nickname`
- Различни `password` и `password_confirm`
- `400 Bad Request`
- Подходящо error поле
- Без създаване на допълнителен потребител

### 3. Успешен login

- Валидни credentials
- `200 OK`
- Създадена session
- Успешен последващ `GET /api/auth/me/`
- Данни за правилния потребител

### 4. Неуспешен login

- Грешна парола
- `400 Bad Request`
- Без authenticated session
- Отказан последващ `GET /api/auth/me/`

### 5. Permissions за `/me`

- Отказан anonymous достъп
- Успешен authenticated достъп
- Данни само за текущия потребител

### 6. Редактиране на профил

- Промяна на `nickname`
- Промяна на `avatar_key`
- Запазени промени в базата
- Невъзможна промяна на защитено поле, например `is_staff`

### 7. Logout

- Login
- Успешен logout
- Прекратена session
- Отказан последващ достъп до `/api/auth/me/`

### 8. CSRF

- `APIClient(enforce_csrf_checks=True)`
- Session-authenticated unsafe заявка без token
- `403 Forbidden` без token
- Успешна същата заявка с валиден token

## Правила за тестовете

- Фокус върху наблюдаемо поведение
- Без тестове на стандартното поведение на Django
- Без отделен тест за всеки model атрибут
- Без дублиране на еднаква validation логика на serializer и API ниво
- Допустими няколко проверки в един логически сценарий
- Независими тестове
- Собствени тестови данни за всеки сценарий
- Без зависимост от реда на изпълнение

## Самостоятелна UI работа

Без React програмиране в часовете.

Задължителни екрани:

- Registration
- Login
- Profile

Задължително поведение:

- Регистрация
- Вход
- Изход
- Показване на текущ потребител
- Редактиране на nickname
- Избор на `avatar_key`
- Показване на validation errors
- Запазена session authentication след refresh
- Защитен profile екран или еквивалентно поведение
- Responsive layout

Свободен избор:

- Компонентна структура
- Routing
- State management
- CSS подход
- Визуален дизайн


## Acceptance criteria

### Models

- [ ] `accounts` application
- [ ] Custom `User`
- [ ] `AUTH_USER_MODEL`
- [ ] `Profile`
- [ ] Уникални username, email и nickname
- [ ] Успешни migrations
- [ ] Models в Django admin

### API

- [ ] Всички задължителни endpoints
- [ ] Спазени request и response формати
- [ ] Подходящи HTTP status codes
- [ ] Единен формат на грешките

### Security

- [ ] Django password hashing
- [ ] Session authentication
- [ ] CSRF защита
- [ ] Permissions
- [ ] Без password данни в responses
- [ ] Без достъп до чужд profile

### Tests

- [ ] Всички задължителни тестови сценарии
- [ ] Всички backend тестове минават

### UI

- [ ] Registration екран
- [ ] Login екран
- [ ] Profile екран
- [ ] Logout
- [ ] Validation errors
- [ ] Session след refresh

### Repository

- [ ] Смислена commit история
- [ ] Без `.venv/`
- [ ] Без `node_modules/`
- [ ] Без `db.sqlite3`
- [ ] Tag `m1-auth`

## Предаване

- Repository URL
- Branch: `main`
- Tag: `m1-auth`
- Commit SHA
- Команда за стартиране на backend тестовете

```bash
git add .
git commit -m "feat: complete authentication and users"
git push origin main
git tag -a m1-auth -m "Milestone 1: Authentication and users"
git push origin m1-auth
```

## Demo

1. Регистрация
2. Създаден profile
3. Logout
4. Отказан достъп до `/me`
5. Login
6. Успешен достъп до `/me`
7. Промяна на nickname и avatar
8. Refresh
9. Запазена authentication
10. Logout

## Чести грешки

- Custom user модел след първата миграция
- Директен import на стандартния Django `User`
- `User.objects.create(password=...)`
- Raw password в базата
- Password hash в API response
- `@csrf_exempt`
- JWT вместо session authentication
- `/me/<user_id>/` вместо `/me/`
- User ID от клиента като основа за authorization
- Административни полета в update serializer
- Само frontend validation
- Липсващи permission tests

## Ресурси

### Django

- [Authentication overview](https://docs.djangoproject.com/en/5.2/topics/auth/)
- [Using Django authentication](https://docs.djangoproject.com/en/5.2/topics/auth/default/)
- [Customizing authentication](https://docs.djangoproject.com/en/5.2/topics/auth/customizing/)
- [Password management](https://docs.djangoproject.com/en/5.2/topics/auth/passwords/)
- [CSRF protection](https://docs.djangoproject.com/en/5.2/howto/csrf/)
- [Database transactions](https://docs.djangoproject.com/en/5.2/topics/db/transactions/)

### Django REST Framework

- [Serializers](https://www.django-rest-framework.org/api-guide/serializers/)
- [Validators](https://www.django-rest-framework.org/api-guide/validators/)
- [Authentication](https://www.django-rest-framework.org/api-guide/authentication/)
- [Permissions](https://www.django-rest-framework.org/api-guide/permissions/)
- [Status codes](https://www.django-rest-framework.org/api-guide/status-codes/)
- [Testing](https://www.django-rest-framework.org/api-guide/testing/)

### Самостоятелна UI работа

- [React input](https://react.dev/reference/react-dom/components/input)
- [MDN — Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [Django — CSRF with AJAX](https://docs.djangoproject.com/en/5.2/howto/csrf/#using-csrf-protection-with-ajax)
- [Vite — Getting Started](https://vite.dev/guide/)
