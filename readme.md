
## Replication


Documentation [here](https://redis.io/topics/replication)

### Docker-Compose Command

```
cd redis-7.2
sudo docker-compose up
sudo docker-compose down
```

### Running Sentinels

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