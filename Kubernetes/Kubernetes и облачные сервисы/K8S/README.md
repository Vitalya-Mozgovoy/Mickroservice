# Домашнее задание к занятию "Обновление приложений"

### Цель задания

Выбрать и настроить стратегию обновления приложения.

### Чеклист готовности к домашнему заданию

1. Кластер k8s.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
2. [Статья про стратегии обновлений](https://habr.com/ru/companies/flant/articles/471620/)

-----

### Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор.

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Какую стратегию обновления выберете и почему?

### Решение
Т.к. ресурсов очень мало остается две стратегии обновления rolling update и recreate.
Т.к. обновление мажорное, то используем recreate, но пользователям придется подождать, пока всё поднимется.
Rolling update не можем использовать, т.к. начнут валиться ошибки, из-за невозможности работать двум версиям сразу, из-за 
чего процесс обновления встанет, нам придется самим решать эту проблему.

### Задание 2. Обновить приложение.

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Кол-во реплик - 5.
2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
4. Откатиться после неудачного обновления.

### Решение
Создадим deployment приложения
```
$ kubectl apply -f ./src/deployment1.yml

$ kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginx-multitool   5/5     5            5           2m50s

$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
nginx-multitool-5d6f4d4f8-2gqg4   2/2     Running   0          3m6s
nginx-multitool-5d6f4d4f8-6cxj8   2/2     Running   0          3m6s
nginx-multitool-5d6f4d4f8-8wqvp   2/2     Running   0          74s
nginx-multitool-5d6f4d4f8-9qjxf   2/2     Running   0          74s
nginx-multitool-5d6f4d4f8-szl9h   2/2     Running   0          74s
```
Обновим версию nginx в приложении до версии 1.20. Увидим, что приложение доступно
```
$ kubectl apply -f ./src/deployment1.yml

kubectl get pods -w
NAME                               READY   STATUS    RESTARTS   AGE
nginx-multitool-54b8756496-k46sl   2/2     Running   0          17m
nginx-multitool-54b8756496-mxtxl   2/2     Running   0          16m
nginx-multitool-54b8756496-rqp6m   2/2     Running   0          17m
nginx-multitool-54b8756496-sqcgk   2/2     Running   0          16m
nginx-multitool-54b8756496-t972t   2/2     Running   0          17m
nginx-multitool-f7f77df6-gvwwb     0/2     Pending   0          0s
nginx-multitool-54b8756496-mxtxl   2/2     Terminating   0          17m
nginx-multitool-f7f77df6-gvwwb     0/2     Pending       0          0s
nginx-multitool-f7f77df6-mjpww     0/2     Pending       0          0s
nginx-multitool-f7f77df6-rdvcp     0/2     Pending       0          0s
nginx-multitool-f7f77df6-rdvcp     0/2     Pending       0          0s
nginx-multitool-f7f77df6-mjpww     0/2     Pending       0          0s
nginx-multitool-f7f77df6-sv5q9     0/2     Pending       0          0s
nginx-multitool-f7f77df6-7rcjq     0/2     Pending       0          0s
nginx-multitool-f7f77df6-7rcjq     0/2     Pending       0          0s
nginx-multitool-f7f77df6-sv5q9     0/2     Pending       0          0s
nginx-multitool-f7f77df6-gvwwb     0/2     ContainerCreating   0          0s
nginx-multitool-f7f77df6-rdvcp     0/2     ContainerCreating   0          0s
nginx-multitool-f7f77df6-mjpww     0/2     ContainerCreating   0          0s
nginx-multitool-f7f77df6-7rcjq     0/2     ContainerCreating   0          0s
nginx-multitool-f7f77df6-sv5q9     0/2     ContainerCreating   0          0s
nginx-multitool-54b8756496-mxtxl   0/2     Terminating         0          17m
nginx-multitool-54b8756496-mxtxl   0/2     Terminating         0          17m
nginx-multitool-54b8756496-mxtxl   0/2     Terminating         0          17m


$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-multitool-f7f77df6-7rcjq   2/2     Running   0          73s
nginx-multitool-f7f77df6-gvwwb   2/2     Running   0          73s
nginx-multitool-f7f77df6-mjpww   2/2     Running   0          73s
nginx-multitool-f7f77df6-rdvcp   2/2     Running   0          73s
nginx-multitool-f7f77df6-sv5q9   2/2     Running   0          73s

```
Попытаемся обновить приложение nginx до версии 28
```
$ kubectl apply -f ./src/deployment1.yml

$ kubectl get pods -w
NAME                             READY   STATUS    RESTARTS   AGE
nginx-multitool-f7f77df6-7rcjq   2/2     Running   0          6m50s
nginx-multitool-f7f77df6-gvwwb   2/2     Running   0          6m50s
nginx-multitool-f7f77df6-mjpww   2/2     Running   0          6m50s
nginx-multitool-f7f77df6-rdvcp   2/2     Running   0          6m50s
nginx-multitool-f7f77df6-sv5q9   2/2     Running   0          6m50s
nginx-multitool-5cd77d79fb-wwh96   0/2     Pending   0          0s
nginx-multitool-5cd77d79fb-wwh96   0/2     Pending   0          0s
nginx-multitool-5cd77d79fb-b54gd   0/2     Pending   0          0s
nginx-multitool-5cd77d79fb-cx97z   0/2     Pending   0          0s
nginx-multitool-5cd77d79fb-d9bxt   0/2     Pending   0          0s
nginx-multitool-5cd77d79fb-bhs2g   0/2     Pending   0          0s
nginx-multitool-f7f77df6-7rcjq     2/2     Terminating   0          6m55s
nginx-multitool-5cd77d79fb-b54gd   0/2     Pending       0          0s
nginx-multitool-5cd77d79fb-cx97z   0/2     Pending       0          0s
nginx-multitool-5cd77d79fb-d9bxt   0/2     Pending       0          0s
nginx-multitool-5cd77d79fb-bhs2g   0/2     Pending       0          0s
nginx-multitool-5cd77d79fb-wwh96   0/2     ContainerCreating   0          0s
nginx-multitool-5cd77d79fb-cx97z   0/2     ContainerCreating   0          0s
nginx-multitool-5cd77d79fb-b54gd   0/2     ContainerCreating   0          0s
nginx-multitool-5cd77d79fb-d9bxt   0/2     ContainerCreating   0          0s
nginx-multitool-5cd77d79fb-bhs2g   0/2     ContainerCreating   0          0s
nginx-multitool-f7f77df6-7rcjq     0/2     Terminating         0          6m56s
nginx-multitool-f7f77df6-7rcjq     0/2     Terminating         0          6m57s
nginx-multitool-f7f77df6-7rcjq     0/2     Terminating         0          6m57s

$ kubectl get pods
NAME                               READY   STATUS             RESTARTS   AGE
nginx-multitool-5cd77d79fb-b54gd   1/2     ImagePullBackOff   0          61s
nginx-multitool-5cd77d79fb-bhs2g   1/2     ImagePullBackOff   0          61s
nginx-multitool-5cd77d79fb-cx97z   1/2     ImagePullBackOff   0          61s
nginx-multitool-5cd77d79fb-d9bxt   1/2     ImagePullBackOff   0          61s
nginx-multitool-5cd77d79fb-wwh96   1/2     ImagePullBackOff   0          61s
nginx-multitool-f7f77df6-gvwwb     2/2     Running            0          7m56s
nginx-multitool-f7f77df6-mjpww     2/2     Running            0          7m56s
nginx-multitool-f7f77df6-rdvcp     2/2     Running            0          7m56s
nginx-multitool-f7f77df6-sv5q9     2/2     Running            0          7m56s

$ kubectl get pods
NAME                               READY   STATUS             RESTARTS   AGE
nginx-multitool-5cd77d79fb-b54gd   1/2     ErrImagePull       0          93s
nginx-multitool-5cd77d79fb-bhs2g   1/2     ImagePullBackOff   0          93s
nginx-multitool-5cd77d79fb-cx97z   1/2     ErrImagePull       0          93s
nginx-multitool-5cd77d79fb-d9bxt   1/2     ErrImagePull       0          93s
nginx-multitool-5cd77d79fb-wwh96   1/2     ImagePullBackOff   0          93s
nginx-multitool-f7f77df6-gvwwb     2/2     Running            0          8m28s
nginx-multitool-f7f77df6-mjpww     2/2     Running            0          8m28s
nginx-multitool-f7f77df6-rdvcp     2/2     Running            0          8m28s
nginx-multitool-f7f77df6-sv5q9     2/2     Running            0          8m28s

```
Откатим
```
$ kubectl rollout history deployment/nginx-multitool
deployment.apps/nginx-multitool 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
4         <none>

$ kubectl rollout undo deployment/nginx-multitool
deployment.apps/nginx-multitool rolled back

$ kubectl rollout history deployment/nginx-multitool
deployment.apps/nginx-multitool 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
4         <none>
5         <none>

$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-multitool-f7f77df6-gvwwb   2/2     Running   0          26m
nginx-multitool-f7f77df6-h8w85   2/2     Running   0          58s
nginx-multitool-f7f77df6-mjpww   2/2     Running   0          26m
nginx-multitool-f7f77df6-rdvcp   2/2     Running   0          26m
nginx-multitool-f7f77df6-sv5q9   2/2     Running   0          26m

```

Далее вернул версию nginx для deployment на первоначальную 19 для удобства проверки работы.

## Дополнительные задания (со звездочкой*)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.**   Их выполнение поможет глубже разобраться в материале.   
Задания под звёздочкой дополнительные (необязательные к выполнению) и никак не повлияют на получение вами зачета по этому домашнему заданию. 

### Задание 3*. Создать Canary deployment.

1. Создать 2 deployment'а приложения nginx.
2. При помощи разных ConfigMap сделать 2 версии приложения (веб-страницы).
3. С помощью ingress создать канареечный деплоймент, чтобы можно было часть трафика перебросить на разные версии приложения.

### Правила приема работы

1. Домашняя работа оформляется в своем Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md
