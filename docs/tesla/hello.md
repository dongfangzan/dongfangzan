---
sidebar_position: 1
---


# Tesla API 简介
:::tip[前言]

这是特斯拉JSON API的中文版文档，大部分原文的内容源自于国外的一篇小哥，原文可以[点击查看](https://tesla-api.timdorr.com/)，不过貌似他也没有更新的很频繁，导致很多内容其实是有缺失的，所以趁着感兴趣，我就连翻译带补充的方式来维护一下这个新的文档吧。

:::


本文主要是源自于特斯拉iOS和Android app所使用的JSON API接口，可以用来监控和控制特斯拉的车（目前主要包括Model S, 3, X, Y）以及PowerWall 产品（特斯拉的储能设备，感兴趣的话可以[查看](https://www.tesla.cn/powerwall)，目前国内貌似没有上市）。
还有一位外国大佬通过逆向特斯拉APP的方式，扒出来特斯拉使用的蓝牙协议以及如何与特斯拉车机进行交互的文档，感兴趣的也可以看下[特斯拉低功耗蓝牙接口](https://teslabtapi.lexnastin.com/)。

有你的支持，我后续也会把蓝牙部分补充完整

## 感谢与支持
<a href="https://github.com/dongfangzan/tesla-java-sdk" target="_blank" rel="noopener noreferrer">
  <img src="https://img.shields.io/github/stars/dongfangzan/tesla-java-sdk?style=social" style={{ top: '5px', position: 'relative' }} alt="GitHub Stars" />
</a>
<a href="https://github.com/dongfangzan/tesla-java-sdk" target="_blank"><img src="https://img.shields.io/github/forks/dongfangzan/tesla-java-sdk?label=Fork&style=social" style={{ top: '5px', position: 'relative' }}></img></a>
如果这份文档对你有帮助，欢迎到GitHub给我一颗星星[Star](https://github.com/dongfangzan/tesla-java-sdk)，你的支持就是我前进的最大动力。
## 术语
有区别与原作者的文档，由于特斯拉是一家美国企业，绝大部分车机信息的用词如果实际翻译成中文的话会显得十分奇怪，我会在文档中尽量采用特斯拉官方的中文翻译，如果找不到，那么在下面的表中我列示了特斯拉车辆的术语，用于统一文档中的说法。

英文术语|中文术语|说明
:-|:-|:-
Authentication|认证|特斯拉使用了oauth的方式来进行SSO单点认证
Token|令牌|在认证过后，主要用该令牌表明身份，后续所有读取数据、执行指令的操作都需要该令牌
Vehicle|车辆|不多解释
State|状况|有朋友看到这个单词可能会觉得这不是状态的意思么，但其实在特斯拉接口中主要读取车辆的基本信息、充电信息、驾驶信息等等，与状态的含义并不相同

## 在开始之前
特斯拉所有的api的base URI地址是`https://owner-api.vn.cloud.tesla.cn/` （不包含流和自动泊车API）。
> 特斯拉在国内和国外的地址是不一样的，如果你不在国内，可以使用`https://owner-api.teslamotors.com/`。
> 同理获取token接口中国内的域名是`auth.tesla.cn`，国外的域名是`auth.tesla.com`

所有的请求需要在请求头中增加一个`User-Agent`，可以在请求头中识别请求来源。
## API结构
车子的API主要分成3大部分
### 状态和指令
状态API一般是用于获取当前时间点车辆的状态数据，如车辆的车架号、温度、位置等等信息。
指令API用于控制车辆发出动作，如按喇叭、锁门、远程启动等。
### 流
流API可以每半秒读取一次车辆的遥测数据，你可以通过这些数据实时获取车辆位置信息、充电信息、驾驶信息等等，如果有用过[teslamate](https://docs.teslamate.org/)的同学，可能会有直观的感受。
### 自动泊车（召唤）
这一块API需要至少购买EAP（增强版自动辅助驾驶功能）或者FSD（完全自动驾驶能力）。这一块API主要可以通过一个流指令来控制具有HW1、HW2和HW2.5芯片的车辆自动停车和召唤。实现方式主要是使用了WebSocket