# Barbican安装过程中遇到的问题  
如果你在本文中找不到你想找的答案，你可以在Freenode IRC频道 **#openstack-barbican**提问。  

# 成功同构Keystone认证后Barbican收到HTTP 401错误  

## 你可能看到的  
即使你使用有效的token还是得到 HTTP 401 无权限的响应  
```bash
    curl -X POST -H "X-Auth-Token: $TOKEN" -H "Content-type: application/json" -d '{"payload": "my-secret-here", "payload_content_type": "text/plain"}' http://localhost:9311/v1/secrets
```  

## 原因  
Barbican服务的签名证书已过期  

## 如何避免  
检查Barbican服务上过期的Keystone签名证书。可从 **/tmp/barbican/cache/signing_cert.pem**中查看过期时间。如果证书过期，请执行以下步骤。
&nbsp;&nbsp;&nbsp;&nbsp; 1. 在Keystone服务器上，验证签名证书是不是同样过期。通常你可以在你的Keystone服务器 **/etc/keystone/ssl/certs**目录下找到 **signing_cert.pem**。  
&nbsp;&nbsp;&nbsp;&nbsp; 2. 如果证书过期，那么执行以下操作生成新的证书：  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a. 从Barbican和Keystone上删除证书。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; b. 编辑 **/etc/keystone/ssl/certs/index.txt.attr**，设置 **unique_subject**为 **no**。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; c. 在你下次调用Barbican API时，证书将下载到你的Barbican服务器上。  
&nbsp;&nbsp;&nbsp;&nbsp; 3. 如果证书没过期，那么删除Barbican服务器上的证书。千万不要删除Keystone服务器上的证书。下次调用Barbican API时，证书将从Keystone服务器上下载到你的Barbican服务器上。  

# 返回的引用使用localhost而不是正确的主机名  

## 你可能看到的  
```bash
    curl -X POST -H "X-Auth-Token: $TOKEN" -H "Content-type: application/json" -d '{"payload": "my-secret-here", "payload_content_type": "text/plain"}' http://myhostname.com/v1/secrets

    # Response:
    {
        "secret_ref": "http://localhost:9311/v1/secrets/UUID_HERE"
    }
```

## 原因   
响应主机名的默认配置不会修改端点的主机名（通常是负载均衡器的DNS名称和端口）。  

## 如何避免  
将 **barbican.conf**文件中的 **host_href**从 **localhost:9311**更改为正确的主机名。  

# 
