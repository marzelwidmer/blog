---
title: Kubernetes Ingress with Ngnix Ingress Controller
subTitle: Set up Ingress on Minikube with the Ngnix Ingress Controller  
date: "2020-05-01"
draft: false
tags: [k8s]
ShowToc: true
TocOpen: true
---

# OSX Minikube Kubernetes 
If you use `ohmyzsh`  is there a nice Plugin with some `kubectl` for more details check [ohmyzsh plugins kubectl](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/kubectl) 

## Install Minikube with Ingress Controller
```bash
brew install minikube
```

## Start Minikube
```bash
minikube start
😄  minikube v1.9.2 on Darwin 10.15.4
    ▪ MINIKUBE_ACTIVE_DOCKERD=minikube
✨  Using the hyperkit driver based on existing profile
👍  Starting control plane node m01 in cluster minikube
🔄  Restarting existing hyperkit VM for "minikube" ...
🐳  Preparing Kubernetes v1.18.0 on Docker 19.03.8 ...
🌟  Enabling addons: dashboard, default-storageclass, ingress, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube"
```

## Minkikube Status
```bash
minikube status
m01
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
## Enable Ingress Addons
We need the addon `ingress` this can be done with the followng command.
```bash
minikube addons enable ingress
```
Check minikube addons with `minikube addons list` if `ingress` is enabled.

Run the following command:
```bash
minikube addons list

|-----------------------------|----------|--------------|
|         ADDON NAME          | PROFILE  |    STATUS    |
|-----------------------------|----------|--------------|
| dashboard                   | minikube | enabled ✅   |
| default-storageclass        | minikube | enabled ✅   |
| efk                         | minikube | disabled     |
| freshpod                    | minikube | disabled     |
| gvisor                      | minikube | disabled     |
| helm-tiller                 | minikube | disabled     |
| ingress                     | minikube | enabled ✅   |
| ingress-dns                 | minikube | disabled     |
| istio                       | minikube | disabled     |
| istio-provisioner           | minikube | disabled     |
| logviewer                   | minikube | disabled     |
| metrics-server              | minikube | disabled     |
| nvidia-driver-installer     | minikube | disabled     |
| nvidia-gpu-device-plugin    | minikube | disabled     |
| registry                    | minikube | disabled     |
| registry-aliases            | minikube | disabled     |
| registry-creds              | minikube | disabled     |
| storage-provisioner         | minikube | enabled ✅   |
| storage-provisioner-gluster | minikube | disabled     |
|-----------------------------|----------|--------------|
```

## Ingress Controller 
Let's check if the `Nginx Ingress Controller` is up and running.

Run the following command:
```bash
k get pods -n kube-system | grep nginx-ingress
nginx-ingress-controller-6d57c87cb9-rtgpk   1/1     Running   1          12m
```

# Create Namespace
Let's create first a `dev` name space where we deploy our sample application.

Run the following command:
```bash
k create ns dev
namespace/dev created
```
# Switch Namespace Permanently
With the following command we can switch the namespace permanently like `oc project` in a `OpenShift` environment.
This command will save the namespace for all other `kubectl` commands in that context. 
This required the [ohmyzsh plugins kubectl](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/kubectl) Plugin.
Without this plugin will be the command `kubectl config set-context --current --namespace=dev`.
```bash
kcn dev
```

# Create Deployment
Let's create a deployment now for a sample application with the name `myapp` with the image `google-samples/hello-app:1.0` in the current namespace `dev`

Run the following command:
```bash
k create deployment myapp --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/myapp created
```

If you want see what deployments we have you can use `kubectl get deployments` command.

Run the following command:
```bash
k get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
myapp   1/1     1            1           44m
```

Let's also check if the pod is running.

Run the following command:
```bash
k get pods -n dev -o wide --show-labels --field-selector=status.phase=Running

NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES   LABELS
myapp-75d86849b-2gbps   1/1     Running   0          67m   172.17.0.3   minikube   <none>           <none>            app=myapp,pod-template-hash=75d86849b
```


# Create Service
To reach our application we have to create a `service` this can be easily done with the `expose` command and the `NodePort` 

```bash
k expose deployment myapp --port=8080 --type=NodePort
```

Let's check the service.

Run the following command:
```bash
k get svc
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
myapp   NodePort   10.110.101.126   <none>        8080:30506/TCP   21m
```


## Test Service internal in a Cluster
Let's check the service with a small container based on the `tutum/curl:alpine` Docker image, which contains the curl command.
Run the `curl -s 'http://myapp:8080'` command inside the container and redirect the output to the Terminal using the `-i` option.
Delete the pod using the `--rm` option.

Run the following command:
```bash
k run -i --rm --restart=Never curl-client --image=tutum/curl:alpine --command -- curl -s 'http://myapp:8080'

Hello, world!
Version: 1.0.0
Hostname: myapp-75d86849b-2gbps
pod "curl-client" deleted
```

This mean we can call a pod internal a cluster with the internal `DNS`.

## Test Service outside a Cluster
Let's call again the command `k get svc` to get the exposed port from out service.
Run the following command again :
```bash
k get svc
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
myapp   NodePort   10.110.101.126   <none>        8080:30506/TCP   21m
```

You see the exposed port is here `30506` now let's get the `IP` address from out minikube with `minikube ip`
```bash
minikube ip
192.168.64.18
```

Now let's also call out service `myapp` with `HTTPie` or in a browser `http://192.168.64.18:30506`
```bash
http http://192.168.64.18:30506

HTTP/1.1 200 OK
Content-Length: 61
Content-Type: text/plain; charset=utf-8
Date: Sun, 03 May 2020 08:07:35 GMT

Hello, world!
Version: 1.0.0
Hostname: myapp-75d86849b-2gbps
```


