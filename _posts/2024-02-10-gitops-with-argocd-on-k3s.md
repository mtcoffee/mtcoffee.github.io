---
title: GitOps with ArgoCD on K3s
---

# Summary
For my ServiceNow readers, this post will not touch on ServiceNow but if you like Kubernetes and are interested in Continuous Deployment, read on! First of all, What is **GitOps**? Essentially it means managing your infrastructure using **infrastructure as code**, more specifically through Git repositories. This ensures a single source of truth for your infrastructure. **ArgoCD** is an open source project for automating your Kubernetes Deployments. **ArgoCD** will poll the GitRepo associated to your Kubernetes application and automatically deploy the changes to the target Kubernetes cluster. If there is a deployment issue it will auto rollback the change.

# Setting up ArgoCD on K3s
First we need a K3s instance. On your linux server run the K3s installer if you haven't already.
```
sudo curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -
```

Now we can install ArgoCD. It installs into the cluster and does not use PVCs. All data is stored in etcd. Run this script below. It will create the ArgoCD instance and supply the default admin password. 

<script src="https://gist.github.com/mtcoffee/69a2a27e285fcb2bd0b61ab77debde97.js"></script>

Once completed you can login to http://hostIPorName/argocd
![]({{ 'assets/images/ArgoCDWeb.PNG' | relative_url }})
# Adding an App to ArgoCD
Now we  are ready to create an app - [https://argo-cd.readthedocs.io/en/stable/getting_started/](https://argo-cd.readthedocs.io/en/stable/getting_started/)

First we need to install the argocd CLI
```
sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo chmod +x /usr/local/bin/argocd
```

Now we can login
```
ip=`hostname -i` #get IP from host and use generated password collected earlier
argocd login $ip --grpc-web --grpc-web-root-path argocd --insecure --username admin --password $defaultPass
```

Finally we can create our app. You will also be able to see this in the web ArgoCD web interface
```
argocd app create nginxweb --repo https://github.com/mtcoffee/argocd-example-apps.git --path nginxweb --dest-server https://kubernetes.default.svc --dest-namespace default
argocd app sync nginxweb
```
Just like that ArgoCD has automatically deployed our application from the github repo. Open http://hostIPorName/web to see the resulting Nginx deployment.
![]({{ 'assets/images/argocdappdemo.PNG' | relative_url }})

Now if you want to have some fun, clone the example repo and change the HTML in the config map. Within 3 minutes ArgoCD will automatically update the app with your new html.

# Bonus Guestbook app
There are some sample applications in the official Kubernetes repo. Try out the guestbook app.
```
argocd app create guestbook-go --repo https://github.com/kubernetes/examples.git --path guestbook-go --dest-server https://kubernetes.default.svc --dest-namespace default
argocd app sync guestbook-go

##the example repo is broken and needs a Service for redis
kubectl apply -f - << "EOF"
kind: Service
apiVersion: v1
metadata:
  name: redis-slave
  labels:
    app: redis
    role: replica
spec:
  ports:
    - port: 6379
      targetPort: redis-server
  selector:
    app: redis
    role: replica
EOF
```
Now visit your new guestbook!
http://hostIPorName:3000/
