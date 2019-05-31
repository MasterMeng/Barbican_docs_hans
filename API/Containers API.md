# Containers API参考  

# GET /v1/container  
列出项目的所有容器。  
返回的容器将按创建时间进行排序：从最开始到最近。  

## 参数  
| 名称 | 数据类型 | 说明 |
| --- | --- | --- |
| offset|int|待检索信息在总列表中的起始索引|
|limit|int|要返回的最大记录数（最大100个），默认限制为10|

## 响应属性  
|名称|类型|说明|
|---|---|---|
|containers|list|包含填充容器数据的词典列表|
|total|int|用户可用的容器总数|
|next|string|一个HATEOAS URL，用于根据offset和limit参数来获取下一组容器。该属性只有在容器总数大于offset和limit参数组合时可用|
|previous|string|一个HATEOAS URL，用于根据offset和limit参数来获取上一组容器。该属性只有在offset大于0时可用|  

## 请求  
```
GET /v1/containers
Headers:
    X-Auth-Token: <token>
```  

## 响应  
```
{
    "containers": [
        {
            "consumers": [],
            "container_ref": "https://{barbican_host}/v1/containers/{uuid}",
            "created": "2015-03-26T21:10:45.417835",
            "name": "container name",
            "secret_refs": [
                {
                    "name": "private_key",
                    "secret_ref": "https://{barbican_host}/v1/secrets/{uuid}"
                }
            ],
            "status": "ACTIVE",
            "type": "generic",
            "updated": "2015-03-26T21:10:45.417835"
        }
    ],
    "total": 1
}
```  

## HTTP状态码
|code|说明|
|--|--|
|200|成功的请求|
|401|无效的X-Auth-Token或者此token无权访问该资源| 

# GET /v1/containers/{uuid}  
检索单个容器  

## 响应属性
|名称|类型|说明|
|---|---|---|
|name|string|（可选项）可读的容器名称|
|tyep|string|容器类型。可选：generic,rsa,certificate|
|secret_refs|list|容器中存储的机密引用的词典列表|  

## 请求  
```
GET /v1/containers/{uuid}
Headers:
    X-Auth-Token: <token>
```  

## 响应  
```
{
    "type": "generic",
    "status": "ACTIVE",
    "name": "container name",
    "consumers": [],
    "container_ref": "https://{barbican_host}/v1/containers/{uuid}",
    "secret_refs": [
        {
            "name": "private_key",
            "secret_ref": "https://{barbican_host}/v1/secrets/{uuid}"
        }
    ],
    "created": "2015-03-26T21:10:45.417835",
    "updated": "2015-03-26T21:10:45.417835"
}
```

## HTTP状态码  
|code|说明|
|--|--|
|200|成功的请求|
|401|无效的X-Auth-Token或者此token无权访问该资源| 
|404|容器未找到或容器不可用|

# POST /v1/containers  
创建一个容器。  
可以创建三种不同类型的容器：generic，rsa，certificate。  

## Generic  
此类型的容器可以存储任意数量的机密引用。每一个机密引用都附有一个名称。与其他容器不同，此类型不对内容、名称等属性进行特定限制。  

## RSA  
此类型中仅包含三中不同的机密引用，且对引用的名称有强制限制：public_key，private_key和private_key_passphrase。  

## Certificate  
此类型被设计用来存储证书，也可用来存public_key，private_key，private_key_passphrase和中间人。  

## 请求属性
|名称|类型|说明|
|---|---|---|
|name|string|（可选项）可读的容器名称|
|tyep|string|容器类型。可选：generic,rsa,certificate|
|secret_refs|list|容器中存储的机密引用的词典列表|  

## 请求  
```
POST /v1/containers
Headers:
    X-Auth-Token: <token>

Content:
{
    "type": "generic",
    "name": "container name",
    "secret_refs": [
        {
            "name": "private_key",
            "secret_ref": "https://{barbican_host}/v1/secrets/{secret_uuid}"
        }
    ]
}
```

## 响应  
```
{
    "container_ref": "https://{barbican_host}/v1/containers/{container_uuid}"
}
```

## HTTP状态码
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权执行此操作。跟用户角色或项目配额有关。|  

# DELETE /v1/containers/{uuid}  
删除指定容器  

## 请求  
```
DELETE /v1/containers/{container_uuid}
Headers:
    X-Auth-Token: <token>
```  

## 响应  
```
204 No Content
```  

## HTTP状态码
|code|说明|
|--|--|
|204|成功删除|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|404|容器未找到或不可用|  


# POST /v1/containers/{container_uuid}/secrets  
向已存在的容器中添加机密，仅generic容器支持。  

## 请求属性 
|名称|类型|说明|
|---|---|---|
|name|string|（可选项）可读的容器名称|
|secret_ref|uri|（必选项）已有机密的全URI引用|  

## 请求
```
POST /v1/containers/{container_uuid}/secrets
Headers:
    X-Project-Id: {project_id}

Content:
{
    "name": "private_key",
    "secret_ref": "https://{barbican_host}/v1/secrets/{secret_uuid}"
}
```  

## 响应  
```
{
    "container_ref": "https://{barbican_host}/v1/containers/{container_uuid}"
}
```

## HTTP状态码  
通常，容器POST调用产生的错误代码也在这里，特别是关于提供的机密引用。
|code|说明|
|--|--|
|201|成功更新容器|
|400|缺少secret_ref|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权执行此操作。跟用户角色或项目配额有关。|

# DELETE /v1/containers/{container_uuid}/secrets  
从容器中删除机密。仅generic容器支持。  

## 请求属性  
|名称|类型|说明|
|---|---|---|
|name|string|（可选项）可读的容器名称|
|secret_ref|uri|（必选项）已有机密的全URI引用|  

## 请求  
```
DELETE /v1/containers/{container_uuid}/secrets
Headers:
    X-Project-Id: {project_id}

Content:
{
    "name": "private key",
    "secret_ref": "https://{barbican_host}/v1/secrets/{secret_uuid}"
}
```

## 响应  
```
204 No Content
```

## HTTP状态码  
通常，容器POST调用产生的错误代码也在这里，特别是关于提供的机密引用。
|code|说明|
|--|--|
|201|成功更新容器|
|400|缺少secret_ref|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权执行此操作。跟用户角色或项目配额有关。|
|404|容器中未找到指定的secret_ref|