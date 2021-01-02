## 开放认证

### OAuth2.0授权（WEB）

#### 场景说明

OAuth2是基于统一用户中心进行应用开发的用户认证方式，目前统一认证中心打通了企业微信、钉钉等第三方用户认证方式，应用开发商不需要关心应用运行环境造成的认证方式不同，只需要基于统一用户中心OAuth2协议进行开发即可适配不同的移动社交平台。

通过此接口获取成员身份会有一定的时间开销。对于频繁获取成员身份的场景，建议采用如下方案：
1. 应用开发商使用自己的token或cookie机制保存应用登录状态。
2. 如果没有匹配的token或cookie，则重定向到统一用户中心OAuth验证链接，获取成员的身份信息后，由应用后台自己颁发token或cookie到客户端，用于识别当前登录用户身份。

#### 授权流程

##### 获取code

应用判断用户未登录后，构造如下的连接来获取code:

https://统一用户中心IP/auth/realms/{realm-name}/protocol/openid-connect/auth?response_type=code&client_id=CLIENT_ID&
redirect_uri=CALLBACK_URL&scope=openid&kc_idp_hint=wechat-work&state=STATE

参数说明：

| 参数          | 是否必须 | 说明                                                                          |
|---------------|----------|-------------------------------------------------------------------------------|
| client_id     | 是       | 应用唯一标识                                                                  |
| response_type | 是       | 返回类型，此时固定为：code                                                    |
| scope         | 是       | 应用授权作用域。 openid：手动授权，可获取成员的基础信息；                     |
| redirect_uri  | 是       | 授权后重定向的回调链接地址，请使用urlencode对链接进行处理                     |
| kc_idp_hint         | 否       | 如果为空，系统将使用默认的用户名密码认证，目前支持以下认证方式：wechat-work 企业微信认证； dingtalk 钉钉认证； wechat 微信认证 |
| state         | 否       | 重定向后会带上state参数，应用可以填写a-zA-Z0-9的参数值，长度不可超过128个字节 |

应用确认授权信息后，页面将跳转至 redirect_uri?code=CODE&state=STATE，应用可根据code参数获得access_token。

##### 获取access_token

应用拿到code后，使用code换到access_token，向下面的连接发送POST请求：

```
POST https://统一用户中心IP/auth/realms/{realm-name}/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&code=AUTHORIZATION_CODE&redirect_uri=REDIRECT_URI
```
参数说明：

| 参数          | 是否必须 | 说明                                                                          |
|---------------|----------|-------------------------------------------------------------------------------|
| grant_type     | 是       | 固定值authorization_code                                                                  |
| client_id | 是       | 应用唯一标识                                                    |
| client_secret         | 是       | 应用管理页面获取到的secret                     |
| redirect_uri  | 是       | 授权后重定向的回调链接地址，请使用urlencode对链接进行处理                     |
| code         | 是       | 上一步同获得的code |

返回值：

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token":"eyJz93a...k4laUWw",
  "refresh_token":"GEbRxBN...edjnXbL",
  "id_token":"eyJ0XAi...4faeEoQ",
  "token_type":"Bearer",
  "expires_in":86400
}
```

##### 获取用户信息

发送下面的HTTP请求获取用户信息

```
GET https://统一用户中心IP/auth/realms/{realm-name}/protocol/openid-connect/userinfo
Authorization: 'Bearer {ACCESS_TOKEN}'
```

参数说明：

* ACCESS_TOKEN 上一步中返回的access_token

返回值：

```
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
   "sub": "248289761001",
   "name": "Jane Doe",
   "given_name": "Jane",
   "family_name": "Doe",
   "preferred_username": "j.doe",
   "email": "janedoe@example.com",
   "picture": "http://example.com/janedoe/me.jpg"
  }
```

返回值根据不同的应用，和用户授权scope会有差异。

### 小程序授权

#### 客户端获取企业微信code

小程序前端先通过下面的接口获取到用户授权码CODE:
```
wx.qy.login({
  success: function(res) {
    if (res.code) {
      //将code发送到服务器端调用统一认证相关接口
      wx.request({
        url: 'https://您自己的服务器/',
        data: {
          code: res.code
        }
      })
    } else {
      console.log('登录失败！' + res.errMsg)
    }
  }
});
```

#### 获取accessToken

** 此接口为了安全，必须从服务器端调用，严禁在客户端直接发起请求 **

通过下面的HTTP请求获取access_token:

GET https://统一用户中心IP/auth/realms/master/miniappext/gettoken?client_id=CLIENT_ID&secret=SECRET&code=CODE


参数说明：

| 参数          | 是否必须 | 说明                                                                          |
|---------------|----------|-------------------------------------------------------------------------------|
| client_id     | 是       | 应用唯一标识                                                                  |
| secret | 是       | 应用secret，由统一认证平台提供                                                    |
| code         | 是       | 小程序前端获取到的code                     |

返回值：

```
{
    "access_token": "xxxxxx",
    "expires_in": 60,
    "refresh_expires_in": 3600,
    "refresh_token": "xxxxx"
}
```

#### 获取用户信息

同OAuth2.0授权（WEB）中的获取用户信息接口。

