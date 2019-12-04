# IView REST API Guide

Author [shidenghui@gmail.com](mailto:shidenghui@gmail.com)

# 1 Overview

## 1.1 HTTP verbs

我们遵守 RESTFul 风格的接口，尽量也采用 HTTP 的语义规范来设计接口，比如 GET 用来获取数据，POST 用来提交数据。

| Verb     | Usage                                                        |
| -------- | ------------------------------------------------------------ |
| `GET`    | Used to retrieve a resource                                  |
| `POST`   | Used to create a new resource                                |
| `PUT`    | Used to update an existing resource, including partial updates |
| `DELETE` | Used to delete an existing resource                          |

## 1.2 Response status codes

IView API 的 Response 接近于标准的 HTTP Status Code，但是目前只会有三种 HTTP Status Code 情况的返回。不管是正确的 200 返回 还是 错误的4XX和5XX 返回都会被用 HTTP Status Code = 200 包装。在包装的返回 json 体里面通过自定义的 code 来区分返回的内容。

| Status code        | Usage                                                      |
| ------------------ | ---------------------------------------------------------- |
| `200 OK`           | The request completed successfully                         |
| `401 UnAuthorized` | The request was not authorized with provided API key/token |
| `404 Not Found`    | The requested resource did not exist                       |

1. API response example

```
 {
     "code": 0, 
     "message": "ok", 
     "content": { 
         "uuid": "c7a6c549-4bbe-42a4-a46e-3ca9bcbc89c0",
         "createDate": 1551937155000,
         "modifyDate": 1551937157000,
         "entityName": "Projector"
         }
 }
```

| **code**    | 自定义的返回code                                             |
| ----------- | ------------------------------------------------------------ |
| **message** | **code对应的消息说明**                                       |
| **content** | **code对一个的payload，code = 0的时候一般是正确的返回结果， code = 4XX 5XX的时候一般是错误的信息** |

## 1.3 Errors

```json
这里集中列举后台接口返回的Error code 列表，供前端参考错误代表的含义。
    ERROR_400001("unAuth error", 400001),
    ERROR_500001("Sending verification code failed", 500001),
    ERROR_500002("Get sts code error", 500002),
    ERROR_500003("SN already bind by another user", 500003),
    ERROR_500004("SN already unbind by another user", 500004),
    ERROR_500006("SN is not belonging to current user", 500006),
    ERROR_500007("Push message to device failed", 500007),
    ERROR_500008("Push message already acked", 500008),
    ERROR_500009("Duplicated request", 500009),
    ERROR_500010("Telephone already registered", 500010),
    ERROR_999999("Service unavailable now, try later.", 999999);
```

## 1.4 Hypermedia

