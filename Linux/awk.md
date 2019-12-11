# awk

## 目录
- [取出Nginx日志里访问量前N的IP地址](#取出Nginx日志里访问量前N的IP地址)
- [批量终止进程](#批量终止进程)

### 取出Nginx日志里访问量前N的IP地址
access.log内容如下：
```
222.178.10.177 - - [22/Feb/2019:12:07:10 +0000] "GET / HTTP/1.1" 200 109 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.178 - - [22/Feb/2019:12:07:10 +0000] "GET /favicon.ico HTTP/1.1" 200 109 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.179 - - [22/Feb/2019:12:23:04 +0000] "GET / HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.177 - - [22/Feb/2019:12:23:05 +0000] "GET /install/index.html HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.178 - - [22/Feb/2019:12:23:05 +0000] "GET /install/index.html HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.179 - - [22/Feb/2019:12:23:05 +0000] "GET /install/index.html HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.177 - - [22/Feb/2019:12:23:05 +0000] "GET /install/index.html HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.178 - - [22/Feb/2019:12:23:05 +0000] "GET /install/index.html HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.179 - - [22/Feb/2019:12:23:05 +0000] "GET /install/index.html HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
222.178.10.179 - - [22/Feb/2019:12:23:05 +0000] "GET /install/index.html HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0" "-"
```
awk命令：
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr -k1 | head -n N
```
命令说明：  

打印第1列：
```bash
# 将一行中的内容以空格为分隔符，打印第一列
awk '{print $1}' access.log
```

排序：
```bash
# 由于access.log文件中的第一列是IP，所以sort是对IP进行排序
$ awk '{print $1}' access.log | sort
222.178.10.177
222.178.10.177
222.178.10.177
222.178.10.178
222.178.10.178
222.178.10.178
222.178.10.179
222.178.10.179
222.178.10.179
222.178.10.179
```

检查重复出现的行以及重复次数：
```shell
# uniq用于检查及删除在文本文件中重复出现的行
# 删除的重复的行必须是相邻行，所以在使用uniq之前先使用sort使重复行相邻
# -c用于在每行前面显示重复出现的次数
$ awk '{print $1}' access.log | sort | uniq -c
      3 222.178.10.177
      3 222.178.10.178
      4 222.178.10.179
```
按照重复出现的次数(第一列)降序排序：
```shell
# -n用于依照数值的大小排序
# -r用于降序排序
# -k1用于按照第一列排序
$ awk '{print $1}' access.log | sort | uniq -c | sort -nr -k1
      4 222.178.10.179
      3 222.178.10.178
      3 222.178.10.177
```
显示前N行：
```
# head -n N
$ awk '{print $1}' access.log | sort | uniq -c | sort -nr -k1 | head -n 2
      4 222.178.10.179
      3 222.178.10.178
```

### 批量终止进程
目前进程如下：
```bash
$ ps aux | grep test.php
root      1022  0.2  1.0 262436 21188 pts/3    S+   14:32   0:00 php test.php
root      1023  0.2  1.0 262436 21024 pts/2    S+   14:32   0:00 php test.php
root      1025  0.0  0.0   9096   820 pts/1    R+   14:32   0:00 grep --color=auto test.php
```
系统运行了2个test.php相关的进程，使用下面的命令可以批量杀死这两个进程：
```bash
ps aux | grep test.php | grep -v grep |awk '{print $2}'| xargs kill -9
```
命令说明：  

grep -v中的-v参数是反向选择，选择不匹配grep的行：
```bash
# ps aux命令返回的进程列表的第二列是进程ID
$ ps aux | grep test.php | grep -v grep
root      1022  0.0  1.0 262436 21188 pts/3    S+   14:32   0:00 php test.php
root      1023  0.0  1.0 262436 21024 pts/2    S+   14:32   0:00 php test.php
```
将一行中的内容以空格为分隔符，打印第2列：
```bash
$ ps aux | grep test.php | grep -v grep |awk '{print $2}'
1022
1023
```
xargs命令的作用，是将标准输入转为命令行参数。
```bash
# 将前面的进程ID作为命令行参数传给kill -9命令
ps aux | grep test.php | grep -v grep |awk '{print $2}'| xargs kill -9
```