# Выполнено ДЗ №2 (ReplicaSet, Deployment, DaemonSet)

 - [x] Основное ДЗ
 - [x] Задание со *
 - [x] Задание со **

## В процессе сделано:
 - Поднят кластер kind с 1 Control-plane node и 3 worker nodes
 - Создана replicaset сервиса frontend c 1 репликой, версия микросервиса 0.0.1, [frontend-replicaset.yaml](frontend-replicaset.yaml)
 - Увеличено количество реплик во frontend с 1 до 3, как через ad-hoc так и через манифест frontend-replocaset.yaml
 - Обновлена версия микросервиса frontend до верси 0.0.2 с помощью манифеста frontend-replocaset.yaml. Обновление происходит после применения нового манифеста с версией 0.0.2 и после удаления подов.
 - После удаления подов c меткой frontend они восстанавливаются с версией образа указанной в шаблоне контейнера рабочего replicaset.
 - Созданы образы для сервиса paymentservice [svs123/paymentservice](https://hub.docker.com/r/svs123/paymentservice/tags) c версиями 0.0.1 и 0.0.2
 - Создан replicaset c 3 репликами для сервиса paymentservice, [paymentservice-replicaset.yaml](paymentservice-replicaset.yaml)
 - Создан deployment для сервиса paymentservice, [paymentservice-deployment.yaml](paymentservice-deployment.yaml)
 - Обновлен сервис paymentservice до версии 0.0.2 с помощью deployment
 - Сделан откат сервиса paymentservice к версии 0.0.1
 - (*) Сделан deployment для сервиса paymentservice (аналог blue-green), [paymentservice-deployment-bg.yaml](paymentservice-deployment-bg.yaml)
 - (*) Сделан deployment для сервиса paymentservice (Reverse Rolling Update), [paymentservice-deployment-reverse.yaml](paymentservice-deployment-reverse.yaml)
 - Создан deployment для сервиса frontend c пробами, [frontend-deployment.yaml](frontend-deployment.yaml)
 - Сделан тест с некорректными пробами у deployment сервиса frontend. С проверкой статуса rollout по timeout. Если за отведенный таймаут сервис не откатиться команда проверки статуса rollout вернет exit code 1
 - (*) Создан DaemonSet для node-exporter, [node-exporter-daemonset.yaml](node-exporter-daemonset.yaml)
 - (**) Для daemonset node-exporter добавлены tolerations (допуски) для разворачивания на ноде control-plane

## Как запустить проект:
 - Пример запуска кластера kind

    ```
    echo """kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker""" > kind-config.yaml

    $ kind create cluster --config kind-config.yaml
    Creating cluster "kind" ...
    ✓ Ensuring node image (kindest/node:v1.27.3) �
    ✓ Preparing nodes � � � �
    ✓ Writing configuration �
    ✓ Starting control-plane �️
    ✓ Installing CNI �
    ✓ Installing StorageClass �
    ✓ Joining worker nodes �
    Set kubectl context to "kind-kind"
    You can now use your cluster with:

    kubectl cluster-info --context kind-kind

    Have a nice day! �
    vssadovnikov@kind:~$ kubectl get nodes -A
    NAME                 STATUS   ROLES           AGE   VERSION
    kind-control-plane   Ready    control-plane   44s   v1.27.3
    kind-worker          Ready    <none>          25s   v1.27.3
    kind-worker2         Ready    <none>          24s   v1.27.3
    kind-worker3         Ready    <none>          22s   v1.27.3
    vssadovnikov@kind:~$ kubectl get nodes
    NAME                 STATUS   ROLES           AGE   VERSION
    kind-control-plane   Ready    control-plane   56s   v1.27.3
    kind-worker          Ready    <none>          37s   v1.27.3
    kind-worker2         Ready    <none>          36s   v1.27.3
    kind-worker3         Ready    <none>          34s   v1.27.3
    ```
 - Создание replicaset для frontend, 1 реплика

    ```
    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl apply -f kubernetes-controllers/frontend-replicaset.yaml
    replicaset.apps/frontend created

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get rs
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   1         1         0       11s

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get po
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-rlndf   1/1     Running   0          33s
    ```
 - Поднимаем количество реплик до 3 у сервиса frontend с помощью replicaset (ad-hoc/манифест)

    ```
    # ad-hoc
    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl scale replicaset frontend --replicas=3
    replicaset.apps/frontend scaled

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get po
    NAME             READY   STATUS              RESTARTS   AGE
    frontend-nnndx   0/1     ContainerCreating   0          2s
    frontend-rlndf   1/1     Running             0          93s
    frontend-wpl92   0/1     ContainerCreating   0          2s

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get po
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-nnndx   1/1     Running   0          28s
    frontend-rlndf   1/1     Running   0          119s
    frontend-wpl92   1/1     Running   0          28s

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get rs frontend
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   3         3         3       2m12s

    # манифест

    # Установлена 1 реплика
    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl apply -f kubernetes-controllers/frontend-replicaset.yaml
    replicaset.apps/frontend configured

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get rs
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   1         1         1       6m49s


    # Увеличено до 3 реплик
    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl apply -f kubernetes-controllers/frontend-replicaset.yaml
    replicaset.apps/frontend configured

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get rs
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   3         3         1       7m17s

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get rs
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   3         3         3       7m37s
    ```
 - Обновление версии сервиса frontend с помощью replicaset

    ```
    ~/VladimirSVS_platform(kubernetes-controllers)$ docker tag svs123/frontend-boutique:0.0.1 svs123/frontend-boutique:0.0.2

    ~/VladimirSVS_platform(kubernetes-controllers)$ docker push svs123/frontend-boutique:0.0.2
    The push refers to repository [docker.io/svs123/frontend-boutique]
    54e89fccaee3: Layer already exists
    f70dc8f17b80: Layer already exists
    8b7d833255cf: Layer already exists
    e86b4b767b43: Layer already exists
    835e2fb08128: Layer already exists
    4693057ce236: Layer already exists
    0.0.2: digest: sha256:74383f41136d214301cabb8f89cabd3a27ece23a3b6cf03930b3a718c3af713d size: 1577

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get pod -l app=frontend -o=json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/frontend-boutique:0.0.1"
    "svs123/frontend-boutique:0.0.1"
    "svs123/frontend-boutique:0.0.1"

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl apply -f kubernetes-controllers/frontend-replicaset.yaml | kubectl get pods -l app=frontend -w
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-ph2hx   1/1     Running   0          5m58s
    frontend-qkq9g   1/1     Running   0          5m58s
    frontend-xxhg7   1/1     Running   0          10m

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get rs frontend -o json | jq '.spec.template.spec.containers[0].image'
    "svs123/frontend-boutique:0.0.2"

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get pod -l app=frontend -o=json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/frontend-boutique:0.0.1"
    "svs123/frontend-boutique:0.0.1"
    "svs123/frontend-boutique:0.0.1"

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl delete pod -l app=frontend
    pod "frontend-ph2hx" deleted
    pod "frontend-qkq9g" deleted
    pod "frontend-xxhg7" deleted

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get po
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-c4wd9   0/1     Running   0          5s
    frontend-m2m7n   0/1     Running   0          5s
    frontend-m4c69   0/1     Running   0          5s

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get po
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-c4wd9   1/1     Running   0          29s
    frontend-m2m7n   1/1     Running   0          29s
    frontend-m4c69   1/1     Running   0          29s

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get rs frontend -o json | jq '.spec.template.spec.containers[0].image'
    "svs123/frontend-boutique:0.0.2"

    ~/VladimirSVS_platform(kubernetes-controllers)$ kubectl get pod -l app=frontend -o=json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/frontend-boutique:0.0.2"
    "svs123/frontend-boutique:0.0.2"
    "svs123/frontend-boutique:0.0.2"
    ```

 - Создание образов для paymentservice

    ```
    ~/microservices-demo/src/paymentservice(main)$ docker build -t svs123/paymentservice:0.0.1 .

    ~/microservices-demo/src/paymentservice(main)$ docker image ls
    REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
    svs123/paymentservice      0.0.1     f89909ff532d   13 seconds ago   300MB
    frontend-boutique          0.0.1     1ea520e0393a   25 hours ago     42.4MB
    svs123/frontend-boutique   0.0.1     1ea520e0393a   25 hours ago     42.4MB
    svs123/frontend-boutique   0.0.2     1ea520e0393a   25 hours ago     42.4MB
    svs123/web-server          0.0.1     c79ee4764bc6   47 hours ago     187MB
    kindest/node               <none>    89e7dc9f9131   3 months ago     932MB

    ~/microservices-demo/src/paymentservice(main)$ docker tag svs123/paymentservice:0.0.1 svs123/paymentservice:0.0.2

    ~/microservices-demo/src/paymentservice(main)$ docker push svs123/paymentservice:0.0.1
    The push refers to repository [docker.io/svs123/paymentservice]
    7fd8f7b945b3: Pushed
    af291a43d5a0: Pushed
    a04d719af9d7: Pushed
    5bbcb46a3da5: Mounted from library/node
    9a09a6062e1b: Mounted from library/node
    122fc55b6d78: Mounted from library/node
    4693057ce236: Mounted from svs123/frontend-boutique
    0.0.1: digest: sha256:a4d53ac559086d0f7cc8f510130cfd28a9310d0a02cf9a03697b026eb0187cd8 size: 1786

    ~/microservices-demo/src/paymentservice(main)$ docker push svs123/paymentservice:0.0.2
    The push refers to repository [docker.io/svs123/paymentservice]
    7fd8f7b945b3: Layer already exists
    af291a43d5a0: Layer already exists
    a04d719af9d7: Layer already exists
    5bbcb46a3da5: Layer already exists
    9a09a6062e1b: Layer already exists
    122fc55b6d78: Layer already exists
    4693057ce236: Layer already exists
    0.0.2: digest: sha256:a4d53ac559086d0f7cc8f510130cfd28a9310d0a02cf9a03697b026eb0187cd8 size: 1786
    ```
 - Создание replicaset для paymentservice

    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f paymentservice-replicaset.yaml
    replicaset.apps/paymentservice created

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get rs
    NAME             DESIRED   CURRENT   READY   AGE
    frontend         3         3         3       4h33m
    paymentservice   3         3         0       8s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -w
    NAME                   READY   STATUS              RESTARTS   AGE
    frontend-bsxbs         1/1     Running             0          22m
    frontend-llths         1/1     Running             0          22m
    frontend-xvcsx         1/1     Running             0          22m
    paymentservice-gjzcv   0/1     ContainerCreating   0          34s
    paymentservice-qfc8c   0/1     ContainerCreating   0          34s
    paymentservice-wwr9j   0/1     ContainerCreating   0          34s
    paymentservice-wwr9j   0/1     Running             0          46s
    paymentservice-qfc8c   0/1     Running             0          46s
    paymentservice-gjzcv   0/1     Running             0          46s
    paymentservice-wwr9j   1/1     Running             0          47s
    paymentservice-qfc8c   1/1     Running             0          47s
    paymentservice-gjzcv   1/1     Running             0          48s


    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po
    NAME                   READY   STATUS    RESTARTS   AGE
    frontend-bsxbs         1/1     Running   0          23m
    frontend-llths         1/1     Running   0          23m
    frontend-xvcsx         1/1     Running   0          23m
    paymentservice-gjzcv   1/1     Running   0          55s
    paymentservice-qfc8c   1/1     Running   0          55s
    paymentservice-wwr9j   1/1     Running   0          55s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get rs
    NAME             DESIRED   CURRENT   READY   AGE
    frontend         3         3         3       4h34m
    paymentservice   3         3         3       59s
    ```
 - Создание deployment для сервиса paymentservice
    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f paymentservice-deployment.yaml
    deployment.apps/paymentservice configured

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get deployments paymentservice
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    paymentservice   3/3     3            3           27s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-55bc57cbf7-d4d7z   1/1     Running   0          62s
    paymentservice-55bc57cbf7-vgq8k   1/1     Running   0          60s
    paymentservice-55bc57cbf7-zxgv5   1/1     Running   0          57s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"

    ```
 - Обновление сервиса paymentservice с версии 0.0.1 до 0.0.2 с помощью deployment
    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f paymentservice-deployment.yaml
    deployment.apps/paymentservice configured

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice
    NAME                              READY   STATUS              RESTARTS   AGE
    paymentservice-55bc57cbf7-d4d7z   1/1     Running             0          6m32s
    paymentservice-55bc57cbf7-vgq8k   1/1     Terminating         0          6m30s
    paymentservice-55bc57cbf7-zxgv5   1/1     Terminating         0          6m27s
    paymentservice-6bc8799b86-9b887   1/1     Running             0          5s
    paymentservice-6bc8799b86-fcfvc   1/1     Running             0          2s
    paymentservice-6bc8799b86-mws5n   0/1     ContainerCreating   0          0s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice
    NAME                              READY   STATUS        RESTARTS   AGE
    paymentservice-55bc57cbf7-d4d7z   1/1     Terminating   0          6m49s
    paymentservice-55bc57cbf7-vgq8k   1/1     Terminating   0          6m47s
    paymentservice-55bc57cbf7-zxgv5   1/1     Terminating   0          6m44s
    paymentservice-6bc8799b86-9b887   1/1     Running       0          22s
    paymentservice-6bc8799b86-fcfvc   1/1     Running       0          19s
    paymentservice-6bc8799b86-mws5n   1/1     Running       0          17s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-6bc8799b86-9b887   1/1     Running   0          43s
    paymentservice-6bc8799b86-fcfvc   1/1     Running   0          40s
    paymentservice-6bc8799b86-mws5n   1/1     Running   0          38s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.2"
    "svs123/paymentservice:0.0.2"
    "svs123/paymentservice:0.0.2"
    ```
 - Откат сервиса paymentservice

    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl rollout history deployment paymentservice
    deployment.apps/paymentservice
    REVISION  CHANGE-CAUSE
    2         <none>
    3         <none>
    
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl rollout undo deployment paymentservice --to-revision=2 | kubectl get rs -l app=paymentservice -w
    NAME                        DESIRED   CURRENT   READY   AGE
    paymentservice-55bc57cbf7   0         0         0       10m
    paymentservice-6bc8799b86   3         3         3       11m
    paymentservice-55bc57cbf7   0         0         0       10m
    paymentservice-55bc57cbf7   1         0         0       10m
    paymentservice-55bc57cbf7   1         0         0       10m
    paymentservice-55bc57cbf7   1         1         0       10m
    paymentservice-55bc57cbf7   1         1         1       10m
    paymentservice-6bc8799b86   2         3         3       11m
    paymentservice-55bc57cbf7   2         1         1       10m
    paymentservice-6bc8799b86   2         3         3       11m
    paymentservice-6bc8799b86   2         2         2       11m
    paymentservice-55bc57cbf7   2         1         1       10m
    paymentservice-55bc57cbf7   2         2         1       10m
    paymentservice-55bc57cbf7   2         2         2       10m
    paymentservice-6bc8799b86   1         2         2       11m
    paymentservice-55bc57cbf7   3         2         2       10m
    paymentservice-6bc8799b86   1         2         2       11m
    paymentservice-55bc57cbf7   3         2         2       10m
    paymentservice-6bc8799b86   1         1         1       11m
    paymentservice-55bc57cbf7   3         3         2       10m
    paymentservice-55bc57cbf7   3         3         3       10m
    paymentservice-6bc8799b86   0         1         1       11m
    paymentservice-6bc8799b86   0         1         1       11m
    paymentservice-6bc8799b86   0         0         0       11m

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice
    NAME                              READY   STATUS        RESTARTS   AGE
    paymentservice-55bc57cbf7-cjx5d   1/1     Running       0          28s
    paymentservice-55bc57cbf7-gn2bv   1/1     Running       0          25s
    paymentservice-55bc57cbf7-v4spc   1/1     Running       0          30s
    paymentservice-6bc8799b86-9b887   1/1     Terminating   0          4m55s
    paymentservice-6bc8799b86-fcfvc   1/1     Terminating   0          4m52s
    paymentservice-6bc8799b86-mws5n   1/1     Terminating   0          4m50s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ $ kubectget po -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-55bc57cbf7-cjx5d   1/1     Running   0          44s
    paymentservice-55bc57cbf7-gn2bv   1/1     Running   0          41s
    paymentservice-55bc57cbf7-v4spc   1/1     Running   0          46s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"
    ```
 - (*) Создание deployment (blue-green) paymentservice-deployment-bg.yaml

    В деплоймент добавлена стратегия обновления (blue-green),
    создание доп. 3 подов, отключение 3 подов при реплике 3.
    ```
    spec:
      strategy:
        type: RollingUpdate
        rollingUpdate:
            maxUnavailable: 3
            maxSurge: 6
    ```
    Поведение при обновлении версии сервиса

    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f paymentservice-deployment-bg.yaml
    deployment.apps/paymentservice created

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get deployments paymentservice
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    paymentservice   3/3     3            3           23s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-55bc57cbf7-kntlc   1/1     Running   0          11s
    paymentservice-55bc57cbf7-lff68   1/1     Running   0          11s
    paymentservice-55bc57cbf7-snbjd   1/1     Running   0          11s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f paymentservice-deployment-bg.yaml | kubectl get pods -l app=paymentservice -w
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-55bc57cbf7-kntlc   1/1     Running   0          94s
    paymentservice-55bc57cbf7-lff68   1/1     Running   0          94s
    paymentservice-55bc57cbf7-snbjd   1/1     Running   0          94s
    paymentservice-6bc8799b86-clqz4   0/1     Pending   0          0s
    paymentservice-6bc8799b86-clqz4   0/1     Pending   0          0s
    paymentservice-6bc8799b86-72zwk   0/1     Pending   0          0s
    paymentservice-6bc8799b86-c29wg   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-kntlc   1/1     Terminating   0          94s
    paymentservice-55bc57cbf7-snbjd   1/1     Terminating   0          94s
    paymentservice-55bc57cbf7-lff68   1/1     Terminating   0          94s
    paymentservice-6bc8799b86-c29wg   0/1     Pending       0          0s
    paymentservice-6bc8799b86-72zwk   0/1     Pending       0          0s
    paymentservice-6bc8799b86-clqz4   0/1     ContainerCreating   0          0s
    paymentservice-6bc8799b86-72zwk   0/1     ContainerCreating   0          1s
    paymentservice-6bc8799b86-c29wg   0/1     ContainerCreating   0          1s
    paymentservice-6bc8799b86-c29wg   0/1     Running             0          2s
    paymentservice-6bc8799b86-clqz4   0/1     Running             0          2s
    paymentservice-6bc8799b86-72zwk   0/1     Running             0          2s
    paymentservice-6bc8799b86-c29wg   1/1     Running             0          3s
    paymentservice-6bc8799b86-clqz4   1/1     Running             0          3s
    paymentservice-6bc8799b86-72zwk   1/1     Running             0          3s
    paymentservice-55bc57cbf7-lff68   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-snbjd   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-lff68   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-lff68   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-lff68   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-kntlc   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-snbjd   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-snbjd   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-snbjd   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-kntlc   0/1     Terminating         0          2m6s
    paymentservice-55bc57cbf7-kntlc   0/1     Terminating         0          2m6s
    paymentservice-55bc57cbf7-kntlc   0/1     Terminating         0          2m6s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ $ kubectget po -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-6bc8799b86-72zwk   1/1     Running   0          58s
    paymentservice-6bc8799b86-c29wg   1/1     Running   0          58s
    paymentservice-6bc8799b86-clqz4   1/1     Running   0          58s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.2"
    "svs123/paymentservice:0.0.2"
    "svs123/paymentservice:0.0.2"
    ```
 - (*) Создание deployment для сервиса paymentservice (Reverse Rolling Update)

    Добавлена стратегия обновления (Reverse Rolling Update)
    Удаление одного старого пода, добавление 1 нового пода
    ```
    spec:
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: 1
          maxSurge: 3
    ```
    Пример обновления версии приложения
    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f paymentservice-deployment-reverse.yaml | kubectl get pods -l app=paymentservice -w
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-55bc57cbf7-z5pp6   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-z5pp6   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-pz2rd   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-4p7bb   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-pz2rd   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-4p7bb   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-z5pp6   0/1     ContainerCreating   0          0s
    paymentservice-55bc57cbf7-pz2rd   0/1     ContainerCreating   0          0s
    paymentservice-55bc57cbf7-4p7bb   0/1     ContainerCreating   0          0s
    paymentservice-55bc57cbf7-pz2rd   0/1     Running             0          2s
    paymentservice-55bc57cbf7-4p7bb   0/1     Running             0          2s
    paymentservice-55bc57cbf7-z5pp6   0/1     Running             0          2s
    paymentservice-55bc57cbf7-pz2rd   1/1     Running             0          3s
    paymentservice-55bc57cbf7-4p7bb   1/1     Running             0          3s
    paymentservice-55bc57cbf7-z5pp6   1/1     Running             0          4s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get deployments paymentservice
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    paymentservice   3/3     3            3           15s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ $ kubect get po -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-55bc57cbf7-4p7bb   1/1     Running   0          21s
    paymentservice-55bc57cbf7-pz2rd   1/1     Running   0          21s
    paymentservice-55bc57cbf7-z5pp6   1/1     Running   0          21s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"
    "svs123/paymentservice:0.0.1"

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f paymentservice-deployment-reverse.yaml | kubectl get pods -l app=paymentservice -w
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-55bc57cbf7-4p7bb   1/1     Running   0          94s
    paymentservice-55bc57cbf7-pz2rd   1/1     Running   0          94s
    paymentservice-55bc57cbf7-z5pp6   1/1     Running   0          94s
    paymentservice-6bc8799b86-dqpcj   0/1     Pending   0          0s
    paymentservice-6bc8799b86-dqpcj   0/1     Pending   0          0s
    paymentservice-6bc8799b86-hfpx7   0/1     Pending   0          0s
    paymentservice-6bc8799b86-d9cgq   0/1     Pending   0          0s
    paymentservice-55bc57cbf7-pz2rd   1/1     Terminating   0          94s
    paymentservice-6bc8799b86-d9cgq   0/1     Pending       0          0s
    paymentservice-6bc8799b86-hfpx7   0/1     Pending       0          0s
    paymentservice-6bc8799b86-dqpcj   0/1     ContainerCreating   0          0s
    paymentservice-6bc8799b86-hfpx7   0/1     ContainerCreating   0          0s
    paymentservice-6bc8799b86-d9cgq   0/1     ContainerCreating   0          0s
    paymentservice-6bc8799b86-hfpx7   0/1     Running             0          2s
    paymentservice-6bc8799b86-dqpcj   0/1     Running             0          2s
    paymentservice-6bc8799b86-dqpcj   1/1     Running             0          2s
    paymentservice-6bc8799b86-d9cgq   0/1     Running             0          2s
    paymentservice-55bc57cbf7-z5pp6   1/1     Terminating         0          97s
    paymentservice-6bc8799b86-hfpx7   1/1     Running             0          3s
    paymentservice-55bc57cbf7-4p7bb   1/1     Terminating         0          97s
    paymentservice-6bc8799b86-d9cgq   1/1     Running             0          3s
    paymentservice-55bc57cbf7-pz2rd   0/1     Terminating         0          2m4s
    paymentservice-55bc57cbf7-pz2rd   0/1     Terminating         0          2m4s
    paymentservice-55bc57cbf7-pz2rd   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-pz2rd   0/1     Terminating         0          2m5s
    paymentservice-55bc57cbf7-z5pp6   0/1     Terminating         0          2m7s
    paymentservice-55bc57cbf7-z5pp6   0/1     Terminating         0          2m7s
    paymentservice-55bc57cbf7-4p7bb   0/1     Terminating         0          2m7s
    paymentservice-55bc57cbf7-z5pp6   0/1     Terminating         0          2m7s
    paymentservice-55bc57cbf7-z5pp6   0/1     Terminating         0          2m7s
    paymentservice-55bc57cbf7-4p7bb   0/1     Terminating         0          2m7s
    paymentservice-55bc57cbf7-4p7bb   0/1     Terminating         0          2m7s
    paymentservice-55bc57cbf7-4p7bb   0/1     Terminating         0          2m7s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ $ kubectget po -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-6bc8799b86-d9cgq   1/1     Running   0          77s
    paymentservice-6bc8799b86-dqpcj   1/1     Running   0          77s
    paymentservice-6bc8799b86-hfpx7   1/1     Running   0          77s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=paymentservice -o json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/paymentservice:0.0.2"
    "svs123/paymentservice:0.0.2"
    "svs123/paymentservice:0.0.2"
    ```
 - Создание deployment для сервиса frontend
    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f frontend-deployment.yaml | kubectl get pods -l app=frontend -w
    NAME                        READY   STATUS    RESTARTS   AGE
    frontend-786c467dc5-fg2fl   0/1     Pending   0          0s
    frontend-786c467dc5-fg2fl   0/1     Pending   0          0s
    frontend-786c467dc5-6s2gk   0/1     Pending   0          0s
    frontend-786c467dc5-5bgfc   0/1     Pending   0          0s
    frontend-786c467dc5-6s2gk   0/1     Pending   0          0s
    frontend-786c467dc5-5bgfc   0/1     Pending   0          0s
    frontend-786c467dc5-fg2fl   0/1     ContainerCreating   0          0s
    frontend-786c467dc5-6s2gk   0/1     ContainerCreating   0          0s
    frontend-786c467dc5-5bgfc   0/1     ContainerCreating   0          0s
    frontend-786c467dc5-fg2fl   0/1     Running             0          2s
    frontend-786c467dc5-6s2gk   0/1     Running             0          2s
    frontend-786c467dc5-5bgfc   0/1     Running             0          2s
    frontend-786c467dc5-fg2fl   1/1     Running             0          20s
    frontend-786c467dc5-6s2gk   1/1     Running             0          20s
    frontend-786c467dc5-5bgfc   1/1     Running             0          20s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get deployments frontend
    NAME       READY   UP-TO-DATE   AVAILABLE   AGE
    frontend   3/3     3            3           18s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ $ kubectget po -l app=frontend
    NAME                        READY   STATUS    RESTARTS   AGE
    frontend-786c467dc5-5bgfc   1/1     Running   0          32s
    frontend-786c467dc5-6s2gk   1/1     Running   0          32s
    frontend-786c467dc5-fg2fl   1/1     Running   0          32s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po -l app=frontend -o=json | jq '.items[0:3][].spec.containers[0].image'
    "svs123/frontend-boutique:0.0.1"
    "svs123/frontend-boutique:0.0.1"
    "svs123/frontend-boutique:0.0.1"

    # В отдельном окне терминала пробрасываем порт
    $ kubectl port-forward frontend-786c467dc5-5bgfc 8080:8080
    Forwarding from 127.0.0.1:8080 -> 8080
    Forwarding from [::1]:8080 -> 8080
    Handling connection for 8080
    Handling connection for 8080

    # В основном окне терминала
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ curl -o - -I --cookie "shop_session-id=x-readiness-probe"  http://localhost:8080/_healthz
    HTTP/1.1 200 OK
    Date: Wed, 27 Sep 2023 21:32:34 GMT
    Content-Length: 2
    Content-Type: text/plain; charset=utf-8

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ curl -o - -I --cookie "shop_session-id=x-liveness-probe"  http://localhost:8080/_healthz
    HTTP/1.1 200 OK
    Date: Wed, 27 Sep 2023 21:32:47 GMT
    Content-Length: 2
    Content-Type: text/plain; charset=utf-8
    ```
 - (**) Создание DaemonSet node-exporter
    ```
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl apply -f node-exporter-daemonset.yaml | kubectl get po -A -w
    NAMESPACE            NAME                                         READY   STATUS    RESTARTS      AGE
    default              frontend-786c467dc5-tts8h                    1/1     Running   0             8m27s
    default              frontend-786c467dc5-x47cm                    1/1     Running   0             8m27s
    default              frontend-7cf974ccd6-56j25                    0/1     Running   0             5m41s
    default              frontend-7cf974ccd6-n87l5                    0/1     Running   0             5m41s
    default              frontend-7cf974ccd6-vpmv4                    0/1     Running   0             5m41s
    default              paymentservice-6bc8799b86-d9cgq              1/1     Running   0             45m
    default              paymentservice-6bc8799b86-dqpcj              1/1     Running   0             45m
    default              paymentservice-6bc8799b86-hfpx7              1/1     Running   0             45m
    kube-system          coredns-5d78c9869d-ql9vd                     1/1     Running   4 (25h ago)   30h
    kube-system          coredns-5d78c9869d-qxwdw                     1/1     Running   4 (25h ago)   30h
    kube-system          etcd-kind-control-plane                      1/1     Running   1 (25h ago)   25h
    kube-system          kindnet-d65sz                                1/1     Running   4 (25h ago)   30h
    kube-system          kindnet-gt4tq                                1/1     Running   4 (25h ago)   30h
    kube-system          kindnet-jx6ln                                1/1     Running   4 (25h ago)   30h
    kube-system          kindnet-nkx8f                                1/1     Running   4 (25h ago)   30h
    kube-system          kube-apiserver-kind-control-plane            1/1     Running   1 (25h ago)   25h
    kube-system          kube-controller-manager-kind-control-plane   1/1     Running   6 (25h ago)   30h
    kube-system          kube-proxy-9kj4v                             1/1     Running   4 (25h ago)   30h
    kube-system          kube-proxy-czqm7                             1/1     Running   4 (25h ago)   30h
    kube-system          kube-proxy-mwxg4                             1/1     Running   4 (25h ago)   30h
    kube-system          kube-proxy-zs6bd                             1/1     Running   4 (25h ago)   30h
    kube-system          kube-scheduler-kind-control-plane            1/1     Running   5 (25h ago)   30h
    local-path-storage   local-path-provisioner-6bc4bddd6b-mdkgw      1/1     Running   6 (25h ago)   30h
    default              node-exporter-xf29n                          0/1     Pending   0             0s
    default              node-exporter-xf29n                          0/1     Pending   0             0s
    default              node-exporter-bkrdc                          0/1     Pending   0             0s
    default              node-exporter-csv7f                          0/1     Pending   0             0s
    default              node-exporter-bkrdc                          0/1     Pending   0             1s
    default              node-exporter-xf29n                          0/1     ContainerCreating   0             1s
    default              node-exporter-9nrfv                          0/1     Pending             0             0s
    default              node-exporter-csv7f                          0/1     Pending             0             1s
    default              node-exporter-9nrfv                          0/1     Pending             0             0s
    default              node-exporter-csv7f                          0/1     ContainerCreating   0             1s
    default              node-exporter-bkrdc                          0/1     ContainerCreating   0             1s
    default              node-exporter-9nrfv                          0/1     ContainerCreating   0             0s
    default              node-exporter-xf29n                          1/1     Running             0             2s
    default              node-exporter-bkrdc                          1/1     Running             0             2s
    default              node-exporter-csv7f                          1/1     Running             0             3s
    default              node-exporter-9nrfv                          1/1     Running             0             2s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get daemonsets
    NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    node-exporter   4         4         4       4            4           <none>          61s

    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ kubectl get po
    NAME                              READY   STATUS    RESTARTS   AGE
    frontend-786c467dc5-tts8h         1/1     Running   0          9m45s
    frontend-786c467dc5-x47cm         1/1     Running   0          9m45s
    frontend-7cf974ccd6-56j25         0/1     Running   0          6m59s
    frontend-7cf974ccd6-n87l5         0/1     Running   0          6m59s
    frontend-7cf974ccd6-vpmv4         0/1     Running   0          6m59s
    node-exporter-9nrfv               1/1     Running   0          77s
    node-exporter-bkrdc               1/1     Running   0          78s
    node-exporter-csv7f               1/1     Running   0          78s
    node-exporter-xf29n               1/1     Running   0          78s
    paymentservice-6bc8799b86-d9cgq   1/1     Running   0          46m
    paymentservice-6bc8799b86-dqpcj   1/1     Running   0          46m
    paymentservice-6bc8799b86-hfpx7   1/1     Running   0          46m

    # Прокидываем порт 9100 на один из подов в отдельном окне терминала
    $ kubectl port-forward node-exporter-9nrfv 9100:9100
    Forwarding from 127.0.0.1:9100 -> 9100
    Forwarding from [::1]:9100 -> 9100
    Handling connection for 9100
    Handling connection for 9100
    Handling connection for 9100
    Handling connection for 9100
    Handling connection for 9100

    # Тест получения метрик
    ~/VladimirSVS_platform/kubernetes-controllers(kubernetes-controllers)$ curl -s localhost:9100/metrics | egrep '^node_' | head -n 10
    node_arp_entries{device="eth0"} 4
    node_arp_entries{device="veth2b28e620"} 1
    node_arp_entries{device="veth5805bc22"} 1
    node_arp_entries{device="veth92ca7df7"} 1
    node_boot_time_seconds 1.695759541e+09
    node_context_switches_total 2.18242649e+08
    node_cooling_device_cur_state{name="0",type="Processor"} 0
    node_cooling_device_max_state{name="0",type="Processor"} 0
    node_cpu_guest_seconds_total{cpu="0",mode="nice"} 0
    node_cpu_guest_seconds_total{cpu="0",mode="user"} 0
    ```

## Как проверить работоспособность:

    Описанно в разделе Как запустить проект

## PR checklist:
 - [x] Выставлен label с темой домашнего задания
