#### kafka 现场常用运维命令

1. ##### kafka扩分区

   ./kafka-topics.sh --alter --zookeeper 192.168.13.23:32181 --topic ifaas-target --partitions 6   

   解释： 将 ifaas-target  topic分区数修改为6（分区数只能调大），如果命令没问题执行依然报错，先执行 unset JMX_PORT 命令后再执行分区命令

2. ##### kafka修改comsumer group offset

    ./kafka-consumer-groups.sh --bootstrap-server 192.168.13.23:31090 --group ifaas-data --topic ifaas-target --reset-offsets --to-earliest –execute

   解释：将ifaas-target topic的ifaas-data消费组的消费偏移量重置到最初

​       ./kafka-consumer-groups.sh --bootstrap-server 192.168.13.23:31090 --group ifaas-data --topic ifaas-target --reset-offsets --to-latest –execute

​      解释：将ifaas-target topic的ifaas-data消费组的消费偏移量移动到最近的

​    ./kafka-consumer-groups.sh --bootstrap-server 192.168.13.23:31090 --group  ifaas-data --topic ifaas-target --reset-offsets --to-offset 10 –execute

​      解释：将ifaas-target topic的ifaas-data消费组的消费偏移量移动 10

3. ##### 删除topic

​    /kafka-topics.sh --zookeeper  192.168.13.23:32181 --topic  ifaas-target --delete 

​     解释： 删除 ifaas-target topic