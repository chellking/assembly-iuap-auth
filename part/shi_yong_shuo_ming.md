# 使用说明 #

## 组件包说明 ##
iuap-auth组件利用shiro的上述登录认证的步骤，构造了token并进行了认证动作，并提供SessionManager管理类，将传统方式的一些Session信息存储在了分布式缓存Redis中，方便后续的访问。

iuap-auth组件依赖分布式缓存Redis，所以部署时候，需要先安装部署Redis，使用步骤可以参考iuap-cache组件的说明文档。

本组件只提供基础的认证登录过程，shiro中涉及到的权限相关集成，请参考应用组件库中的权限组件相关手册。

##组件配置##

**1:在属性文件中，配置session所需要的redis的连接url、sessionTimeout等信息**

	配置示例：
	
	redis.session.url=direct://localhost:6379?poolSize=50&poolName=mypool
	sessionTimeout=6000
	//是否登录时候剔除当前用户在其他位置的登录,默认为不互踢
	sessionMutex=false

**2:Spring的配置文件中，加入shiro的配置文件，如applicationContext-shiro.xml**

	1）配置statelessRealm

	2）配置TokenProcessor，pc端或者手机端登录的token生成用

	3）shiroFilter和需要过滤的资源url

	4）配置会话管理的sessionJedisPool和sessionMgr


**3:工程中引入对iuap-auth组件的依赖**

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-auth</artifactId>
		<version>2.0.1-SNAPSHOT</version>
	</dependency>	

**4:代码中调用组件提供的API，生成token等信息并保存cookies**

	TokenParameter tp = new TokenParameter();
	tp.setUserid(userName);
	//设置登录时间
	tp.setLogints(String.valueOf(System.currentTimeMillis()));
	//租户信息
	tp.getExt().put(AuthConstants.PARAM_TENANTID , "tenant01");
	Cookie[] cookies = webTokenProcessor.getCookieFromTokenParameter(tp);
	for(Cookie cookie : cookies){
	    response.addCookie(cookie);
	}

**5:调用组件提供的对session信息的API存取session信息**
	
参考SessionManager中的API。

## 工程样例 ##


<img src="/images/auth_example.jpg"/>

开发工具包DevTool中携带了对认证组件的示例工程，位置位于DevTool/examples/example\_iuap\_auth下，在IUAP_STUDIO中导入已有的Maven工程，可以将示例工程导入到工作区。示例工程中有较为完整的对iuap-auth组件的使用示例代码。

