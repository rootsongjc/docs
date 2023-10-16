---
title: TSB 支持策略
description: TSB 支持策略，发布计划和组件版本矩阵。
weight: 2
---

本文档提供有关 TSB 的发布节奏、不同版本的支持期以及组件版本矩阵的信息。

##  TSB 发布说明

TSB 遵循基于 semver.org 的语义版本控制模型。每个版本号代表的含义如下：

- `MAJOR` ：因不兼容的 API 更改而增加。
- `MINOR` ：针对新功能增加。
- `PATCH` ：针对错误和安全修复而增加。

### TSB 发布节奏和支持

- TSB 定期且至少每季度发布 `PATCH` 更新 (1.x.y)。
- 每个新的 `MINOR` 版本都属于长期支持 (LTS) 政策。
- Tetrate 为 LTS 版本提供从一般可用性 (GA) 到一般支持终止 (EoGS) 的支持，通常在 GA 后 12 个月设置。
- 在支持窗口期间通过 `PATCH` 版本提供错误和安全修复。
- 新功能不会向后移植到 LTS 版本。

### TSB 候选版本

候选版本提供对新功能的早期访问以进行测试，但不建议用于生产使用。

{{<callout warning "发布候选版本">}}

候选版本可能包含已知或未知的错误，并且不适合生产使用。

{{</callout>}}

##  支持的版本

Tetrate 根据以下时间表提供支持和补丁：

| TSB 版本   |         一般可用性 |        一般支持结束 | Kubernetes 版本 | OpenShift 版本 |
| :--------- | -----------------: | ------------------: | :-------------- | :------------- |
| TSB v1.6.x |  2023 年 1 月 1 日 | 2023 年 12 月 31 日 | 1.22 - 1.26 [1] | 4.7 - 4.12 [1] |
| TSB v1.5.x | 2022 年 7 月 15 日 |  2023 年 7 月 15 日 | 1.19 - 1.24     | 4.6 - 4.11     |
| TSB v1.4.x | 2021 年 11 月 1 日 | 2022 年 10 月 31 日 | 1.18 - 1.21 [2] | 4.6 - 4.8      |
| TSB v1.3.x |  2021 年 6 月 1 日 |  2022 年 5 月 31 日 | 1.18 - 1.20     | 4.6 - 4.8      |
| TSB v1.2.x |  2021 年 5 月 1 日 |  2022 年 4 月 30 日 | 1.18 - 1.20 [3] | 4.6 - 4.8      |
| TSB v1.1.x |  2021 年 4 月 1 日 |  2022 年 3 月 31 日 |                 |                |
| TSB v1.0.x |  2021 年 3 月 1 日 |  2022 年 2 月 28 日 |                 |                |

###  注意

- [1] Kubernetes 1.25、1.26 和 OpenShift 4.12 需要 TSB 1.6.1 或更高版本。
- [2] Kubernetes 1.21 和 TSB 1.4.x - 支持但有警告，请参阅 Tetrate 支持。
- [3] Kubernetes 1.19 和 1.20 以及 TSB 1.2.x - 支持但有警告，请参阅 Tetrate 支持。

## TSB 组件版本矩阵

Tetrate Service Bridge 包括以下开源组件：

| TSB   | Istio | Envoy    | SkyWalking            | Zipkin   | OpenTelemetry Collector |
| :---- | :------- | :------ | :------------------ | :----- | :------------- |
| 1.6.3 | 1.15.7   | 1.23.11 | 9.4.0-20230726-0846 |        | 0.81.0         |
| 1.6.2 | 1.15.7   | 1.23.7  | 9.4.0-20230407-0339 |        | 0.77.0         |
| 1.6.1 | 1.15.7   | 1.23.7  | 9.4.0-20230331-1055 | 2.24.0 | 0.70.0         |
| 1.6.0 | 1.15.2   | 1.23.2  | 9.4.0-20221215-0956 | 2.24.0 | 0.62.1         |