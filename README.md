
# TensorSheetCore: 增强AI长对话记忆的结构化表内核

> [!IMPORTANT]
> **重要提示：您正在阅读的是TensorSheet Core开发文档。**
>
> 该项目不涉及应用层面的实现细节，专注于核心算法和数据结构的设计与优化。

## 项目介绍

TensorSheet是一个专为增强AI长对话场景设计的结构化记忆系统。它通过多维表格结构组织对话信息，使AI能够更有效地管理、检索和关联记忆内容。与传统的线性对话历史或向量存储方案不同，TensorSheet支持信息之间的结构化关系保存，同时提供直观的可视化界面，让用户能够参与记忆的编辑和管理过程。

### 这种设计允许
- 稀疏表格高效存储（只存储有值的单元格）
- 高效链式寻址（例如行、列、单元或整表历史记录、语义关联、方向等）
- 动态扩展表格范围和方向

### 核心创新点

- **结构化记忆范式转变**：从传统的向量或简单文本序列转变为结构化表格模型，AI能够像人类使用表格一样组织信息
- **互动式记忆管理**：将AI记忆从黑盒过程转变为用户可直接参与的协作过程，赋予用户对AI记忆的直接控制权
- **多维度数据关联**：突破了纯向量空间的相似度匹配局限，通过灵活的单元格定位机制和哈希映射存储建立丰富的关系网络
- **领域驱动的表格设计**：通过领域分层(全局、角色、对话)和类型系统(自由、动态、固定、静态)建立灵活的记忆组织框架
- **对象与嵌套单元**：单元格可存储复杂对象结构，包括嵌套对象、数组、函数引用甚至可视化组件，实现"数据+逻辑+交互"的一体化存储

## 案例
正在使用该数据库的测试或案例项目：

[巧思 (Chaosole)]()

[SmartTable]()

[ST记忆增强插件](https://github.com/muyoou/st-memory-enhancement/tree/dev)


## 核心模块

### `Sheet` 类 (核心数据结构)

`Sheet`类是TensorSheet的核心数据结构，负责表格数据的组织和管理。它使用哈希映射实现灵活的单元格定位，支持复杂关系的表示。

### 数据流架构

TensorSheet采用了基于事件的数据流架构，确保各模块间的低耦合和高内聚：

1. **Sheet实例处理数据变更**：当数据变更时发出事件
2. **事件总线传递变更**：变更通过事件总线传递给订阅者
3. **渲染器响应变更**：渲染器订阅事件并更新UI
4. **用户交互触发操作**：UI操作经编辑器转换为Sheet操作

### 分步API设计

系统采用分步API设计，将对话与附属流程分离：

## 再次感谢您的参与和支持！

如果您在开发过程中遇到任何问题，或者有任何建议和想法，欢迎随时通过GitHub Issues或其他渠道与我们联系。
