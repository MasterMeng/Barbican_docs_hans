# 密钥管理服务升级指南  
本文档列出了操作员在从之前版本的OpenStack中升级Barbican时的几个步骤和注意事项。  

# 计划升级  
* 升级Barbican服务前应该仔细阅读**发行说明**。从Mitaka版本开始，发行说明中详细记录了特定的升级步骤和注意事项。  
* 仅支持顺序版本之间的升级  
* 升级时，应该按照以下步骤：
&emsp;&emsp;1. 销毁所有Barbican服务  
&emsp;&emsp;2. 更新源码到下一个发行版
&emsp;&emsp;3. 升级barbican数据库到下一个发行版  
```
        barbican-db-manage upgrade
```  
&emsp;&emsp;4. 重启barbican服务  

# 从Newton升级到Ocata  
粘贴管道配置文件*barbican-api-paste.ini* 已被添加到*http_proxy_to_wsgi* 中间件中。 当它被放在TLS代理（例如HAProxy）之后，该中间件可以帮助barbican使用正确的URL引用响应请求。默认情况下，该中间件被禁用，但可以通过*oslo_middleware*组中的配置选项启用。  
详情参阅<u>Ocate发行说明</u>  

# 从Mitaka升级到Newton  
此次升级没有额外的注意事项。  
详情参阅<u>Newton发行说明</u>   

# 从Liberty升级到Mitaka  
元数据API需要更新数据库架构。升级到Mitaka的现有部署应该使用‘**barbican-manage**’工具来升级架构。  
如果你想升级之前使用**PKCS#11加密插件驱动**的barbican版本，你应该执行数据迁移插件。  
```
python barbican/cmd/pkcs11_migrate_kek_signatures.py
```  
详情参阅<u>Mitaka发行说明</u>