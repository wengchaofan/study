#### FastDfs 问题定位

1. 查看FastDfs组件状态是否正常 

​    输入以下 命令查看服务状态：

~~~shell
service iface_storaged status    
service iface_trackerd status
~~~

FastDfs正常状态下入如下图所示：

![image-20210720175753581](C:\Users\43574\AppData\Roaming\Typora\typora-user-images\image-20210720175753581.png)

如果服务不正常，可执行以下命令重启FastDfs组件。

~~~shell
service iface_storaged restart 
service iface_trackerd restart
~~~

2. 经过步骤1之后，依然存在问题

 ~~~ 
 执行 tail -f /data/fastdfs/tracker/logs/trackerd.log 观察tracker组件日志
 
 执行 tail -f /data/fastdfs/storage/logs/storaged.log 观察storage组件日志
 ~~~



#### CPU 占用率过高

- ps

```
查看消耗CPU前10排序的进程
   ps aux | sort -k3nr | head -n 10 
```

- top

```
执行命令后输入大写P，进程按照CPU消耗动态排序

top - 15:42:37 up 4 days,  3:34,  3 users,  load average: 0.42, 0.52, 0.50
Tasks: 678 total,   1 running, 677 sleeping,   0 stopped,   0 zombie
%Cpu(s): 13.5 us,  0.3 sy,  0.0 ni, 86.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 13174574+total, 10507505+free, 16186600 used, 10484088 buff/cache
KiB Swap: 13392691+total, 13392691+free,        0 used. 11464280+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                     
33803 intellif  20   0   41060   4040   3084 R  11.8  0.0   0:00.02 top                                                        
50083 root      20   0 1539468 166652  33436 S   5.9  0.1  25:23.63 mongod                                                      
50225 root      20   0 1040444 113548  31960 S   5.9  0.1  10:27.60 mongod                                                       
    1 root      20   0   38180   6260   4008 S   0.0  0.0  18:12.70 systemd                                                     
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.03 kthreadd                                                     
    3 root      20   0       0      0      0 S   0.0  0.0   0:02.70 ksoftirqd/0                                                 
    4 root      20   0       0      0      0 S   0.0  0.0   1:39.30 kworker/0:0                                                 
    5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H    
    
```

#### 内存占用率过高

- top

```
执行命令后输入大写M，进程按照内存消耗动态排序

intellif@ubuntu:~$ top 
top - 15:46:47 up 4 days,  3:39,  3 users,  load average: 0.68, 0.54, 0.50
Tasks: 678 total,   1 running, 677 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.5 sy,  0.0 ni, 97.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 13174574+total, 10507038+free, 16189252 used, 10486104 buff/cache
KiB Swap: 13392691+total, 13392691+free,        0 used. 11463962+avail Mem 
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                     
12638 root      20   0 42.394g 2.559g  26988 S   4.0  2.0  88:58.74 java                                                        
12447 root      20   0 31.734g 1.693g  24976 S   0.7  1.3 274:16.43 java                                                         
 7182 root      20   0 38.909g 1.633g  16364 S   0.7  1.3  37:36.75 java                                                         
 8240 root      20   0 36.488g 1.573g  16384 S   0.0  1.3   8:41.33 java                                                         
10920 root      20   0 35.718g 1.429g  16288 S   0.0  1.1   7:00.03 java                                                         
 5734 root      20   0 11.479g 1.231g  20772 S   0.0  1.0  20:06.09 java                                                         
12523 root      20   0 11.314g 1.210g  24172 S   0.3  1.0   6:52.05 java                                                         
 3685 root      20   0 4225544 942064  11380 S   1.0  0.7 122:18.96 java                                                         
 5225 root      20   0 34.753g 796552  15736 S   0.0  0.6   8:27.94 java                                                         
50253 root      20   0 1902508 488032  33704 S   0.3  0.4  22:21.90 mongod                                                       
 3737 999       20   0 5282356 277816  19200 S   0.0  0.2  22:59.40 mysqld                                                      
50017 root      20   0 1635020 176452  34168 S   1.7  0.1  27:00.58 mongod                                                       
50077 root      20   0 1535988 169288  33340 S   1.7  0.1  25:04.08 mongod                                                       
50083 root      20   0 1539468 166888  33436 S   2.3  0.1  25:25.89 mongod 
```

