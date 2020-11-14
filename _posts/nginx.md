# nginx 配置反向代理解决前后端分离开发
```
    补充一点关于配置文件，默认的肯定是去找 /etc/nginx/nginx.conf , 通过看里面会发现
    在http块下有一句 include /etc/nginx/conf.d/*.conf; 如果是自己加的qiyuan.conf内的信
    息可以写到默认的nginx.conf中但是一本肯定不这样写，而是通过文件包含的形式引入自己的conf配置
    可以将qiyuan.conf放到conf.d/文件夹下，也可以使用第二种方式，自己写一个include去包含sites-enable文件夹，然后 sites-enable 下简历软连接 sites-available中的.conf文件
    第一种好理解，第二种更正规、可扩展性也更好。
```
静态文件去前端的目录下找，动态api请求到接口服务器
/etc/nginx/sites/dev.youmuyou.me.conf   
```
server {
    server_name biqiyuan.dev.youmiyou.cn;
    client_max_body_size 300M;
    
    location /stats/(.*) {
        alias /home/dengbo/easyshop_bi/build/templates/;
    }
    
    location /static/ {
        alias /home/dengbo/easyshop_bi/build/static/;
    }
    
    location / {
        proxy_pass  http://0.0.0.0:16242;
        proxy_set_header Host $host;
    }
    
    # add_header Access-Control-Allow-Credentials true;
    # add_header Access-Control-Allow-Origin 'http://fanfeng.dev.youmiyou.cn';
    # location ~ ^/media/([^/]+)/(\d+x\d+/.+)$ {
    #     root /data/easyshop;
    #     try_files /media/$1/$2  /_img/$1/$2;
    # }
    # location ^~ /qd/ {
    #     root /home/fanfen/easyshop_channel/qd-build/;
    #     try_files $uri /templates/index.html break;
    # }
    # location / {
    #     try_files $uri /build/templates/index.html;
    # }
```

easyshop_bi/local_settings.py
```  
# 使用了前端的代码，并且前端是最新的代码，则在前端同事所在的项目下起服务
DEBUG = True
PORT = 16242
SERVER_NAME = 'biqiyuan.dev.youmiyou.cn'
```
有关 location \ 的注意事项
```
这个块是一个必填想，至于为什么有一个 /stats/ ，在对应的 flask 项目中找到了原因

@app.route('/')
def index():
    return redirect('/stats')
```
