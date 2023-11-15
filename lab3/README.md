University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies) \
Year: 2023/2024 \
Group: K4113с \
Author: Nesterenko Mikhail Yurievich \
Lab: Lab3 \
Date of create: 15.11.2023 \
Date of finished: <none>

## Скачивание образа приложения
    docker pull ifilyaninitmo/itdt-contained-frontend:master

## Подключаем ingress
    $ minikube addons enable ingress
```
* Verifying ingress addon...
* The 'ingress' addon is enabled
```

## Создаем сертификат
    openssl req -x509 -nodes -days 999 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=qveex.dev"

## Публичный ключ
    $ cat tls.crt
```
-----BEGIN CERTIFICATE-----
***************************
-----END CERTIFICATE-----
```

## Приватный ключ
    $ cat tls.key
```
-----BEGIN PRIVATE KEY-----
***************************
-----END PRIVATE KEY-----
```

## Создаем config map
``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-map
data:
  REACT_APP_USERNAME: "qveex"
  REACT_APP_COMPANY_NAME: "qveex inc"
```

## Создаем replica set и его сервиса
``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-set-react
spec:
  replicas: 2
  selector:
    matchLabels:
      app: replica-set-react
  template:
    metadata:
      labels:
        app: replica-set-react
    spec:
      containers:
        - name: app-react
          image: ifilyaninitmo/itdt-contained-frontend:master
          envFrom:
            - configMapRef:
                name: config-map

---

apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      protocol: TCP
  selector:
    app: replica-set-react
```

## Создаем secret и ingress
``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: kubernetes.io/tls
stringData:
  tls.crt: |
    -----BEGIN CERTIFICATE-----
    ***************************
    -----END CERTIFICATE-----
  tls.key: |
    -----BEGIN PRIVATE KEY-----
    ***************************
    -----END PRIVATE KEY-----

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  tls:
    - hosts:
        - qveex.dev
      secretName: app-secret
  rules:
    - host: qveex.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 3000
```

## Добавим в файл hosts строку
    192.168.0.150 qveex.dev

## Создание config map
    kubectl apply -f configMap.yaml

## Создание replica set
    kubectl apply -f replicaSet.yaml

## Создание ingress
    kubectl apply -f ingress.yaml

## Узнаем заупущенные объекты

    kubectl get all
```
NAME                      READY   STATUS    RESTARTS      AGE
replica-set-react-7vwqg   1/1     Running   0             9m17s
replica-set-react-xcz9d   1/1     Running   0             9m5s
vault                     1/1     Running   3 (27m ago)   28d

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/app-service   NodePort    10.109.236.113   <none>        3000:32298/TCP   27d

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/replica-set-react   2         2         2       161m
```

## Включаем туннелирование
    $ minikube tunnel
```
* Tunnel successfully started
* Starting tunnel for service app-ingress.
```

## Открываем сайт
![1](front.png)

## Сертификат
![2](crt.png)

## Схема организации контейнеров и сервисов
![3](lab3.png)