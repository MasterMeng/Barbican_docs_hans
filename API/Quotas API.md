# Quotas API 参考  

# GET /v1/quotas  
获取请求者项目的有效配额。请求者的项目ID是从X-Auth-Token头中提供的身份验证令牌中获取的。  

## 请求/响应  
```
Request:

  GET /v1/quotas
  Headers:
    X-Auth-Token:<token>
    Accept: application/json


Response:

  HTTP/1.1 200 OK
  Content-Type: application/json

  {
    "quotas": {
      "secrets": 10,
      "orders": 20,
      "containers": 10,
      "consumers": -1,
      "cas": 5
    }
  }
```

## 响应属性  
|名称|类型|说明|
|--|--|--|
|quotas|dict|一组包含配额信息的字典|
|secrets|int|包含机密资源的当前项目的有效配额值|
|orders|int|包含订单资源的当前项目的有效配额值|
|containers|int|包含容器资源的当前项目的有效配额值|
|consumers|int|包含消费者资源的当前项目的有效配额值|
|cas|int|包含证书资源的当前项目的有效配额值|  
有效配置的解释如下：  
|值|说明|
|--|--|
|-1|负数表示资源不受配额限制|
|0|零表示资源已禁用|
|int|正数表示当前项目可被创建的最大资源数|  

## HTTP状态码  
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或无权访问此资源|  

# GET /v1/project-quotas  
获取已配置项目的配额记录的列表。使用可选参数offset和limit可支持分页。  

## 请求/响应  
```
Request:

  GET /v1/project-quotas
  Headers:
    X-Auth-Token:<token>
    Accept: application/json

Response:

  200 OK

  Content-Type: application/json

  {
    "project_quotas": [
      {
        "project_id": "1234",
        "project_quotas": {
             "secrets": 2000,
             "orders": 0,
             "containers": -1,
             "consumers": null,
             "cas": null
         }
      },
      {
        "project_id": "5678",
        "project_quotas": {
             "secrets": 200,
             "orders": 100,
             "containers": -1,
             "consumers": null,
             "cas": null
         }
      },
    ],
    "total" : 30,
  }
```

## 参数  
|名称|类型|说明|
|--|--|--|
|offset|int|你想检索的项目配额在总列表中的起始索引。|
|limit|int|要返回的最大记录数|  

## 响应属性  
|名称|类型|说明|
|--|--|--|
|project-id|string|已配置配额信息的项目的UUID|
|project-quotas|dict|包含项目配额信息的字典|
|secrets|int|包含机密资源的当前项目的有效配额值|
|orders|int|包含订单资源的当前项目的有效配额值|
|containers|int|包含容器资源的当前项目的有效配额值|
|consumers|int|包含消费者资源的当前项目的有效配额值|
|cas|int|包含证书资源的当前项目的有效配额值|  
|total|int|已配置项目配额的总记录数|
|next|string|一个HATEOAS URL，用于根据offset和limit参数来获取下一组配额集合。该属性只有在机密总数大于offset和limit参数组合时可用|
|previous|string|一个HATEOAS URL，用于根据offset和limit参数来获取上一组配额集合。该属性只有在offset大于0时可用|  

已配置的项目配额值的解释如下：
|值|说明|
|--|--|
|-1|负数表示资源不受配额限制|
|0|零表示资源已禁用|
|int|正数表示当前项目可被创建的最大资源数|
|null|null表示使用当前项目的默认配额|  

## HTTP状态码  
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或无权访问此资源|  

# GET /v1/project-quotas/{uuid}  
检索已配置项目的项目配额信息。  

## 请求/响应  
```
Request:

  GET /v1/project-quotas/{uuid}
  Headers:
    X-Auth-Token:<token>
    Accept: application/json


Response:

  200 OK

  Content-Type: application/json

  {
    "project_quotas": {
      "secrets": 10,
      "orders": 20,
      "containers": -1,
      "consumers": 10,
      "cas": 5
    }
  }
```  

## 响应属性  
|名称|类型|说明|
|--|--|--|
|project-quotas|dict|包含项目配额信息的字典|
|secrets|int|包含机密资源的当前项目的有效配额值|
|orders|int|包含订单资源的当前项目的有效配额值|
|containers|int|包含容器资源的当前项目的有效配额值|
|consumers|int|包含消费者资源的当前项目的有效配额值|
|cas|int|包含证书资源的当前项目的有效配额值|  

## HTTP状态码  
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或无权访问此资源|
|404|未找到。请求的项目未配置任何配额|  

# PUT /v1/project-quotas/{uuid}  
为执行UUID的项目创建或更新已配置的项目配额。  

## 请求/响应  
```
Request:

  PUT /v1/project-quotas/{uuid}
  Headers:
    X-Auth-Token:<token>
    Content-Type: application/json

  Body::

    {
      "project_quotas": {
        "secrets": 50,
        "orders": 10,
        "containers": 20
      }
    }


Response:

  204 OK
```

## 响应属性  
|名称|类型|说明|
|--|--|--|
|project-quotas|dict|包含项目配额信息的字典|
|secrets|int|为此项目设置的机密配额的值|
|orders|int|为此项目设置的订单配额的值|
|containers|int|为此项目设置的容器配额的值|
|consumers|int|为此项目设置的消费者配额的值|
|cas|int|为此项目设置的证书配额的值|  

已配置的项目配额值的解释如下：
|值|说明|
|--|--|
|-1|负数表示资源不受配额限制|
|0|零表示资源已禁用|
|int|正数表示当前项目可被创建的最大资源数|
||如果未给定资源数值，表示使用当前项目的默认配额|  

## HTTP状态码  
|code|说明|
|--|--|
|200|成功|
|400|错误的请求|
|401|无效的X-Auth-Token或无权访问此资源|  

# DELETE /v1/project-quotas/{uuid}  
删除请求UUID的项目的项目配额配置。当配置的项目配额被删除，将使用此项目的默认配额。  

## 请求/响应  
```
Request:

  DELETE v1/project-quotas/{uuid}
  Headers:
    X-Auth-Token:<token>


Response:

  204 No Content
```

## HTTP状态码  
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或无权访问此资源|
|404|未找到|  