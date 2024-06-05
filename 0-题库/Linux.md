1. 建立符号连接的Linux命令是：

   答： ln

2. 在vi编辑器中，强制退出不保存的命令是：

   答：:q!

3. vi编辑器中，自下而上查找字符串和自上而下查找字符串应该怎么做？

   答：在命令模式下使用?xxx 自下而上查找，/xxxx自上而下查找

4. linux防火墙iptabls拒绝所有客户端ping数据包的规则

   答：

   ```bash
   iptables -A INPUT -s ! 127.0.0.1 -p icmp -j DROP
   iptables -A INPUT -s 0.0.0.0 -p icmp -j DROP
   ```

5. 在Linux上，对于多进程，子进程继承了父进程的下列哪些？

   答：共享内存、信号掩码、已打开的文件描述符

   

