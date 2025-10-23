# Redis on Kubernetes
Create a cluster with [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

```bash
kind create cluster --name redis --image kindest/node:v1.23.5
```

## Namespace

```bash
kubectl create ns redis
kubectl delete ns redis
kubectl get namespace
```

## Redis `ConfigMap` theory
```bash
ConfigMap (redis-config)
        |
        v
 /tmp/redis/redis.conf   (mounted read-only)
        |
        |-- initContainer copies to -->  /etc/redis/redis.conf  (shared emptyDir)
                                           ↑
                                           |
                                    main container reads this
```

```bash
StatefulSet: redis
│
├─ Pod redis-0
│   └─ PVC data-redis-0  (64Mi from standard SC)
│       └─ Mounted at /data
│
├─ Pod redis-1
│   └─ PVC data-redis-1
│       └─ Mounted at /data
│
└─ Pod redis-2
    └─ PVC data-redis-2
        └─ Mounted at /data
```

## Changes Need to be made in `redis.conf` file
Add these in redis.conf:

```
masterauth my-redis-password
requirepass my-redis-password

dir "/data"

appendonly yes
```

## Storage Class

```bash
kubectl get storageclass
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  84s
```

## Deployment: Redis nodes

```bash
cd kubernetes-redis-cluster/
kubectl apply -n redis -f ./redis/redis-configmap.yml
kubectl apply -n redis -f ./redis/redis-statefulsets.yml

kubectl delete -n redis -f ./redis/redis-configmap.yml
kubectl delete -n redis -f ./redis/redis-statefulsets.yml

kubectl get all -n redis
kubectl describe statefulsets -n redis

kubectl get pods -n redis -o=jsonpath='{.items[*].metadata.labels}'

kubectl -n redis get pods
kubectl -n redis get pv

kubectl -n redis logs redis-0
kubectl -n redis logs redis-1
kubectl -n redis logs redis-2
```

## Test replication status

```bash
kubectl -n redis exec -it redis-0 -- redis-cli
auth my-redis-password
info replication
```


## Deployment: Redis Sentinel (3 instances)

```bash
cd kubernetes-redis-cluster/
kubectl apply -n redis -f ./sentinel/sentinel-statefulsets.yml

kubectl delete -n redis -f ./sentinel/sentinel-statefulsets.yml

kubectl -n redis get pods

# to get IP address of the Pods
kubectl -n redis get pods -o wide

kubectl -n redis get pv

# take down 1 redis node
kubectl -n redis delete pods redis-0

# see "sentinel" logs to see latest elected Master
kubectl -n redis logs sentinel-0
```

## Delete All `Persistent Volumes Claim (PVC)`
```bash
kubectl -n redis delete pvc --all
```

## Delete All Resources
```bash
kubectl delete -n redis -f kubernetes-redis-cluster/redis/redis-configmap.yml
kubectl delete -n redis -f kubernetes-redis-cluster/redis/redis-statefulsets.yml
kubectl delete -n redis -f kubernetes-redis-cluster/sentinel/sentinel-statefulsets.yml
kubectl -n redis delete pvc --all
kubectl delete ns redis
```