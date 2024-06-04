# hdfs_rebalance

# Ambrai/CDH集群HDFS数据重新平衡的方法

1. hadoop集群的磁盘满了，得挑选其中几个快满得磁盘看看是否都是hdfs数据目录占用得，如果有非hdfs组件的数据写入到这个目录，要根据实际情况看是否可以删除；

2. 重新平台带宽。hdfs的配置中找到**dfs.datanode.balance.bandwidthPerSec**，这个是hdfs重新平衡带宽的配置，字节单位，一般千兆网卡的集群配置为50M左右，万兆网卡的集群配置为500M左右；

3. 数据写入的卷选择策略。datanode的写入数据的选择卷的设置为**可用空间**，按照可用空间多少来进行选择，即剩余越多的磁盘优先写数据

   - ambari

   ```bash
     <property>  
       <name>dfs.datanode.fsdataset.volume.choosing.policy</name>  
       <value>org.apache.hadoop.hdfs.server.datanode.fsdataset.AvailableSpaceVolumeChoosingPolicy</value>   
     </property>
   ```

   - CDH

   ![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/cdh.png)

4. 优先写入空间磁盘的比例。将数据的多少比例写入剩余的磁盘空间更多的卷，一般设置为**0.75**，即把75%的数据写入到空间的盘，25%写入到空间紧张的盘

   ```bash
     <property>  
       <name>dfs.datanode.available-space-volume-choosing-policy.balanced-space-preference-fraction</name>  
       <value>0.75</value>  
     </property>
   ```

5. 适当设置保留空间

   - 即数据盘除了给datanode写数据外，还应该留有余地给其他组件写数据，一般5T左右的磁盘保留150G到200G作为保留空间

     ```
     dfs.datanode.du.reserved=161061273600     ## 保留150G
     ```

6. 以上配置好后，就可以开始进行数据重新平衡

   - Ambari

   ![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/ambari.png)

   - CDH

   ![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/cdh1.png)



# 注意

- 以上配置需要重启hdfs才能生效，如果情况紧急，集群业务环境不允许更改配置并重启hdfs的话，就直接执行步骤6也能起到平衡的效果，只是效果不够理想；
- 这个重新平衡会执行很久，时间取决于数据到底有多么不平衡，以及取决于上面配置的平衡带宽是多少。