# free命令

free命令是用来查看内存占用情况, -m表示以M为单位显示, -h表示以方便阅读的方式显示
```shell
[root@localhost ~]# free 
              total        used        free      shared  buff/cache   available
Mem:         999936      346300      194540       21800      459096      421092
Swap:       2097148        1568     2095580
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:            976         338         190          21         448         411
Swap:          2047           1        2046
[root@localhost ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           976M        338M        189M         21M        448M        411M
Swap:          2.0G        1.5M        2.0G
```

参数说明
```
total 内存总数
used 已经使用的内存数
free 空闲的内存数
shared 不必关心
buff/cache 缓存内存数
available 可用内存数

说明: buff/cache分为used和free两部分, free部分是可以回收的内存
total = used + free + buff/cache
available = free + buff/cache中可以回收的内存
```