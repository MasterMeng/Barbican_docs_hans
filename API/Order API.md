# Order API 参考  

# GET /v1/orders  
列出项目的订单。  
可以通过传递URL来过滤订单列表。  

## 参数
| 名称 | 数据类型 | 说明 |
| --- | --- | --- |
| offset|int|待检索信息在总订单列表中的起始索引|
|limit|int|要返回的最大记录数（最大100个），默认限制为10|   

## 请求  
```
GET /v1/orders
Headers:
    Content-Type: application/json
    X-Auth-Token: {token}
```

## 响应  
```
200 Success

{
    "orders": [
    {
        "created": "2015-10-20T18:38:44",
        "creator_id": "40540f978fbd45c1af18910e3e02b63f",
        "meta": {
            "algorithm": "AES",
            "bit_length": 256,
            "expiration": null,
            "mode": "cbc",
            "name": "secretname",
            "payload_content_type": "application/octet-stream"
        },
        "order_ref": "http://localhost:9311/v1/orders/2284ba6f-f964-4de7-b61e-c413df5d1e47",
        "secret_ref": "http://localhost:9311/v1/secrets/15dcf8e4-3138-4360-be9f-fc4bc2e64a19",
        "status": "ACTIVE",
        "sub_status": "Unknown",
        "sub_status_message": "Unknown",
        "type": "key",
        "updated": "2015-10-20T18:38:44"
    },
    {
        "created": "2015-10-20T18:38:47",
        "creator_id": "40540f978fbd45c1af18910e3e02b63f",
        "meta": {
            "algorithm": "AES",
            "bit_length": 256,
            "expiration": null,
            "mode": "cbc",
            "name": "secretname",
            "payload_content_type": "application/octet-stream"
        },
        "order_ref": "http://localhost:9311/v1/orders/87b7169e-3aa2-4cb1-8800-b5aadf6babd1",
        "secret_ref": "http://localhost:9311/v1/secrets/80183f4b-c0de-4a94-91ad-6d55251acee2",
        "status": "ACTIVE",
        "sub_status": "Unknown",
        "sub_status_message": "Unknown",
        "type": "key",
        "updated": "2015-10-20T18:38:47"
    }
    ],
    "total": 2
}
```

## 响应属性  
|名称|类型|说明|
|---|---|---|
|orders|list|包含填充订单元数据的词典列表|
|total|int|用户可用的订单总数|
|next|string|一个HATEOAS URL，用于根据offset和limit参数来获取下一组对象集合。该属性只有在消费者总数大于offset和limit参数组合时可用|
|previous|string|一个HATEOAS URL，用于根据offset和limit参数来获取上一组对象集合。该属性只有在offset大于0时可用|  

## HTTP状态码  
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或无权访问此资源|  

# POST /v1/orders  
创建一个订单。  

## 参数  
|参数名称|类型|说明|默认值|
|--|--|--|--|
|type|string|要生成的密钥类型。有效类型：key（对称密钥），asymmetric（非对称密钥）和certificate（证书）。|无|
|meta|dict|包含用来创建机密的机密元数据字典。|无|

## 请求  
```
POST /v1/orders
Headers:
    Content-Type: application/json
    X-Auth-Token: {token}

Content:
{
    "type":"key",
    "meta":
        {
            "name":"secretname",
            "algorithm": "AES",
            "bit_length": 256,
            "mode": "cbc",
            "payload_content_type":"application/octet-stream"
        }
}
```

## 响应  
```
202 Created

{
    "order_ref": "http://{barbican_host}/v1/orders/{order_uuid}"
}
```

## 响应属性  
|名称|类型|说明|
|--|--|--|
|order_ref|string|订单引用|  

## HTTP状态码  
|code|说明|
|--|--|
|202|成功创建|
|400|错误的请求|
|401|无效的X-Auth-Token或用户无权访问此资源|
|415|不支持的媒体类型|

# GET /v1/orders/{uuid}  
检索订单元数据  

## 请求
```
GET /v1/orders/{order_uuid}
Headers:
    Accept: application/json
    X-Auth-Token: {token}
```

## 参数  
无  

## 响应  
```
200 Success

{
    "created": "2015-10-20T18:49:02",
    "creator_id": "40540f978fbd45c1af18910e3e02b63f",
    "meta": {
        "algorithm": "AES",
        "bit_length": 256,
        "expiration": null,
        "mode": "cbc",
        "name": "secretname",
        "payload_content_type": "application/octet-stream"
    },
    "order_ref": "http://localhost:9311/v1/orders/5443d349-fe0c-4bfd-bd9d-99c4a9770638",
    "secret_ref": "http://localhost:9311/v1/secrets/16f8d4f3-d3dd-4160-a5bd-8e5095a42613",
    "status": "ACTIVE",
    "sub_status": "Unknown",
    "sub_status_message": "Unknown",
    "type": "key",
    "updated": "2015-10-20T18:49:02"
}
```

## 响应属性  
|名称|类型|说明|
|--|--|--|
|created|string|ISO8601格式的创建时间戳|
|creator_id|string|创建者的Keystone id|
|meta|dict|用于创建信息的机密元数据|
|order_ref|string|与订单相关的订单引用|
|secret_ref|string|与订单相关的机密引用|
|status|string|订单的状态|
|sub_status|string|与订单相关的元数据|
|sub_status_messages|string|与订单相关的元数据|
|type|string|订单类型|
|updated|string|ISO8601格式的最后更新时间戳|

## HTTP状态码
|code|说明|
|--|--|--|
|200|成功检索订单|
|400|错误的请求|
|401|无效的X-Auth-Token或用户无权访问此资源|
|404|未找到|

# DELETE /v1/orders/{uuid}  
删除订单  

## 请求  
```
DELETE /v1/orders/{order_uuid}
Headers:
    X-Auth-Token: {token}
```

## 参数  
无

## 响应  
```
204 Success
```

## HTTP状态码
|code|说明|
|--|--|--|
|204|成功删除订单|
|400|错误的请求|
|401|无效的X-Auth-Token或用户无权访问此资源|
|404|未找到|