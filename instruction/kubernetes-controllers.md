ДЗ ОТУС
Устанваливаем kind:
```
[root@kuber 2kw92_platform]# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    98  100    98    0     0    298      0 --:--:-- --:--:-- --:--:--   300
100   655  100   655    0     0    925      0 --:--:-- --:--:-- --:--:--   925
100 6660k  100 6660k    0     0  3257k      0  0:00:02  0:00:02 --:--:-- 5504k

chmod +x ./kind
mv ./kind /usr/local/bin/kind

[root@kuber 2kw92_platform]# kind --version
kind version 0.11.1
```

Чтобы kind взлетел в centos необходимо поправить сервис kubelet:
```
sed -i 's#Environment="KUBELET_KUBECONFIG_ARGS=-.*#Environment="KUBELET_KUBECONFIG_ARGS=--kubeconfig=/etc/kubernetes/kubelet.conf --require-kubeconfig=true --cgroup-driver=systemd"#g' /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
```

Теперь создадим кластер, так как машина слабая архитектура будет из 1 мастера и 3 воркера:
```
[docker@kuber ~]$ kind create cluster --config kind-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
[docker@kuber ~]$ kubectl get nodes
NAME                 STATUS     ROLES                  AGE    VERSION
kind-control-plane   Ready      control-plane,master   107s   v1.21.1
kind-worker          NotReady   <none>                 72s    v1.21.1
kind-worker2         Ready      <none>                 72s    v1.21.1
kind-worker3         Ready      <none>                 72s    v1.21.1
[docker@kuber ~]$ kubectl get nodes
NAME                 STATUS   ROLES                  AGE    VERSION
kind-control-plane   Ready    control-plane,master   115s   v1.21.1
kind-worker          Ready    <none>                 80s    v1.21.1
kind-worker2         Ready    <none>                 80s    v1.21.1
kind-worker3         Ready    <none>                 80s    v1.21.1
```

Пробуем применить frontend-replicaset.yaml:
```
[docker@kuber ~]$ kubectl apply -f frontend-replicaset.yaml
error: error validating "frontend-replicaset.yaml": error validating data: ValidationError(ReplicaSet.spec): missing required field "selector" in io.k8s.api.apps.v1.ReplicaSetSpec; if you choose to ignore these errors, turn validation off with --validate=false
```
Сделав необходимые правки в frontend-replicaset.yaml запустим его снова и провеим:
```
[docker@kuber ~]$ kubectl apply -f frontend-replicaset.yaml
replicaset.apps/frontend created
[docker@kuber ~]$ kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-kvg78   1/1     Running   0          33s
```

Увеличим количество реплик:
```
[docker@kuber ~]$ kubectl scale replicaset frontend --replicas=3
replicaset.apps/frontend scaled
[docker@kuber ~]$ kubectl get pods -l app=frontend
NAME             READY   STATUS              RESTARTS   AGE
frontend-2r88b   0/1     ContainerCreating   0          93s
frontend-kvg78   1/1     Running             0          3m47s
frontend-x2g8c   0/1     ContainerCreating   0          93s

[docker@kuber ~]$ kubectl get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       20m
```

Проверим что поды восстанавливаются после ручного удаления:
```
[docker@kuber ~]$ kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
NAME             READY   STATUS    RESTARTS   AGE
frontend-2r88b   1/1     Running   0          18m
frontend-kvg78   1/1     Running   0          20m
frontend-x2g8c   1/1     Running   0          18m
frontend-2r88b   1/1     Terminating   0          18m
frontend-kvg78   1/1     Terminating   0          20m
frontend-x2g8c   1/1     Terminating   0          18m
frontend-7bkjj   0/1     Pending       0          0s
frontend-pw6hn   0/1     Pending       0          0s
frontend-kvcm6   0/1     Pending       0          0s
frontend-kvcm6   0/1     Pending       0          1s
frontend-7bkjj   0/1     Pending       0          1s
frontend-pw6hn   0/1     Pending       0          1s
frontend-kvcm6   0/1     ContainerCreating   0          1s
frontend-pw6hn   0/1     ContainerCreating   0          1s
frontend-7bkjj   0/1     ContainerCreating   0          1s
frontend-x2g8c   0/1     Terminating         0          18m
frontend-2r88b   0/1     Terminating         0          18m
frontend-kvg78   0/1     Terminating         0          20m
frontend-kvcm6   1/1     Running             0          3s
frontend-pw6hn   1/1     Running             0          3s
frontend-7bkjj   1/1     Running             0          3s
frontend-2r88b   0/1     Terminating         0          18m
frontend-2r88b   0/1     Terminating         0          18m
frontend-x2g8c   0/1     Terminating         0          18m
frontend-kvg78   0/1     Terminating         0          21m
frontend-x2g8c   0/1     Terminating         0          18m
frontend-kvg78   0/1     Terminating         0          21m

[docker@kuber ~]$ kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-7bkjj   1/1     Running   0          3m5s
frontend-kvcm6   1/1     Running   0          3m5s
frontend-pw6hn   1/1     Running   0          3m5s
```

