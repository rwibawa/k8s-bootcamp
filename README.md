# kubernetes-bootcamp
* [Kubernetes Tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
* [Minikube Tutorial](https://kubernetes.io/docs/tutorials/hello-minikube/)
* [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [Minikube Cheat Sheet](https://medium.com/@wisegain/minikube-cheat-sheet-a273385e66c9)
* [k8s proxy](https://kubernetes.io/docs/tasks/access-kubernetes-api/http-proxy-access-api/)

## k8s Commands
### 1. Deploy a docker container
```bash
$ kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080
$ kubectl get deployments

$ kubectl proxy &
$ curl http://localhost:8001/version
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.nam}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5c69669756-r6hzs
$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
$ kubectl logs $POD_NAME

$ kubectl get pods
$ kubectl describe pods

$ kubectl exec $POD_NAME env
$ kubectl exec -ti $POD_NAME bash

$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5c69669756-9xl9q   1/1       Running   0          24s
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   40s
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
service/kubernetes-bootcamp exposed
$ kubectl get services
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP          2m
kubernetes-bootcamp   NodePort    10.102.173.108   <none>        8080:32183/TCP   29s
$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.102.173.108
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32183/TCP
Endpoints:                172.18.0.2:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT
NODE_PORT=32183
$ curl $(minikube ip):$NODE_PORT

$ kubectl describe deployment
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Sun, 09 Dec 2018 06:58:30 +0000
Labels:                 run=kubernetes-bootcamp
$ kubectl get pods -l run=kubernetes-bootcamp
$ kubectl get services -l run=kubernetes-bootcamp
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5c69669756-9xl9q
$ kubectl label pod $POD_NAME app=v1
pod/kubernetes-bootcamp-5c69669756-9xl9q labeled
$ kubectl describe pods $POD_NAME
$ kubectl get pods -l app=v1

$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           30s
$ kubectl scale deployments/kubernetes-bootcamp --replicas=4
deployment.extensions/kubernetes-bootcamp scaled
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4         4         4            4           47s
$ kubectl get pods -o wide
NAME                                   READY     STATUS    RESTARTS   AGE       IP           NODE
kubernetes-bootcamp-5c69669756-2mdgv   1/1       Running   0          31s       172.18.0.7   minikube
kubernetes-bootcamp-5c69669756-8zrt9   1/1       Running   0          31s       172.18.0.5   minikube
kubernetes-bootcamp-5c69669756-tmzjq   1/1       Running   0          31s       172.18.0.6   minikube
kubernetes-bootcamp-5c69669756-zs775   1/1       Running   0          1m        172.18.0.2   minikube
```

### 2. Update with a new version
```bash
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
$ kubectl rollout status deployments/kubernetes-bootcamp
deployment "kubernetes-bootcamp" successfully rolled out
```

### 3. Rollback an update
```bash
$ kubectl rollout undo deployments/kubernetes-bootcamp
deployment.extensions/kubernetes-bootcamp
```

## Connect to a remote minikube cluster
### Local host:
```bash
$ scp -r ryan@73.202.185.98:/home/ryan/.minikube/client.* ~/.minikube/
$ scp -r ryan@73.202.185.98:/home/ryan/.kube ~/
$ cat ~/.kube/config
piVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: http://localhost:30000
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/rwibawa/.minikube/client.crt
    client-key: /home/rwibawa/.minikube/client.key

$ export DOCKER_TLS_VERIFY="1"
$ export DOCKER_API_VERSION="1.35"
$ export DOCKER_HOST="tcp://localhost:30001"
$ export DOCKER_CERT_PATH="/home/rwibawa/.minikube/certs"
$ docker images
```

### Remote host:
```bash
$ ssh -L 30000:localhost:8001 -L 30001:192.168.39.32:2376 ryan@192.168.1.101
$ kubectl proxy &
```

## k8s Links
* [k8s dashboard](http://127.0.0.1:30000/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/)
* [k8s ui](http://localhost:30000/ui)
* [http://localhost:30000/api/v1/namespaces/default/pods/$POD_NAME/proxy/](http://localhost:30000/api/v1/namespaces/default/pods/nginx-app-56f6bb6776-zvpqw/proxy/)

## k8s with private docker registry
* [local k8s cluster with insecure registries](https://medium.com/@alombarte/setting-up-a-local-kubernetes-cluster-with-insecure-registries-f5aaa34851ae)
* [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
```bash
$ minikube stop
$ minikube delete
Deleting local Kubernetes cluster...
Machine deleted.
$ minikube start --vm-driver kvm2 --cpus 6 --disk-size 300g --memory 32000 --insecure-registry "hela:8443"

$ minikube addons configure registry-creds -disk-size
Do you want to enable AWS Elastic Container Registry? [y/n]: n
Do you want to enable Google Container Registry? [y/n]: n
Do you want to enable Docker Registry? [y/n]: y
-- Enter docker registry server url: hela:8443
-- Enter docker registry username: rwibawa
-- Enter docker registry password:
registry-creds was successfully configured
$ minikube addons enable registry-creds
registry-creds was successfully enabled

$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ docker login -u <username> -p <password> hela:8443
Login Succeeded
$ docker pull hela:8443/nginx:1.15.6-alpine
Error response from daemon: manifest for hela:8443/nginx:1.15.6-alpine not found
$ docker pull hela:8443/hello-k8s:v1
v1: Pulling from hello-k8s
Digest: sha256:036874a2767bcbbb755ca91fca74508e7e6fdaafa1d84977e2e8ad6aa969daa5
Status: Downloaded newer image for hela:8443/hello-k8s:v1

$ kubectl create secret docker-registry regcred --docker-server="https://hela:8443" --docker-username="rwibawa" --docker-password="<password>" --docker-email="ryan.wibawa@gmail.com"
secret/regcred created
$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
regcred               kubernetes.io/dockerconfigjson        1      5s

$ kubectl get secret regcred --output=yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJoZWxhOjg0NDMiOnsiVXNlcm5hbWUiOiJyd2liYXdhIiwiUGFzc3dvcmQiOiJBc3RhZ2EwMDEhIiwiRW1haWwiOiJyeWFuLndpYmF3YUBnbWFpbC5jb20ifX19
kind: Secret
metadata:
  creationTimestamp: "2018-12-09T19:35:01Z"
  name: regcred
  namespace: default
  resourceVersion: "961992"
  selfLink: /api/v1/namespaces/default/secrets/regcred
  uid: 852b147f-fbe9-11e8-a347-8cd514c64884
type: kubernetes.io/dockerconfigjson

$ kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode

$ docker build -t hela:8443/hello-k8s:v1 .
$ docker run -d --name hello-k8s -p 8096:8080 hela:8443/hello-k8s:v1

$ curl http://localhost:8096
Hello Kubernetes bootcamp! | Running on: 2f1833b9cbeb | v=1
$ curl http://localhost:8096
Hello Kubernetes bootcamp! | Running on: 2f1833b9cbeb | v=1
$ docker logs hello-k8s
Kubernetes Bootcamp App Started At: 2018-12-09T19:43:31.756Z | Running On:  2f1833b9cbeb

Running On: 2f1833b9cbeb | Total Requests: 1 | App Uptime: 28.161 seconds | Log Time: 2018-12-09T19:43:59.917Z
Running On: 2f1833b9cbeb | Total Requests: 2 | App Uptime: 30.709 seconds | Log Time: 2018-12-09T19:44:02.465Z

$ docker push hela:8443/hello-k8s:v1

$ kubectl create -f my-private-reg-pod.yaml
$ kubectl get pod private-reg
```
