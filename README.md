# 2kw92_platform
2kw92 Platform repository

Ставим kubectl: 
```
[root@kuber ~]# curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 44.4M  100 44.4M    0     0  9917k      0  0:00:04  0:00:04 --:--:-- 9920k
[root@kuber ~]# chmod +x ./kubectl
[root@kuber ~]# sudo mv ./kubectl /usr/local/bin/kubectl
[root@kuber ~]# kubectl version --client
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:25:17Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/amd64"}
```


Далее Ставим minicube, предварительно установив докер:
```
[root@kuber ~]# systemctl start docker
[root@kuber ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2022-02-03 07:32:40 UTC; 5s ago
     Docs: https://docs.docker.com
 Main PID: 1323 (dockerd)
    Tasks: 8
   Memory: 31.0M
   CGroup: /system.slice/docker.service
           └─1323 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Feb 03 07:32:39 kuber dockerd[1323]: time="2022-02-03T07:32:39.496672934Z" level=info msg="scheme \"unix\" not registered, fallback to defa...dule=grpc
Feb 03 07:32:39 kuber dockerd[1323]: time="2022-02-03T07:32:39.499021607Z" level=info msg="ccResolverWrapper: sending update to cc: {[{unix...dule=grpc
Feb 03 07:32:39 kuber dockerd[1323]: time="2022-02-03T07:32:39.499084228Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
Feb 03 07:32:39 kuber dockerd[1323]: time="2022-02-03T07:32:39.692217137Z" level=info msg="Loading containers: start."
Feb 03 07:32:40 kuber dockerd[1323]: time="2022-02-03T07:32:40.067069184Z" level=info msg="Default bridge (docker0) is assigned with an IP ... address"
Feb 03 07:32:40 kuber dockerd[1323]: time="2022-02-03T07:32:40.211586731Z" level=info msg="Loading containers: done."
Feb 03 07:32:40 kuber dockerd[1323]: time="2022-02-03T07:32:40.259549999Z" level=info msg="Docker daemon" commit=459d0df graphdriver(s)=ove...=20.10.12
Feb 03 07:32:40 kuber dockerd[1323]: time="2022-02-03T07:32:40.259889146Z" level=info msg="Daemon has completed initialization"
Feb 03 07:32:40 kuber systemd[1]: Started Docker Application Container Engine.
Feb 03 07:32:40 kuber dockerd[1323]: time="2022-02-03T07:32:40.326425127Z" level=info msg="API listen on /var/run/docker.sock"
Hint: Some lines were ellipsized, use -l to show in full.
```
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Создаем юзера docker, который имеет право на запуск контейнеров, и запускаем minicube:
```
[docker@kuber ~]$ minikube start
* minikube v1.25.1 on Centos 7.9.2009 (vbox/amd64)
* Automatically selected the docker driver

X Requested memory allocation (1837MB) is less than the recommended minimum 1900MB. Deployments may fail.


X The requested memory allocation of 1837MiB does not leave room for system overhead (total system memory: 1837MiB). You may face stability issues.
* Suggestion: Start minikube with less memory allocated: 'minikube start --memory=1837mb'

* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Downloading Kubernetes v1.23.1 preload ...
    > preloaded-images-k8s-v16-v1...: 504.42 MiB / 504.42 MiB  100.00% 6.47 MiB
    > gcr.io/k8s-minikube/kicbase: 378.98 MiB / 378.98 MiB  100.00% 4.06 MiB p/
* Creating docker container (CPUs=2, Memory=1837MB) ...
* Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
  - kubelet.housekeeping-interval=5m
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Смотрим текущую конфигурацию:
```
[docker@kuber ~]$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Зайдем на ВМ по ssh и глянем запущенные контейнеры:
```
[docker@kuber ~]$ minikube ssh
docker@minikube:~$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED              STATUS              PORTS     NAMES
3c1e82fd994e   6e38f40d628d           "/storage-provisioner"   About a minute ago   Up About a minute             k8s_storage-provisioner_storage-provisioner_kube-system_426d833c-02d3-4baf-bf25-dd5046741543_1
7e5668c6e8d4   a4ca41631cc7           "/coredns -conf /etc…"   2 minutes ago        Up 2 minutes                  k8s_coredns_coredns-64897985d-p544v_kube-system_c2851219-2a35-4280-af11-1dc2494cb9ae_0
0063ef3522f9   b46c42588d51           "/usr/local/bin/kube…"   2 minutes ago        Up 2 minutes                  k8s_kube-proxy_kube-proxy-p8llp_kube-system_34dd7be8-6352-4f48-b44a-0dfe5eb02dcc_0
9173d441900a   k8s.gcr.io/pause:3.6   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_coredns-64897985d-p544v_kube-system_c2851219-2a35-4280-af11-1dc2494cb9ae_0
b04553354921   k8s.gcr.io/pause:3.6   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_kube-proxy-p8llp_kube-system_34dd7be8-6352-4f48-b44a-0dfe5eb02dcc_0
28f89e56d121   k8s.gcr.io/pause:3.6   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_storage-provisioner_kube-system_426d833c-02d3-4baf-bf25-dd5046741543_0
aae3efac0d47   71d575efe628           "kube-scheduler --au…"   2 minutes ago        Up 2 minutes                  k8s_kube-scheduler_kube-scheduler-minikube_kube-system_b8bdc344ff0000e961009344b94de59c_0
2a02c4749f90   25f8c7f3da61           "etcd --advertise-cl…"   2 minutes ago        Up 2 minutes                  k8s_etcd_etcd-minikube_kube-system_9d3d310935e5fabe942511eec3e2cd0c_0
6feceb9ac81e   b6d7abedde39           "kube-apiserver --ad…"   2 minutes ago        Up 2 minutes                  k8s_kube-apiserver_kube-apiserver-minikube_kube-system_96be69ce9ff7dc0acff6fda2873a009a_0
00c36bc3d78f   f51846a4fd28           "kube-controller-man…"   2 minutes ago        Up 2 minutes                  k8s_kube-controller-manager_kube-controller-manager-minikube_kube-system_3db91997554714e5ece3296773cf98a5_0
48d573e904db   k8s.gcr.io/pause:3.6   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_kube-scheduler-minikube_kube-system_b8bdc344ff0000e961009344b94de59c_0
cd72c6dfe3a7   k8s.gcr.io/pause:3.6   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_kube-controller-manager-minikube_kube-system_3db91997554714e5ece3296773cf98a5_0
f5207f8914e8   k8s.gcr.io/pause:3.6   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_kube-apiserver-minikube_kube-system_96be69ce9ff7dc0acff6fda2873a009a_0
a840c15dcfef   k8s.gcr.io/pause:3.6   "/pause"                 2 minutes ago        Up 2 minutes                  k8s_POD_etcd-minikube_kube-system_9d3d310935e5fabe942511eec3e2cd0c_0
```
Проверим, что Kubernetes обладает некоторой устойчивостью к
отказам, удалим все контейнеры:
```
docker@minikube:~$ docker rm -f $(docker ps -a -q)
3c1e82fd994e
7e5668c6e8d4
be6b9a8e10c3
0063ef3522f9
9173d441900a
b04553354921
28f89e56d121
aae3efac0d47
2a02c4749f90
6feceb9ac81e
00c36bc3d78f
48d573e904db
cd72c6dfe3a7
f5207f8914e8
a840c15dcfef
```
Эти же компоненты, но уже в виде pod можно увидеть в namespace kube-system:
```
[docker@kuber ~]$ kubectl get pods -n kube-system
NAME                               READY   STATUS    RESTARTS      AGE
coredns-64897985d-p544v            1/1     Running   1             5m
etcd-minikube                      1/1     Running   1             5m13s
kube-apiserver-minikube            1/1     Running   1             5m13s
kube-controller-manager-minikube   1/1     Running   1             5m13s
kube-proxy-p8llp                   1/1     Running   1             5m
kube-scheduler-minikube            1/1     Running   1             5m13s
storage-provisioner                1/1     Running   3 (86s ago)   5m8s
```

