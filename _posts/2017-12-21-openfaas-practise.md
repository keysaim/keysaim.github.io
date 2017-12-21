---
layout:     post
title:      "无服务（Serverless）Openfaas实战"
subtitle:   ""
date:       2017-12-21
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - serverless
    - 教程
---

# 简述

技术的世界真的是一日千里，当有的人还满足与经典的C/S架构时，SOA（面向服务）架构已经遍地都是了；当很多人正打算将系统微服务改造的时候，别人都已经开始玩起无服务器（Serverless）架构了。随着2014年AWS发布Lambda，无服务器架构已经是越来越热了。那么，无服务器（Serverless）到底是个什么东东呢？这里推荐阅读以下这篇文章[从IaaS到FaaS—— Serverless架构的前世今生](https://aws.amazon.com/cn/blogs/china/iaas-faas-serverless/)，对于Serverless，目前业内并没有一个明确的定义，引用这篇文章中所采用的定义如下：“无服务器架构是基于互联网的系统，其中应用开发不使用常规的服务进程。相反，它们仅依赖于第三方服务（例如AWS Lambda服务），客户端逻辑和服务托管远程过程调用的组合。”这么说，估计很多人还是一头雾水，所以还请仔细阅读以下上面推荐的那篇文章。形象的来说，以后开发一个需要网络交互的移动APP，如果这个APP比较简单，基于Serverless架构，你甚至根本不需要进行服务端的开发，这放在以前是根本是无法想象的。当然，这可能是比较理想的情况，很多情况下，你可能还是需要参与服务端的开发，但跟以往决然不同的是，你并不需要开发一个完整的服务端，而只需要设计好几个函数，然后将这些函数上传到服务器提供商，如AWS的Lambda，服务器引擎便帮你提供完整的服务端功能，甚至包括了当流量激增时，需要动态的扩展服务器的数量等工作。虽然仍需要部分服务端的开发工作，但是毫无疑问，Serverless的架构将极大的减少服务端的开发量，同时也将极大的减少运维的工作量。

对于Serverless，目前使用最多的是Faas（Function as a Service）范式。本文将基于开源项目[Openfaas](https://github.com/openfaas/faas)来讲述如何搭建一套Serverless系统。

TODO

