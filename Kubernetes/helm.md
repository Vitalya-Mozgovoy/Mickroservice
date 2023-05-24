# Домашнее задание к занятию "Helm"
"Helm"

### Цель задания

В тестовой среде Kubernetes необходимо установить и обновить приложения с помощью Helm.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S)
2. Установленный локальный kubectl
3. Установленный локальный helm
4. Редактор YAML-файлов с подключенным github-репозиторием

------

Helm установлен 
```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm version
version.BuildInfo{Version:"v3.11.2", GitCommit:"912ebc1cd10d38d340f048efaf0abda047c3468e", GitTreeState:"clean", GoVersion:"go1.18.10"}
root@ansibleserv:~/helm/40-helm/01-templating/charts#
```


### Инструменты/ дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://helm.sh/docs/intro/install/) по установке Helm. [Helm completion](https://helm.sh/docs/helm/helm_completion/)

------

### Задание 1. Подготовить helm чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом/statefulset’ом/
3. В переменных чарта измените образ приложения для изменения версии.

#### Решение

- Скачиваем файлы для подготовки деплоя. Файлы, которые применялись в лекции.

```bash
root@ansibleserv:~/helm# git clone https://github.com/aak74/kubernetes-for-beginners.git
Cloning into 'kubernetes-for-beginners'...
remote: Enumerating objects: 983, done.
remote: Counting objects: 100% (236/236), done.
remote: Compressing objects: 100% (173/173), done.
remote: Total 983 (delta 72), reused 173 (delta 45), pack-reused 747
Receiving objects: 100% (983/983), 2.64 MiB | 3.10 MiB/s, done.
Resolving deltas: 100% (362/362), done.
root@ansibleserv:~/helm# cp /root/helm/kubernetes-for-beginners/40-helm/ /root/helm/
cp: -r not specified; omitting directory '/root/helm/kubernetes-for-beginners/40-helm/'
root@ansibleserv:~/helm# cp -R /root/helm/kubernetes-for-beginners/40-helm/ /root/helm/
root@ansibleserv:~/helm# rm -R kubernetes-for-beginners/
root@ansibleserv:~/helm# ls -l
total 4
drwxr-xr-x 5 root root 4096 Apr  8 19:32 40-helm
```

- Создаем шаблон на базе данных файлов

```bash
root@ansibleserv:~/helm# cd /root/helm/40-helm/01-templating/charts/
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm template 01-simple
---
# Source: hard/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: demo
  labels:
    app: demo
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: demo
---
# Source: hard/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo
  labels:
    app: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: hard
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          resources:
            limits:
              cpu: 200m
              memory: 256Mi
            requests:
              cpu: 100m
              memory: 128Mi
```
Получаем что helm при запуске создаст service(port 80) и deployment(nginx version 1.16.0) 

- В переменных чарта меняем образ приложения

В файле `Chart.yaml` меняем номер версии приложения

```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# nano 01-simple/Chart.yaml
root@ansibleserv:~/helm/40-helm/01-templating/charts# cat 01-simple/Chart.yaml
apiVersion: v2
name: hard
description: A minimal chart for demo

type: application

version: 0.1.2
appVersion: "1.19.0"
```

- Смотрим шаблон
```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm template 01-simple
---
# Source: hard/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: demo
  labels:
    app: demo
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: demo
---
# Source: hard/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo
  labels:
    app: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: hard
          image: "nginx:1.19.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          resources:
            limits:
              cpu: 200m
              memory: 256Mi
            requests:
              cpu: 100m
              memory: 128Mi
root@ansibleserv:~/helm/40-helm/01-templating/charts#
```

------
### Задание 2. Запустить 2 версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе;третью версию в namespace=app2.
3. Продемонстрируйте результат/

#### Решение

