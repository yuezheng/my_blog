---
layout: post
title:  "Ansible配置"
description: "记录在Ansible中如何使用配置"
tags: [Ansible]
categories: 配置管理,运维开发,工具
---

# Ansible中支持的配置方式
Ansible提供了三种方式对执行环境进行配置：
* 配置文件
* 环境变量
* 命令行选项

## 配置文件
Ansible的配置文件默认名称叫做：ansible.cfg，放在/etc/ansible目录下，如果你是从源码或者使用pip安装的Ansible,
那么就需要下载一份放在你的环境里，下载地址：[Example Config File](https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg)

在Ansible 2.4以后的版本还可以通过ansible-config命令行工具来查看你当前的配置情况。

## 环境变量
Ansible同样可以通过环境变量的方式来修改配置，具体支持的环境变量列表可以在[这里](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)查询。

## 命令行选项
在命令行中同样可以调整Ansible配置，不过不是所有的配置项都可以在命令行中使用，仅支持最常用的配置项。

# 优先级
配置生效的优先级： 命令行选项 > 环境变量 > 配置文件