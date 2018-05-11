---
title: 2018-starctf
date: 2018-05-07 20:08:15
categories: CTF
tags: 浅拷贝
toc: true
---

就随便看了2题，做个简单的记录。
### 0x01 Web-simpleweb 

```
var net = require('net');

flag='fake_flag';

var server = net.createServer(function(socket) {
	socket.on('data', (data) => { 
		//m = data.toString().replace(/[\n\r]*$/, '');
		ok = true;
		arr = data.toString().split(' ');
		arr = arr.map(Number);
		if (arr.length != 5) 
			ok = false;
		arr1 = arr.slice(0);
		arr1.sort();
		for (var i=0; i<4; i++)
			if (arr1[i+1] == arr1[i] || arr[i] < 0 || arr1[i+1] > 127)
				ok = false;
		arr2 = []
		for (var i=0; i<4; i++)
			arr2.push(arr1[i] + arr1[i+1]);
		val = 0;
		for (var i=0; i<4; i++)
			val = val * 0x100 + arr2[i];
		if (val != 0x23332333)
			ok = false;
		if (ok)
			socket.write(flag+'\n');
		else
			socket.write('nope\n');
	});
	//socket.write('Echo server\r\n');
	//socket.pipe(socket);
});

HOST = '0.0.0.0'
PORT = 23333

server.listen(PORT, HOST);
```

题意比较简单，输入5个数，经过sort()后，前4个要大于0，后4个要小于127，相邻的不能相等，经过
```
for (var i=0; i<4; i++)
    arr2.push(arr1[i] + arr1[i+1]);
val = 0;
for (var i=0; i<4; i++)
	val = val * 0x100 + arr2[i];
```
迭代后要等于0x23332333。
如果5个数都是整数，则等价于以下条件
```
有5个在0-127之间的并按大小顺序排好的不同的数，
a1+a2 == 0x23
a2+a3 == 0x33
a3+a4 == 0x23
a4+a5 == 0x33
```
但是可以在数学层面简单证明整数是不能实现这个要求的，于是考虑用小数。
算了一早上，终于手动算出5个数。

0x1| 0x21.EF | 0x22.AC | 0x65.89 | 0x98.77
-|
 1 |33.93359375 | 34.671875 | 101.53515625 | 152.46484375

心里还想着明明是数学题，怎么放到web里了。
但当输入结果过去的时候，发现竟然错了！手动验证一遍，没错。
开始调试代码。终于发现
**sort()是按字符串比较大小的！！！**
即：
```
[21,3,1].sort()
(3) [1, 21, 3]
```
也就意味着上面那个数学证明是错的。。。
然后花了3分钟就找到了5个满足要求的整数。。。
`15, 20, 31, 4, 47`


### 0x02 PPC-magic_number
```
Given n(1<=n<=14) integers a1,a2,...,an in interval [0,1024), you should determine them by sending several queries.

For each query, you can ask "how many integers are in interval [l,r)?" through stdout in format "? l r" where 0<=l<r<=1024, and you will recieve an integer through stdin as the answer.

Finally, if all the integers are determined, you should output them in arbitrary order and in format "! a1 a2 ... an".

Please notice that some of the integers can be the same and that you can send no more than 99 queries in each level.

```
用二分法，但是限制了查询次数少于100，对于n>=10时就不能对每个数都二分一次。
主要根据返回的范围内数字个数进行优化。
脚本源码如下：
```
#-*- encoding:utf-8 -*-

import telnetlib
import re
import time

host = "47.89.18.224"
port = "10011"

def Guess(a,level_num):
	#判断是否计算出所有a
	solved = 0
	for i in range(level_num):
		if a[i][1] - a[i][0] == 1:
			solved += 1
	if solved == level_num:
		question = "!"
		for i in range(level_num):
			question = question + " "+ str(a[i][0])
		question = question + "\n"
		print question,
		tn.write(question)
		#tn.read_until("-------------\n")
		#answer = tn.read_until("\n")
		return "level " + str(level_num - 1) + " done!",a
		
	same_all = 1
	for i in range(solved,level_num-1):
		if a[i][0] == a[i+1][0] and a[i][1] == a[i+1][1]:
			same_all += 1
		else:
			break
	
	print "solved:",solved,"same_all:",same_all
	
	question = "? %d %d \n"%(a[solved][0],(a[solved][1]+a[solved][0])/2)
	print question,
	tn.write(question)
	#time.sleep(1)
	same = int(tn.read_until("\n",5))
	
	print same
	
	for i in range(solved,solved+same):
		a[i][1] = (a[i][0]+a[i][1])/2

		
	for i in range(solved+same,solved+same_all):
		t = (a[i][0]+a[i][1])/2
		#a[i][1] = ((a[i][1] - a[i][0])/2) + a[i][1]
		a[i][0] = t

	print a
	return Guess(a,level_num)
	

def one_level():
	#time.sleep(1)
	print tn.read_until("---\n",5)
	#time.sleep(1)
	level = tn.read_until("\n",5)
	level_num = int(re.match(r'Level (.*) : n = (.*)',level).group(2))
	
	#a = [[0]*2]*level_num			#日！！！浅拷贝
	a = [[0 for i in range(2)] for i in range(level_num)]
	for i in range(level_num):
		a[i][0] = 0
		a[i][1] = 1024
		
	print level
	print Guess(a,level_num)

	
tn = telnetlib.Telnet(host,port)
time.sleep(1)
tn.read_until("each level.\n")
for i in range(14):
	one_level()
	#time.sleep(1)

tn.close()
```