Можно устроить еще одну проверку на прочность и удалить все pod с системными компонентами:
```
[docker@kuber ~]$ kubectl delete pod --all -n kube-system
pod "coredns-64897985d-p544v" deleted
pod "etcd-minikube" deleted
pod "kube-apiserver-minikube" deleted
pod "kube-controller-manager-minikube" deleted
pod "kube-proxy-p8llp" deleted
pod "kube-scheduler-minikube" deleted
pod "storage-provisioner" deleted
```

И проверим что наш кластер находится в рабочем состоянии:
```
[docker@kuber ~]$ kubectl get componentstatuses
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true","reason":""}
[docker@kuber ~]$ kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true","reason":""}
```

Под coredns восстановился потомучто контролиреутся: ReplicaSet
```
[docker@kuber ~]$ kubectl describe pods coredns -n kube-system | grep Controlled
Controlled By:  ReplicaSet/coredns-64897985d
```

Под kube-proxy восстановился потомучто контролиреутся: DaemonSet
```
[docker@kuber ~]$ kubectl describe pods kube-proxy-xrx7s -n kube-system | grep Controlled
Controlled By:  DaemonSet/kube-proxy
```

Оставшиеся поды восстановились потомучто контролировались kubelet ноды:
``` 
[docker@kuber ~]$ kubectl describe pods kube-apiserver-minikube -n kube-system | grep Controlled
Controlled By:  Node/minikube

docker@minikube:~$ ls -l /etc/kubernetes/manifests/
total 16
-rw-------. 1 root root 2309 Feb  3 07:48 etcd.yaml
-rw-------. 1 root root 4071 Feb  3 07:48 kube-apiserver.yaml
-rw-------. 1 root root 3390 Feb  3 07:48 kube-controller-manager.yaml
-rw-------. 1 root root 1436 Feb  3 07:48 kube-scheduler.yaml
```

