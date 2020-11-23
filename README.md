# Developing for Kubernetes with minikube+GitLab

### Install minikube and kubectl (the same k8s minor versions for minikube k8s cluster and kubectl: v1.16.2 for example)

~~~
https://github.com/kubernetes/minikube/releases
~~~

```
$ curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.5.2/minikube-linux-amd64 && chmod +x minikube && sudo mv ./minikube /usr/local/bin/
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.16.2/bin/linux/amd64/kubectl && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:18:23Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
```

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

### Go to http://gitlab and configure GitLab (groups, k8s clusters, etc)

