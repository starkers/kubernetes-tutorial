

# Download and install:
- kubectl
- minikube
- k8stail



# URLS for linux
- https://github.com/dtan4/k8stail/releases/download/v0.2.1/k8stail-v0.2.1-linux-amd64.zip
- https://storage.googleapis.com/kubernetes-release/release/v1.5.2/bin/linux/amd64/kubectl
- https://storage.googleapis.com/minikube/releases/v0.15.0/minikube-linux-amd64


# start minikube and tell it your registry is self-contained
minikube start --insecure-registry localhost:5000

## ensure your "docker" commands use the docker inside minikube

eval $(minikube docker-env)


# check status of your "cluster"

```
ᐅ  kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

# deploy the local-registry
```
ᐅ  kubectl apply -f ./local-registry.yml
replicationcontroller "kube-registry-v0" created
service "kube-registry" created
pod "kube-registry-proxy" created
```


# what just happened?

Looking at the previous output we can see we just deployed three things:
- a replicationcontroler (lets call them *rc*'s)
- a service (like a loadbalancer)
- a pod (the actual container)


You can see the pods running in your minikube with:

`k get pods`

Note that we do NOT see one called `kube-registry-proxy`

Thats because its deployed them into a different `namespace`

# namespaces

namespaces are used to logically seperate deployed apps.
If you deploy:
 - app1 into namespace foo
 - app2 into namespace bar

app1 and app2 will not be able to communicate with each other

There is also a special one called `kube-system`
This is a priv place.. essentially "ops" stuff will go here.

a quick `grep namespace local-registry.yml` will reveal that this got deployed into the kube-system namespace.


so lets add `--namespace=kube-system` to `k get pods` now:

```
ᐅ  kubectl --namespace=kube-system get pods
NAME                          READY     STATUS    RESTARTS   AGE
kube-addon-manager-minikube   1/1       Running   0          12m
kube-dns-v20-s51mx            3/3       Running   0          11m
kube-registry-proxy           1/1       Running   0          5m
kube-registry-v0-g16mf        1/1       Running   0          5m
kubernetes-dashboard-09s1j    1/1       Running   0          12m
kubernetes-dashboard-b4rzn    1/1       Running   1          88d
```





# getting the service from k8s to your localhost


get a list of services in kube-system:
```
ᐅ  kubectl get svc --namespace=kube-system
NAME                   CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kube-dns               10.0.0.10    <none>        53/UDP,53/TCP   88d
kube-registry          10.0.0.122   <none>        5000/TCP        25m
kubernetes-dashboard   10.0.0.228   <nodes>       80:30000/TCP    88d
```

Lets see the the ports and details on the registry:
```
kubectl describe svc/kube-registry --namespace=kube-system
Name:     kube-registry
Namespace:    kube-system
Labels:     k8s-app=kube-registry
      kubernetes.io/name=KubeRegistry
      Selector:   k8s-app=kube-registry
      Type:     ClusterIP
      IP:     10.0.0.122
      Port:     registry  5000/TCP
      Endpoints:    172.17.0.7:5000
      Session Affinity: None
      No events.
```

We can "port-forward" :5000 using kubectl.. then you will have a docker registry running localy on :5000


```
POD=$(kubectl get pods --namespace kube-system -l k8s-app=kube-registry \
      -o template --template '{{range .items}}{{.metadata.name}} {{.status.phase}}{{"\n"}}{{end}}' \
      | grep Running | head -1 | cut -f1 -d' ')
kubectl port-forward --namespace kube-system $POD 5000:5000 &
```

verify its up:

```
curl localhost:5000
Handling connection for 5000
```

# lets tag something and push it

```
ᐅ  cd sample_app_hello

ᐅ  docker build .
Sending build context to Docker daemon 3.072 kB
Step 1 : FROM caarlos0/alpine-go
 ---> 3c5fe9414eed
Step 2 : WORKDIR /gopath/src/app
 ---> Using cache
 ---> 3e4821d29fa7
Step 3 : ADD . /gopath/src/app/
 ---> Using cache
 ---> cbd27bb09b2e
Step 4 : RUN go get -v app
 ---> Using cache
 ---> 8f70b96bac7f
Step 5 : ENTRYPOINT /gopath/bin/app
 ---> Using cache
 ---> 532abd0000aa
Successfully built 532abd0000aa

ᐅ  docker tag 532abd0000aa localhost:5000/hello/1

ᐅ  docker push localhost:5000/hello/1
The push refers to a repository [localhost:5000/hello/1]
bb810286635f: Mounted from hello/three 
8184b846aefd: Mounted from hello/three 
51a8a83b9792: Mounted from hello/three 
5f70bf18a086: Mounted from hello/three 
c954c0da97ee: Mounted from hello/three 
745737c319fa: Mounted from hello/three 
latest: digest: sha256:2052725c4f72a37fb3c4b43ae0e88b7a537f37041e2bdcdf7eb3271bdfbcb1b2 size: 2395
```

OK so now you should be able to deploy it!



### links
- https://mtpereira.com/local-development-k8s.html
- https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/registry