Проверяем
```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm template 01-simple
---
# Source: hard/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: demo
  labels:
    app: demo
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: demo
---
# Source: hard/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo
  labels:
    app: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: hard
          image: "nginx:1.19.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          resources:
            limits:
              cpu: 200m
              memory: 256Mi
            requests:
              cpu: 100m
              memory: 128Mi
root@ansibleserv:~/helm/40-helm/01-templating/charts#
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm install demo1 01-simple
NAME: demo1
LAST DEPLOYED: Sat Apr  8 20:22:24 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
---------------------------------------------------------

Content of NOTES.txt appears after deploy.
Deployed version 1.19.0.

---------------------------------------------------------
root@ansibleserv:~/helm/40-helm/01-templating/charts# kubectl get all
NAME                             READY   STATUS    RESTARTS      AGE
pod/myapp-pod-7d9b9c8bd5-5xpc7   1/1     Running   3 (76m ago)   11d
pod/demo-69c897c87-jdcrj         1/1     Running   0             29s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
service/kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP                         56d
service/np-mysvc     NodePort    10.152.183.210   <none>        9001:30080/TCP,9002:32080/TCP   44d
service/fe-svc       ClusterIP   10.152.183.119   <none>        80/TCP                          43d
service/be-svc       ClusterIP   10.152.183.234   <none>        80/TCP                          43d
service/myservice    NodePort    10.152.183.102   <none>        80:32000/TCP                    11d
service/demo         ClusterIP   10.152.183.112   <none>        80/TCP                          29s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp-pod   1/1     1            1           11d
deployment.apps/demo        1/1     1            1           30s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-pod-7d9b9c8bd5   1         1         1       11d
replicaset.apps/demo-69c897c87         1         1         1       30s

```

```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
demo1   default         1               2023-04-08 20:22:24.564257789 +0300 MSK deployed        hard-0.1.2      1.19.0
```

- Запустим несколько версий приложения

```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm upgrade demo1 --set replicaCount=3 01-simple
Release "demo1" has been upgraded. Happy Helming!
NAME: demo1
LAST DEPLOYED: Sat Apr  8 20:30:12 2023
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
---------------------------------------------------------

Content of NOTES.txt appears after deploy.
Deployed version 1.19.0.

---------------------------------------------------------
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
demo1   default         2               2023-04-08 20:30:12.959975194 +0300 MSK deployed        hard-0.1.2      1.19.0
root@ansibleserv:~/helm/40-helm/01-templating/charts# kubectl get pod
NAME                         READY   STATUS    RESTARTS      AGE
myapp-pod-7d9b9c8bd5-5xpc7   1/1     Running   3 (83m ago)   11d
demo-69c897c87-jdcrj         1/1     Running   0             8m9s
demo-69c897c87-tc2sp         1/1     Running   0             21s
demo-69c897c87-zghpf         1/1     Running   0             21s
```

- Удалим наш `helm demo1` чтобы потом создать новый в `namespace` из условия задачи

```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm uninstall demo1
release "demo1" uninstalled
root@ansibleserv:~/helm/40-helm/01-templating/charts# kubectl get pod
NAME                         READY   STATUS    RESTARTS       AGE
myapp-pod-7d9b9c8bd5-5xpc7   1/1     Running   3 (118m ago)   11d
```

- Создаем

```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm install demo2 --namespace app1 --create-namespace --wait --set replicaCount=2 01-simple                 NAME: demo2
LAST DEPLOYED: Sat Apr  8 21:26:22 2023
NAMESPACE: app1
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
---------------------------------------------------------

Content of NOTES.txt appears after deploy.
Deployed version 1.19.0.

---------------------------------------------------------

root@ansibleserv:~/helm/40-helm/01-templating/charts# helm install demo2 --namespace app2 --create-namespace --wait --set replicaCount=1 01-simple
NAME: demo2
LAST DEPLOYED: Sat Apr  8 21:28:59 2023
NAMESPACE: app2
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
---------------------------------------------------------

Content of NOTES.txt appears after deploy.
Deployed version 1.19.0.

---------------------------------------------------------
```
```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# kubectl get pod -n app1
NAME                   READY   STATUS    RESTARTS   AGE
demo-69c897c87-llrgv   1/1     Running   0          4m52s
demo-69c897c87-2wjfd   1/1     Running   0          4m52s
root@ansibleserv:~/helm/40-helm/01-templating/charts# kubectl get pod -n app2
NAME                   READY   STATUS    RESTARTS   AGE
demo-69c897c87-4jj8x   1/1     Running   0          2m18s
```

- Удалим хелмы

```bash
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm uninstall demo2 --namespace app1
release "demo2" uninstalled
root@ansibleserv:~/helm/40-helm/01-templating/charts# helm uninstall demo2 --namespace app2
release "demo2" uninstalled
```


### Правила приема работы

1. Домашняя работа оформляется в своем Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, `helm`, а также скриншоты результатов
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md