# Deploy Kubernetes (EKS) with Argo CD using terraform

## Requirements
- Configured AWS profile with permissions to create the required resources
- Terraform (Code was tested on version 1.5.2)
- AWS, Helm, and Kubernetes terraform providers
- Key pair in EC2 service called `ssh`

## Resources created
- VPC with public and private subnets
- IAM roles and instance profile
- NAT and Internet Gateways 
- EKS cluster
- EC2 instances using auto-scaling groups for Kubernetes worker nodes
- Namespace in Kubernetes for Argo CD
- Argo CD deployment in Kubernetes

# Deploy Kubernetes (AWS EKS) with Argo CD using terraform

##### Deploy Repo:

[albert-achterkamp/terraform-eks-argocd: Deploy AWS Kubernetes (EKS) with Argo CD using terraform (github.com)](https://github.com/albert-achterkamp/terraform-eks-argocd/tree/main)

##### Before3 deployment

1. Use the correct version of TF
    ```bash
    albert.achterkamp@linux ~/aws/gh_terraform-eks-argocd (eks_argo_1)$ terraform version
    Terraform v1.5.2
    on linux_amd64
    + provider registry.terraform.io/hashicorp/aws v5.5.0
    + provider registry.terraform.io/hashicorp/helm v2.10.1
    + provider registry.terraform.io/hashicorp/kubernetes v2.21.1

    Your version of Terraform is out of date! The latest version
    is 1.5.3. You can update by downloading from https://www.terraform.io/downloads.html

    ```
2. Before deployment check the region of deployment and also check/create a SSH keypair to be used (should be present in AWs Console before deployment)

##### Test deployment on CLI (linux)

```bash
terraform plan  -out=eks-157-argocdB.tfplan
terraform apply "eks-157-argocdB.tfplan"
terraform version
```

##### Check in AWS Console if the Cluster is up and running


##### Configure and test kubectl to access the API Server

```bash
Last login: Sat Jul 15 16:01:05 CEST 2023 on pts/3
albert.achterkamp@linux ~ $ aws eks update-kubeconfig --region eu-central-1 --name test
Added new context arn:aws:eks:eu-central-1:453578755218:cluster/test to /home/albert.achterkamp/.kube/config

albert.achterkamp@linux ~ $ k get po -A
NAMESPACE     NAME                                                READY   STATUS    RESTARTS   AGE
argocd        argocd-application-controller-0                     1/1     Running   0          85s
argocd        argocd-applicationset-controller-674668f95c-9prft   1/1     Running   0          85s
argocd        argocd-dex-server-6c54598f4c-r7tbl                  1/1     Running   0          85s
argocd        argocd-notifications-controller-5689b78499-rzjld    1/1     Running   0          85s
argocd        argocd-redis-56c599fd8c-gft8l                       1/1     Running   0          85s
argocd        argocd-repo-server-8f8b99479-csdk6                  1/1     Running   0          85s
argocd        argocd-server-696575b6b-9fh97                       1/1     Running   0          85s
kube-system   aws-node-7b2bm                                      1/1     Running   0          52s
kube-system   coredns-7bc655f56f-7fw96                            1/1     Running   0          13m
kube-system   coredns-7bc655f56f-hhnxk                            1/1     Running   0          13m
kube-system   kube-proxy-vhkjf                                    1/1     Running   0          52s
albert.achterkamp@linux ~ $ k get no
NAME                                          STATUS   ROLES    AGE   VERSION
ip-10-0-3-106.eu-central-1.compute.internal   Ready    <none>   76s   v1.27.1-eks-2f008fe
albert.achterkamp@linux ~ $ argoPass=$(kubectl -n argocd get secret argocd-initial-admin-secret \
>     -o jsonpath="{.data.password}" | base64 -d)

albert.achterkamp@linux ~ $ echo $argoPass
XXXXXXXXXXXXXXXXXXX
albert.achterkamp@linux ~ $ kubectl get svc -n argocd
NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
argocd-applicationset-controller   ClusterIP   172.20.132.161   <none>        7000/TCP            18m
argocd-dex-server                  ClusterIP   172.20.75.26     <none>        5556/TCP,5557/TCP   18m
argocd-redis                       ClusterIP   172.20.252.80    <none>        6379/TCP            18m
argocd-repo-server                 ClusterIP   172.20.106.209   <none>        8081/TCP            18m
argocd-server                      ClusterIP   172.20.173.51    <none>        80/TCP,443/TCP      18m
```