写脚本时碰到的坑主要就是：
python生成多维数组
```
a = [[0]*2]*level_num
```
这样生成的数组，是**浅拷贝**。所有的列是同时修改的。
```
>>> a = [[0]*2]*3
>>> a
[[0, 0], [0, 0], [0, 0]]
>>> a[0][0]=1
>>> a
[[1, 0], [1, 0], [1, 0]]
```
正确的写法应该是
```
a = [[0 for i in range(2)] for i in range(level_num)]
```
**浅拷贝**与**深拷贝**
```
浅拷贝相当于引用，只是多了一个指针指向同一个地址。即，所有的a[*][0]都是对a[0][0]的一个引用，所以当修改a[0][0]时，所有的a[*][0]都会改变。
深拷贝就是常说的复制。又开辟了一块空间存储相同内容的数据。地址也就不会相同了。
>>> b = [[0 for i in range(2)] for i in range(3)]
>>> b
[[0, 0], [0, 0], [0, 0]]
>>> b[0][0]=1
>>> b
[[1, 0], [0, 0], [0, 0]]
```
但同时，好像又发现了点有意思的东西。
```
# 给b[0][0]赋完值，其他数组元素地址一样？？？黑人问号
>>> id(b[0][0])
32407976L
>>> id(b[0][1])
32408000L
>>> id(b[1][0])
32408000L
>>> id(b[2][0])
32408000L
# 改个值看看
>>> b[1][0]=1
>>> id(b[1][0])
32407976L
# 看看0,1的地址
>>> id(0)
32408000L
>>> id(1)
32407976L		# 变量地址和对应的值有关
# id(a[0][1]) == id(b[0][1]) == id(0)
>>> id(a[0][1])
32408000L
# 随机字符串的地址会变
>>> id('sdafdas')
43318728L
>>> id('sdafdas')
43318648L
>>> id('sdafdas')
43318408L
# 凭啥下面的又不变？
>>> id('1')
42434648L
>>> id('1')
42434648L
>>> id('1')
42434648L
>>> id(888345)
42992176L
>>> id(888345)
42992176L
>>> id(888345)
42992176L
# 为啥会共用一个地址？
>>> id(8883459999)
32268656L
>>> id(8883459999)
32268656L
>>> id(8883459999)
32268656L
>>> id(888345999912342)
32268656L
>>> id(888345999912342)
32268656L
# 又变了
>>> id(888345999912342234234)
43317448L
>>> id(888345999912342234234)
43317408L
>>> id(888345999912342234234)
43317208L
# 字符串不变？
>>> id('abc')
41990184L
>>> id('abc')
41990184L
# 还是会变，凭啥abc不变？
>>> id('abcxsd')
43318648L
>>> id('abcxsd')
43318528L
>>> id('abcd')
43318488L
>>> id('abcd')
43317768L
>>> id('un')
43318288L
>>> id('un') 
43318248L		# 和字符串长度无关
>>> id('df')
43318008L
>>> id('df')
43318008L
>>> id('e0')
43317568L
>>> id('e0')
43317528L		
>>> id('ae')
43317208L
>>> id('ae')
43318688L		# 并不是因为把df识别成了十六进制数，
```
查看官方文档对`id()`的描述，并没有找到答案。
下面是**不负责任猜测**
貌似对于**一定大小**以内的数字或**某些特殊**字符串，一开始就会存储在某些固定地址，当给变量赋值时，就是把这个变量指向这个数或字符串。
其他的数字和字符串就是随用随时分配一个地址。

可参考[Python中的函数参数的传递问题](https://blog.csdn.net/tycoon1988/article/details/38850443)
[在 Python 中，浅拷贝和用等号引用有何使用上的区别？ - 酱油哥的回答 - 知乎](https://www.zhihu.com/question/28803425/answer/285967047)

