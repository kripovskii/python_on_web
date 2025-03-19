# Работа с базой данных во Flask

## Введение
Flask поддерживает работу с различными базами данных, но сам по себе не включает встроенный механизм для работы с ними. Чаще всего используется SQLAlchemy — мощный ORM (Object Relational Mapper), который облегчает взаимодействие с базами данных.

## Установка SQLAlchemy
Перед началом работы установим нужный пакет:

```sh
pip install Flask-SQLAlchemy
```

## Подключение базы данных
Создадим Flask-приложение и подключим базу данных SQLite:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
db = SQLAlchemy(app)
```

## Определение модели данных
Создадим таблицу `User` с полями `id` и `name`:

```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)

    def __repr__(self):
        return f'<User {self.name}>'
```

## Создание таблиц
Перед первым использованием базы данных необходимо создать таблицы:

```python
with app.app_context():
    db.create_all()
```

## Работа со связями между таблицами
SQLAlchemy поддерживает несколько типов связей: **one-to-many**, **many-to-one**, **many-to-many**.

### One-to-Many (Один ко многим)
Один пользователь может иметь несколько постов:

```python
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    user = db.relationship('User', backref=db.backref('posts', lazy=True))
```

Теперь можно получить все посты пользователя через `user.posts`.

### Many-to-Many (Многие ко многим)
Допустим, у нас есть пользователи и группы, и каждый пользователь может состоять в нескольких группах:

```python
user_group = db.Table('user_group',
    db.Column('user_id', db.Integer, db.ForeignKey('user.id'), primary_key=True),
    db.Column('group_id', db.Integer, db.ForeignKey('group.id'), primary_key=True)
)

class Group(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    users = db.relationship('User', secondary=user_group, backref=db.backref('groups', lazy=True))
```

Теперь можно добавлять пользователей в группы через `group.users.append(user)`.

## Добавление данных
Добавим пользователя в базу данных:

```python
with app.app_context():
    new_user = User(name='Alice')
    db.session.add(new_user)
    db.session.commit()
```

## Получение данных
Извлечём всех пользователей:

```python
with app.app_context():
    users = User.query.all()
    print(users)
```

## Обновление данных
Обновим имя пользователя:

```python
with app.app_context():
    user = User.query.first()
    user.name = 'Bob'
    db.session.commit()
```

## Удаление данных
Удалим пользователя:

```python
with app.app_context():
    user = User.query.first()
    db.session.delete(user)
    db.session.commit()
```

