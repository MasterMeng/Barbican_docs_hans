# 安装Barbican开发环境  
本文档只在帮助你安装使用SQLite作为数据库后端的Barbican独立版本。因为缺乏认证以及诸如HSM之类的安全加密系统接口，所以不适合用于生产环境。此外，SQLite后端已知存在线程安全问题。此设置纯粹是为了便于开发。  

# 安装系统依赖  

## Ubuntu：  
```bash
    # Install development tools
    sudo apt-get install git python-tox

    # Install dependency build requirements
    sudo apt-get install libffi-dev libssl-dev python-dev gcc
```  

# 设置虚拟开发环境  
我们强烈建议您使用Python虚拟开发环境进行开发。你可以在Python教程中了解有关<u>虚拟开发环境</u>的更多信息。  

如果你在上一步中安装了tox，那么你应该已经安装了virtualenv。  
```bash
    # Clone barbican source
    git clone https://opendev.org/openstack/barbican
    cd barbican

    # Create and activate a virtual environment
    virtualenv .barbicanenv
    . .barbicanenv/bin/activate

    # Install barbican in development mode
    ./bin/barbican.sh install
```

# 配置Barbican
Barbican使用oslo.config进行配置。默认情况下，Barbican进程会在 **$HOME**或者 **/etc/barbican/**目录下查找配置文件 **barbican.conf**。源码中包含的配置文件默认使用 **/etc/barbican**目录存放配置文件。  
```bash
    # Create the directories and copy the config files
    sudo mkdir /etc/barbican
    sudo chown $(whoami) /etc/barbican
```  

# 启动Barbican  
如果你想启动Barbican，你可以在Barbican开发服务器上执行以下命令：  
```bash
    ./bin/barbican.sh start
```  

上述命令将会启动一个Barbican实例来监听 **http://localhost:9311**。  
# 构建文档  
你可以使用tox构建HTML文档：  
```bash
    tox -e docs
```

# 运行测试单元  
你可以使用tox运行单元测试：  
```bash  
    tox -e py36
```