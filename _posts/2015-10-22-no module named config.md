---
layout: post
title: no module named config
tags: python  pip
categories: 工作记录 
---

当把代码上传到gerrit后，会触发jenkins做四项工作，


##`1`.

git fetch 新的commit并进行merge，

##`2`.

装上requirements.txt和test-requirements.txt里列出的第三方库的包，并对代码进行pep8规范检查

##`3`.

python27检查本模块测试用例

##`4`.

tempest devstack测试，在某个环境中部署完整的相关服务，并且根据不同的提交运行不同的tempest用例。感觉是写了一些涉及各个模块交互的用例。


上传了umbxxxa的代码后，在第二步时候时报了个错误

~~~bash
from oslo.config import config
No module named config
~~~

这是由于oslo的版本造成的问题。

oslo是早期openstack的与配置相关的功能的代码（该功能原先放在common文件夹中）独立出来做的一个包。

该包也在进化中，可能最新的版本中已经没有config这个module了。

在第二步pip install -r装requirements.txt中的包时，该文件中写的是oslo.config>=1.2.0

如果在实际安装时，pip源中的该第三方库的版本过高，那么可能就没有config这个module了，因此对其限制一下最高版本范围：

写成oslo.config>=1.2.0,<2.0.0

再执行提交和检测，过了~

P.S.
在社区发布的openstack/nova的各个稳定版本中，在requirement.txt中的python第三方库的版本范围都是做了限制的，会有版本上下限，应该就是为了防止pip install直接安装时，pip源中该第三方库的版本号过高的问题。如果遇到类似问题的话，可以去查看相应的稳定版本中的第三方库的版本号范围。


####`再补几个pip命令`


安装时指定版本：

~~~bash
pip install "oslo.config>=1.2.0,<2.0.0"
~~~

从依赖列表安装：

~~~bash
pip install -r requirement.txt
~~~

指定安装源：

~~~bash
pip install xxx -i 源的地址（比如豆瓣源http://pypi.douban.com/simple）
~~~

指定安装路径：

~~~bash
pip install xxx -t 目标dir
~~~

当前安装包列表：

~~~bash
pip list （加--outdated可以看当前版本和最新版本号）
~~~

以requirements格式输出安装信息：

~~~bash
pip freeze，输出结果可以用于requirements文件中
~~~

已经安装包的初步信息：

~~~bash
pip -V XXX
~~~

包详细信息，显示安装位置和版本号：

~~~bash
pip show XXX
~~~

从源码安装包：下载好源码包，通常是压缩文件，用

~~~bash
 tar -xzvf 源码包位置 
来解压，然后进入文件夹，用
python setup.py install进行安装
~~~

在源里查找包：

~~~bash
pip search XXX
~~~

卸载包：
~~~bash
pip uninstall XXX
~~~




