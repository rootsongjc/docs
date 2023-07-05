---
weight: 3
linkTitle: 概念
title: 概念
date: '2023-06-21T16:00:00+08:00'
type: book
---

## Rollout

Rollout 是 Kubernetes 工作负载资源，相当于 Kubernetes Deployment 对象。它旨在在需要更高级的部署或渐进式交付功能的场景中替换 Deployment 对象。Rollout 提供以下功能，而 Kubernetes Deployment 无法提供：

- 蓝绿部署
- 金丝雀部署
- 与入口控制器和服务网格集成，用于高级流量路由
- 与度量提供程序集成，用于蓝绿和金丝雀分析
- 基于成功或失败的度量自动升级或回滚

## 渐进式交付

渐进式交付是以受控和逐步的方式发布产品更新的过程，从而降低发布的风险，通常通过耦合自动化和度量分析来驱动更新的自动升级或回滚。

渐进式交付通常被描述为持续交付的演进，将 CI/CD 中实现的速度优势扩展到部署过程中。这是通过将新版本的曝光限制为子集用户，并观察和分析其正确行为，然后逐步增加曝光范围以涵盖更广泛的受众并持续验证正确性来实现的。

## 部署策略

虽然行业已经使用一致的术语来描述各种部署策略，但这些策略的实现在各种工具之间存在差异。为了清楚地了解 Argo Rollouts 的行为，以下是 Argo Rollouts 提供的各种部署策略实现的描述。

### 滚动更新（Rolling Update）

`RollingUpdate` 逐步用新版本替换旧版本。随着新版本的启动，旧版本会缩小以维护应用程序的总数。这是 Deployment 对象的默认策略。

### 重建（Recreate）

重建部署在启动新版本之前删除应用程序的旧版本。因此，这确保了两个版本的应用程序永远不会同时运行，但在部署期间会有停机时间。

### 蓝绿（Blue-Green）

蓝绿部署（有时称为红黑）同时部署新旧版本的应用程序。在此期间，只有旧版本的应用程序将接收生产流量。这使开发人员可以在将实时流量切换到新版本之前对新版本运行测试。

![蓝绿部署示意图](../images/blue-green-deployments.png)

### 金丝雀（Canary）

金丝雀部署将一部分用户暴露给应用程序的新版本，同时为其余流量提供旧版本。一旦验证了新版本的正确性，新版本就可以逐渐替换旧版本。入口控制器和服务网格（如 NGINX 和 Istio）可实现比本地提供的更复杂的金丝雀流量整形模式（例如实现非常细粒度的流量分割或基于 HTTP 头分割）。

![金丝雀部署示意图](../images/canary-deployments.png)

上图显示了具有两个阶段（10％和 33％的流量流向新版本）的金丝雀，但这仅是一个示例。使用 Argo Rollouts，可以根据你的用例定义确切数量的阶段和流量百分比。