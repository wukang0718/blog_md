# nginx设置编码格式utf-8
 
 > 在server下配置charset   utf-8;

 ```
    server {
        listen       8000;
        server_name  localhost;
        
        charset utf-8;
    }
 ```
 
 > 后台使用tomcat时，get请求参数乱码更改nginx编码格式设置无效  
   需要更改tomcat编码格式
   
   [方法请参考](https://blog.csdn.net/pcxbest/article/details/24418303)