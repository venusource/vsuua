## API网关

### 获取access_token

获取access_token是通过认证中心调用企业微信API接口的第一步，相当于创建了一个登录凭证，其它的业务API接口，都需要依赖于access_token来鉴权调用者身份。
因此开发者，在使用业务接口前，要获取access_token。

请求方式： POST（HTTPS）

请求地址： https://auth.jsyccloud.com/auth/realms/master/protocol/openid-connect/token

POST参数如下：

client_id	认证中心提供的client_id

client_secret	认证中心提供的应用的凭证密钥

grant_type  固定值：client_credentials

返回结果：
```
{
    "access_token":"xxxx",
    "expires_in":300,
    "refresh_expires_in":1800,
    "refresh_token":"xxxx",
    "token_type":"bearer",
    "not-before-policy":0,
    "session_state":"b9744ce9-3e9b-4d6e-b482-1748a830d59f",
    "scope":"profile email"
}
```
参数说明：

access_token	获取到的接口调用凭证
refresh_token	更新access_token的凭证

### 调用企业微信接口

调用企业微信接口时，需要将原api地址：`https://qyapi.weixin.qq.com` 换为企业微信接口网关地址：`https://qyapi.jsyccloud.com`，其它参数保持不变，以读取成员接口为例，原接口地址为：`https://qyapi.weixin.qq.com/cgi-bin/user/get?access_token=ACCESS_TOKEN&userid=USERID`，现需要更换为：`https://qyapi.jsyccloud.com/cgi-bin/user/get?access_token=ACCESS_TOKEN&userid=USERID`，其它请求参数与企业微信官方接口文档相同，地任何改变。

所有返回格式与企业微信官方文档相同，但系统会根据统一认证平台后端配置的应用权限，过滤掉无权限的数据，如获取用户信息返回值为：
```
{
    "errcode": 0,
    "errmsg": "ok",
    "userid": "zhangsan",
    "name": "张三",
    "department": [1, 2],
    "order": [1, 2],
    "position": "后台工程师",
    "gender": "1",
    "is_leader_in_dept": [1, 0],
    "avatar": "http://wx.qlogo.cn/mmopen/ajNVdqHZLLA3WJ6DSZUfiakYe37PKnQhBIeOQBO4czqrnZDS79FH5Wm5m4X69TBicnHFlhiafvDwklOpZeXYQQ2icg/0",
    "thumb_avatar": "http://wx.qlogo.cn/mmopen/ajNVdqHZLLA3WJ6DSZUfiakYe37PKnQhBIeOQBO4czqrnZDS79FH5Wm5m4X69TBicnHFlhiafvDwklOpZeXYQQ2icg/100",
    "alias": "jackzhang",
    "address": "广州市海珠区新港中路",
    "main_department": 1,
    "extattr": {
        "attrs": [
            {
                "type": 0,
                "name": "文本名称",
                "text": {
                    "value": "文本"
                }
            },
            {
                "type": 1,
                "name": "网页名称",
                "web": {
                    "url": "http://www.test.com",
                    "title": "标题"
                }
            }
        ]
    },
    "status": 1,
    "qr_code": "https://open.work.weixin.qq.com/wwopen/userQRCode?vcode=xxx",
    "external_position": "产品经理",
    "external_profile": {
        "external_corp_name": "企业简称",
        "external_attr": [{
                "type": 0,
                "name": "文本名称",
                "text": {
                    "value": "文本"
                }
            },
            {
                "type": 1,
                "name": "网页名称",
                "web": {
                    "url": "http://www.test.com",
                    "title": "标题"
                }
            },
            {
                "type": 2,
                "name": "测试app",
                "miniprogram": {
                    "appid": "wx8bd80126147dFAKE",
                    "pagepath": "/index",
                    "title": "my miniprogram"
                }
            }
        ]
    }
}

```

和官方文档相比，手机号、电话号码、邮箱等信息不返回给应用（根据具体的应用会有所不同）。

### 当前已支持的企业微信接口

#### 通讯录管理

[读取成员](https://open.work.weixin.qq.com/api/doc/90000/90135/90196)

[获取部门成员详情](https://open.work.weixin.qq.com/api/doc/90000/90135/90201)

[获取部门列表](https://open.work.weixin.qq.com/api/doc/90000/90135/90208)

#### 消息推送

[发送应用消息](https://open.work.weixin.qq.com/api/doc/90000/90135/90236)