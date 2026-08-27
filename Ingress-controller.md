why ingress-controller 
===========================
If there is 4 service are there, to expose this we need 4 different LB are required. 
But If i have ingress controller , it act as single entry for all services with one loadbalancer. 

Create a Ingress resource --> with annotation for alb --> connect AWS alb controller (Ingress Controller) via admission webhook

Aws alb controller <--  permission from IRSA

alb controller validate the kubernetes resources with IRSA (IAM role and service account ) with help of OIDC

alb controller will call aws API call.
It creates 1. target groups 2. alb  3. security group

Pre-Requisite 
=============
kubectl 
eksctl
aws cli

1.create one IAM user , create access key and secrete key
2.create one EKS cluster 

 $aws eks update-kubeconfig --region ap-south-1 --name <clustername>
 above command will update the cluster details to the local. 
 it will create one folder .kube/config
 in this path it will update all cluster details.

 
 




