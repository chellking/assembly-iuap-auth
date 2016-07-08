# 接口API和使用说明


## 1，客户端使用
#### 1.1,公有部署的配置，首先引入jar包

1.1.1，公有部署需要的Jar包：

<pre>
     &lt;dependency>
       &lt;groupId>com.yonyou.iuap&lt;/groupId>
       &lt;artifactId>iuap-securitylog-rest-sdk&lt;/artifactId>
       &lt;version>1.0.1-SNAPSHOT&lt;/version>
     &lt;/dependency>
</pre>

1.1.2，公有部署需要配置的配置文件，securitylogrestsdk-applicationContext.xml和securitylogrestsdk-applicationContext-mq-provider.xml

1.1.3，	需要的MQ配置文件，logConfig.properties ，里面是关于MQ的内容。支持通过环境变量传入路径的方式，key值是securitylog-logConfig-filePath，即securitylog-logConfig-filePath=配置文件路径。

当然，若是从环境变量中读取不到，最后还是会走默认的classpath下的配置文件，注意和服务端保持一致：

    #集群地址配置，多个的用逗号隔开
    mq.address=172.20.14.133:5672

    #如果mq.isLocal=true, 可以不用配置下面两项的值
    mq.username=admin
    mq.password=admin

1.1.4,调用工具类：在要记录日志的地方，调用com.iuap.log.security.utils.SecurityLogUtil.saveLog(SecurityLog)方法来记录日志

### 1.2，私有部署的配置，首先引入jar包
1.2.1，私有部署需要的jar包
<pre>
  &lt;dependency>
      &lt;groupId>com.yonyou.iuap&lt;/groupId>
      &lt;artifactId>iuap-securitylog-local-sdk&lt;/artifactId>
      &lt;version>1.0.1-SNAPSHOT&lt;/version>  
  &lt;/dependency>
</pre>

1.2.2，	私有部署，需要配置服务信息的配置文件securitylogconfiger.properties，也支持从环境变量中传入，key值为securitylogconfiger-filePath，传入方式为securitylogconfiger-filePath = 配置文件路径。

    #使用非公有服务时，需要配置日志服务所在的ip和端口
    serverip=127.0.0.1
    serverport=8080

    #应用名
    appname=SecurityLog_Server

1.2.3,调用工具类：在要记录日志的地方，调用com.iuap.log.security.utils.SecurityLogUtil.saveLog(SecurityLog)方法来记录日志


## 2，服务端部署
### 2.1，说明：

（1），服务端的war包是相同的，只是对于共有的和私有的，有些不同的配置。

（2），服务端部署，部署一个war包
<pre>
&lt;dependency>
  &lt;groupId>com.yonyou.iuap&lt;/groupId>
  &lt;artifactId>iuap-securitylog-service&lt;/artifactId>
  &lt;version>1.0.1-SNAPSHOT&lt;/version>
  &lt;type>war&lt;/type>
&lt;/dependency>
</pre>

### 2.2，配置，不管是公有的还是私有部署，都有以下配置：
2.2.1，数据库信息配置（securitylog-application.properties）：

    #下面几项必须配置
    jdbc.url=jdbc:mysql://172.20.14.207:3306/securitylog?useUnicode=true&characterEncoding=gb2312
    jdbc.driverclass=com.mysql.jdbc.Driver
    jdbc.user=root
    jdbc.password=root

    #默认数据库连接池配置,不需要设置
    initPoolSize=10
    maxPoolSize=5


2.2.2，对于公有部署，有下面关于MQ的额外配置，私有部署跳过，
MQ服务器配置securitylogMQConfig.properties，需要放在classpath目录下：

    #集群地址配置，多个的用逗号隔开，
    mq.address=172.20.14.133:5672

    #如果mq.isLocal=true, 可以不用配置下面两项的值
    mq.username=admin
    mq.password=admin
