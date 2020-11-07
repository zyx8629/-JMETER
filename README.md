# -JMETER
最近需要用jmeter测试火车票，采集微服务系统的负载基准数据

环境：centos8、jdk1.8

        【注】
        jdk一定要用官方版本，openjdk不可，卸载！！！【别卸载，修改路径吧还是，可能有依赖存在】
        重装后记得配环境 /etc/profile

下载：

        jmeter5.3
        influxdb(数据库)
        grafana(数据可视化)

# Step 1:Jmeter 安装

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

# Step 2:influxdb(数据库) 安装与配置

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

5、修改完成之后，可以使用以下命令启动 influxDB服务，两种方法：

        A、influxd -config /etc/influxdb/influxdb.conf

        B、添加到环境变量中(推荐）
          vim /etc/profile
          export INFLUXDB_CONFIG_PATH=/etc/influxdb/influxdb.conf
          source /etc/profile
          [root@jmeter  ~]# influxd &
          influxDB的tcp端口：8088
          查看端口有没有起来
          [root@jmeter ~]# netstat -anp|grep 8088
          
          特别说明：
             「
                新版本 influxdb没有UI展示了好像吧·····😄我是没调出来
                8086端口：Grafana用来从数据库取数据的端口
                2003端口：刚刚设置的，Jmeter往数据库发数据的端口
                                                           」

# Step 3: grafana 下载和安装

        官网：https://grafana.com/grafana/download
        下载地址：https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.2.1-1.x86_64.rpm
        安装：rpm -ivh grafana-5.2.1-1.x86_64.rpm
        启动：
        service grafana-server start
        Starting Grafana Server: ... [ OK ]

        浏览器访问：http://IP:3000/login
        grafana的默认用户名密码都是admin，第一次登录会要求更改密码
        
# Step 4：Jmeter添加并配置 BackendListener 连接 influxdb数据库

        Implementatin是 GraphiteBackendListenerClient ，每个配置项的含义：
        1、graphiteHost：InfluxDB安装的服务器的ip
        2、graphitePort：端口；默认就是2003，除非你自己安装InfluxDB时设置了其他端口是哦（可见上面安装InfluxDB后关于graphite的配置）
        3、rootMetricsPrefix：指标的根前缀；将测试结果存入数据库时，不同指标会生成不同表，但这些表都最好要有一个共同的前缀，这个就是了；后面会讲到不同的指标的含义（重点哦）
        4、summaryOnly：当你线程组有多个请求又想知道每个请求的结果数据时，最好填false，因为true只会返回所有请求的集合数据报告，不会输出每条请求的数据报告
        5、samplersList：取样器列表；想收集哪些请求就填哪些，最好用正则去匹配，减轻工作量
        6、useRegexpForSamplersList：是否使用正则；如果true则使用，samplersList里可以匹配正则表达式
        7、percentiles：百分比；即类似聚合报告里90% Line，95% Line，99% Line的数据；倘若想要99.9时，需要写成【99_9】，用下划线代替点

# Step 5:配置 Grafana

        参考：https://www.cnblogs.com/poloyy/p/12219145.html
