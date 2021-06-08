### 01 Nginx基础Http协议

####  1.1 Http工作原理

> ![image-20210607150601257](images\image-20210607150601257.png)
>
> 在用户点击URL为http://www.qq.com/index.html的链接后，浏览器和web服务器执行以下动作：
>
> 1. 浏览器分析超链接中的URL
> 2. 浏览器向DNS请求解析www.qq.com的IP地址
> 3. DNS将解析出的IP地址返回给浏览器
> 4. 浏览器域服务器建立TCP链接        
> 5. 浏览器请求文档：GET /index.html
> 6. 服务器给出响应，将文档index.html发送给浏览器
> 7. 释放TCP链接
> 8. 浏览器显示index.html中的内容                                                                                                             

#### 1.2 Http请求Request

##### 1.2.1 请求Method

- 客户端向服务端发送请求时，会根据不同的资源发送不同的请求方法Method：
  - GET：用于获取URI对应的资源；（比如看朋友圈）
  - POST：用于提交请求，可以更新或者创建资源，是非幂等的；（发布朋友圈）
  - PUT：用于向指定的URI传送更新资源，是幂等的；（更新朋友圈）
  - DELETE：用于向指定的URI删除资源；（比如删朋友圈）
  - HEAD：用于检查
- 一般创建对象时用POST，更新对象时用PUT；
  - PUT是幂等的，POST是非幂等的；
  - 幂等：对于相同的输入，每次得到的结果都是相等的

##### 1.2.2响应Status

http响应状态码Status-Code以3位数字组成，用来标识该请求是否成功，比如是正常还是错误等，HTTP/1.1中状态码可以分为五大类

| 状态码 | 说明                                           |
| ------ | ---------------------------------------------- |
| 1xx    | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2xx    | 成功，操作被成功接收并处理                     |
| 3xx    | 重定向，需要进一步的操作以完成请求             |
| 4xx    | 客户端错误，请求包含语法错误或无法完成请求     |
| 5xx    | 服务器错误，服务器在处理请求的过程中发生了错误 |

### 02 Nginx Web快速入门

#### 2.1 Nginx虚拟主机

Nginx配置虚拟主机有如下三种方式：

- 基于主机多IP方式

  ![image-20210607172931855](images\image-20210607172931855.png)

- 基于端口的配置方式

  ![image-20210607172947527](images\image-20210607172947527.png)

- 基于多个hosts名称方式（多域名方式）

  ​			`		![image-20210607173010850](images\image-20210607173010850.png)

### 03 Nginx常用模块

#### 3.1 Nginx目录索引

当ngx_http_index_module模块找不到索引文件时，通常会将请求传递给ngx_http_autoindex_module模块

ngx_http_autoindex_module模块处理以斜杠字符结尾的请求，并生成目录列表

##### 3.1.1 ngx_http_autoindex_module场景示例：

模拟搭建企业内部yum仓库

```
server {
	listen	80;
	server_name	mirror.micah.com;
	root /mirror;
	charset	uft-8;		#设定字符集，防止中文字符乱码
	
	location / {
		index index.html;
	} 
	
	location /repo {
		autoindex	on;		#启用目录列表输出
		autoindex_exact_size	off;	#是否在目录列表中输出确切的文件大小，on显示字节，off显示大概单位
		autoindex_locatime	on;		#是否显示本地时间，on本地时区，off UTC时间
	}
}
```

#### 3.2 Nginx访问控制

ngx_http_access_module模块允许限制对某些客户端地址的访问

##### 3.2.1 ngx_http_access_module场景示例：

只允许指定的来源IP访问/centos，其他网段拒绝

```
server {
	listen	80;
	server_name	mirror.micah.com;
	charset	utf-8;
	autoindex	on;		#启用目录列表输出
	autoindex_exact_size	off;	#是否在目录列表中输出确切的文件大小，on显示字节，off显示大概单位
	autoindex_locatime	on;
	
	location / {
		index index.html;
	}
	
	location /centos {
		allow 127.0.0.1;
		allow 10.0.0.1/24;	#允许地址或网段
		deny all;			#拒绝所有人
	}
}
```

ngx_http_auth_basic_module模块允许使用http基本身份验证，验证用户名和密码来限制对资源的访问

##### 3.2.2 ngx_http_auth_basic_module场景示例

基于用户名和密码认证实践

```
server {
	listen	80;
	server_name mirror.micah.com;
	charset	utf-8;
	autoindex	on;		#启用目录列表输出
	autoindex_exact_size	off;	#是否在目录列表中输出确切的文件大小，on显示字节，off显示大概单位
	autoindex_locatime	on;
	
	location / {
		index index.html;
	}
	
	location /centos {
		auth_basic "Auth access Blog Input your Passwd!";
        auth_basic_user_file /etc/nginx/auth.passwd;	#账号密码文件
	}
}
```

#### 3.3 Nginx限流限速

##### 3.3.1 场景一：综合案例

限制web服务器请求数处理为1秒一个，触发值为5，限制用户尽可同时下载一个文件。当下载超过100M则现在下载速度为500k。如果同时下载超过2个视频，则返回提示“请进行会员充值”

```
limit_req_zone $binary_remote_addr zone=req_mg:10m rate=1r/s;
#$binary_remote_addr表示通过这个标识来做限制，限制同一客户端ip地址
#zone=req_one:10m表示生成一个大小为10m，名为req_one的内存区域，用来存储访问的频次信息#rate=1r/s表示允许相同标识的客户端的访问频次，这里限制的是每秒1次

limit_conn_zone $binary_remote_addr zone=conn_mg:10m;
    
server {
	listen 80;
	server_name	mirror.micah.com;
	root /code;
	charset utf-8;
	autoindex	on;	
	autoindex_exact_size	off;
	autoindex_locatime	on;
	
	limit_req zone=req_one burst=5 nodelay;
	#zone=req_one设置使用哪个配置区域来做限制，与上面limit_req_zone里面的name对应
	#burst=5,设置一个大小为5的缓冲区，当有大量请求过来时，超过了访问频次限制的请求可以先放到这个缓冲区内
	#nodelay，超过访问频次并且缓冲区也满了的时候，则会返回503，如果没有设置，则所有请求会等待排队
	
	limit_conn conn_mg 1;
	limit_rate_after 100m;
	limit_rate 500k; 
	
	error_page 503 @errpage;
	location @errpage {
		default_type text/html;
		return 200 "请进行会员充值"
	}
	location / {
		index index.html;
	}
}
```

