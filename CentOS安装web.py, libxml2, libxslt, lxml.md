# 安装web.py, libxml2, libxslt, lxml #

1、安装web.py

    pip3 install web.py

 

2、安装lxml

    pip3 install lxml

 

3、安装libxml2 （参考 http://c.biancheng.net/view/1116.html）

第一步、安装python-devel

    yum -y install python-devel

第二步、下载文件

    wget ftp://xmlsoft.org/libxml2/libxml2-2.9.10.tar.gz

第三步、配置

    ./configure --prefix=/usr/local/libxml2/

第四步、`make`

第五步、`make install`

 

4、安装libxslt

    yum -y install libxslt

————————————————

版权声明：本文为CSDN博主「Cheinyx」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/Cheinyx/article/details/107444600