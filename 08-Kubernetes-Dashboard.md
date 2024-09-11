## Kubernetes Dashboard

###  To install k8 dashboard use this command:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```
Verification

```
kubectl get all -n kubernetes-dashboard
```

### create an admin user account

```
kubectl apply -f admin-service-account.yaml
```

admin-service-account.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-course-admin
  namespace: kube-system

---
  
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: eks-course-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-course-admin
  namespace: kube-system
```
  
### access the dashboard

* get a security token

n Kubernetes 1.24, you need to manually create the Secret; the token key in the data field will be automatically set for you.
```
apiVersion: v1
kind: Secret
metadata:
  name: sa1-token
  annotations:
    kubernetes.io/service-account.name: service-account-name
type: kubernetes.io/service-account-token
```
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```

record the output of ```token:```

* start kube proxy via ```kubectl proxy```

* open browser at `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login`

and choose _token_ as login method, where you have to provide the token you recorded two steps before



Install the Metrics Server using the following commands:
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Verify that the Metrics Server is running:
```
kubectl get deployment metrics-server -n kube-system
```
