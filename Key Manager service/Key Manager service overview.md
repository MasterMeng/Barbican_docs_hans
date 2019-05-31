# 密钥管理服务概述  
密钥管理服务提供机密的安全存储、配置和管理，例如密码，加密密钥。  

密钥管理服务包括以下组件：  
**barbican-api 服务**  
&nbsp;&nbsp;&nbsp;&nbsp;提供配置和管理Barbican机密的OpenStack原生 RESTful API。 

**barbican-worker 服务**  
&nbsp;&nbsp;&nbsp;&nbsp;提供与‘barbican-api’交互的OpenStack RPC接口，并从barbican消息队列里读取信息。支持barbican订单的执行。  

**barbican-keystone-listener 服务**  
&nbsp;&nbsp;&nbsp;&nbsp;监听来自Keystone的通知服务消息。用于当项目删除时管理在barbican数据库的Keystone项目的表示。