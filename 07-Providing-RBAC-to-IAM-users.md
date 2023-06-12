# Providing RBAC to IAM users
## Add a cluster admin
Steps:

1. create IAM user in AWS console (k8s-cluster-admin)
2. create access key for this user and store it locally
3. add user to configmap aws-auth
4. add user+accesskey to aws credentials file in dedicated section (profile)

first look at the config map

```
kubectl -n kube-system get configmap
```
Here we are working on aws-auth

fetch current configmap before adding our user mapping

```
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
Then apply the changes.
```
kubectl -f apply aws-auth-configmap.yaml -n kube-system
```
To check the user is succfully added  
```
kubectl -n kube-system describe cm aws-auth
```

add user to ~/.aws/credentials by creating a new section

```
[clusteradmin]
aws_access_key_id=.....
aws_secret_access_key=.....
region=us-east-1
output=json
```
To check which user is currently active
``` 
aws sts get-caller-identity
export AWS_PROFILE="clusteradmin"
aws sts get-caller-identity
```


## Add read-only user for particular namespace


### step 1 Create a new namespace called production

```
kubectl create namespace production
```
### step 2 create IAM user and grab access-key

1. create IAM user in AWS console (k8s-production)
2. create access key for this user and store it locally

### step 3 create role
 
role.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: prod-viewer-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "watch", "list"]
  
```
### step 4 create rolebinding

create rolebinding rolebinding

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
  
### step 4 create role and binding

```
kubectl apply -f role.yaml rolebinding.yaml
```

### step 5 Add the user on config map (same steps we use in admin user to add configmap)

```
kubectl -n kube-system get configmap aws-auth -o yaml > aws-auth-configmap.yaml
```

edit the yaml file and add a "mapUsers" section

```
mapUsers: |
    - userarn: arn:aws:iam::xxxxxxxxx:user/k8s-production
      username: k8s-production
      groups:
        - prod-viewer-role
```

Then apply the changes.
```
kubectl -f apply aws-auth-configmap.yaml -n kube-system
```
### step 6 add user to aws-auth configmap

add this user to ~/.aws/credentials file

```
[clusteradmin]
aws_access_key_id=
aws_secret_access_key=
region=us-east-1
output=json

[productionviewer]
aws_access_key_id=
aws_secret_access_key=
region=us-east-1
output=json
```

### step 7 set this user as the active one

```
aws sts get-caller-identity
export AWS_PROFILE="productionviewer"
aws sts get-caller-identity
```

if we access any resourse under kube-system or other namespace with this user will not works. this user only work on namespace production. also on production namespace we dont have access to create pods or other services. because we created this user as ready only privilage 

 
  


