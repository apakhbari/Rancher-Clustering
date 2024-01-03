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

# Tips & Tricks
- RAncher can be run on anything, K8s cluster, dedicated VM, on Prem machine, Outside of k8s infra.
- As long as Rancher can reach IP Addresses of machines that are k8s nodes, You are good to go.
- Rancher is source of truths for k8s cluster, so make sure its HA + Its Data is persistent

# Requirements
- Hardware requirementes: https://docs.rke2.io/install/requirements
- Supported Kubernetes Platforms for Rancher Manager: https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-7-9/

# Implementation
- Note that some tools, such as kubectl, are installed by default into ```/var/lib/rancher/rke2/bin```
- Leverage the KUBECONFIG environment variable:
```
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
kubectl get pods --all-namespaces
helm ls --all-namespaces
```
- Accessing the Cluster from Outside with kubectl: Copy ```/etc/rancher/rke2/rke2.yaml``` on your machine located outside the cluster as ```~/.kube/config``` Then replace 127.0.0.1 with the IP or hostname of your RKE2 server. kubectl can now manage your RKE2 cluster.

# Creating Snapshots
- ```/var/lib/rancher/rke2/server/db/snapshots``` --> The snapshot directory defaults
- In RKE2, snapshots are stored on each etcd node. If you have multiple etcd or etcd + control-plane nodes, you will have multiple copies of local etcd snapshots.
- You can take a snapshot manually while RKE2 is running with the etcd-snapshot subcommand. For example: ```$ rke2 etcd-snapshot save --name pre-upgrade-snapshot```
- By default, Rancher snapshots every 12 hours
- You can list local snapshots with the ```$ etcd-snapshot ls```




# acknowledgment
## Contributors

APA 🖖🏻

## Links
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
