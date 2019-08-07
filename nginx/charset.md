# nginx设置编码格式utf-8
 
 > 在server下配置charset   utf-8;

 ```
    server {
        listen       8000;
        server_name  localhost;
        
        charset utf-8;
    }
 ```