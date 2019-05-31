# 在Barbican中使用KeyStone中间件  

# 先决条件  
要使Keystone与Barbican联通，你需要使用对应版本的Keystone。如果你安装的是全服务的OpenStack云，包括Keystone和Barbican，那就不需要其他操作了，因为它们是一起发布的。如果你没有可用的Keystone实例，那么你可以通过以下几种方法安装配置自己的Keystone实例：  
1. [简单的Dockerized Keystone](https://registry.hub.docker.com/u/jmvrbanac/simple-keystone/)  
2. [安装Keystone](https://docs.openstack.org/keystone/latest/install/index.html)  
3. 附带Keystone的OpenStack云  

# 如何联通Barbican和Keystone  
如果您硬安装好了Keystone实例，那么连接Barbican和Keystone是相当简单的。完成后，Barbican应该要求使用者提供有效的X-Auth-Token以及除了获取版本调用之外的所有API调用。  
1. 关闭所有激活的Barbican实例。  
2. 编辑 **/etc/barbican/barbican-api-paste.ini**文件  
&nbsp;&nbsp;&nbsp;&nbsp; a. 将 **/v1**管道的值从无认证的 **barbican_api**修改为带认证的 **barbican-api-keystone**。  
&nbsp;&nbsp;&nbsp;&nbsp; 如果OpenStack是Newton或更高的版本，那么这一步就不是必须的，因为从Newton，barbican默认使用Keystone认证。  
```text
        [composite:main]
        use = egg:Paste#urlmap
        /: barbican_version
        /v1: barbican-api-keystone
```  

&nbsp;&nbsp;&nbsp;&nbsp; b. 替换 **authtoken**的值来匹配你的Keystoneshez。  
```text
        [filter:authtoken]
        paste.filter_factory =                    keystonemiddleware.auth_token:filter_factory
        auth_plugin = password
        username = {YOUR_KEYSTONE_USERNAME}
        password = {YOUR_KEYSTONE_PASSWORD}
        user_domain_id = {YOUR_KEYSTONE_USER_DOMAIN}
        project_name = {YOUR_KEYSTONE_PROJECT}
        project_domain_id = {YOUR_KEYSTONE_PROJECT_DOMAIN}
        www_authenticate_uri = http://{YOUR_KEYSTONE_ENDPOINT}:5000/v3
        auth_url = http://{YOUR_KEYSTONE_ENDPOINT}:5000/v3
```  
或者，你可以将其简写为：  
```text
        [filter:authtoken]
        paste.filter_factory = keystonemiddleware.auth_token:filter_factory
```
并将Barbican的Keystone凭证存储在 **/etc/barbican/barbican.conf**中 **[keystone_authtoken]**部分中：  
```text
        [keystone_authtoken]
        auth_plugin = password
        username = {YOUR_KEYSTONE_USERNAME}
        password = {YOUR_KEYSTONE_PASSWORD}
        user_domain_id = {YOUR_KEYSTONE_USER_DOMAIN}
        project_name = {YOUR_KEYSTONE_PROJECT}
        project_domain_id = {YOUR_KEYSTONE_PROJECT_DOMAIN}
        www_authenticate_uri = http://{YOUR_KEYSTONE_ENDPOINT}:5000/v3
        auth_url = http://{YOUR_KEYSTONE_ENDPOINT}:5000/v3
```

3. 启动Barbican服务  
**{barbican_home}/bin/barbican.sh start**