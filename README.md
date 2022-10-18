# 记录一些在实际工作中碰到的问题以及解决方案

## 前端使用 `vue-pdf` 进行 pdf 预览和打印

+ 文档链接地址：https://www.npmjs.com/package/vue-pdf

+ 已经将修改过源码的vue-pdf包放在同级目录，已经解决了预览和打印乱码的问题（版本：4.2.0）

+ 字体问题导致的乱码解决方案的地址：https://github.com/FranckFreiburger/vue-pdf/issues/229

  ```js
  // IE 139.5 其他 145
  let userAgent = navigator.userAgent; //判断浏览器内核
  if (userAgent.indexOf('Trident') != -1) {
      // IE 内核
      this.$refs.myPdfComponent[0].print(139.5);
  }else{
      // 非 IE 内核
   	this.$refs.myPdfComponent[0].print(145);
  }
  // 以上代码解决不同浏览器打印出现的不完整的调整（比如：每页下面的页码没显示出来）
  ```

  ```vue
  <template>
  	<pdf
          ref="myPdfComponent"
          v-for="i in numPages"
          :key="i"
          :src="src"
          :page="i"
          style="display: inline-block;width: 100%;border: 1px solid #ccc;"
  ></pdf>
  </template>
  <script>
  import pdf from 'vue-pdf'
  export default {
  	components: {pdf},
      data(){
          return{
              src:'',
              numPages: undefined
          }
      },
      mounted(){
          this.$post('/rcfz/expertDeclarationFileService/eleDeclarForm').then(res => {
              this.fileUpload.getFileUrl(res.alikey).then(key => {
                  this.src = pdf.createLoadingTask( key )
              }).then(_=>{
                  this.src.promise.then(pdf => {
                      this.numPages = pdf.numPages;
              });
          }) 
      },
      methods:{
      	handlePrint() {
              if (!this.canClick) return;
              // IE 139.5 其他 145
              let userAgent = navigator.userAgent; //判断浏览器内核
              if (userAgent.indexOf('Trident') != -1) {
                  // IE 内核
                  this.$refs.myPdfComponent[0].print(139.5);
              }else{
                  // 非 IE 内核
                  this.$refs.myPdfComponent[0].print(145);
              }
              
          }
      }
  }
  </script>
  ```

# nginx 学习笔记

## 安装
+ 首先需要安装一些前置的东西，因为这些是nginx所需要的
```sh
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel #后面两个是和https的ssl有关，不需要可以不安装
```
+ 在终端输入`vim /etc/yum.repos.d/nginx.repo`创建 nginx.repo，然后复制如下内容：
```sh
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/8/$basearch/ #你可以查一下自己的centos版本 cat /etc/redhat-release 填写自己的版本
gpgcheck=0
enabled=1
```
+ 然后在终端输入`yum install nginx`就开始安装了
+ 最后`nginx -v`查看是否正确安装
+ 查看所有 nginx 文件 `find / -name nginx`

## 配置
+ 查看nginx的所有安装位置：`rpm -ql nginx` rpm 是linux的rpm包管理工具，-q 代表询问模式，-l 代表返回列表
+ 进入 `cd /etc/nginx` 目录
    * `vim /etc/nginx/nginx.conf` 打开nginx总配置文件
    ```sh
    user nginx; # 运行用户，默认是nginx，一般不用改
    worker_processes auto; # nginx进程，一般设置和cpu核心数一样，利于处理高并发
    error_log /var/log/nginx/error.log; # 错误日志存放位置
    pid /run/nginx.pid; # 进程pid存放位置

    include /usr/share/nginx/modules/*.conf;

    events {
        worker_connections 1024; # 单个后台进程的最大并发数
    }
    http {
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"'; # 格式化输出日志

        access_log  /var/log/nginx/access.log  main; # 成功日志存放位置

        sendfile            on; # 开启高效传输模式
        tcp_nopush          on; # 减少网络报文段的数量
        tcp_nodelay         on;
        keepalive_timeout   65; # 保持连接时间，也叫超时时间
        types_hash_max_size 2048;

        include             /etc/nginx/mime.types; # 文件扩展名与类型映射表
        default_type        application/octet-stream; # 默认文件类型

        gzip on; # 开启gzip压缩
        gzip_types text/plain application/javascript text/css # 给指定类型的文件进行压缩

        # include /etc/nginx/conf.d/*.conf; # 子配置项的位置

        server {
            listen       80 default_server; # 配置监听端口
            listen       [::]:80 default_server; # 配置监听端口
            server_name  _; # 服务器名，如果有多域名或多ip，可以写在这里，使用虚拟主机技术
            root         /usr/share/nginx/html; # 服务默认启动目录

            # Load configuration files for the default server block.
            # include /etc/nginx/default.d/*.conf;

            location / {
            }

            error_page 404 /404.html;
                location = /40x.html {
            }

            error_page 500 502 503 504 /50x.html;
                location = /50x.html {
            }
        }
    ```

