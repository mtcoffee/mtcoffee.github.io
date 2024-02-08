---
title: Kubernetes Demystified and ServiceNow Discovery part1
---

# Summary 
What is Kubernetes exactly? If you're not in the DevOps space it can sound like voodoo. Let's start with a little history. 

Once upon a time, we had to run everything on **Physical Hardware** and then along came **Virtual Machines**! Now we can run many isolated servers on a single piece of hardware thanks to products like **VMware**. However, as time progressed along came an even more light weight option called **Containers**. Containers offer rapid deployment and are **ephemeral** in nature. You can build them up and throw them away since the data is typically abstracted from the application. The most widely known example of this is **Docker**. However, Docker presented a new problem. Managing all of these containers quickly became challenging for large organizations. This is where **Kubernetes** comes in. It's purpose is to handle the orchestration and management of all of these containers, including load balancing, scaling and more.

# Instant Kubernetes Cluster with K3s
Let's see this in a real world example.  You will need a Linux Host to run your Kubernetes install on. Ubuntu is an easy distro to start with. We will use K3s which is a special lightweight Kubernetes distribution that is certified for production use. In this example we're running it on a 2GB and 1CPU host.

1. Install K3s with the ServiceLB. Open and command shell and paste this in:
```
sudo curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -
```
Thats it! If you run "kubectl cluster-info" it should return some results.

2. Install an Nginx webserver to run on our K3s cluster Ingress. Copy this Kuberenetes manifest block and paste into your shell. 
```
kubectl create -f - <<EOF
---
kind: Namespace
apiVersion: v1
metadata:
  name: nginxweb
  labels:
    name: nginx
---
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: nginx
   namespace: nginxweb
   labels:
     app: nginx
 spec:
   replicas: 1
   selector:
      matchLabels:
        app: nginx
   template:
     metadata:
       labels:
         app: nginx
 
     spec:
      containers:
         - name: nginx
           image: nginx:alpine
           imagePullPolicy: Always
           ports:
             - containerPort: 80
---
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
  namespace: nginxweb
spec:
  ports:
  - name: nginx-service-port
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
---
# Middleware
# Strip prefix /web
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: mw-admin
  namespace: nginxweb
spec:
  stripPrefix:
    forceSlash: false
    prefixes:
      - /web
---
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: nginx-ingress
  namespace: nginxweb
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/router.middlewares: nginxweb-mw-admin@kubernetescrd
spec:
  ingressClassName: traefik #can be left blank if a default exists
  rules:
  - host: 
    http:
        paths:
          - path: /web
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
EOF
```
Now wait a minute or so and browse to the host **http://ipddress/web**. That's it, you now have a webserver running in your cluster.
You can check the status of your containerized webserver by running: "kubectl get deployments"

3. Bonus Management Interface - Portainer is a nice simple add-on for anyone looking to get a Dashboard View. Use this manifest to create.
```
kubectl create -f - <<EOF
---
kind: Namespace
apiVersion: v1
metadata:
  name: portainer
  labels:
    name: portainer
---
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: portainer
   namespace: portainer
   labels:
     app: portainer
 spec:
   replicas: 1
   selector:
      matchLabels:
        app: portainer
   template:
     metadata:
       labels:
         app: portainer
 
     spec:
      serviceAccountName: portainer-sa-clusteradmin
      containers:
         - name: portainer
           image: portainer/portainer-ce:latest
           imagePullPolicy: Always
           ports:
             - containerPort: 9000
---
kind: Service
apiVersion: v1
metadata:
  name: portainer-service
  namespace: portainer
spec:
  ports:
  - name: portainer-service-port
    port: 9000
    protocol: TCP
    targetPort: 9000
  selector:
    app: portainer
---
# Middleware
# Strip prefix /portainer
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: portainer-admin
  namespace: portainer
spec:
  stripPrefix:
    forceSlash: false
    prefixes:
      - /portainer
---
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: portainer-ingress
  namespace: portainer
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/router.middlewares: portainer-portainer-admin@kubernetescrd
spec:
  ingressClassName: traefik #can be left blank if a default exists
  rules:
  - host: 
    http:
        paths:
          - path: /portainer
            pathType: Prefix
            backend:
              service:
                name: portainer-service
                port:
                  number: 9000
---
# Source: portainer/templates/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  namespace: portainer
  name: portainer-sa-clusteradmin
---
# Source: portainer/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: portainer-sa-clusteradmin
  namespace: portainer
  labels:
    app.kubernetes.io/name: portainer
    app.kubernetes.io/instance: portainer	  			  
EOF
```
Now wait a minute or so and browse to the host **http://ipddress/portainer**. That's it. Login with admin and set your password.
![]({{ 'assets/images/portainer.PNG' | relative_url }})
You now have a fully functional Single Node Kubernetes Cluster. In the [next post](/kubernetes-demystified-and-servicenow-discovery-part2/) we will Connect ServiceNow and Discover it into the CMDB.
[Kubernetes discovery](https://docs.servicenow.com/csh?topicname=kubernetes-discovery.html&version=latest)
