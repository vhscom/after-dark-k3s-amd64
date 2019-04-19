
# After Dark K3s

> Run [After Dark](https://after-dark.habd.as) in a lightweight [K3s](https://k3s.io/) cluster providing the power of [Kubernetes](https://kubernetes.io/) container orchestration for a fraction of the resources.

## Installation

In two steps. First setup k3s, then deploy After Dark to your cluster.

### Step 1: Download and install k3s via script

Download and install k3s [via script](https://get.k3s.io) as suggested on the [K3s website](https://k3s.io/):

```sh
curl -sfL https://get.k3s.io | sh -
# Check for Ready node, takes maybe 30 seconds
kubectl get node
```

Run on host machine you wish to use to manage your cluster. This could be locally for development, on an [RPi](https://raspberrypi.org/) or even a VPS with 512 MB RAM. The K3s installation script gives you access to the [`kubectl` command](https://kubernetes.io/docs/reference/kubectl/overview/) giving you the ability to perform container orchestration straight-away.

### Step 2: Deploy After Dark to your k3s cluster

Pull down [Deployment](https://kubernetes.io/docs/concepts/) files to host machine:

```sh
git clone https://git.habd.as/teowood/after-dark-k3s-amd64
```

Use [`kubectl`](https://kubernetes.io/docs/reference/kubectl/overview/) to apply the deployment files and get things kicked off:

```sh
cd after-dark-k3s-amd64 && kubectl apply -f .
```

Your cluster is now running and you have a containerized version of [After Dark](https://after-dark.habd.as) running inside your cluster and accessible via browser.

Continue reading for an overview of the deployment and basic usage.

## Overview of deployment

Multi-tier deployment consisting of:  

* ``after-dark-k3s-hugo.yaml`` deploys a pod using two containers. First, an ephemeral initialization tasked with downloading After Dark from [source repo](https://git.habd.as/comfusion/after-dark) and, finally, the actual hugo container which kicks in and installs the site. When done it runs [`hugo server`](https://gohugo.io/commands/hugo_server/) in _watch mode_ so [Hugo](https://gohugo.io/) rebuilds After Dark site as files change.
* ``after-dark-nginx.yaml`` deploys an [nginx](https://www.nginx.com/) web server that serves the content of the rendered site. As file changes occur Hugo will rebuild the After Dark site and nginx will pick up the changes.
* ``after-dark-service.yaml`` exposes nginx to a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) so we can actually reach the site from a browser.

## Usage

### Operating your k3s After Dark site  

* First retrieve your pod name and store it in a variable for your convenience ``$ AD_POD=$(kubectl get pods -l app=after-dark-hugo -o jsonpath='{.items[0].metadata.name}')``  
* Create a new post  
``$ kubectl exec $AD_POD -- hugo new post/new-post.md -c /my-content -d /output``  or you can go ahead and copy your stuff under ``/my-content`` on your host  
* Apply custom styling  
``$ kubectl cp custom.css $AD_POD:/after-dark/flying-toasters/assets/css/custom.css``  
* Build your draft posts  
``$ kubectl exec  $AD_POD -- hugo  -D -c /my-content -d /output``
* Full rebuilt your site e.g after new styles appied ``$ kubectl exec  $AD_POD -- hugo -c /my-content -d /output`` Note this will also revert drafts.  
* Edit your posts ``$ kubectl exec -it $AD_POD -- vi /my-content/my-post.md`` or directly inside your host's ``/my-content``

### Browsing to your site  

To view your site run `kubectl get svc` to locate the NodePort exposed to nginx. You should see output like:

| Name | Type | Cluster-IP | External-IP | Port(s) | Age |
|--------------------|----------|---------------|-------------|----------------|-----|
| after-dark-service | NodePort | 10.43.129.249 | `<none>` | 8080:32146/TCP | 1h |

Grab the port number next to `8080:` and use it to browse to your site using the node IP or, locally on the host, using the loopback e.g. `http://localhost:32146`. (Note: Ignore the <samp>Cluster-IP</samp> and the <samp>External-IP</samp>, you want the real IP of your host.)
