---
layout:     post
title:      私人专线PROXY的搭建
subtitle:   使用google cloud设置socks5,http/https
date:       2017-12-07
author:     Oscar Zhang
header-style: text
catalog:      true
tags:
    - IT
    - Proxy
    - Shocks5
---
# 一、创建google cloud账号 
该填啥填啥，需要绑定一张信用卡     
![img](/img/in-posts/post-proxy/af4e49f529b65c957316cf26277c93be.png)

创建VM实例（做代理最小型机器就可以）    
![img](/img/in-posts/post-proxy/62f3f5cfa805516e8b8765db7be2bb08.png)

防火墙设置      
![img](/img/in-posts/post-proxy/c82558bcfb0afd87c725feb5ed1b9b3f.png)      
![img](/img/in-posts/post-proxy/56cf67e4b1988f8f3fb3efda695cb2bc.png)

# 二、SOCKS5       
![img](/img/in-posts/post-proxy/1403aa78e50c4d13b910a48da8068227.png)     
![img](/img/in-posts/post-proxy/1030a9162fd97f5e61f5345858a83e40.png)

#### 手动设置SS服务器    

    sudo apt-get update  
    sudo apt-get install python-pip     
    sudo pip install shadowsocks     
    sudo ssserver -p id -k pwd -m aes-256-cfb -d start  #id、pwd自行设置     

在/etc/rc.local中正确设置开机启动：   
1./etc/rc.local中不需要sudo(默认root); 2.需要写全路径，可通过which ssserver查看

    /usr/local/bin/ssserver -p 8388 -k mypassword -m aes-256-cfb -d start  

ps. 输出诊断信息到log，写在执行语句之前，不要盲目修改/etc/rc.local：

    exec 2> /tmp/rc.local.log      # send stderr from rc.local to a log file
    exec 1>&2                      # send stdout to the same log file
    set -x                         # tell sh to display commands before execution

在/etc/crontab中设置系统定时重启，增加ssserver稳定性：

    30 22 * * 1,4 root sleep 70 || /sbin/reboot          # 照猫画虎，注意时区换算(uptime, last reboot, 服务器22:3即北京6:30, 每周1,4) 

#### 使用[大神脚本](https://teddysun.com/486.html)设置SS服务器

    wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
    chmod +x shadowsocks-all.sh
    sudo ./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log

在客户端设置ss连接

#### 在windows中的配置客户端
![img](/img/in-posts/post-proxy/91d7e986ce6f422b38da130de2c723b6.png)   
该怎么连就怎么连   
Enjoy！   

# 三、HTTP/HTTPS

配置相关软件并编辑

    sudo apt-get install squid3     
    sudo vim /etc/squid3/squid.conf
            
对文件做相应修改

                    --- 文件内容修改部分 ---
    # http_access deny all          # 位于1060行, 注视掉
    http_port 10086                 # 再翻几页，个性化端口id
                    --- 文件末尾添加部分 ---
    acl allcomputers src 0.0.0.0/0.0.0.0
    auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/passwords
    auth_param basic realm proxy
    acl authenticated proxy_auth REQUIRED
    http_access allow authenticated allcomputers
    
设置用户名密码认证
 
    sudo apt-get install apache2-utils
    sudo htpasswd -c -d /etc/squid3/passwords <自定义用户名>
    > 设置密码，连输两次 <
    sudo chmod o+r /etc/squid3/passwords
    sudo service squid3 restart                 # or start, stop
    
验证代理是否起作用

    tail -f /var/log/squid3/access.log
    > 另台linux机器打开shell <
    export http_proxy="http://用户名:密码@代理IP:代理端口"
    curl -l "http://www.baidu.com"
    
如果代理配置正确，回输出html，同时代理服务器上的access.log会记录这次请求

#### [高匿设置](http://hoyoung.net/2017/02/10/squid3-proxy/)