RESTFul Notes uses hypermedia and resources include links to other resources in their responses. Responses are in [Hypertext Application from resource to resource. Language (HAL)](http://stateless.co/hal_specification.html) format. Links can be found beneath the `_links` key. Users of the API should not create URIs themselves, instead they should use the above-described links to navigate

# 2 Resources

## 2.1 RESTFul API

### 2.1.0 接口根路径(/rest/v1/)

第一个版本的API所以的服务接口路径都在根目录路径下，请求路径之外资源都会返回404错误。

```
除了部分地址不需要token以外, 其余全部接口都需要传入有效token做权限认证。
```

## 2.2 API list

### 2.2.1 user (/rest/v1/user)

#### 1. 给用户的手机发短信验证码

**参数**

| 名字      | 属性   | 必填 | 描述         |
| --------- | ------ | ---- | ------------ |
| telephone | String | Y    | 登录的手机号 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/user/verificationCode' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'telephone=18510255749'
```

##### Http 响应

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
  "code" : 0,
  "message" : "OK",
  "content" : null
}
```



#### 2.用户使用密码注册

**参数**

| 名字      | 属性   | 必填 | 描述             |
| --------- | ------ | ---- | ---------------- |
| telephone | String | Y    | 登录的手机号     |
| code      | String | Y    | 短信收到的验证码 |
| password  | String | Y    | 密码             |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/user/signUp' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'telephone=18510255749&code=123456&password=12345'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Length: 261
Content-Type: application/json;charset=UTF-8

{
  "code" : 0,
  "message" : "OK",
  "content" : {
    "uuid" : "f568ebd0-58d1-4f5c-abf8-0344cba5c39d",
    "createDate" : null,
    "modifyDate" : null,
    "telephone" : "18510255749",
    "apiKey" : "2af29cbd-a7a3-4e87-8cc4-55a91a8a9506",
    "roles" : []
  }
}
```

#### 

#### 3. 用户拿着验证码或者密码来校验登录

**参数**

| 名字      | 属性   | 必填 | 描述             |
| --------- | ------ | ---- | ---------------- |
| telephone | String | Y    | 登录的手机号     |
| code      | String | N  | 短信收到的验证码(两个参数同时传的话，优先使用密码) |
| password  | String | N    | 注册时候的密码(两个参数同时传的话，优先使用密码) |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/user/login' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'telephone=18510255749&code=123456&password=12345'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Length: 261
Content-Type: application/json;charset=UTF-8

{
  "code" : 0,
  "message" : "OK",
  "content" : {
    "uuid" : "f568ebd0-58d1-4f5c-abf8-0344cba5c39d",
    "createDate" : null,
    "modifyDate" : null,
    "telephone" : "18510255749",
    "apiKey" : "2af29cbd-a7a3-4e87-8cc4-55a91a8a9506",
    "roles" : []
  }
}
```


#### 4. 用户找回密码

**参数**

| 名字      | 属性   | 必填 | 描述             |
| --------- | ------ | ---- | ---------------- |
| telephone | String | Y    | 登录的手机号     |
| code      | String | Y    | 短信收到的验证码 |
| password  | String | Y    | 新密码 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/user/password' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'telephone=18510255749&code=123456&password=12345'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Length: 261
Content-Type: application/json;charset=UTF-8

{
  "code" : 0,
  "message" : "OK",
  "content" : {
    "uuid" : "f568ebd0-58d1-4f5c-abf8-0344cba5c39d",
    "createDate" : null,
    "modifyDate" : null,
    "telephone" : "18510255749",
    "apiKey" : "2af29cbd-a7a3-4e87-8cc4-55a91a8a9506",
    "roles" : []
  }
}
```



> 登录验证失败，服务器返回HTTP 401。登录验证成功后，客户端可以使用apiKey加在HTTP头，作为权限认证。  
>
> HTTP header的格式   **'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506'**  

#### 5. 根据token获取用户信息

**Header参数**

| 名字          | 属性   | 必填 | 描述                        |
| ------------- | ------ | ---- | --------------------------- |
| Authorization | String | Y    | TOKEN + 成功登录后返回的key |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/user/details' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Length: 261
Content-Type: application/json;charset=UTF-8

{
  "code" : 0,
  "message" : "OK",
  "content" : {
        "uuid": "222612b0-bb48-4195-87a9-dece84b0bc96",
        "createDate": "2019-03-17T03:09:19.000+0000",
        "modifyDate": "2019-03-17T03:09:19.000+0000",
        "telephone": "18510255749",
        "roles": []
    }
  }
}
```



#### 6. 获取阿里云的STS的token

**Header参数**

| 名字          | 属性   | 必填 | 描述                        |
| ------------- | ------ | ---- | --------------------------- |
| Authorization | String | Y    | TOKEN + 成功登录后返回的key |

##### Curl 请求

```bash
  $ curl 'https://api.iviewdisplays.com/rest/v1/sts' -i -X POST \
      -H 'Content-Type: application/json;charset=UTF-8' \
      -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Length: 261
Content-Type: application/json;charset=UTF-8

{
    "code": 0,
    "message": "OK",
    "content": {
        "securityToken": "CAISkwJ1q6Ft5B2yfSjIr4nHBv/Rh4Vr/LKNan/GokciXf8drLTFhzz2IHtMenlpB+kXsfUznGhU7f8SlqB6T55OSAmcNZIoK0LlaPHkMeT7oMWQweEuuv/MQBquaXPS2MvVfJ+OLrf0ceusbFbpjzJ6xaCAGxypQ12iN+/m6/Ngdc9FHHP7D1x8CcxROxFppeIDKHLVLozNCBPxhXfKB0ca3WgZgGhku6Ok2Z/euFiMzn+Ck79J99Spe8L/NpczZcsvDu3YhrImKvDztwdL8AVP+atMi6hJxCzKpNn1ASMKuE3YY7OKq4wwfVMhPvRhQv5e3/H4lOxlvOvIjJjwyBtLMuxTXj7WWIe62szAFfN8/wz0hHE3UBqAAaGW0gXl9a05YZMqOykMWw5IdHJCJTT1IDtpgPr/QniYMC5pKoSTitZoKubC6pwneR+vnA175WJR+qF4pFnjFJbEvb9Ka7MjCnTvm1nD9zIEsw1OdsaPbJAjUI7qrp/aArOGv3cTbX7gS2X5bRCVGRnhrubXxvIlPMyui6erfOPI",
        "expiration": "2019-03-24T05:40:09Z",
        "errorCode": null,
        "errorMessage": null
    }
}
```

#### 7. transfer账户给另外一个手机号
**参数**

| 名字      | 属性   | 必填 | 描述             |
| --------- | ------ | ---- | ---------------- |
| telephone | String | Y    | 接管账户的新手机号手机号(后台会判断是否已经占用) |
| code | String | Y    | 调用此接口之前，先调用/rest/v1/user/verificationCode接口获得code |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/user/transfer' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'telephone=18510255749&code=123456'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Length: 261
Content-Type: application/json;charset=UTF-8

{
  "code" : 0,
  "message" : "OK",
  "content" : {
    "uuid" : "f568ebd0-58d1-4f5c-abf8-0344cba5c39d",
    "createDate" : null,
    "modifyDate" : null,
    "telephone" : "18510255749",
    "apiKey" : "2af29cbd-a7a3-4e87-8cc4-55a91a8a9506",
    "roles" : []
  }
}
```









### 2.2.2 projector (/rest/v1/projector)

#### 1. App和Logo灯绑定成功后提交绑定信息

**参数**

| 名字      | 属性   | 必填 | 描述                           |
| --------- | ------ | ---- | ------------------------------ |
| sn | String | Y    | 投影灯的物理SN号，要求全局唯一 |
| key | String | Y    | 给设备分发的device key，20Byte以内的一个随意String |
| bleAddress | String | Y    | logo灯的蓝牙地址 |
| deviceType | String | N | 设备类型，用于区分带镜子和不带镜子的logo灯 |
##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/binding' -i -X POST \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'sn=xxx12345&key=123456789&bleAddress=12345&deviceType=00'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "5120bbe1-de99-4318-a68b-bac5c352dd35",
        "createDate": "2019-03-27T12:32:48.000+0000",
        "modifyDate": "2019-03-28T12:59:09.274+0000",
        "sn": "xxx12345",
        "status": "init"
    }
}
```


#### 2. App提交解绑信息

**参数**

| 名字      | 属性   | 必填 | 描述                           |
| --------- | ------ | ---- | ------------------------------ |
| sn | String | Y    | 投影灯的物理SN号，要求全局唯一 |
| code | String | Y    | 调用此接口之前，先调用/rest/v1/user/verificationCode接口获得code |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/unbinding' -i -X POST \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'sn=xxx12345&code=123'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "5120bbe1-de99-4318-a68b-bac5c352dd35",
        "createDate": "2019-03-27T12:32:48.000+0000",
        "modifyDate": "2019-03-28T12:59:13.169+0000",
        "sn": "xxx12345",
        "status": "unbinded"
    }
}
```





#### 3. Logo灯提交解绑信息

**参数**

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| sn   | String | Y    | 投影灯的物理SN号，要求全局唯一 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/projectorUnbinding' -i -X POST \
    -H 'Authorization: KEY 2af29cbd06' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'sn=xxx12345'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "5120bbe1-de99-4318-a68b-bac5c352dd35",
        "createDate": "2019-03-27T12:32:48.000+0000",
        "modifyDate": "2019-03-28T12:59:13.169+0000",
        "sn": "xxx12345",
        "status": "unbinded"
    }
}
```

