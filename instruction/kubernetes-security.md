# homework №3

task_1

Создаем сервис аккаунт bob и проверяем:
```
[docker@kuber 3_hw]$ kubectl apply -f sa_bob.yaml
clusterrolebinding.rbac.authorization.k8s.io/crb-bob created
[docker@kuber 3_hw]$ kubectl auth can-i get deployments --as system:serviceaccount:default:bob -n default
yes
```

Создаем сервис акаунт dave и проверяем:
```
[docker@kuber 3_hw]$ kubectl apply -f sa-dave.yaml
serviceaccount/dave created
[docker@kuber 3_hw]$ kubectl auth can-i get deployments --as system:serviceaccount:default:dave
no
```

task_2

Создаем namespace:
```
[docker@kuber task02]$ kubectl apply -f ns-prometheus.yaml
namespace/prometheus created
```
Создаем сервис аккаунт
```
[docker@kuber task02]$ kubectl apply -f sa-carol.yaml
serviceaccount/carol created
```
Создаем ClusterRole 
```
[docker@kuber task02]$ kubectl apply -f cr-pod-reader
clusterrole.rbac.authorization.k8s.io/pod-reader created
```
И создаем ClusterRoleBinding
```
[docker@kuber task02]$ kubectl apply -f crb-prometheus.yaml
clusterrolebinding.rbac.authorization.k8s.io/rb-prometheus created
```

И теперь делаем проверку:
```
[docker@kuber task02]$ kubectl auth can-i get replicaset --as system:serviceaccount:prometheus:carol -n prometheus
no
[docker@kuber task02]$ kubectl auth can-i list pods --as system:serviceaccount:prometheus:carol
yes
[docker@kuber task02]$ kubectl auth can-i list pods --as system:serviceaccount:prometheus:carol -n prometheus
yes
[docker@kuber task02]$ kubectl auth can-i get pods --as system:serviceaccount:prometheus:carol -n prometheus
yes
[docker@kuber task02]$ kubectl auth can-i get deployments --as system:serviceaccount:prometheus:carol
no
[docker@kuber task02]$ kubectl auth can-i get pods --as system:serviceaccount:prometheus:carol -n prometheus
yes
```
И видим что созданные нами сервис аккаунт имеет возможность делать get, list, watch в отношении только Pods всего кластера 

task_3

Создадим необходимые сервис-акакунты namespace и RoleBinding и применим их
```
[docker@kuber task03]$ kubectl apply -f ns-dev.yaml
namespace/dev created
[docker@kuber task03]$ kubectl apply -f sa-jane.yaml
serviceaccount/jane created
[docker@kuber task03]$ kubectl apply -f rb-jane.yaml
rolebinding.rbac.authorization.k8s.io/rb-jane created
[docker@kuber task03]$ kubectl apply -f sa-ken.yaml
serviceaccount/ken created
[docker@kuber task03]$ kubectl apply -f rb-ken.yaml
rolebinding.rbac.authorization.k8s.io/rb-ken created
[docker@kuber task03]$
```
И теперь проверим:
```
[docker@kuber task03]$ kubectl auth can-i create deployments --as system:serviceaccount:dev:jane -n dev
yes
[docker@kuber task03]$ kubectl auth can-i create deployments --as system:serviceaccount:dev:jane
no
[docker@kuber task03]$ kubectl auth can-i create pods --as system:serviceaccount:dev:jane
no
[docker@kuber task03]$ kubectl auth can-i create pods --as system:serviceaccount:dev:ken
no
[docker@kuber task03]$ kubectl auth can-i create pods --as system:serviceaccount:dev:ken -n dev
no
[docker@kuber task03]$ kubectl auth can-i get pods --as system:serviceaccount:dev:ken -n dev
yes
```
Видим что права соответсвует заданию описанном в домашем задании.
