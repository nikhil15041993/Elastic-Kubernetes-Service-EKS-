# Elastic-Kubernetes-Service-EKS-


## Delete EKS Cluster & Node Groups
Step-01: Delete Node Group
We can delete a nodegroup separately using below eksctl delete nodegroup

## List EKS Clusters

```
eksctl get clusters
```

## Capture Node Group name

```
eksctl get nodegroup --cluster=<clusterName>
eksctl get nodegroup --cluster=eksdemo1
```

##  Delete Node Group

```
eksctl delete nodegroup --cluster=<clusterName> --name=<nodegroupName>
eksctl delete nodegroup --cluster=eksdemo1 --name=eksdemo1-ng-public1
```
## Step-02: Delete Cluster

We can delete cluster using eksctl delete cluster

Delete Cluster
  
 ```
eksctl delete cluster <clusterName>
eksctl delete cluster eksdemo1
```




If you get error like this 
==========================
```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
}
```


You get this error because you're getting blocked by RBAC policies. Basically, RBAC policies set to restrict the resources you use and limits a few of your action. 

There are two possibilities, either you haven't created an RBAC or it's somehow restricting the cluster access.

By default, your clusterrolebinding has system:anonymous set which blocks the cluster access.

Execute the following command, it will set a clusterrole as cluster-admin which will give you the required access.

```
kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous
```
