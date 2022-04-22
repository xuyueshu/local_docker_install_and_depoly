# docker中安装kafka集群

### 1、集群规划

本机：192.168.10.10

| HOSTNAME | IP ADDR     | PORT | LISTENER |
| -------- | ----------- | ---- | -------- |
| zoo1     | 172.19.0.21 | 2181 |          |
| zoo2     | 172.19.0.22 | 2181 |          |
| zoo3     | 172.19.0.23 | 2181 |          |
| kafka1   | 172.19.0.31 | 9092 | kafka1   |
| kafka2   | 172.19.0.32 | 9093 | kafka2   |
| kafka3   | 172.19.0.33 | 9094 | kafka3   |

### 2、查看dockers内部网络

```shell
[root@localhost docker-zookeeper]# docker network ls
NETWORK ID          NAME                   DRIVER              SCOPE
4c1161d9ac77        bridge                 bridge              local
ed1ab10ce80d        docker-spark_default   bridge              local
d8bd90132281        host                   host                local
9f47f942cb56        none                   null                local
66d14b38c40e        zkkf-net               bridge              local
```

选择zkkf-net

### 3、部署kafka集群



* 创建目录

```shell
mkdir -p /home/docker_apps/docker-kafka
```

​	

* 编排文件

  ​在/home/docker_apps/docker-kafka下编辑`docker-compose.yml`

```yaml
version: '2'

services:
  kafka1:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka1
    container_name: kafka1
    ports:
    - 9092:9092
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
      KAFKA_LISTENERS: PLAINTEXT://kafka1:9092
      KAFKA_ADVERTISED_HOST_NAME: kafka1
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
    - "/home/docker_apps/docker-kafka/kafka1/:/kafka"
    external_links:
    - zoo1
    - zoo2
    - zoo3
    networks:
        zkkf-net:
            ipv4_address: 172.19.0.31

  kafka2:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka2
    container_name: kafka2
    ports:
    - 9093:9093
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9093
      KAFKA_LISTENERS: PLAINTEXT://kafka2:9093
      KAFKA_ADVERTISED_HOST_NAME: kafka2
      KAFKA_ADVERTISED_PORT: 9093
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
    - "/home/docker_apps/docker-kafka/kafka2/:/kafka"
    external_links:  # 连接本compose文件以外的container
    - zoo1
    - zoo2
    - zoo3
    networks:
      zkkf-net:
        ipv4_address: 172.19.0.32

  kafka3:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka3
    container_name: kafka3
    ports:
    - 9094:9094
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9094
      KAFKA_LISTENERS: PLAINTEXT://kafka3:9094
      KAFKA_ADVERTISED_HOST_NAME: kafka3
      KAFKA_ADVERTISED_PORT: 9094
      KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
    volumes:
    - "/home/docker_apps/docker-kafka/kafka3/:/kafka"
    external_links:  # 连接本compose文件以外的container
    - zoo1
    - zoo2
    - zoo3
    networks:
        zkkf-net:
            ipv4_address: 172.19.0.33
networks:
  zkkf-net:
    external:   # 使用已创建的网络
      name: zkkf-net
```

* 启动

  docker-compose up -d

  > [root@localhost docker-kafka]# docker-compose up -d
  > Building with native build. Learn about native build in Compose here: https://docs.docker.com/go/compose-native-build/
  > Pulling kafka1 (wurstmeister/kafka:)...
  > latest: Pulling from wurstmeister/kafka
  > 540db60ca938: Pull complete
  > f0698009749d: Pull complete
  > d67ee08425e3: Pull complete
  > 1a56bfced4ac: Pull complete
  > dccb9e5a402a: Pull complete
  > Digest: sha256:4916aa312512d255a6d82bed2dc5fbee29df717fd9efbdfd673fc81c6ce03a5f
  > Status: Downloaded newer image for wurstmeister/kafka:latest
  > Creating kafka1 ... done
  > Creating kafka3 ... done
  > Creating kafka2 ... done

### 4、安装kafka可视化web `kafka-map`

​        利用docker安装：

```shell
docker run -d \
    -p 8080:8080 \
    -v /home/docker_apps/docker-kafka-map/data:/usr/local/kafka-map/data \
    -e DEFAULT_USERNAME=admin \
    -e DEFAULT_PASSWORD=admin \
    --name kafka-map \
    --restart always dushixiang/kafka-map:latest
```

### 5、开启防火墙端口

firewall-cmd --zone=public --add-port=9092/tcp --permanent 

firewall-cmd --zone=public --add-port=9093/tcp --permanent 

firewall-cmd --zone=public --add-port=9094/tcp --permanent 

重新加载：

firewall-cmd --reload