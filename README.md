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
