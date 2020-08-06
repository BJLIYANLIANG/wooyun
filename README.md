# Ubuntu 18.04下搭建WooYun漏洞库

## 前言

自从wooyun关闭以后，网络安全人员缺少了一个学习平台，作为汇聚了大量前辈知识结晶的wooyun漏洞库、知识库对于搞安全行业的人员还是有很大的学习价值；目前互联网上也有很多人搭建了自己的wooyun平台，鉴于平台的稳定性以及后续使用的便利性，还是建议各位小伙伴自行搭建wooyun知识平台，本文主要以个人亲身经历为背景，基本ubuntu 18.04平台撰写了搭建手册，仅供有需要的小伙伴参考；

## 一、ubuntu系统下设置SSH密钥登陆步骤（可选项）

注：之所以使用密钥登陆，主要是为了提高安全性以及登陆的便捷性，可以根据实际情况选择紫荆喜欢的登陆方式；

### 1、首先登陆Ubuntu 18.04 生成密钥

- 我们先要生成一个密钥对，也就是公钥和私钥。生成密钥对需要**linux**操作系统。所以如果你在客户机上生成密钥对，你需要把公钥上传到服务器上。如果你在服务器上生成密钥对，你需要把私钥下载到客户机上。本人的客户机是**Windows10**，所以我选择在服务器上生成密钥对。
- 先通过ssh密码登陆服务器，然后：

```C++
ssh-keyge
    
Enter file in which to save the key (/root/.ssh/id_rsa):
//这句话的意思是你希望把生成的密钥对放在哪个位置，默认的是：/root/.ssh/id_rsa 默认就好，所以我们这里直接回车就好。

Enter passphrase (empty for no passphrase):
//这句话的意思是是否设置双重认证，就是为你的私钥添加一个密码，如果图方便，直接回车就好，如果为了更加安全，设置个简单密码就好。
//如果设置了双重认证，需要再一次确认密码
Your identification has been saved in /root/.ssh/id_rsa.
    
Your public key has been saved in /root/.ssh/id_rsa.pub.
    
The key fingerprint is:
aa:8b:61:13:38:ad:b5:49:ca:51:45:b9:77:e1:97:e1 root@localhost.localdomain
The key's randomart image is:
+--[ RSA 2048]----+
|    .o.          |
|    ..   . .     |
|   .  . . o o    |
| o.  . . o E     |
|o.=   . S .      |
|.*.+   .         |
|o.*   .          |
| . + .           |
|  . o.           |
+-----------------+
//完成
```

### 2、把密钥下载到客户机

