
Информационная безопасность
Лабораторная работа №1
Стек
    Python 3.10
    Flask 3.1.3
    Flask-SQLAlchemy (ORM)
    Flask-Bcrypt (хэширование паролей)
    PyJWT (JWT-токены)
    SQLite

Описание API

POST /api/register: метод для регистрации пользователя (принимает логин и пароль).
{
  "username": "student",
  "password": "StrongPass123"
}

POST /auth/login: метод для аутентификации пользователя (принимает логин и пароль, возвращает JWT).
{
  "username": "student",
  "password": "StrongPass123"
}

GET /api/data: метод для получения данных (список пользователей). Доступ только у аутентифицированных пользователей.
Описание реализованных мер защиты
    От SQLi код защищен с помощью ORM SQLAlchemy. Все запросы к БД выполняются через параметризованные вызовы, конкатенация строк в SQL не используется:
user = User.query.filter_by(username=data['username']).first()
    От XSS защищен с помощью экранирования пользовательских данных перед возвратом в ответе API:
from markupsafe import escape
output = [{'id': u.id, 'username': escape(u.username)} for u in users]
    От Broken Authentication:
        пароли хэшируются через bcrypt (автоматическая соль, адаптивная сложность)
        при успешном логине выдается JWT-токен (HS256, срок жизни 30 минут)
        на защищенных эндпоинтах работает middleware @token_required, которое проверяет подпись и срок действия токена:
hashed = bcrypt.generate_password_hash(data['password']).decode('utf-8')
и
token = jwt.encode({
    'user_id': user.id,
    'exp': datetime.datetime.now(datetime.UTC) + datetime.timedelta(minutes=30)
}, app.config['SECRET_KEY'], algorithm="HS256")

Отчеты из pipeline
В CI/CD настроены два security-сканера:
    SAST - Bandit (статический анализ Python-кода).
    SCA - pip-audit (проверка зависимостей на известные CVE).


Последний запуск пайплайна:
https://github.com/krokodelopuma/ib_lab1/actions/runs/42473011830
