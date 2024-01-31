# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

## Развертывание Kubernetes кластера с помощью Minikube

Установите [minikube](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/) и [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/).
Запустите minikube:
```
minikube start
```

Активируйте Ingress:
```
minikube addons enable ingress
```

Создайте образ Django-приложения в кластере командой:
```
minikube image build -t имя_образа backend_main_django/
```
Отредактируйте файл `test-django-configmap.yaml`, подставив свои значения переменных окружения.
Запустите Django-приложение в кластере командой:
```
kubectl apply -f kuber/
```
Установите и настройте [Helm](https://helm.sh/):
```
sudo snap install helm --classic
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install postgres-15 bitnami/postgresql
helm list
```

Следуя инструкциям, появившимся после создания пода базы данных, создайте создайте базу данных и пользователя:
```
CREATE DATABASE yourdbname;
CREATE USER youruser WITH ENCRYPTED PASSWORD 'yourpass';
GRANT ALL PRIVILEGES ON DATABASE yourdbname TO youruser;
ALTER USER youruser SUPERUSER;
```

Для применения миграций базы данных используйте команду:
```
kubectl apply -f kubernetes/test-django-migrate.yaml
```

Для создания суперпользователя используйте команду:
```
kubectl run ... -ti -- bash
```

Узнайте IP-адрес minikube:
```
minikube ip
```

Приложение будет доступно по адресу `star-burger.test` после добавления в файл `/etc/hosts` строки:
```
your_minikube_ip star-burger.test
```

Раз в месяц будет запускаться автоматическая очистка сессий Django-приложения.
Если возникла необходимость очистить сессии Django-приложения не по расписанию, то это можно сделать в ручную:
```
kubectl create job --from=cronjob/test-django-clearsessions any_job_name
```

Для применения миграций базы данных используйте команду:
```
kubectl apply -f kubernetes/test-django-migrate.yaml
```

После внесения изменений в конфигурационный файл `test-django-configmap.yaml` введите следующие команды:
```
kubectl apply -f kuber/
kubectl rollout restart -f kuber/
```
