# Отчет по лабораторной работе №7

## Контейнеризация веб-приложения с использованием Docker и Docker Compose

**Выполнила:** Артеменко Арина ИУ8-22  
**GitHub ID:** gest475  
**Репозиторий:** https://github.com/gest475/lab07  


## 1. Цель работы

Освоение практических навыков контейнеризации приложений с использованием Docker, включая:
- создание Dockerfile для Python-приложений;
- настройку мультиконтейнерных приложений с Docker Compose;
- интеграцию веб-сервера и базы данных;
- использование healthcheck для контроля зависимостей между сервисами.

---

## 2. Подготовка окружения

### 2.1 Настройка репозитория

```bash
export GITHUB_USERNAME=gest475
cd ~/gest475/workspace
git clone https://github.com/${GITHUB_USERNAME}/lab06 projects/lab_docker
cd projects/lab_docker
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab_docker
```
##2.2 Установка Docker

```bash
sudo apt-get update
sudo apt-get install -y docker.io docker-compose
sudo service docker start
sudo docker ps
```
Результат: Docker демон успешно запущен.

##3. Тестовый запуск контейнера
###3.1 Создание тестового скрипта

```bash
cat > main.py <<EOF
print("Hello, Docker!")
EOF
```
##3.2 Dockerfile для теста

```
FROM python:3.9-slim
WORKDIR /app
COPY main.py .
CMD ["python", "main.py"]
```
##3.3 Сборка и запуск
```bash
sudo docker build -t test-python .
sudo docker run --rm test-python
```
Вывод: Hello, Docker!

Вывод: Контейнеризация базового Python-скрипта выполнена успешно.

##4. Разработка веб-приложения

###4.1 Flask-приложение (app/app.py)
Приложение реализует следующий функционал:

Веб-интерфейс с формой для добавления задач

Подключение к MySQL с механизмом повторных попыток

Автоматическое создание таблицы tasks при запуске

Отображение списка всех добавленных задач

**Листинг кода app.py:**

```
import os
import time
from flask import Flask, request, render_template_string, redirect, url_for
import mysql.connector

app = Flask(__name__)

# Функция подключения к базе данных с повторными попытками
def get_db_connection():
    for i in range(15):
        try:
            conn = mysql.connector.connect(
                host=os.getenv('DB_HOST', 'db'),
                user=os.getenv('DB_USER', 'root'),
                password=os.getenv('DB_PASSWORD', 'password'),
                database=os.getenv('DB_NAME', 'tasks_db')
            )
            return conn
        except mysql.connector.Error:
            time.sleep(2)
    return None

# Создание таблицы при запуске
conn = get_db_connection()
if conn:
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS tasks (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL);")
    conn.commit()
    cursor.close()
    conn.close()

# HTML шаблон
HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>Docker Tasks App</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        input[type="text"] { padding: 8px; width: 250px; }
        input[type="submit"] { padding: 8px 15px; background-color: #28a745; color: white; border: none; cursor: pointer; }
        li { padding: 10px; background: #f4f4f9; margin-bottom: 5px; }
    </style>
</head>
<body>
    <h1>Список задач</h1>
    <form action="/add" method="POST">
        <input type="text" name="task_name" placeholder="Введите задачу" required>
        <input type="submit" value="Добавить">
    </form>
    <ul>
        {% for task in tasks %}
            <li>ID: {{ task[0] }} — {{ task[1] }}</li>
        {% else %}
            <li>Задач пока нет</li>
        {% endfor %}
    </ul>
</body>
</html>
"""

# Главная страница
@app.route('/')
def index():
    conn = get_db_connection()
    tasks = []
    if conn:
        cursor = conn.cursor()
        cursor.execute("SELECT id, name FROM tasks ORDER BY id DESC")
        tasks = cursor.fetchall()
        cursor.close()
        conn.close()
    return render_template_string(HTML_TEMPLATE, tasks=tasks)

# Добавление задачи
@app.route('/add', methods=['POST'])
def add_task():
    task_name = request.form.get('task_name')
    if task_name:
        conn = get_db_connection()
        if conn:
            cursor = conn.cursor()
            cursor.execute("INSERT INTO tasks (name) VALUES (%s)", (task_name,))
            conn.commit()
            cursor.close()
            conn.close()
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
##4.2 Зависимости (app/requirements.txt)
```
flask
mysql-connector-python
```
##4.3 Инициализация БД (db/init.sql)
```
CREATE TABLE IF NOT EXISTS tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```
