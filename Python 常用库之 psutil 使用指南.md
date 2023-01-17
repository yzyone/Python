# Python 常用库之 psutil 使用指南

剧照：约定的梦幻岛（第二季）

作者：古明地盆

来源：www.cnblogs.com/traditional/p/12580638.html

# 楔子

**Python 有一个第三方模块：psutil，专门用来获取操作系统以及硬件相关的信息，比如：CPU、磁盘、网络、内存等等。**

**首先我们要安装，直接 pip install psutil 即可，安装之后来看看它的用法。**

# CPU 相关

**获取 CPU 的逻辑数量**

```
import psutil

print(psutil.cpu_count())  # 12
```

**获取 CPU 的物理核心数量**

```
import psutil

print(psutil.cpu_count(logical=False))  # 6
```

> **结果为 6，说明是 6 核超线程；如果 CPU 的物理核心数 和 逻辑数相等，也为 12，则说明是 12 核非超线程。**

**统计 CPU 的用户／系统／空闲时间**

```
import psutil

print(psutil.cpu_times()) 
# scputimes(user=65531.796875, system=42440.76562500023, idle=1783904.3593749998, interrupt=5676.375, dpc=1846.609375)

# psutil.cpu_times_percent() 功能与之类似, 只不过返回的比例
```

> **返回的是一个 namedtuple，后面凡是结构长的和这里类似的，都是 namedtuple。**

**查看 CPU 的使用率**

```
import psutil

for x in range(3):
    # interval：表示每隔0.5s刷新一次
    # percpu：表示查看所有的cpu使用率
    print(psutil.cpu_percent(interval=0.5, percpu=True))
"""
[6.1, 6.2, 9.4, 3.1, 0.0, 0.0, 0.0, 6.2, 3.1, 3.1, 3.1, 0.0]
[0.0, 0.0, 6.1, 0.0, 6.1, 3.0, 0.0, 3.0, 3.0, 3.0, 0.0, 9.1]
[0.0, 0.0, 6.2, 3.1, 3.1, 0.0, 3.1, 3.1, 3.1, 3.1, 0.0, 0.0]
"""
# 我这里cpu的逻辑数量是12, 所以每个列表里面有12个元素
```

**查看 CPU 的统计信息，包括上下文切换、中断、软中断，以及系统调用次数等等**

```
import psutil

print(psutil.cpu_stats())
# scpustats(ctx_switches=2912990332, interrupts=4290503758, soft_interrupts=0, syscalls=2698751096)
```

**查看 CPU 的频率**

```
import psutil

print(psutil.cpu_freq())  # scpufreq(current=2208.0, min=0.0, max=2208.0)
```

# 内存相关

**查看内存使用情况**

```
import psutil

print(psutil.virtual_memory())  
# svmem(total=17029259264, available=8437215232, percent=50.5, used=8592044032, free=8437215232)
```

total: 总内存

available: 可用内存

percent: 内存使用率

used: 已使用的内存

**查看交换内存信息**

```
import psutil

print(psutil.swap_memory())
# sswap(total=19579396096, used=15708250112, free=3871145984, percent=80.2, sin=0, sout=0)
```

> **关于物理内存和交换内存之间的关系。**
>
> **物理内存：就是实际的内存条提供的临时数据存储空间，在 Windows 上右键点击计算机，****再****点击属性时，上面显示的安装内存（RAM）就是电脑的物理内存。这些内存是实际存在的，在你不给机器增加内存条的时候是不会改变的。**
>
> **交换内存：交换内存是专门用来临时存储数据的，通常在页面调度和交换进程数据时使用。相当于在进行内存整理的时候，会先把部分数据放在硬盘的某个地方，类似我们整理衣柜，衣服一多直接整理会很麻烦，因此会先把部分衣服放在其他地方，等衣柜里的衣服整理完了再把放在其它地方的衣服拿回来。这个其他地方放在计算机中则代表硬盘的某个地方，也就是我们所说的交换区。通常当使用交换内存时，是因为物理内存不足，正所谓衣柜，如果足够大的话就没必要拿出部分衣服放在其它地方， 直接在衣柜里就能解决了。**
>
> **虚拟内存：首先，如果想要操作文件，可执行程序等等，那么首先要把它们从磁盘上读取到内存中，因此 CPU 除了自己的那一部分小小的空间外，要想操作数据，只能操作内存里的数据。但是当内存不够了，那么便会在硬盘上开辟一份虚拟内存，将物理内存里的部分数据放在虚拟内存当中。硬盘的空间很大，即使普通电脑安装的固态硬盘也有一百个 G，因此可以拿出一部分充当虚拟内存。不过虚拟内存虽说是内存，但毕竟在硬盘上，速度绝对和 CPU 直接从物理内存里读取数据的速度相差甚远。这也是为什么那些大型网站将经常被访问的一些数据放在 Redis 缓存里，而不是放在硬盘或者数据库上。**

