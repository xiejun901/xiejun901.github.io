title: 环境安装
date: 2017-11-23 00:23:09
tags: [install, environment, tensorflow, nvidia, cuda]
categories: other
---

# 记录一些软件的安装过程，帮助后续环境的重新安装之类的


## Nvidia 驱动以及cuda，cudnn的安装

看网上说安装驱动的时候可能会出现循环重启，但是看一个人提供了不会出现这种问题的安装方式，就直接使用了，因此并没有感受到循环重启的问题

```shell
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda
# 环境变量可以写入配置文件中
export PATH=/usr/local/cuda-8.0/bin:PATH
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:LD_LIBRARY_PATH
``` 

这儿有一点需要的是由于由于可能有多个版本，可以通过命令查看版本后安装指定的版本，多个版本安装了之后可以使用软链接进行管理

```shell
apt-cache madison <<package name>>
```
还得再安装cudnn才能使用tensorflow的，可以按照如下的方式安装

```
CUDNN_TAR_FILE="cudnn-8.0-linux-x64-v6.0.tgz"
wget http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/${CUDNN_TAR_FILE}
tar -xzvf ${CUDNN_TAR_FILE}
sudo cp -P cuda/include/cudnn.h /usr/local/cuda-8.0/include
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda-8.0/lib64/
sudo chmod a+r /usr/local/cuda-8.0/lib64/libcudnn*
```

在找cudnn安装的时候发现一个脚本，集成了以上两步

```shell
#!/bin/bash

# install CUDA Toolkit v8.0
# instructions from https://developer.nvidia.com/cuda-downloads (linux -> x86_64 -> Ubuntu -> 16.04 -> deb (network))
CUDA_REPO_PKG="cuda-repo-ubuntu1604_8.0.61-1_amd64.deb"
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG}
sudo dpkg -i ${CUDA_REPO_PKG}
sudo apt-get update
sudo apt-get -y install cuda

# install cuDNN v6.0
CUDNN_TAR_FILE="cudnn-8.0-linux-x64-v6.0.tgz"
wget http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/${CUDNN_TAR_FILE}
tar -xzvf ${CUDNN_TAR_FILE}
sudo cp -P cuda/include/cudnn.h /usr/local/cuda-8.0/include
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda-8.0/lib64/
sudo chmod a+r /usr/local/cuda-8.0/lib64/libcudnn*

# set environment variables
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
## anaconda 的安装

官网速度太慢，可以去清华的开源软件镜像站下载，很快。

[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)

## tensorflow 安装

安装好nvidia驱动之后直接安装官网的方式安装使用pip安装就行

我用的anaconda安装

tfBinaryURL 可以到这儿查看 [URL of the TensorFlow Python package](https://www.tensorflow.org/install/install_linux#the_url_of_the_tensorflow_python_package)

```shell
pip install --ignore-installed --upgrade tfBinaryURL
```

## hexo 的安装

进入官网，查看安装方法

可能需要先安装npm, node.js, node.js 可以使用淘宝的源，会快一些  [node.js淘宝](https://npm.taobao.org/mirrors/node/)

```shell
# 安装node, 版本可以自己选择
wget https://npm.taobao.org/mirrors/node/v9.2.0/node-v9.2.0-linux-x64.tar.gz
tar -xvf node-v9.2.0-linux-x64.tar.gz
sudo mv node-v9.2.0-linux-x64 /opt/
sudo ln -s /opt/node-v9.2.0-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /opt/node-v9.2.0-linux-x64/bin/npm /usr/local/bin/npm
# 如上安装啦npm之后这一步可以不用啦安装 npm 了
sudo apt-get install npm
sudo npm install hexo-cli -g
# 进入到博客的目录
cd PATH
npm install
hexo server
```

然后就启动了，可以使用了。

## hexo mathjax 安装和使用

hexo 使用mathjax需要先更换markdown渲染插件，然后再在设置中设置使用mathjax

```shell
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-pandoc --save
```
然后修改配置文件的一些转意设置
node_modules\kramed\lib\rules\inline.js 这个文件修改如下内容

```
//  escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
  escape: /^\\([`*\[\]()#$+\-.!_>])/,

//  em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
  em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

打开配置开关

进入主题的目录找到   themes/next/_config.yml 进行修改

```
mathjax:
  enable: true
  per_page: true
```

如果per_page设置成了true, 那么需要在页面开头写下如下语句才能在本页启动mathjax渲染

```
---
title: index.html
date: 2016-12-28 21:01:30
tags:
mathjax: true
--
```


