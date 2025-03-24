# Авторизация Flask

## **1. Основные концепции авторизации**
Во Flask есть несколько способов реализовать авторизацию пользователей:

1. **Ручная проверка сессий** – хранение пользовательских данных в `session`.
2. **Flask-Login** – специальная библиотека для управления аутентификацией пользователей.
3. **OAuth (Google, Facebook, GitHub и т. д.)** – используется, если нужна авторизация через сторонние сервисы.

Наиболее удобный и безопасный способ – использование **Flask-Login**.

---

## **2. Установка и настройка проекта**
### Установка зависимостей:
```sh
pip install flask flask-login flask-sqlalchemy werkzeug
```
Если планируется хеширование паролей (что обязательно для безопасности), то `werkzeug` уже идет в составе Flask.

---

## **3. Создание структуры проекта**
Предлагаемая структура проекта:
```
/flask_auth_project
  ├── app.py           # Основной файл приложения
  ├── models.py        # Описание модели пользователя
  ├── config.py        # Конфигурация приложения
  ├── templates/       # HTML-шаблоны
  │   ├── login.html
  │   ├── profile.html
  ├── static/          # Статика (CSS, JS)
```

---

## **4. Настройка базы данных и модели пользователя (`models.py`)**
В качестве базы данных используем **SQLite** (можно заменить на PostgreSQL или MySQL).

```python
from flask_sqlalchemy import SQLAlchemy
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash

db = SQLAlchemy()

class User(db.Model, UserMixin):  # UserMixin добавляет поддержку Flask-Login
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)

    def set_password(self, password):
        """Хеширует и сохраняет пароль"""
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        """Проверяет введенный пароль с хешированным"""
        return check_password_hash(self.password_hash, password)
```

> **Зачем хеширование паролей?**  
> Никогда не храните пароли в открытом виде! Хеширование делает их безопасными.

---

## **5. Настройка приложения Flask (`app.py`)**
```python
from flask import Flask, render_template, redirect, url_for, request, flash
from flask_login import LoginManager, login_user, logout_user, login_required, current_user
from models import db, User

app = Flask(__name__)

# Настройка базы данных
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'supersecretkey'  # Защищает сессии

# Инициализация базы данных
db.init_app(app)

# Настройка Flask-Login
login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view = 'login'  # Страница логина

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))  # Flask-Login требует функцию загрузки пользователя

# Главная страница (только для авторизованных пользователей)
@app.route('/')
@login_required
def profile():
    return f'Привет, {current_user.username}! <a href="{url_for("logout")}">Выйти</a>'

# Страница входа
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        user = User.query.filter_by(username=username).first()

        if user and user.check_password(password):
            login_user(user)
            return redirect(url_for('profile'))
        
        flash('Неверные имя пользователя или пароль')

    return render_template('login.html')

# Выход из системы
@app.route('/logout')
@login_required
def logout():
    logout_user()
    return redirect(url_for('login'))

if __name__ == '__main__':
    with app.app_context():
        db.create_all()  # Создание таблиц
    app.run(debug=True)
```

---

## **6. Создание HTML-шаблонов**
### **Шаблон страницы входа (`templates/login.html`)**
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Вход</title>
</head>
<body>
    <h2>Вход</h2>
    {% with messages = get_flashed_messages() %}
        {% if messages %}
            <ul>
                {% for message in messages %}
                    <li>{{ message }}</li>
                {% endfor %}
            </ul>
        {% endif %}
    {% endwith %}
    <form method="POST">
        <input type="text" name="username" placeholder="Имя пользователя" required>
        <input type="password" name="password" placeholder="Пароль" required>
        <button type="submit">Войти</button>
    </form>
</body>
</html>
```

---

## **7. Добавление пользователя в базу данных**
Теперь нам нужно создать пользователя вручную (в реальном приложении лучше сделать форму регистрации).

Открываем Python-терминал:
```python
from app import db, User
db.create_all()  # Создаст таблицу, если её нет

# Создаем пользователя
user = User(username='admin')
user.set_password('password')
db.session.add(user)
db.session.commit()
```

Теперь можно зайти с логином `admin` и паролем `password`.

---

## **8. Улучшение безопасности**
### **1. Ограничение количества попыток входа**
Можно использовать `Flask-Limiter`, чтобы предотвратить подбор паролей:
```sh
pip install flask-limiter
```
Добавляем в `app.py`:
```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(get_remote_address, app=app, default_limits=["5 per minute"])  # 5 попыток в минуту
```

### **2. Защита от CSRF-атак**
Можно подключить `Flask-WTF`:
```sh
pip install flask-wtf
```
И использовать `csrf_token` в формах.

---

