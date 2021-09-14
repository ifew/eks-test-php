# AWS CLI Create Cluster
## Create VPC and Role for EKS
create file cluster-role-trust-policy.json
`aws cloudformation create-stack \
  --region ap-southeast-1 \
  --stack-name test-php-web \
  --template-url https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml \
  --profile few_orm

aws iam create-role \
  --role-name TestPHPWebRole \
  --assume-role-policy-document file://"cluster-role-trust-policy.json" \
  --profile few_orm

aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
  --role-name TestPHPWebRole \
  --profile few_orm`

## Create Cluster

## Linkage my computer to comunicate with EKS Cluster
`rm ~/.kube/config
aws eks update-kubeconfig \
  --region ap-southeast-1 \
  --name test-php-web \
  --profile few_orm`

## Create an IAM OpenID Connect (OIDC) provider

## Create Node
create file cni-role-trust-policy.json
edit config account id
`aws iam create-role \
  --role-name TestPHPWebCNIRole \
  --assume-role-policy-document file://"cni-role-trust-policy.json" \
  --profile few_orm
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
  --role-name TestPHPWebCNIRole \
  --profile few_orm
aws eks update-addon \
  --cluster-name test-php-web \
  --addon-name vpc-cni \
  --service-account-role-arn arn:aws:iam::[MY-ARN]:role/TestPHPWebCNIRole \
  --profile few_orm \
  --region ap-southeast-1
create file node-role-trust-policy.json
aws iam create-role \
  --role-name TestPHPWebNodeRole \
  --assume-role-policy-document file://"node-role-trust-policy.json" \
  --profile few_orm
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
  --role-name TestPHPWebNodeRole \
  --profile few_orm
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
  --role-name TestPHPWebNodeRole \
  --profile few_orm`
go to cluster > Configuration > Compute > Add Node Group
- create node
- choose instant type
- allow ssh to access node


## Update custer to public
`aws eks update-cluster-config --region ap-southeast-1 --name test-php-web --resources-vpc-config endpointPublicAccess=true,publicAccessCidrs="0.0.0.0/0",endpointPrivateAccess=true --profile few_orm`
## Check custer are public
`aws eks describe-update --region ap-southeast-1 --name test-php-web --update-id a6ff0b7a-8461-461d-8bfb-1dfbd91e1c59 --profile few_orm`

## Check all pod in all namespace
kubectl get pods --all-namespaces -o wide``

## Create namespace
`kubectl create namespace test-php-web`

## Deploy pod php and nginx
`kubectl apply -f deploy-php-nginx.yml`

## Check pod in namespace "test-php-web"
`kubectl get all -n test-php-web`

## Check service in namespace
`kubectl -n test-php-web describe service test-php-web-service`
## Check deployment in namespace
`kubectl -n test-php-web describe deployment test-php-web-deployment`
## Check pod in namespace
`kubectl -n test-php-web describe pod test-php-web-deployment-96ddcf945-mqp6b
kubectl -n test-php-web describe pod test-php-web-deployment-96ddcf945-nfkkh
kubectl -n test-php-web describe pod test-php-web-deployment-96ddcf945-wqfqg`

## Execute shell into container
`kubectl exec -it test-php-web-deployment-96ddcf945-mqp6b -n test-php-web -- /bin/bash`

## Delete service, deployment, pod, namespace
`kubectl delete namespace test-php-web`





# EKSCTL Create Cluster
## create cluster
`eksctl create cluster --name=myweb --nodes=2 --node-type=t2.micro --region ap-southeast-1 --with-oidc --ssh-access --ssh-public-key ifew_orm_macbook --managed --profile few_orm`

## Check node
`kubectl get nodes`

## cerate namespace
`kubectl create namespace myweb`

## Create secret
`kubectl apply -f eks-myweb-secret.yml`

## Create deploy pod php and nginx
`kubectl apply -f eks-myweb-deploy.yml`

## Check pod and everything in namespace
`kubectl get all -n myweb`

## Create service
`kubectl apply -f eks-myweb-service.yml`

## Check service in namespace
`kubectl get services -n myweb`

## Access to lead balance url

## Update content in Pod
Edit new image version in eks-myweb-deploy.yml
`kubectl apply -f eks-myweb-deploy.yml`

# Update secret
apply secret again and restart pod for get new environment
`kubectl apply -f eks-myweb-secret.yml`
`kubectl -n myweb rollout restart deployment myweb-deployment`