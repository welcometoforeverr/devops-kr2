# Анализ исходного kr2-template

Источник: https://gitverse.ru/dgimatdinov/kr2-template

| Место | Дефект | Последствие | Исправление |
|---|---|---|---|
| Dockerfile:1 | python:latest | Непредсказуемая смена Python/системных пакетов | python:3.12-slim, тот же базовый образ, что в ПР7 |
| Dockerfile:5 | COPY . . при отсутствии .dockerignore | Попадание .env и истории Git в контекст/слой | .dockerignore и отдельные COPY requirements.txt/app.py |
| Dockerfile:5–6 | Весь код копируется до pip install | Изменение кода сбрасывает кэш зависимостей | requirements → pip install → app.py |
| Dockerfile | Нет USER | Процесс приложения работает от root | Системный пользователь app, USER app |
| Dockerfile:8 | EXPOSE8000 вместо8002 | Неверная документация порта варианта2 | EXPOSE8002; реальную публикацию задаёт Compose |
| Dockerfile:9 | Gunicorn слушает127.0.0.1 внутри контейнера | DNAT на интерфейс контейнера не достигает listener | bind0.0.0.0:8002 внутри, публикация127.0.0.1:8002 на хосте |
| Dockerfile:9 | CMD в shell-форме | Промежуточный sh, некорректная доставка сигналов | Exec-массив Gunicorn, штатная остановка |
| Корень шаблона | Отслеживается.env, отсутствует.gitignore | Пароль доступен через Git/историю | Новый пароль на VM, .gitignore, .env.example без секрета |

Исходные app.py, requirements.txt и Dockerfile импортированы в main для сравнения.
Значения исходного .env не переопубликованы. app.py в исправленной ветке отличается
только первой строкой `# KR2: Kalugin`; requirements.txt сохранён.
Пароль из шаблона не используется. Новый пароль создаётся только на сервере,
не входит в Git, образ и отчёт. Перезапись существующего .env при rsync исключена.
