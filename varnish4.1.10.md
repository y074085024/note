### Varnish使用说明
#### 启动命令
    varnishd -f example.vcl -s malloc,32M -T 127.0.0.1:2000 -a 0.0.0.0:1111
    -f vcl配置文件
    -s 存储类型，存储空间
    -T 管理ip:端口
    -a 对外服务器的ip:端口
