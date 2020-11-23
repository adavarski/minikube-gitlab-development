# Developing for Kubernetes with minikube+GitLab

<img src="https://raw.githubusercontent.com/adavarski/minikube-gitlab-development//main/pictures/devops-toolchain-cicd.png?raw=true" width="800">



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

### Go to http://gitlab and configure GitLab (groups, k8s clusters, etc.)

Example: Integrate GitLab with minikube k8s cluster running locally on your laptop (minikube where GitLab is deployed):

```

$ kubectl cluster-info | grep 'Kubernetes master' | awk '/http/ {print $NF}'
https://192.168.99.101:8443

$ kubectl get secret $(kubectl get secret | grep default-token | awk '{print $1}') -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
a3ViZUNBMB4XDTIwMTExNjIzMTQ0MFoXDTMwMTExNTIzMTQ0MFowFTETMBEGA1UE
AxMKbWluaWt1YmVDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOjO
8JnDb7xLQ8UNGeQ81V8AWeuLOvuM9Bf45cY95pIllsOajPZeihSKbwyIGlewonrq
cT6a1temY3/xz5kvQIXoAnhTcpRpBr+ABrDr7OlJV8auSavkBj3XIBl80qycHY2H
slwPzX3u45bvhFwUnuUbRuFboLc4XTTRumN/V64iWIor7mkZEXNq1dBrLLdd51o9
U61DNOIfhMXOnusJDA8sxcIerxyzoFMysJghId6yDg2AUHIIBqCtEWmJMdDlxyBc
x3cREwqLDkez2+w1+cHazoVRPDhiPkN+28+tVKPZ9p4zKdwLdkQ+YccA0+LHP0VM
AiN71ObiJB1wwGeUKlMCAwEAAaNCMEAwDgYDVR0PAQH/BAQDAgKkMB0GA1UdJQQW
MBQGCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3
DQEBCwUAA4IBAQBptuv1zpxDGo4iik+rRgI49TpldkCBLuYnM100ZcudmQKCd4xv
bFpyZivNd/DgfCg0JgI6O4Ousz97MqjkgY8itdMsmiaPYKbBjwFMOL2Ly38t8PVA
U0ErF2R+aoTo6NjQJzqt/jCPOgIUGJv73S/9ajl9z0/bOHwQl0qz78OQkaRfg/Wv
feZW0kLi8+d4EyNIrF57jLuwxpyAwUC5oySkj0IZZYf8qeq6Y7o0i07c9RrpNDl6
r2jFQm0MsJZB8egFZSEMPwPhdj0zdIdv29YezNli8AuDa+7+u7/i9exncmiHR8r7
Gn1ytp/St1cV1OGnrVgK1gk0Mz4FRuC9/03v
-----END CERTIFICATE-----

$ cd k8s/utils
$ kubectl create -f gitlab-admin-service-account.yaml 
serviceaccount/gitlab created
clusterrolebinding.rbac.authorization.k8s.io/gitlab-admin created  

$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab | awk '{print $1}')
Name:         gitlab-token-mhmjq
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: gitlab
              kubernetes.io/service-account.uid: 8f0ad85c-df9b-433f-8653-2ba3605e2106

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkVJcXRVMlhWa3FxZTFPVXZIVnBGWF81UEpKVVYyOWdFNUx4SVNHdnZQTHcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJnaXRsYWItdG9rZW4tbWhtanEiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZ2l0bGFiIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiOGYwYWQ4NWMtZGY5Yi00MzNmLTg2NTMtMmJhMzYwNWUyMTA2Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmdpdGxhYiJ9.W5VFKmL0GwJpKfxWHo-PDC0CdwRS4G6CZZjc5AGGHq8I1zEnGkiu_-JgHGTqCaB6PAnebAIowxX8bfmsGu9T0A7inlSYIvt36MYshb4-r0hP59NH4cFn3hPrrVB6NCwYPMYlVGy81br3EqJqYQ2SvADKYbf-j4dbAHh0ybmJId2euyuDfBBOpyzE99V5wW0OlTxRQgHGcR0wmvfsxKEw566kvjP4LlfRjRQQxiWYIE_WU5if1gVQvFa0Slfa_CrdQ3zj2iT5qYADKRG45QCkMJjf0STJL9ifPkax3THuU6BlU2rXjP2G5qwHcteM3i968KnUGF-9QDZMZcuxY6R8_g
```
Open http://gitlab/ -> create group, and integrate group k8s minikube cluster providing above: API URL (https://192.168.99.102:8443), CA Certificate and Service Token. Install Helm, GitLab Runner, etc.

<img src="https://raw.githubusercontent.com/adavarski/minikube-gitlab-development//main/pictures/gitlab-minikube-add-existing-cluster.png?raw=true" width="800">



Note: GitLab can be integrated (Add existing k8s cluster) with kubernetes versions < v1.18

GitLab supports (october.2020) the following Kubernetes versions (check suported versions: https://docs.gitlab.com/ee/user/project/clusters/), and you can upgrade your Kubernetes version to any supported version at any time:
```
1.17
1.16
1.15
1.14 (deprecated, support ends on December 22, 2020)
1.13 (deprecated, support ends on November 22, 2020)
```
The helm tiller can't be installed in kubernetes v1.18.+ because is not supported Gitlab versions (helm tiller install issue, and helm is needed for all the other GitLab sub apps depend on Helm Tiller: you cannot use cert manager, ingress, etc.) :

```
$ kubectl get pod -n gitlab-managed-apps
NAME           READY   STATUS   RESTARTS   AGE
install-helm   0/1     Error    0          18h
$ kubectl logs install-helm -n gitlab-managed-apps
+ helm init --tiller-tls --tiller-tls-verify --tls-ca-cert /data/helm/helm/config/ca.pem --tiller-tls-cert /data/helm/helm/config/cert.pem --tiller-tls-key /data/helm/helm/config/key.pem --service-account tiller
Creating /root/.helm 
Creating /root/.helm/repository 
Creating /root/.helm/repository/cache 
Creating /root/.helm/repository/local 
Creating /root/.helm/plugins 
Creating /root/.helm/starters 
Creating /root/.helm/cache/archive 
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /root/.helm.
Error: error installing: the server could not find the requested resource
```
