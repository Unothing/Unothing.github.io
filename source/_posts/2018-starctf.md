---
title: 2018-starctf
date: 2018-05-07 20:08:15
categories: CTF
tags: 
---

就随便看了2题，做个简单的记录。
### Web-simpleweb 

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


### PPC-magic_number
```
Given n(1<=n<=14) integers a1,a2,...,an in interval [0,1024), you should determine them by sending several queries.

For each query, you can ask "how many integers are in interval [l,r)?" through stdout in format "? l r" where 0<=l<r<=1024, and you will recieve an integer through stdin as the answer.

Finally, if all the integers are determined, you should output them in arbitrary order and in format "! a1 a2 ... an".

Please notice that some of the integers can be the same and that you can send no more than 99 queries in each level.

```
用二分法，但是限制了查询次数少于100，对于n>=10时就不能对每个数都二分一次。
简单优化下就好。

