# nginx作静态web资源静态服务

> nginx做为静态资源的http web server可以接收客户端jpeg\html\flv之类的静态资源请求  
  可以直接通过静态资源的存储得到这些文件，返回到客户端，这种方式是一个典型的非常高效的传输模式  
  这种场景常常会用于对于静态资源的处理请求以及动静分离场景下的应用
  
### 动态资源
> 在请求一个网站的时候可以简单的分为静态请求和动态请求  
  动态请求需要经过服务端的解析器进行一些比较复杂的逻辑运算，
  然后对应的数据进行实时性的封装然后返回给用户，这种数据是动态生成的
  
### 静态资源  
> 不需要经过服务端进行运算处理，从文件系统就可以拿到文件的请求，这种资源的请求，就是静态资源的请求  
  这种文件就是静态资源的文件
  
  | 类型 | 种类 |
  | :---: | :--- |
  | 浏览器端渲染 | HTML、CSS、JS | 
  | 图片 | JPEG、GIF、PNG |
  | 视频 | FLV、MPEG |
  | 文件 | TXT、等任意下载文件 |

### nginx配置语法

```
    Syntax: sendfile on | off;
    Default: sendfile off;
    Context: http, server, location, if in location
```

> tcp_nopush  
  作用：sendfile开启的情况下，提升网络包的传输效率  
  不需要实时性的把数据返回到客户端，可以等待多个资源一次发送
```
    Syntax: tcp_nopush on | off;
    Default: tcp_nopush off;
    Context: http, server, location
```

> tcp_nodelay  
  作用：keepalive连接下，提高网络包的传输实时性  
  资源实时的返回到客户端

```
    Syntax: tcp_nodelay on |off;
    Default tcp_nodelay on;
    Context: http, server, location
```

> 压缩   
  作用：资源压缩
  
```
    Syntax: gzip on | off;
    Default: gzip off;
    Context: http, server, location, if in location
```

> 压缩比例， 比例越大压缩越小，但是压缩本身会消耗服务端的性能
```
    Syntax: gzip_comp_level level;
    Default: gzip_comp_level 1;
    Context: http, server, location
```

> http协议版本
```
    Syntax: gzip_http_version 1.0 | 1.1;
    Default: gzip_http_version 1.1;
    Context: http, server, location
```

### 扩展nginx压缩模块

 * http_gzip_static_module -  预读gzip功能
    > 在读取文件，会首先读取该文件的预压缩文件，如果有这个文件就可以直接返回给客户端  
      会节省cpu的压缩时间和cpu请求的压缩的性能的损耗  
      但是会对硬盘有一定的要求
    
    ```
        location ~ ^/download {
            gzip_static on;
            tcp_nopush on;
            root /opt/app/code;
        }
    ```
        
 * http_gunzip_module - 应用支持gunzip的压缩方式
    > 为了解决很少一部分浏览器无法支持gzip解压
    
### 浏览器缓存

 > 校验过期机制
 
   | 校验是否过期 | Expires、Cache-Control(max-age) |
   | :---: | :--- |
   | 协议中Etag头信息校验 | Etag |
   | Last-Modified头信息校验 | Last-Modified |
   
 > nginx配置语法
 
 ```
    # 添加Cache-Control、Expires头
    Syntax: expires [modified] time;
            expires epoch | max | off;
    Default: expires off;
    Context: http, server, location, if in location
 ```
 
 ```
  location ~ .*\.(html|htm)$ {
    expires 24h;
    root    /opt/app/code;
  }
 ```
 
### 跨域访问
 > 为什么浏览器禁止跨域访问  
   不安全，容易出现CSRF攻击！
   
 > nginx语法
  ```
    # Access-Control-Allow-Origin
    Syntax:  add_header name value [always];
    Default: --
    Context: http, server, location, if in location  
  ```
  
  ```
    location ~ .*\.(html|htm)$ {
        add_header Access-Control-Allow-Origin http://www.mywww.com;
        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
    }
  ```
  
### 防盗链
 > 目的： 防止资源被盗用  
   nginx语法
   
   ```
    # http_refer 
    Syntax: valid_referers none | blocked | server_names | string...;
    Default: --
    Context: server, location
   ```  
 
   ```
    location ~ .*\.(jpg|gif|png)$ {
        # none 表示没有refer信息
        # bloked 表示非正常协议的
        # 0.0.0.0 表示允许ip是0.0.0.0或者域名也是可以的
        # ~/google\./ 这种正则的方式也是可以匹配的
        
        valid_referers none bloched 0.0.0.0;
        if ($invalid_referer) {
            return 403;
        }
    }
   ```
