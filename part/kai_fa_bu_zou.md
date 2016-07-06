## 开发步骤 ##

- 配置示例工程中的redis.session.url为正确的redis地址，redis可以采用DevTool中bin目录下的redis，例如示例工程下的application.properties

		#auth组件需要的redis地址配置
		redis.session.url=direct://localhost:6379?poolSize=50&poolName=mypool
		
		#session过期时间，对应的是redis中的缓存失效时间
		sessionTimeout=6000
		
		#是否登录时候剔除当前用户在其他位置的登录,默认为不互踢
		sessionMutex=false
		
		#客户定义的不进行shiro过滤的url后缀
		filter_excludes=.woff2
		
		#当前应用的context名
		context.name=/example_iuap_auth
		
		#应用的编号
		sysid=hr_cloud

- 配置web.xml

		<filter>
	    	<filter-name>shiroFilter</filter-name>
	    	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	    	<init-param>
	      		<param-name>targetFilterLifecycle</param-name>
	      		<param-value>true</param-value>
	    	</init-param>
	    </filter>
	    <filter-mapping>
	    	<filter-name>shiroFilter</filter-name>
	    	<url-pattern>/*</url-pattern>
	    </filter-mapping>

- 配置applicationContex-shiro.xml,例如示例工程中的配置文件

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans"
			xmlns:util="http://www.springframework.org/schema/util" xmlns:aop="http://www.springframework.org/schema/aop"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi:schemaLocation="
		       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
		       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
		
			<bean id="statelessRealm" class="com.yonyou.iuap.auth.shiro.StatelessRealm">
				<property name="cachingEnabled" value="false" />
			</bean>
		
			<!-- Subject工厂 -->
			<bean id="subjectFactory"
				class="com.yonyou.iuap.auth.shiro.StatelessDefaultSubjectFactory" />
			<bean id="webTokenProcessor" class="com.yonyou.iuap.auth.token.DefaultTokenPorcessor">
				<property name="id" value="web"></property>
				<property name="path" value="${context.name}"></property> 
				<property name="expr" value="${sessionTimeout}"></property>
				<property name="exacts">
					<list>
						<value type="java.lang.String">tenantid</value>
					</list>
				</property>
			</bean>
			<bean id="maTokenProcessor" class="com.yonyou.iuap.auth.token.DefaultTokenPorcessor">
				<property name="id" value="ma"></property>
				<property name="path" value="${context.name}"></property> 
				<property name="expr" value="-1"></property>
				<property name="exacts">
					<list>
						<value type="java.lang.String">tenantid</value>
					</list>
				</property>
			</bean>
		
			<bean id="tokenFactory" class="com.yonyou.iuap.auth.token.TokenFactory">
				<property name="processors">
					<list>
						<ref bean="webTokenProcessor" />
						<ref bean="maTokenProcessor" />
					</list>
				</property>
			</bean>
		 
		
			<!-- 会话管理器 -->
			<bean id="sessionManager" class="org.apache.shiro.session.mgt.DefaultSessionManager">
				<property name="sessionValidationSchedulerEnabled" value="false" />
			</bean>
		
			<!-- 安全管理器 -->
			<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
				<property name="realms">
					<list>
						<ref bean="statelessRealm" />
					</list>
				</property>
				<property name="subjectDAO.sessionStorageEvaluator.sessionStorageEnabled"
					value="false" />
				<property name="subjectFactory" ref="subjectFactory" />
				<property name="sessionManager" ref="sessionManager" />
			</bean>
		
			<!-- 相当于调用SecurityUtils.setSecurityManager(securityManager) -->
			<bean
				class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
				<property name="staticMethod"
					value="org.apache.shiro.SecurityUtils.setSecurityManager" />
				<property name="arguments" ref="securityManager" />
			</bean>
		
			<bean id="statelessAuthcFilter" class="com.yonyou.iuap.auth.shiro.StatelessAuthcFilter">
				<property name="sysid" value="${sysid}" />
				<property name="tokenFactory" ref="tokenFactory" />
			</bean>
		
			<bean id="logout" class="com.yonyou.iuap.auth.shiro.LogoutFilter"></bean>
		
			<!-- Shiro的Web过滤器 -->
			<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
				<property name="securityManager" ref="securityManager" />
				<property name="loginUrl" value="/login" />
				<property name="successUrl" value="/" />
				<property name="filters">
					<util:map>
						<entry key="statelessAuthc" value-ref="statelessAuthcFilter" />
					</util:map>
				</property>
				<property name="filterChainDefinitions">
					<value>
						/logout = logout
						/static/** = anon
						/css/** = anon
						/images/** = anon
						/trd/** = anon
						/js/** = anon
						/api/** = anon
						/cxf/** = anon
						/jaxrs/** = anon
						/** = statelessAuthc
					</value>
				</property>
			</bean>
		
			<!-- Shiro生命周期处理器 -->
			<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
		
			<bean id="sessionJedisPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory" scope="prototype" factory-method="createJedisPool">
				<constructor-arg value="${redis.session.url}" />
			</bean>
		    
		    <bean id="sessionMgr" class="com.yonyou.iuap.auth.session.SessionManager">
		        <property name="sessionJedisPool" ref="sessionJedisPool"/>
		        <property name="sessionMutex" value="${sessionMutex}"/>
		    </bean>	
		</beans>

关键配置项为shiroFilter，这里配置了那些路径的访问会需要statelessAuthc类型的过滤器拦截，如果特殊的路径不需要拦截，配置成anon。sessionJedisPool和sessionMgr即对redis的配置。

	loginUrl 验证不成功，跳转到此url
	successUrl 登录成功后，跳转到此url

其中webTokenProcessor和maTokenProcessor分别配置的是对浏览器登录和手机端登录的token生成处理bean，其中path和expr即为登录成功后写cookies的一些参数，exacts为业务中需要携带的额外参数，会一并写入到cookies中。示例中例如tenantid参与了token的生成和cookies的回写。在用户登录成功后，这些cookies信息会随着下一次的服务端调用，携带到后端进行验证。

如果是移动端调用，需要登录成功后，将cookies信息以自身的方式进行存储，下次请求发送的时候，以http header的方式携带回来。
配置文件中，退出系统的默认filter如下：

	<bean id="logout" class="com.yonyou.iuap.auth.shiro.LogoutFilter"></bean>

业务信息可以重写此filter，清除自身的其它信息。

- 编写登录处理类，示例中默认的登录示例代码如下：

		package com.yonyou.iuap.auth.example.web;
		
		import java.io.IOException;
		
		import javax.servlet.http.Cookie;
		import javax.servlet.http.HttpServletRequest;
		import javax.servlet.http.HttpServletResponse;
		
		import org.apache.shiro.web.filter.authc.FormAuthenticationFilter;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.stereotype.Controller;
		import org.springframework.ui.Model;
		import org.springframework.web.bind.annotation.RequestMapping;
		import org.springframework.web.bind.annotation.RequestMethod;
		import org.springframework.web.bind.annotation.RequestParam;
		
		import com.yonyou.iuap.auth.session.SessionManager;
		import com.yonyou.iuap.auth.shiro.AuthConstants;
		import com.yonyou.iuap.auth.token.ITokenProcessor;
		import com.yonyou.iuap.auth.token.TokenParameter;
	
		/**
		 * 默认登录逻辑
		 */
		@Controller
		@RequestMapping(value = "/login")
		public class LoginController {
			
		    private final Logger logger = LoggerFactory.getLogger(getClass());
			
			public static final int HASH_INTERATIONS = 1024;
			
		    @Autowired
		    private SessionManager sessionManager;
		
			//为网页版本的登录Controller指定webTokenProcessor 相应的移动的指定为maTokenProcessor
			@Autowired()
			protected ITokenProcessor webTokenProcessor;
			
			@RequestMapping(method = RequestMethod.GET)
			public String login(Model model) {
				return "login";
			}
			
			@RequestMapping(method = RequestMethod.POST,value="formLogin")
			public String formLogin(HttpServletRequest request, HttpServletResponse response, Model model) throws IOException {
		        String userName = request.getParameter("username");
		        String passWord = request.getParameter("password");
				
				if (passWord != null && userName != null) {
					// 模拟用户
					if("admin".equals(userName) && passWord.equals("admin")){
		                
		                TokenParameter tp = new TokenParameter();
		                tp.setUserid(userName);
		                //设置登录时间
		                tp.setLogints(String.valueOf(System.currentTimeMillis()));
		                //租户信息
		                tp.getExt().put(AuthConstants.PARAM_TENANTID , "hr_cloud");
		                Cookie[] cookies = webTokenProcessor.getCookieFromTokenParameter(tp);
		                for(Cookie cookie : cookies){
		            	    response.addCookie(cookie);
		                }
		            } else {
		            	logger.error("用户名密码错误!");
		                model.addAttribute("accounterror", "用户名密码错误!");
		                return "login";
		            }
		            return "redirect";
				} else {
		            model.addAttribute("accounterror", "你输入的用户不存在!");
		            return "login";
				}
			}
			
			@RequestMapping(method = RequestMethod.POST)
			public String fail(@RequestParam(FormAuthenticationFilter.DEFAULT_USERNAME_PARAM) String userName, Model model) {
				model.addAttribute(FormAuthenticationFilter.DEFAULT_USERNAME_PARAM, userName);
				return "login";
			}
		
		}