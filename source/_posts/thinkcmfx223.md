---
title: Thinkcmfx2.2.3 File Deletion Vulnerability
date: 2018-08-29 22:10:14
categories: 代码审计
tags: thinkcmf
---

### Official website:
https://www.thinkcmf.com/


### Download link:
http://www.mycodes.net/do/job.php?job=down_encode&fid=45&id=7058&rid=7088&i_id=4759&mid=106&field=softurl&ti=0


### Version:
ThinkCMFX2.2.3


### Vulnerability type:
File Manipulation


### Description:
Thinkcmfx2.2.3 has an arbitrary file deletion vulnerability in the \application\User\Controller\ProfileController.class.php.

A member user can delete any file in the windows server.


### File:
\application\User\Controller\ProfileController.class.php


### Detail

![](code.png)

In function do_avatar , the program doesn’t verify the post parameter ‘imgurl’ , so we can deliver a file path to imgurl . Then we post the data twice to trigger sp_delete_avatar function which can delete any file . 
In line 176 , the program delete any ‘/’ , but in windows we can use ‘\’ to bypass it.


POC:
```
http://localhost/ThinkCMFX/index.php?g=user&m=profile&a=do_avatar
POST:imgurl=..\..\..\data\install.lock
```

First register a member test

Then use the poc twice , we can delete the install.lock file.

Now we can reinstall the website ! 

 
![](reinstall.png)

