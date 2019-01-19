### Varnish使用说明
#### 启动命令
    varnishd -f example.vcl -s malloc,32M -T 127.0.0.1:2000 -a 0.0.0.0:1111
    -f vcl配置文件
    -s 存储类型，存储空间
    -T 管理ip:端口
    -a 对外服务器的ip:端口
#### 软关闭命令
    pkill varnishd
#### 语法说明
    '~':表示匹配的意思，用来匹配acl块或正则表达式
    '\':在字符串字面值中没有特殊含义
    set:为变量赋值
    unset:清空变量值
    
    注：可以给backend、request、document等变量属性复制,有if条件控制，没有循环
#### 集群directors
    1.new vdir = directors.random();
      vdir.add_backend(b1,5);
      创建随机算法的集群，随机数种子可以用户指定，默认、clientId、hash,
      在调用add_backend方法的时候需要将bankend和权重参数传入
    
    2.new vdir = directors.round_robin();
      vdir.add_backend(b1);
      不需要指定权重否则varnish启动报错
    
    3.new vdir = directors.fallback();
      vdir.add_backend(b1);
      vdir.add_backend(b2);
      这个集群算法主要用在高可用，预期效果在b1不可用时所有请求访问b2，但是实际测试没有达到预期
      (后来测试发现需要增加probe探针,fallback才起作用)
    
    4.还有一种集群方案是dns
    
    注：在测试集群的效果时，需要考虑浏览器默认发的一些请求和varnish是否直接从缓存中获取内容
    
#### Grace模式
    当多个客户端同时请求同一个页面时，varnish只会发送一个请求给后端服务器，
    然后让其他几个请求挂起等待返回结果，返回结果后，将这个请求结果发送给其他客户端，
    但是如果请求数量巨大那么请求队列就会很长，如果第一个请求没有获取到内容，
    会造成等待的请求无效等待，所以为了解决这个问题，
    可以只是varnish保持缓存的对象超过他们的TTL(Time-To-Live),并提供旧的内容给正在等待的请求
    
#### Saint模式
