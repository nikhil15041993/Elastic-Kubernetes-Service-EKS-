## Add a cluster admin

Steps:

1. create IAM user in AWS console (_k8s-cluster-admin_)
2. create access key for this user and store it locally
3. add user to configmap aws-auth
4. add user+accesskey to aws credentials file in dedicated section (profile)

fetch current configmap before adding our user mapping

```bash
kubectl -n kube-system get configmap aws-auth -o yaml > aws-auth-configmap.yaml
```

edit the yaml file and add a "mapUsers" section

```
  mapUsers: |
    - userarn: arn:aws:iam::xxxxxxxxx:user/k8s-cluster-admin
      username: k8s-cluster-admin
      groups:
        - system:masters
```

add user to ~/.aws/credentials by creating a new section

```
[clusteradmin]
aws_access_key_id=.....
aws_secret_access_key=.....
region=us-east-1
output=json
```

check which user is currently active

```
aws sts get-caller-identity

export AWS_PROFILE="clusteradmin"

aws sts get-caller-identity
```
Check whether is user is successfully  configured by using 

### listing the resource 
```
kubectl get nodes

kubectl get pods..

etc....

```
### configuring the resource


```
kubectl run nginx --image=nginx --restart=Never
```


## Add Readonly User 

### Add read-only user for particular namespace
 
 Create a Namespace

```
kubectl create namespace production
```

1. create IAM user in AWS console (_k8s-cluster-admin_)
2. create access key for this user and store it locally

### create role yml file and paste the following code

role.yml
========
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: production
  name: prod-viewer-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]  # can be further limited, e.g. ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch"] 
```

save it

### create  role binding yml file 

rolebinding.yaml
================

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: prod-viewer-binding
  namespace: production
subjects:
- kind: User
  name: prod-viewer
  apiGroup: ""
roleRef:
  kind: Role
  name: prod-viewer-role
  apiGroup: ""
```

create role and binding

```
kubectl apply -f role.yaml rolebinding.yaml
```

### add user to aws-auth configmap

```
kubectl -n kube-system get configmap aws-auth -o yaml > aws-auth-configmap.yaml
```
move on to the mapUser section and add the user details

```
  mapUsers: |
    - userarn: arn:aws:iam::xxxxxxxxx:user/k8s-cluster-admin
      username: k8s-cluster-admin
      groups:
        - system:masters
        
    - userarn: arn:aws:iam::xxxxxxxxx:user/k8s-cluster-admin
      username: <username>
      groups:
        - <iam role>
        
```
    
    Apply the aws-auth-configmap.yaml file
    
```
    kubectl apply -f aws-auth-configmap.yaml
```
    
   add user to ~/.aws/credentials by creating a new section

```
[production]
aws_access_key_id=.....
aws_secret_access_key=.....
region=us-east-1
output=json
```     

check which user is currently active

```
  aws sts get-caller-identity

  export AWS_PROFILE="production"

  aws sts get-caller-identity
```
Check whether is user is successfully  configured by using 

```
 kubectl get nodes
```

```
 kubectl -n kube-system get pods
```
Change the user to cluster admin ```export AWS_PROFILE="production"``` now we can crete and list the resoures


        
