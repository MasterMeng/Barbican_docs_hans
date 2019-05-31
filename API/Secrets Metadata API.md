# Secrets Metadata API 参考  

# GET /v1/secrets/{uuid}/metadata  
列出用户定义的机密的元数据。  
如果机密并未包含任何用户元数据，将返回一个空列表。  
## 请求  
```
GET /v1/secrets/{uuid}/metadata
Headers:
    Accept: application/json
    X-Auth-Token: <token>
```  

## 响应  
```
{
  'metadata': {
    'description': 'contains the AES key',
    'geolocation': '12.3456, -98.7654'
    }
}
```  
## 响应属性  
|名称|类型|说明|
|--|--|--|
|metadata|list|包含机密元数据的键/值对的列表。用户提供的键必须是小写字母，否则将被转换为小写字母。|

## HTTP状态码
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权创建。原因在于用户的角色或项目的配额|
|404|未找到|  

# PUT /v1/secrets{uuid}/metadata  
为机密设置元数据。任何原有的元数据都将被删除，并被新设置的元数据替代。  

## 参数  
|名称|类型|说明|
|--|--|--|
|metadata|list|包含机密元数据的键/值对的列表。用户提供的键必须是小写字母，否则将被转换为小写字母。|

## 请求  
```
PUT /v1/secrets/{uuid}/metadata
Headers:
    Content-Type: application/json
    X-Auth-Token: <token>

Content:
{
  'metadata': {
      'description': 'contains the AES key',
      'geolocation': '12.3456, -98.7654'
    }
}
```  

## 响应  
```
201 OK
{
    "metadata_ref": "https://{barbican_host}/v1/secrets/{secret_uuid}/metadata"
}
```

## HTTP状态码  
|code|说明|
|--|--|
|200|成功创建/更行机密元数据|
|400|错误的请求|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权创建。原因在于用户的角色或项目的配额|

# GET /v1/secrets/{uuid}/metadata/{key}  
检索用户添加的元数据。  

## 请求  
```
GET /v1/secrets/{uuid}/metadata/{key}
Headers:
    Accept: application/json
    X-Auth-Token: <token>
```

## 响应  
```
200 OK
{
  "key": "access-limit",
  "value": "0"
}
```

## HTTP状态码
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权创建。原因在于用户的角色或项目的配额|
|404|未找到|  

# POST /v1/secrets/{uuid}/metadata/  
添加一组新的键/值对到机密信息的用户元数据中。提供的键必须是不存在于现有元数据中的。键必须是小写字母，不然会自动转为小写字母。  

## 请求  
```
POST /v1/secrets/{uuid}/metadata/
Headers:
    X-Auth-Token: <token>
    Content-Type: application/json

Content:
  {
    "key": "access-limit",
    "value": "11"
  }
```

## 响应  
```
201 Created
Secret Metadata Location: http://example.com:9311/v1/secrets/{uuid}/metadata/access-limit
  {
    "key": "access-limit",
    "value": "11"
  }
```

## HTTP状态码
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权创建。原因在于用户的角色或项目的配额|
|409|冲突。所提供的键已存在|

# PUT /v1/secrets/{uuid}/metadata/{key}  
更新机密信息用户元数据中已存在的键/值对。提供键必须是存在于现有元数据中的。键必须是小写字母，不然会自动转为小写字母。  

## 请求  
```
PUT /v1/secrets/{uuid}/metadata/{key}
Headers:
    X-Auth-Token: <token>
    Content-Type: application/json

Content:
  {
    "key": "access-limit",
    "value": "11"
  }
```

## 响应  
```
200 OK

{
  "key": "access-limit",
  "value": "11"
}
```

## HTTP状态码
|code|说明|
|--|--|
|200|成功|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权创建。原因在于用户的角色或项目的配额|
|404|未找到|

# DELETE /v1/secrets/{uuid}/metadata/{key}  
通过键删除机密元数据。  

## 请求  
```
DELETE /v1/secrets/{uuid}/metadata/{key}
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
|200|成功|
|401|无效的X-Auth-Token或者此token无权访问该资源|
|403|访问被拒绝。用户已经通过验证，但是无权创建。原因在于用户的角色或项目的配额|
|404|未找到|