#### 4. 修改某个logo灯的名字

**参数**

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| sn   | String | Y    | 投影灯的物理SN号，要求全局唯一 |
| name   | String | Y    | 投影灯的新名字 |

##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/projector/name \
  -H 'Authorization: KEY 123456789' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Postman-Token: b2a1362a-0cd3-4d15-80d1-55000c5bd4d3' \
  -H 'cache-control: no-cache' \
  -d 'sn=xxx12345&name=newName'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "5120bbe1-de99-4318-a68b-bac5c352dd35",
        "createDate": "2019-03-27T12:32:48.000+0000",
        "modifyDate": "2019-05-10T10:06:24.000+0000",
        "sn": "xxx12345",
        "name": "newName",
        "wifi": null,
        "status": "init",
        "playState": null
    }
}
```



#### 5. 获取某个logo灯的状态(触发最新状态刺探)

**参数**

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| sn   | String | Y    | 投影灯的物理SN号，要求全局唯一 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/status' -i -X POST \
    -H 'Authorization: KEY 2af29cbd06' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d 'sn=xxx12345'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "5120bbe1-de99-4318-a68b-bac5c352dd35",
        "createDate": "2019-03-27T12:32:48.000+0000",
        "modifyDate": "2019-04-22T07:53:00.000+0000",
        "sn": "xxx12345",
        "status": "init",
        "state": "stopped"
    }
}
```


