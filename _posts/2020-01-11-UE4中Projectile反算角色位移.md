---
layout:  post
title:		UE4中Projectile反算角色位移
subtitle:	角色的特殊位移方式（二）
date:     2020-01-11
author:   Francis-wu
header-img: img/m_bg.jpg
catalog: true
tags:
    - UE4

---

> 很久没写博客，最近开始更频繁一点。专注UE4的学习，加班时间写点笔记。
>
> 任何问题欢迎邮件:[admin@franciswu.top](admin@franciswu.top)

有时候我们会出现一些特殊的位移需求，前面有个方法是使用Trace计算出点之后，通过Spline来实现弧形轨迹，这里还有一种方法，在已知起点和终点的情况下，反算出一个抛物线的初始速度，然后将角色发射出去。

### `SuggestProjectileVelocity_CustomArc` 函数
![SuggestProjectileVelocity_CustomArc](https://pichost1-1253970255.cos.ap-shanghai.myqcloud.com/LaunchRole/SuggestProjectileVelocity_CustomArc.png)

具体的计算过程可以推导一下。

### 具体应用场景
![launch](https://pichost1-1253970255.cos.ap-shanghai.myqcloud.com/LaunchRole/launch.png)
如图，角色要从下面这个方块跳跃到上面的台阶上，如果我们使用固定的RootMotion美术做起来就比较麻烦了，不同高度限制。这个时候我们就可以通过起点和终点反算出一个抛物线的初始速度然后将角色发射出去。
![](https://pichost1-1253970255.cos.ap-shanghai.myqcloud.com/LaunchRole/BluePrintCall.png)
具体的发射角色逻辑
![](https://pichost1-1253970255.cos.ap-shanghai.myqcloud.com/LaunchRole/CallLauchBP.png)
需要注意的是，Lauch之后角色会进入Fall状态，截图的Launch是用来做悬挂在墙上然后跳上去这个的速度。









