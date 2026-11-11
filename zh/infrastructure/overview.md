---
slug: "infrastructure/overview"
title: "基础设施选项"
description: "选择工作负载运行的位置：MIOSA 管理的计算、通过 OpenComputers 的本地服务器或客户云区域。"
section: "Self-hosting"
order: 15000
---

在 MIOSA 管理的计算上运行工作，连接您已拥有的计算机，或在您自己的云帐户中使用受管理的工作人员容量。

MIOSA 提供组织、工作区、API 和编排层。
所选的基础设施决定了执行的地点以及硬件的操作者。

## 选择工作运行的位置

|您的需求 |基础设施路径|谁操作机器？ |
|---|---|---|
|无需拥有服务器即可开始 | MIOSA 管理的计算 |MIOSA |
|在办公室、实验室或数据中心使用服务器 |通过 OpenComputers 连接的本地服务器 |您的组织 |
|使用工作站、GPU 计算机或现有托管服务器 |OpenComputers |您或您的托管提供商|
|在您的 AWS 账户或 Google Cloud 项目中配置受管理的工作线程池 |自带云 (BYOC) |在您的组织、MIOSA 和云提供商之间共享 |
|使用这些位置中的多个位置 |具有明确工作负载分配的混合基础架构 |取决于每个选定的目标 |

- **MIOSA 管理的计算**：MIOSA 负责运行计算容量、主机生命周期和平台网络。
  当您想要创建资源而不维护自己的服务器时，请使用它。
- **本地服务器**：通过 OpenComputers 连接办公室、实验室或数据中心中受支持的服务器。
  您的组织保留对硬件、操作系统、电源、存储和本地网络的责任。

  [OpenComputers 概述 →](/docs/opencomputers)
- **OpenComputers**：通过经过身份验证的出站智能体会话连接客户控制的计算机。
  该计算机可以是本地服务器、工作站、GPU 主机或托管提供商处的现有服务器。

  [连接机器→](/docs/opencomputers/connect)
- **自带云**：在客户 AWS 账户或 Google Cloud 项目内配置受管理的 MIOSA 工作区域。
  这是一个具有特定于提供商的入职和验收检查的工作人员能力池。

  [BYOC架构→](/docs/infrastructure/byoc/overview)

## 比较能力

资源类型决定执行模型；光是位置就不行。
调度工作前请检查选择的主机或区域。

|资源 |执行和桌面|持久化与运营 |
|---|---|---|
|托管沙箱 |代码和智能体的独立 Linux 执行；使用计算机作为完整桌面。 |遵循[快照和持久化生命周期](/docs/develop/snapshots)； MIOSA 操作主机。 |
|托管计算机| Linux 桌面、浏览器自动化和命令执行；等待桌面就绪。 |遵循[计算机准备情况](/docs/computers/fast-start)和恢复指南； MIOSA 操作主机。 |
|OpenComputers |在您现有的机器上运行并具有其宣传的功能；直接执行使用智能体的操作系统权限。 |您维护操作系统、磁盘、备份、网络和智能体服务；桌面和 GPU 的使用取决于主机。 |
| BYOC 员工容量 |您的云帐户中接受的工作人员池；可用的资源类型取决于提供程序的部署。 |您的云帐户拥有基础设施；遵循提供商的接受和[共同责任](/docs/infrastructure/shared-responsibility)。 |

OpenComputer 中的 GPU 仅当其驱动程序、模型运行时和选定的工作流程支持时才可用。
它并不意味着托管 GPU 沙箱的可用性。
发布的应用程序需要一个部署目标；连接主机不会自动在其上配置 Docker Host。

## 选择前的可用性

- **托管沙箱：** 根据[可用性页面](/docs/availability)通常可用。
- **托管计算机：** 合作伙伴预览；桌面配置和准备状态仍在强化中。
- **OpenComputers：**在选择主机之前验证帐户访问权限和安装的智能体的功能；仅注册既不能证明准备就绪，也不能证明功能平等。
- **BYOC：** 激活由提供商金丝雀和主机接受控制；基础设施配套本身并不是生产能力。

[连接您的第一个 Linux 服务器](/docs/opencomputers/connect) 逐步完成安装、注册、完成命令、服务启动和断开连接。
如果现有主机出现故障，请使用 [OpenComputers 故障排除](/docs/opencomputers/troubleshooting)。

## 选项如何关联

```mermaid
flowchart TB
  Product["Your product or agent"] --> MIOSA["MIOSA control plane"]
  MIOSA --> Managed["MIOSA-managed compute"]
  MIOSA --> Connected["OpenComputers<br/>connected machines"]
  MIOSA --> BYOC["BYOC<br/>governed cloud worker pools"]
  Connected --> Prem["On-premises servers<br/>office, lab, or data center"]
  Connected --> Other["Workstations, GPU machines,<br/>or existing hosted servers"]
  BYOC --> AWS["Your AWS account"]
  BYOC --> GCP["Your Google Cloud project"]
```**本地**描述了硬件的位置和操作责任。
**OpenComputers** 是受支持计算机的连接和远程操作路径。
通过 OpenComputers 连接的本地服务器是 **OpenComputer**。
OpenComputers 还可以与您所在场所之外受支持的机器配合使用。

在这些文档中，**BYOC** 意味着 **自带云**。
将现有云虚拟机连接为 OpenComputer 不会使其成为受管控的 BYOC 工作池。

## 混合基础设施和布局

混合基础设施将两个或多个基础设施位置组合到一个产品中。
例如，智能体可以在显式选择的本地主机上运行命令，而另一个任务则使用 MIOSA 管理的沙箱。
根据每个目标的功能、数据访问权限和允许的执行位置选择每个目标。

不要假设脱机或完整的本地主机会导致自动云溢出。
后备目标需要明确的授权、兼容的功能、对所需文件和机密的访问以及任何未完成工作的恢复计划。
通用 API 并不能使跨目标的硬件、桌面支持或隔离相同。

## 可用性和责任

连接主机使用MIOSA控制平面；它不会在您的场所安装完整的 MIOSA 平台。
当命令、状态、输出和任何请求的工件通过配置的 MIOSA 服务传递时，工作可以在您的计算机上执行。
使用 [OpenComputers 安全模型](/docs/opencomputers/security) 来了解连接和数据边界。

> **注意：客户云激活受到限制**
>
> AWS 和 Google Cloud 基础设施包已存在，但生产激活仍需提供商接受和实时金丝雀证据。
> 在计划推出之前，请检查提供商指南以了解适用的成熟度边界。

- [OpenComputers](/docs/opencomputers)：连接拥有的服务器和计算机，并了解执行、连接和操作职责。

- [共同责任](/docs/infrastructure/shared-responsibility)：查看谁操作硬件、操作系统、网络和控制平面。

- [基础设施术语表](/docs/glossary#customer-infrastructure)：本地、OpenComputers、BYOC、主机和混合基础设施的定义。
