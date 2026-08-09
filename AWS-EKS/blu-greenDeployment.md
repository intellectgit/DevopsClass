create ec2 instance - ubuntu - size gp3 volume 

$ sudo apt update 

install aws cli 

create IAM role with access key and secrete key 
select region - ap-south-1

Install terraform command 

with terraform scripts create EKS CLUSTER - once created validate the cluster 

update the cluster with kube-config command 

RBAC
1.create namespace first 
2.create service account for the specific namespace 
3.create role and assign the roles to service account using role binding .
4.create the token for service account. (creating secret)
5. kubectl apply -f secret.yaml -n namespace 
token will display 
6. store this token jenkins credentials as alias k8s.

create 2 EC2 instance
ec2 - jenkins server  & docker & kubectl 
ec2 - sonar server -- using docker 

push image to ECR 

https://www.youtube.com/watch?v=tstBG7RC9as&t=32s





































