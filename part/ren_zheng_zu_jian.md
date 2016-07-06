# 认证组件

本组件提供对身份认证的功能，提供多种token生成方式，对指定路径的服务资源进行认登录认证。支持cookie和Http header的方式验证用户信息，并提供无状态的方式与shiro集成，利用分布式缓存组件，管理Session信息。

Apache Shiro是Java的一个轻量级的安全框架，简单易用。Shiro可以帮助开发人员完成认证、授权、加密、会话管理、Web集成、缓存等功能。

iUAP auth组件利用spring和shiro进行集成，使用token的方式对用户进行认证，token的生成参数可以由用户定制。同时组件采用无状态的方式，将shiro和web应用组装，配合分布式缓存redis的使用对session进行管理，实现了web服务的无状态，便于服务的水平扩展。


## 本版更新


1. 1. 提供对指定资源的认证拦截

2. 支持PC和移动端的不同的token生成方式

3. 支持分布式缓存方式存储session信息

4. 支持session信息的无状态方案

5. 支持cookie和header的方式传递用户信息

6. 支持互斥登录配置




