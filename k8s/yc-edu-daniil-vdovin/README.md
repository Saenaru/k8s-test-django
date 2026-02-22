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