# 磁盘相关

**查看磁盘分区、磁盘使用率和磁盘 IO 信息**

```
from pprint import pprint
import psutil

pprint(psutil.disk_partitions())
"""
[sdiskpart(device='C:\\', mountpoint='C:\\', fstype='NTFS', opts='rw,fixed'),
 sdiskpart(device='D:\\', mountpoint='D:\\', fstype='NTFS', opts='rw,fixed'),
 sdiskpart(device='E:\\', mountpoint='E:\\', fstype='NTFS', opts='rw,fixed')]
"""
# 可以看到一共有三个盘符，fstype表示文件系统格式是NTFS，opts中的rw表示可读写
# 里面有一个参数 all, 默认为 False, 如果指定为 True, 那么返回的内容还会包含 /proc 等特殊文件系统的挂载信息
# 由于我这里是 Windows, 所以两者没区别
```

**查看某个磁盘使用情况**

```
import psutil

print(psutil.disk_usage("C:\\"))
# sdiskusage(total=267117391872, used=88213196800, free=178904195072, percent=33.0)
```

**查看磁盘 IO 统计信息**

```
from pprint import pprint
import psutil

pprint(psutil.disk_io_counters())
# sdiskio(read_count=1270037, write_count=2146886, read_bytes=34637616128, write_bytes=53505994240, read_time=551, write_time=1258)
```

read_count: 读次数

write_count: 写次数

read_bytes: 读的字节数

write_bytes: 写的字节数

read_time: 读时间

write_time: 写时间

**默认返回的是所有磁盘加起来的统计信息，我们可以指定 perdisk=True，则分别列出每一个磁盘的统计信息。**

```
from pprint import pprint
import psutil

pprint(psutil.disk_io_counters(perdisk=True))
"""
{'PhysicalDrive0': sdiskio(read_count=1262459, write_count=2149207, read_bytes=34598280704, write_bytes=53708976128, read_time=532, write_time=1261),
 'PhysicalDrive1': sdiskio(read_count=7702, write_count=98, read_bytes=41695232, write_bytes=4730880, read_time=19, write_time=0)}
"""
```

# 网络相关

**查看网卡的网络 IO 统计信息**

```
from pprint import pprint
import psutil

pprint(psutil.net_io_counters())
"""
snetio(bytes_sent=536008958, bytes_recv=8676204996, packets_sent=2725499, packets_recv=7225179, errin=0, errout=9, dropin=0, dropout=0)
"""
# bytes_sent: 发送的字节数
# bytes_recv: 接收的字节数
# packets_sent: 发送的包数据量
# packets_recv: 接收的包数据量
# errin: 接收包时, 出错的次数
# errout: 发送包时, 出错的次数
# dropin: 接收包时, 丢弃的次数
# dropout: 发送包时, 丢弃的次数


# 里面还有一个 pernic 参数, 如果为 True, 则列出所有网卡的信息
pprint(psutil.net_io_counters(pernic=True))
"""
{'Loopback Pseudo-Interface 1': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0),
 'WLAN': snetio(bytes_sent=534497477, bytes_recv=8678905297, packets_sent=2706204, packets_recv=7244187, errin=0, errout=0, dropin=0, dropout=0),
 '以太网': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0),
 '以太网 2': snetio(bytes_sent=3612804, bytes_recv=7955853, packets_sent=32818, packets_recv=26442, errin=0, errout=9, dropin=0, dropout=0),
 '本地连接* 2': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0),
 '本地连接* 3': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0),
 '蓝牙网络连接': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0)}
"""
```

**查看网络接口信息**

