# -JMETER
最近需要用jmeter测试火车票，采集微服务系统的负载基准数据

环境：centos8、jdk1.8

        【注】
        jdk一定要用官方版本，openjdk不可，卸载！！！
        重装后记得配环境 /etc/profile

下载：

        jmeter5.3
        influxdb(数据库)
        grafana(数据可视化)

Step 1:jmeter 安装

 1、下载 
 
        wget https://mirrors.bfsu.edu.cn/apache//jmeter/binaries/apache-jmeter-5.3.tgz 【下载到自己喜欢的目录也行】
2、解压 

        tar -xzvf apache-jmeter-5.3.tgz
3、配环境 

vim /etc/profile

        export JMETER=/.../apache-jmeter-5.3 【写自己的路径】
        export CLASSPATH=$JMETER/lib/ext/ApacheJMeter_core.jar:$JMETER/lib/jorphan.jar:$JMETER/lib/logkit-2.0.jar:$CLASSPATH
        export PATH=$JMETER/bin/:$PATH
4、运行配置文件

        source /etc/profile
5、验证

        jmeter -version

Step 2:influxdb(数据库) 安装与配置

1、下载

        wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.0.x86_64.rpm --no-check-certificate

2、解压

        rpm -ivh influxdb-1.7.0.x86_64.rpm 

3、修改influxDB配置文件

        【在配置文件中找到graphite配置项，去掉前面的“#”号】
        vim /etc/influxdb/influxdb.conf 
        [[graphite]]
        # Determines whether the graphite endpoint is enabled.
        enabled = true
        database = "jmeter"    # 数据库名称
        retention-policy = ""
        bind-address = ":2003"    # 端口
        protocol = "tcp"
        consistency-level = "one"
        修改以下信息
        [meta]
        dir = "/usr/local/influxdb/meta"    #存放最终存储的数据，文件以.tsm结尾
        [data]
        dir = "/usr/local/influxdb/data"    #存放数据库元数据 wal
        wal-dir = "/usr/local/influxdb/wal"    #存放预写日志文件
        修改HTTP端口信息
        [http]
        # Determines whether HTTP endpoint is enabled.
        enabled = true
        # The bind address used by the HTTP service.
        bind-address = ":8086"

4、创建目录更新权限

        mkdir -p /usr/local/influxdb/
        chown -R influxdb:influxdb /usr/local/influxdb/

5、修改完成之后，可以使用以下命令启动influxDB服务，两种方法：

        A、influxd -config /etc/influxdb/influxdb.conf

        B、添加到环境变量中(推荐）
          vim /etc/profile
          export INFLUXDB_CONFIG_PATH=/etc/influxdb/influxdb.conf
          source /etc/profile
          [root@jmeter  ~]# influxd &
          influxDB的tcp端口：8088
          查看端口有没有起来
          [root@jmeter ~]# netstat -anp|grep 8088
        
Step3: grafana 下载和安装

        官网：https://grafana.com/grafana/download
        下载地址：https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.2.1-1.x86_64.rpm
        安装：rpm -ivh grafana-5.2.1-1.x86_64.rpm
        启动：
        service grafana-server start
        Starting Grafana Server: ... [ OK ]

        浏览器访问：http://IP:3000/login
        grafana的默认用户名密码都是admin，第一次登录会要求更改密码
