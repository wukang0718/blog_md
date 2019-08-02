# nginx缓存

 > 配置语法
 >> proxy_cache
   ```
    Syntax: proxy_cache zone | off;
    Default: proxy_cache off;
    Context: http, server, location
   ```
  
 >> 缓存过去周期配置
 
   ```
    Syntax: proxy_cache_valid [code...] time;
    Default: --
    Context: http, server, location
   ```
   
 >> 缓存的维度
 
   ```
    # $scheme  协议
    # $proxy_host 主机域名
    # $rquest_uri url
    Syntax: proxy_cache_key string;
    Default: proxy_cache_key $scheme$proxy_host$rquest_uri;
    Context: http, server, location
   ```
   
 ```
    upstream test {
        server localhost:8080;
        server localhost:8081;
        server localhost:8082;
    }
    
    # /opt/app/cache 缓存存放的目录
    # levels=1:2 分级
    # keys_zone 定义开辟的zone空间的名字，后面的proxy_cache_path就是这个名字
    # 10m 就是大小，开辟的zone空间的大小，一般1m可以存放8000k
    # max_size 控制目录最大多大
    # inactive=60m 不活跃的60分钟，如果缓存文件没有被访问到，就会被清掉
    # use_temp_path 存放临时文件的，一般关闭
    proxy_cache_path /opt/app/cache levels=1:2 keys_zone=test_cache:10m max_size=10g inactive=60m use_temp_path=off;
    
    server {
        listen      80;
        server_name locahost;
        
        location / {
            proxy_cache  test_cache; # 开启缓存
            proxy_pass   http://test; 
            prox_cache_valid   200 304 12h; # 200和304的头信息12h过期，其他的10分钟过期
            proxy_cache_key  $host$uri$is_args$args; # 重新定义proxy_cache_key缓存的key
            add_header     Nginx-cache  "$upstrean_cache_stayus"; # 增加一个头信息
            
            # 如果后端有一台服务器出现500~状态，就跳过这一台
            proxy_next_upstream error timeout invalid_header  http_500 http_502 http_503 http_504;
            include proxy_params;
        }  
    }
 
 ```
 
 > 如何清理指定缓存
 
   * rm -rf 缓存目录内容
   * 第三方扩展模块 ngx_cache_purge
   
 > 让部分页面不缓存
 
   ```
    Syntax: proxy_no_cache string...;
    Default: --
    Context: http, server, location
   ```
   
   ```
    if ($request_uri ~ ^/(login|register|password\/reset)) {
        set $cookie_nocache 1;
    }
    
    location / {
        proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
    }
   ```
 
 > 大文件分片请求  
   优势：每个子请求收到的数据会形成一个独立的文件，一个请求断了，其他的请求不受影响  
   缺点：当文件很大或者slice很小的时候，可能会导致文件描述符耗尽等情况
   ```
    Syntax: slice size;
    Default: slice 0;
    Context: http, server, location
   ```
 
 
 
 
 
 
 
 
 
 
 
 















