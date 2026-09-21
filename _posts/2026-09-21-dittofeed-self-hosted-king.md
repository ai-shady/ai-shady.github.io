---
layout: post
title: "从营销自动化说到 Dittofeed:或许是企业自部署该领域的王者"
date: 2026-09-21 10:00:00 +0800
categories: [营销自动化, 开源]
tags: [Dittofeed, Iterable, CDP, 自部署, ClickHouse, Kafka]
---

营销自动化这个赛道,过去十年被 SaaS 厂商按"每用户每月"的计价方式收了一大笔税。而 Dittofeed 这类项目正在证明:当你把数据主权和单元成本放回自己手里时,整件事的形态会完全不同。这篇文章想聊清楚一个问题——在"企业自部署"这个前提下,Dittofeed 凭什么可能是这个领域里最完整的那个选择。

## 一、营销自动化的三层结构

不管厂商怎么包装,一个客户互动平台底层永远是三层:

1. **数据层**:接入用户事件与属性(identify / track / page / screen)。
2. **决策层**:分群(segment)、编排(journey)、判断谁在什么时刻该收到什么。
3. **触达层**:通过邮件、短信、推送、Webhook 等渠道把消息送出去。

Iterable、Braze、Klaviyo、Customer.io 是这三层的"成品":你付钱,它托底,你得到界面、送达率、合规和 AI 优化。代价是两条:

- **按 profile 计费**,用户越多越贵,且成本和你的用户规模而非你的价值强绑定;
- **数据在别人机房里**,对医疗、金融、以及任何在意 PII 出境的团队,这是一道过不去的坎。

于是有了第三层需求:**我要的是这套内核,但由我自己运营。**

## 二、大多数"开源替代"只替代了三分之一

市面上的开源选择不少,但多数是"单点工具":

- **Novu** 强在通知基础设施(Inbox、多渠道投递),但它不是营销自动化——它不做分群、不做生命周期旅程;
- **Laudspeaker** 是较早的开放源码竞品,但成熟度与生态沉淀有限;
- 各类"开源的 Customer.io"大多只覆盖邮件或只覆盖旅程。

问题在于:营销自动化的价值恰恰来自三层**同时成立**。你缺了分群,旅程就是盲发;你缺了编排,分群就只是一张静态名单。

## 三、Dittofeed 的不同:它是"地基",不是"成品"

Dittofeed 是目前少见的、把三层都做完并且开源的方案(MIT 许可)。拆开看它的技术选型:

| 能力 | 实现 |
| --- | --- |
| 事件接入 | Segment 兼容的 `identify` / `track` / `page` / `screen` / `group` / `batch` API |
| 分群 | 基于 **ClickHouse** 的列式重算,支持 trait / performed / and / or / 订阅组 / 随机分桶 / 手动名单等节点 |
| 编排 | 基于 **Temporal** 的可视化 Journey(Entry / Message / Wait / Delay / Split / Exit) |
| 触达 | 邮件(SendGrid、Postmark、Resend、SES、SMTP、Mandrill)、短信(Twilio)、Webhook、移动推送 |
| 模板 | MJML / HTML + Liquid,外加低代码编辑器 |
| 存储 | **Postgres**(元数据)+ **ClickHouse**(事件与分析) |
| 可选缓冲 | **Kafka**(把 API 层写入 ClickHouse 的路径变成流式) |

有两个设计点值得单独说:

**1. Segment 兼容的 API 是杀手级迁移策略。** 大量团队早已按 Segment 的事件模型埋好点。Dittofeed 直接吃这套协议,意味着迁移成本从"重做数据采集"降到"改一个 endpoint"。

**2. 它是唯一提供自托管"嵌入式组件"的平台。** 你可以把 journey builder、邮件模板编辑器直接嵌进自己的 SaaS,给自己的客户用。这对**垂直 SaaS**是决定性的:你不是在买一个营销工具,你是在给自己的产品长出一个营销模块,还不用把 PII 交出去。

再加上 workspace 级别的数据隔离和(child workspace)多租户模型,它的定位很清楚——**面向"要把能力内化进自己产品"的团队**。

## 四、一个细节,折射出它的工程务实

Dittofeed 的本地开发栈里,消息队列用的是 **Redpanda**,不是 Apache Kafka。

