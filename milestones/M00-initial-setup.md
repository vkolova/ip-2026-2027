# M00 — Initial Project Setup

## Цел

- GitHub repository за проектa
- Начална структура на проекта
- Django backend
- Django REST Framework
- React + Vite frontend
- Git конфигурация
- Работещи development сървъри

## Технологии

- Python 3.12+
- Django 5.2 LTS
- Django REST Framework
- Node.js LTS
- React
- Vite
- Git
- GitHub
- SQLite/MySQL/

## Структура

```text
quiz-conquest/
├── backend/
│   ├── config/
│   │   ├── __init__.py
│   │   ├── asgi.py
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── manage.py
│   └── requirements.txt
│
├── frontend/
│   ├── public/
│   ├── src/
│   ├── index.html
│   ├── package.json
│   ├── package-lock.json
│   └── vite.config.js
│
└── .gitignore
```

## GitHub repository

- Име: `quiz-conquest` (или друго)
- Един repository за целия проект
- Основен branch: `main`
- Папки `backend/` и `frontend/`
- Без отделни копия по седмици
- Без архиви като `.zip` и `.rar`
- Всеки Milestone се отбелязва с таг, напр. `m0-setup`

## Начална структура

```bash
mkdir quiz-conquest
cd quiz-conquest
mkdir backend
```

## Backend

### Virtual environment

Linux, macOS или WSL:

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
```

Windows PowerShell:

```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### Инсталиране

```bash
python -m pip install --upgrade pip
python -m pip install "Django~=5.2" djangorestframework
python -m pip freeze > requirements.txt
```

### Django проект

Изпълнение от `backend/`:

```bash
django-admin startproject config .
```

### Django REST Framework

`backend/config/settings.py`:

```python
INSTALLED_APPS = [
    # Стандартните Django приложения
    "rest_framework",
]
```

Стандартните приложения, генерирани от Django, остават в `INSTALLED_APPS`.

### Проверка

```bash
python manage.py check
python manage.py runserver
```

Очакван резултат:

- Успешен `python manage.py check`
- Django development server на `http://127.0.0.1:8000/`
- Начална Django страница

> **Без `python manage.py migrate` в M00.** Custom user моделът и `AUTH_USER_MODEL` се добавят в M01 преди първата миграция.

## Frontend

Изпълнение от основната папка `quiz-conquest/`:

```bash
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm run dev
```

Очакван резултат:

- React + Vite проект във `frontend/`
- Успешен `npm install`
- Успешен `npm run dev`
- Начална Vite страница
- `package-lock.json` в Git
- Без `node_modules/` в Git

## `.gitignore`

```gitignore
# Python
backend/.venv/
**/__pycache__/
*.py[cod]
*.sqlite3

# Django
backend/staticfiles/
backend/media/

# Node
frontend/node_modules/
frontend/dist/

# Tests
.coverage
htmlcov/
.pytest_cache/
coverage/

# IDE and OS
.vscode/
.idea/
.DS_Store
Thumbs.db
```

## Commit-и

```text
chore: initialize project structure
chore: initialize Django and DRF
chore: initialize React with Vite
```

## Проверка

### Repository

- [ ] GitHub repository `quiz-conquest`
- [ ] Branch `main`
- [ ] Папка `backend/`
- [ ] Папка `frontend/`
- [ ] `.gitignore`
- [ ] Поне три смислени commit-а

### Backend

- [ ] Virtual environment
- [ ] `requirements.txt`
- [ ] Django проект в `backend/`
- [ ] Django package с име `config`
- [ ] `rest_framework` в `INSTALLED_APPS`
- [ ] Успешен `python manage.py check`
- [ ] Успешен `python manage.py runserver`
- [ ] Без изпълнени migrations
- [ ] Без `db.sqlite3` в repository-то
- [ ] Без `.venv/` в repository-то

### Frontend

- [ ] React + Vite проект във `frontend/`
- [ ] `package.json`
- [ ] `package-lock.json`
- [ ] Успешен `npm install`
- [ ] Успешен `npm run dev`
- [ ] Без `node_modules/` в repository-то

## Предаване

- Repository URL
- Branch: `main`
- Tag: `m0-setup`
- Commit SHA

```bash
git add .
git commit -m "chore: complete initial project setup"
git push origin main
git tag -a m0-setup -m "Milestone 0: Initial project setup"
git push origin m0-setup
```

## Ресурси

- [Django — Installation](https://docs.djangoproject.com/en/5.2/topics/install/)
- [Django — Creating a project](https://docs.djangoproject.com/en/5.2/intro/tutorial01/)
- [Django REST Framework — Quickstart](https://www.django-rest-framework.org/tutorial/quickstart/)
- [Python — Virtual environments](https://docs.python.org/3/tutorial/venv.html)
- [Vite — Getting Started](https://vite.dev/guide/)
- [GitHub — Ignoring files](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files)