#### 6. 获取一个logo灯列表里所有logo灯的状态(触发最新状态刺探)

**参数**

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| snList   | String Array | Y    | 投影灯的物理SN号列表，要求非空 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/list/status' -i -X POST \
    -H 'Authorization: KEY 2af29cbd06' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d '{
 		   "snList": [
       		 "xxx12345",
       		 "xxx67890"
    		]
		}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```



#### 7. 给指定的logo灯推送download/delete文件指令-Callback
`logo灯在收到这条推送的时候应该调用本接口{"type":"maintain","action":"status","arguments":{}}`

**参数**

| 名字            | 属性   | 必填 | 描述                                              |
| --------------- | ------ | ---- | ------------------------------------------------- |
| sn              | String | Y    | 设备的sn号                         |
| msgId | String  | Y    | 上一个推送的msgId                  |
| bindingStatus | Enum  | Y    | Logo灯的绑定状态，枚举[unbind,bind]   |
| status | Enum | Y| Logo灯在线状态，枚举[online,offline] |
| playState | Enum | Y | logo灯的播放状态,枚举[playing,paused,stopped] |


##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/status/callback' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
     -d '{
	"sn":"xxx67890",
	"msgId":"20266218872924086",
	"bindingStatus":"bind",
	"status":"online",
	"playState":"paused"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```




#### 8. 获取用户个人名下的所有绑定的Logo灯列表

**分页参数(json格式)**

```json
{
	"page":"0",
	"size":"10"
}
```

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| page | String | Y    | 页码，默认是0页开始 |
| size | String | Y    | 每页的个数，默认是20 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/list' -i -X POST \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d '{
					"page":"0",
					"size":"10"
				}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": [
        {
            "uuid": "5120bbe1-de99-4318-a68b-bac5c352dd35",
            "createDate": "2019-03-27T12:32:48.000+0000",
            "modifyDate": "2019-05-10T10:06:24.000+0000",
            "sn": "xxx12345",
            "name": "newName",
            "wifi": null,
            "status": "init",
            "playState": "stopped"
        },
        {
            "uuid": "bb447f9c-757e-45c6-bb5f-e27dce7d8586",
            "createDate": "2019-04-22T07:58:56.000+0000",
            "modifyDate": "2019-04-22T07:58:56.000+0000",
            "sn": "xxx67890",
            "name": null,
            "wifi": null,
            "status": "init",
            "playState": null
        }
    ]
}
```

```
status枚举：
				init,
        running,
        halted,
        unbinded,
        online,
        offline,
```

```
playState枚举:
        playing,
        paused,
        stopped
```





#### 9. 给指定的logo灯推送download/delete文件指令

**参数**

```json
{
	"sn":"xxx67890",
	"action":"download",
	"uuids":["81f712c7-5486-4267-8564-f47339786d91","7764a601-a604-4ee4-8061-596cc3ec7933"]
}
```

| 名字   | 属性   | 必填 | 描述                                              |
| ------ | ------ | ---- | ------------------------------------------------- |
| sn     | String | Y    | 设备的sn号                                        |
| action | Enum   | Y    | CommandAction枚举类，这里只能是download或者delete |
| uuids  | Array  | Y    | 文件uuid列表数组[]                                |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/maintain' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
     -d '{
	"sn":"xxx67890",
	"action":"download",
	"uuids":["81f712c7-5486-4267-8564-f47339786d91","7764a601-a604-4ee4-8061-596cc3ec7933"]
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```



