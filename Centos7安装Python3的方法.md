# Centos7安装Python3的方法 #

由于centos7原本就安装了Python2，而且这个Python2不能被删除，因为有很多系统命令，比如yum都要用到。

```
[root@VM_105_217_centos Python-3.6.2]# python
Python 2.7.5 (default, Aug  4 2017, 00:39:18)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-16)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
```

输入Python命令，查看可以得知是Python2.7.5版本

输入

	which python

可以查看位置，一般是位于/usr/bin/python目录下。

下面介绍安装Python3的方法

首先安装依赖包

    yum -y groupinstall "Development tools"
    yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel

然后根据自己需求下载不同版本的Python3，我下载的是Python3.6.2

    wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tar.xz


如果速度不够快，可以直接去官网下载，利用WinSCP等软件传到服务器上指定位置，我的存放目录是/usr/local/python3，使用命令：

    mkdir /usr/local/python3 

建立一个空文件夹

然后解压压缩包，进入该目录，安装Python3

    tar -xvJf  Python-3.6.2.tar.xz
    cd Python-3.6.2
    ./configure --prefix=/usr/local/python3
    make && make install

最后创建软链接

    ln -s /usr/local/python3/bin/python3 /usr/bin/python3
    ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

在命令行中输入python3测试

![](./images/1043829-20170924220710103-1511072281.png)