 # Шаблонизатор Jinja2 в Flask

## Введение
Jinja2 — это мощный шаблонизатор, используемый во Flask для генерации HTML-страниц с динамическим содержимым.

## Подключение Jinja2
Flask автоматически использует Jinja2 для рендеринга HTML-файлов, находящихся в папке `templates`.

### Пример структуры проекта:
```
myapp/
│── app.py
│── templates/
│   ├── index.html
│   ├── about.html
```

## Использование шаблонов
Для рендеринга страницы используется функция `render_template()`:

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html', title='Главная', message='Добро пожаловать!')

if __name__ == '__main__':
    app.run(debug=True)
```

## Переменные в шаблоне
В файле `index.html` можно использовать переданные переменные:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ message }}</h1>
</body>
</html>
```

## Управляющие конструкции
### Условные операторы:

```html
{% if user %}
    <p>Привет, {{ user }}!</p>
{% else %}
    <p>Вы не авторизованы.</p>
{% endif %}
```

### Циклы:

```html
<ul>
{% for item in items %}
    <li>{{ item }}</li>
{% endfor %}
</ul>
```

## Расширение шаблонов
Создадим базовый шаблон `base.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Мой сайт{% endblock %}</title>
</head>
<body>
    <header>
        <h1>Шапка сайта</h1>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

Расширим его в `index.html`:

```html
{% extends "base.html" %}

{% block title %}Главная страница{% endblock %}

{% block content %}
    <p>Добро пожаловать на сайт!</p>
{% endblock %}
```

## Фильтры Jinja2
Jinja2 поддерживает фильтры для обработки данных:

```html
<p>{{ username | upper }}</p> {# Выведет имя пользователя заглавными буквами #}
<p>{{ items | length }}</p> {# Выведет количество элементов в списке #}
```