#### 10. 给指定的logo灯推送download/delete文件指令-Callback

**参数**

| 名字            | 属性   | 必填 | 描述                                              |
| --------------- | ------ | ---- | ------------------------------------------------- |
| sn              | String | Y    | 设备的sn号                                        |
| action          | Enum   | Y    | CommandAction枚举类，这里只能是download或者delete |
| failedUuids | Array  | N    | Logo灯本地操作失败文件名列表数组[]   |
| msgId | String  | Y    | 上一个推送的msgId                  |
| result| Enum | Y| Logo灯下载结果的枚举类，success,failed,partial_success |
| successUuids | Array | N | Logo灯本地操作成功的文件名列表数组[] |


##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/maintain/callback' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
     -d '{
	"sn":"xxx67890",
	"msgId":"9007212187885957",
	"action":"download",
	"failedUuids":[],
	"successUuids":["81f712c7-5486-4267-8564-f47339786d91"],
	"result":"success"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```


#### 11. 获取某个Logo灯的文件列表

**分页参数(json格式)**

```json
{
	"page":"0",
	"size":"10"
}
```

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| sn | String | Y    | pathParam, 在路径里填写设备的sn |
| page | String | Y    | 页码，默认是0页开始 |
| size | String | Y    | 每页的个数，默认是20 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/projector/list/{sn}' -i -X POST \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -d '{
					"page":"0",
					"size":"10"
				}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": [
        {
            "uuid": "7764a601-a604-4ee4-8061-596cc3ec7933",
            "createDate": "2019-04-22T07:40:35.000+0000",
            "modifyDate": "2019-04-22T07:40:35.000+0000",
            "fileName": "test2.png",
            "url": "https://iview-document.oss-cn-shenzhen.aliyuncs.com/user/222612b0-bb48-4195-87a9-dece84b0bc96/test2.png?Expires=1561781003&OSSAccessKeyId=LTAIWpeYO6wXpWmn&Signature=KoI5Q3%2FAmKqk2GMMgT0m5ZwmQMw%3D",
            "type": "image/png",
            "status": "downloading"
        },
        {
            "uuid": "81f712c7-5486-4267-8564-f47339786d91",
            "createDate": "2019-04-22T07:40:18.000+0000",
            "modifyDate": "2019-04-22T07:40:18.000+0000",
            "fileName": "test1.png",
            "url": "https://iview-document.oss-cn-shenzhen.aliyuncs.com/user/222612b0-bb48-4195-87a9-dece84b0bc96/test1.png?Expires=1561781003&OSSAccessKeyId=LTAIWpeYO6wXpWmn&Signature=GTEHEczCUs%2FKOJFTiGSmstjD9qY%3D",
            "type": "image/png",
            "status": "downloaded"
        }
    ]
}
```



#### 12. logo灯状态变化的推送Notification

**参数**

| 名字         | 属性   | 必填 | 描述                                                   |
| ------------ | ------ | ---- | ------------------------------------------------------ |
| sn           | String | Y    | 设备的sn号                                             |
| playState       | Enum   | Y    | 枚举播放状态，[playing,paused,stopped]|
| position     | Array  | Y    | 位置      |
| docUuids | String | Y    | 文件的uuid     |


##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/projector/notification \
  -H 'Authorization: KEY 123456789' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: abb9184d-6ed1-4c6d-afa3-3582bd0b7ad8' \
  -H 'cache-control: no-cache' \
  -d '{
	"sn":"xxx67890",
	"playState":"playing",
	"position":1,
	"docUuids":["7764a601-a604-4ee4-8061-596cc3ec7933"]
}
'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```

####  



### 2.2.3 document (/rest/v1/document)

#### 1. App上传文件到OSS成功后提交文件信息

**参数**

| 名字      | 属性   | 必填 | 描述                           |
| --------- | ------ | ---- | ------------------------------ |
| fileName | String | Y    | 文件名列表， 按照时间_英文名_随机数.格式}  比如201904101130_logo_123.png |