## 使用
- 启动
    + 直接在控制台键入 `nginx` 即可启动
    + 也可以使用 linux 命令启动 `systemctl start nginx.service`
    + 查看服务运行状态 `ps aux | grep nginx`
- 停止
    + 丛容停止：`nginx -s quit`
    + 立即停止：`nginx -s stop`
        * 区别在于：当前nginx进程正在工作，是否立即停止，还是等工作处理完再停止
    + 利用 linux命令停止：`systemctl stop nginx.service`
    + 杀死进程：`killall nginx`
- 重启
    + `systemctl restart nginx.service`
- 重载配置文件
    + `nginx -s reload` 这样修改了配置文件就不用停止再启动了
- 查看端口占用情况
    + `netstat -tlnp`

## nginx的权限管理
```sh
http {
    server {
        listen 80;
        root /usr/share/nginx/html;

        location /a {
            deny 123.9.51.42; # 设置禁止访问的ip
            allow 45.76.202.231; # 设置允许访问的ip
        }

        location /b {
            allow 45.76.202.231;
            deny all; # deny在后，这样设置就只允许指定ip访问
        }

        location /c {
            deny all; # deny在前，这样设置下面的就不会生效，就所有人都不能访问了
            allow 45.76.202.231;
        }

        location =/img { # 精确匹配以 /img 开头的路径
            allow all;
        }
        location =/admin {
            deny all;
        }
        
        location ~\.php { # 使用正则匹配 .php 结尾的文件
            deny all;
        }
    }
}
```

## 反向代理解决前端跨域问题
+ 例如：前端应用部署在 116.62.148.16:80 上，但是接口请求的是 157.18.123.67:3000，就可以如下配置nginx
```sh
http {
    server {
        listen 80;
        server_name 116.62.148.16;
        location / {
            root /usr/share/nginx/html; #前端应用部署的位置
        }
    }
    server {
        listen 3000;
        server_name 157.18.123.67;
        location /api { # 前端应用中 ajax 请求的开头，如：axios.get('http://157.18.123.67:3000/api/getlist')
            proxy_pass http://157.18.123.67:3000; # 真实的接口地址

            # 下面还有一些关于反向代理的其他设置
            #proxy_set_header : 在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息。
            #proxy_connect_timeout : 配置Nginx与后端代理服务器尝试建立连接的超时时间。
            #proxy_read_timeout : 配置Nginx向后端服务器组发出read请求后，等待相应的超时时间。
            #proxy_send_timeout : 配置Nginx向后端服务器组发出write请求后，等待相应的超时时间。
            #proxy_redirect : 用于修改后端服务器返回的响应头中的Location和Refresh。
        }
    }
}
```

## 适配PC和移动端
+ 像淘宝、京东那样，写的是两份代码，然后根据 user_agent 来判断走哪个文件夹
```sh
http {
    server {
        listen 80;
        server_name 116.62.148.16;
        location / {
            root /usr/share/nginx/pc; # 默认pc端项目
            if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
                root /usr/share/nginx/mobile; # 切换到移动端项目
            }
            try_files $uri $uri/ /index /index.html /index.htm;
        }
    }
}
```

## nginx root与alias 区别
+ alias 只能用于 location 中
```sh
http {
    server {
        listen 80;
        server_name 116.62.148.16;

        location /root {
            root /usr/share/nginx/roots;
            index index.html index.htm;
        }

        location /alia {
            alias  /usr/share/nginx/alias;
            index index.html index.htm;
        }
    }
}
```
+ 如上，如果访问 /root/1.jpg，实际url会被拼接成：/usr/share/nginx/roots/root/1.jpg
+ 如上，如果访问 /alia/1.jpg，实际url会被拼接成：/usr/share/nginx/alias/1.jpg
