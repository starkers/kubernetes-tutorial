

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





### links
- https://mtpereira.com/local-development-k8s.html
