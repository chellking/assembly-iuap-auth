# 整体设计

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖Apache的shiro框架,引入了shiro.shiro-spring的1.2.3版本和iUAP平台的一些基础组件如iuap-log和iuap-cache，其对外提供的依赖方式如下：
```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-auth</artifactId>
	  <version>2.0.1-SNAPSHOT</version>
	</dependency>
```
## 功能结构 ##

<img src="/images/shiro_auth.jpg"/>

**基本概念**

身份验证，即在应用中谁能证明他就是他本人。一般提供如他们的身份ID一些标识信息来表明他就是他本人，如提供身份证，用户名/密码来证明。在shiro中，用户需要提供principals （身份）和credentials（证明）给shiro，从而应用能验证用户身份：

- principals：身份，即主体的标识属性，可以是任何东西，如用户名、邮箱等，唯一即可。一个主体可以有多个principals，但只有一个Primary principals，一般是用户名/密码/手机号。
- credentials：证明/凭证，即只有主体知道的安全值，如密码/数字证书等。
最常见的principals和credentials组合就是用户名/密码了。接下来先进行一个基本的身份认证。
 
另外两个相关的概念是之前提到的Subject及Realm，分别是主体及验证主体的数据源。

## 流程说明 ##

- 1、首先调用Subject.login(token)进行登录，其会自动委托给Security Manager，调用之前必须通过SecurityUtils. setSecurityManager()设置；
- 2、SecurityManager负责真正的身份验证逻辑；它会委托给Authenticator进行身份验证；
- 3、Authenticator才是真正的身份验证者，Shiro API中核心的身份认证入口点，此处可以自定义插入自己的实现；
- 4、Authenticator可能会委托给相应的AuthenticationStrategy进行多Realm身份验证，默认ModularRealmAuthenticator会调用AuthenticationStrategy进行多Realm身份验证；
- 5、Authenticator会把相应的token传入Realm，从Realm获取身份验证信息，如果没有返回/抛出异常表示身份验证失败了。此处可以配置多个Realm，将按照相应的顺序及策略进行访问。



