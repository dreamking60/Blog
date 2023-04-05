# Wordpress博客配置

inList: No
inMenu: No
publish: Yes
template: post

本文是关于如何在ubuntu22.02服务器上配置wordpress博客。

本文参考了

1. [保姆级教程！在ECS上部署高版本Wordpress以及运行环境！-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/792151)
2. [Wordpress Sakura🌸主题使用手册 | 流年，谁给过的倾城 | Yremp](https://yremp.live/wordpress-sakura-teach/)
3. [如何解决WordPress上传的文件大小超过php.ini限制 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/444137840)

## 配置环境安装

先对系统进行更新，根据你的服务器地址选择速度比较快的源，请自行查询源替换的相关知识（源替换不难，最简单的方式就是复制粘贴，更改相关配置文件）

```bash
sudo apt update
sudo apt upgrade
```

### Apache

在系统更新完之后，安装wordpress需要的apache

```bash
sudo apt install apache2
```

安装完后使用命令开启apache2，访问服务器对应的ip地址，如果服务运行正常会出现apache default page。

```bash
systemctl start apache2
```

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled.png)

### Mysql

接着需要安装mysql服务。

```bash
sudo apt install mysql-server
```

类似的，在安装完后使用systemctl start打开相关服务。

```bash
systemctl start mysql
```

使用下面的命令访问进入mysql。

```bash
mysql -u root -p
```

首次进入passwd应该为空，无需输入直接Enter即可。

进去后，可以首先修改密码。(下面的命令属于sql，感兴趣的可以学习数据库相关知识)

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '[YourPassword]';
```

然后创建相关的数据库，这个数据库用于存放博客的相关数据。(如果要学习数据库知识，可以创建一些自己的数据库，练习create，insert，alter等其他操作)

```sql
create database wordpress;
```

创建完数据库后使用exit离开数据库。

```sql
exit;
```

### PHP

运行wordpress博客需要php，接下来安装php。（请注意php的版本一般不能低于7.2，如果你使用一些旧版本的操作系统，php默认安装版本过低，请前往php官网安装符合要求的版本）

```bash
sudo apt install php
```

为了测试php的相关安装情况，使用下面的命令创建一个网页输出phpinfo的信息。

```bash
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
```

然后访问你的ip地址后面加上/phpinfo.php去。如果出现了下面的页面，便说明了php成功安装，下面会有你的服务器的一些详细信息，这个页面可能造成一些安全隐患，建议在测试完后删去。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%201.png)

## Wordpress安装

前往wordpress的官网，[下载 | WordPress.org China 简体中文](https://cn.wordpress.org/download/)。

```bash
wget https://cn.wordpress.org/latest-zh_CN.tar.gz
```

解压文件。(下载的文件如果是tar.gz格式使用-zxvf，如果是别的格式请使用别的命令)

```bash
tar -zxvf [file-name]
```

解压完后将wordpress文件转移到/var/www/html目录下，改个名字

```bash
cp -r wordpress /var/www/html
cd /var/www/html
mv wordpress wp-blog
```

进入博客目录，复制配置文件使用。

```bash
cd wp-blog
cp wp-config-sample.php wp-config.php
```

依次输入下面的命令修改配置。

```bash
sed -i 's/database_name_here/wordpress/' /var/www/html/wp-blog/wp-config.php
sed -i 's/username_here/root/' /var/www/html/wp-blog/wp-config.php
sed -i 's/password_here/[之前设定的数据库密码]/' /var/www/html/wp-blog/wp-config.php
```

也可以使用vim或gedit来修改，取决于你的习惯。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%202.png)

我出现了mysqli插件没有安装的错误，在查询了网络上的资料后，执行了如下命令。

```bash
sudo apt install php-mysqli php-curl php-gd php-imagick php-json php-mbstring php-xml php-zip
systemctl restart apache2
```

然后进入[ip]/wp-blog/wp-admin/install.php进行安装配置，注意不要使用弱密码。

注册完成后，按照它的指示进行登录即可，访问的地址是[ip]/wp-blog/wp-login.php

那么，到这里wordpress就算安装完成了。

### ftp

wordpress更新需要ftp服务器。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%203.png)

进入wp-config.php配置文件进行配置，在最后一行加入

```bash
define("FS_METHOD","direct");
define("FS_CHMOD_DIR", 0777);
define("FS_CHMOD_FILE", 0777);
```

返回上级目录/var/www/html

```bash
chown -R www-data wp-blog
```

将 wp-blog 文件夹以及内容的所有者和所属用户组改为 www-data 。
这个语句的目的是更改从网页端操作的权限，使用更改所有者和所属用户组的方式而不是直接修改内容本身其他用户的权限主要是为了方便和安全。
此处原理涉及 Linux 权限、用户与用户组相关的知识，小白可以不用掌握，用就行了~
重启服务器，在命令行窗口输入：

```bash
systemctl restart apache2
```

现在便可以正常更新了，那么到这里wordpress算安装完毕了。

## Wordpress主题配置

我选择了sakura主题，二次元风格，还很好看。

[https://github.com/mirai-mamori/Sakurairo](https://github.com/mirai-mamori/Sakurairo)

进入github页面，建议阅读一下它的相关文档。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%204.png)

进入github release下载压缩包，或者直接下载也可以。

进入外观-主题。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%205.png)

点击安装主题，进入后点击上传，上传下载的zip压缩包。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%206.png)

如果上传失败了，显示大小超限，前往修改php.ini文件，这个文件一般在/etc/php/8.1/apache2/php.ini这个地址，可以考虑用vim进行修改。

需要同时修改upload_max_filesize，post_max_size（这个值要比upload的值大才行）和memory_limit（如果为-1无需修改）

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%207.png)

修改完后重启apache2服务即可实现上传。

```bash
systemctl restart apache2
```

然后回到主题页面选择启用。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%208.png)

安装后进入管理页面逐步配置即可。

![Untitled](Wordpress%E5%8D%9A%E5%AE%A2%E9%85%8D%E7%BD%AE%20bdee573fb0cf42a9960dd4c2b84210db/Untitled%209.png)