```
from pprint import pprint
import psutil

# 以字典的形式返回网卡的配置信息, 包括 IP 地址、Mac地址、子网掩码、广播地址等等
pprint(psutil.net_if_addrs())
"""
{'Loopback Pseudo-Interface 1': [snicaddr(family=<AddressFamily.AF_INET: 2>, address='127.0.0.1', netmask='255.0.0.0', broadcast=None, ptp=None),
                                 snicaddr(family=<AddressFamily.AF_INET6: 23>, address='::1', netmask=None, broadcast=None, ptp=None)],
 'WLAN': [snicaddr(family=<AddressFamily.AF_LINK: -1>, address='04-EA-56-8C-36-24', netmask=None, broadcast=None, ptp=None),
          snicaddr(family=<AddressFamily.AF_INET: 2>, address='192.168.8.115', netmask='255.255.255.0', broadcast=None, ptp=None),
          snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fd94:e9ee:f230:6e00:55c9:8d1e:f23d:3acc', netmask=None, broadcast=None, ptp=None),
          snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fd94:e9ee:f230:6e00:dc07:1987:2395:d871', netmask=None, broadcast=None, ptp=None),
          snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fe80::55c9:8d1e:f23d:3acc', netmask=None, broadcast=None, ptp=None)],
 '以太网': [snicaddr(family=<AddressFamily.AF_LINK: -1>, address='9C-7B-EF-15-FC-2F', netmask=None, broadcast=None, ptp=None),
         snicaddr(family=<AddressFamily.AF_INET: 2>, address='10.254.61.6', netmask='255.255.255.0', broadcast=None, ptp=None),
         snicaddr(family=<AddressFamily.AF_INET: 2>, address='169.254.54.71', netmask='255.255.0.0', broadcast=None, ptp=None),
         snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fe80::4826:a6a6:b5f4:3647', netmask=None, broadcast=None, ptp=None)],
 '以太网 2': [snicaddr(family=<AddressFamily.AF_LINK: -1>, address='00-FF-B3-BA-07-AE', netmask=None, broadcast=None, ptp=None),
           snicaddr(family=<AddressFamily.AF_INET: 2>, address='2.0.1.32', netmask='255.255.255.0', broadcast=None, ptp=None),
           snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fe80::cc21:26c6:9327:1355', netmask=None, broadcast=None, ptp=None)],
 '本地连接* 2': [snicaddr(family=<AddressFamily.AF_LINK: -1>, address='04-EA-56-8C-36-25', netmask=None, broadcast=None, ptp=None),
             snicaddr(family=<AddressFamily.AF_INET: 2>, address='169.254.45.234', netmask='255.255.0.0', broadcast=None, ptp=None),
             snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fe80::d958:b3fe:ef3d:2dea', netmask=None, broadcast=None, ptp=None)],
 '本地连接* 3': [snicaddr(family=<AddressFamily.AF_LINK: -1>, address='06-EA-56-8C-36-24', netmask=None, broadcast=None, ptp=None),
             snicaddr(family=<AddressFamily.AF_INET: 2>, address='169.254.41.166', netmask='255.255.0.0', broadcast=None, ptp=None),
             snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fe80::6474:c583:1626:29a6', netmask=None, broadcast=None, ptp=None)],
 '蓝牙网络连接': [snicaddr(family=<AddressFamily.AF_LINK: -1>, address='04-EA-56-8C-36-28', netmask=None, broadcast=None, ptp=None),
            snicaddr(family=<AddressFamily.AF_INET: 2>, address='169.254.135.205', netmask='255.255.0.0', broadcast=None, ptp=None),
            snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fe80::5cbd:8913:7499:87cd', netmask=None, broadcast=None, ptp=None)]}

"""

# 返回网卡的详细信息, 包括是否启动、通信类型、传输速度、mtu
pprint(psutil.net_if_stats())
"""
{'Loopback Pseudo-Interface 1': snicstats(isup=True, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=1073, mtu=1500),
 'WLAN': snicstats(isup=True, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=866, mtu=1500),
 '以太网': snicstats(isup=False, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=0, mtu=1500),
 '以太网 2': snicstats(isup=True, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=10, mtu=1400),
 '本地连接* 2': snicstats(isup=False, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=0, mtu=1500),
 '本地连接* 3': snicstats(isup=False, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=0, mtu=1500),
 '蓝牙网络连接': snicstats(isup=False, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=3, mtu=1500)}
"""
```

