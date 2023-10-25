---
weight: 1
title: "SPIFFE ID 和 SVID"
---

SPIFFE 标准提供了一个规范，用于在异构环境和组织边界中引导和颁发服务的身份。它包括各种规范，每个规定了 SPIFFE 功能的特定子集的操作。

特别是本文档作为 SPIFFE 标准的核心规范。虽然在 SPIFFE 范围内还有其他规范，但符合本文档就足以实现 SPIFFE 合规性，并获得 SPIFFE 标准本身的互操作性好处。

## 引言

本文档提出了正式的 SPIFFE 规范。它定义了 SPIFFE 标准的两个最基本组件：SPIFFE 身份和 SPIFFE 可验证身份文档。

第 2 节概述了 SPIFFE 身份（SPIFFE ID）及其命名空间。SPIFFE ID 被定义为符合[RFC 3986](https://tools.ietf.org/html/rfc3986)标准的 URI，包括“信任域名”和相关路径。信任域名作为 URI 的授权组件，用于识别发放给定身份的系统。以下示例演示了如何构造 SPIFFE ID：

```spiffe://trust-domain-name/path```

有效的 SPIFFE ID 必须将方案设置为`spiffe`，包含非零的信任域名，并且不能包含查询或片段组件。换句话说，SPIFFE ID 由`spiffe`方案和一个特定站点的`hier-part`（其中包括授权组件和可选路径）完全定义。

### 信任域
信任域对应于系统的信任根。信任域可以代表独立运行其自己的 SPIFFE 基础设施的个人、组织、环境或部门。

信任域名称通常是自我注册的，与公共 DNS 不同，没有委托权机构来断言并注册基本域名到实际的法律实体，或者断言该法律实体对任何特定信任域名拥有公正和正当的权利。

信任域名被定义为 URI 的授权组件，并应用以下限制：
- 授权组件的`host`部分不得为空。
- 授权组件的`userinfo`和`port`部分必须为空。
- 授权组件的`host`部分必须小写。
- 授权组件的`host`部分只能包含字母、数字、点、破折号和下划线（[a-z0-9.-_]）。
- 授权组件的`host`部分不能包含百分比编码的字符。

请注意，此定义不排除用点分四段表示法表示的 IPv4 地址，但排除了 IPv6 地址。DNS 名称是有效信任域名的严格子集。实现在处理信任域名时，无论它们是有效的 IP 地址还是有效的 DNS 名称，都不得以不同方式处理它们。

#### 信任域名称冲突
信任域操作员可以自由选择任何他们认为合适的信任域名称：没有中央权威机构来监管或注册信任域名称。因此，不能保证全局唯一性，也没有技术手段阻止不同的信任域使用相同的信任域名称。

为防止意外碰撞（两个信任域选择相同的名称），建议操作员选择高度可能全球唯一的信任域名称。即使信任域名称不是 DNS 名称，但如果可用，使用注册的域名作为信任域名的后缀将降低意外碰撞的可能性；例如，如果信任域操作员拥有域名`example.com`，那么使用类似`trust_domain_name.example.com`的信任域名可能不会产生冲突。当信任域名在没有操作员输入的情况下自动生成时，强烈建议随机生成一个唯一的名称（例如 UUID）。

发生冲突时，这些信任域将继续独立运行，但将无法联合（相互连接）。因为每个信任域使用独特的信任根，由一个信任域发放的身份声明将在另一个信任域中验证失败。

### 路径
SPIFFE ID 的路径组件允许唯一标识给定的工作负载。路径的含义是开放式的，由管理员负责定义。

有效的 SPIFFE ID 路径组件必须遵循以下规则：
- 路径组件不能包含百分比编码的字符。
- 路径组件不能包含空段或相对路径修饰符（即`.`、`..`）。
- 路径组件不能以斜杠结尾。
- 单个路径段只能包含字母、数字、点、破折号和下划线（[a-zA-Z0-9.-_]）。

路径可以是分层的，类似于文件系统路径。路径的具体含义保留给实施者，不属于 SVID 规范的范围之内。以下是一些示例和约定。

* 直接标识服务

  通常，直接标识服务是有价值的。例如，管理员可能会决定在特定一组节点上运行的任何进程都应该能够以特定的身份呈现自己。例如：

  ```spiffe://staging.example.com/payments/mysql```
  或
  ```spiffe://staging.example.com/payments/web-fe```

  上述两个 SPIFFE ID 指代了两个不同的组件 - mysql 数据库服务和一个运行在暂存环境中的支付服务的 web 前端。环境“staging”的含义和“payments”作为高级服务集合的含义由实施者定义。

* 标识服务所有者

  通常，更高级别的编排器和平台可能已经内置了它们自己的身份概念（如 Kubernetes 服务账户或 AWS/GCP 服务账户），直接将 SPIFFE 身份映射到这些身份是很有帮助的。例如：

  ```spiffe://k8s-west.example.com/ns/staging/sa/default```

  在这个示例中，example.com 的管理员正在运行一个名为 k8s-west.example.com 的 Kubernetes 集群，该集群有一个“staging”命名空间，在其中有一个名为“default”的服务账户（sa）。这些都是由 SPIFFE 管理员定义的约定，而不是本规范所保证的断言。

* 不透明的 SPIFFE 身份

  上述示例是说明性的，在最一般的情况下，SPIFFE 路径可能是不透明的，不包含任何可见的分层信息。例如，地理位置、逻辑系统分区和/或服务名称等元数据可以由注册身份及其属性的次级系统提供。可以查询以检索与 SPIFFE 标识符相关联的任何元数据。例如：

  ```spiffe://example.com/9eebccd2-12bf-40a6-b262-65fe0487d453```

### 最大 SPIFFE ID 长度

如[RFC 3986](https://tools.ietf.org/html/rfc3986)定义的 URI 没有最大长度。出于互操作性考虑，SPIFFE 实现必须支持最长为 2048 字节的 SPIFFE URI，并且不应生成长度大于 2048 字节的 URI。[RFC 3986](https://tools.ietf.org/html/rfc3986)仅允许 ASCII 字符，因此 SPIFFE ID 的推荐最大长度为 2048 字节。

所有 URI 组件都会影响 URI 的长度，包括“spiffe”方案、“：//”分隔符、信任域名和路径组件。非 ASCII 字符在将其编码为 ASCII 字符后会影响 URI 的长度。请注意，[RFC 3986](https://tools.ietf.org/html/rfc3986)为 URI 的“host”组件定义了最大长度为 255 个字符；因此，信任域名的最大长度为 255 字节。

### SPIFFE ID 解析

SPIFFE ID 遵循由[RFC 3986](https://tools.ietf.org/html/rfc3986)定义的 URI 规范。SPIFFE ID 的方案和信任域名对大小写不敏感，而路径对大小写敏感。

## SPIFFE 可验证身份文档
SPIFFE 可验证身份文档（SVID）是工作负载将其身份通信给资源或调用者的机制。如果 SVID 已由 SPIFFE ID 所在信任域内的授权方签名，则认为 SVID 是有效的。

### SVID 信任
SPIFFE 信任根植于给定 ID 的信任域。每个信任域必须存在一个签名授权机构，该授权机构必须携带自己的 SVID。签名授权机构的 SPIFFE ID 应该驻留在其具有权威性的信任域中，并且不应具有路径组件。授权机构的 SVID 然后形成了给定信任域的信任基础。

如果需要，可以通过使用外部信任域授权机构的私钥对授权机构的 SVID 进行签名来实现信任链。如果不需要链接信任，那么授权机构的 SVID 将进行自签名。

### SVID 组件
SVID 是一个相当简单的构造，包括三个基本组件：

- 一个 SPIFFE ID
- 一个有效的签名
- 一个可选的公钥

SPIFFE ID 和公钥（如果存在）必须包含在签名的有效载荷的一部分中。如果包含了公钥，则相应的私钥将由发放 SVID 的实体保留，并用于证明对 SVID 本身的所有权。

个别的 SVID 规范可能要求或以其他方式允许在 SVID 中包含超出此处描述的内容。所包含信息的性质可能或可能不会严格由相关的 SPIFFE 规范定义 - 例如，JWT-SVID 规范允许用户在 SVID 本身中包含任意信息。在相关 SVID 规范未明确指定此附加信息的情况下，操作者在将此信息用作安全决策的输入时应格外小心，特别是如果要验证的 SVID 属于不同的信任域。有关更多信息，请参阅安全注意事项部分。

### SVID 格式
SVID 本身不是一种文件类型。已经存在许多文件格式可以满足 SPIFFE SVID 的需求，我们不希望重新发明这些格式。相反，我们定义了一组特定于格式的规范，规范化了 SVID 信息的编码。

为了使 SVID 被视为有效，它必须利用已定义相应规范的文件类型。在撰写本文时，唯一受支持的文件类型是 X.509 和 JWT。请注意，特定于格式的 SVID 规范可能会升级本文中规定的要求。

## 安全注意事项
本节包含在使用 SPIFFE ID 和 SVID 时实施者和用户应考虑的安全注意事项。

### SVID 断言
SVID 始终包含一组数据 - 至少是一个 SPIFFE ID。有时，此数据代表了信任域授权机构对 SVID 主体所做的断言。在从此数据中解释含义时，必须小心确保所有涉及方都充分理解所使用信息的含义和重要性。

在考虑给定断言的相对安全性时，有四个主要问题：

- 首先是时间上的准确性 - SVID 在到期之前一段时间内是有效的，SVID 中的断言在 SVID 的整个生命周期内是否为真？
- 其次，断言的范围和影响 - 断言最初是在什么上下文下进行的，它的影响有多大？
- 第三是解释和含义的问题 - 断言对授权机构和消费者是否具有相同的含义或解释，或者存在着不同的解释可能性？
- 最后，断言本身的真实性在某些情况下可能会受到质疑。

本节探讨了这四个关注领域的所有方面，并提供了操作者可以评估任何给定 SVID 断言的相对安全性的指导方针。一般来说，操作者应该以谨慎为原则，只包含那些对所涉及的断言的安全性具有非常高度信心的断言。

值得注意的是，虽然通常由 SPIFFE 规范直接形式化的断言通常不容易受到与解释和含义相关的问题的影响，但它们仍然可能容易受到与真实性相关的问题的影响。但是，由于 SPIFFE 定义的断言的范围非常有限，因此在这方面的真实性问题表明了与问题相关的信任域的安全姿态的更大问题，此时操作者应该认真考虑是否应该在第一时间与这些系统交换数据。

#### 时间上的准确性
SVID 在一段有限的时间内有效，主要是为了降低密钥被泄露和相关损害的可能性。虽然通常情况下，SVID 中的断言在签发时是真实的，但并不一定意味着在使用时也是真实的。

某些类型的断言比其他类型更容易受到此问题的影响。服务所有者的名称、角色或组成员资格以及访问策略都是在 SVID 签发时和验证或使用时之间更有可能发生变化的示例。相反，工作负载及其运行时的自然属性（例如 SPIFFE ID 或工作负载所在的区域）通常与工作负载的生命周期绑定，因此不太可能发生变化，这使得它们不太容易受到时间上的准确性的问题影响。

在决定是否应该在 SVID 中包含某个特定的断言时，考虑到这一点是很重要的。在 SVID 中作出的断言将被认为在 SVID 的生命周期内都是有效的，并且对于具有旧断言的所有 SVID 来说，将首先过期，因此在活动系统上对此断言进行更改（或撤销）将会很费时。如果对于所考虑的断言的波动性不清楚，操作者应该以谨慎为原则，并将其排除在 SVID 之外。

#### 范围和影响
SVID 由位于其信任域中的授权机构签名。签名授权机构有责任验证其签署的 SVID 中的所有信息，而包含在 SVID 中的所有断言实际上都是由签名授权机构所做的断言。

此授权机构的影响和断言所做的范围是自然有限的。一个信任域的授权机构的权限不应该对其他信任域中的实体做出断言（即其断言的范围仅限于其控制下的实体）。同样，在消费 SVID 数据时，消费者应该将其中包含的所有断言视为受到 SVID 所在信任域的限制。

例如：如果信任域 A 和 B 都使用名为“role”的属性，那么信任域 A 中具有“admin”角色的实体可以使用该角色做出自己信任域中 SVID 的断言，但信任域 B 中的实体不能使用与 A 中相同的断言对其 SVID 进行断言。

在这方面，SPIFFE 设计意图是将这些信任域之间的安全隔离形式化并保证在接受 SPIFFE SVID 的所有系统中得到正确执行。

#### 解释
通过签名 SVID 断言，签名授权机构明确其对所签名断言的含义的解释。此解释的范围由信任域的信任基础确定。

此外，消费者和其他参与者也可以对断言的含义进行自己的解释。例如，可能存在一个交叉信任域的场景，其中包含了不同信任域中的实体。这些实体可能会在实体之间以不同方式解释相同的断言。

操作者和开发人员在评估任何给定 SVID 断言的相对安全性时应该非常小心，特别是如果要验证的 SVID 属于不同的信任域。尽管通常情况下这种情况不会出现问题，但它也可能会导致复杂的安全问题，甚至不可知的问题。

#### 真实性
就像所有数字证书和断言一样，SVID 的真实性取决于其颁发方的安全性。签名授权机构的私钥的保护是 SVID 真实性的主要保障。如果授权机构的私钥暴露或泄漏，那么可以生成无效 SVID，并可能会导致错误的授权。

授权机构的私钥的安全性是信任域操作员的责任，他们应该采取必要的措施来确保私钥的安全，包括使用强大的密码学方法（如硬件安全模块）来保护私钥。此外，应定期更换私钥以降低突破的风险。

授权机构的私钥的安全性也是操作员选择是否使用外部信任域授权机构的一个重要考虑因素。如果使用外部授权机构的私钥进行签名，那么授权机构的私钥的安全性不再完全由信任域操作员控制，而是由外部授权机构的授权机构控制。这可能会引入一些风险和不确定性，特别是如果外部授权机构是第三方服务或实体。

---

请注意，上述文档是一个假设的 SPIFFE（面向所有人的安全生产身份框架）标准的核心规范的示例草稿。实际的 SPIFFE 规范可能会包含更多细节和具体规定，同时也可能会参考其他相关规范。在实际使用中，请始终参考最新的 SPIFFE 规范文档以确保遵守正确的标准和规定。