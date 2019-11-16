---
title: 解决gitlab账号修改密码后Sourcetree拉取提交无权限的问题
date: 2019-11-16 23:23:31
urlname: resolve-sourcetree-access-denied
categories: ["技术"]
tags: ["Sourcetree"]
---

某一天，当你使用Sourcetree准备提交刚刚写好的bug时，忽然跳出这样一个提示:

![access-denied](https://s2.ax1x.com/2019/11/16/MBHJc6.png)

那么很可能是你最近修改了git仓库的账号密码，不必惊慌，也不必像只无头苍蝇一样乱找配置，解决办法无非是更新一下密码。我们打开偏好设置:

![menu](https://s2.ax1x.com/2019/11/16/MBHYjK.png)

选择高级，找到账户将其移除:

![remove-account](https://s2.ax1x.com/2019/11/16/MBHNnO.png)

重新提交或者拉取代码，弹出提示重新输入用户名密码，搞定！

![update](https://s2.ax1x.com/2019/11/16/MBHUBD.png)

