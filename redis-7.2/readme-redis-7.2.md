
## Replication


Documentation [here](https://redis.io/topics/replication)

### Configuration

```
#persistence
dir "/data" (search for "dir ./" & then replace)
dbfilename dump.rdb
appendonly yes
appendfilename "appendonly.aof"
appenddirname "appendonlydir"
```
### redis-0 Configuration

```
protected-mode no
bind 0.0.0.0
port 6379

#authentication
masterauth "akash-pass" (masterauth is in Double Quotes)
requirepass akash-pass

//we set both the Password same because if the Replica becomes master then we don't have to change the other Replicas "masterauth" configuration
```
### redis-1 Configuration

```
protected-mode no
bind 0.0.0.0
port 6379
slaveof redis-0 6379 (search "replicaof <masterip> <masterport>") (in redis-7.2 give docker network's ip address instead of "redis-0" ---> example: slaveof 172.18.0.4 6379)

#authentication
masterauth "akash-pass"
requirepass akash-pass

```
### redis-2 Configuration

```
protected-mode no
bind 0.0.0.0
port 6379
slaveof redis-0 6379 (search "replicaof <masterip> <masterport>") (in redis-7.2 give docker network's ip address instead of "redis-0" ---> example: slaveof 172.18.0.4 6379)

#authentication
masterauth "akash-pass"
requirepass akash-pass
```

### Create Docker Network
```
sudo docker network create redis-network
```

### Create redis containers
sudo docker network create redis-network

```

# remember to update above in configs!

sudo docker network create redis-network

cd /redis-7.2/clustering/

#redis-0
sudo docker run -d --rm --name redis-0 --net redis-network -v ${PWD}/redis-0:/etc/redis/ redis:7.2-alpine redis-server /etc/redis/redis.conf

#redis-1
sudo docker run -d --rm --name redis-1 --net redis-network -v ${PWD}/redis-1:/etc/redis/ redis:7.2-alpine redis-server /etc/redis/redis.conf

#redis-2
sudo docker run -d --rm --name redis-2 --net redis-network -v ${PWD}/redis-2:/etc/redis/ redis:7.2-alpine redis-server /etc/redis/redis.conf

```


## Test Replication

Technically written data should now be on the replicas

```
# go to one of the clients
sudo docker exec -it redis-2 sh
redis-cli
auth "akash-pass"
keys *
set ramos “4”
get ramos
```

## Running Sentinels

Documentation [here](https://redis.io/topics/sentinel)

```
(in redis-7.2 give docker network's ip address instead of "redis-0")

#********BASIC CONFIG************************************
port 5000
sentinel monitor mymaster redis-0 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster akash-pass
#********************************************

After Starting Sentinel's if a Master Dies, then Sentinel Automatically Updates it Config Files
```
Starting Redis in sentinel mode

```

cd /redis-7.2/clustering/

sudo docker run -d --rm --name sentinel-0 --net redis-network -v ${PWD}/sentinel-0:/etc/redis/ redis:7.2-alpine redis-sentinel /etc/redis/sentinel.conf

sudo docker run -d --rm --name sentinel-1 --net redis-network -v ${PWD}/sentinel-1:/etc/redis/ redis:7.2-alpine redis-sentinel /etc/redis/sentinel.conf

sudo docker run -d --rm --name sentinel-2 --net redis-network -v ${PWD}/sentinel-2:/etc/redis/ redis:7.2-alpine redis-sentinel /etc/redis/sentinel.conf


sudo docker logs sentinel-0
sudo docker exec -it sentinel-0 sh
redis-cli -p 5000
info
sentinel master mymaster

# clean up
sudo docker rm -f redis-0 redis-1 redis-2
sudo docker rm -f sentinel-0 sentinel-1 sentinel-2

```

## Running HAProxy
```
# in HAProxy-2.9.6 --> be Careful with "Space & Enter" after "10s"(any seconds) line... Always Place "Enter" after LASTLINE of "Seconds" in Configuration File

sudo docker run -it --rm --name haproxy-container --net redis-72_redis-network -v ${PWD}:/usr/local/etc/haproxy/ -p 6392:6392 haproxytech/haproxy-alpine:2.9.6
```
