Python
==============================================

.. toctree::
   :maxdepth: 2

玉符SDK集成了签署和验证JWT令牌的功能，使得身份提供者（IDP）和服务提供者（SP）只需要用很少的代码就可以轻松将玉符提供的单点登录和其他功能集成到现有的服务之中。

--------------

使用SDK之前
-----------

- 作为身份提供者（IDP），可以使用玉符SDK为用户生成并签署JWT令牌，并送到玉符服务器上认证并登录到第三方服务提供商。
  需从玉符注册并获取相应的身份提供者ID，并生成pem格式的公私钥文件。
- 作为服务提供者（SP），可以使用玉符SDK验证JWT令牌的有效性（包括有效期、签名等），成功后进行相应的建权。
- 如果服务提供者（SP）还为用户/租户提供身份联合、第三方登录的功能，可以使用玉符SDK生成并签署JWT令牌，并送到玉符服务器上认证并跳转到第三方身份提供者。(需额外生成pem格式的公私钥文件)

--------------

使用SDK
-------

**身份提供者（IDP)**

1. 实例化SDK的身份提供商::

    from YufuAuth import YufuAuth
    idProvider = YufuAuth.builder()
      .issuer("Your id provider id")                  # 从玉符注册并获取相应的身份提供者ID
      .tenant("Your tenant name")                     # 默认在玉符注册的租户名称，如果IDP支持多租户，此处可不填。
      .privateKeyPath("{Path/to/yourPrivateKey}")     # 租户在服务器上生成私钥文件（.pem格式），需安全保管。
      .build();

#. 根据用户选取或提前定义的服务应用ID，可以为用户生成并签署对应令牌\ ``generateIDPRedirectUrl``\ ，并向用户发送302跳转登录到第三方服务提供商，代码样式::

    payload = {
        'sub': "username123123",                      # 用户名称或ID
        'spid': "123"                                 # 用户选取或提前定义的服务应用ID
    }
    url = idProvider.generateIDPRedirectUrl(payload);
    redirectURL(url);                                 # 跳转用户至玉符云服务器认证授权，可根据需求在URL中添加定制的请求参数(如内网ip)便于后期审计

**服务提供者（SP)**

1. 实例化SDK的服务提供商::
    
    from YufuAuth import YufuAuth
    serviceProvider = YufuAuth.builder()      # 若需要token签署功能（租户在服务器上生成私钥文件（.pem格式），需安全保管）
      .privateKeyPath("{Path/to/yourPrivateKey}")
      .build();

#. 实现单点登录：接收并验证\ ``verify``\ JWT令牌的有效性（包括有效期、签名等），如通过，说明该令牌来自玉符信任的有效租户(企业/组织)的用户，样例::

    idToken = getIdToken();                   # 从URL中获得 ID token
    claims = serviceProvider.verify(idToken); # 使用验证玉符SDK实例进行验证, 如果成功会返回包含用户信息的对象，失败则会产生授权错误的异常
    username = claims.getSubject();           # 用户名称   String clientId = claims.getAudience();       // 玉符签发的唯一识别码
    tenant = claims.getClaims().get("tnt");   # 租户名称``

#. 根据第2步获取的用户信息，您的系统可进行鉴权和赋予登录系统等相应权限，否则提示用户登录失败。推荐鉴权方案:

   -  服务提供商(SP)允许租户管理员输入一个玉符的唯一识别码
   -  在第2步Token验证通过后，根据获取的租户名称/ID和识别码,
     查看是否匹配该租户在服务提供商(SP)所提供的唯一识别码，如匹配则表示租户为有效租户。
   -  接着查看用户是否存在于租户中，如果存在，则赋予登录系统的权限。

异常说明
--------

1. 在调用verify()方法时可能抛出以下异常:

   -  InvalidTokenException : 无效token
   -  InvalidFormatException : token格式不正确
   -  TokenExpiredException : token过期
   -  TokenParseException : 解析token出错
   -  TokenTooEarlyException : token在生效之前使用
   -  MissingKeyIdException : 缺少KeyID
   -  CannotRetrieveKeyException : key为空

#. 创建RSATokenGenerator对象时可能抛出YufuInitException异常，存在以下几种情况：

   -  找不到private key文件路径
   -  private key文件无效
   -  无法读取private key文件

#. 调用generate()方法时，方法体中的sign()方法可能抛出GenerateException异常。

FAQ
---
