# 项目结构  

1. **Barbican/**（Barbican Python源文件目录）  
a. **api/**（REST API相关源文件）  
&nbsp;&nbsp;&nbsp;&nbsp;A. **controllers/**（基于Pecan的控制器，处理基于REST的请求）  
&nbsp;&nbsp;&nbsp;&nbsp;B. **middleware/**(处理REST请求的中间件业务逻辑)  
b. **cmd/**（Barbican管理员命令源文件）  
c. **common/**（公用模块）  
d. **locale/**（翻译模板）  
e. **model/**（基于SQLAlchemy的模型类）  
f. **plugin/**（有关逻辑、接口和上层管理的插件）
&nbsp;&nbsp;&nbsp;&nbsp;A. **resources.py**（支持与插件交互）  
&nbsp;&nbsp;&nbsp;&nbsp;B. **crypto/**（HSM逻辑和插件）  
&nbsp;&nbsp;&nbsp;&nbsp;C. **interface/**（证书管理和机密存储接口）  
g. **queue/**（客户端和服务器接口队列）  
&nbsp;&nbsp;&nbsp;&nbsp;A. **client.py**（允许客户端将任务发布到队列）  
&nbsp;&nbsp;&nbsp;&nbsp;B. **server.py**（运行工作服务，响应队列任务）  
h. **tasks/**（与worker相关的控制和实现）  
i. **tests/**（单元测试）  

2. **bin/**（barbican节点启动脚本）  

3. **devstack/**（Barbican DevStack插件，DevStack配置和用于安装DevStack VM的Vagrantfile）  

4. **etc/barbican/**（配置文件）  

5. **functionaltests**（barbican功能测试）  

6. **doc/source**（Sphinx文档）  

7. **releasenotes**（Barbican发行说明）