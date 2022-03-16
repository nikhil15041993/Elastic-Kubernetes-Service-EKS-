## Create a yml file 

eg: my-first-eksctl.yml

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
      
```

## Use existing VPC: other custom configuration

```

apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-test
  region: us-west-2

vpc:
  id: "vpc-11111"
  subnets:
    private:
      us-west-2a:
          id: "subnet-0ff156e0c4a6d300c"
      us-west-2c:
          id: "subnet-0426fb4a607393184"
    public:
      us-west-2a:
          id: "subnet-0153e560b3129a696"
      us-west-2c:
          id: "subnet-009fa0199ec203c37"

nodeGroups:
  - name: ng-1
    instanceType: t2.small
    desiredCapacity: 3
    ssh: # use existing EC2 key
      publicKeyName: eks-course
  
  ```

## Create Cluser

```
eksctl create cluster -f my-first-eksctl.yml

```

## Scaling Managed Nodegroups
eksctl scale nodegroup also supports managed nodegroups. The syntax for scaling a managed or unmanaged nodegroup is the same.

```
eksctl scale nodegroup --name=managed-ng-1 --cluster=managed-cluster --nodes=4 --nodes-min=3 --nodes-max=5
```

