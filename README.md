# mycat 1.6.6 读写分离


------

由于工作需要，最近测试下mycat读写分离的实验，同时发现最新的mysql-8.0.15版本mycat1.6.6老是失败，不知道为什么。
**实验环境**

| 主机      | ip     | 角色    | mysql |
| :----------- | -----------------: | ---------------------: | ---------: |
| mysql-01     | 192.168.178.61   |  mycat/master |  mysql-5.7.25 |
| mysql-02     | 192.168.178.62   |  slave  |  mysql-5.7.25 |

## 在master 机器上下载安装 [mycat](http://dl.mycat.io),由于mycat 是Java写的，所以还需要安装java-openjdk-1.7 以上。

```
# wget http://dl.mycat.io/1.6.6.1/Mycat-server-1.6.6.1-release-20181031195535-linux.tar.gz
# tar -zvxf Mycat-server-1.6.6.1-release-20181031195535-linux.tar.gz
# cat /etc/profile.d/mycat.sh 
  export PATH=$PATH:/root/mycat/bin
# source /etc/profile.d/mycat.sh 
```
## 配置mycat ，mycat 主要配置文件说明如下：


| 文件名称 | 文件说明 | 
| ------- | ------- |
| server.xml | mycat 用户，连接等配置 |
| schema.xml | mycat 逻辑库 表 分片 规则 |
| rule.xml | mycat 分片的规则算法 |

**本实验配置如下：**
```
# cat  /root/mycat/conf/schema.xml 
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">


        <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100" >
               <table name="ff" dataNode="dn1,dn2,dn3" rule="crc32slot" />
               <table name="greg" dataNode="dn8" rule="crc32slot" />
            
        </schema>
        <dataNode name="dn1" dataHost="localhost1" database="db1" />
        <dataNode name="dn2" dataHost="localhost1" database="db2" />
        <dataNode name="dn3" dataHost="localhost1" database="db3" />
        <dataNode name="dn8" dataHost="localhost1" database="db8" />

        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <writeHost host="hostM1" url="192.168.178.61:3306" user="root" password="Password@1234">
                        <readHost host="hostS2" url="192.168.178.62:3306" user="root" password="Password@1234" />
                </writeHost>
        </dataHost>

</mycat:schema>

```
## 启动mycat 服务
```
  mycat start   # 如果正常的话会监听8066 端口   

```

 