**查看当前机器的网络连接**

```
from pprint import pprint
import psutil

# 以列表的形式返回每个网络连接的详细信息
# 里面接受一个参数, 默认是 "inet", 当然我们也可以指定为其它, 比如 "tcp"
pprint(psutil.net_connections())
"""
[sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='0.0.0.0', port=1024), raddr=(), status='LISTEN', pid=940),
 sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='0.0.0.0', port=5432), raddr=(), status='LISTEN', pid=7620),
 sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='127.0.0.1', port=10637), raddr=addr(ip='127.0.0.1', port=10638), status='ESTABLISHED', pid=10152),
 sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='127.0.0.1', port=10613), raddr=addr(ip='127.0.0.1', port=10612), status='ESTABLISHED', pid=10152),
 sconn(fd=-1, family=<AddressFamily.AF_INET6: 23>, type=<SocketKind.SOCK_DGRAM: 2>, laddr=addr(ip='::', port=5353), raddr=(), status='NONE', pid=8820),
 sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='127.0.0.1', port=54541), raddr=(), status='LISTEN', pid=2908),
 sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, 
 ....
 ....
 ....
]
"""

# 是不是很方便呢? 在 Linux 中有两个命令可以做到这一点, netstat 和 ss
# $ netstat -nat
# $ ss -nat

# 但是在生产环境中, 线上服务器很多都是最小化安装, 并不能保证每台机器上都有 ss 或者 netstat 命令, 而这个时候 psutil 就派上用场了
```

**查看当前登录的用户信息**

```
from pprint import pprint
import psutil

pprint(psutil.users())  
# [suser(name='satori', terminal=None, host='0.0.0.0', started=1609841661.0, pid=None)]
```

name: 用户名

terminal: 终端

host: 主机地址

started: 登录时间

pid: 进程id

**查看系统的启动时间**

```
from pprint import pprint
import psutil
import datetime

pprint(psutil.boot_time())  # 1585282271.0
print(datetime.datetime.fromtimestamp(psutil.boot_time()))  # 2020-03-27 12:11:11
```

# 进程管理

**psutil 还提供了很多和进程管理相关的功能函数，非常的丰富，我们来看一下。**

**查看当前存在的所有进程的 pid**

```
from pprint import pprint
import psutil

pprint(psutil.pids())
"""
[0,
 4,
 144,
 512,
 536,
 632,
 640,
 664,
 696,
 768,
 776,
 ...
 ...
 ...
]
"""
```

**查看某个进程是否存在**

```
from pprint import pprint
import psutil

pprint(psutil.pid_exists(22333))  # False
pprint(psutil.pid_exists(0))  # True
```

**返回所有进程（Process）对象组成的迭代器**

```
from pprint import pprint
import psutil

pprint(psutil.process_iter())  # <generator object process_iter at 0x000001F12032C9E0>
```

**根据 pid 获取一个进程对应的 Process 对象**

```
from pprint import pprint
import psutil

pprint(psutil.Process(pid=0))  # psutil.Process(pid=0, name='System Idle Process', started='2020-2-27 09:07:47')
```

# 获取进程相关的具体信息

**我们说根据 pid 可以获取一个进程对应的 Process 对象，而这个对象里面包含了该进程的全部信息。**

