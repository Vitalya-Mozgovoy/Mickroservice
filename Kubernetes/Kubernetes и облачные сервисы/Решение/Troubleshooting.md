# Домашнее задание к занятию "Troubleshooting"

### Цель задания

Устранить неисправности при деплое приложения.

### Чеклист готовности к домашнему заданию

1. Кластер k8s.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить.

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.

### Решение
Убедимся, что кластер работает
```
$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
kubeadm1   Ready    control-plane   4m53s   v1.27.3
kubeadm2   Ready    <none>          57s     v1.27.3
```

Установим приложение по вышеописанной команде
```
$ kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "web" not found
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "data" not found
Error from server (NotFound): error when creating "https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml": namespaces "data" not found
```

Не хватает namespace data и web, создадим их
```
$ kubectl create namespace data
namespace/data created

$ kubectl create namespace web
namespace/web created
```
Выполним команду ещё раз
```
$ kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
deployment.apps/web-consumer created
deployment.apps/auth-db created
service/auth-db created
```

Убедимся, что кластер работает
```
$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
kubeadm1   Ready    control-plane   11m     v1.27.3
kubeadm2   Ready    <none>          7m14s   v1.27.3
```

Проверим deployments
```
$ kubectl get deployment -A
NAMESPACE     NAME           READY   UP-TO-DATE   AVAILABLE   AGE
data          auth-db        1/1     1            1           11m
kube-system   coredns        2/2     2            2           20m
web           web-consumer   2/2     2            2           11m
```

Убедимся в работе подов
```
kubectl get pods -A
NAMESPACE      NAME                               READY   STATUS    RESTARTS   AGE
data           auth-db-864ff9854c-vmxnh           1/1     Running   0          83s
kube-flannel   kube-flannel-ds-xf479              1/1     Running   0          9m11s
kube-flannel   kube-flannel-ds-z7c69              1/1     Running   0          6m22s
kube-system    coredns-5d78c9869d-jcmw5           1/1     Running   0          10m
kube-system    coredns-5d78c9869d-zz7nw           1/1     Running   0          10m
kube-system    etcd-kubeadm1                      1/1     Running   0          10m
kube-system    kube-apiserver-kubeadm1            1/1     Running   0          10m
kube-system    kube-controller-manager-kubeadm1   1/1     Running   0          10m
kube-system    kube-proxy-gv4cr                   1/1     Running   0          10m
kube-system    kube-proxy-pmsts                   1/1     Running   0          6m22s
kube-system    kube-scheduler-kubeadm1            1/1     Running   0          10m
web            web-consumer-84fc79d94d-4hpmp      1/1     Running   0          83s
web            web-consumer-84fc79d94d-n89sr      1/1     Running   0          83s
```

Проверим логи приложений
```
$ kubectl logs auth-db-864ff9854c-vmxnh -n data
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up

$ kubectl logs web-consumer-84fc79d94d-4hpmp -n web
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
...
```
Приложение web-consumer не видит приложение auth-db. Странно...


### Правила приема работы

1. Домашняя работа оформляется в своем Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md

