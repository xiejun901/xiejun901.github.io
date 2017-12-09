title: 安装jupyter
date: 2017-06-12 22:56:06
categories: jupyter
tags: [python, jupyter]
---

参考官网安装指南

从官网下载安装脚本 Anaconda2-4.3.1-Linux-x86_64.sh， 执行以下命令

```shell
bash Anaconda2-4.3.1-Linux-x86_64.sh
```

### 配置 jupyter notebook

- 使用 ipython 生成秘钥

```python
from notebook.auth import passwd
passwd()
```



- 生成配置文件

```shell
jupyter notebook --generate-config
vim ~/.jupyter/jupyter_notebook_config.py
```

- 修改配置文件, 修改以下几项

```
c.NotebookApp.ip='*' # 就是设置所有ip皆可访问
c.NotebookApp.password = u'sha:ce...刚才复制的那个密文'
c.NotebookApp.open_browser = False # 禁止自动打开浏览器
c.NotebookApp.port =8888 #随便指定一个端口
```

- 启动 jupyter notebook 测试

```shell
jupyter notebook
```

然后在浏览器中输入ip和端口，然后输入密码即可进入，此时的 kernel 只有 python 的，可以输入如下指令查看 安装的 kernel

```shell
jupyter kernelspec list

Available kernels:
  python2    /home/dm/anaconda2/lib/python2.7/site-packages/ipykernel/resources
```

### 安装 toree 连接spark

[incubator-toree](https://github.com/apache/incubator-toree) 可以参考这儿安装
更多的 kernel 可以去jupyter官网查找 [Jupyter-kernels](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels)

确认下 pip 是不是 Anaconda 中的pip， 然后直接使用 pip 进行安装

```shell
pip install toree
jupyter toree install --interpreters=Scala --spark_home=$SPARK_HOME --user --kernel_name=apache_toree --interpreters=PySpark,SparkR,Scala,SQL
```

以上中需确定$SPARK_HOME 的正确， --user 表示的是安装在用户目录，这样就不需要管理员权限了，安装完成后可以查看下当前已经安装的kernel

```shell
jupyter kernelspec list
Available kernels:
  python2                 /home/dm/anaconda2/lib/python2.7/site-packages/ipykernel/resources
  apache_toree_pyspark    /data/home/dm/.local/share/jupyter/kernels/apache_toree_pyspark
  apache_toree_scala      /data/home/dm/.local/share/jupyter/kernels/apache_toree_scala
  apache_toree_sparkr     /data/home/dm/.local/share/jupyter/kernels/apache_toree_sparkr
  apache_toree_sql        /data/home/dm/.local/share/jupyter/kernels/apache_toree_sql
```

启动 jupyter notebook 测试一下

```shell
jupyter notebook
```

spark 的配置文件在这个路径下  /data/home/dm/.local/share/jupyter/kernels/apache_toree_scala
