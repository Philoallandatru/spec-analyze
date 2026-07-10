# 链路训练过程 (Link Training Process)

## 概述

链路训练（Link Training）是 PCIe 物理层初始化的核心过程，通过链路训练状态机（Link Training and Status State Machine, LTSSM）协调两个设备之间建立可靠的高速串行连接。训练过程包括检测链路、协商参数、配置 Lane，最终使链路进入正常工作状态。

## LTSSM 状态机

### 状态概览

```
LTSSM 主要状态：
