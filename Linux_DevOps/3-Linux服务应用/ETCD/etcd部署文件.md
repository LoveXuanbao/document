```yaml
apiVersion: v1
kind: Service
metadata:
  name: etcd-headless
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
  labels:
    appKey: etcd-cluster
spec:
  ports:
    - port: 2380
      name: etcd-server
    - port: 2379
      name: etcd-client
  clusterIP: None
  selector:
    appKey: etcd-cluster
  publishNotReadyAddresses: true
---
kind: Service
apiVersion: v1
metadata:
  labels:
    appKey: etcd-cluster
  name: etcd-svc
spec:
  type: NodePort
#  sessionAffinity: None
  ports:
    - port: 2379
      targetPort: 2379
      nodePort: 32084
      name: "etcd-cluster"
    - port: 2380
      nodePort: 32085
      targetPort: 2380
      name: "node"
  selector:
    appKey: etcd-cluster
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: etcd
  labels:
    appKey: etcd-cluster
spec:
  selector:
    matchLabels:
      appKey: etcd-cluster
  serviceName: etcd-headless
  replicas: 3
  template:
    metadata:
      labels:
        appKey: etcd-cluster
#      name: etcd
    spec:
      containers:
        - env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: CLUSTER_NAMESPACE  #名称空间
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: ALLOW_NONE_AUTHENTICATION
              value: "yes"
            - name: ETCD_ENABLE_V2
              value: "true"
            - name: "name"
              value: MY_POD_NAME
            - name: SERVICE_NAME   #内部通信的无头服务名称
              value: "etcd-headless"
#            - name: "listen-peer-urls"
#              value: "http://0.0.0.0:2380"
#            - name: ETCD_LISTEN_CLIENT_URLS
#              value: "http://0.0.0.0:2379"
            - name: ETCD_ADVERTISE_CLIENT_URLS
              value: http://$(MY_POD_NAME).$(SERVICE_NAME).$(CLUSTER_NAMESPACE):2379
#            - name: "initial-advertise-peer-urls"
#              value: http://${MY_POD_NAME}.${SERVICE_NAME}:2380
#            - name: "initial-cluster-token"
#              value: "etcd-cluster"
#            - name: "initial-cluster-state"
#              value: "new"
            - name: INITIAL_CLUSTER   #initial-cluster的值
              value: "etcd-0=http://etcd-0.etcd-headless.$(CLUSTER_NAMESPACE).svc.cluster.local:2380,etcd-1=http://etcd-1.etcd-headless.$(CLUSTER_NAMESPACE).svc.cluster.local:2380,etcd-2=http://etcd-2.etcd-headless.$(CLUSTER_NAMESPACE).svc.cluster.local:2380"
          image: harbor.service.moebius.com/tarzan/metaspace/bitnami/etcd:3.4.9
          imagePullPolicy: IfNotPresent
          name: etcd
          command: ["etcd"]
          args: ["--name=$(MY_POD_NAME)","--listen-peer-urls=http://0.0.0.0:2380","--initial-advertise-peer-urls=http://$(MY_POD_NAME).$(SERVICE_NAME).$(CLUSTER_NAMESPACE).svc.cluster.local:2380","--initial-cluster-state=new","--initial-cluster-token=etcd-cluster-token","--initial-cluster=$(INITIAL_CLUSTER)"]
          ports:
            - containerPort: 2379
              protocol: TCP
            - containerPort: 2380
              protocol: TCP
          volumeMounts:
            - name: etcd-config
              mountPath: /data1/share/etcd/data
              readOnly: false
      volumes:
        - name: etcd-config
          hostPath:
            path: /data1/share/etcd/data
            type: DirectoryOrCreate
#  volumeClaimTemplates:
#    - metadata:
#        name: etcd-config
#      spec:
#        accessModes: [ "ReadWriteMany" ]
#        storageClassName: managed-nfs-storage
#        resources:
#          requests:
#            storage: 10Gi
```

