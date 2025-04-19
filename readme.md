# Redis Replication & Sentinel Setup

Redis-6.0 & Redis-7.2 versions configuration are provided.  
Official Redis Replication Documentation: [redis.io/topics/replication](https://redis.io/topics/replication)

---

## Docker-Compose Command (Redis 7.2)

```bash
cd redis-7.2
sudo docker-compose up
sudo docker-compose down
```

---

## Running Sentinels

Redis Sentinel Documentation: [redis.io/docs/management/sentinel](https://redis.io/docs/management/sentinel/)

In `redis-7.2`, replace `"redis-0"` with the actual **Docker network IP address**.

### Example Sentinel Config (`sentinel.conf`)

```ini
#******** BASIC CONFIG ********#
port 5000
sentinel monitor mymaster redis-0 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster akash-pass
#******************************#
```

> After starting Sentinels, if a master dies, Sentinel will **automatically update** its config files and elect a new master.

---

## Starting Redis in Sentinel Mode

```bash
cd /redis-7.2/clustering/

sudo docker run -d --rm --name sentinel-0 --net redis-network -v ${PWD}/sentinel-0:/etc/redis/ redis:7.2-alpine redis-sentinel /etc/redis/sentinel.conf

sudo docker run -d --rm --name sentinel-1 --net redis-network -v ${PWD}/sentinel-1:/etc/redis/ redis:7.2-alpine redis-sentinel /etc/redis/sentinel.conf

sudo docker run -d --rm --name sentinel-2 --net redis-network -v ${PWD}/sentinel-2:/etc/redis/ redis:7.2-alpine redis-sentinel /etc/redis/sentinel.conf
```

### Logs and CLI Access

```bash
sudo docker logs sentinel-0
sudo docker exec -it sentinel-0 sh
redis-cli -p 5000
info
sentinel master mymaster
```

---

## Clean Up

```bash
sudo docker rm -f redis-0 redis-1 redis-2
sudo docker rm -f sentinel-0 sentinel-1 sentinel-2
```
