# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.
2. Создать Deployment приложения _backend_ из образа multitool. 
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

#### Решение

frontend:
- [Deployment](POD/Deployment2.yml)
- [Service](POD/Service3.yml)

backend:
- [Deployment](POD/Deployment3.yml)
- [Service](POD/Service.yml)

```
$ kubectl apply -f ./POD/Deployment2.yml
$ kubectl apply -f ./POD/Service3.yml

$ kubectl apply -f ./POD/Deployment3.yml
$ kubectl apply -f ./POD/Service4.yml

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
backend-69fdc9f5fb-bj6nf    1/1     Running   0          111s
backend-69fdc9f5fb-rmvl5    1/1     Running   0          111s
backend-69fdc9f5fb-swgzs    1/1     Running   0          111s
frontend-6bc548fcdb-hhpx9   1/1     Running   0          116s
frontend-6bc548fcdb-rrfvh   1/1     Running   0          116s
frontend-6bc548fcdb-t6bh2   1/1     Running   0          116s

kubectl exec -it backend-69fdc9f5fb-bj6nf -- curl frontend:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```
------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

#### Решение
В minikube Ingress-controller уже есть. Нужно включить.
```
minikube addons enable ingress
```

[Ingress](POD/Ingress.yml)

```
$ kubectl apply -f ./POD/Ingress.yml

$ kubectl get ingress
NAME         CLASS    HOSTS       ADDRESS        PORTS   AGE
my-ingress   <none>   my-domain.com   192.168.67.2   80      3m

$ kubectl describe ingress my-ingress
Name:             my-ingress
Labels:           <none>
Namespace:        default
Address:          192.168.67.2
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  my-domain.com   
              /      frontend:80 (10.244.0.93:80,10.244.0.94:80,10.244.0.95:80)
              /api   backend:8080 (10.244.0.96:8080,10.244.0.97:8080,10.244.0.98:8080)

```

Далее прописываем в hosts
```
192.168.67.2 my-domain.com
```

```
$ curl my-domain.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

$ curl my-domain.com/api
WBITT Network MultiTool (with NGINX) - backend-69fdc9f5fb-clj89 - 10.244.0.98 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.