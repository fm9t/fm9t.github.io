---
layout:     post
title:      使用.net core进行DDD开发实践
subtitle:   
date:       2024-03-19
author:     ZSJ
header-img: img/post-bg-debug.png
catalog: true
tags:
    - C#
    - .net core
    - DDD
---

## 解决方案结构
    WebApi: 向外部提供Api
    Application: 应用服务接口层，同时包括DTO
    Application.Impl: 应用服务层接口对应的实现，调用领域服务来实现应用层接口
    Domain: 领域层，包括领域服务接口
    Domain.Repository: 领域层的仓储抽象类
    Domain.Repository.Impl: 领域层的仓储抽象类对应的实现
    DomainSvc.Impl: 领域服务接品的实现
    Infrastructure: 基础架构

## 各层的相关约定

### WebApi
用户界面层，向用户提供Web服务，依赖于Application层和Infrastructure层。

### Application
应用服务层，该层有DTO结构和相关服务的接口，由WebApi来调用，此层为纯粹的接口，仅仅依赖于Infrastructure层。

### Application.Impl
应用服务层的实现，依赖于Domain层和Infrastructure层，每一个方法对应着一个用户用例。按DDD的理论可以同时依赖于Domain.Repository层，但在实践中发现能过领域服务来调用仓储会更适合开发，这也是一个取舍吧。

### Domain
领域模型，项目中最重的一层，依赖于Infrastrucure层，单独的领域模型分别放在各自的目录下，每个领域模型目录下除了对应的模型外还有领域服务接口。

### Domain.Repository
领域仓储抽象层，依赖于Domain层和Infrastructure层，在.net core中由于EF Core中的DbContext已经按Unit of Work来设计了，因此没有必要使用仓储接口，可以直接使用继续自DbContext的抽象类来作为仓储，可以参考 [微软文档] (https://learn.microsoft.com/zh-cn/aspnet/core/data/ef-mvc/advanced?view=aspnetcore-8.0#create-an-abstraction-layer)。 之所以使用抽象类，是为了将数据库细节隐藏起来，如果更换数据库，只需要更换数据映射的配置。

### Domain.Repository.Impl
领域仓储抽象层对应的实现，依赖于Domain.Repository层和Infrastrucure层，主要是配置数据表的相关映射，将数据库连接信息和具体的数据库类型关联起来。

### DomainSvc.Impl
领域服务的实现，依赖于Domain， Domain.Repository, Infrastructure层，提供领域服务的实现。

### Infrastructure
基础架构服务，将一些通用操作，非领域模型的操作提取出来，包括定义一些公用常量, 日志服务，邮件服务，拦截器等，不会变化的不需要提供接口，直接实现即可，有可能变化的提供接口，例如缓存。


