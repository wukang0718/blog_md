# 基于nginx的HTTPS服务

 > 为什么需要HTTPS  
   因为HTTP不安全  
   1. 传输数据被中间人盗用、信息泄露
   2. 数据内容劫持、篡改
   
 > HTtPS协议的实现  
   对传输内容进行加密以及身份验证  
     
 ```
    Syntax: ssl on | off;
    Default: ssl off;
    Context: http, server
    
    Syntax: ssl_certificate file;
    Default: --
    Context: http, server
    
    Syntax: ssl_certificate_key file;
    Default: --
    Context: http, server
 ```
 
 ```
    server  {
        listen      443;
        server_name localhost;
        ssl on;
        ssl_certificate /etc/mgimx/ssl_key/test.crt;
        ssl_certificate_key /etx/nginx/ssl_key/test.key;
    }
 ```


# nginx高级模块

 > secure_link_module  
   制定并允许检查请求的链接的真实性以及保护资源免遭未经授权的访问  
   限制链接生效周期  
   配置语法
   
   ```
    Syntax: secure_link expression;
    Default: --
    Context: http, server, location
    
    Syntax: secure_link_md5 expression;
    Default: --
    Context: http, server, location
   ```
   
   ```
    location / {
        secure_link $arg_md5,arg_expires;
        secure_link_md5 "$secure_link_expires$uri  test";
        
        if ($secure_link = "") {
            return 403;
        }
        
        if ($secure_link = "0") {
            return 10;
        }
    }
   ```
 > geoip_module   
   基于ip地址匹配MaxMind GeoIP二进制文件，读取IP所在地域信息  
   区别国内外网做http访问规则  
   区别国内城市地域做http访问规则  
   ```
    load_module "modules/ngx_http_geoip_module.so";
    load_module "modules/ngx_stream_geoip_module.so";
   ```
   
   ```
    geoip_country /etc/nginx/geoip/GeoIP.dat;
    geoip_city /etc/nginx/GeoLiteCity.dat;
    
    server {
        listen              80;
        server_name         localhost;
        location / {
            if ($geoip_country_code != CN) {
                return 403;
            }
            root    /usr/share/nginx/html;
            index   index.html index.htm;
        }
        
        location /myip {
            default_type   text/pluin;
            return 200 "$remote_addr $geoip_country_name $geoip_country_code $geoip_city";
        }
    }
   ```













































