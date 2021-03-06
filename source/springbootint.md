## 应用开发指南

### 应用架构

目前uuac支持前后端分离架构的应用开发，架构如下所示：

```
                      +---------------+
                      |  keycloak     |
            +-------->+     uuac      +<--+
            |         |               |   |
check token |         +---------------+   |oauth2
            |get userinfo                 |
            |                             |
  +---------+------------+           +----+---------+
  |                      |   token   |              |
  |  spring boot server  +<----------+  vue client  |
  |                      |           |              |
  +----------------------+           +--------------+

```

我们会提供vue客户端jssdk，通过客户端完成oauth2认证，并将token缓存在客户端。

客户端带着token请求服务端api。

spring boot的服务端可以根据token与uuac交互，确认token的有效性，并获取认证的用户信息。


### Spring boot集成

统一用户中心支持直接与Spring boot应用集成，完成用户认证和资源访问控制功能。

#### 增加依赖

maven:
```
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-spring-boot-starter</artifactId>
</dependency>
```
gradle:
```
implementation 'org.keycloak:keycloak-spring-boot-starter:12.0.0'
```

#### 在配置文件中增加如下配置：

```
keycloak:
  auth-server-url: https://uuc.ide.gwunion.cn/auth/
  bearer-only: true
  principal-attribute: preferred_username
  public-client: true
  realm: master
  resource: vsapptobacco
  security-constraints:
  - authRoles:
    - myuser
    securityCollections:
    - patterns:
      - /api/product/*
  - authRoles:
    - myadmin
    securityCollections:
    - patterns:
      - /api/admin/*
  ssl-required: external
  use-resource-role-mappings: true
  verify-token-audience: false
```
如下所示, use-resource-role-mappings 配置为true表示获取uuac中配置的应用角色。

security-constraints配置项为角色与需要保护的api的对应关系，上面代码表示的是，具有myuser角色的用户，都可以调用/api/product/*下的所有接口，具有myadmin角色的用户，才可以调用/api/admin/*下的所有接口。

#### 获取用户信息

```
@PostMapping
public Task addTask(@RequestBody TaskVO taskVo,Principal principal){
  log.debug(principal.getName());
  taskVo.setUserId(principal.getName());
  return taskService.addTask(taskVo);
}
```
如上所示，通过Principal即可获取到当前登录用户的用户ID。

### VUE集成

#### 安装客户端jssdk

```
npm install @dsb-norge/vue-keycloak-js --save
```

#### jssdk使用

```
import VueKeycloakJs from '@dsb-norge/vue-keycloak-js'
import Vue from 'vue'
import App from './App.vue'
//import router from './router'
import store from './store'
import Vant from 'vant';
import axios from 'axios'
import 'vant/lib/index.css';
import { Lazyload } from 'vant';
import { Dialog } from 'vant';
import VConsole from 'vconsole'
import './style/index.less'

Vue.use(Lazyload);
//new VConsole();
Vue.config.productionTip = false
Vue.use(Vant)

function tokenInterceptor () {
  axios.defaults.baseURL = 'https://kcrestapidemo.ide.gwunion.cn/'
  axios.interceptors.request.use(config => {
    config.headers.Authorization = `Bearer ${Vue.prototype.$keycloak.token}`
    return config
  }, error => {
    return Promise.reject(error)
  })
  axios.interceptors.response.use(
    function(response) {
      //请求正常则返回
      return Promise.resolve(response)
    },
    function(error) {
      // 请求错误则向store commit这个状态变化
      console.log(error)
      if(401 == error.response.status){
        Dialog.alert({
          title: '温馨提示',
          message: '登录超时，系统将引导您重新获取授权。',
        }).then(() => {
          var ua = window.navigator.userAgent.toLowerCase();
          var loginUrl = router.app.$keycloak.createLoginUrl();
          console.log(ua)
          if (ua.match(/wxwork/i) == 'wxwork') {           
            loginUrl = router.app.$keycloak.createLoginUrl({
              idpHint: 'wechat-work'
            })
          }
          window.location.replace(loginUrl)
        });   
      }
      return Promise.reject(error)
    }
  )
}

Vue.use(VueKeycloakJs, {
  init: {
    // Use 'login-required' to always require authentication
    // If using 'login-required', there is no need for the router guards in router.js
    onLoad: 'check-sso',
    checkLoginIframe: false
  },
  config: {
    url: 'https://auth.jsyccloud.com/auth',
    clientId: 'tmlmain',
    realm: 'master'
  },
  onReady: (keycloak) => {
    tokenInterceptor();
    new Vue({
      store,
      render: h => h(App)
    }).$mount('#app')
  }
})
```

如上所示，主要配置uuac的url,clientId,realm等信息，在完成配置后，为了方便在所有请求中带上token到服务端，对axios进行了初始化配置。