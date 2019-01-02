# [etcd](https://blog.csdn.net/u010278923/article/details/71727682)

## 使用etcdv3

``` bash
export ETCDCTL_API=3
```

## 设置、更新key

```bash
# champly @ ChamPlydeMBP in ~/soft/etcd [22:40:55]
$ ./etcdctl put /key/1 1
OK

# champly @ ChamPlydeMBP in ~/soft/etcd [22:41:35]
$ ./etcdctl put /key/1 2
OK
```

## 获取key
```bash
# champly @ ChamPlydeMBP in ~/soft/etcd [22:41:38]
$ ./etcdctl get /key/1
/key/1
2

# 匹配前缀查询
# champly @ ChamPlydeMBP in ~/soft/etcd [22:42:54] C:1
$ ./etcdctl get /key --prefix
/key/1
2
```

## 删除key

```bash
# champly @ ChamPlydeMBP in ~/soft/etcd [22:44:53] C:1
$ ./etcdctl put /key/1 1
OK

# champly @ ChamPlydeMBP in ~/soft/etcd [22:44:56]
$ ./etcdctl del /key/1
1

# 匹配前缀删除
# champly @ ChamPlydeMBP in ~/soft/etcd [22:45:00]
$ ./etcdctl put /key/1 1
OK

# champly @ ChamPlydeMBP in ~/soft/etcd [22:45:02]
$ ./etcdctl del /key --prefix
1
```

## 监听一个key
```bash
# champly @ ChamPlydeMacBook-Pro in ~/soft/etcd [22:46:38] C:130
$ ./etcdctl watch /key/1
PUT
/key/1
1
DELETE
/key/1
```

## 申请租约
从申请开始计算时间
```bash
# champly @ ChamPlydeMBP in ~/soft/etcd [22:47:17]
$ ./etcdctl lease grant 100
lease 694d680edbc7579e granted with TTL(100s)
```

## 授权租约
节点的生命伴随着租约到期将会被删除
```bash
# champly @ ChamPlydeMBP in ~/soft/etcd [22:48:09]
$ ./etcdctl put --lease=694d680edbc7579e /key/1 1
OK
```

## 租约续约
每当到期将会续约
``` bash
# champly @ ChamPlydeMBP in ~/soft/etcd [22:50:03]
$ ./etcdctl lease grant 100
lease 694d680edbc757a2 granted with TTL(100s)

# champly @ ChamPlydeMBP in ~/soft/etcd [22:50:05]
$ ./etcdctl lease keep-alive 694d680edbc757a2
lease 694d680edbc757a2 keepalived with TTL(100)
lease 694d680edbc757a2 keepalived with TTL(100)
lease 694d680edbc757a2 keepalived with TTL(100)
^C
```
