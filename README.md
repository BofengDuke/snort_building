# snort_building
---

## 搭建Ubuntu环境，用户名、密码全设置为 snort
    改root的密码，后面都基于root的权限执行（
    >sudo passwd root
## 安装lamp环境（是为了检测到数据可以可视化）,网上随便找个lamp意见安装包，比如wdlinux的lanmp（http://www.wdlinux.cn/bbs/thread-53213-1-1.html）
    >wget http://dl.wdlinux.cn/files/lanmp_v3.1.tar.gz
    >tar zxvf lanmp_v3.1.tar.gz
    >sh lanmp.sh                   默认安装
    >sh lanmp.sh cus             自定义安装
    安装好后,可以先查看你装的lamp包的使用说明，wdlinux(http://www.wdlinux.cn/wdcp/manual.html)
    打开各种服务apache，mysql，php，
    基于上面的wdlinux只需要打开apache服务
    >service apache2 start
    打开浏览器，输入 http://localhost/ 回车，出现apache的页面就成功了
    
    （可选）进入mysql改root的密码（默认 wdlinux.cn)
    创建一个用来保存snort监听并输出的日志信息的数据库
    顺便创建个snort的账户,赋予权限
    >mysql -u root -p
    >>wdlinux.cn
    >>set password for root@localhost = password('snort');
    >>create database snort;
    >>create user 'snort'@'localhost' identified by 'snort';
    >>grant all privileges on snort.* to snort@localhost identified by 'snort';
    >>flush privileges;
    >>quit;
## snort官网（https://www.snort.org）
## 方法一：snort官网下载安装方法（https://www.snort.org/#documents）
## 方法二：下载安装snort --基于apt-get
    >apt-get install snort snort-rules-default 
    安装过程会让你输入网卡，打开另外一个bash（终端），查看网卡
    >ifconfig
    此时测试用的网卡是ens33，回去原来的bash，按下tab键，回车确认就可以输入了
    配置完网卡，会让你配置网络
 ###安装完成，查看使用说明
    >snort -h
    尝试使用
    >snort
 ### 官网使用说明（https://www.snort.org/documents）
    建议看HTML版（http://manual-snort-org.s3-website-us-east-1.amazonaws.com/）
    snort基本直接使用方法，直接看官方文档第一章节
    或者百度一些博客比如：入侵检测技术――Snort（http://blog.sina.com.cn/s/blog_976123090100w299.html）
## snort扩展，让其支持数据库存储
## （可选）可以下载snort看看README文档
    >wget https://www.snort.org/downloads/snort/snort-2.9.9.0.tar.gz
    可参考文档：
        Snort:Barnyard2+MySQL+BASE 基于Ubuntu 14.04SNORT，snortbarnyard2（http://www.bkjia.com/xtzh/1013893.html）
        CentOS6.6下基于snort+barnyard2+base的入侵检测系统的搭建（http://wenku.baidu.com/link?url=U2-JYsXyJuh4N0ZiJ2oes_9mlmoZpcyKU0cicIibp7gHfw3ffkt1Z5bAfPUxK5jU89ymxXf0DSLrhG_Z2nUwh2dOAVeR03zLviAFMSuL-Cm）
## 准备工具
 ### 下载安装libpcap
    >apt-get install flex libdnet
    >wget http://www.tcpdump.org/release/libpcap-1.8.1.tar.gz 
    >tar -zxf libpcap-1.8.1.tar.gz
    >cd libpcap-1.8.1
    >./configure && make && make install
 ###下载安装daq
    >wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
    >tar -zxf daq-2.0.6.tar.gz
    >cd daq-2.0.6
    >./configure && make && make install
 
 ###下载安装Barnyard
    >wget https://github.com/firnsy/barnyard2/archive/v2-1.13.tar.gz -O barnyard2-2-1.13.tar.gz
    >tar zxf barnyard2-2-1.13.tar.gz
    >cd barnyard2-2-1.13
    >autoreconf -fvi -I ./m4
    >./configure --with-mysql --with-mysql-libraries=/usr/lib/$(uname -m)-linux-gnu --with-mysql-includes=/usr/include/
    >make && make install
    
    创建数据库表
    >mysql -u snort -p -D snort < /root/barnyard2-2-1.13/schemas/create_mysql
    
    创建所需要的文件和目录
    >mkdir /var/log/barnyard2
    >touch /var/log/snort/barnyard2.conf    
    >cp /root/barnyard2-2-1.13/etc/barnyard2.conf /etc/snort
    
    编辑配置文件
    >vim /etc/snort/barnyard2.conf
        config logdir:/var/log/barnyard2  
        config hostname:localhost  
        config interface:ens33  
        config waldo_file:/var/log/snort/barnyard.waldo   
        output database: log, mysql, user=snort password=snort dbname=snort host=localhost
    配置snort的规则：
    cp /etc/snort/community-sid-msg.map /etc/snort/sid-msg.map
    
    测试barnyad2:
    >barnyard2 -c /etc/snort/barnyard2.conf -d  /var/log/snort  -f  snort.log  -w  /var/log/snort/barnyard2.waldo
    
    如果报错有: database mysql_error: Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
    则执行以下命令,建立一个mysqld的链接
    >cd /var/run
    >mkdir mysqld
    >ln -s /tmp/mysql.sock /var/run/mysqld/mysqld.sock
    
    数据库支持到此结束
## 以网页可视化的形式浏览，基于base
    
## （可选）snort基本配置
    用vim或gedit打开snort的配置文件在 /etc/snort/snort.conf,,这里使用vim(需安装)
    >apt-get install vim
    >vim /etc/snort/snort.conf
    打开文件后，会看到配置文件里面写得很清楚，配置过程有9步，其中：
 ### 变量说明：
        有三个变量 var，portvar，ipvar
        官网使用例子：
        var RULES_PATH rules/
        portvar MY_PORTS [22,80,1024:1050]
        ipvar MY_NET [192.168.1.0/24,10.1.1.0/24]
        alert tcp any any -> $MY_NET $MY_PORTS (flags:S; msg:"SYN packet";)
        include $RULE_PATH/example.rule
        在最新版本中，不推荐使用var来设置IP地址了，但是会向老版本兼容
### 一、配置网络
 #### 一般的系统来说：
    在第一部分中，找到 ipvar HMOE_NET any (line 51)，将any改成自己的ip
        ipvar HOME_NET [192.168.117.0/24]   // 注意这里的ip是自己的，而可以用ifconfig查看
    把下面的两行 ipvar EXTERNAL_NET any 改成 $HOME_NET
        ipvar EXTERNAL_NET $HOME_NET
    在100行左右，可以定义自己的检测规则的文件路径（var RULE_PATH /etc/snnort/rules）
    
 #### debian（包括Ubuntu）系统配置
    在/etc/snort/目录下，有一个snort.debian.conf，在这里面配置网络信息，如果上面已经被配置了，会被这个文件的配置所取代（这个文件在最开始apt安装的时候就已经配置好，所以apt安装的话，可以不配置网络



