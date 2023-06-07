
## create namespace
to separate your cluster resources logically it is *best practice* to use _namespaces_. You can separate either by project, customer, team, environment,...
The benefit you gain is by getting control over resource qoutas, access control etc

```
kubectl create namespace ns-eks-course
```
check
```
kubectl get namespace
```
## persistent volume

in AWS EKS a persistent volume (PV) is implemented via a EBS volume, which has to be declared as a _storage class_ first.
A stateful app can then request a volume, by specifying a _persistent volume claim_ (PVC) and mount it in its corresponding pod.

## **!!Only execute step 1. if you are running Kubernetes version 1.10 in EKS...which is outdated. Since Jan 2019 EKS is using v1.11 and the gp2 storageclass is created automatically there...by default!!**

gp2-storage-class.yaml

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
mountOptions:
  - debug
```


>    1. define a storage class
>    ```
>    kubectl apply -f gp2-storage-class.yaml --namespace=ns-eks-course
>    ```
>    If you receive the following error, then there is already gp2 storageclass available and you can skip above command:  
>
>>    *The StorageClass "gp2" is invalid:*  
>>    *parameters: Forbidden: updates to parameters are forbidden.*  
>>    *reclaimPolicy: Forbidden: updates to reclaimPolicy are forbidden.*  
>
>    ...and set this storage class as *default* !
>    check current situation:
>    ```
>    kubectl get storageclasses --namespace=ns-eks-course
>    ```
>    set default:
>    ```
>    kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}' --namespace=ns-eks-course
>    ```
>    check again:
>    ```
>    kubectl get storageclasses --namespace=ns-eks-course
>    ```

2. define a persistent volume claim

pvcs.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
----
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```  

```
kubectl apply -f pvcs.yaml --namespace=ns-eks-course
```
and check:
```
kubectl get pvc --namespace=ns-eks-course
```

## create secret which stores mysql pw, to be injected as env var into container
```
kubectl create secret generic mysql-pass --from-literal=password=eks-course-mysql-pw --namespace=ns-eks-course
```

check:
```
kubectl get secrets --namespace=ns-eks-course
```

# deploy mysql
## create secret which stores mysql pw, to be injected as env var into container
```
kubectl create secret generic mysql-pass --from-literal=password=eks-course-mysql-pw --namespace=ns-eks-course
```
check:
```
kubectl get secrets --namespace=ns-eks-course
```

## launch mysql deployment

deploy-mysql.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
 ```
 
```
kubectl apply -f deploy-mysql.yaml --namespace=ns-eks-course
```
## checks
* persistent volumes
* persistent volume claims
* pods
```
kubectl get pv --namespace=ns-eks-course
kubectl get pvc --namespace=ns-eks-course
kubectl get pods -o wide --namespace=ns-eks-course
```
* EBS volumes
goto AWS mgm console => EC2 => Elastic Block store => volumes


# deploy wordpress
```
kubectl apply -f deploy-wordpress.yaml --namespace=ns-eks-course
```
deploy-wordpress.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

get URL of the app:
```
kubectl describe service wordpress --namespace=ns-eks-course | grep Ingress
```
or goto AWS console => EC2