```
intellif@ubuntu:~$ free -h
              total        used        free      shared  buff/cache   available
Mem:           125G         15G        100G         47M         10G        109G
Swap:          127G          0B        127G

命令解析：
	Total（全部） : 125G
	Used（用户空间进程已用） : 15G
	Free（可用） : 100G
```

#### 磁盘IO使用率过高

- iotop

```
iotop是一个用来监视磁盘I/O使用状况的 top 类工具，可监测到程序使用的磁盘IO的信息
命令解释：
--version #显示版本号
-h, --help #显示帮助信息
-o, --only #显示进程或者线程实际上正在做的I/O，而不是全部的，可以随时切换按o
-b, --batch #运行在非交互式的模式
-n NUM, --iter=NUM #在非交互式模式下，设置显示的次数，
-d SEC, --delay=SEC #设置显示的间隔秒数，支持非整数值
-p PID, --pid=PID #只显示指定PID的信息
-u USER, --user=USER #显示指定的用户的进程的信息
-P, --processes #只显示进程，一般为显示所有的线程
-a, --accumulated #显示从iotop启动后每个线程完成了的IO总数
-k, --kilobytes #以千字节显示
-t, --time #在每一行前添加一个当前的时间
```

- iostat

```
iostat -d -k 2
参数 -d 表示，显示设备（磁盘）使用状态；-k某些使用block为单位的列强制使用Kilobytes为单位；2表示，数据显示每隔2秒刷新一次.

root@ubuntu:~# iostat -d -k 1 10
Linux 4.4.0-87-generic (ubuntu) 	07/17/2020 	_x86_64_	(56 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sdb              68.67         4.54       696.20    1633869  250534018
sdc               0.00         0.02         0.00       5460          0
sda               0.01         1.79         0.00     645072          0
dm-0            129.77         4.51       696.20    1622881  250534012
dm-1              0.00         0.01         0.00       3268          0

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sdb               3.00         0.00        12.00          0         12
sdc               0.00         0.00         0.00          0          0
sda               0.00         0.00         0.00          0          0
dm-0              3.00         0.00        12.00          0         12
dm-1              0.00         0.00         0.00          0          0

命令解释：
tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）。"一次传输"意思是"一次I/O请求"。多个逻辑请求可能会被合并为"一次I/O请求"。"一次传输"请求的大小是未知的。

kB_read/s：每秒从设备（drive expressed）读取的数据量；
kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
kB_read：读取的总数据量；
kB_wrtn：写入的总数量数据量；这些单位都为Kilobytes。

iostat -d -x -k 1 10
root@ubuntu:~# iostat -d -x -k 1 10
Linux 4.4.0-87-generic (ubuntu) 	07/17/2020 	_x86_64_	(56 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00    62.24    0.44   68.21     4.54   696.00    20.41     0.04    0.61    0.17    0.61   0.07   0.46
sdc               0.00     0.00    0.00    0.00     0.02     0.00    47.07     0.00    0.33    0.33    0.00   0.21   0.00
sda               0.00     0.00    0.01    0.00     1.79     0.00   245.74     0.00    1.24    1.24    2.00   0.67   0.00
dm-0              0.00     0.00    0.42  129.31     4.51   696.00    10.80     0.03    0.21    0.16    0.21   0.04   0.46
dm-1              0.00     0.00    0.00    0.00     0.01     0.00    47.36     0.00    0.26    0.26    0.00   0.14   0.00

命令解释：
rrqm/s：每秒这个设备相关的读取请求有多少被Merge了（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge）；wrqm/s：每秒这个设备相关的写入请求有多少被Merge了。

rsec/s：每秒读取的扇区数；
wsec/：每秒写入的扇区数。
rKB/s：The number of read requests that were issued to the device per second；
wKB/s：The number of write requests that were issued to the device per second；
avgrq-sz 平均请求扇区的大小
avgqu-sz 是平均请求队列的长度。毫无疑问，队列长度越短越好。    
await：  每一个IO请求的处理的平均时间（单位是微秒毫秒）。这里可以理解为IO的响应时间，一般地系统IO响应时间应该低于5ms，如果大于10ms就比较大了。这个时间包括了队列时间和服务时间，也就是说，一般情况下，await大于svctm，它们的差值越小，则说明队列时间越短，反之差值越大，队列时间越长，说明系统出了问题。
svctm    表示平均每次设备I/O操作的服务时间（以毫秒为单位）。如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，         系统上运行的应用程序将变慢。
%util： 在统计时间内所有处理IO时间，除以总共统计时间。例如，如果统计间隔1秒，该设备有0.8秒在处理IO，而0.2秒闲置，那么该设备的%util = 0.8/1 = 80%，所以该参数暗示了设备的繁忙程度。一般地，如果该参数是100%表示设备已经接近满负荷运行了（当然如果是多磁盘，即使%util是100%，因为磁盘的并发能力，所以磁盘使用未必就到了瓶颈）。

```

