#### 视图库 MongoDB集群 哪些集合需要分片

* face
* persion
* sub_image_info
* motor_vehicle
* non_motor_vehicle

#### 视图库 MongoDB集群 集合分片操作步骤

- 在集群主节点上执行 docker exec -it mongos bash 进入容器内
- 执行 mongo 命令进入mongoDB Shell
- 执行 use admin
- 执行 sh.enableSharding("ifaas_data")
- sh.shardCollection("ifaas_data.motor_vehicle", {"_id": "hashed" }) 对 motor_vehicle 集合分片
- 按照上一步的方式执行 sh.shardCollection("ifaas_data.**集合名称**", {"_id": "hashed" })  依次对其他表分片

#### 检查 MongoDB集群 分片状态

- 执行 db.motor_vehicle.getShardDistribution() 查看motor_vehicle集合的分片状态,看到以下结果表示分片成功，且数据分布均匀。

![](C:\Users\43574\Desktop\分片 .png)

- 依次检查其他表的分片情况

#### MongoDB 数据清理

```shell
#!/bash/bin

#删除创建时间在30天前的数据
MIN_DATE_NANO=`date -d \`date -d '30 days ago' +%Y%m%d\` +%s%N`;
MIN_DATE_MILL=`expr $MIN_DATE_NANO / 1000000`

echo "[`date '+%Y-%m-%d %H:%M:%S'`] start............................................."  >> /root/cron_config/mongo_sweeper/mongo_swepper.log

mongo mongo_server_ip:27017<<EOF 
use admin;
db.auth("user","password");
db.dbIndicators.find({"CREATE_TIME":{\$lt:$MIN_DATE_MILL}},{"_id":1}).forEach(function(item){db.dbIndicators.remove({"_id":item._id});});
exit;
EOF

echo "[`date '+%Y-%m-%d %H:%M:%S'`] end............................................."  >> /root/cron_config/mongo_sweeper/mongo_swepper.log
```