翻提交记录,这次替换发生在 **2023 年 3 月**(commit `fd54650`,PR #68,作者即创始人 Max Gurewitz),提交信息写得很直白:本地开发降低资源占用,生产获得更好性能。

有意思的是,替换之后**上层代码几乎没动**:

- 客户端仍然是 `kafkajs`;
- 环境变量仍然是 `KAFKA_BROKERS` / `KAFKA_USERNAME` / `KAFKA_PASSWORD`;
- compose 里的服务名、broker 主机名,依然叫 `kafka`。

这不是"没清理干净",而是正确判断:**Redpanda 与 Kafka 协议级兼容,换的是实现,不是契约**。重命名反而会破坏所有既有部署脚本与文档。这个选择很能说明 Dittofeed 的工程风格——务实地拿性能与成本,而不追求名义上的"纯粹"。

## 五、诚实地对比 Iterable

把两者放在一张表里,差异会非常清楚:

| 维度 | Iterable(闭源 SaaS) | Dittofeed(开源自部署) |
| --- | --- | --- |
| 渠道 | 邮件 / 短信 / 推送 / in-app / Web Push / 直邮 | 邮件 / 短信 / 推送 / Webhook / WhatsApp / Slack |
| 旅程编排 | 强,含实验与分支 | 有完整可视化编排 |
| A/B 实验 | 强(多变量、发送时间优化) | 原生较弱 |
| AI / 预测 | Brand Affinity、发送时机优化等 | 无,定位是基础设施 |
| 商品目录与推荐 | 有 | 无 |
| 送达率 | 托管 IP、预热、投递保障 | 自己接 ESP,自行承担 |
| 合规 | SOC 2 Type II、HIPAA(高阶套餐) | 自己实现;优势是 PII 不出网 |
| 计费 | 按 profile + 发送量 | 无 per-user 费用,成本=基础设施 |
| 部署 | 纯云 | Docker Compose / K8s / Render / CloudFormation |
| 许可 | 专有 | MIT |

**结论不是"Dittofeed 功能更强"**,而是:**两者在卖的其实是完全不同的东西。**

- Iterable 卖的是"托管 + AI + 合规 + 送达率"的**成品**;
- Dittofeed 卖的是"可控 + 可嵌入 + 无按用户计费"的**地基**。

## 六、它不免费,代价在这里

任何鼓吹自部署的文章如果只谈省钱,都是不诚实的。Dittofeed 的真实成本在于你要运营一整套分布式系统:

- **运维负担**:ClickHouse + Postgres + Kafka(可选)+ Temporal,外加 API / Dashboard / Worker 三个进程。它不是单二进制工具,跨服务的一致性与故障处理要自己扛。
- **延迟地板**:worker 是轮询式架构,相比事件驱动的推模式,存在固有反应延迟,不适合亚秒级送达场景。
- **无 AI 层**:预测、发送时机优化、商品推荐,要么自己做,要么不指望。
- **送达率自负**:邮件最终仍走第三方 ESP,供应商限流与故障仍是你的失败域。
- **部分能力在 EE(企业版)**:多租户等更完整的能力需要商业授权。

换句话说,**Dittofeed 把"按用户计费"换成了"按工程能力付费"**。如果你的团队运维不了 ClickHouse 和 Temporal,省下的钱会以人力的形式还回去。

## 七、那"王者"到底成不成立

我的判断是分场景的:

- **它是王者的场景**:你需要数据主权 / PII 合规,或你要把营销能力**白标嵌入自己的 SaaS**,或你的用户量已经让 SaaS 账单不可接受——在这些前提下,Dittofeed 目前是开源阵营里把"数据 → 分群 → 编排 → 触达"整条链路做得最完整的一个,而且 MIT 许可、有活跃社群。
- **它不是王者的场景**:你要的是开箱即用的 AI 个性化、企业级送达率和合规背书,且团队没有运维分布式栈的意愿——这时买 Iterable 才是理性的。

营销自动化的下一程,大概率不会是"某个 SaaS 打败另一个 SaaS",而是**越来越多的公司选择自己持有这套内核**。Dittofeed 真正的价值,是让这件事第一次变得足够可行。

在这个意义上,说它是"企业自部署这个领域的王者",我认为不过分。

---

*参考来源:*
- *Dittofeed 官方文档:docs.dittofeed.com*
- *Dittofeed 仓库与提交记录:github.com/dittofeed/dittofeed(commit `fd54650` / PR #68,2023-03-07)*
- *Iterable 产品能力与合规信息:iterable.com*
- *第三方对比目录:Product Hunt、selfhost.directory、Zaira Labs 工具指南*
