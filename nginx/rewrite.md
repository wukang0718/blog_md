# nginx rewrite规则

 > 实现url的重写以及对匹配的url的重定向
 
 > 配置语法
 
  ```
    # regex  要匹配的路径
    # replacement 要跳转的路径
    # flag 标记ngin rewrite的类型
    Syntax: rewrire regex replacement [flag];
    Default: --
    Context: server, location, if
    
    # rewrite  ^(.*)$ /page/maintain.html break;
  ```
  * flag
  
   | flag | 作用 |
   | :---: | :--- |
   | last | 停止rewrite检测（匹配到以后会访问root配置下的跳转路径） |
   | break | 停止rewrite检测（匹配到后会新建一个请求重新请求对应的路径+跳转） |
   | redirect | 返回302临时重定向，地址栏会显示跳转后的地址 |
   | permanent | 返回301永久重定向，地址栏会显示跳转后的地址 |
   
 > rewrite 规则优先级  
   1. 执行server快的rewrite指令
   2. 执行location匹配
   3. 执行选定的location中的rewrite
   

# nginx 动静分离

 > 什么是动静分离  
 >>  通过中间件将动态请求和静态请求分离
   
 > 为什么动静分离
 >> 分离资源，减少不必要的请求消耗，减少请求延时
 
 ```
    server {
        listen        80;
        server_name   localhost;
        
        root   /opt/app/code;
        
        location ~ \.do$ {
            proxy_pass http://localhost:8080;
            index      index.html index.htm;
        }
        
        location ~ \.(jpg|png|gif)$ {
            expires      1h;
            gzip         on;
        }
    }
 ```
