Повторно применим манифест и убедимся что останетяс одна нода:
```
[docker@kuber ~]$ kubectl apply -f frontend-replicaset.yaml
replicaset.apps/frontend configured
[docker@kuber ~]$ kubectl get pods -l app=frontend
NAME             READY   STATUS        RESTARTS   AGE
frontend-7bkjj   0/1     Terminating   0          3m36s
frontend-kvcm6   1/1     Running       0          3m36s
frontend-pw6hn   0/1     Terminating   0          3m36s
[docker@kuber ~]$ kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-kvcm6   1/1     Running   0          4m23s
```
После правки манифеста реплики снова 3:
```
[docker@kuber ~]$ kubectl apply -f frontend-replicaset.yaml
replicaset.apps/frontend configured
[docker@kuber ~]$ kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-kvcm6   1/1     Running   0          10m
frontend-kwnt2   1/1     Running   0          5s
frontend-xglgz   1/1     Running   0          5s
```

Запушив в докерхаб новую версию образа с тэгом 0.2 и пропишем ее в нашем манифесте и сделаем apply:
```
[docker@kuber ~]$ kubectl apply -f frontend-replicaset.yaml | kubectl get pods -l app=frontend -w
NAME             READY   STATUS    RESTARTS   AGE
frontend-kvcm6   1/1     Running   0          33m
frontend-kwnt2   1/1     Running   0          23m
frontend-xglgz   1/1     Running   0          23m
```
ВРоде ничего не пменялось, но образ указанный в репликасете отличается отличается от образа от которого запущены поды
```
[docker@kuber ~]$ kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
2kw92/frontend:0.2

kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
2kw92/frontend:0.1 2kw92/frontend:0.1 2kw92/frontend:0.1
```

Обновление манифеста ReplicaSet не повлекло обновление запущенных pod т.к. replicaset controller не умеет рестартовать запущенные поды    
при обновлении шаблона, поэтому версия контейнеров оказалась старой

Далее запушим образы на докерхаб для микросервиса paymentservice, доступны по адресу:
```
https://hub.docker.com/repository/docker/2kw92/paymentservice
```

Напишем deplyment и применим его, после этого проверим что все поднялось:
```
[docker@kuber ~]$ kubectl apply -f paymentservice-deployment.yaml
deployment.apps/paymentservice created
[docker@kuber ~]$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
paymentservice-6ssph   1/1     Running   0          6m9s
paymentservice-ggf8x   1/1     Running   0          6m30s
paymentservice-jrklz   1/1     Running   0          7m
[docker@kuber ~]$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
paymentservice   3/3     3            3           24s
[docker@kuber ~]$ kubectl get rs
NAME             DESIRED   CURRENT   READY   AGE
paymentservice   3         3         3       14m
```

