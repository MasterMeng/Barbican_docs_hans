# 验证操作  
验证密钥管理服务（barbican）的操作。  
## 注意
在控制接节点执行以下操作。  
1. 安装python-barbicanclient包：  
* openSUSE和SUSE Linux Enterprise:  
```bash
    $ zypper install python-barbicanclient
```  
* Red Hat Enterprise Linux 和 CentOS：  
```bash
    $ yum install python-barbicanclient
```
* Ubuntu：  
```bash
    $ apt-get install python-barbicanclient
```  

2. 获取可执行barbican API调用的 **admin**凭证：  
```bash
    $ . admin-openrc
```  

3. 使用OpenStack CLI存储密码：  
```bash
    $ openstack secret store --name mysecret --payload j4=]d21
+---------------+-----------------------------------------------------------------------+
| Field         | Value                                                                 |
+---------------+-----------------------------------------------------------------------+
| Secret href   | http://10.0.2.15:9311/v1/secrets/655d7d30-c11a-49d9-a0f1-34cdf53a36fa |
| Name          | mysecret                                                              |
| Created       | None                                                                  |
| Status        | None                                                                  |
| Content types | None                                                                  |
| Algorithm     | aes                                                                   |
| Bit length    | 256                                                                   |
| Secret type   | opaque                                                                |
| Mode          | cbc                                                                   |
| Expiration    | None                                                                  |
+---------------+-----------------------------------------------------------------------+  
``` 

4. 通过检索确认机密已存储：  
```bash
$ openstack secret get http://10.0.2.15:9311/v1/secrets/655d7d30-c11a-49d9-a0f1-34cdf53a36fa
+---------------+-----------------------------------------------------------------------+
| Field         | Value                                                                 |
+---------------+-----------------------------------------------------------------------+
| Secret href   | http://10.0.2.15:9311/v1/secrets/655d7d30-c11a-49d9-a0f1-34cdf53a36fa |
| Name          | mysecret                                                              |
| Created       | 2016-08-16 16:04:10+00:00                                             |
| Status        | ACTIVE                                                                |
| Content types | {u'default': u'application/octet-stream'}                             |
| Algorithm     | aes                                                                   |
| Bit length    | 256                                                                   |
| Secret type   | opaque                                                                |
| Mode          | cbc                                                                   |
| Expiration    | None                                                                  |
+---------------+-----------------------------------------------------------------------+
```  

5. 通过减速payload确认已存储：  
```bash
$ openstack secret get http://10.0.2.15:9311/v1/secrets/655d7d30-c11a-49d9-a0f1-34cdf53a36fa --payload
+---------+---------+
| Field   | Value   |
+---------+---------+
| Payload | j4=]d21 |
+---------+---------+
```