# 概念索引

本索引按功能分类组织了 NVMe 规范的核心概念，帮助你快速定位需要了解的内容。

## 系统模型

理解 NVMe 系统的整体架构和组成部分：

- [规范族与适用范围](concepts/specification-family-and-scope.md) - NVMe 规范的组织结构
- [NVM 子系统](concepts/nvm-subsystem.md) - 系统边界与组成
- [控制器](concepts/controller.md) - 静态/动态模型；I/O、管理、发现三种类型
- [控制器虚拟化资源](concepts/controller-virtualization-resources.md) - 主/从控制器的虚拟队列和中断资源池
- [命名空间](concepts/namespace.md) - 存储抽象的基本单元
- [命名空间标识符](concepts/namespace-identifiers.md) - 有效性、分配、控制器挂载、广播、身份
- [命名空间访问模型](concepts/namespace-access-models.md) - 私有/共享/分散命名空间、多路径 I/O、非对称访问
- [命名空间拓扑与变更日志](concepts/namespace-topology-and-change-logs.md) - 分散参与者、可达性组/关联、变更追踪
- [命名空间管理生命周期](concepts/namespace-management-lifecycle.md) - 创建/删除分配、持久化控制器挂载
- [命名空间写保护](concepts/namespace-write-protection.md) - 保护状态、转换限制、缓存提交屏障
- [传输模型](concepts/transport-models.md) - 端口、网络接口、胶囊、SGL/数据放置
- [关联与命令生命周期](concepts/association-and-command-lifecycle.md) - 连接建立与命令执行
- [Fabrics 发现与认证](concepts/fabrics-discovery-and-authentication.md) - 过滤器协商、一致性快照、推荐、安全门控
- [Fabric 分区数据传输](concepts/fabric-zoning-data-transfer.md) - 查询、密钥/事务上下文、分片收发、错误共享

## 通信机制

命令和数据如何在主机与控制器间传输：

- [数据表示](concepts/data-representation.md) - 数据编码和字节序
- [传输模型](concepts/transport-models.md) - 基于内存和基于消息的传输
- [队列对](concepts/queue-pair.md) - 内存和消息模式的创建、流量控制、容量、删除
- [命令集](concepts/command-sets.md) - 队列路由与命名空间生命周期绑定
- [命令排序与仲裁](concepts/command-ordering-and-arbitration.md) - 融合操作、候选选择、优先级、未完成命令
- [命令生命周期](concepts/association-and-command-lifecycle.md) - 从提交到完成的全过程
- [公共命令格式](concepts/common-command-format.md) - 64 字节提交队列条目、CDW0、NSID、元数据和数据指针
- [数据指针布局](concepts/data-pointer-layouts.md) - PRP 条目/列表和 SGL 段边界
- [完成队列条目与状态](concepts/completion-queue-entry-and-status.md) - 关联、消费、状态族、重试提示、阶段标签
- [特性值与作用域](concepts/feature-values-and-scope.md) - 默认/当前/保存值、重置恢复、作用域选择器
- [通用控制器特性](concepts/common-controller-features.md) - 仲裁、电源、温度阈值、滞后、易失性缓存
- [安全协议交换](concepts/security-protocol-exchange.md) - Security Send/Receive 信封、发现、NVMe RPMB 选择
- [身份、名称与列表格式](concepts/identity-name-and-list-formats.md) - EUI64/NGUID/UUID、NQN、有序列表、UTF-8 处理
- [主机元数据与管理地址](concepts/host-metadata-and-management-addresses.md) - 代理 URI、类型化的控制器/命名空间注解
- [主机内存缓冲区](concepts/host-memory-buffer.md) - 主机内存所有权、描述符列表、禁用屏障、电源状态限制
- [识别命令模型](concepts/identify-command-model.md) - CNS 选择的 4 KiB 结构、依赖门控的能力、运行时限制
- [电源状态描述符](concepts/power-state-descriptors.md) - 操作状态、延迟、相对性能、功耗、掉电时序
- 胶囊与胶囊内数据（参见[传输模型](concepts/transport-models.md)）

## 存储组织

底层存储资源如何组织和管理：

- [存储资源层级](concepts/storage-resource-hierarchy.md) - NVM 集与回收组的组织方式
- [NVM 集与持久组](concepts/nvm-sets-and-endurance-groups.md) - 容量和耐久性管理单元
- [NVM 容量模型](concepts/nvm-capacity-model.md) - 容量分配、介质单元、通道、组织权衡
- [灵活数据放置配置](concepts/flexible-data-placement-configurations.md) - 配置目录、放置标识符拆分、运行时使用、统计、事件
- [域与分区](concepts/domains-and-divisions.md) - 故障边界、标识符、ANA/预留恢复
- [导出与底层 NVM 资源](concepts/exported-and-underlying-resources.md) - 目录、创建/清除、命名空间关联、删除门控
- 域（参见[域与分区](concepts/domains-and-divisions.md)）
- 持久组
- NVM 集
- 回收组
- 回收单元
- 介质单元

