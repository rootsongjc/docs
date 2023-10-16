---
title: 安全
weight: 3
description: "TSB 架构中的安全考虑。"
---

安全性是融入 Tetrate Service Bridge (TSB) 架构每一层的一个基本方面。 TSB 的方法建立在零信任原则之上，其中安全性是网格管理环境中每个特性和功能的首要考虑因素。本节深入探讨 TSB 如何将安全视为一等公民，并提供全面的措施来保护你的应用程序、支持合规性工作并防止中断。

在本节中，你将深入了解以下关键方面：

- 租赁和抽象基础设施
-  访问控制策略
- 可审计性和日志记录
- 安全通信的服务身份

## 租赁和抽象基础设施

TSB 引入了资源的逻辑层次结构，为你的物理基础设施提供了抽象层。这种抽象实现了超越单个虚拟机 (VM) 或 Pod 的安全考虑和操作。 TSB的管理平面提供了一个结构化的框架，简化了服务配置，使其更安全、更易于管理。

TSB 管理平面中所做的更改适用于环境中的资源集合，而不是处理特定的 VM 或 Pod。这种抽象有助于更好地组织基础设施和理解共享资源，从而最大限度地减少与共享所有权相关的潜在陷阱。

### 资源层次结构

![Tetrate Service Bridge 资源层次结构](../../assets/operations/tsb-resources.svg)

TSB 层级结构的核心是组织。

#### 组织

每个 TSB 安装都包含一个组织，充当 TSB 范围设置的容器。该组织还包括团队、用户和集群。在组织内，资源被分类为租户。

#### 租户

TSB 中的租户代表一组共享资源并在指定工作空间内运行的个人或团队。租赁边界提供访问控制和配置分离。租户可以容纳多个团队，例如安全或开发团队。租户拥有的资源进一步划分为工作区，这些工作区通常与特定团队相关联。在租户级别设置的团队级别权限是继承的，允许团队修改其分配的工作区中的资源。

#### 工作空间

TSB 中的工作空间提供了分区区域，团队可以在其中专门管理其资源和网格配置。工作区下面是组，这些组进一步细分为物理服务，最终细分为单个服务实例。

工作区提供配置流量和安全访问策略所需的粒度。 TSB 允许为整个工作区设置默认配置，这些配置会自动应用于这些工作区中的所有服务。但是，可以在组或配置级别创建覆盖。

## 访问控制策略

TSB 强制执行两个主要类别的访问控制策略：运行时访问策略和用户管理。

### 运行时访问策略

TSB 的逻辑模型允许灵活配置运行时安全策略，例如传输中加密（双向 TLS）和最终用户身份验证（例如基于 JWT 的身份验证）。这些运行时访问策略利用服务身份，这是 TSB 零信任架构 (ZTA) 的基本组件。

用户管理访问策略有助于配置不同用户角色的权限，确保对 TSB 功能进行清晰且精细的访问控制。

###  用户访问策略

TSB 内的每个资源都与定义特定条件下授权访问的访问策略相关联。 TSB 通过集成来自企业身份提供商（例如 LDAP）的数据并使用下一代访问控制 (NGAC) 在内部进行映射来与你的组织结构保持一致。这种一致性导致访问控制策略尊重组织边界并防止用户影响其指定范围之外的资源。

TSB 的用户和团队资源可以通过团队和用户 API 进行管理。然而，TSB 与 LDAP 和 OIDC 等身份提供商的集成可确保大多数用户数据自动同步，从而减少与这些 API 直接交互的需要。

### 用户访问和资源层次结构

TSB 的 RBAC API 允许向 TSB 资源层次结构中的用户和团队分配权限。这些权限在层次结构中向下继承。结合支持自定义角色和细粒度权限的 TSB 复杂的 IAM 系统，用户可以广泛控制谁可以在托管基础设施上执行特定操作。

例如，在受监管的行业中，TSB 授权中央安全团队为整个基础设施配置传输中的加密。他们可以审核整个系统并在组织级别控制加密设置。业务线安全团队可以进一步为其基础设施的特定部分配置安全控制，而应用程序团队可以在不影响安全设置的情况下专注于开发。

## 可审计性和日志记录

TSB 维护对受控资源所做的所有更改的审计跟踪，提供强大的审计功能。 TSB 内的每个资源都可以进行审核，提供全面的详细信息，例如谁进行了更改、何时进行更改以及更改的确切性质。此信息可通过 TSB 审核日志 API 访问。

TSB审计日志API与权限系统紧密结合，确保审计日志仅显示与用户有权访问的资源相关的条目。

## 安全通信的服务身份

TSB 依靠服务身份而不是 IP 或网络位置来增强网络安全性和访问控制。 Istio 支持 TSB 内的服务身份，为每个工作负载提供可验证的身份，以便在运行时进行身份验证和授权。这些身份由 Istio 发布，在运行时轮换，为工作负载之间的安全通信和传输中的加密奠定基础。

TSB 利用这些服务身份使应用程序开发人员和安全团队能够定义精细的访问控制策略。这些策略规定了哪些服务可以通信以及如何通信，同时还限制了潜在攻击的范围。通过使用服务身份，TSB 将访问控制策略与底层网络和基础设施组件解耦，确保其在云、Kubernetes、本地和虚拟机等各种环境中的可移植性。

## 结论

安全性是 TSB 架构的核心原则，深入融入其设计原则和每一层功能中。从通过资源层次结构提供细粒度的访问控制到通过服务身份实现安全通信，TSB 可确保为你的网格管理环境提供强大的安全态势。通过使安全性与组织结构保持一致并提供全面的审计功能，TSB 使组织能够自信地管理资源、遵守合规性要求并维护高度可用且安全的应用程序环境。