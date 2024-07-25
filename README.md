# Rancher-Clustering
```
  _____                  _                       _____ _           _            _             
 |  __ \                | |                     / ____| |         | |          (_)            
 | |__) |__ _ _ __   ___| |__   ___ _ __ ______| |    | |_   _ ___| |_ ___ _ __ _ _ __   __ _ 
 |  _  // _` | '_ \ / __| '_ \ / _ \ '__|______| |    | | | | / __| __/ _ \ '__| | '_ \ / _` |
 | | \ \ (_| | | | | (__| | | |  __/ |         | |____| | |_| \__ \ ||  __/ |  | | | | | (_| |
 |_|  \_\__,_|_| |_|\___|_| |_|\___|_|          \_____|_|\__,_|___/\__\___|_|  |_|_| |_|\__, |
                                                                                         __/ |
                                                                                        |___/ 
```


## Tips & Tricks
- RAncher can be run on anything, K8s cluster, dedicated VM, on Prem machine, Outside of k8s infra.
- As long as Rancher can reach IP Addresses of machines that are k8s nodes, You are good to go.
- Rancher is source of truths for k8s cluster, so make sure its HA + Its Data is persistent
- By default, certificates in RKE2 expire in 12 months. If the certificates are expired or have fewer than 90 days remaining before they expire, the certificates are rotated when RKE2 is restarted.
- 

## Requirements
- Hardware requirementes: https://docs.rke2.io/install/requirements
- Supported Kubernetes Platforms for Rancher Manager: https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-7-9/

## Inbound Network Rules
|Protocol	|Port|	Source|	Destination|	Description|
| --- | --- | --- | --- | --- |
|TCP|	9345|	RKE2 agent nodes |	RKE2 server nodes |	RKE2 supervisor API
TCP|	6443	RKE2 agent nodes	| RKE2 server nodes	|Kubernetes API
UDP|	8472	|All RKE2 nodes	| All RKE2 nodes |	Required only for Flannel VXLAN
TCP|	10250	|All RKE2 nodes	|All RKE2 nodes |	kubelet metrics
TCP	|2379 |	RKE2 server nodes	|RKE2 server nodes |	etcd client port
TCP	|2380	|RKE2 server nodes |	RKE2 server nodes	| etcd peer port
TCP	|2381	|RKE2 server nodes |	RKE2 server nodes |	etcd metrics port
TCP	|30000-32767|	All RKE2 nodes |	All RKE2 nodes |	NodePort port range
UDP	|8472|	All RKE2 nodes |	All RKE2 nodes |	Cilium CNI VXLAN
TCP	|4240|	All RKE2 nodes |	All RKE2 nodes |	Cilium CNI health checks
ICMP|	8/0|	All RKE2 nodes	| All RKE2 nodes |	Cilium CNI health checks
TCP	|179|	All RKE2 nodes |	All RKE2 nodes |	Calico CNI with BGP
UDP	|4789|	All RKE2 nodes |	All RKE2 nodes |	Calico CNI with VXLAN
TCP	|5473|	All RKE2 nodes |	All RKE2 nodes |	Calico CNI with Typha
TCP	|9098|	All RKE2 nodes |	All RKE2 nodes |	Calico Typha health checks
TCP	|9099|	All RKE2 nodes |	All RKE2 nodes |	Calico health checks
TCP	|5473	|All RKE2 nodes |	All RKE2 nodes |	Calico CNI with Typha
UDP	|8472	|All RKE2 nodes |	All RKE2 nodes |	Canal CNI with VXLAN
TCP	|9099	|All RKE2 nodes |	All RKE2 nodes |	Canal CNI health checks
UDP	|51820|	All RKE2 nodes |	All RKE2 nodes |	Canal CNI with WireGuard IPv4
UDP	|51821|	All RKE2 nodes |	All RKE2 nodes |	Canal CNI with WireGuard IPv6/dual-stack

## Which OS to use
- Suse bought Rancher in 2023
- What is your OS of choice to run k8s ON PREM (Poll): https://www.reddit.com/r/kubernetes/comments/11kw5ss/poll_what_is_your_os_of_choice_to_run_k8s_on_prem/
- Ubuntu: flexibility for additional addons and tweaks for some of my runtime and storage solutions. While I don't love Ubuntu, and hate snap, I can disable all of the bloat and still get the benefits of a popular and well-maintained distro that I can be sure stays mostly up-to-date, and get all the host flexibility I need.
- The standard OS for Kubernetes is Ubuntu as it is used by CNCF in CKA/CKAD courses

## Implementation
- Note that some tools, such as kubectl, are installed by default into ```/var/lib/rancher/rke2/bin```
- Leverage the KUBECONFIG environment variable:
```
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
kubectl get pods --all-namespaces
helm ls --all-namespaces
```
- Accessing the Cluster from Outside with kubectl: Copy ```/etc/rancher/rke2/rke2.yaml``` on your machine located outside the cluster as ```~/.kube/config``` Then replace 127.0.0.1 with the IP or hostname of your RKE2 server. kubectl can now manage your RKE2 cluster.

# Starting the Server with the Installation Script
The installation script provides units for systemd, but does not enable or start the service by default.

When running with systemd, logs will be created in /var/log/syslog and viewed using journalctl -u rke2-server or journalctl -u rke2-agent.

An example of installing with the install script:
```
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server
systemctl start rke2-server
```

## Schema
Cluster consists of:

| Machine | OS | IP + MAC Address | Resources |
| --- | --- | --- | --- |
| k8s-rancher (WEB) | Ubuntu 22.04.03 LTS | 192.168.100.100 | CPU: 8 - Memory: 16 - HDD: 80 GB - Network: k8s |
| --- | --- | --- | --- |
| k8s-control-plane-1 | Ubuntu 22.04.03 LTS | 192.168.100.101 | CPU: 4 - Memory: 8 - HDD: 50 GB - Network: k8s |
| k8s-control-plane-2 | Ubuntu 22.04.03 LTS | 192.168.100.102 | CPU: 4 - Memory: 8 - HDD: 50 GB- Network: k8s |
| k8s-control-plane-3 | Ubuntu 22.04.03 LTS | 192.168.100.103 | CPU: 4 - Memory: 8 - HDD: 50 GB - Network: k8s |
| --- | --- | --- | --- |
| k8s-worker-1 | Ubuntu 22.04.03 LTS | 192.168.100.111 | CPU: 4 - Memory: 8 - HDD: 50 GB - Network: k8s |
| k8s-worker-2 | Ubuntu 22.04.03 LTS | 192.168.100.112 | CPU: 4 - Memory: 8 - HDD: 50 GB - Network: k8s |
| --- | --- | --- | --- |
| k8s-storage-1 | Ubuntu 22.04.03 LTS | 192.168.100.121 | CPU: 4 - Memory: 6144 MB - HDD: 120 GB - Network: k8s |
| k8s-storage-2 | Ubuntu 22.04.03 LTS | 192.168.100.122 | CPU: 4 - Memory: 6144 MB - HDD: 120 GB - Network: k8s |

Other Cluster
| Machine | OS | IP + MAC Address | Resources |
| --- | --- | --- | --- |
| k8s-rancher (WEB) | Ubuntu 22.04.03 LTS | 192.168.2.200 | CPU: 8 - Memory: 16 - HDD: 50 GB - Network: VM Network|
| --- | --- | --- | --- |
| k8s-control-plane-1 | Ubuntu 22.04.03 LTS | 192.168.2.201 | CPU: 4 - Memory: 12 - HDD: 40 GB - Network: VM Network|
| k8s-control-plane-2 | Ubuntu 22.04.03 LTS | 192.168.2.202 | CPU: 8 - Memory: 10 - HDD: 40 GB- Network: VM Network|
| --- | --- | --- | --- |
| k8s-worker-1 | Ubuntu 22.04.03 LTS | 192.168.2.211 | CPU: 4 - Memory: 12 - HDD: 80 GB - Network: VM Network|
| k8s-worker-2 | Ubuntu 22.04.03 LTS | 192.168.2.212 | CPU: 6 - Memory: 10 - HDD: 80 GB - Network: VM Network|
| k8s-worker-3 | Ubuntu 22.04.03 LTS | 192.168.2.213 | CPU: 8 - Memory: 16 - HDD: 80 GB - Network: VM Network|


## Running Rancher in Docker
IMPORTANT: Stop UFW Service / Give appropiate permissions to it before starting Rancher
```
$ docker run -d --name rancher-server  -v ${PWD}/volume:/var/lib/rancher --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher
```

For getting Login Password with Admin User:
```
docker logs  container-id  2>&1 | grep "Bootstrap Password:"
```


## Creating Snapshots
- ```/var/lib/rancher/rke2/server/db/snapshots``` --> The snapshot directory defaults
- In RKE2, snapshots are stored on each etcd node. If you have multiple etcd or etcd + control-plane nodes, you will have multiple copies of local etcd snapshots.
- You can take a snapshot manually while RKE2 is running with the etcd-snapshot subcommand. For example: ```$ rke2 etcd-snapshot save --name pre-upgrade-snapshot```
- By default, Rancher snapshots every 12 hours
- You can list local snapshots with the ```$ etcd-snapshot ls```

## Automatically Deploying Manifests and Helm Charts
- Any Kubernetes manifests found in ```/var/lib/rancher/rke2/server/manifests``` will automatically be deployed to RKE2 in a manner similar to ```$ kubectl apply```
- Manifests deployed in this manner are managed as AddOn custom resources, and can be viewed by running ```$ kubectl get addon -A```
- You will find AddOns for packaged components such as CoreDNS, Nginx-Ingress, etc. AddOns are created automatically by the deploy controller, and are named based on their filename in the manifests directory.

## Longhorn
### Set it up
1. Create a project called storage
2. Install longhorn from: Apps > Lunch > find longhorn
3. Keep options on its default. Keep Default storage class true, whether or not expose app using Layer 7 LB should be false

## Tips & Tricks
- You can use existing nodes and reclaim their free storage + It is possible to create nodes and dedicate them just to storage.
- volumes are mounted inside Nodes in /var/lib/longhorn
- It is best practice to Add tags to dedicated storage
- Set up backup outside of your cluster, recommendation: MINIO(s3 storage) - NFS

## Dedicated Storage Nodes
approach 1: 
- Longhorn UI > Nodes > Edit > Node Scheduling Disable (for not dedicated nodes)
approach 2:
- Tainting my storage nodes using this command
```
kubectl taint nodes luna-01 luna-02 luna-03 luna-04 CriticalAddonsOnly=true:NoExecute
kubectl taint nodes luna-01 luna-02 luna-03 luna-04 StorageOnly=true:NoExecute
```
Then applying that toleration to Longhorn in settings
```
StorageOnly=true:NoExecute;CriticalAddonsOnly=true:NoExecute
```
This ensures that the storage nodes won’t take on any general workloads and still allow Lonhorn to use these as storage.

## ISTIO
- Install Istio with the Istio CNI plugin: https://istio.io/latest/docs/setup/additional-setup/cni/
- https://ranchermanager.docs.rancher.com/pages-for-subheaders/istio
- https://ranchermanager.docs.rancher.com/pages-for-subheaders/istio-setup-guide
- https://ranchermanager.docs.rancher.com/how-to-guides/advanced-user-guides/istio-setup-guide/enable-istio-in-cluster

## Kubernetes Gateway API CRDs
Note that the Kubernetes Gateway API CRDs do not come installed by default on most Kubernetes clusters, so make sure they are installed before using the Gateway API:
```
$ kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
  { kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.0.0" | kubectl apply -f -; }
```

## Candidate OSs
- https://vmware.github.io/photon/
- https://github.com/kairos-io/kairos
- https://www.talos.dev/v1.3/introduction/getting-started/
- https://en.opensuse.org/Portal:MicroOS


# acknowledgment
## Contributors

APA 🖖🏻

## Links
- Kubernetes Master Class: GitOps, Rancher and ArgoCD with Codefresh (Rancher by suse channel) : https://youtu.be/3zrhC3iALRk?si=6HmEiiMuHvqj2yH-
---
- Architecture Recommendations:
 https://ranchermanager.docs.rancher.com/reference-guides/rancher-manager-architecture/architecture-recommendations#environment-for-kubernetes-installations
- Setting up a High-availability RKE2 Kubernetes Cluster for Rancher: https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/kubernetes-cluster-setup/rke2-for-rancher
---
- RKE2 High Availability Pre-Deployment & Installation Guide: https://docs.expertflow.com/cx/4.3/rke2-high-availability-pre-deployment-installati-1
---
- Teraform:
- rancher2_cluster_v2 Resource: https://registry.terraform.io/providers/rancher/rancher2/latest/docs/resources/cluster_v2
- terraform-vsphere-rke2: https://gricad-gitlab.univ-grenoble-alpes.fr/kubernetes-alpes/terraform-vsphere-rke2
---
- Introduction to Rancher: On-prem Kubernetes: https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/kubernetes/rancher
- Title: Setting Up an On-Premises Rancher Cluster with k3s, Helm and Hyper-V Manager(2023): https://medium.com/@saadullahkhanwarsi/title-setting-up-an-on-premise-k3s-cluster-with-rancher-helm-and-hyper-v-manager-cc888edb178c
- Setup local kubernetes multi-node cluster with Rancher Server (2019): https://kwonghung-yip.medium.com/setup-local-kubernetes-multi-node-cluster-with-rancher-server-fdb7a0669b5c
- Setup Kubernetes cluster using Rancher 2.x (2019): https://medium.com/@torkashvand/https-medium-com-setup-kubernetes-cluster-using-rancher-2-x-a70f004de0f5

---
- Simple RKE2, Longhorn, and Rancher Install: https://ranchergovernment.com/blog/article-simple-rke2-longhorn-and-rancher-install
---
- Removing Kubernetes Components from Nodes: https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/manage-clusters/clean-cluster-nodes

```                                                                                                       
  aaaaaaaaaaaaa  ppppp   ppppppppp     aaaaaaaaaaaaa   
  a::::::::::::a p::::ppp:::::::::p    a::::::::::::a  
  aaaaaaaaa:::::ap:::::::::::::::::p   aaaaaaaaa:::::a 
           a::::app::::::ppppp::::::p           a::::a 
    aaaaaaa:::::a p:::::p     p:::::p    aaaaaaa:::::a 
  aa::::::::::::a p:::::p     p:::::p  aa::::::::::::a 
 a::::aaaa::::::a p:::::p     p:::::p a::::aaaa::::::a 
a::::a    a:::::a p:::::p    p::::::pa::::a    a:::::a 
a::::a    a:::::a p:::::ppppp:::::::pa::::a    a:::::a 
a:::::aaaa::::::a p::::::::::::::::p a:::::aaaa::::::a 
 a::::::::::aa:::ap::::::::::::::pp   a::::::::::aa:::a
  aaaaaaaaaa  aaaap::::::pppppppp      aaaaaaaaaa  aaaa
                  p:::::p                              
                  p:::::p                              
                 p:::::::p                             
                 p:::::::p                             
                 p:::::::p                             
                 ppppppppp                             
                                                       
```
