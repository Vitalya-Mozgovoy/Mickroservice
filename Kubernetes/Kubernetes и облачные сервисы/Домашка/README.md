# Домашнее задание к занятию "Установка Kubernetes"

### Цель задания

Установить кластер K8s.

### Чеклист готовности к домашнему заданию

1. Развернутые ВМ с ОС Ubuntu 20.04-lts


### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция по установке kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
2. [Документация kubespray](https://kubespray.io/)

-----

### Задание 1. Установить кластер k8s с 1 master node

1. Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
2. В качестве CRI — containerd.
3. Запуск etcd производить на мастере.
4. Способ установки выбрать самостоятельно.

### Решение

Создал VM kubeadm1 вручную через yc, будем считать её мастер нодой.

Создал VM kubeadm2 вручную через yc, будем считать ее воркер нодой
```
yc compute instance list
+----------------------+----------+---------------+---------+----------------+-------------+
|          ID          |   NAME   |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP |
+----------------------+----------+---------------+---------+----------------+-------------+
| epd0vuaicn6bht7s8vu1 | kubeadm1 | ru-central1-b | RUNNING | 51.250.107.104 | 10.129.0.20 |
| epd1j6dshhoadh4r66op | kubeadm2 | ru-central1-b | RUNNING | 158.160.24.201 | 10.129.0.34 |
+----------------------+----------+---------------+---------+----------------+-------------+
```

Подготавливаю все необходимое для мастер ноды
```
ssh somov@51.250.107.104

sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl containerd
sudo apt-mark hold kubelet kubeadm kubectl

sudo -i
modprobe br_netfilter
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-arptables=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables=1" >> /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
logout
```

Инициирую мастер ноду
```
sudo kubeadm init \
--apiserver-advertise-address=10.129.0.20 \
--pod-network-cidr 10.244.0.0/16 \
--apiserver-cert-extra-sans=51.250.107.104
```

Донастраиваю работу с конфигами матер ноды
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Проверяю работоспособность.
```
kubectl get nodes
NAME       STATUS     ROLES           AGE     VERSION
kubeadm1   NotReady   control-plane   9m34s   v1.27.3
```

Подготавливаю все необходимое для воркер ноды, точно также, как для мастер, и 
инициирую воркер ноду, соединяя её с мастер нодой.
```
sudo kubeadm join 10.129.0.20:6443 --token lyo5z0.dxsjycutgk7jj56p \
        --discovery-token-ca-cert-hash sha256:19d6c324b72dfbc0a274dd5d7c1ab02d2371e75888ca91920d234d5c4e0efac9
```

Проверяем работоспособность.
```
$ kubectl get nodes
NAME       STATUS     ROLES           AGE   VERSION
kubeadm1   NotReady   control-plane   28m   v1.27.3
kubeadm2   NotReady   <none>          10m   v1.27.3
```

Ищем проблему, связанную с NotReady
```
kubectl describe node kubeadm1
```

Находим плагин для работы с сетью, устанавливаем его
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

Проверяем работоспособность.
```
$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
kubeadm1   Ready    control-plane   28m   v1.27.3
kubeadm2   Ready    <none>          11m   v1.27.3
```

Проделываем тоже самое ещё 3 раза, т.е. создание VM как воркер ноды, и 
проверяем работоспособность.
```
$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
kubeadm1   Ready    control-plane   59m     v1.27.3
kubeadm2   Ready    <none>          42m     v1.27.3
kubeadm3   Ready    <none>          6m52s   v1.27.3
kubeadm4   Ready    <none>          3m19s   v1.27.3
kubeadm5   Ready    <none>          21s     v1.27.3

```

## Дополнительные задания (со звездочкой*)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.**   Их выполнение поможет глубже разобраться в материале.   
Задания под звёздочкой дополнительные (необязательные к выполнению) и никак не повлияют на получение вами зачета по этому домашнему заданию. 

------
### Задание 2*. Установить HA кластер

1. Установить кластер в режиме HA
2. Использовать нечетное кол-во Master-node
3. Для cluster ip использовать keepalived или другой способ

### Правила приема работы

1. Домашняя работа оформляется в своем Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl get nodes`, а также скриншоты результатов
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md
