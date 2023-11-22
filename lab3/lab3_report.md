# 2023_2024-introduction_to_distributed_technologies-k4111c-antsiferova_t_a

University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111c
Author: Antsiferova Tatiana Anatolievna
Lab: Lab3
Date of create: 22.11.2023
Date of finished: 

## Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."
### Описание работы
В данной лабораторной работе предлагается познакомится с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.
### Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.
### Ход работы
#### Запуск minikube cluster
Аналогично предыдущим лабораторным работам запускаем minikube:

```bash
😄  minikube v1.31.2 on Ubuntu 22.04 
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔄  Updating the running docker "minikube" container ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
#### Создание манифеста `lab3.yaml`:
В созданном манифесте `lab3.yaml` находятся : `configMap` с двумя переменными `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` со значениями  `Antsiferova` и `ITMO` соотвественно; `Deployment` c `ReplicaSet` и `Service` 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
data:
  REACT_APP_USERNAME: "Antsiferova"
  REACT_APP_COMPANY_NAME: "ITMO"

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-frontend
  labels:
    app: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: ifilyaninitmo/itdt-contained-frontend:master
        envFrom:
          - configMapRef:
              name: config
        ports:
          - containerPort: 3000
          
---
          
apiVersion: v1
kind: Service
metadata:
  name: service-frontend
spec:
  selector:
    app: frontend
  type: NodePort
  ports:
    - port: 3000
      protocol: TCP
      name: http
      targetPort: 3000
```
#### Создание Ingress
Включяем `minikube addons enable ingress`:

```bash
ganeev@ganeev-HVY-WXX9:~/Документы/ВРТ/L_3$ minikube addons enable ingress  
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.8.1
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```
Сгенерируем TLS сертификат и импортируем его в minikube, добавив также в Ingress:
```bash
