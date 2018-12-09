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
