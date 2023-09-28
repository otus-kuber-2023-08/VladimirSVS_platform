# Выполнено ДЗ №1

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:
 - Добавлены тесты для Github actions. с правкой на использование модуля kentaro-m/auto-assign-action@v1.2.5  в workflows/auto_assign.yml (для использования в тестах node16 вместо node12, который deprecated)
 - Для локальной работы с kubernetes, развернут кластер на основе kind
 - Создан докер образ svs123/web-server:0.0.1 c помощью kubernetes-intro/web/Dockerfile и вспомогательных файлов homework.html, nginx.conf. Образ размещен на [hub.docker.com](https://hub.docker.com/r/svs123/web-server)

   Приложение в контейнере работает от имени пользователя webuser с UID=1001, GID=1001

 - Создан под с именем web с помощью манифеста kubernetes-intro/web-pod.yml на основе образа svs123/web-server:0.0.1
 - Добавлен init контейнер в манифест kubernetes-intro/web-pod.yml и под пересоздан.
 - В манифест kubernetes-intro/web-pod.yml добавлен volume c именем app и типом emptyDir: {}. В контейнере приложения и init контейнере добавлено монтирование volume с именем app.
 - Осуществлена проверка работы приложения web (см. ниже)
 - В задании со * создан docker образ svs123/frontend-boutique:0.0.1 и размещен на [hub.docker.com](https://hub.docker.com/r/svs123/frontend-boutique)
 - Создан манифест на основе ad-hoc режима kubernetes-intro/frontend-pod-healthy.yml c правками при отладке запуска пода.

    Добавлены переменные окружения от которых зависит запуск приложения в поде. А также readinessProbe и livenessProbe для последующей проверки доступности работы сервиса.
 Данные по переменным окружения и настройкам readinessProbe и livenessProbe повзаимствованы из манифеста деплоймента frontend, проекта [microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/kubernetes-manifests/frontend.yaml)

## Как запустить проект:
 - Запуск web-server контейнера в docker

    ```docker
    ~/VladimirSVS_platform(kubernetes-intro)$ docker pull svs123/web-server:0.0.1
    0.0.1: Pulling from svs123/web-server
    a803e7c4b030: Pull complete
    8b625c47d697: Pull complete
    4d3239651a63: Pull complete
    0f816efa513d: Pull complete
    01d159b8db2f: Pull complete
    5fb9a81470f3: Pull complete
    9b1e1e7164db: Pull complete
    e660196235e7: Pull complete
    066ee0b20db5: Pull complete
    a3b297fe8ed4: Pull complete
    8c335e71d378: Pull complete
    Digest: sha256:29a493b37dd8f9f9a8ac08f68dfa146f845d31a1bf291446431013ea970af6e6
    Status: Downloaded newer image for svs123/web-server:0.0.1
    docker.io/svs123/web-server:0.0.1

    ~/VladimirSVS_platform(kubernetes-intro)$ docker image ls
    REPOSITORY                 TAG       IMAGE ID       CREATED             SIZE
    frontend-boutique          0.0.1     1ea520e0393a   About an hour ago   42.4MB
    svs123/frontend-boutique   0.0.1     1ea520e0393a   About an hour ago   42.4MB
    svs123/web-server          0.0.1     c79ee4764bc6   23 hours ago        187MB
    kindest/node               <none>    89e7dc9f9131   3 months ago        932MB

    ~/VladimirSVS_platform(kubernetes-intro)$ docker run --rm -d --name=web-server -p 8000:8000 svs123/web-server:0.0.1
    4591da291ce1aba5bd4177efce1b76f27a63427f879dbbdb17278f9879c9aa5e

    ~/VladimirSVS_platform(kubernetes-intro)$ docker ps
    CONTAINER ID   IMAGE                     COMMAND                  CREATED         STATUS         PORTS                            NAMES
    4591da291ce1   svs123/web-server:0.0.1   "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp, 0.0.0.0:8000->8000/tcp   web-server

    ~/VladimirSVS_platform(kubernetes-intro)$ curl -v http://localhost:8000/homework.html
    *   Trying 127.0.0.1:8000...
    * TCP_NODELAY set
    * Connected to localhost (127.0.0.1) port 8000 (#0)
    > GET /homework.html HTTP/1.1
    > Host: localhost:8000
    > User-Agent: curl/7.68.0
    > Accept: */*
    >
    * Mark bundle as not supporting multiuse
    < HTTP/1.1 200 OK
    < Server: nginx/1.25.2
    < Date: Mon, 25 Sep 2023 21:06:26 GMT
    < Content-Type: text/html
    < Content-Length: 67
    < Last-Modified: Sun, 24 Sep 2023 21:32:12 GMT
    < Connection: keep-alive
    < ETag: "6510aadc-43"
    < Accept-Ranges: bytes
    <
    <html>
    <head>homework</head>
    <body><p>homework</p>
    </body>
    </html>
    * Connection #0 to host localhost left intact

    ~/VladimirSVS_platform(kubernetes-intro)$ docker exec -it web-server bash
    webuser@4591da291ce1:/$ id
    uid=1001(webuser) gid=1001(webuser) groups=1001(webuser)
    webuser@4591da291ce1:/$ exit
    exit
    ```

 - Запуск web-server в под с именем web в kubernetes

    ```docker
    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl apply -f kubernetes-intro/web-pod.yaml
    pod/web created

    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl get po
    NAME   READY   STATUS    RESTARTS   AGE
    web    1/1     Running   0          10s

    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl get events
    LAST SEEN   TYPE     REASON           OBJECT                    MESSAGE
    30m         Normal   RegisteredNode   node/kind-control-plane   Node kind-control-plane event: Registered Node kind-control-plane in Controller
    59m         Normal   Killing          pod/web                   Stopping container web-server
    59m         Normal   Scheduled        pod/web                   Successfully assigned default/web to kind-control-plane
    59m         Normal   Pulled           pod/web                   Container image "busybox:1.36" already present on machine
    59m         Normal   Created          pod/web                   Created container init-index
    59m         Normal   Started          pod/web                   Started container init-index
    58m         Normal   Pulled           pod/web                   Container image "svs123/web-server:0.0.1" already present on machine
    58m         Normal   Created          pod/web                   Created container web-server
    58m         Normal   Started          pod/web                   Started container web-server
    49s         Normal   Killing          pod/web                   Stopping container web-server
    21s         Normal   Scheduled        pod/web                   Successfully assigned default/web to kind-control-plane
    22s         Normal   Pulled           pod/web                   Container image "busybox:1.36" already present on machine
    22s         Normal   Created          pod/web                   Created container init-index
    22s         Normal   Started          pod/web                   Started container init-index
    19s         Normal   Pulled           pod/web                   Container image "svs123/web-server:0.0.1" already present on machine
    19s         Normal   Created          pod/web                   Created container web-server
    19s         Normal   Started          pod/web                   Started container web-server
    ```

 - Запуск сервиса frontend-boutique в поде c именем frontend кластера kubernetes

    ```docker
    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl apply -f kubernetes-intro/frontend-pod-healthy.yml
    pod/frontend created

    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl get events -w
    LAST SEEN   TYPE     REASON           OBJECT                    MESSAGE
    22s         Normal   Scheduled        pod/frontend              Successfully assigned default/frontend to kind-control-plane
    22s         Normal   Pulled           pod/frontend              Container image "svs123/frontend-boutique:0.0.1" already present on machine
    22s         Normal   Created          pod/frontend              Created container frontend
    22s         Normal   Started          pod/frontend              Started container frontend
    48m         Normal   RegisteredNode   node/kind-control-plane   Node kind-control-plane event: Registered Node kind-control-plane in Controller
    18m         Normal   Killing          pod/web                   Stopping container web-server
    17m         Normal   Scheduled        pod/web                   Successfully assigned default/web to kind-control-plane
    17m         Normal   Pulled           pod/web                   Container image "busybox:1.36" already present on machine
    17m         Normal   Created          pod/web                   Created container init-index
    17m         Normal   Started          pod/web                   Started container init-index
    17m         Normal   Pulled           pod/web                   Container image "svs123/web-server:0.0.1" already present on machine
    17m         Normal   Created          pod/web                   Created container web-server
    17m         Normal   Started          pod/web                   Started container web-server

    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl get po
    NAME       READY   STATUS    RESTARTS   AGE
    frontend   1/1     Running   0          29s
    web        1/1     Running   0          18m
    ```

## Как проверить работоспособность:

 - Проверка работы web-server в под web kubernetes
    Открыть 2 окна терминала подключения к кластеру, в первом окне терминала запустить проброс порта 8000.

    Во втором окне терминала выполнить проверку с помощью curl запроса.

    Примеры запуска команд с выводом результата при проверке.

    ```
    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl get po
    NAME   READY   STATUS    RESTARTS   AGE
    web    1/1     Running   0          10s

    ~/VladimirSVS_platform/kubernetes-intro(kubernetes-intro)$ kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
    Forwarding from 0.0.0.0:8000 -> 8000
    Handling connection for 8000
    Handling connection for 8000

    ~/VladimirSVS_platform(kubernetes-intro)$ curl -o - -I  http://localhost:8000/index.html
    HTTP/1.1 200 OK
    Server: nginx/1.25.2
    Date: Mon, 25 Sep 2023 21:45:51 GMT
    Content-Type: text/html
    Content-Length: 84549
    Last-Modified: Mon, 25 Sep 2023 21:22:06 GMT
    Connection: keep-alive
    ETag: "6511f9fe-14a45"
    Accept-Ranges: bytes
    ```

 - Проверка работы сервиса frontend-boutiqueпода в поде frontend

    ```docker
    ~/VladimirSVS_platform(kubernetes-intro)$ kubectl logs frontend
    {"message":"Tracing disabled.","severity":"info","timestamp":"2023-09-25T22:38:20.662038107Z"}
    {"message":"Profiling disabled.","severity":"info","timestamp":"2023-09-25T22:38:20.662298734Z"}
    {"message":"starting server on :8080","severity":"info","timestamp":"2023-09-25T22:38:20.663585605Z"}
    {"http.req.id":"4196b2f9-c86d-4eee-9b4f-8a4a338c6b73","http.req.method":"GET","http.req.path":"/_healthz","message":"request started","session":"x-liveness-probe","severity":"debug","timestamp":"2023-09-25T22:38:30.290530954Z"}
    {"http.req.id":"4196b2f9-c86d-4eee-9b4f-8a4a338c6b73","http.req.method":"GET","http.req.path":"/_healthz","http.resp.bytes":2,"http.resp.status":200,"http.resp.took_ms":0,"message":"request complete","session":"x-liveness-probe","severity":"debug","timestamp":"2023-09-25T22:38:30.290631571Z"}
    {"http.req.id":"9a095cd3-f292-485a-be17-df576f708e1c","http.req.method":"GET","http.req.path":"/_healthz","message":"request started","session":"x-readiness-probe","severity":"debug","timestamp":"2023-09-25T22:38:30.291001784Z"}
    ...

    ~/VladimirSVS_platform(kubernetes-intro)$ curl -o - -I --cookie "shop_session-id=x-readiness-probe"  http://localhost:8080/_healthz
    HTTP/1.1 200 OK
    Date: Mon, 25 Sep 2023 22:49:30 GMT
    Content-Length: 2
    Content-Type: text/plain; charset=utf-8

    ~/VladimirSVS_platform(kubernetes-intro)$ curl -o - -I --cookie "shop_session-id=x-liveness-probe"  http://localhost:8080/_healthz
    HTTP/1.1 200 OK
    Date: Mon, 25 Sep 2023 22:50:11 GMT
    Content-Length: 2
    Content-Type: text/plain; charset=utf-8
    ```

## PR checklist:
 - [x] Выставлен label с темой домашнего задания