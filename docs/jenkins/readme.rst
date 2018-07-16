Jenkins 2.1XX SAML
==============================================

.. toctree::
   :maxdepth: 2

玉符作为身份提供者（IDP），能够轻松提供SAML单点登录将Jenkins集成到现有的服务之中。

--------------


集成步骤
-------

**玉符管理界面（IDP)**

1. 创建新应用Jenkins-SAML:
   - 第一步： 填写应用名称和描述等基本信息
   - 第二步： 复制metadata信息，并在`Auth 2.0 标记端点`填写您的Jenkins认证地址：
    `http(s)://{您的Jenkins服务地址}/securityRealm/finishLogin`。注意http或https需填写正确
   - 直接启用应用并设置对应的安全组/权限

**Jenkins管理页面（SP)**

1. 安装插件:
   - 第一步： 进入`管理` -> `插件管理`
   - 第二步： 在`可用插件`搜索SAML并安装
2. 设置插件:
   - 第一步：进入`管理` -> `全局安全管理`
   - 第二步：在`权限管理（Security Realm）`里面选择 `SAML 2.0`
   - 第三步: 在`IdP Metadata`下面填入玉符应用配置界面提供的metadata信息，点击`验证（Validate Idp Metadata）`，
   继续在`Username Attribute`填入 `NameID`，并选择`Data Binding Method	`为 `HTTP-POST`。
   - （可选）第四步：在`logoutUrl`填入`https://{您的玉符租户网址}`
3. 在保存上面设置并进行测试之前，最好在`权限（Authorization）`一栏暂时运行所有人做任何操作，以免错误操作导致登录失败，则需进入Jenkins机器重置安全选项。

**玉符用户界面（IDP)**
1. 测试单点登录并进入到Jenkins界面，并确认登录信息

常见问题FAQ
---
1. 登录过程出现405 Method Not Allowed
  - Jenkins全局设定的Jenkins Url可能参数有错
1. 登录失败，无法定位
   - Jenkins有自带的日志查询，http(s)://{您的Jenkins服务地址}/log/all