# 在DevStack上运行Barbican  
Barbican目前可以通过devstack中的插件界面获得。  
我们提供了两种将正在运行的Barbican部署到DevStack环境中的方法。简单模式通过vagrant自动创建一个拥有所有必要依赖的VN来运行DevStack。如果你是个新手的话，这种方法很适合你。  
如果你是DevStack的老玩家，那你可以按照手工安装步骤在你已有的DevStack环境中安装Barbican。  

# 简单模式  
为了简化在DevStack上运行Barbican的设置过程，Vagrantfile会自动在DevStack上创建一个运行Barbican的VM。  

1. 获取Barbican源码  
```bash
        git clone https://github.com/openstack/barbican.git
```  

2. 将 **barbican-vagrant**文件夹从Barbican目录中移出，移动到当前目录用以查找vagrant文件。  
```bash  
        cp -r barbican/devstack/barbican-vagrant <directory>
```  

3. 进入 **barbican-vagrant**目录  
```bash
        cd barbican-vagrant
```  

4. 创建新的VM  
```bash
        vagrant up
```  

5. 成功创建并配置好VM后，ssh登录  
```bash  
        vagrant ssh
```  

6. 登录VM后，切换目录到 **devstack**下  
```bash
        cd /opt/stack/devstack
```  

7. 启动DevStack  
```bash  
        ./stack.sh
```  

# 手工配置  
以下步骤假设你在一个干净的Ubuntu14.04虚拟机（本地或者云上）中运行。如果你在本地运行，切记要放行以下端口：  

1. Barbican - 9311  
2. Keystone API - 5000  
3. Keystone Admin API - 5000  

## 安装  
1. 确保你登录的用户是拥有 **sudo**权限的非root用户  

2. 安装git  
```bash  
    sudo apt-get install git
```  

3. 克隆DevStack  
```bash
    git clone https://github.com/openstack-dev/devstack.git
```  

4. 添加Barbican插件到 **local.conf**文件中，并确保包含最低服务器要求。你可以通过将名称附加到git URL末尾来拉去指定分支。如果你将末尾空间留空如下，将拉去origin/master分支。  
```bash
    enable_plugin barbican https://opendev.org/openstack/barbican
    enable_service rabbit mysql key
```  
如果你是第一次使用，并且没有 **local.conf**文件，你可以从<u>Barbican github</u>获取示例，复制并存放到devstack/目录下即可。  

5. 启动DevStack  
```bash  
    cd devstack/
    ./stack.sh
```
