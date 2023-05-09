# 快速搭建doris数据湖分析测试环境

本文介绍如何使用docker-compose快速搭建doris数据湖分析hive catalog的测试环境，使用的doris版本为1.2.4.1
1. 构建镜像
  - 1. doris镜像构建，可参考文档https://doris.apache.org/zh-CN/docs/dev/install/construct-docker/construct-docker-image 注意点：
    - 1. 构建be的镜像时需要entry_point.sh，可以在doris项目的init_be.sh文件同级目录下找到
    - 2. init_be.sh中使用了sysctl -w vm.max_map_count=2000000，需要root权限才能执行。可以将此语句注释掉，后面在docker-compose.yaml中设置，或直接在宿主机中执行
  - 2. hive相关镜像，直接使用
    - 1. bde2020/hadoop-datanode:2.0.0-hadoop2.7.4-java8
    - 2. bde2020/hive-metastore-postgresql:2.3.0
    - 3. bde2020/hive:2.3.2-postgresql-metastore
2. 构建docker集群
doris的docker集群部署可参考 https://doris.apache.org/zh-CN/docs/dev/install/construct-docker/run-docker-cluster
编写docker-compose.yaml，为了部署hive，还需要entrypoint.sh、hadoop-hive.env、startup.sh这三个文件，以上文件均可在这里找到。
3. 启动
执行docker-compose up -d 
4. 创建hive表
  - 1. 执行docker-compose exec hive-metastore bash
  - 2. 进入容器环境后，执行hive
  - 3. 建表，并写入数据
5. 在doris中查询hive catalog
  - 1. 执行mysql -u root -h 127.0.0.1 -P 9231
  - 2. 连接上doris后，执行如下语句创建hive catalog
```
create catalog hive properties (
"type"="hms",
'hive.metastore.uris' = 'thrift://10.244.1.11:9083',
'dfs.nameservices'='namenode',
'dfs.ha.namenodes.namenode'='nn1',
'dfs.namenode.rpc-address.namenode.nn1'='10.244.1.8:8020',
'dfs.client.failover.proxy.provider.namenode'='org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider'
);
```
6. 执行switch hive切换catalog后，即可查询hive中的表


其中部署hive相关的配置文件来源：https://www.cnblogs.com/upupfeng/p/13452385.html#docker-hive
