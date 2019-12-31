# MyCAT安装指南

> MyCAT 1.2版本

## 快速上手-安装指南（安装单机）

Mycat的server和mysql位于同一台服务器，centos6.2.4环境

Mycat:10.191.116.175

Mysql:10.191.116.175

是用Java开发，需要有JAVA运行环境，mycat依赖jdk1.7的环境，若本机没有，则需要下载安装：

[[http://www.java.com/zh\_CN/]{.underline}](http://www.java.com/zh_CN/)

获取**MyCAT的**最新开源版本，项目主页[[http://code.google.com/p/MyCAT/]{.underline}](http://code.google.com/p/opencloudb/)

目前最新代码暂时在淘宝上托管，二进制包下载地址：

[[http://code.taobao.org/svn/openclouddb/downloads/]{.underline}](http://code.taobao.org/svn/openclouddb/downloads/)

windows下可以下载Mycat-server-xxxx.ZIP，linux下可以下载tar.gz解开在某个目录下，注意，目录不能有空格，在Linux(Unix)下，建议放在/usr/local/MyCAT目录下，如下面类似的：

![OW)T8\@W\~VX(S\]ZI26L594CJ](media/image1.jpeg){width="3.1979166666666665in" height="1.8125in"}

下面是修改MyCat用户的密码方式（仅供参考）

![P\`}59C\_\`FRNG8CXSZABD\`\_Q](media/image2.jpeg){width="4.854166666666667in" height="0.8958333333333334in"}

![](media/image3.png){width="1.28125in" height="1.34375in"}

目录解释如下：

Bin 程序目录，存放了window版本和linux版本，除了提供封装成服务的版本之外，也提供了nowrap的shell脚本命令，方便大家选择和修改，进入到bin目录：

-   Windows 下 运行: mycat.bat console 在控制台启动程序，也可以装载成服务，若此程序运行有问题，也可以运行startup\_nowrap.bat，确保java命令可以在命令执行。

-   Linux下运行：nohup sh mycat console &,首先要chmod +x mycat

Warp方式的命令，可以安装成服务并启动或停止。

-   mycat install (可选)

-   mycat start

**注意，wrap方式的程序，其JVM配置参数在conf/wrap.conf中，可以修改为合适的参数，参数调整参照[[http://wrapper.tanukisoftware.com/doc/english/properties.html]{.underline}](http://wrapper.tanukisoftware.com/doc/english/properties.html)。用下面是一段实例：**

**注：mycat必须依赖jdk1.7，在1.6的情景下会报错，如果机器未升级可以指定jdk的目录，我考了一个jdk的包出来的，添加的绝对路径，根据情况定。**

**wrapper.java.command=/usr/local/Mycat/jdk1.7.0/bin/java**

**\# Java Additional Parameters**

**wrapper.java.additional.5=-XX:MaxDirectMemorySize=2G**

**wrapper.java.additional.6=-Dcom.sun.management.jmxremote**

**\# Initial Java Heap Size (in MB)**

**wrapper.java.initmemory=2048**

**\# Maximum Java Heap Size (in MB)**

**wrapper.java.maxmemory=2048**

**若启动报内存不够，可以试着将上述内存都改小，改为1G或500M。**

Conf目录下存放配置文件，server.xml是Mycat服务器参数调整和用户授权的配置文件，schema.xml是逻辑库定义和表以及分片定义的配置文件，rule.xml是分片规则的配置文件，分片规则的具体一些参数信息单独存放为文件，也在这个目录下，配置文件修改，需要重启Mycat或者通过9066端口reload。

日志存放在logs/mycat.log中，每天一个文件，日志的配置是在conf/log4j.xml中，根据自己的需要，可以调整输出级别为debug，debug级别下，会输出更多的信息，方便排查问题。

建议本地有一个Mysql Server，若没有，建议安装一个，下载地址：

[[http://dev.mysql.com/downloads/mysql/5.5.html\#downloads]{.underline}](http://dev.mysql.com/downloads/mysql/5.5.html#downloads)

启动Mysql，确保能正常登录访问数据，msyql命令行工具mysql\\bin\\mysql.exe建议加入PATH路径中，为了方便使用。

Service mysql start

用命令行工具或图形化客户端，连接MYSQL，创建DEMO所用三个分片数据库；

CREATE database db1;

CREATE database db2;

CREATE database db3;

注意：若是LINUX版本的MYSQL，则需要设置为Mysql大小写不敏感，否则可能会发生表找不到的问题。

**在MySQL的配置文件中my.ini \[mysqld\] 中增加一行**

**　　lower\_case\_table\_names = 1**

编辑MYCAT\_HOME/conf/schema.xml文件，修改dataHost对应的连接信息：

![](media/image4.png){width="6.0in" height="1.78125in"}

**注意writeHost/readHost中的location,user,password的值符合你所采用的Mysql的连接信息。**

修改完成后保存，进入到MYCAT\_HOME/bin目录下，执行启动命令：startup.bat ，启动成功以后显示如下信息：

![](media/image5.png){width="6.0in" height="1.2291666666666667in"}

注意，默认数据端口为8066，管理端口为9066。

客户端也可以用图形化的客户端如：mysqlworkbench、 navicat 、以及一些基于Java的数据库客户端来访问，注意要填写端口号8066，以及database 为TESTDB。

命令行运行：mysql -utest -ptest -h127.0.0.1 -P8066 -DTESTDB 就能访问OpenCloudDB了，**以下操作都在此命令行里执行**（JDBC则将mysql的URL中的端口3306改为8066即可）

**提示：访问MyCAT的用户账号和授权信息是在conf/server.xml文件中配置，而MyCAT用来连接后端MySQL库的用户名密码信息则在conf/schema.xml中，这是两套完全独立的系统，类似的还有MyCAT的逻辑库(schema)，逻辑表（table）也是类似的。**

Employee表，是根据规则sharding-by-intfile （分片字段为sharding\_id）进行分片。创建employee表：输入如下SQL

create table employee (id int not null primary key,name varchar(100),sharding\_id int not null);

运行explain指令，查看该SQL被发往哪些分片节点执行：

explain create table employee (id int not null primary key,name varchar(100),sharding\_id int not null);

![](media/image6.png){width="6.0in" height="0.71875in"}

建议参照schema.xml中employee表的定义，以及其分片规则，来看看什么数据会出现在dn1上，什么数据会出现在dn2上。

**温馨提示：explain可以用于任何正确的SQL上，其作用是告诉你，这条SQL会路由到哪些分片节点上执行，这对于诊断分片相关的问题很有帮助。另外，explain可以安全的执行多次，它仅仅是告诉你SQL的路由分片，而不会执行该SQL。**

插入数据：

insert into employee(id,name,sharding\_id) values(1,\'leader us\',10000);

insert into employee(id,name,sharding\_id) values(2, \'me\',10010);

insert into employee(id,name,sharding\_id) values(3, \'mycat\',10000);

insert into employee(id,name,sharding\_id) values(4, \'mydog\',10010);

company表是根据规则auto-sharding-long（主键范围）进行分片。创建company表：输入如下SQL

create table company(id int not null primary key,name varchar(100));

录入数据：

insert into company(id,name) values(1,\'hp\');

insert into company(id,name) values(2,\'ibm\');

insert into company(id,name) values(3,\'oracle\');

你会看到三个分片上都插入了3条数据，因为company定义为全局表，用explain来确认这个情况：

explain insert into company(id,name) values(1,\'hp\')

返回3个节点的信息：

\| DATA\_NODE \| SQL \|

+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+

\| dn1 \| insert into company(id,name) values(1,\'hp\') \|

\| dn2 \| insert into company(id,name) values(1,\'hp\') \|

\| dn3 \| insert into company(id,name) values(1,\'hp\') \|

+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+

创建客户表：

create customer: create table customer(id int not null primary key,name varchar(100),company\_id int not null,sharding\_id int not null);

插入数据：

insert into customer (id,name,company\_id,sharding\_id )values(1,\'wang\',1,10000); //stored in db1;

insert into customer (id,name,company\_id,sharding\_id )values(2,\'xue\',2,10010); //stored in db2;

insert into customer (id,name,company\_id,sharding\_id )values(3,\'feng\',3,10000); //stored in db1;

查询结果：

Select \* from customer;

explain Select \* from customer; （确认数据是分片存储）

创建表格orders，并插入数据：

create table orders (id int not null primary key ,customer\_id int not null,sataus int ,note varchar(100) );

insert into orders(id,customer\_id) values(1,1); //stored in db1 because customer table with id=1 stored in db1

insert into orders(id,customer\_id) values(2,2); //stored in db2 because customer table with id=1 stored in db2

explain insert into orders(id,customer\_id) values(2,2);

select customer.name ,orders.\* from customer ,orders where customer.id=orders.customer\_id;

travelrecord根据ID主键的范围进行分片：

create travelrecord: create table travelrecord (id bigint not null primary key,user\_id varchar(100),traveldate DATE, fee decimal,days int);

insert into travelrecord (id,user\_id,traveldate,fee,days) values(1,\'wang\',\'2014-01-05\',510.5,3);这个ID就存放在分片0上了

![](media/image7.png){width="6.0in" height="0.7291666666666666in"}

explain insert into travelrecord (id,user\_id,traveldate,fee,days) values(7000001,\'wang\',\'2014-01-05\',510.5,3); 这个ID就存放在分片1上了

![](media/image8.png){width="6.0in" height="1.4166666666666667in"}

看到支持跨分片的JOIN！

热点新闻，用取摸的方式随机分配到dn1,dn2,dn3上

create table hotnews(id int not null primary key ,title varchar(400) ,created\_time datetime);

插入数据

insert into hotnews(id,title,created\_time) values(1,\'first\',now()); 在分片1上

![](media/image9.png){width="6.0in" height="1.0625in"}

而Id为5，则到dn3上，5%3=2 ,即对应dn3的 index

![](media/image10.png){width="6.0in" height="1.3020833333333333in"}

其他：

goods表

create table goods(id int not null primary key,name varchar(200),good\_type tinyint,good\_img\_url varchar(200),good\_created date,good\_desc varchar(500), price double);一起探索MyCAT的奇妙新世界吧！ QQ群：106088787
