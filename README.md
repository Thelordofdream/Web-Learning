# Web-Learning
A learning guide and history of Web Technology mainly with html/css/JavaScript/mysql/PHP

##学习门槛
目前，由于WordPress等轻博客技术的兴起，个人从事简单的Web开发已经不是什么门槛很高的事情。但是，读者依然需要:  

* 比较熟练地掌握诸如bash等Linux Shell的基本指令，如cd、wget、curl、touch、mkdir、echo、rm等指令；
* 掌握一点点SSH的操作，诸如ssh连接、scp上传下载等指令；
* 掌握诸如vim、nano等其中一种编辑器的操作；  

由于在这个学习笔记中，笔者是想要通过html+css+js自主实现前端、通过mysql+PHP实现后台，因此如果读者也有这样的野心，则还需要： 
 
* 接触过html或者xml编程，了解一点基本语法；
* 熟悉java或者javascript的编程；
* 掌握mysql的基本语法及其提供的python或PHP接口；
* 具备一点基本的web架构常识；
* 有比较丰富的app或者应用开发经历，了解开发流程和界面布局；
* 熟悉Python或者PHP其中一种脚本编程语言；
* 当然，掌握一点Markdown来做笔记也是很好的；

因此，这份入门指南只是描述基本步骤，不会在细节上过多地展开。因此对毫无基础的读者会有一定难度，请读者根据自己的情况进行参考。

##购置服务器
随着云计算时代的到来，诸如AWS、阿里云、腾讯云等一众优秀的云服务提供商的出现使个人租用服务器以搭建网站变得方面而价格低廉。 
 
* AWS向大众提供一年免费试用，但是需要绑定一张VISA信用卡；  
* 阿里云向身份认证认证学生群体提供半年期低于10元每月的租用优惠；  
* 腾讯云近期针对学生群体提供1元租用活动，每天12点后有200名额抢购；  


笔者选择了阿里云，半年租用费用是56元，操作系统选择了Ubutun 14.04。在用户购买成功后，阿里云会发送包含服务器IP地址、root用户名信息的短信到绑定的手机上，用户需要在线或者通过阿里云app设置服务器登录密码后，通过ssh等方式登录服务器。登录后，对操作系统中的软件进行必要的更新：  

* apt-get update
* apt-get upgrade  

##远程桌面
因为传输带宽、网络延迟、操作有限等原因，在很多情况下远程桌面是不必要的，但是还是讲一下如何快速地搭建远程。目前有诸如VNC等几种优秀的远程桌面技术，这里讲解最容易实现的xrdp远程桌面（root帐号不需要sudo）：

* apt-get install xrdp
* apt-add-repository ppa:ubuntu-mate-dev/ppa
* apt-add-repository ppa:ubuntu-mate-dev/trusty-mate
* apt-get update 
* apt-get upgrade
* apt-get install ubuntu-mate-core ubuntu-mate-desktop
* echo mate-session >~/.xsession
* service xrdp restart  

以上步骤是安装了一个xrdp远程桌面工具和mate图形桌面。当然我们还需要设置服务器开机启动远程桌面工具：  

* nano /etc/rc.local
* 在文件中`exit 0`前添加`/etc/xrdp/xrdp.sh start`  

然后我们需要在本地访问服务器远程桌面的软件：

* Mac端使用 Microsoft Remote Desktop，由于该软件是传输整帧桌面，延迟比较高；
* Windows端自带有远程桌面工具，在开始菜单可以找到，由于该软件是差量传输，所以延迟比较低，性能更佳；

在软件中配置好服务器信息，其中xrdp的默认端口号是3389，而不是ssh的默认端口22，因此软件中应填写端口号为3389。配置完成后即可进行连接，如果出现连接问题，在本地Shell中重启xrdp即可。

## 配置VPN
在服务器上运行VPN也不是必要的，但是由于有时候会使用到pip工具，还是给读者普及一下。在Shell中翻墙，需要Shadowsocks和Proxychains这两个工具：  

* apt-get install python-pip
* apt-get install python-setuptools m2crypto
* pip install shadowsocks
* 在服务器某文件夹下，touch shadowsocks.json
* nano shadowsocks.json  
```
{
"server":"11.22.33.44", 
"server_port":50003,
"local_port":1080,
"password":"123456",
"timeout":600,
"method":"aes-256-cfb"
}
```

