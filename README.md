# Run etcd single node by docker-compose

doccker-compose管理的单实例etcd集群，支持数据持久化，支持冷重启，支持公网访问

### Usage

```shell
# 初始data目录为空
gongpengjun@nuc ~$ tree
.
├── README.md
├── data
└── docker-compose.yml

1 directory, 2 files

# 启动etcd cluster
gongpengjun@nuc ~$ docker-compose up -d
Creating network "etcd-single-node_bridge-network" with driver "bridge"
Creating etcd-single-node_etcd-1_1 ... done

# 查看容器状态
gongpengjun@nuc:docker-compose-etcd-single-node$ docker-compose ps
          Name                         Command               State                 Ports
------------------------------------------------------------------------------------------------------
etcd-single-node_etcd-1_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:12379->2379/tcp, 2380/tcp

# 查看data目录树，etcd已经初始化好了数据目录
gongpengjun@nuc ~$ tree
.
├── README.md
├── data
│   └── etcd1
│       └── member
│           ├── snap
│           │   └── db
│           └── wal
│               ├── 0.tmp
│               └── 0000000000000000-0000000000000000.wal
└── docker-compose.yml

5 directories, 5 files

# 集群只有一个etcd节点
gongpengjun@mbp ~$ etcdctl --endpoints=$NUC:12379 -w=table member list
+------------------+---------+--------+--------------------+--------------------+------------+
|        ID        | STATUS  |  NAME  |     PEER ADDRS     |    CLIENT ADDRS    | IS LEARNER |
+------------------+---------+--------+--------------------+--------------------+------------+
| b8c6addf901e4e46 | started | etcd-1 | http://etcd-1:2380 | http://etcd-1:2379 |      false |
+------------------+---------+--------+--------------------+--------------------+------------+
gongpengjun@mbp ~$ etcdctl --endpoints=$NUC:12379 -w=table endpoint health
+----------------+--------+------------+-------+
|    ENDPOINT    | HEALTH |    TOOK    | ERROR |
+----------------+--------+------------+-------+
| 127.0.0.1:2379 |   true | 5.480427ms |       |
+----------------+--------+------------+-------+
gongpengjun@mbp ~$ etcdctl --endpoints=$NUC:12379 -w=table endpoint status
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 127.0.0.1:2379 | b8c6addf901e4e46 |   3.5.7 |   20 kB |      true |      false |         2 |          4 |                  4 |        |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+

# 初始状态没有数据 secret不存在
gongpengjun@mbp ~$ etcdctl --endpoints=$NUC:12379 get secret

# 设置并获取secret
gongpengjun@mbp ~$ etcdctl --endpoints=$NUC:12379 put secret baby-im
OK
gongpengjun@mbp ~$ etcdctl --endpoints=$NUC:12379 get secret
secret
baby-im

# 关闭etcd cluster
gongpengjun@nuc ~$ docker-compose down
Stopping etcd-single-node_etcd-1_1 ... done
Removing etcd-single-node_etcd-1_1 ... done
Removing network etcd-single-node_bridge-network

# 启动etcd cluster
gongpengjun@nuc ~$ docker-compose up -d
Creating network "etcd-single-node_bridge-network" with driver "bridge"
Creating etcd-single-node_etcd-1_1 ... done

# 启动后可以直接查询到secret
gongpengjun@mbp ~$ etcdctl --endpoints=$NUC:12379 get secret
secret
baby-im
```

### Docker Images

- [docker-image-etcd](https://quay.io/repository/coreos/etcd?tab=tags&tag=v3.5.7)
- [etcd official release](https://github.com/etcd-io/etcd/releases/tag/v3.5.7)
