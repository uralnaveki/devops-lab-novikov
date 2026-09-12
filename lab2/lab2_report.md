# Отчёт по лабораторной работе №2

## «Настройка CI/CD пайплайна с GitHub Actions»

## 1. Цель работы

Изучить принципы непрерывной интеграции и доставки (CI/CD) с использованием GitHub Actions.

Научиться:

- настраивать автоматическую сборку Docker-образа при пуше в репозиторий;
- публиковать образ в Docker Hub;
- безопасно хранить учётные данные через секреты GitHub;
- добавлять шаг имитации деплоя в пайплайн.

---

## 2. Оборудование и программное обеспечение

- **Операционная система:** Windows 10 (10.0.26200.9168) с WSL2.
- **Docker Desktop.**
- **Git.**
- **Аккаунт GitHub.**
- **Аккаунт Docker Hub.**
- **Файлы из лабораторной работы №1:** `app.py`, `requirements.txt`, `Dockerfile`.

---

## 3. Ход выполнения работы

### 3.1. Подготовка проекта

Файлы из первой лабораторной работы (`app.py`, `requirements.txt`, `Dockerfile`) были скопированы в новый репозиторий на GitHub под названием `flask-docker-app`.

Структура проекта на момент начала работы:

```text
flask-docker-app/
├── app.py
├── requirements.txt
└── Dockerfile
```

### 3.2. Создание аккаунта и репозитория на Docker Hub

Создан аккаунт на [hub.docker.com](https://hub.docker.com/).

В личном кабинете создан новый публичный репозиторий для образа — **my-flask-app**.

В разделе **Account Settings → Security** сгенерирован Personal Access Token с правами **Read, Write**. Токен сохранён — он понадобится для настройки секретов GitHub.

### 3.3. Настройка секретов GitHub

В репозитории на GitHub через **Settings → Secrets and variables → Actions** добавлены два секрета:

| Имя секрета | Значение | Назначение |
|-------------|----------|------------|
| `DOCKER_USERNAME` | Логин на Docker Hub | Имя пользователя для входа |
| `DOCKERHUB_TOKEN` | Personal Access Token | Пароль для входа в Docker Hub |

Использование токена вместо пароля является рекомендуемой практикой и позволяет отзывать доступ без смены основного пароля.

### 3.4. Создание workflow-файла

В корне репозитория создана папка `.github/workflows/`, а внутри — файл `docker-build.yml` со следующим содержимым:


```yaml
name: Docker Build and Push

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      # 1. Клонирование кода
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. Настройка Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 3. Логин в Docker Hub
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 4. Сборка и публикация образа
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/my-flask-app:latest

      # 5. Имитация деплоя
      - name: Deploy
        run: echo "Deploying ${{ secrets.DOCKER_USERNAME }}/my-flask-app:latest to production..."
```

![Проверка логов](images/laba2_4.png)

## 3.5. Описание пайплайна

| Шаг | Экшен | Назначение |
|-----|-------|------------|
| 1 | `actions/checkout@v4` | Клонирование исходного кода репозитория в раннер |
| 2 | `docker/setup-buildx-action@v3` | Настройка Docker Buildx для сборки образов |
| 3 | `docker/login-action@v3` | Аутентификация в Docker Hub с использованием секретов |
| 4 | `docker/build-push-action@v6` | Сборка образа из Dockerfile и его публикация в Docker Hub с тегом `username/my-flask-app:latest` |
| 5 | `run: echo ...` | Имитация шага деплоя (в реальном проекте здесь был бы деплой на сервер) |

**Триггер** пайплайна — событие `push` в ветку `main`. **Раннер** — `ubuntu-latest`.

## 3.6. Тестирование пайплайна

### 1. Коммит и пуш workflow-файла

```bash
git add .github/workflows/docker-build.yml
git commit -m "Add CI/CD workflow"
git push origin main
```

### 2. Автоматический запуск пайплайна

Пайплайн запустился автоматически. Во вкладке **Actions** репозитория отображается запуск workflow **Docker Build and Push**.


### 3. Проверка логов шагов

Проверены логи каждого шага — все завершились успешно. В логе шага **Log in to Docker Hub** отображается сообщение **Login Succeeded**, в логе **Build and push Docker image** — информация о сборке и публикации слоёв образа.

![Сводка сборки](images/laba2_5.png)

### 4. Сводка сборки

По завершении сборки сформирована сводка **Docker Build summary** с информацией о кэше и длительности сборки.

![Предупреждение о Node.js 20](images/laba2_6.png)

### 5. Предупреждение о Node.js 20

В ходе выполнения появилось следующее сообщение:

```text
Warning: Node.js 20 is deprecated. The following actions target Node.js 20 but are
being forced to run on Node.js 24: actions/checkout@v4, docker/build-push-action@v6,
docker/login-action@v3, docker/setup-buildx-action@v3.
```
![Запуск контейнера](images/laba2_9.png)

Это **предупреждение, а не ошибка**. Оно означает, что GitHub постепенно переводит раннеры на Node.js 24, а используемые экшены пока заявлены как поддерживающие Node.js 20. Пайплайн при этом завершился успешно. 

### 6. Проверка результата на Docker Hub

Результат проверен на Docker Hub: в репозитории **my-flask-app** появился тег **latest** со свежей датой сборки.

![Сборка образа локально](images/laba2_8.png)



## 4. Итоговая структура проекта

```text
flask-docker-app/
├── .github/
│   └── workflows/
│       └── docker-build.yml      ← CI/CD пайплайн
├── app.py                        ← Flask-приложение
├── requirements.txt              ← зависимости
└── Dockerfile                    ← инструкция сборки образа
```


## 5. Выводы

В ходе лабораторной работы был настроен полноценный CI/CD-пайплайн с использованием GitHub Actions. Пайплайн автоматически:

- запускается при каждом пуше в ветку `main`;
- клонирует исходный код;
- настраивает Docker Buildx;
- аутентифицируется в Docker Hub через секреты;
- собирает Docker-образ из `Dockerfile`;
- публикует образ в Docker Hub с тегом `latest`;
- выполняет имитацию шага деплоя.

Учётные данные Docker Hub хранятся в виде секретов репозитория, что исключает их попадание в логи и исходный код. Токен доступа позволяет отозвать права без смены пароля.

Таким образом, цель работы достигнута: изучены принципы непрерывной интеграции и доставки, освоены инструменты GitHub Actions и Docker Hub, получен работающий автоматизированный пайплайн сборки и публикации контейнерных образов.