После этого изменим наш манифест и проставим там образ 2kw92/paymentservice:0.0.2 и применим наш манифест:
```
[docker@kuber ~]$ kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
NAME                   READY   STATUS    RESTARTS   AGE
paymentservice-6ssph   1/1     Running   0          10m
paymentservice-ggf8x   1/1     Running   0          10m
paymentservice-jrklz   1/1     Running   0          11m
paymentservice-d848f99f7-wnn5m   0/1     Pending   0          0s
paymentservice-d848f99f7-wnn5m   0/1     Pending   0          0s
paymentservice-d848f99f7-wnn5m   0/1     ContainerCreating   0          0s
paymentservice-d848f99f7-wnn5m   1/1     Running             0          3s
paymentservice-jrklz             1/1     Terminating         0          11m
paymentservice-d848f99f7-xn9rs   0/1     Pending             0          0s
paymentservice-d848f99f7-xn9rs   0/1     Pending             0          0s
paymentservice-d848f99f7-xn9rs   0/1     ContainerCreating   0          0s
paymentservice-jrklz             0/1     Terminating         0          11m
paymentservice-jrklz             0/1     Terminating         0          11m
paymentservice-jrklz             0/1     Terminating         0          11m
paymentservice-d848f99f7-xn9rs   1/1     Running             0          2s
paymentservice-6ssph             1/1     Terminating         0          10m
paymentservice-d848f99f7-2nhnz   0/1     Pending             0          0s
paymentservice-d848f99f7-2nhnz   0/1     Pending             0          0s
paymentservice-d848f99f7-2nhnz   0/1     ContainerCreating   0          0s
paymentservice-6ssph             0/1     Terminating         0          10m
paymentservice-6ssph             0/1     Terminating         0          10m
paymentservice-6ssph             0/1     Terminating         0          10m
paymentservice-d848f99f7-2nhnz   1/1     Running             0          2s
paymentservice-ggf8x             1/1     Terminating         0          10m
paymentservice-ggf8x             0/1     Terminating         0          10m
paymentservice-ggf8x             0/1     Terminating         0          10m
paymentservice-ggf8x             0/1     Terminating         0          10m
```
Убедимся что новые поды развернуты с новой версией образа:
```
[docker@kuber ~]$ kubectl get deployments paymentservice -o=jsonpath='{.spec.template.spec.containers[0].image}'
2kw92/paymentservice:0.0.2
```

Убеждаемя что страый репликасет управляет нулевым количеством под:
```
[docker@kuber ~]$ kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
paymentservice             0         0         0       24m
paymentservice-d848f99f7   3         3         3       6m34s
```

Посмотрим историю версий нашего deployment:
```
[docker@kuber ~]$ kubectl rollout history deployment paymentservice
deployment.apps/paymentservice
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

Теперь сделаем откат до предидущей версии deployment:
```
[docker@kuber ~]$ kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
paymentservice             3         3         3       28m
paymentservice-d848f99f7   0         0         0       10m
[docker@kuber ~]$ kubectl get deployments paymentservice -o=jsonpath='{.spec.template.spec.containers[0].image}'
2kw92/paymentservice:0.0.1
```

Видим что откат прошел успешно.

Для репализации  blue-green необходимо вставить следующий блок в манифест
```
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 100%
      maxUnavailable: 0
```

И применим манифест:
```
[docker@kuber ~]$ kubectl apply -f paymentservice-deployment-bg.yaml
deployment.apps/paymentservice configured
[docker@kuber ~]$ kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
paymentservice             0         0         0       41m
paymentservice-d848f99f7   3         3         3       23m
[docker@kuber ~]$ kubectl get deployments paymentservice -o=jsonpath='{.spec.template.spec.containers[0].image}'
2kw92/paymentservice:0.0.2
```
Видим что деплой прошел успешно.

Для репализации  Reverse Rolling Update: необходимо вставить следующий блок в манифест
```
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
```
И применить его:
```
[docker@kuber ~]$ kubectl apply -f paymentservice-deployment-reverse.yaml
deployment.apps/paymentservice configured
```

Далее будем работать с Probes. Создадим новый манифест frontend-deployment.yaml и применем его:
```
[docker@kuber ~]$ kubectl apply -f frontend-deployment.yaml
deployment.apps/frontend created
[docker@kuber ~]$ kubectl get pods -l app=frontend
NAME                        READY   STATUS    RESTARTS   AGE
frontend-7df74dc858-dxwsw   0/1     Running   0          26s
frontend-7df74dc858-tg9bh   0/1     Running   0          26s
frontend-7df74dc858-ztm9n   0/1     Running   0          26s
```
Видим что он running, но не ready, посмотрим почему:
```
kubectl describe pod frontend-7df74dc858-dxwsw
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  4m27s                default-scheduler  Successfully assigned default/frontend-7df74dc858-dxwsw to kind-worker2
  Normal   Pulling    4m27s                kubelet            Pulling image "2kw92/frontend:0.1"
  Normal   Pulled     4m25s                kubelet            Successfully pulled image "2kw92/frontend:0.1" in 1.839041826s
  Normal   Created    4m25s                kubelet            Created container server
  Normal   Started    4m25s                kubelet            Started container server
  Warning  Unhealthy  47s (x21 over 4m7s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
```
И если нам нужно откатиться то выполняем:
```
[docker@kuber ~]$ kubectl rollout status deployment/frontend
Waiting for deployment "frontend" rollout to finish: 0 of 3 updated replicas are available...
``` 
