# ha-redis 
**Redis high availability with Docker Compose** 

Using Docker Compose to setup an  with sentinel.


## Prerequisite

Install [Docker][4] and [Docker Compose][3] in testing environment


## Docker Compose template of Redis cluster

There are following services in the cluster,

* master: Redis master
* slave:  Redis slave
* sentinel: Redis sentinel
* proxy: The load balancing proxy


The sentinels are configured with a "mymaster" instance with the following properties -

```
sentinel monitor mymaster redis-master 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 5000
```

The details could be found in sentinel/sentinel.conf

The default values of the environment variables for Sentinel are as following

* SENTINEL_QUORUM: 2
* SENTINEL_DOWN_AFTER: 30000
* SENTINEL_FAILOVER: 180000



## Play with it

Build the sentinel and load balancing proxy Docker images

```
docker-compose build
```

Start the redis cluster

```
docker-compose up -d
```

Check the status of redis cluster

```
docker-compose ps
```

The result is 

```
         Name                        Command               State          Ports        
--------------------------------------------------------------------------------------
haredis_master_1     docker-entrypoint.sh redis ...   Up      6379/tcp                                       
haredis_proxy_1      /haproxy-entrypoint.sh hap ...   Up      0.0.0.0:6379->6379/tcp, 0.0.0.0:9000->9000/tcp 
haredis_sentinel_1   sentinel-entrypoint.sh           Up      26379/tcp, 6379/tcp                            
haredis_slave_1      docker-entrypoint.sh redis ...   Up      6379/tcp       
```

Scale out the instance number of sentinel

```
docker-compose scale sentinel=3
```

Scale out the instance number of slaves

```
docker-compose scale slave=2
```

Check the status of redis cluster

```
docker-compose ps
```

The result is 

```
         Name                        Command               State          Ports        
--------------------------------------------------------------------------------------
rediscluster_master_1     docker-entrypoint.sh redis ...   Up      6379/tcp            
rediscluster_sentinel_1   docker-entrypoint.sh redis ...   Up      26379/tcp, 6379/tcp 
rediscluster_sentinel_2   docker-entrypoint.sh redis ...   Up      26379/tcp, 6379/tcp 
rediscluster_sentinel_3   docker-entrypoint.sh redis ...   Up      26379/tcp, 6379/tcp 
rediscluster_slave_1      docker-entrypoint.sh redis ...   Up      6379/tcp            
rediscluster_slave_2      docker-entrypoint.sh redis ...   Up      6379/tcp            
```

 

You can do the test manually to pause/unpause redis server through

```
docker-compose pause master
docker-compose unpause master
```
And get the sentinel infomation with following commands

```
docker-compose exec sentinel redis-cli -p 26379 info Sentinel
docker-compose exec sentinel redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster

```

## References


[1]: https://github.com/mdevilliers/docker-rediscluster
[2]: https://registry.hub.docker.com/u/joshula/redis-sentinel/
[3]: https://docs.docker.com/compose/
[4]: https://www.docker.com