## 控制平面

控制器配置、管理和监控：

- [控制器属性](concepts/controller-properties.md) - 访问约定、能力/配置握手、版本、中断屏蔽
- [控制器内存窗口](concepts/controller-memory-windows.md) - CMB 与 PMR 地址、能力、就绪状态、健康
- [引导分区](concepts/boot-partitions.md) - 引导镜像存储
- [控制器支持需求](concepts/controller-support-requirements.md) - 按控制器类型划分的命令、日志页、特性
- [管理命令模型](concepts/admin-command-model.md) - 操作码分类、传输划分、Format/Sanitize 门控
- [命令中止语义](concepts/command-abort-semantics.md) - 立即/延迟中止、效果屏障
- [公共 I/O 控制命令](concepts/common-io-control-commands.md) - 取消选择、刷新持久化、I/O 管理交换
- [异步事件报告](concepts/asynchronous-event-reporting.md) - AER 生命周期、屏蔽、日志清除、事件族
- [容量管理操作](concepts/capacity-management-operations.md) - 固定/可变配置、持久组与 NVM 集生命周期
- [控制器数据队列](concepts/controller-data-queues.md) - 主机内存队列生命周期、用户数据迁移队列
- [设备自检](concepts/device-self-test.md) - 作用域、启动/中止转换、日志交互
- [指令交换](concepts/directive-exchange.md) - 共享的 Directive Send/Receive 信封
- [日志页检索](concepts/log-page-retrieval.md) - LID 选择、偏移量、作用域、支持发现
- [预留通知日志](concepts/reservation-notification-log.md) - 破坏性读取队列、序号间隙、通知类型、命名空间
- [命名空间预留生命周期](concepts/namespace-reservation-lifecycle.md) - 注册、获取、抢占、释放、持久化、状态报告
- [清理操作状态](concepts/sanitize-operation-status.md) - 持久化进度、结果、时间估算、状态详情、擦除证据
- [清理操作生命周期](concepts/sanitize-operation-lifecycle.md) - 后台擦除启动、介质验证、选项门控、串行化
- [资源与管理清单日志](concepts/resource-and-management-inventory-logs.md) - 旋转介质生命周期数据、管理 URI
- [错误与健康日志](concepts/error-and-health-logs.md) - 最近错误、命令关联、SMART 警告与状态
- [命令效果与支持](concepts/command-effects-and-support.md) - 操作码支持、影响作用域、串行化、能力变更
- [命令与特性锁定](concepts/command-and-feature-lockdown.md) - 按标识符命名空间与提交路径划分的可锁定与禁止操作
- [控制器迁移](concepts/controller-migration.md) - 状态格式选择、快照一致性、挂起/恢复/恢复生命周期
- [迁移变更追踪](concepts/migration-change-tracking.md) - UDMQ 日志、破坏性主机内存变更范围报告
- [遥测捕获日志](concepts/telemetry-capture-logs.md) - 主机/控制器捕获、块区域、保留、可用性、生成
- [可预测延迟模式](concepts/predictable-latency-mode.md) - NVM 集窗口、预算、估算、事件聚合
- [持久事件日志](concepts/persistent-event-log.md) - 持久化历史、冻结报告上下文、生成验证、重置与硬件错误证据
- Fabrics 命令集
- I/O 命令集
- [特性值与作用域](concepts/feature-values-and-scope.md)
- [日志页检索](concepts/log-page-retrieval.md)
- [异步事件报告](concepts/asynchronous-event-reporting.md)
- [Fabrics 发现与认证](concepts/fabrics-discovery-and-authentication.md)
- [识别命令模型](concepts/identify-command-model.md)

## 生命周期与弹性

系统初始化、错误恢复和电源管理：

- [控制器启用、关机与重置](concepts/controller-enable-shutdown-reset.md) - 控制器/子系统关机、重置作用域
- [控制器初始化](concepts/controller-initialization.md) - 基于内存和 Fabrics 的引导序列
- [控制器就绪模式与超时](concepts/controller-ready-modes.md) - 独立于介质的就绪、截止时间、超时失败
- [保活机制](concepts/keep-alive.md) - 配置、激活、命令/流量看门狗模式、超时清理
- [固件更新生命周期](concepts/firmware-update-lifecycle.md) - 下载、提交、激活、重置、串行化
- [格式化 NVM 生命周期](concepts/format-nvm-lifecycle.md) - 格式化作用域、安全擦除、并发、拒绝
- 电源状态
- [清理操作状态](concepts/sanitize-operation-status.md)
- [错误与健康日志](concepts/error-and-health-logs.md)
