# Scaling Managed Nodegroups

eksctl scale nodegroup also supports managed nodegroups. The syntax for scaling a managed or unmanaged nodegroup is the same.

```
eksctl scale nodegroup --name=managed-ng-1 --cluster=managed-cluster --nodes=4 --nodes-min=3 --nodes-max=5
```

To list the details about a nodegroup or all of the nodegroups, use:
```
eksctl get nodegroup --cluster=<clusterName> [--name=<nodegroupName>]
```

### maintain nodegroups and add spot instances

add a nodegroup
 
extend the yaml file by adding a second nodegroup, consisting of a mix of ondemand and spot instances
```
eksctl create nodegroup --config-file=eks-course.yaml --include='ng-mixed'
```

eks-course.yaml
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: EKS-course-cluster
  region: us-east-1

nodeGroups:
  - name: ng-1
    instanceType: t2.small
    desiredCapacity: 3
    ssh: # use existing EC2 key
      publicKeyName: eks-course

  - name: ng-mixed
    minSize: 3
    maxSize: 5
    instancesDistribution:
      maxPrice: 0.2
      instanceTypes: ["t2.small", "t3.small"]
      onDemandBaseCapacity: 0
      onDemandPercentageAboveBaseCapacity: 50
    ssh: 
      publicKeyName: eks-course
```      
