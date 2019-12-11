# curl

语法结构
```bash
curl [options...] <url>
```
GET请求，参数可以直接附加在URL后面。
```bash
curl http://www.baidu.com?param1=val1
```
GET请求，可以指定参数。
```bash
# -G参数等同于--get参数，表示使用GET方法发送参数
# -d参数等同于--data参数，表示要发送的POST参数，如果没有显式指定请求方式则默认是POST方式
curl -G -d 'param1=val1&param2=val2' http://www.baidu.com
```
POST请求
```bash
# -d参数等同于--data参数，表示要发送的POST参数
curl -d 'param1=val1&param2=val2' http://www.baidu.com
```
请求也可以显式地指定GET或POST或其他HTTP动词。
```bash
# 其实这里的-d参数默认已经是POST，不需要使用-X POST来指定POST请求方式
curl -X POST -d 'param1=val1&param2=val2' http://www.baidu.com
```
自定义请求头
```bash
# -H参数等同于--header参数，表示要发送到服务端的自定义请求头
curl -H 'Content-Type:application/json' -d 'param1=val1&param2=val2' http://www.baidu.com
```
cookie
```bash
# --cookie参数让curl发送cookie
curl --cookie 'cookieKey=cookieValue' -d -d 'param1=val1&param2=val2' http://www.baidu.com
```