---
layout: post
title:  " 以太坊挖矿和EVM "
date:   2018-07-27 11:10:09 +0800
categories: 以太坊 100days
typora-copy-images-to: ../assets/images/2018-07
typora-root-url: ../../blog_alexcode
---
深入浅出ETH原理与智能合约课程笔记


* TOC
{:toc}






## ETHASH 算法

共识算法基础

![](/assets/images/2018-07/2018-07-27-082020.jpg)





![](/assets/images/2018-07/2018-07-27-082114.jpg)



![](/assets/images/2018-07/2018-07-27-082226.jpg)



![](/assets/images/2018-07/2018-07-27-082246.jpg)



![](/assets/images/2018-07/2018-07-27-082408.jpg)





![](/assets/images/2018-07/2018-07-27-082441.jpg)



> 难度炸弹作用： 未来挖矿没有收益， 便于系统升级切换。 





## 深入EVM原理



evm设计理念

![](/assets/images/2018-07/2018-07-27-082518.jpg)

>  栈式虚拟机



evm的操作流程

![](/assets/images/2018-07/2018-07-27-083102.jpg)





EVM常见指令：

![](/assets/images/2018-07/2018-07-27-084631.jpg)







GAS计价策略

![](/assets/images/2018-07/2018-07-27-084711.jpg)



临时/永久存储的生命周期

![](/assets/images/2018-07/2018-07-27-085130.jpg)

> 类比全局变量/局部变量







## Bloomfilter和收据解析



![](/assets/images/2018-07/2018-07-27-095807.jpg)



> 布隆过滤器： 否， 是肯定的； 是， 有一定的概率是错的。 



Recipt and Log

![](/assets/images/2018-07/2018-07-27-100433.jpg)



## 原理部分总结

![](/assets/images/2018-07/2018-07-27-100910.jpg)



