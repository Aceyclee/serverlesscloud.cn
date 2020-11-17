---
title: 腾讯 IMWEB 前端团队一站式 Serverless 开发解决方案
description: 本文将分享 IMWEB 团队对 Serverless 的实践方案
date: 2020-11-11
thumbnail: https://img.serverlesscloud.cn/20201111/1605093413809-QQ20201111-191440%402x.jpg
categories:
  - best-practice
authors:
  - 陈丽杭
tags:
  - Serverless
  - IMWEB
---

IMWeb 团队隶属腾讯公司，是国内最专业的前端团队之一。

IMWeb 团队专注前端领域多年，负责过 QQ 资料、QQ 注册、QQ 群等亿级业务。目前聚焦于在线教育领域，精心打磨 腾讯课堂、企鹅辅导及 ABCmouse 三大产品。

学习成就梦想，我们希望能用技术改变教育，改变世界。

![](https://img.serverlesscloud.cn/20201116/1605494640818-image%20%284%29%20%282%29.jpg)

> 前言：如今的 Serverless 可以说是一大有潜力的新技术方向，尤其在当下上云的热潮中，Serverless 因其免运维、自动扩容、支持多种编程语言等优势，对前端来说，是一大提升服务开发、维护效率的利器也是可尝试全栈发展的方向，但也因为其新，对落地到团队开发中，结合团队开发流也是遇到了一些挑战，本文将分享 IMWEB 团队对 Serverless 的实践方案

## 一、IMWEB 团队 Serverless 研发模式的演进与思考

在过去一、两年，我们团队在多个服务项目中尝试使用 serverless，腾讯云 Serverless 提供了一站式服务，通过使用该服务，前端可独立完成接口服务开发，对前端个人而言可往全栈发展，也因此可缓解团队后台人力紧张问题

![img](https://img.serverlesscloud.cn/20201111/1605082170041-image0.jpeg)            


在开发 Serverless 云函数的过程中，我们也遇到了对比传统服务，云函数开发的一些挑战点

### （1）云函数开发特点

前端传统项目的开发流模式相对已经比较成熟，通过 git 协同管理代码， 再通过 CI 来规范项目的部署流程，整个工作流可以查看、回滚代码，部署也做到了自动化

![img](https://img.serverlesscloud.cn/20201111/1605082170253-image0.jpeg)            

再来看云函数的开发特点：

- 云函数独立的账号和权限管理
- 以函数为单位进行创建、更新和部署
- 创建网关 API 与函数关联，借此可通过网关 API 访问到云函数

以上是最基础的开发云函数三个基础

![img](https://img.serverlesscloud.cn/20201111/1605082169342-image0.jpeg)            

而云函数的创建、更新有两种方式：

- 腾讯云官网云函数控制台，可视化的操作界面，点击按钮即可创建、更新
- 通过 CLI 创建，SERVERLESS 提供 SDK，调用 SDK 可完成自定义创建、更新操作，其优点为灵活编写，也易于做成工程化

考虑团队的协作，第二种方式通过调用 SDK 的方式因其灵活更适合定制为团队规范

![img](https://img.serverlesscloud.cn/20201111/1605082168683-image0.jpeg)            

总结下来可以看到云函数开发的三个特性：

- 因其有独立于 git 账号的云函数账号，导致了云函数的代码缺乏像 GIT 一样可以查看历史代码版本，代码修改记录等
- 因其有多重方式可以用来创建、更新函数，导致多人协作时，有互相覆盖云函数的风险
- 提供的云函数网关，可帮助快速配置访问云函数，而无需运维同学帮忙做域名指向，机器申请等

![img](https://img.serverlesscloud.cn/20201111/1605082170037-image0.jpeg)            

### （2）团队协作上手云函数开发问题

在初期团队探索尝试云函数开发时，对比传统项目的开发流，云函数的开发步骤更多，也暴露出了一些缺点：  

![img](https://img.serverlesscloud.cn/20201111/1605082168445-image0.jpeg)            

#### 1) 上手成本高

首先有不小的学习成本，像云函数配置文件，云函数官网界面操作学习成本，实际使用时，由于云函数网关 API 链接过长、域名限制等，需要配置 nginx，用特定域名访问云函数网关 API，因为多数前端对 nginx 部署，导致有了 nginx 学习成本

![img](https://img.serverlesscloud.cn/20201113/1605252781340-JLHuakQNrjSbk4ZDQgHX_Q.jpeg)            

#### 2) 调试云函数效率低

因为云函数是部署在云端的，Serverless 有其独特的环境，context、event 等，有别于 NODE 服务的请求体等，本地要完全模拟 serverless 请求比较困难，导致开发想要调试定位问题时，只能先将代码部署到 serverless 上，这里就需要等待部署了，由于 serverless 是外网的，部署时间就更长了

![img](https://img.serverlesscloud.cn/20201111/1605082167641-image0.jpeg)            

#### 3) 开发流困惑

- 由于云函数直接就是部署在云端，没有我们传统的机器用于做环境区分，对团队协作保证部署质量来说并不友好
- 上述也有提到的，往往因为想要自己业务域名访问服务接口，而云函数网关 API 是比较长的缺乏语义化的链接，通常使用时会想配置 nginx 去通过自定义域名访问云函数，不止是成本问题也有容易配置错误的风险问题

![img](https://img.serverlesscloud.cn/20201113/1605252814797-4dgRVX4WesEZ_0izMi7UUA.jpeg)            

#### 4) 管理困难，存在质量问题

因云函数独立的账号管理，没有 git 进行管理，导致无法追踪代码记录，甚至任何有权限的人创建同名函数进行部署都会导致函数莫名被覆盖，同理云函数网关 API 也可以随意更改指向其它云函数

![img](https://img.serverlesscloud.cn/20201111/1605082168265-image0.jpeg)            

总结下来，在团队协作 SCF 开发的时候，遇到的挑战点如下：

![img](https://img.serverlesscloud.cn/20201111/1605082166593-image0.jpeg)

## 二、IMFLOW 一站式 Serverless 开发解决方案的破局与落地

总结上面的云函数在团队协作中遇到的一些问题，对应地提出解决方案：

- 制定规范保证统一的协作，统一的规范保证统一的工作流，提升开发效率进而保证质量
- 优化云函数开发体验，通过工具去自动化完成重复冗余的操作，并通过封装过滤掉一些开发学习成本
- 根据云函数特点制定 CI 和 CD，保证流程统一，也提升部署效率；统一网关规则，减少云函数网关 API 学习和操作            

![img](https://img.serverlesscloud.cn/20201111/1605082168042-image0.jpeg)            

### （1）制定规范，提升协作效率

#### 1） 统一云账号管理

对于独立云函数账号，每个开发在上手开发前都需要单独申请，同时还有开通各种权限，快点半天，慢点一两天，针对这个问题，考虑使用团队公共账号进行统一云函数管理，工具使用公共账号进行云函数部署、更新，免去开发的学习成本、账号上手成本

![img](https://img.serverlesscloud.cn/20201113/1605257479797-0yHaN3xT2HsXhPl9GUtKhQ.jpeg)            

#### 2）基于 GIT 管理云函数

对于云函数独立的管理方式，为了能唯一追踪云函数，保留了原有的 git 管理项目代码，制定一系列规范，将 git 项目与云函数唯一关联，保证云函数唯一不可覆盖

![img](https://img.serverlesscloud.cn/20201111/1605082168186-image0.jpeg)

#### 3）命名空间隔离函数环境

为提供云函数的开发流，针对云函数的特点，使用云函数命名空间的概念来隔离云函数，同时限制测试环境的网关服务只允许内网访问，保证业务安全

![img](https://img.serverlesscloud.cn/20201113/1605257560655-obp-7_y4rDmqmwZidPtuqw.jpeg)      

#### 4）统一云函数规则配置

制定云函数名、对应网关服务 API 名、环境命名空间的命名规范，以达到命名空间、函数名、网关服务 API 能一一对应，可通过其一推导其二，如知道函数名，可知其访问 API 是什么，对应环境命名空间是什么

![img](https://img.serverlesscloud.cn/20201111/1605082168183-image0.jpeg)            

### （2）自研 CLI 工具， IMFLOW 提升 SCF 研发效率

在第一项制定了规范之后，要让规范落地，就需要使用工具来辅助，IMWEB 团队自研了 CLI 工具 -- IMFLOW， 提供 SCF 团队开发流实践方案，通过工具的方式提升 SCF 研发效率； 诸如创建账号、申请权限、创建云函数、开发云函数调试、云函数网关 API 关联、函数隔离等等，通过 CLI 工具， 输入命令即可完成。

![img](https://img.serverlesscloud.cn/20201111/1605082166187-image0.jpeg)            

#### 1）上手开发更快

使用了 CLI 工具来辅助之后，对比团队过往的开发模式，通过 CLI 可达到 2 分钟上手进入开发

![img](https://img.serverlesscloud.cn/20201111/1605082164651-image0.jpeg)            

#### 2）调试体验同传统服务开发一致

通过同构 + 构建的方式，保留传统服务开发体验，工具封装屏蔽了云函数文件，开发者开发时可同以往一样

![img](https://img.serverlesscloud.cn/20201111/1605082161521-image0.jpeg)            

#### 3）一键定位调试云函数

云函数的真实运行环境相对复杂，若是遇到了涉及云函数环境调试的问题，需要真实调试云函数，此时本地即可完成调试，工具封装了一系列操作，如实时调试、监听文件变更等，实时部署，实现一键定位调试云函数

![img](https://img.serverlesscloud.cn/20201113/1605257728660-zKuoQMZlffubmmmBhpTH2w.jpeg)            

#### 4）极致优化云函数部署时间

云函数的部署是走的外网部署，而云函数的部署时间影响到了云函数的发布时间，甚至在做本地实时调试云函数时，影响了云函数的调试效率，为了极致优化云函数部署时间，利用了云函数的 layer 功能、项目的 node_module 变动几率较小、同时代码包大小会影响部署时间这些特点，对云函数项目部署进行了拆分，当 node_modules 没有变动时无需部署 node_modules，进而减少了了部署时间

![img](https://img.serverlesscloud.cn/20201111/1605082164795-image0.jpeg)            

在做了部署优化后，查看项目的部署时间，大部分时间 35s 即可完成函数部署

![img](https://img.serverlesscloud.cn/20201111/1605082161977-image0.jpeg)            

### （3）质量保证

在质量保证方面，主要是通过 CI | CD 规范部署流程，制定网关服务规范来隔离云函数和降低网关配置成本。

![img](https://img.serverlesscloud.cn/20201111/1605082160952-image0.jpeg)            

限制测试环境网关服务为内网可访问。

![img](https://img.serverlesscloud.cn/20201113/1605252917296-Cr0lwD5xyck3Ugjxlk1fIw.jpeg)            

另外，为了保证云函数的运行稳定，避免因为云函数的冷启动导致云函数访问失败，即对云函数的容灾处理，做了一层 STKE 的容灾，通过代码同构的方式，利用工具构建打包，完成一套代码实现既可部署 serverless ，也可以部署 STKE， 配合网关的处理，完成云函数的降级容灾

![img](https://img.serverlesscloud.cn/20201111/1605082163848-image0.jpeg)            

## 三、IMFLOW 使用

### imflow 内置命令

![img](https://img.serverlesscloud.cn/20201111/1605082164582-image0.jpeg)

至此，感谢阅读。在探索云函数的开发之路中，感谢腾讯云 Serverless 团队的支持，希望 Serverless 可以越做越好！

---

---
<div id='scf-deploy-iframe-or-md'></div>

---

欢迎访问：[Serverless 中文网](https://serverlesscloud.cn/)，您可以在 [最佳实践](https://serverlesscloud.cn/best-practice) 里体验更多关于 Serverless 应用的开发！