![这里写图片描述](https://img-blog.csdn.net/20180817143804777?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYWRhbmRlYWppYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
**id_rsa** 是私钥 **id_rsa.pub**是公钥。我们只需要右键私钥，点击下载就好了。
这样，我们的客户机上就有钥匙了。后续就可以使用xshell使用密钥登陆设备了；

### 3、Ubuntu 18.04启用SSH密钥 禁用密码

```C++
:cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys //这一步是把生成的公钥导入到ssh的配置文件中

chmod 600 authorized_keys //以防万一，我们给予文件更高的权限
chmod 700 /root/.ssh

vim /etc/ssh/sshd_config //打开ssh配置文件
    
找到PubkeyAuthentication（在第37行）,默认的话，是被注释的，并且为no，我们把注释去掉，并且改为yes //开启密钥登陆
    
找到PasswordAuthentication（在第56行）,默认的话，是被注释的，并且为yes，我们把注释去掉，并且改为no //关闭密码登陆
    
service sshd restart //重启ssh服务

```

如图所示：
![这里写图片描述](https://img-blog.csdn.net/20180817144822877?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYWRhbmRlYWppYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



## 二、搭建LAMP环境

### 1、安装Apache

`apt install apache2 -y`

![img](https://img-blog.csdnimg.cn/20190818145510466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

检查是否开启Apache，一般安装完会默认开启。

`systemctl status apache2`

![img](https://img-blog.csdnimg.cn/20190818145510467.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

开启 、关闭和重启Apache服务器

```bash
systemctl start apache2      # 开启

systemctl stop apache2       # 关闭

systemctl restart apache2    # 重启
```

访问你的 Web 服务器，打开浏览器并输入Ubuntu18.04的IP地址，不出意外访问到Apache的默认信息页面。

![img](https://img-blog.csdnimg.cn/20190818145508670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

### 2、数据库的安装，这里安装MySQL5.7

![img](https://img-blog.csdnimg.cn/20190818145508180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

查看有没有安装MySQL：

`dpkg -l | grep mysql`

安装MySQL：

`apt install mysql-server -y`

![img](https://img-blog.csdnimg.cn/20190818145509532.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

安装完成之后可以使用如下命令来检查是否安装成功，如果看到有 mysql 的 socket 处于 LISTEN 状态则表示安装成功。

`netstat -tap | grep mysql`

![img](https://img-blog.csdnimg.cn/20190818145510470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

登录mysql数据库可以通过如下命令：

`mysql -u root -p`

> -u 表示选择登陆的用户名， -p 表示登陆的用户密码，全新安装的mysql数据库是没有密码的，Enter password: 处直接回车，就能够进入mysql数据库。然后通过 show databases; 就可以查看当前的所有数据库。

![img](https://img-blog.csdnimg.cn/20190818145508725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

### 3、PHP的安装

PHP添加了支持动态网页的服务器端网页处理。

`apt install php -y`

![img](https://img-blog.csdnimg.cn/20190818145510622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

安装完成后，使用如下命令查看PHP的版本：php -v

![img](https://img-blog.csdnimg.cn/20190818145507879.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

### 4、phpMyAdmin 的安装（可选，主要是为了方便操作数据库，如比较倾向于命令行操作，可以不安装）

`apt install phpmyadmin -y`

![img](https://img-blog.csdnimg.cn/20190818145510210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

选择 Apache2 并点击确定。（以下的选择用TAB键）

![img](https://img-blog.csdnimg.cn/20190818145507389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

点击确定来配置 phpMyAdmin 管理的数据库。

![img](https://img-blog.csdnimg.cn/20190818145508681.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

指定 phpMyAdmin 向数据库服务器注册时所用的密码。（由于mysql初始化安装以后是没有设置密码的，这里可以不写或者写准备作为作为mysql的密码）

![img](https://img-blog.csdnimg.cn/20190818145507561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

再次确认密码。

![img](https://img-blog.csdnimg.cn/20190818145509481.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

出现如下图所示，就表示phpMyAdmin安装完毕了。

![img](https://img-blog.csdnimg.cn/20190818145508435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

安装完成后，创建phpMyAdmin的软链接到Apache的根目录下（我的是/var/www/html/)

```
ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```

现在开始尝试访问phpMyAdmin，打开浏览器并输入：IP地址/phpmyadmin

![img](https://img-blog.csdnimg.cn/20190818145506773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

​    phpMyAdmin 是一个以PHP为基础，以Web-Base方式架构在网站主机上的MySQL的数据库管理工具，让管理者可用Web接口管理MySQL数据库。借由此Web接口可以成为一个简易方式输入繁杂SQL语法的较佳途径，尤其要处理大量资料的汇入及汇出更为方便。其中一个更大的优势在于由于phpMyAdmin跟其他PHP程式一样在网页服务器上执行，但是您可以在任何地方使用这些程式产生的HTML页面，也就是于远端管理MySQL数据库，方便的建立、修改、删除数据库及资料表。也可借由phpMyAdmin建立常用的php语法，方便编写网页时所需要的SQL语法正确性。

### 5、针对首次登录mysql未设置密码或忘记密码解决方法

1：首先输入以下指令：

```
sudo cat /etc/mysql/debian.cnf
```

运行截图如下：

![img](https://img-blog.csdn.net/20180831144759364?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM3OTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2： 再输入以下指令：

```
mysql -u debian-sys-maint -p

//注意! 
//这条指令的密码输入是输入第一条指令获得的信息中的 password = ZCt7QB7d8O3rFKQZ 得来。
//请根据自己的实际情况填写！
```

运行截图如下：**(注意! 这步的密码输入的是 ZCt7QB7d8O3rFKQZ，密码是由第一条指令获得的信息中的**

**password = ZCt7QB7d8O3rFKQZ 得来，每个人不一样，请根据自己的实际情况输入，输入就可以得到以下运行情况）**

![img](https://img-blog.csdn.net/20180717234206251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM3OTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3. 修改密码，本篇文章将密码修改成 root , 用户可自行定义。

```
use mysql;

update mysql.user set authentication_string=password('root') where user='root' and Host ='localhost';

update user set plugin="mysql_native_password"; 

flush privileges;
quit;
```

![img](https://img-blog.csdn.net/20180717234900348?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM3OTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

4： 重新启动mysql:

```
sudo service mysql restart

mysql -u root -p // 启动后输入已经修改好的密码：root
```

![img](https://img-blog.csdn.net/20180717235251402?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM3OTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180717235329735?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM3OTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

5：至此，mysql初始化完成；

## 三、搭建WooYun漏洞库

### 1、下载资源

访问https://github.com/grt1st/wooyun_search，下载所需文件。

![image-20200806170748914](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200806170748914.png)

使用git克隆代码(文件名为：wooyun_search-master)，其中文件bugs和drops内容为空，需另外下载，下载链接如下所示：

> | bugs   链接: http://pan.baidu.com/s/1bpC8wkn 密码: q88g(9.25更新) |      |      |
> | ------------------------------------------------------------ | ---- | ---- |
> | drops  链接：http://pan.baidu.com/s/1i5Q8L3f 密码：6apj      |      |      |

![](C:\Users\Administrator\Desktop\Snipaste_2020-08-06_16-47-18.png)

下载好WooYun_Bugs(漏洞库)、WooYun_Drops(知识库)之后，需对里面的压缩文件进行解压。

注意：只需分别对文件WooYun_Bugs(漏洞库)、WooYun_Drops(知识库)中的一个压缩文件进行彻底解压即可。解压成功后，分别将这两个文件中的内容放到对应的bugs、drops文件中。

![img](https://img-blog.csdnimg.cn/20190818184356674.png)

![img](https://img-blog.csdnimg.cn/20190818184357364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

接下来就是将文件wooyun_search-master放进先前搭建好的web服务器目录下(/var/www/html)，并将文件wooyun_search-master重命名为wooyun，方便后面进行搜索。

![image-20200806164907846](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200806164907846.png)

```bash
文件说明：

	app_bugs.py                      bugs的索引，依赖lxml

	app_drops.py                     drops的索引，依赖lxml

	index.html                       搜索的主页

	search.php                       执行搜索的页面

	config.php                       php配置文件

	./bugs                           bugs静态文件的目录

	./drops                          drops静态文件的目录
```

> app_bugs.py为建立bugs索引的脚本，app_drops.py为建立drops索引的脚本。
>
> 因为python脚本中open()函数打开的文件名不能为中文，建议将drops目录下的中文文件名改为英文(例如，安全运维-xxxx.html=>safe-xxxx.html)

### 2、安装依赖组件

> - python 2.7和pip
> - python依赖：MySQLdb，lxml(推荐)

提示：以下操作均在root权限下进行（每个人搭建过程中遇到的问题会不一样，本文不保证能解决所有问题，个别问题需自己去手动查搜索。）

**2.1 安装Python**

```
apt install python
```

**2.2 安装pip**

![img](https://img-blog.csdnimg.cn/20190818184358497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

安装时若出现询问框，按默认的安装就行。安装完成后，可输入如下命令进行版本查询。

![img](https://img-blog.csdnimg.cn/20190818184355808.png)

**2.3 安装MySQLdb**

> MySQLdb — Python 连接 MySQL 的模块。

pip install mysql-python   

![img](https://img-blog.csdnimg.cn/20190818184357549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

使用pip安装，若出现以上异常 — EnvironmentError: mysql_config not found，需安装另一个依赖 apt-get install libmysqld-dev

![img](https://img-blog.csdnimg.cn/20190818184358361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

再次安装pip install mysql-python，安装成功之后，进入到python环境确认— import MySQLdb

![img](https://img-blog.csdnimg.cn/20190818184358125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190818184355158.png)

**2.4 安装lxml**

> lxml是python的一个解析库，支持HTML和XML的解析，支持XPath解析方式，而且解析效率非常高。

安装依赖 — apt-get install libxml2-dev libxslt-dev

![img](https://img-blog.csdnimg.cn/20190818184357705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

安装lxml —— apt-get install python-lxml  或  pip install lxml

![img](https://img-blog.csdnimg.cn/20190818184357824.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

### 3、索引配置

**3.1 修改py文件**

打开终端，切到/var/www/html/wooyun目录下，分别在脚本app_bugs.py、app_drops.py中修改如下语句，更改参数如主机、端口号、用户名、密码(将光标定位到需要更改的位置，输入i，即可进行添加)。

![img](https://img-blog.csdnimg.cn/20190818184355476.png)

![img](https://img-blog.csdnimg.cn/20190818184356522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

**3.2 创建数据库、表**

在mysql中建立数据库wooyun，数据表bugs、drops，分别建立字段title、dates、author、type、corp、doc与title、dates、author、type、doc。

```sql
CREATE DATABASE `wooyun` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

create table bugs(title VARCHAR(500),dates DATETIME, author CHAR(255),type CHAR(255),corp CHAR(255),doc VARCHAR(200) PRIMARY KEY);

create table drops(title VARCHAR(500),dates DATETIME, author CHAR(255),type CHAR(255),doc VARCHAR(200) PRIMARY KEY);
```

注意mysql编码如下，需要为utf-8（character_set_server不为utf-8要修改mysql配置文件）

```sql
use wooyun;

show variables like 'character%'; #查看编码

+--------------------------+----------------------------+



| Variable_name            | Value                      |



+--------------------------+----------------------------+



| character_set_client     | utf8                       |



| character_set_connection | utf8                       |



| character_set_database   | utf8                       |



| character_set_filesystem | binary                     |



| character_set_results    | utf8                       |



| character_set_server     | utf8                       |



| character_set_system     | utf8                       |



| character_sets_dir       | /usr/share/mysql/charsets/ |



+--------------------------+----------------------------+
```

以下是修改character_set_server的编码的方法。修改完之后，记得重启mysql服务。

![img](https://img-blog.csdnimg.cn/20190818184355646.png)

![img](https://img-blog.csdnimg.cn/20190818184355633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

**3.3 建立索引**

切换到/var/www/html/wooyun目录下，分别执行app_bugs.py、app_drops.py文件

```bash
python ./app_bugs.py

python ./app_drops.py
```

执行完后在mysql中进行查询，bugs数目为40280，drops数目为1264

```sql
use wooyun;

select count(*) from bugs;

select count(*) from drops;
```

在浏览器中输入"Web服务器http://ip/wooyun"，便可以进入到WooYun查询页面。

![img](https://img-blog.csdnimg.cn/2019082310275029.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190823103824337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYyNTU3Nw==,size_16,color_FFFFFF,t_70)