# Проект: "Разработка личного кабинета Е-ОСАГО с использованием микросервисной архитектуры"

### Инструкция по запуску приложения

**Клонирум проект в локальный репозиторий**

 ```
 git clone https://github.com/Jony2Good/insurance-k8s.git

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
helm install prometheus prometheus-community/prometheus --namespace k8s-basics --create-namespace -f prometheus-values.yaml
```

*Пробрасываем порт наружу*

```
kubectl -n k8s-basics get svc
kubectl -n k8s-basics port-forward svc/prometheus-server
```


### Первый этап - устанавливаем namespace, запускаем API Gateway

**Инициализируем манифесты**

```
kubectl apply -f k8s/first/api-gateway
```

#### Подготовка к установке остальных сервисов

1. Установка ingress-nginx

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

#### Второй этап - поднимаем сервисы

**Инициализируем манифесты**

```
kubectl apply -f k8s/second/auth-service/
```

```
kubectl apply -f k8s/second/billing-service/
```

```
kubectl apply -f k8s/second/notification-service/
```

```
kubectl apply -f k8s/second/account-service/
```

```
kubectl apply -f k8s/second/calc-service/
```

```
kubectl apply -f k8s/second/policy-service/
```

```
kubectl apply -f k8s/second/storage-service/
```

```
kubectl apply -f k8s/second/verify-service/
```
