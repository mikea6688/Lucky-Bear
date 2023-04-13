## OAuth 

### 什么时候用到OAuth2 ？

OAuth2用于第三方授权。比如，用户在使用A网站，想获得B网站的数据，这时就需要B网站提供oauth2的授权功能让A网站获取。例如在登录百度时，可以选择第三方账号登录（微信、钉钉等）

### 第三方应用程序为什么要用OAuth2？

假设有某个流程图网站，支持将流程图存入百度云盘或者从百度云盘读出。最简单的办法就是百度云盘直接将用户的账号密码给流程图网站，但这样有如下问题：

1. “流程图网站”为了后续的服务，会保存用户的密码，这样很不安全。
2. “流程图网站”拥有了获取用户储存在百度云盘所有资料的权力，用户没法限制”流程图网站”获得授权的范围和有效期。
3. 用户只有修改密码，才能收回赋予”流程图网站”的权力。
4. 只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有被密码保护的数据泄漏。

### 介绍OAuth

OAuth2.0是一种允许第三方应用程序使用资源所有者的凭据获得对资源有限访问权限的一种授权协议。例如通过微信登录百度，相当于微信允许百度作为第三方应用程序在经过微信用户授权后，通过微信颁发的授权凭证有限地访问用户的微信头像、手机号，性别等资源，从而来构建自身的登录逻辑。 在OAuth2.0协议中第三方应用程序获取的凭证并不是资源拥有者的用户名和密码。OAuth2允许用户提供一个令牌（token）给第三方网站，一个令牌对应一个特定的第三方网站，且该令牌只能在特定的时间内访问特定的资源。

### OAuth2.0的角色

1. resource owner（资源拥有者）：我们使用通过微信账号登录百度，而微信账号信息的资源拥有者就是微信用户。
2. client（客户端）：我们使用通过微信账号登录百度，客户端就是百度。
3. authorization server（授权服务器）：我们使用通过微信账号登录百度，授权服务器就是微信开放平台提供的用于认证的服务器，用于给resource server管理第三方授权。
4. resource server（资源服务器）：我们使用通过微信账号登录百度，资源服务器就是微信服务器。

### 流程图

![OAuth2--流程/原理_服务器_04](https://s2.51cto.com/images/blog/202202/16130046_620c84fe3582958761.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

### 授权码模式及其流程

第三方应用先申请获取一个授权码，然后再使用该授权码获取令牌，最后使用令牌获取资源；授权码模式是工能最完整、流程最严密的授权模式。它的特点是通过客户端的后台服务器，与服务提供商的认证服务器进行交互。

![授权码模式](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bf4454bd4474e8eaca411a878f0723d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)



https://server.example.com/oauth/token?client_id=CLIENT_ID&client_secret=CLIENT_SECRET&grant_type=authorization_code&code=AUTHORIZATION_CODE&redirect_uri=CALLBACK_URL

**client_id**：客户端id，第三方应用在服务提供者平台注册的；

**client_secret**：授权服务器的秘钥，第三方应用在服务提供者平台注册的；

**grant_type**：值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码；

**code**:表示上一步获得的授权码;

**redirect_uri**:redirect_uri参数是令牌颁发后的回调网址,且必须与A步骤中的该参数值保持一致;

（注：client_id参数和client_secret参数用来让 B 确认 A 的身份（client_secret参数是保密的，因此只能在后端发请求））



#### 参考文档：https://juejin.cn/post/7010636081305485319