После того как создали докерфайл и загрузили его в регистри cоздаем web-pod.yaml:
```
[docker@kuber web]$ kubectl apply -f web-pod.yaml
pod/myweb created
[docker@kuber web]$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
myweb   1/1     Running   0          12s
[docker@kuber web]$
```
Посмотрим манфест уже запущенного pod.
```
[docker@kuber web]$ kubectl get pod myweb -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"web"},"name":"myweb","namespace":"default"},"spec":{"containers":[{"image":"2kw92/homework_docker:homework_v1","name":"web"}]}}
  creationTimestamp: "2022-02-03T13:28:23Z"
  labels:
    app: web
  name: myweb
  namespace: default
  resourceVersion: "5071"
  uid: 81a24da6-2c8d-4253-8b04-7faa049b38c4
spec:
  containers:
  - image: 2kw92/homework_docker:homework_v1
    imagePullPolicy: IfNotPresent
    name: web
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-92wkv
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-92wkv
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-02-03T13:28:23Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-02-03T13:28:29Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-02-03T13:28:29Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-02-03T13:28:23Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://088c9c11a6dde41cf70b30580f5454863ea29569eafff8ae040e7cb53b2539ba
    image: 2kw92/homework_docker:homework_v1
    imageID: docker-pullable://2kw92/homework_docker@sha256:91c216946e71dc5eb5f1d97553a4fd6b5b9545a25f9f92b6209e24a2a6fa6de3
    lastState: {}
    name: web
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2022-02-03T13:28:29Z"
  hostIP: 192.168.49.2
  phase: Running
  podIP: 172.17.0.2
  podIPs:
  - ip: 172.17.0.2
  qosClass: BestEffort
  startTime: "2022-02-03T13:28:23Z"
```
Посмотрим что pod успешно стартанул:
```
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m42s  default-scheduler  Successfully assigned default/myweb to minikube
  Normal  Pulling    3m42s  kubelet            Pulling image "2kw92/homework_docker:homework_v1"
  Normal  Pulled     3m37s  kubelet            Successfully pulled image "2kw92/homework_docker:homework_v1" in 4.455288618s
  Normal  Created    3m37s  kubelet            Created container web
  Normal  Started    3m37s  kubelet            Started container web
```

