# Задание
*Цель*: Добавить в приложение аутентификацию и регистрацию пользователей

### Задача

**Реализовать сценарий "Изменение и просмотр данных в профиле клиента".**
 - Пользователь регистрируется.
 - Заходит под собой и по определенному урлу получает данные о своем профиле
 - Может поменять данные в профиле.
 - Данные профиля для чтения и редактирования не должны быть доступны другим клиентам (аутентифицированным или нет).

#### На выходе должны быть предоставлена

1. Описание архитектурного решения и схема взаимодействия сервисов (в виде картинки).
2. Команда установки приложения.
3. Спецификация по методам api.

##### Как сделать запросы к приложению с помощью POSTMAN?

  - импортируем файл openapi.yaml в POSTMAN (выбираем чек-бокс Postman collection). 
  - название коллекции "Тестовый сервис"

------------

### Результат
1. [Схема][1]
2. Команда установки состоит из двух этапов. Ниже по тексту
3. [Спецификация][2]
------------
### Инструкция по запуску приложения

**Клонирум проект в локальный репозиторий**

 ```
 git clone https://github.com/Jony2Good/k8s-api-gateway.git

```
**Стартуем minikube**
```
minikube start
```

**Устанавливаем Prometheus стэк из helm репозитория**
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```
helm repo update
```

```
helm install prometheus prometheus-community/kube-prometheus-stack --namespace k8s-basics --create-namespace
```

#### Первый этап - устанавливаем namespace, запускаем API Gateway

**Инициализируем манифесты**

```
kubectl apply -f k8s/api-gateway
```

### Установка ingress-nginx + конфигурация Prometheus

**Получаем нужный репозиторий из helm**

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

**Устанавливаем ingress-controller в namespase k8s-basics и конфигурируем Prometheus**

```
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --namespace k8s-basics --set controller.metrics.enabled=true --set-string controller.podAnnotations."prometheus.io/scrape"="true" --set-string controller.podAnnotations."prometheus.io/port"="10254"
```

**Проверяем services**

```
kubectl get svc -n k8s-basics
```

Должна появится таблица. Нас интересует только ingress-nginx-controller. У него должен быть *Type:LoadBalanser* и *External-IP:pending*

| NAME                    | TYPE         | CLUSTER-IP     | EXTERNAL-IP    | PORT(S)                    | AGE |
| ----------------------- | ------------ | -------------- | -------------- | -------------------------- | --- |
| ngress-nginx-controller | LoadBalancer | 10.104.118.219 |  pending  | 80:31047/TCP,443:31617/TCP | 95m |


### Проброс ingress-nginx наружу

**Нам необходимо, чтобы домен arch.homework открывался без порта и по специальному url. Для этого мы должны прописать команду:**

```
minikube tunnel
```

Далее снова смотрим в таблицу на значение в колонке **EXTERNAL-IP** (вместо pending должно появится значение, к примеру, 10.107.105.245) в ней должен прописаться внешний IP-адрес.

```
kubectl get svc -n k8s-basics
```

Копируем данный внешний адрес и прописываем в файле hosts машины правило маршрутизации, где развернут кластер:

```
10.107.105.245 arch.homework
```
Справка: для ОС windows прописываем в файле и сохраняем:
```
C:\Windows\System32\drivers\etc\hosts
```

#### Второй этап - устанавливаем auth-service

**Инициализируем манифесты**

```
kubectl apply -f k8s/auth-service
```

[1]: https://github.com/Jony2Good/assets/blob/main/gateway-schema.png "Схема"
[2]: https://github.com/Jony2Good/k8s-helm/blob/main/openapi.yaml "Спецификация"