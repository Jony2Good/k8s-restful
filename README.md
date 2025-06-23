# Задание
*Цель*: Реализовать сервис заказа, биллинга, нотификации. Настроить их взаимодействие с помощью брокера сообщений

### Задача

**Реализовать сценарий "Изменение и просмотр данных в профиле клиента".**
 - При регистрации пользователя создается аккаунт в сервисе биллинга. В сервисе биллинга должна быть возможность положить деньги на аккаунт и снять деньги.
 - Сервис нотификаций позволяет отправить сообщение на email. И позволяет получить список сообщений по методу API.
 - Пользователь может создать заказ. У заказа есть параметр - цена заказа.
 - Заказ происходит в 2 этапа: 1) сначала снимаем деньги с пользователя с помощью сервиса биллинга 2) отсылаем пользователю сообщение на почту с результатами оформления заказа.
 - Спроектировать взаимодействие сервисов при создании заказов. 

#### На выходе должны быть предоставлена

1. Описание архитектурного решения и схема взаимодействия сервисов (в виде картинки).
2. Команда установки приложения.
3. Спецификация по методам api.

##### Как сделать запросы к приложению с помощью POSTMAN?

  - импортируем файл openapi.yaml в POSTMAN (выбираем чек-бокс Postman collection). 
  - название коллекции "Тестовый сервис"

------------

### Результат
1. [Схема-картинка с работой сервисов][1]
2. [Описание работы сервисов ][3]
3. Команда установки состоит из двух этапов. Ниже по тексту
4. [Спецификация][2]
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

### Первый этап - устанавливаем namespace, запускаем API Gateway

**Инициализируем манифесты**

```
kubectl apply -f k8s/api-gateway
```

#### Подготовка к установке остальных сервисов

1. Установка ingress-nginx + конфигурация Prometheus

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


2. Проброс ingress-nginx наружу

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

3. Установка брокера сообщений RabbitMС
   
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

```
helm install rabbitmq bitnami/rabbitmq \
  --namespace k8s-basics \
  --set auth.username=guest \
  --set auth.password=guest \
  --set auth.vhost=/ \
  --set service.type=ClusterIP
```

Проверяем запуск RabbitMQ

```
kubectl get pods -n k8s-basics
```

#### Второй этап - поднимаем сервисы

**Инициализируем манифесты**

```
kubectl apply -f k8s/
```

[1]: https://github.com/Jony2Good/k8s-restful/blob/main/restful-schema.png "Схема-картинка"
[2]: https://github.com/Jony2Good/k8s-restful/blob/main/openapi.yaml "Спецификация"
[3]: https://github.com/Jony2Good/k8s-restful/blob/main/restful-description.md "Описание работы сервисов"