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

```bash
  mapUsers: |
    - userarn: arn:aws:iam::xxxxxxxxx:user/k8s-cluster-admin
      username: k8s-cluster-admin
      groups:
        - system:masters
```

add user to ~/.aws/credentials by creating a new section

```bash
[clusteradmin]
aws_access_key_id=.....
aws_secret_access_key=.....
region=us-east-1
output=json
```

check which user is currently active

```bash
aws sts get-caller-identity
export AWS_PROFILE="clusteradmin"
aws sts get-caller-identity
```
