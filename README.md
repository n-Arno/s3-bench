s3-bench
========

Quick and dirty setup to bench Scaleway object using several clients on a Kapsule cluster

prerequisites
--------------

- A Kapsule cluster with at least 5 nodes
- An API key with ObjectStorageFullControl

usage
-----

Edit warp.yaml and configure the API key (and optionally the region/endpoint if PAR is not the target)

```
apiVersion: v1
kind: Secret
metadata:
  name: warp-secret
stringData:
  WARP_HOST: s3.fr-par.scw.cloud:443
  WARP_REGION: fr-par
  WARP_ACCESS_KEY: SCWxxxxx
  WARP_SECRET_KEY: yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
  WARP_TLS: "true"
```

Start the environment

```
kubectl apply -f warp.yaml
```

List the created pods

```
kubectl get pods

NAME                           READY   STATUS    RESTARTS   AGE
warp-client-0                  1/1     Running   0          81m
warp-client-1                  1/1     Running   0          81m
warp-client-2                  1/1     Running   0          81m
warp-client-3                  1/1     Running   0          81m
warp-client-4                  1/1     Running   0          81m
warp-server-7c969f4f5d-5xms5   1/1     Running   0          9m20s
```

Connect to the server pod

```
kubectl exec -it warp-server-7c969f4f5d-5xms5 -- /bin/ash
```

Run the benchmark through the clients

```
./run.sh <objsize> <bucketname> --warp-client=file:clients
```

Where `objsize` is for example `1MiB`, `10MiB` or `1GiB` and `bucketname` is a unique bucket name (it will be created if missing, populated and cleaned up beforehand and afterward). For example:

```
./run.sh 100MiB arno-warp --warp-client=file:clients
+ /warp mixed --autoterm '--obj.size=100MiB' '--bucket=arno-warp' --get-distrib 40 --put-distrib 30 --stat-distrib 20 --delete-distrib 10 '--warp-client=file:clients'
╭─────────────────────────────────╮
│ WARP S3 Benchmark Tool by MinIO │
╰─────────────────────────────────╯

Preparing: Client  100.64.4.172:7761 : Requested stage prepare start...
(...snip...)
```

Optionally, you can run the benchmark directly

```
./run.sh <objsize> <bucketname>
```

cleanup
-------

```
kubectl delete -f warp.yaml
```