##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/document \
  -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
	"fileNames":["201904101130_logo_123.png"]
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": [
        {
            "uuid": "5c2b231b-3b6f-48f1-90e0-8583e08065b7",
            "createDate": "2019-04-17T06:24:26.138+0000",
            "modifyDate": "2019-04-17T06:24:26.138+0000",
            "fileName": "201904101130_logo_123.png",
            "url": "https://iview-document.oss-cn-shenzhen.aliyuncs.com/user/222612b0-bb48-4195-87a9-dece84b0bc96/201904101130_logo_123.png?Expires=1555485866&OSSAccessKeyId=LTAIWpeYO6wXpWmn&Signature=9tY4nDDWRvowdh1T8AwsmVHoIy8%3D",
            "type": "image/png"
        }
    ]
}
```



#### 2. App删除文件(oss也会删除)

HTTP **DELETE**



**参数**

| 名字  | 属性         | 必填 | 描述   |
| ----- | ------------ | ---- | ------ |
| uuids | Array String | Y    | 文件名 |

##### Curl 请求

```bash
$ curl -X DELETE \
  http://localhost/rest/v1/document \
  -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
	"uuids":["5c2b231b-3b6f-48f1-90e0-8583e08065b7"]
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": [
        {
            "uuid": "5c2b231b-3b6f-48f1-90e0-8583e08065b7",
            "createDate": "2019-04-17T06:24:26.000+0000",
            "modifyDate": "2019-04-17T06:24:26.000+0000",
            "fileName": "cloudProjectionER.png",
            "url": null,
            "type": "image/png"
        }
    ]
}
```

#### 



#### 3. 获取用户个人名下的所有文件列表

**分页参数(json格式)**

```json
{
	"page":"0",
	"size":"10"
}
```

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| page | String | Y    | 页码，默认是0页开始 |
| size | String | Y    | 每页的个数，默认是20 |

##### Curl 请求

```bash
$ curl 'https://api.iviewdisplays.com/rest/v1/document/list' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
    -d '{
					"page":"0",
					"size":"10"
				}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": [
        {
            "uuid": "b9f4ca58-a678-4673-8044-ea363e8c72ab",
            "createDate": "2019-04-10T15:46:52.000+0000",
            "modifyDate": "2019-04-10T15:46:52.000+0000",
            "fileName": "201904101130_logo_123.png",
            "url": "https://iview-document.oss-cn-shenzhen.aliyuncs.com/user/222612b0-bb48-4195-87a9-dece84b0bc96/201904101130_logo_123.png?Expires=1554915200&OSSAccessKeyId=LTAIWpeYO6wXpWmn&Signature=Qi6wzU50nEFSj3iPcK8MCEOzHeg%3D",
            "type": "image/png"
        },
        {
            "uuid": "04a4a927-83e3-4cbb-8e57-1091c7c701e5",
            "createDate": "2019-04-10T15:48:00.000+0000",
            "modifyDate": "2019-04-10T15:48:00.000+0000",
            "fileName": "201904101130_logo_456.png",
            "url": "https://iview-document.oss-cn-shenzhen.aliyuncs.com/user/222612b0-bb48-4195-87a9-dece84b0bc96/201904101130_logo_456.png?Expires=1554915200&OSSAccessKeyId=LTAIWpeYO6wXpWmn&Signature=Qi6wzU50nEFSj3iPcK8MCEOzHeg%3D",
            "type": "image/png"
        }
    ]
}
```









### 2.2.4 Heartbeat (/rest/v1/heartbeat)

#### 1. Logo灯定时上传心跳给服务器

**参数**
```bash
    "sn" : "xxx12345",
	  "registerId":"123",
    "networkOperatorName" : "Unioncom",
    "netType" : null,
    "ipAddress" : "119.6.61.122",
    "macAddress" : "0C:25:76:01:7F:9E",
    "totalRamSpace" : "1006690304",
    "availableRamSpace" : "319229952",
    "totalRomSpace" : "5403115520",
    "availableRomSpace" : "2361106432",
    "cpuUsedRate" : "0",
    "romVersion" : "1.10.1",
    "androidVersion" : "6.0",
    "appVersion" : "V0.1",
    "longitude" : "22.5491840858",
    "latitude" : "113.9465045939",
    "timeStamp" : "2019-04-10T10:41:11.876Z",
    "heartBeatInterval" : "1600"