#### 磁盘空间不足

1. 查看磁盘剩余空间大小 

   输入df -h 重点观察/data 磁盘挂载目录剩余空间大小，如下图

   ![image-20210720181858353](C:\Users\43574\AppData\Roaming\Typora\typora-user-images\image-20210720181858353.png)

2. 磁盘空间清理

登录CMP，组件管理 > 清理  > 业务数据清理，选择保留天数，进行业务数据清理。如下图所示：

![image-20210720182318747](C:\Users\43574\AppData\Roaming\Typora\typora-user-images\image-20210720182318747.png)

#### 视频图片结构化能力不足

1. 查看GPU卡利用率

~~~ 
root@intellif-0:/data/fastdfs/storage/logs# nvidia-smi
Tue Jul 20 18:46:44 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.59       Driver Version: 440.59       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla P4            Off  | 00000000:02:00.0 Off |                    0 |
| N/A   48C    P0    36W /  75W |   6877MiB /  7611MiB |     67%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla P4            Off  | 00000000:82:00.0 Off |                    0 |
| N/A   39C    P0    23W /  75W |    561MiB /  7611MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  Tesla P4            Off  | 00000000:85:00.0 Off |                    0 |
| N/A   32C    P8     7W /  75W |      0MiB /  7611MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  Tesla P4            Off  | 00000000:86:00.0 Off |                    0 |
| N/A   41C    P0    25W /  75W |   6987MiB /  7611MiB |      6%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0     58402      C   ...as-structure-engine-cuda-2/IFaasEngineR  6867MiB |
|    1     24472      C   .../ifaas-surveillance-cuda-1/IFaasEngineR   551MiB |
|    3     57683      C   ...as-structure-engine-cuda-1/IFaasEngineR  6977MiB |
+-----------------------------------------------------------------------------+


解释说明：
Fan：显示风扇转速，数值在0到100%之间，是计算机的期望转速，如果计算机不是通过风扇冷却或者风扇坏了，显示出来就是N/A； 
Temp：显卡内部的温度，单位是摄氏度；
Perf：表征性能状态，从P0到P12，P0表示最大性能，P12表示状态最小性能；
Pwr：能耗表示； 
Bus-Id：涉及GPU总线的相关信息； 
Disp.A：是Display Active的意思，表示GPU的显示是否初始化； 
Memory Usage：显存的使用率； 
Volatile GPU-Util：浮动的GPU利用率；
Compute M：计算模式；

下边的Processes显示每块GPU上每个进程所使用的显存情况
~~~

#### 通过进程ID定位某个服务

root 账户执行以下命令

~~~
ll /proc/{pid}/
~~~

入下图所示

![image-20210721150043182](C:\Users\43574\AppData\Roaming\Typora\typora-user-images\image-20210721150043182.png)

