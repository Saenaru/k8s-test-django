# k8s-test-django: Production Deployment (Yandex Cloud)

Этот репозиторий содержит конфигурацию для развертывания проекта Star-k8s-test-django в кластере Managed Kubernetes (Yandex Cloud).


## Ссылки проекта
* **Работающая версия сайта:** [https://edu-daniil-vdovin.yc-sirius-dev.pelid.team/](https://edu-daniil-vdovin.yc-sirius-dev.pelid.team/)

## Как задеплоить код

В этом окружении трафик обрабатывается внешним балансировщиком Yandex ALB и направляется в локальный Nginx (`main-nginx`).

### Развертывание базовых сервисов Nginx
Если по какой-то причине ресурсы были удалены, восстановить рабочую конфигурацию (Service и Deployment) можно командами:

```bash
kubectl apply -f nginx-service.yaml -n edu-daniil-vdovin
kubectl apply -f nginx-deployment.yaml -n edu-daniil-vdovin
```

## Как подготовить dev окружение

Для безопасного подключения к внешней базе данных Managed PostgreSQL требуется SSL-сертификат.

### Создание секрета с сертификатом
Скачайте корневой сертификат и создайте из него Secret в Kubernetes:

```bash
# Пример создания секрета из файла
kubectl create secret generic postgres --from-file=root.crt=./root.crt -n edu-daniil-vdovin
```

## Сборка и публикация Docker-образов

Для деплоя новых версий приложения в кластер необходимо собрать Docker-образ и отправить его в Docker Hub. Версионирование образов строго привязано к короткому хэшу Git-коммита.

### Инструкция (для PowerShell):

1. Получите короткий хэш текущего коммита:

```powershell
$HASH = git rev-parse --short HEAD
```

2. Соберите образ приложения (команду нужно выполнять из корня репозитория):

```powershell
docker build -t логин_аккаунта/k8s-test-django:$HASH backend_main_django/
```

3. Опубликуйте собранный образ в Docker Hub:

```powershell
docker push логин_аккаунта/k8s-test-django:$HASH
```

## Инструкции по деплою в кластер

Все манифесты разделены по компонентам в директории k8s/yc-edu-daniil-vdovin/.

1. **Подготовка конфигурации и секретов:**
   Убедитесь, что в кластере созданы файлы с настройками.

```bash
kubectl create secret generic postgres --from-file=root.crt=./root.crt -n edu-daniil-vdovin
```

2. Развертывание всех компонентов

```bash
kubectl apply -f k8s/yc-edu-daniil-vdovin/ --recursive -n edu-daniil-vdovin
```

3. Применение миграций базы данных:

```bash
kubectl exec -it deployment/django-app -n edu-daniil-vdovin -- python manage.py migrate
```