# /etc/hosts
To get on the service with the URL `minikube.me` we update now out local `/etc/hosts` file with `IP` address from your minikube `minikube ip`
Run the following command:
```bash
echo $(minikube ip) minikube.me | sudo tee -a /etc/hosts

Password:
192.168.64.11 minikube.me
```

Verify it the `IP` is in the `/etc/hosts` with the following command:
```bash
cat /etc/hosts | tail -n 1

192.168.64.11 minikube.me
```

Now you can also call the service wit the `DNS`  `http://minikube.me:30506`
```bash
http http://minikube.me:30506

HTTP/1.1 200 OK
Content-Length: 61
Content-Type: text/plain; charset=utf-8
Date: Sun, 03 May 2020 08:12:51 GMT

Hello, world!
Version: 1.0.0
Hostname: myapp-75d86849b-2gbps
```

#  Nginx Ingress 
Now is time to create our `nginx ingress` for this we creat a file named `nginx-ingess.yaml`
where we point `/` to the `myapp` service on internal port `8080`.

Run the following command:
```bash
echo  "apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
 name: nginx-ingress
 namespace: dev
 annotations:
   nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
 rules:
 - host: minikube.me
   http:
     paths:
     - path: /
       backend:
         serviceName: myapp
         servicePort: 8080"  | > nginx-ingess.yaml
```

Now let's also take a new `alias` in the game `kaf` is the alias for `kubectl -f` and let's apply the `nginx-ingess.yaml` file. 
For this run the following command:

```bash
kaf nginx-ingess.yaml

ingress.networking.k8s.io/nginx-ingress created
```

With `kubectl get ingress` you can list all ingress in the namespace.
```bash
k get ingress

NAME            CLASS    HOSTS         ADDRESS   PORTS   AGE
nginx-ingress   <none>   minikube.me             80      22s
```

## Test Ingress
Open a browser now `http://minikube.me` or with `HTTPie` in a terminal.

```bash
http http://minikube.me

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 61
Content-Type: text/plain; charset=utf-8
Date: Sun, 03 May 2020 09:18:45 GMT
Server: openresty/1.15.8.2

Hello, world!
Version: 1.0.0
Hostname: myapp-75d86849b-v72vs
```

Now let's also deploy a second application and expose it as service.
Configure the `nginx ingress` also to the second service.

```bash
k create deployment myapp2 --image=gcr.io/google-samples/hello-app:2.0
k expose deployment myapp2 --port=8080 --type=NodePort
``` 

Update the `nginx-ingess.yaml` file with `vi`
```yaml
  - path: /v2/*
    backend:
      serviceName: myapp2
      servicePort: 8080
```
Check the `nginx-ingess.yaml` file with `cat nginx-ingess.yaml`
```bash
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
 name: nginx-ingress
 namespace: dev
 annotations:
   nginx.ingress.kubernetes.io/rewrite-target: /
spec:
 rules:
 - host: minikube.me
   http:
     paths:
     - path: /
       backend:
         serviceName: myapp
         servicePort: 8080     
     - path: /v2
       backend:
         serviceName: myapp2
         servicePort: 8080
```

Apply the changes with the following command:
```bash
kaf nginx-ingess.yaml
ingress.networking.k8s.io/nginx-ingress configured

```

Now we have one Ingress who point to 2 individual services.
Verify it with `HTTPie` or with the browser `http://minikube.me/` will give back `Version: 1.0.0` 
```bash
http http://minikube.me/

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 61
Content-Type: text/plain; charset=utf-8
Date: Sun, 03 May 2020 09:31:12 GMT
Server: openresty/1.15.8.2

Hello, world!
Version: 1.0.0
Hostname: myapp-75d86849b-v72vs
```

On the second route `http://minikube.me/v2` we get back `Version: 2.0.0`

```bash
http http://minikube.me/v2

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 63
Content-Type: text/plain; charset=utf-8
Date: Sun, 03 May 2020 09:32:23 GMT
Server: openresty/1.15.8.2

Hello, world!
Version: 2.0.0
Hostname: myapp2-548b98644f-rqqmp
```



# kubectl Cheat Sheet
```bash
k get pods -o wide --show-labels  --field-selector=status.phase=Running  -w   | grep -vi "Terminating"
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES   LABELS
web-6785d44d5-vb2wb    1/1     Running   1          19h   172.17.0.4   minikube   <none>           <none>            app=web,pod-template-hash=6785d44d5
web2                   1/1     Running   1          19h   172.17.0.3   minikube   <none>           <none>            run=web2
web2-8474c56fd-wk4kh   1/1     Running   1          18h   172.17.0.5   minikube   <none>           <none>            app=web2,pod-template-hash=8474c56fd
```

## Namespaces 
* `kubectl get ns` 
* `k get ns`  
* `kgns`
```bash
k get ns
NAME              STATUS   AGE
default           Active   19h
kube-node-lease   Active   19h
kube-public       Active   19h
kube-system       Active   19h
```

## Delete with given label
```bash
k delete all -l  app=web
```


## Troubleshooting
```
sudo dscacheutil -flushcache;sudo killall -HUP mDNSResponder
```

---
_References:_
> [ingress-minikube](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)
