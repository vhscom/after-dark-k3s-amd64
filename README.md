
# After-Dark on k3s  

## [... a lightweight Kubernetes by Rancher](https://k3s.io/)

## Install K3s on your host  

``$ curl -sfL https://get.k3s.io | sh -``
Check for Ready node, takes maybe 30 seconds  
``$ kubectl get node``  

## Deploy After-Dark  

Download deployment files on your host:  
``$ git clone https://git.habd.as/teowood/after-dark-k3s-amd64.git``  
Apply deployment files:  
``$ cd after-dark-k3s-amd64`` then ``$ kubectl apply -f .``  

## Basic explanation of deployment

This is a multi tier deployment:  

* ``after-dark-k3s-hugo.yaml`` deploys a pod with two containers. First an  init container which downloads the after-dark repository and finally the actual hugo container which kicks in and installs the site. When done it fires hugo in watch mode.  
* ``after-dark-nginx.yaml`` deploys an nginx server that servers our rendered site.
* ``after-dark-service.yaml`` exposes our nginx to a nodeport so we can actually reach the site.

## Operate your k3s after-dark site  

* First retrieve your pod name and store it in a variable for your convenience ``$ AD_POD=$(kubectl get pods -l app=after-dark-hugo -o jsonpath='{.items[0].metadata.name}')``  
* Create a new post  
``$ kubectl exec $AD_POD -- hugo new -k post /posts/new-post.md``  or you can go ahead and copy your stuff under ``/posts`` on your host  
* Apply custom styling  
``$ kubectl cp custom.css $AD_POD:/after-dark/flying-toasters/assets/css/custom.css``  
* Build your draft posts  
``$ kubectl exec  $AD_POD -- hugo  -D -c /posts -d /output``
* Full rebuilt your site e.g after new styles appied ``$ kubectl exec  $AD_POD -- hugo -c /posts -d /output`` Note this will also revert drafts.  
* Edit your posts ``$ kubectl exec -it $AD_POD -- vi /posts/my-post.md`` or directly inside your host's ``/posts``

## Browse to your site  

Locate the nodeport we exposed our nginx for after-dark  
``$ kubectl get svc``
The port we are looking for is the the one next to ``8080:``
For instance in the example below, you would point your browser to `http://your-node-IP:32146`. Ignore the Cluster-IP shown, you want the real IP of your host.

| Name | Type | Cluster-IP | External-IP | Port(s) | Age |
|--------------------|----------|---------------|-------------|----------------|-----|
| after-dark-service | NodePort | 10.43.129.249 | `<none>` | 8080:32146/TCP | 1h |  
