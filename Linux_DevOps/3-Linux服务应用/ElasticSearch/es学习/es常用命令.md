### 集群访问测试

```bash
[root@k8s-master ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200
{
  "name" : "elasticsearch-cluster-2",
  "cluster_name" : "es-cluster",
  "cluster_uuid" : "wNYCcR1QSEuype3zKMbCPw",
  "version" : {
    "number" : "7.16.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "5b38441b16b1ebb16a27c107a4c3865776e20c53",
    "build_date" : "2021-12-11T00:29:38.865893768Z",
    "build_snapshot" : false,
    "lucene_version" : "8.10.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### 查看集群状态

```bash
[root@k8s-master ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200/_cat/nodes
172.20.253.28 61 73 0 0.02 0.18 0.39 cdfhlmrstw - elasticsearch-cluster-2
172.20.153.32 42 78 0 0.02 0.03 0.06 cdfhlmrstw - elasticsearch-cluster-0
172.20.55.25  70 79 0 0.43 0.42 0.49 cdfhlmrstw * elasticsearch-cluster-1

# 查看集群详细信息，后面添加 "?v" 注意： 带 * 符号的表示是当前的master主节点
[root@k8s-master ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200/_cat/nodes?v
ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role  master name
172.20.153.32           54          78   0    0.14    0.06     0.06 cdfhlmrstw -      elasticsearch-cluster-0
172.20.253.28           72          74   0    0.25    0.20     0.38 cdfhlmrstw -      elasticsearch-cluster-2
172.20.55.25            41          79   1    0.40    0.41     0.48 cdfhlmrstw *      elasticsearch-cluster-1

#
[root@localhost ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200/_cluster/state/nodes?pretty
{
  "cluster_name" : "es-cluster",
  "cluster_uuid" : "wNYCcR1QSEuype3zKMbCPw",
  "nodes" : {
    "XrMsfUf8TuSrHnSIaARFHg" : {
      "name" : "elasticsearch-cluster-0",
      "ephemeral_id" : "ve0R7FTgQhmDkH1JiJ7Ucg",
      "transport_address" : "172.20.153.32:9300",
      "attributes" : {
        "ml.machine_memory" : "16774938624",
        "ml.max_open_jobs" : "512",
        "xpack.installed" : "true",
        "ml.max_jvm_size" : "268435456",
        "transform.node" : "true"
      },
      "roles" : [
        "data",
        "data_cold",
        "data_content",
        "data_frozen",
        "data_hot",
        "data_warm",
        "master",
        "ml",
        "remote_cluster_client",
        "transform"
      ]
    },
    "W-w58FrwTdaIO-nmBEi-pA" : {
      "name" : "elasticsearch-cluster-1",
      "ephemeral_id" : "a3O4IOPXTbKXA3L5aphjBw",
      "transport_address" : "172.20.55.25:9300",
      "attributes" : {
        "ml.machine_memory" : "16774938624",
        "ml.max_open_jobs" : "512",
        "xpack.installed" : "true",
        "ml.max_jvm_size" : "268435456",
        "transform.node" : "true"
      },
      "roles" : [
        "data",
        "data_cold",
        "data_content",
        "data_frozen",
        "data_hot",
        "data_warm",
        "master",
        "ml",
        "remote_cluster_client",
        "transform"
      ]
    },
    "YpZYCGF0R0ec9BeWQrqHmw" : {
      "name" : "elasticsearch-cluster-2",
      "ephemeral_id" : "JzRzusCCQd228phS1YZ8PA",
      "transport_address" : "172.20.253.28:9300",
      "attributes" : {
        "ml.machine_memory" : "16774930432",
        "ml.max_open_jobs" : "512",
        "xpack.installed" : "true",
        "ml.max_jvm_size" : "268435456",
        "transform.node" : "true"
      },
      "roles" : [
        "data",
        "data_cold",
        "data_content",
        "data_frozen",
        "data_hot",
        "data_warm",
        "master",
        "ml",
        "remote_cluster_client",
        "transform"
      ]
    }
  }
}
```

### 查看集群中的master

```bash
# 方法一
[root@k8s-master ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200/_cluster/state/master_node?pretty
{
  "cluster_name" : "es-cluster",
  "cluster_uuid" : "wNYCcR1QSEuype3zKMbCPw",
  "master_node" : "W-w58FrwTdaIO-nmBEi-pA"
}

# 方法二
[root@k8s-master ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200/_cat/master?v
id                     host         ip           node
W-w58FrwTdaIO-nmBEi-pA 172.20.55.25 172.20.55.25 elasticsearch-cluster-1
```

### 查看集群健康状态

- ES集群一种三种状态：green、yellow、red，其中 green 表示健康

```bash
# 方法一
[root@k8s-master ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200/_cat/health?v
epoch      timestamp cluster    status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1715658387 03:46:27  es-cluster green           3         3      4   2    0    0        0             0                  -                100.0%

# 方法二
[root@k8s-master ~]# curl -u elastic:gZio7RJ8YJrWtswN http://172.10.148.241:9200/_cluster/health?pretty
{
  "cluster_name" : "es-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 2,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

### 查看所有索引

```bash
http://127.0.0.1:9200/_cat/indices?v&pretty
```

