# Developing for Kubernetes with minikube+GitLab

### Install minikube and kubectl

Note: install the same k8s minor versions for minikube k8s cluster and kubectl (v1.16.2 for example)

```
$ curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.5.2/minikube-linux-amd64 && chmod +x minikube && sudo mv ./minikube /usr/local/bin/
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.16.2/bin/linux/amd64/kubectl && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:18:23Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
```
~~~
https://github.com/kubernetes/minikube/releases
~~~

### Start minikube and enable minikube ingress (nginx)

```
$ minikube start --cpus 2 --memory 4096
$ minikube addons enable ingress
```
Note: If you don't want to use minikube ingress controler, but your own custom ingress (example):

```
$ cd 100-ingress; for i in `ls -1 *.yaml`; do kubectl create -f $i;done
```

### Add ip to hostfile

```
echo "### GitLab on minikube" |sudo tee -a /etc/hosts
echo $(minikube ip)" gitlab"|sudo tee -a /etc/hosts
```
### Create internal docker-registry

```
$ cd 200-docker-registry; for i in `ls -1 *.yaml`; do kubectl create -f $i;done
```

### Create gitlab server

```
$ cd 300-gitlab; for i in `ls -1 *.yaml`; do kubectl create -f $i;done
```

Note: You can create secret with

~~~
kubectl create secret generic gitlab-secret --namespace gitlab --from-file=./gitlab/GITLAB_OMNIBUS_CONFIG --from-file=./gitlab/REGISTRATION_TOKEN
~~~

Example:

```
$ kubectl delete -f 20-gitlab-secret.yaml
$ kubectl create secret generic gitlab-secret --namespace gitlab --from-file=./GITLAB_OMNIBUS_CONFIG --from-file=./REGISTRATION_TOKEN
secret/gitlab-secret created
$ kubectl get pod -n gitlab
NAME                            READY   STATUS    RESTARTS   AGE
gitlab-7997f688d8-696f9         1/1     Running   0          12m
$ kubectl delete -f 50-gitlab-ingress.yaml
ingress.extensions "gitlab" deleted
```
### Create gitlab runner
```
$ cd 400-gitlab-runner; for i in `ls -1 *.yaml`; do kubectl create -f $i;done
```

### Check minikube

```
$ kubectl get all --all-namespaces
NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE
gitlab        pod/gitlab-7997f688d8-696f9                     1/1     Running   0          6m3s
gitlab        pod/gitlab-runner-dd9696c8b-l5qvc               2/2     Running   0          5m33s
kube-system   pod/coredns-5644d7b6d9-tgbjx                    1/1     Running   0          13m
kube-system   pod/coredns-5644d7b6d9-xmvzk                    1/1     Running   0          13m
kube-system   pod/etcd-minikube                               1/1     Running   0          12m
kube-system   pod/kube-addon-manager-minikube                 1/1     Running   0          13m
kube-system   pod/kube-apiserver-minikube                     1/1     Running   0          12m
kube-system   pod/kube-controller-manager-minikube            1/1     Running   0          12m
kube-system   pod/kube-proxy-pv5t7                            1/1     Running   0          13m
kube-system   pod/kube-registry-proxy-tdzfd                   1/1     Running   0          7m2s
kube-system   pod/kube-registry-wkcm9                         1/1     Running   0          6m55s
kube-system   pod/kube-scheduler-minikube                     1/1     Running   0          12m
kube-system   pod/nginx-ingress-controller-6fc5bcc8c9-t89rn   1/1     Running   0          13m
kube-system   pod/storage-provisioner                         1/1     Running   0          13m

NAMESPACE     NAME                                  DESIRED   CURRENT   READY   AGE
kube-system   replicationcontroller/kube-registry   1         1         1       6m55s

NAMESPACE     NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP                  13m
gitlab        service/gitlab          NodePort    10.101.203.56   <none>        80:30623/TCP             5m57s
kube-system   service/kube-dns        ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   13m
kube-system   service/kube-registry   NodePort    10.96.162.226   <none>        5000:31246/TCP           6m49s

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-system   daemonset.apps/kube-proxy            1         1         1       1            1           beta.kubernetes.io/os=linux   13m
kube-system   daemonset.apps/kube-registry-proxy   1         1         1       1            1           <none>                        7m2s

NAMESPACE     NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
gitlab        deployment.apps/gitlab                     1/1     1            1           6m3s
gitlab        deployment.apps/gitlab-runner              1/1     1            1           5m33s
kube-system   deployment.apps/coredns                    2/2     2            2           13m
kube-system   deployment.apps/nginx-ingress-controller   1/1     1            1           13m

NAMESPACE     NAME                                                  DESIRED   CURRENT   READY   AGE
gitlab        replicaset.apps/gitlab-7997f688d8                     1         1         1       6m3s
gitlab        replicaset.apps/gitlab-runner-dd9696c8b               1         1         1       5m33s
kube-system   replicaset.apps/coredns-5644d7b6d9                    2         2         2       13m
kube-system   replicaset.apps/nginx-ingress-controller-6fc5bcc8c9   1         1         1       13m

$ kubectl-minikube get ing --all-namespaces
NAMESPACE     NAME              HOSTS            ADDRESS          PORTS     AGE
gitlab        gitlab            gitlab           192.168.99.101   80        5m57s
kube-system   docker-registry   registry.local   192.168.99.101   80, 443   6m49s

```

### Go to http://gitlab and configure GitLab (groups, k8s clusters, etc)

