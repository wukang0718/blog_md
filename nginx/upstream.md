# nginx负载均衡

### 为什么需要负载均衡服务
> 最开始的部署模型都是点对点的部署，
  但是随着企业的用户的增长以及带来的客户的海量的请求，
  给服务端带来海量的并发，
  导致服务的响应不及时这个时候就需要不断的扩容后端的服务，
  对于前端就需要负载均衡来均分请求，以提升整体的后端的吞吐率
  所以对于请求而言，负载均衡就可以很好的均摊请求，
  让后端的整体的吞吐率和性能不断的增加，
  以满足业务增长和需求  
> 另外点对点的对接，一个点挂掉，整体的服务就会挂掉，
  但是有了负载均衡，即使一个点挂掉了，其他的点还可以正常使用，
  只要把这一个点剔除就可以了，这样的话，服务的高可用就可以很好的满足业务的需求
## 基于LVS的中间件架构  
### 按照范围分个类
 * GSLB
 > 按照影响范围定的GSLB是一个全局的负载均衡，节点范围比较庞大，地域比较宽高
   往往都是按照国家为单位或者省为单位，来进行全局的负载均衡
 * SLB
 > SLB的范围小，小的逻辑地域决定了它的部分服务的实时性和响应性是非常好的
   nginx就是一个典型的SLB
### 根据网络模型OSA分类
 * 四层负载均衡
 > OSA里的传输层，支持TCP/IP协议，所以只需要对TCP/IP协议包转发，就可以实现负载均衡，
   性能非常快，只需要对顶层进行应用处理，而不需要做复杂的逻辑
 * 七层负载均衡
 > 七层负载均衡是在应用层，所以它可以完成很多应用方面的请求，
   可以实现http信息的改写、头信息的改写、安全应用信息的控制以及转发等等的规则
   nginx就是一个典型的七层负载均衡的SLB
 
### nginx实现负载均衡的原理
 > 使用proxy_pass把所有客户端的请求转发到对应后端的服务器上，
   只是它转发的不是一台而是一组虚拟的服务池  upstream server   
   upstream server可以定义所有服务的单元，实现对于组内服务器的不断的轮询，
   这样所有的用户请求过来，就可以分发到不同的服务上，实现负载均衡
   
### 配置语法
 ```
    Systax: upstream name {...}
    Dafault: ——
    Context: http
 ```
 > 注意：  upstream 必须在http下一层，即在server层以外
 ```
    upstream test {
        server localhost:3000;
        server localhost:3001;
        server localhost:3002;
    }
    server {
        listen       80;
        server_name  localhost;
        
        location / {
            root   D:/wukang/blog_md/test;
            index  index.html index.htm;
        }

        location /get1 {
            proxy_pass   http://test;
        }
    }
 ```
 > upstream 内的server不仅支持ip的写法也支持域名的写法、域名加端口的写法,
   每个server后面还可以增加配置参数
   * backup        表示这是一个备份节点，正常不提供服务，当其他服务全部挂掉，这个服务提供服务
   * weight        表示server的权重，对于轮询的规则而言，这个参数越大，权重越大,触发的几率也就越高
   * down          表示当前的server暂时不参与负载均衡
   * max_fails     允许请求失败的次数
   * fail_timeout  经过max_fails后服务暂停的时间，默认10s
   * max_conns     限制最大的接收的连接数
 ```
    upstream test {
        server test1.com         weight=5;
        server test2.com:8080    backup;
    }
 ```
### 轮询规则 > 调度算法
 * 轮询         nginx默认的轮询规则是按照时间顺序逐一分配到不同的后端服务器，如果服务down掉，能自动剔除  
 * 加权轮询     weight值越大，分配到的访问几率越高
 * ip_hash     每个请求按访问ip的hash结果分配，这样来自同一个ip的固定访问一个后端服务器，这样可以保证cookie和session的一致
 * url_hash    按照访问的URL的hash结果来分配请求，是每个URL定向到同一个后端服务器
 * leash_conn  最少连接数，哪个机器连接数少就分发
 * hash关键数值 hash自定义的key
 > ip_hash使用方法  
   缺陷： 如果前台还有一层代理的话，ip_hash的就不是用户真实的ip，固定的一个ip总是转发到同一台服务器上，失去了负载均衡的意义
 ```
  upstream test {
     ip_hash;
     server localhost:3000;
     server localhost:3001;
     server localhost:3002;
 }
 ```  
 > url_hash   
   在1.7.2以后的版本出现的  
   语法：  
 ```
    Systax: hash key [consistent]
    Default: ——
    Context: upstream
 ```
 ```
    upstream test {
        hash $request_uri;
        server localhost:3000;
        server localhost:3001;
        server localhost:3002;
    }
 ```
  