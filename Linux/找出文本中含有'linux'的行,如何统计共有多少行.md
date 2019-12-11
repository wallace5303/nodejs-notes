# 找出文本中含有'linux'的行,如何统计共有多少行

准备一个文本文件 1.txt
```shell
[root@VM_12_22_centos /]# cat 1.txt
linux fdjalfd
fjdlsfjdlal linux linux fdlaj
fdjaljf
fdjal
fdjal linuxlinuxfjdla
fjadl
```
找出文本中含有'linux'的行
```shell
[root@VM_12_22_centos /]# cat 1.txt | grep 'linux'
linux fdjalfd
fjdlsfjdlal linux linux fdlaj
fdjal linuxlinuxfjdla
```
统计文本中含有'linux'的行数
```shell
[root@VM_12_22_centos /]# cat 1.txt | grep 'linux' | wc -l
3
```

