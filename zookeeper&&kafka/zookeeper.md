# 在dockers中搭建zookeeper集群

### 1、集群规划

本机：192.168.10.10

| hostname | Ip addr     | port      | listener |
| -------- | ----------- | --------- | -------- |
| zoo1     | 172.19.0.21 | 2181:2181 |          |
| zoo2     | 172.19.0.22 | 2182:2181 |          |
| zoo3     | 172.19.0.23 | 2183:2181 |          |

### 3、部署zookeeper集群

* 创建目录

  ```shell
  mkdir -p /home/docker_apps/docker-zookeeper
  ```

  ​

  ​网上很多攻略都是把zookeeper和kafka放到一个编排里去的，个人认为分开的话，管理会比较容易，zookeeper的`docker-compose.yml`如下：

```yaml
version: '3'
services:
    zoo1:
        image: zookeeper
        container_name: zoo1
        restart: always
        hostname: zoo1
        ports:
            - 2181:2181
        volumes: # 挂载数据卷
            - "/home/docker_apps/docker-zookeeper/zoo1/data:/data"
            - "/home/docker_apps/docker-zookeeper/zoo1/datalog:/datalog"
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
        networks:
            zkkf-net:
                ipv4_address: 172.19.0.21
    zoo2:
        image: zookeeper
        container_name: zoo2
        restart: always
        hostname: zoo2
        ports:
            - 2182:2181
        volumes: # 挂载数据卷
            - "/home/docker_apps/docker-zookeeper/zoo2/data:/data"
            - "/home/docker_apps/docker-zookeeper/zoo2/datalog:/datalog"
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
        networks:
            zkkf-net:
                ipv4_address: 172.19.0.22

    zoo3:
        image: zookeeper
        container_name: zoo3
        restart: always
        hostname: zoo3
        ports:
            - 2183:2181
        volumes: # 挂载数据卷
            - "/home/docker_apps/docker-zookeeper/zoo3/data:/data"
            - "/home/docker_apps/docker-zookeeper/zoo3/datalog:/datalog"
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
        networks:
            zkkf-net:
                ipv4_address: 172.19.0.23
networks:
    zkkf-net:
        external: true
            #name: zkkf-net

```



* 执行

  将`docker-compose.yml`放入/home/docker_apps/docker-zookeeper 执行`docker-compose up -d`

  > [root@localhost docker-zookeeper]# docker-compose up -d
  > Building with native build. Learn about native build in Compose here: https://docs.docker.com/go/compose-native-build/
  > Pulling zoo1 (zookeeper:)...
  > latest: Pulling from library/zookeeper
  > a2abf6c4d29d: Pull complete
  > 2bbde5250315: Pull complete
  > 202a34e7968e: Pull complete
  > 4e4231e30efc: Pull complete
  > 707593b95343: Pull complete
  > b070e6dedb4b: Pull complete
  > 46e5380f3905: Pull complete
  > 8b7e330117e6: Pull complete
  > Digest: sha256:2c8c5c2db6db22184e197afde13e33dad849af90004c330f20b17282bcd5afd7
  > Status: Downloaded newer image for zookeeper:latest
  > Creating zoo2 ... done
  > Creating zoo1 ... done
  > Creating zoo3 ... done







