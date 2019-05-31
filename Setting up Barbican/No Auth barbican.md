# 无认证Barbican  
从OpenStack Newton版本开始，Barbican，像其它的OpenStack服务一样，默认使用Keystone进行身份认证和访问控制。尽管如此，有时候在开发环境下使用无认证服务的Barbican更方便。  

为解决这个问题，**barbican-api-paste.ini**包含了一个无认证的过滤管道：  
```text
    # Use this pipeline for barbican API - DEFAULT no authentication
    [pipeline:barbican_api]
    pipeline = unauthenticated-context apiapp
```  

要使用这个管道，配置如下：  
&nbsp;&nbsp;&nbsp;&nbsp; 1. 关闭所有Barbican实例。  
&nbsp;&nbsp;&nbsp;&nbsp; 2. 编辑 **/etc/barbican/barbican-api-paste.ini**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 将 **/v1**管道的值从无认证的 **barbican-api-keystone**修改为带认证的 **barbican_api**。  
```text
    [composite:main]
    use = egg:Paste#urlmap
    /: barbican_version
    /v1: barbican_api
```  

因为每一个OpenStack服务都与Keystone集成，其API需要访问令牌才能检索某些信息并验证用户的信息和权限。如果你使用无认证服务的Barbican，那么必须指定是从 **project_id**而不是从令牌中检索访问令牌。这种情形下的API，使用 **‘X-Project-Id:{project_id}'**替换 <u>Barbican API文档</u>中 **‘X-Auth-Token:$TOKEN'**。