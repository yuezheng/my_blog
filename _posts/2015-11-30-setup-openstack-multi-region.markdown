---
layout: post
title:  "搭建OpenStack多Region环境"
date:   2015-11-30 21:28:17 +0800
categories: OpenStack
---

#搭建OpenStack多Region环境

###Region
Region 是OpenStack中的一个概念，类似的还有Available Zone, Aggregate 等等，与其他概念相比Region
更偏向于地理区域的划分，各个Region都有独立的全套服务，而多个Region之间共享一个Keystone（身份认证）
服务，通过在Keystone中创建不同的Endpoints来导向指定Region的特定服务，这使得分布在地理上不同位置的Region
可以通过统一的入口来获取服务，极大的方便了管理。

###配置方法
下面具体介绍一下多Region环境的相关配置方法。

前面提到了，每个Region本身是一套独立的OpenStack系统，在这里假设已经有了两个部署好的OpenStack
环境，我们需要做的就是将这两个独立的环境通过Keystone联系起来。

#####两套环境： RegionOne, RegionTwo 使用RegionOne中的Keystone作为公共认证服务

1. 修改Nova/Glance/Cinder/Ceilometer配置，将其认证服务地址指向RegionOne中的Keystone;
<pre> 以/etc/nova/nova.conf为例，各个服务关于Keystone认证的配置都在keystone_authtoken区块中：
<code>
    [keystone_authtoken]
    signing_dir = /var/cache/nova
    cafile = /opt/stack/data/ca-bundle.pem
    auth_uri = http://192.168.1.109:5000
    project_domain_id = default
    project_name = service
    user_domain_id = default
    password = ****
    username = nova
    auth_url = http://192.168.1.109:35357
    auth_plugin = password
</code></pre>
2. 重启各服务，可以将RegionTwo中的keystone关闭;
3. 在RegionOne的Keystone中创建RegionTwo的Endpoints：
    
    (1).获取service列表：
    <pre>keystone service-list</pre>
    (2).获取当前endpoints列表：
    <pre>keystone endpoint-list</pre>
    (3).创建新的endpoints:
    <pre>keystone endpoint-create --region RegionTwo --service-id [获取到的service的id] --publicurl [public-url] --adminurl [admin-url] --internalurl [internal-url]</pre>

关于endpoint中的三种url，其实很好理解，就是为了将不同的服务请求负载到不同的服务地址，一方面提高安全性，一方面方便管理。

当多Region环境配置完成后，你可能会发现一些命令行工具没办法使用了，这时需要导入一个额外的环境变量： OS_REGION_NAME，值的话就是你所希望使用的Region名称(同时也是ID)。方便起见直接写到localrc文件中： export OS_REGION_NAME=RegionOne

上面过程中创建endpoint是个力气活，全靠手动创建的话还容易出错，我特定写了个小脚本来处理：
[yuezheng/RegionBuilder](https://github.com/yuezheng/RegionBuilder)

需要调整的是config.json文件：

1. 其中的auth部分是RegionOne的认证数据，一定得是admin用户;
2. regions中即RegionTwo的相关信息。
