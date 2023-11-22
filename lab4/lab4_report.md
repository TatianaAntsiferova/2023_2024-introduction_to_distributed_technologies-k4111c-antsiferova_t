# 2023_2024-introduction_to_distributed_technologies-k4111c-antsiferova_t_a

University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111c
Author: Antsiferova Tatiana Anatolievna
Lab: Lab4
Date of create: 22.11.2023
Date of finished: 

## Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS."
### Описание работы
В данной лабораторной работе предлагается познакомится с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают `underlay` и `overlay` сети, а управление может быть организованно различными CNI.
### Цель работы
Познакомиться с CNI Calico и функцией `IPAM Plugin`, изучить особенности работы CNI и CoreDNS.
### Ход работы
#### Запуск minikube cluster
Запускаем minikube, установив в него плагин `CNI Calico` и режим работы `Multi-Node Clusters` одновеременно для создания двух нод в кластере:
```
minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```
```bash
😄  [multinode-demo] minikube v1.31.2 on Ubuntu 22.04 
✨  Using the docker driver based on existing profile
❗  With --network-plugin=cni, you will need to provide your own CNI. See --cni flag as a user-friendly alternative
📌  Using Docker driver with root privileges
❗  For an improved experience it's recommended to use Docker Engine instead of Docker Desktop.
Docker Engine installation instructions: https://docs.docker.com/engine/install/#server
👍  Starting control plane node multinode-demo in cluster multinode-demo
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring Calico (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node multinode-demo-m02 in cluster multinode-demo
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.58.2
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.58.2
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "multinode-demo" cluster and "default" namespace by default
```
Проверим ноды:
```
kubectl get nodes
```
![](/lab4/images/image1.png)

Проверка количества подов Calico:
```
kubectl get pods -l k8s-app=calico-node -A
```
![](/lab4/images/image2.png)

Количество подов Calico совпадает с количеством нод.

#### Помечаем ноды
Помечаем каждую ноду по геграфическому положению:
```
kubectl label nodes multinode-demo zone=east  
kubectl label nodes multinode-demo-m02 zone=west
```
![](/lab4/images/image3.png)

Устанавливаем `calicoctl`  и выполняем 
```
kubectl create -f calicoctl.yaml
```
![](/lab4/images/image4.png)

Перед созданием IPPools, удаляем созданные по-умолчанию:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippools -o wide
kubectl delete ippools default-ipv4-ippool
```
![](/lab4/images/image5.png)

Создаем и проверяем новые IPPools:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch create -f - < l_4-ippool.yaml
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippool -o wide
```
![](/lab4/images/image6.png)

#### Создание манифеста `deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: l4-deployment
  labels:
    app: l4-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: l4-frontend
  template:
    metadata:
      labels:
        app: l4-frontend
    spec:
      containers:
      - name: frontend-container
        image: ifilyaninitmo/itdt-contained-frontend:master
        ports:
        - containerPort: 3000
        env:
        - name: REACT_APP_USERNAME
          value: Antsiferova
        - name: REACT_APP_COMPANY_NAME
          value: ITMO
```

```
kubectl apply -f deployment.yaml 
``` 
#### Создание манифеста `service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: l4-service
spec:
  selector:
    app: l4-frontend
  ports:
    - port: 3000
      targetPort: 3000
  type: LoadBalancer
```
```
kubectl apply -f service.yaml 
``` 
![](/lab4/images/image7.png)

#### Проверяем IP:
```
kubectl get pods -o wide
```
![](/lab4/images/image8.png)

#### Прокидываем порт
```
kubectl port-forward service/l4-service 8200:3000
```
![](/lab4/images/image9.png)

Переменные ` Container name`  и ` Container IP`  могут меняться, в зависимости от пода, на который попал запрос.

####  Зайдем в "под" и попробуем попинговать.
```
kubectl exec -ti l4-deployment-76f8cfb86d-nkv4v -- sh
```
```
kubectl exec -ti l4-deployment-76f8cfb86d-h9wzr -- sh
```
![](/lab4/images/image10.png)
![](/lab4/images/image11.png)

#### Схема
