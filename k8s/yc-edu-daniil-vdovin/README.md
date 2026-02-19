## Как задеплоить код

В этом окружении трафик обрабатывается внешним балансировщиком Yandex ALB и направляется в локальный Nginx (`main-nginx`).

### Развертывание базовых сервисов Nginx
Если по какой-то причине ресурсы были удалены, восстановить рабочую конфигурацию (Service и Deployment) можно командами:

```bash
kubectl apply -f nginx-service.yaml -n edu-daniil-vdovin
kubectl apply -f nginx-deployment.yaml -n edu-daniil-vdovin
```
