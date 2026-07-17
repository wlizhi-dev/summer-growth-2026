# Nginx 企业服务器部署实践

> 项目：Linux Server Project
> 环境：Ubuntu Server 24.04
> 虚拟化：VMware
> Web服务：Nginx

------

### 1. 项目目标

在 Ubuntu Server 中部署 Nginx Web 服务，实现：

- 安装并启动 Nginx
- 创建企业站点目录
- 配置虚拟主机
- 使用域名区分站点
- 使用 curl 验证访问结果

最终效果：

访问：

```
enterprise.local
```

返回：

```
Enterprise Server

Nginx deployment successful.
```

------

### 2. 安装 Nginx

更新软件源：

```bash
sudo apt update
```

![1784267016035](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784267016035.png)

 安装： 

```bash
sudo apt install nginx -y
```

![1784267141177](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784267141177.png)

 查看服务状态： 

```bash
systemctl status nginx
```

正常状态： Active: active (running) 

![1784267211210](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784267211210.png)

### 3.Nginx目录结构 

 Ubuntu默认采用： 

```
/etc/nginx/
│
├── nginx.conf
│
├── sites-available
│
└── sites-enabled
```

 说明： 

| 目录            | 作用             |
| --------------- | ---------------- |
| sites-available | 保存所有站点配置 |
| sites-enabled   | 当前启用的站点   |

 关系： 

```
sites-available
        |
        | 软链接
        ↓
sites-enabled
        |
        ↓
    Nginx加载
```

### 4.创建网站目录 

 创建企业网站目录： 

```bash
sudo mkdir -p /var/www/enterprise-site
```

![1784267775754](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784267775754.png)

创建首页：

```bash
sudo vim /var/www/enterprise-site/index.html
```

内容：

```html
<!DOCTYPE html>
<html>

<head>
<title>Enterprise Lab</title>
</head>


<body>

<h1>Enterprise Server</h1>

<p>Nginx deployment successful.</p>

<p>Host: enterprise-lab</p>


</body>

</html>
```

### 5.配置Nginx虚拟主机 

 进入配置目录： 

```bash
cd /etc/nginx/sites-available
```

 创建： 

```bash
sudo vim enterprise-site
```

![1784268198151](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784268198151.png)

配置： 

```nginx
server {

    listen 80;

    server_name enterprise.local;


    root /var/www/enterprise-site;

    index index.html;


    location / {

        try_files $uri $uri/ =404;

    }


    access_log /var/log/nginx/enterprise_access.log;

    error_log /var/log/nginx/enterprise_error.log;

}
```

###  6. 启用站点 

创建软链接：

```bash
sudo ln -s 
/etc/nginx/sites-available/enterprise-site 
/etc/nginx/sites-enabled/
```

 查看： 

```bash
ls /etc/nginx/sites-enabled
```

 应该看到： enterprise-site

![1784268349663](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784268349663.png)

![1784268480182](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784268480182.png)

### 7. 禁用默认站点 

Ubuntu默认站点：

```
default
```

会返回：

```
Welcome to nginx!
```

关闭：

```bash
sudo rm /etc/nginx/sites-enabled/default
```

原因：

避免请求匹配默认server。

### 8. 检查配置

修改后必须检查：

```bash
sudo nginx -t
```

成功：

```
syntax is ok
test is successful
```

![1784269055173](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784269055173.png)

### 9. 重载Nginx

不要直接restart。

使用：

```bash
sudo systemctl reload nginx
```

原因：

| 操作    | 效果                         |
| ------- | ---------------------------- |
| restart | 停止服务再启动，可能中断访问 |
| reload  | 重新读取配置，不影响连接     |

生产环境优先使用reload。

### 10. 测试网站

测试：

```bash
curl -H "Host:enterprise.local" localhost
```

正确结果：

```html
<h1>Enterprise Server</h1>

<p>Nginx deployment successful.</p>
```

说明：

-  Nginx运行正常 
-  server_name匹配成功 
-  网站目录配置正确

### 11. 遇到的问题记录

#### 问题1：访问出现 Welcome to nginx

现象：

```
Welcome to nginx!
```

![1784269107289](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784269107289.png)

原因：

默认站点优先匹配。

![1784269142457](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784269142457.png)

解决：

删除默认配置：

```bash
sudo rm /etc/nginx/sites-enabled/default
```

重新加载：

```bash
sudo systemctl reload nginx
```

![1784269202195](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784269202195.png)

![1784269233073](C:\Users\Bryanchen\AppData\Roaming\Typora\typora-user-images\1784269233073.png)

#### 问题2：sites-available修改后没有效果

原因：

修改配置文件后：

Nginx不会自动读取。

解决：

执行：

```bash
sudo nginx -t

sudo systemctl reload nginx
```