```

##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/heartbeat \
  -H 'Authorization: KEY 123456789' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
	  "sn" : "xxx12345",
	  "registerId":"123",
    "networkOperatorName" : "Unioncom",
    "netType" : null,
    "ipAddress" : "119.6.61.122",
    "macAddress" : "0C:25:76:01:7F:9E",
    "totalRamSpace" : "1006690304",
    "availableRamSpace" : "319229952",
    "totalRomSpace" : "5403115520",
    "availableRomSpace" : "2361106432",
    "cpuUsedRate" : "0",
    "romVersion" : "1.10.1",
    "androidVersion" : "6.0",
    "appVersion" : "V0.1",
    "longitude" : "22.5491840858",
    "latitude" : "113.9465045939",
    "timeStamp" : "2019-04-10T10:41:11.876Z",
    "heartBeatInterval" : "1600"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "167794de-c498-4f04-b9e8-219ed96e62b3",
        "createDate": "2019-04-16T15:00:06.097+0000",
        "modifyDate": "2019-04-16T15:00:06.097+0000",
        "sn": "P10116CQ00044",
        "userUuid": "222612b0-bb48-4195-87a9-dece84b0bc96"
    }
}
```

### 2.2.5 PlayList (/rest/v1/playlist)

#### 1. 获取一个投影灯的playlist

**参数**
| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| sn   | String | Y    | 投影灯的sn号 |

##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/playlist/ \
  -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'cache-control: no-cache' \
  -d 'sn=xxx67890&undefined='
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "bb447f9c-757e-45c6-bb5f-e27dce7d8587",
        "createDate": "2019-04-22T07:58:56.000+0000",
        "modifyDate": "2019-04-22T07:58:56.000+0000",
        "docUuids": [
            "81f712c7-5486-4267-8564-f47339786d91",
            "5c2b231b-3b6f-48f1-90e0-8583e08065b7"
        ],
        "projectorSn": "xxx67890",
        "status": "valid"
    }
}
```

#### 2. Insert/append/delete/播放模式/排序 操作一个logo灯的playlist,

**参数**

| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| sn   | String | Y    | 投影灯的sn号 |
| action |Enum | Y    | 枚举播放列表操作，[insert,append,delete,playOrder, playMode] |
| positions | intArray | N | 插入/删除位置(插入数组长度应该是1，append不需要传这个参数, 删除时候以默认用postion，忽略docUuids，用于精准删除） |
| docUuids | Array | N   | document id 数组(删除时候可以不传，如果删除时候传此参数以docUuids为准，会把播放列表里包含的docUuids全部删除—目前仅用于文件删除的级联操作) |
| playMode | Enum | N | 播放模式枚举 [sequence, singleCycle] |

##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/playlist/update \
  -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
	  	"sn":"xxx67890",
			"action":"insert",
			"positions":[2],
			"docUuids":["81f712c7-5486-4267-8564-f47339786d91"],
			"playMode":"sequence"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": {
        "uuid": "bb447f9c-757e-45c6-bb5f-e27dce7d8587",
        "createDate": "2019-04-22T07:58:56.000+0000",
        "modifyDate": "2019-04-22T07:58:56.000+0000",
        "docUuids": [
            "5c2b231b-3b6f-48f1-90e0-8583e08065b7",
            "81f712c7-5486-4267-8564-f47339786d91",
            "7764a601-a604-4ee4-8061-596cc3ec7933"
        ],
        "projectorSn": "xxx67890",
        "status": "valid"
    }
}
```

#### 3.  Insert/append/delete/播放模式/排序 操作一个logo灯的playlist回调-Callback

服务器会在回调的结果= success后才真正把结果写入数据库

**参数**

| 名字     | 属性   | 必填 | 描述                                                   |
| -------- | ------ | ---- | ------------------------------------------------------ |
| sn       | String | Y    | 投影灯的sn号                                           |
| msgId | String  | Y    | 上一个推送的msgId                  |
| action   | Enum   | Y    | 枚举播放列表操作，[insert,append,delete]               |
| positions |     |     | 插入/删除位置                                          |
| docUuids | Array  |     | document id 数组                                       |
| result   | Enum   | Y    | Logo灯操作结果的枚举类，success,failed,partial_success |
| playMode | Enum | N | 播放模式枚举 [sequence, singleCycle] |

##### Curl 请求

