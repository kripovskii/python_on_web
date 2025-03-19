# Маршрутизация во Flask

## Введение
Маршрутизация — это процесс сопоставления URL-адресов с определёнными функциями Python в веб-приложении. Во Flask маршруты определяются с помощью декораторов.

### Что такое декораторы?
Декораторы в Python — это специальные функции, которые изменяют поведение других функций. В контексте Flask, декоратор `@app.route()` используется для привязки URL к функции обработчика.

## Основные маршруты
Простейший маршрут определяется с помощью декоратора `@app.route()`:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Главная страница"

if __name__ == '__main__':
    app.run(debug=True)
```

## Параметры в маршрутах
Можно передавать параметры в URL:

```python
@app.route('/user/<name>')
def user(name):
    return f'Привет, {name}!'
```

Тип данных можно указать явно:

```python
@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f'Пост номер {post_id}'
```

## Обработчики HTTP-методов
По умолчанию маршрут обрабатывает только `GET`-запросы. Чтобы разрешить другие методы, укажите `methods`:

```python
@app.route('/submit', methods=['GET', 'POST'])
def submit():
    if request.method == 'POST':
        return "Форма отправлена"
    return "Страница с формой"
```

## URL с подмаршрутами
Можно использовать вложенные маршруты:

```python
@app.route('/blog/')
def blog():
    return "Страница блога"

@app.route('/blog/<int:post_id>')
def blog_post(post_id):
    return f'Пост в блоге №{post_id}'
```

## Перенаправления и `url_for`
Функция `redirect()` позволяет перенаправлять пользователей:

```python
from flask import redirect, url_for

@app.route('/old-page')
def old_page():
    return redirect(url_for('home'))
```

