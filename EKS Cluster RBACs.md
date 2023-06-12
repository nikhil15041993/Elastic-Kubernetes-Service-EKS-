##################### Create AWS EKS clsuster #######################################################################

## Pre-requisite links
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## Pre-requisite links
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## Create EKS cluster
eksctl create cluster --name eksrbac --node-type t2.large --nodes 1 --nodes-min 1 --nodes-max 2 --region us-east-1 --zones=us-east-1a,us-east-1b,us-east-1c

## Get EKS Cluster service
eksctl get cluster --name eksrbac --region us-east-1

## Get EKS Pod data.
kubectl get pods --all-namespaces

## Delete EKS cluster
eksctl delete cluster --name eksrbac --region us-east-1

####################################################################################################################

#################### EKS Cluster RBACs #############################################################################
## first create the rbac-test namespace, and then install nginx into it
kubectl create namespace rbac-test

## Deploy nginx pod on clsuster
kubectl create deploy nginx --image=nginx -n rbac-test

## To verify the test pods were properly installed
kubectl get all -n rbac-test

## Create IAM user and create access key
aws iam create-user --user-name rbac-user

aws iam create-access-key --user-name rbac-user

## We will use these set onther context with using above credentials
AWS configure.

## MAP AN IAM USER TO K8S
kubectl apply -f .\aws-auth.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapUsers: |
    - userarn: arn:aws:iam::357171621133:user/rbac-user
      username: rbac-user
```      
## Verify newly created user after login AND it should throw below errors
kubectl get pods -n rbac-test

## Create a role and role binding from Admin access
kubectl apply -f .\rbacuser-role.yaml
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: rbac-test
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["list","get","watch"]
- apiGroups: ["extensions","apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
  ```
kubectl apply -f .\rbacuser-role-binding.yaml
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: rbac-test
subjects:
- kind: User
  name: rbac-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
 ```
## login with Kubernetes user again
## Verify newly created user after login AND it should Not throw any errors
kubectl get pods -n rbac-test

## Verify newly created user after login AND it should throw errors
kubectl get pods -n kube-system

####################################################################################################################
