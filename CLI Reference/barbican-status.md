# Barbican状态  

# 概要  
```
    barbican-status <category> <command> [<args>]
```  

# 说明  
**barbican-status**是用来检查barbican运行状态的工具。  

# 操作  
执行barbican-manage命令的标准格式如下： 
```
    barbican-status <category> <command> [<args>]
```  

不带参数执行此命令将列出可用命令类型：  
```
    barbican-status
```  

类别有：
* &nbsp;upgrade

详细说明如下。  
你可以带类别参数，例如**upgrade**，执行此命令来查看此类别下的所有命令：  
```
    barbican-status upgrade
```  

以下部分说明了**barbican-status**的可用类别和参数。  

# 升级  
**barbican-status upgrade check**  
&nbsp;&nbsp;&nbsp;&nbsp;使用新代码重启服务之前检查特定发行版的准备情况。此命令需要对数据库和服务拥有完整的访问和配置权限。  
&nbsp;&nbsp;&nbsp;&nbsp;**返回值**  
&nbsp;&nbsp;&nbsp;&nbsp;|返回值|说明|
&nbsp;&nbsp;&nbsp;&nbsp;|--|--|
&nbsp;&nbsp;&nbsp;&nbsp;|0|所有准备事项成功通过检查。不需要做其它事情|
&nbsp;&nbsp;&nbsp;&nbsp;|1|至少有一项提交检查失败，需要进一步调查。可视作警告，但是也可能升级成功。|
&nbsp;&nbsp;&nbsp;&nbsp;|2|升级状态检查失败，需要进一步调查。此时应该停止升级。|
&nbsp;&nbsp;&nbsp;&nbsp;|255|出现了一个意料之外的错误。|

&nbsp;&nbsp;&nbsp;&nbsp;**检查历史**  
&nbsp;&nbsp;&nbsp;&nbsp;**8.0.0（Stein）**
*&nbsp;&nbsp;&nbsp;&nbsp;此功能将在Stein版本中添加。