最后一步的信息根据读者拥有的VPS自行设定，以上范例是无效的无需浪费时间尝试。之后就是以此配置运行Shadowsocks：  

* sslocal -c /(path)/shadowsocks.json

其中这一步可能会报错，显示读者的服务器上没有libsodium这个软件，但是下载libsodium-1.0.11又需要翻墙，因此读者需在本地下载后上传服务器进行安装。如配置信息有效则上面语句运行后，Shadowsocks则已成功运行，接下来我们设置Shadowsocks开机启动：  

* apt-get install supervisor
* nano /etc/supervisor/supervisor.conf
* 在文件最末尾添加
```
[program:shadowsocks]
command=sslocal -c /home/mudao/shadowsocks.json
autostart=true
autorestart=true
user=root
log_stderr=true
logfile=/var/log/shadowsocks.log
```  

* cp /usr/local/bin/sslocal /bin
* service supervisor restart 
* nano /etc/rc.local
* 在文件中`exit 0`前添加
`service supervisor start`

接下来安装Proxychains-ng，使Shell指令可以走代理：  

* git clone https://github.com/rofl0r/proxychains-ng.git
* cd proxychains-ng
* ./configure
* make && make install
* cp ./src/proxychains.conf /etc/proxychians.conf
* cd .. && rm -rf proxychains-ng
* nano /etc/proxychains.conf
* 修改文件最末尾一行为
`socks5  127.0.0.1 1080`

如果Shadowsocks已经正确运行，则可运行Proxychains如下：  

* proxychains+空格+需要走代理的指令，如pip

##搭建Web环境
下面要搭建的是Ubuntu LAMP Web环境，LAMP分别是指：  

* Linux
* Apache
* MySQL
* PHP

接下来开始搭建：  

* apt-get install apache2
* 打开浏览器输入http://localhost/，测试apache是否安装成功
* apt-get install php5
* apt-get install php5-curl
* /etc/init.d/apache2 restart
* apt-get install mysql-server mysql-client
* apt-get install php5-mysql
* cd /var/www/ 此目录也是Apache2的默认文件目录
* wget https://files.phpmyadmin.net/phpMyAdmin/3.5.2.2/phpMyAdmin-3.5.2.2-all-languages.tar.gz
* tar zxvf phpMyAdmin-3.5.2.2-all-languages.tar.gz
* cd phpMyAdmin-3.5.2.2-all-languages
* cp config.sample.inc.php config.inc.php
* 访问http://localhost/phpmyadmin，输入mysql的用户名和密码进入

如果上述步骤均已成功完成，则在本地浏览器输入服务器的IP地址，则会显示Apache2的默认网页index.html，此文件就在/var/www/html中。读者将html文件放置在此文件夹中，则公网用户通过服务器IP+文件名.html即可访问该网页。此外，Apache2是开机自启动的，不需要特别设置。

##学习HTML
这里给一个内容基础的学习网站：  
<http://www.runoob.com/html/html-tutorial.html>
##学习CSS
这里给一个CSS的指南类网站：  
<http://cssreference.io/#align-content>
##学习JavaScript
JavaScript的语法类似于Java因此并不难学，这里给一个学习目前最强大的新兴3D建模及渲染技术Three.js的学习网站：  
<http://www.hewebgl.com/article/articledir/1>
##Web开发IDE
最推荐的当然还是JetBrains的WebStorm，除了一般的颜色提示外，此IDE还能提示开发者所使用的语句适配哪些市场上的浏览器。当然，Web开发可以完全不依赖于IDE，只用文本编辑功能也能实现简单的网页。
##几点忠告
* 读者所编写的网页，在移动端和PC端浏览器上显示的效果是不一样；因此要实现两种平台同时适配，需要读者额外付出努力，且比较繁复艰难；初学阶段可以不考虑适配移动端。
* 诸如360、QQ浏览器等以IE浏览器为内核的浏览器对js动画的支持非常有限，因此读者写的很多视觉动画都不能在以上浏览器中很好地展现。
* 目前，以webkit为内核的Safari和以Chromium为内核的Chrome存在一些差异，需要读者注意。
* 好的网页不仅仅依靠开发者的编程水平、强大的后台技术，也依赖于网页设计、美工素材和交互逻辑；为了技术而技术，是技术的死胡同。
* 要多学习借鉴，尽量避免造轮子。

以上是个人的一点浅见。  
Web development is an exciting activity，a little tough but very creative ，so I wish you will love it.