```bash
$ curl 'http://localhost/rest/v1/playlist/update/callback' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
     -d '{
			"sn":"xxx67890",
			"msgId":"1905162488",
			"action":"play",
			"positions":[1],
			"docUuids":["bb447f9c-757e-45c6-bb5f-e27dce7d8587"]
			"result":"failed",
			"playMode":"sequence"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```

#### 



#### 4. 播放列表纠错，强制把logo灯的播放列表同步到服务器和前端

**参数**

| 名字   | 属性   | 必填 | 描述                                                         |
| ------ | ------ | ---- | ------------------------------------------------------------ |
| sn     | String | Y    | 投影灯的sn号                                                 |
| action | Enum   | Y    | 枚举播放列表操作，新增了forceUpdate [insert,append,delete,playOrder, playMode, forceUpdate] 这个接口用forceUpdate |

##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/playlist/forceUpdate \
  -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
			"sn":"xxx67890",
			"action":"forceUpdate"
	}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": true
}
```

#### 5. 播放列表纠错，强制把logo灯的播放列表同步到服务器和前端-Callback

**参数**

| 名字      | 属性   | 必填 | 描述                                                   |
| --------- | ------ | ---- | ------------------------------------------------------ |
| sn        | String | Y    | 投影灯的sn号                                           |
| msgId     | String | Y    | 上一个推送的msgId                                      |
| action    | Enum   | Y    | forceUpdate                                            |
| playState | Enum   | N    | 播放状态                                               |
| docUuids  | Array  | Y    | document id 数组                                       |
| result    | Enum   | Y    | Logo灯操作结果的枚举类，success,failed,partial_success |
| playMode  | Enum   | N    | 播放模式枚举 [sequence, singleCycle]                   |

##### Curl 请求

```bash
$ curl 'http://localhost/rest/v1/playlist/forceUpdate/callback' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
     -d '{
	"sn":"xxx67890",
	"msgId":"9007214735739597",
	"action":"forceUpdate",
	"playMode":"sequence",
	"playState":"playing",
	"docUuids":["30bccfd4-697f-4710-8637-3adc78a5e33a","81f712c7-5486-4267-8564-f47339786d91"],
	"result":"success"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```





#### 6. play/stop/pause/previous/next操作一个playlist

**参数**
| 名字 | 属性   | 必填 | 描述                           |
| ---- | ------ | ---- | ------------------------------ |
| sn   | String | Y    | 投影灯的sn号 |
| action |Enum | Y    | 枚举播放操作，[play,stop,pause,previous,next] |
| position | int | Y    | 操作位置 |
| playListUuid | String | Y    | 播放列表的uuid |

##### Curl 请求

```bash
$ curl -X POST \
  http://localhost/rest/v1/playlist/operation \
  -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
	  	"sn":"xxx67890",
			"action":"play",
			"position":1,
			"playListUuid":"bb447f9c-757e-45c6-bb5f-e27dce7d8587"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": null
}
```



#### 7. logo灯Playlist操作的实时回调-Callback

**参数**

| 名字         | 属性   | 必填 | 描述                                                   |
| ------------ | ------ | ---- | ------------------------------------------------------ |
| sn           | String | Y    | 设备的sn号                                             |
| msgId | String  | Y    | 上一个推送的msgId                  |
| action       | Enum   | Y    | 枚举播放操作，[play,stop,pause,previous,next]          |
| position     | Array  | Y    | 操作位置                                               |
| playListUuid | String | Y    | 播放列表的uuid                                         |
| result       | Enum   | Y    | docUuidsLogo灯操作结果的枚举类，success,failed,partial_success |
| playState       | Enum   | Y    | 枚举播放状态，[playing,paused,stopped]|

##### Curl 请求

```bash
$ curl 'http://localhost/rest/v1/playlist/operation/callback' -i -X POST \
    -H 'Content-Type: application/json;charset=UTF-8' \
    -H 'Authorization: TOKEN 2af29cbd-a7a3-4e87-8cc4-55a91a8a9506' \
     -d '{
			"sn":"xxx67890",
      "msgId":"1905162489",
			"action":"play",
			"position":1,
			"playListUuid":"bb447f9c-757e-45c6-bb5f-e27dce7d8587"
			"result":"failed",
			"playState":"stopped"
}'
```

##### Http 响应

```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Content-Length: 56

{
    "code": 0,
    "message": "OK",
    "content": success
}
```

####  



#### 	