```
 from pprint import pprint
import psutil

p = psutil.Process(pid=16948)

# 进程名称
print(p.name()) # WeChat.exe

# 进程的exe路径
print(p.exe()) # D:\WeChat\WeChat.exe

# 进程的工作目录
print(p.cwd()) # D:\WeChat

# 进程启动的命令行
print(p.cmdline()) # ['D:\\WeChat\\WeChat.exe']

# 当前进程id
print(p.pid) # 16948

# 父进程id
print(p.ppid()) # 11700

# 父进程
print(p.parent()) # psutil.Process(pid=11700, name='explorer.exe', started='09:19:06')

# 子进程列表
pprint(p.children())
"""
[psutil.Process(pid=17452, name='WeChatWeb.exe', started='09:21:02'), 
 psutil.Process(pid=16216, name='WeChatApp.exe', started='09:21:40'), 
 psutil.Process(pid=13452, name='SogouCloud.exe', started='09:22:14')]
"""

# 进程状态
print(p.status()) # running

# 进程用户名
print(p.username()) # LAPTOP-264ORES3\satori

# 进程创建时间,返回时间戳
print(p.create_time()) # 1561775539.0

# 进程终端
# 在windows上无法使用
try:
    print(p.terminal())
except Exception as e:
    print(e) # 'Process' object has no attribute 'terminal'

# 进程使用的cpu时间
print(p.cpu_times())  # pcputimes(user=133.3125, system=188.203125, children_user=0.0, children_system=0.0)

# 进程所使用的的内存
print(p.memory_info())
"""
pmem(rss=128634880, vms=117067776, num_page_faults=12193918, 
     peak_wset=263921664, wset=128634880, peak_paged_pool=1398584, 
     paged_pool=1329936, peak_nonpaged_pool=313896, nonpaged_pool=152192, 
     pagefile=117067776, peak_pagefile=201670656, private=117067776)
"""

# 进程打开的文件
pprint(p.open_files())
"""
[popenfile(path='C:\\Users\\satori\\Documents\\WeChat Files\\wxid_3ksrps1o47mf22\\Msg\\Media.db-wal', fd=-1),
 popenfile(path='C:\\Users\\satori\\AppData\\Roaming\\Tencent\\WeChat\\All Users\\CefResources\\2581\\qb_200_percent.pak', fd=-1),
 popenfile(path='C:\\Users\\satori\\Documents\\WeChat Files\\wxid_3ksrps1o47mf22\\Msg\\Multi\\MSG0.db-shm', fd=-1),
 popenfile(path='C:\\Program Files\\WindowsApps\\Microsoft.LanguageExperiencePackzh-CN_18362.28.87.0_neutral__8wekyb3d8bbwe\\Windows\\System32\\zh-CN\\dui70.dll.mui', fd=-1),
 popenfile(path='C:\\Users\\satori\\Documents\\WeChat Files\\wxid_3ksrps1o47mf22\\Msg\\Multi\\MediaMSG2.db-shm', fd=-1),
 popenfile(path='C:\\Users\\satori\\Documents\\WeChat Files\\wxid_3ksrps1o47mf22\\Msg\\Emotion.db-wal', fd=-1),
 popenfile(path='C:\\Windows\\Fonts\\msyh.ttc', fd=-1),
 ......
 ......
 ......
 ]
"""

# 进程相关的网络连接
pprint(p.connections())
"""
[pconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='192.168.8.115', port=5162), raddr=addr(ip='183.3.234.107', port=443), status='ESTABLISHED'),
 pconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='192.168.8.115', port=13856), raddr=addr(ip='61.151.168.204', port=80), status='CLOSE_WAIT'),
 pconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, laddr=addr(ip='0.0.0.0', port=8680), raddr=(), status='LISTEN')]
"""

# 进程内的线程数量，这个进程开启了多少个线程
print(p.num_threads())  # 66

# 这个进程内的所有线程信息
pprint(p.threads())
"""
[pthread(id=13340, user_time=113.328125, system_time=179.015625),
 pthread(id=17120, user_time=0.0, system_time=0.0625),
 pthread(id=7216, user_time=0.15625, system_time=0.515625),
 pthread(id=13360, user_time=0.703125, system_time=0.21875),
 pthread(id=10684, user_time=0.015625, system_time=0.078125),
 pthread(id=13552, user_time=2.9375, system_time=0.171875),
 pthread(id=12620, user_time=0.265625, system_time=0.296875),
 pthread(id=14492, user_time=0.015625, system_time=0.03125),
 pthread(id=14568, user_time=0.0, system_time=0.046875),
 pthread(id=17112, user_time=0.015625, system_time=0.0625),
 pthread(id=9344, user_time=0.0, system_time=0.015625),
 pthread(id=13544, user_time=0.0, system_time=0.0),
 pthread(id=10028, user_time=0.078125, system_time=0.125),
 pthread(id=4920, user_time=0.015625, system_time=0.0625),
 pthread(id=5744, user_time=0.0, system_time=0.015625),
 pthread(id=7044, user_time=0.0, system_time=0.0),
 pthread(id=14064, user_time=0.0, system_time=0.0),
 pthread(id=11916, user_time=0.0, system_time=0.0),
 pthread(id=1316, user_time=0.0, system_time=0.0),
 pthread(id=18100, user_time=0.0, system_time=0.0),
 pthread(id=2992, user_time=0.0, system_time=0.0),
 pthread(id=8956, user_time=0.0, system_time=0.0),
 pthread(id=8588, user_time=0.03125, system_time=0.03125),
 pthread(id=3944, user_time=0.0, system_time=0.03125),
 pthread(id=15828, user_time=0.0, system_time=0.015625),
 pthread(id=7348, user_time=0.0, system_time=0.03125),
 pthread(id=3400, user_time=0.0, system_time=0.015625),
 pthread(id=8628, user_time=0.0, system_time=0.0),
 pthread(id=2400, user_time=0.0, system_time=0.0),
 pthread(id=9432, user_time=1.28125, system_time=0.171875),
 pthread(id=11544, user_time=0.0, system_time=0.015625),
 pthread(id=12348, user_time=2.96875, system_time=3.78125),
 pthread(id=3444, user_time=0.0, system_time=0.0),
 pthread(id=17476, user_time=0.0, system_time=0.0),
 pthread(id=15856, user_time=0.0, system_time=0.015625),
 pthread(id=12248, user_time=0.0, system_time=0.0),
 pthread(id=17280, user_time=0.0, system_time=0.0),
 ......
 ......
 ......
 ]
"""

# 进程的环境变量
pprint(p.environ())
"""
{'ALLUSERSPROFILE': 'C:\\ProgramData',
 'APPDATA': 'C:\\Users\\satori\\AppData\\Roaming',
 'COMMONPROGRAMFILES': 'C:\\Program Files (x86)\\Common Files',
 'COMMONPROGRAMFILES(X86)': 'C:\\Program Files (x86)\\Common Files',
 'COMMONPROGRAMW6432': 'C:\\Program Files\\Common Files',
 'COMPUTERNAME': 'LAPTOP-264ORES3',
 'COMSPEC': 'C:\\WINDOWS\\system32\\cmd.exe',
 'DRIVERDATA': 'C:\\Windows\\System32\\Drivers\\DriverData',
 'GOPATH': 'C:\\Users\\satori\\go',
 'HOMEDRIVE': 'C:',
 'HOMEPATH': '\\Users\\satori',
 'LOCALAPPDATA': 'C:\\Users\\satori\\AppData\\Local',
 'LOGONSERVER': '\\\\LAPTOP-264ORES3',
 'NUMBER_OF_PROCESSORS': '12',
 'ONEDRIVE': 'C:\\Users\\satori\\OneDrive',
 'ONEDRIVECONSUMER': 'C:\\Users\\satori\\OneDrive',
 'ONLINESERVICES': 'Online Services',
 'OS': 'Windows_NT',
 'PATH': 'C:\\Program Files (x86)\\Intel\\Intel(R) Management Engine '
         'Components\\iCLS\\;C:\\Program Files\\Intel\\Intel(R) Management '
         'Engine '
         'Components\\iCLS\\;C:\\windows\\system32;C:\\windows;C:\\windows\\System32\\Wbem;C:\\windows\\System32\\WindowsPowerShell\\v1.0\\;C:\\windows\\System32\\OpenSSH\\;C:\\Program '
         'Files (x86)\\Intel\\Intel(R) Management Engine '
         'Components\\DAL;C:\\Program Files\\Intel\\Intel(R) Management Engine '
         'Components\\DAL;C:\\Program Files (x86)\\NVIDIA '
         'Corporation\\PhysX\\Common;C:\\Program '
         'Files\\Intel\\WiFi\\bin\\;C:\\Program Files\\Common '
         'Files\\Intel\\WirelessCommon\\;C:\\python37;c:\\python37\\Scripts;C:\\Program '
         'Files\\Git\\cmd;E:\\instantclient_10_2;C:\\Program '
         'Files\\Redis\\;D:\\ffmpeg\\bin;C:\\WINDOWS\\system32;C:\\WINDOWS;C:\\WINDOWS\\System32\\Wbem;C:\\WINDOWS\\System32\\WindowsPowerShell\\v1.0\\;C:\\WINDOWS\\System32\\OpenSSH\\;C:\\Go\\bin;C:\\MingW\\bin;C:\\Users\\satori\\.cargo\\bin;C:\\python37\\Scripts\\;C:\\python37\\;C:\\python38\\Scripts\\;C:\\python38\\;C:\\Users\\satori\\AppData\\Local\\Microsoft\\WindowsApps;C:\\Users\\satori\\go\\bin',
 'PATHEXT': '.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC',
 'PLATFORMCODE': 'AN',
 'PROCESSOR_ARCHITECTURE': 'x86',
 'PROCESSOR_ARCHITEW6432': 'AMD64',
 'PROCESSOR_IDENTIFIER': 'Intel64 Family 6 Model 158 Stepping 10, GenuineIntel',
 'PROCESSOR_LEVEL': '6',
 'PROCESSOR_REVISION': '9e0a',
 'PROGRAMDATA': 'C:\\ProgramData',
 'PROGRAMFILES': 'C:\\Program Files (x86)',
 'PROGRAMFILES(X86)': 'C:\\Program Files (x86)',
 'PROGRAMW6432': 'C:\\Program Files',
 'PSMODULEPATH': 'C:\\Program '
                 'Files\\WindowsPowerShell\\Modules;C:\\WINDOWS\\system32\\WindowsPowerShell\\v1.0\\Modules',
 'PUBLIC': 'C:\\Users\\Public',
 'REGIONCODE': 'APJ',
 'SESSIONNAME': 'Console',
 'SYSTEMDRIVE': 'C:',
 'SYSTEMROOT': 'C:\\WINDOWS',
 'TBS_CONTENT_MAIN_RUNNER_INITIALIZED': '1',
 'TEMP': 'C:\\Users\\satori\\AppData\\Local\\Temp',
 'TMP': 'C:\\Users\\satori\\AppData\\Local\\Temp',
 'USERDOMAIN': 'LAPTOP-264ORES3',
 'USERDOMAIN_ROAMINGPROFILE': 'LAPTOP-264ORES3',
 'USERNAME': 'satori',
 'USERPROFILE': 'C:\\Users\\satori',
 'VS140COMNTOOLS': 'C:\\Program Files (x86)\\Microsoft Visual Studio '
                   '14.0\\Common7\\Tools\\',
 'WINDIR': 'C:\\WINDOWS',
 'WXDRIVE_START_ARGS': '--wxdrive-setting=0 --disable-gpu '
                       '--disable-software-rasterizer '
                       '--enable-features=NetworkServiceInProcess'}
"""

# 结束进程, 返回 None, 执行之后微信就会被强制关闭, 当然这里就不试了
# print(p.terminal())  # None
```