Если изиеним тэк на несуществуюший то поймаем ошибку при поторном запуске pod:
```
  Warning  Failed     15s (x2 over 32s)  kubelet            Failed to pull image "2kw92/homework_docker:homework_v12": rpc error: code = Unknown desc = Error response from daemon: manifest for 2kw92/homework_docker:homework_v12 not found: manifest unknown: manifest unknown
  Warning  Failed     15s (x2 over 32s)  kubelet            Error: ErrImagePull
  Normal   BackOff    4s (x2 over 31s)   kubelet            Back-off pulling image "2kw92/homework_docker:homework_v12"
  Warning  Failed     4s (x2 over 31s)   kubelet            Error: ImagePullBackOff
```

Добавим init-контейнер и снова запустим на pod.
Прверка:
```
[docker@kuber web]$ curl http://localhost:8000/index.html
<html>
<head>
<title>       </title>
<style type="text/css">
<!--
h1      {text-align:center;
        font-family:Arial, Helvetica, Sans-Serif;
        }

p       {text-indent:20px;
        }
-->
</style>
</head>
<body bgcolor = "#ffffcc" text = "#000000">
<h1>Vpered Loko!!!</h1>

</body>
</html>
```
Как мы видим проверка прошла успешно.

HIPSTER SHOP

Стягиваем репо и делаем build:
```
[root@kuber opt]# git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
Cloning into 'microservices-demo'...
remote: Enumerating objects: 4546, done.
remote: Counting objects: 100% (51/51), done.
remote: Compressing objects: 100% (40/40), done.
remote: Total 4546 (delta 25), reused 14 (delta 11), pack-reused 4495
Receiving objects: 100% (4546/4546), 28.85 MiB | 3.08 MiB/s, done.
Resolving deltas: 100% (3034/3034), done.
```
Собиреам докерфайл и пушим его на dockerhub:

```
https://hub.docker.com/repository/docker/2kw92/frontend
```
 Запускаем pod
 ```
[docker@kuber ~]$ kubectl run frontend --image 2kw92/frontend:0.1 --restart=Never
pod/frontend created
 ```

 Смотрим ошибку:
```
[docker@kuber ~]$ kubectl logs frontend
{"message":"Tracing enabled.","severity":"info","timestamp":"2022-02-04T10:41:39.164713124Z"}
{"message":"Profiling enabled.","severity":"info","timestamp":"2022-02-04T10:41:39.164814708Z"}
panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
```

Видим что ошибка из-за не заданой переменной. Зададим пермеенные в yaml и снова запутсим pod:
```
 kubectl describe pod frontend
Name:         frontend
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Fri, 04 Feb 2022 13:17:08 +0000
Labels:       run=frontend
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:  172.17.0.3
Containers:
  frontend:
    Container ID:   docker://098ca09848bdefeaf5368b6351e7eac7240a46a321243c0929de9c360c15c301
    Image:          2kw92/frontend:0.1
    Image ID:       docker-pullable://2kw92/frontend@sha256:fe67d8931aee8c4a1bff1f639b58ede28760bc4cb2c46275b12b2f9f50ce1ee0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 04 Feb 2022 13:17:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      PRODUCT_CATALOG_SERVICE_ADDR:  192.168.1.1
      CURRENCY_SERVICE_ADDR:         192.168.1.2
      CART_SERVICE_ADDR:             192.168.1.3
      RECOMMENDATION_SERVICE_ADDR:   192.168.1.4
      CHECKOUT_SERVICE_ADDR:         192.168.1.5
      SHIPPING_SERVICE_ADDR:         192.168.1.6
      AD_SERVICE_ADDR:               192.168.1.7
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lt9bz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-lt9bz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  64s   default-scheduler  Successfully assigned default/frontend to minikube
  Normal  Pulled     62s   kubelet            Container image "2kw92/frontend:0.1" already present on machine
  Normal  Created    62s   kubelet            Created container frontend
  Normal  Started    62s   kubelet            Started container frontend
```
Видим что все ок.
