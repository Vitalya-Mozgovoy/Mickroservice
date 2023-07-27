# Домашнее задание к занятию "Как работает сеть в K8S" 
Как работает сеть в K8S

### Цель задания

Настроить сетевую политику доступа к подам.

### Чеклист готовности к домашнему заданию

1. Кластер k8s с установленным сетевым плагином calico

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Calico](https://www.tigera.io/project-calico/)
2. [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
3. [About Network Policy](https://docs.projectcalico.org/about/about-network-policy)

-----

### Задание 1. Создать сетевую политику (или несколько политик) для обеспечения доступа

1. Создать deployment'ы приложений frontend, backend и cache и соответствующие сервисы.
2. В качестве образа использовать network-multitool.
3. Разместить поды в namespace app.
4. Создать политики, чтобы обеспечить доступ frontend -> backend -> cache. Другие виды подключений должны быть запрещены.
5. Продемонстрировать, что трафик разрешен и запрещен.

### Решение

Манифесты
 - [Frontend](file/main/10-frontend.yaml)
 
 - [Backend](file/main/20-backend.yaml)

 - [Cache](file/main/30-cache.yaml)

Создаем namespace
```bash
root@masterk8s:~/main# kubectl create namespace app
namespace/app created
root@masterk8s:~/main# kubectl get namespace
NAME              STATUS   AGE
app               Active   12s
default           Active   121m
kube-node-lease   Active   121m
kube-public       Active   121m
kube-system       Active   121m
```

Создаем deployment и svc
```bash
root@masterk8s:~/main# kubectl apply -f 10-frontend.yaml 
deployment.apps/frontend created
service/frontend created
root@masterk8s:~/main# kubectl apply -f 20-backend.yaml 
deployment.apps/backend created
service/backend created
root@masterk8s:~/main# kubectl apply -f 30-cache.yaml 
deployment.apps/cache created
service/cache created

root@masterk8s:~/main# kubectl get -n app deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
backend    1/1     1            1           29s
cache      1/1     1            1           21s
frontend   1/1     1            1           59s

root@masterk8s:~/main# kubectl config set-context --current --namespace=app

root@masterk8s:~/main# kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
backend-7f6ffd4fb4-jj5dv    1/1     Running   0          12m   10.233.83.2      worker3   <none>           <none>         
cache-7bb8f4764b-bzm7c      1/1     Running   0          11m   10.233.125.2     worker2   <none>           <none>         
frontend-85f54fff68-dqsrh   1/1     Running   0          12m   10.233.105.129   worker1   <none>           <none>

root@masterk8s:~/main# kubectl get svc -o wide
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR    
backend    ClusterIP   10.233.40.56    <none>        80/TCP    13m   app=backend 
cache      ClusterIP   10.233.53.246   <none>        80/TCP    13m   app=cache   
frontend   ClusterIP   10.233.60.123   <none>        80/TCP    13m   app=frontend
```

Выборочно проверяем доступность сервисов

```bash
root@masterk8s:~/main# kubectl exec frontend-85f54fff68-dqsrh -- curl backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed  
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0Praqma Network MultiTool (with NGINX) - backend-7f6ffd4fb4-jj5dv - 10.233.83.2
100    79  100    79    0     0  14319      0 --:--:-- --:--:-- --:--:-- 15800                                                                              
root@masterk8s:~/main# kubectl exec frontend-85f54fff68-dqsrh -- curl cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0Praqma Network MultiTool (with NGINX) - cache-7bb8f4764b-bzm7c - 10.233.125.2
100    78  100    78    0     0   9364      0 --:--:-- --:--:-- --:--:--  9750
root@masterk8s:~/main# kubectl exec cache-7bb8f4764b-bzm7c -- curl frontend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    83  100    83    0     0  13847      0 --:--:-- --:--:-- --:--:-- 16600
Praqma Network MultiTool (with NGINX) - frontend-85f54fff68-dqsrh - 10.233.105.129
```

Создаем сетевые политики, чтобы обеспечить выполнение пункта 4 данного задания
 - [NP-Default](file/network-policy/00-default.yaml)

 - [NP-Frontend](file/network-policy/10-frontend.yaml) - не обязательно, тест(запрещает доступ непосредственно к frontend)
 
 - [NP-Backend](file/network-policy/20-backend.yaml)
 
 - [NP-Cache](file/network-policy/30-cache.yaml)

Общее запрещающее правило. Сразу проверяем.
```bash
root@masterk8s:~/main# kubectl apply -f n-pol/00-default.yaml 
networkpolicy.networking.k8s.io/default-deny-ingress created
root@masterk8s:~/main# kubectl exec frontend-85f54fff68-dqsrh -- curl cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:02:10 --:--:--     0
curl: (28) Failed to connect to cache port 80 after 130383 ms: Operation timed out
command terminated with exit code 28

```

Разрешающие правила для наших подов

```bash

root@masterk8s:~/main# kubectl apply -f n-pol/20-backend.yaml 
networkpolicy.networking.k8s.io/backend created
root@masterk8s:~/main# kubectl apply -f n-pol/30-cache.yaml 
networkpolicy.networking.k8s.io/cache created

root@masterk8s:~/main# kubectl get networkpolicy
NAME                   POD-SELECTOR   AGE
backend                app=backend    11m
cache                  app=cache      11m
default-deny-ingress   <none>         18m
frontend               app=frontend   11m
```

Проверяем 

```bash
root@masterk8s:~/main# kubectl exec frontend-85f54fff68-dqsrh -- curl --max-time 10 backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    79  100    79    0     0   6779      0 --:--:-- --:--:-- --:--:--  7181
Praqma Network MultiTool (with NGINX) - backend-7f6ffd4fb4-jj5dv - 10.233.83.2
root@masterk8s:~/main# kubectl exec frontend-85f54fff68-dqsrh -- curl --max-time 10 cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
curl: (28) Connection timed out after 10000 milliseconds
command terminated with exit code 28

root@masterk8s:~/main# kubectl exec backend-7f6ffd4fb4-jj5dv -- curl --max-time 10 cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    78  100    78    0     0  11787      0 --:--:-- --:--:-- --:--:-- 13000
Praqma Network MultiTool (with NGINX) - cache-7bb8f4764b-bzm7c - 10.233.125.2
root@masterk8s:~/main# kubectl exec backend-7f6ffd4fb4-jj5dv -- curl --max-time 10 frontend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
curl: (28) Connection timed out after 10000 milliseconds
command terminated with exit code 28

root@masterk8s:~# kubectl exec cache-7bb8f4764b-bzm7c -- curl --max-time 10 frontend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
curl: (28) Connection timed out after 10001 milliseconds
command terminated with exit code 28
root@masterk8s:~# kubectl exec cache-7bb8f4764b-bzm7c -- curl --max-time 10 backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0
curl: (28) Connection timed out after 10001 milliseconds
command terminated with exit code 28
```
### Правила приема работы

1. Домашняя работа оформляется в своем Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md