**我们还可以调用 psutil.test 来模拟 ps 命令。**

```
import psutil

psutil.test()
"""
USER         PID  %MEM     VSZ     RSS  NICE STATUS  START   TIME  CMDLINE
SYSTEM         0   0.0   60.0K    8.0K        runni  Dec30  00:39  System Idle P
SYSTEM         4   0.0  236.0K    1.4M        runni  Dec30  14:32  System
             144   0.2    8.1M   32.2M        runni  Dec30  00:03  Registry
             512   0.0    1.1M  304.0K        runni  Dec30  00:00  smss.exe
             536   0.0  912.0K    1.0M        runni  Dec30  00:00  svchost.exe
             632   0.1   13.0M   15.0M        runni  Dec30  00:29  svchost.exe
             640   0.0    2.1M    1.3M        runni  Dec30  00:00  fontdrvhost.e
satori       664   0.1   26.1M   17.1M    32  runni  09:19  00:02  C:\WINDOWS\Sy
             696   0.0    6.7M    3.8M        runni  Dec30  00:04  WUDFHost.exe
             768   0.0    1.9M    1.9M        runni  Dec30  00:26  csrss.exe
......................
......................
......................
......................
......................
"""
```

**它是怎么做的呢？还记得我们之前说的 process_iter 吗？会返回所有进程的 Process 对象，直接依次输出里面的信息即可。同理，我们也可以通过 process_iter 找到某一个进程对应的进程 id。**

```
import psutil

for prcs in psutil.process_iter():
    if prcs.name().lower() == "wechat.exe":
        print(prcs.pid)
"""
16948
"""

# 有了这个骚操作之后，我们便可以通过进程 id 找到对应的进程
# 然后修改里面的数据
```

# 小结

**总的来说，这个库是非常强大的，很好用，可以获取很多底层的信息。**

[了解更多](https://api.toutiaoapi.com/pgcui/extern_redirect/?item_id=6920046874061701644)