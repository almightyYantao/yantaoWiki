---
title: 安装&破解Openvpn Access Server
tags: [ '2.9.0','openvpn','openvpn access server','破解' ]
categories: ['OpenVPN']
date: 2023-11-19
---

## 在线安装（需要翻墙）

```shell
yum -y install https://as-repository.openvpn.net/as-repo-centos7.rpm
yum -y install openvpn-as
```

## 管理员登录

首先需要修改管理员密码

```shell
passwd openvpn
```

输入之后可以登录管理员页面了。

如果密码一直错误的话，说明修改失败了，可以查看下原来的密码是什么

```shell
cat /usr/local/openvpn_as/init.log
```

登录后点击User Management–>User Permissions 添加用户

## 破解用户数

- 主要操作的文件是一个名叫 `pyovpn-2.0-pyx.x.egg` 的文件，以我了解的情况来看，从 `2.5.0` 到 `2.9.x` 文件名一直都是这个，只是不同版本里面的内容不一样.
- 这个文件有点类似 `Java` 当中的 `jar` 库文件，也是一个 `zip` 压缩文件，里面包含了一些 `Python` 的字节码文件.
- 破解的原理大概是在 `Python` 中采用类似 `Java` 动态代理的技术，将原本读取用户属性的调用返回值拦截，修改用户限制数量再返回.

### 方法

`2.9.0` 以下版本破解的目标文件是 `/pyovpn/lic/uprop.pyo`, `2.9.0` 及以上是 `/pyovpn/lic/uprop.pyc`; 按照网上流行的破解方法，把这个文件解压出来并改名为 `uprop2.pyo` 或 `uprop2.pyc`, 然后新建一个 `uprop.py` 文件，内容如下:

`**2.9.0**`**以下版本内容:**

```shell
import uprop2
old_figure = None

def new_figure(self, licdict):
    ret = old_figure(self, licdict)
    ret['concurrent_connections'] = 1024
    return ret


for x in dir(uprop2):
    if x[:2] == '__':
        continue
    if x == 'UsageProperties':
        exec('old_figure = uprop2.UsageProperties.figure')
        exec('uprop2.UsageProperties.figure = new_figure')
    exec('%s = uprop2.%s' % (x, x))
```

`**2.9.0**`**及以上版本内容:**

```shell
from pyovpn.lic import uprop2
old_figure = None

def new_figure(self, licdict):
    ret = old_figure(self, licdict)
    ret['concurrent_connections'] = 1024
    return ret


for x in dir(uprop2):
    if x[:2] == '__':
        continue
    if x == 'UsageProperties':
        exec('old_figure = uprop2.UsageProperties.figure')
        exec('uprop2.UsageProperties.figure = new_figure')
    exec('%s = uprop2.%s' % (x, x))
```

再将上面的 `uprop.py` 编译为库文件 `uprop.pyo` 或 `uprop.pyc`:

```shell
# <2.9.0
python2 -O -m compileall uprop.py
# >=2.9.0
python3 -O -m compileall uprop.py && mv __pycache__/uprop.cpython-37.opt-1.pyc uprop.pyc
```

> 注意 `uprop.cpython-37.opt-1.pyc` 文件名会随着 `python` 版本变化而变化.

现在我们得到了一个改文件名的文件 `uprop2.pyo` 或 `uprop2.pyc`, 和一个编译出来的 `uprop.pyo` 或 `uprop.pyc`; 把这两个文件压缩到 `pyovpn-2.0-pyx.x.egg` 的 `/pyovpn/lic/` 目录下，然后去服务器替换目标文件，重启服务就 OK 了.

## 所有的操作命令

```shell
# 把操作的文件复制出来到另外的一个文件夹下面操作
cd /usr/local/openvpn_as/lib/python & mkdir pojie & cp python/pyovpn-2.0-py3.6.egg ./pojie & cd pojie/
# 改一个文件名，防止后面打包的时候把源文件覆盖了
mv pyovpn-2.0-py3.6.egg pyovpn-2.0-py3.6.egg.bak & unzip pyovpn-2.0-py3.6.egg.bak
# 进入到需要修改的目录
cd pyovpn/lic
# 复制源文件改名
mv uprop.pyc uprop2.pyc
# 新增一个py文件，然后把文件内容复制进去，并保存
vi uprop.py
# 退到 pojie 文件夹，进行打包
zip -r pyovpn-2.0-py3.6.egg EGG-INFO/ common/ pyovpn/
# 复制文件到运行目录，覆盖
cp pyovpn-2.0-py3.6.egg /usr/local/openvpn_as/lib/python
# 重启服务
systemctl restart openvpnas
```
