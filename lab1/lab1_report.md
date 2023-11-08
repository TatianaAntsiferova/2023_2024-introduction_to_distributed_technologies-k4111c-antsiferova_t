# 2023_2024-introduction_to_distributed_technologies-k4111c-antsiferova_t_a

University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023/2024
Group: K4111c
Author: Antsiferova Tatiana Anatolievna
Lab: Lab1
Date of create: 01.11.2023
Date of finished: 

## Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."

### Описание работы
Это первая лабораторная работа курса, в ходе которой предлагается установить и запустить Minikube с использованием Docker в качестве драйвера и развернуть первый под. 
### Цель работы
Ознакомиться с Docker и Minikube, создать манифест для запуска пода, запустить и протестировать под. 
### Ход работы
Установлен  Docker Desktop и Minikube на рабочий компьютер.
#### Запуск minikube cluster

```
😄  minikube v1.31.2 on Ubuntu 22.04
✨  Using the docker driver based on existing profile
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=3616MB) ...
🐳  Preparing Kubernetes v1.27.4 на Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
#### Запуск пода
Для запуска пода был создан манифест `vault-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
     name: vault
spec:
  containers:
    - name: vault
      image: vault:1.13.3
```
Далее с помощью следующей команды запускаем под:
```bash
minikube kubectl -- apply -f vault-pod.yaml
```
#### Создание сервиса для доступа к контейнеру
Для создания сервиса для доступа к контейнеру была использована следующая команда:
```bash
minikube kubectl -- expose pod vault --type=NodePort --port=8200
```
Для того, чтобы попасть в контейнер используем следующую команду:
```bash
minikube kubectl -- port-forward service/vault 8200:8200
```

После того, как minikube прокинул порт компьютера в контейнер, можно перейли в   vault по ссылке http://localhost:8200



