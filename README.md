# Django Site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри контейнера Django приложение запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).


## Структура директорий

```text
k8s-test-django/
├── backend_main_django/
│   ├── src/
│   │   ├── webapp/
│   │   └── manage.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── unit_config.json
├── k8s-base/
│   ├── 01-config/
│   ├── 02-database/
│   ├── 03-app/
│   └── 04-jobs/
├── k8s-extras/
│   └── ingress/
├── docker-compose.yml
└── README.md
```

## Как подготовить окружение к локальной разработке

Код в репозитории полностью докеризирован, поэтому для запуска приложения вам понадобится Docker. Инструкции по его установке ищите на официальных сайтах:

- [Get Started with Docker](https://www.docker.com/get-started/)

Вместе со свежей версией Docker к вам на компьютер автоматически будет установлен Docker Compose. Дальнейшие инструкции будут его активно использовать.

## Как запустить сайт для локальной разработки

Запустите базу данных и сайт:

```shell
$ docker compose up
```

В новом терминале, не выключая сайт, запустите несколько команд:

```shell
$ docker compose run --rm web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker compose run --rm web ./manage.py createsuperuser  # создаём в БД учётку суперпользователя
```

Готово. Сайт будет доступен по адресу [http://127.0.0.1:8080](http://127.0.0.1:8080). Вход в админку находится по адресу [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/).

## Как вести разработку

Все файлы с кодом django смонтированы внутрь докер-контейнера, чтобы Nginx Unit сразу видел изменения в коде и не требовал постоянно пересборки докер-образа -- достаточно перезапустить сервисы Docker Compose.

### Как обновить приложение из основного репозитория

Чтобы обновить приложение до последней версии подтяните код из центрального окружения и пересоберите докер-образы:

``` shell
$ git pull
$ docker compose build
```

После обновлении кода из репозитория стоит также обновить и схему БД. Вместе с коммитом могли прилететь новые миграции схемы БД, и без них код не запустится.

Чтобы не гадать заведётся код или нет — запускайте при каждом обновлении команду `migrate`. Если найдутся свежие миграции, то команда их применит:

```shell
$ docker compose run --rm web ./manage.py migrate
…
Running migrations:
  No migrations to apply.
```

### Как добавить библиотеку в зависимости

В качестве менеджера пакетов для образа с Django используется pip с файлом requirements.txt. Для установки новой библиотеки достаточно прописать её в файл requirements.txt и запустить сборку докер-образа:

```sh
$ docker compose build web
```

Аналогичным образом можно удалять библиотеки из зависимостей.

<a name="env-variables"></a>
## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

## Запуск в Kubernetes

Этот проект можно запустить в локальном кластере Kubernetes (Minikube).


1. **Запуск кластера и загрузка образа**

```shell
minikube image load django_app
```

2. **Развертывание приложения:**

```shell
kubectl apply -R -f k8s-base/
```

Проверьте, что все поды перешли в статус Running, а задача миграции завершилась статусом Completed:

```shell
kubectl get pods -w
```


3. **Доступ к сайту**

```shell
minikube service django-k8s-service
```

4. **Создание администратора**

```shell
kubectl exec -it <ИМЯ_ПОДА_DJANGO> -- python manage.py createsuperuser
```

## Запуск в Kubernetes (Ingress)

Проект настроен для работы через **Ingress Controller**, что позволяет открывать сайт по красивому домену `http://star-burger.test` на стандартном 80 порту.

### Предварительные требования
Убедитесь, что у вас запущен Minikube и включен аддон Ingress:
```shell
minikube start
minikube addons enable ingress
```

1. Настройка DNS (файл hosts)

Чтобы браузер понимал, где искать локальный домен, добавьте запись в файл hosts.

Windows: C:\Windows\System32\drivers\etc\hosts (открыть от имени Администратора)

Linux/macOS: /etc/hosts (sudo nano /etc/hosts)

Добавьте строку:

```
127.0.0.1 star-burger.test
```

2. В целях безопасности пароли не хранятся в репозитории. Создайте файл django-secrets.yaml (или используйте пример) и примените его:

```
kubectl apply -f django-secrets.yaml
```

3. Запуск приложения
```
kubectl apply -f django-k8s.yaml
kubectl apply -f k8s-extras/ingress/
```

## Очистка устаревших сессий

**Применение конфигурации:**

```shell
kubectl apply -f cronjob.yaml
```