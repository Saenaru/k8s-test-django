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