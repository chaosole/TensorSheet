
# TensorSheet: 增强AI长对话的结构化记忆系统

> [!IMPORTANT]
> **重要提示：您正在阅读的是TensorSheet Core开发文档。**
>
> 该项目不涉及应用层面的实现细节，专注于核心算法和数据结构的设计与优化。

## 项目介绍

TensorSheet是一个专为增强AI长对话场景设计的结构化记忆系统。它通过多维表格结构组织对话信息，使AI能够更有效地管理、检索和关联记忆内容。与传统的线性对话历史或向量存储方案不同，TensorSheet支持信息之间的结构化关系保存，同时提供直观的可视化界面，让用户能够参与记忆的编辑和管理过程。

### 核心创新点

- **结构化记忆范式转变**：从传统的向量或简单文本序列转变为结构化表格模型，AI能够像人类使用表格一样组织信息
- **互动式记忆管理**：将AI记忆从黑盒过程转变为用户可直接参与的协作过程，赋予用户对AI记忆的直接控制权
- **多维度数据关联**：突破了纯向量空间的相似度匹配局限，通过灵活的单元格定位机制和哈希映射存储建立丰富的关系网络
- **领域驱动的表格设计**：通过领域分层(全局、角色、对话)和类型系统(自由、动态、固定、静态)建立灵活的记忆组织框架
- **Object单元存储**：单元格可存储复杂对象结构，包括嵌套对象、数组、函数引用甚至可视化组件，实现"数据+逻辑+交互"的一体化存储

<br>

### 正在使用该数据库的测试或案例项目：

[ST记忆增强插件](https://github.com/muyoou/st-memory-enhancement)

<br>

## 开发准备与流程

1. **Fork仓库**：强烈建议先将[主仓库](https://github.com/tensorsheet/tensorsheet)Fork到您自己的GitHub账号下

2. **Clone开发分支**：
   ```bash
   git clone -b dev https://github.com/your-username/tensorsheet.git
   cd tensorsheet
   ```

3. **安装依赖**：
   ```bash
   npm install
   ```

4. **启动开发服务器**：
   ```bash
   npm run dev
   ```

5. **运行测试**：
   ```bash
   npm test
   ```

## 代码结构概览

```
tensorsheet/
├── src/                    # 源代码目录
│   ├── core/               # 核心功能模块
│   │   ├── sheet.js        # Sheet类定义
│   │   ├── sheetBase.js    # 基础表格类
│   │   └── sheetTemplate.js # 表格模板类
│   ├── editor/             # 编辑器UI相关模块
│   ├── renderer/           # 渲染相关模块  
│   ├── runtime/            # 运行时数据管理
│   ├── utils/              # 通用工具函数
│   └── services/           # 高层服务抽象
├── assets/                 # 静态资源
│   ├── templates/          # HTML模板
│   ├── css/                # 样式表
│   └── images/             # 图片资源
├── tests/                  # 测试目录
├── docs/                   # 文档
├── examples/               # 示例代码
├── package.json            # 项目配置
└── README.md               # 项目说明
```

## 核心模块详解

### `Sheet` 类 (核心数据结构)

`Sheet`类是TensorSheet的核心数据结构，负责表格数据的组织和管理。它使用哈希映射实现灵活的单元格定位，支持复杂关系的表示。

```javascript
class Sheet extends SheetBase {
  constructor(id, options = {}) {
    super(id, options);
    this.cells = {}; // 哈希映射存储单元格
    this.relations = {}; // 单元格间关系
    this.metadata = {}; // 表格元数据
  }
  
  getCell(row, col) {
    const key = this._generateCellKey(row, col);
    return this.cells[key] || null;
  }
  
  setCell(row, col, value, type = 'string') {
    const key = this._generateCellKey(row, col);
    this.cells[key] = { value, type, timestamp: Date.now() };
    this._triggerListeners('cellUpdate', { row, col, value, type });
    return this;
  }
  
  // 支持Object存储的特殊方法
  setObjectCell(row, col, object) {
    return this.setCell(row, col, object, 'object');
  }
  
  // 其他方法...
}
```

### 管理器模块 (`manager.js`)

管理器模块作为系统的中枢，组织各个功能模块并提供统一的访问接口。

```javascript
// 示例: manager.js核心结构
const Manager = {
  USER: {
    // 用户数据和设置管理
    settings: {},
    getSettings() { /* ... */ },
    saveSettings() { /* ... */ },
    getChatPiece() { /* ... */ }
  },
  
  BASE: {
    // 核心数据管理
    Sheet: Sheet,
    templates: {},
    contextSheets: {},
    loadUserAllTemplates() { /* ... */ },
    loadContextAllSheets() { /* ... */ }
  },
  
  EDITOR: {
    // UI编辑器控制
    Drag: {},
    PopupMenu: {},
    Popup: {},
    info() { /* ... */ },
    error() { /* ... */ }
  },
  
  DERIVED: {
    // 运行时数据
    runtimeData: {},
    Table: Table
  },
  
  SYSTEM: {
    // 系统工具
    getTemplate() { /* ... */ },
    htmlToDom() { /* ... */ },
    generateRandomString() { /* ... */ },
    calculateStringHash() { /* ... */ }
  }
};
```

### 数据流架构

TensorSheet采用了基于事件的数据流架构，确保各模块间的低耦合和高内聚：

1. **Sheet实例处理数据变更**：当数据变更时发出事件
2. **事件总线传递变更**：变更通过事件总线传递给订阅者
3. **渲染器响应变更**：渲染器订阅事件并更新UI
4. **用户交互触发操作**：UI操作经编辑器转换为Sheet操作

```javascript
// 示例: 事件驱动架构
class EventBus {
  constructor() {
    this.subscribers = {};
  }
  
  subscribe(event, callback) {
    if (!this.subscribers[event]) {
      this.subscribers[event] = [];
    }
    this.subscribers[event].push(callback);
    return () => this.unsubscribe(event, callback);
  }
  
  publish(event, data) {
    if (!this.subscribers[event]) return;
    this.subscribers[event].forEach(callback => callback(data));
  }
  
  unsubscribe(event, callback) {
    if (!this.subscribers[event]) return;
    this.subscribers[event] = this.subscribers[event].filter(cb => cb !== callback);
  }
}
```

## 核心技术实现

### Object单元存储机制

单元格对象存储是TensorSheet的关键创新之一，允许在单元格中存储任何JavaScript对象：

```javascript
// 示例: 存储复杂对象
sheet.setObjectCell('A1', 'B1', {
  data: [1, 2, 3],
  metadata: { source: 'user', timestamp: Date.now() },
  render: function() { return `<div>${this.data.join(',')}</div>`; },
  calculate: function(x) { return this.data.reduce((a, b) => a + b, x); }
});

// 获取并使用对象方法
const cell = sheet.getCell('A1', 'B1');
const sum = cell.value.calculate(10); // 16
const html = cell.value.render(); // "<div>1,2,3</div>"
```

### 哈希表映射存储

TensorSheet使用哈希映射代替传统二维数组，实现灵活的单元格寻址：

```javascript
// 内部实现示例
_generateCellKey(row, col) {
  return `${row}:${col}`;
}

setCell(row, col, value) {
  const key = this._generateCellKey(row, col);
  this.cells[key] = { value, timestamp: Date.now() };
}

getCell(row, col) {
  const key = this._generateCellKey(row, col);
  return this.cells[key];
}
```

这种设计允许：
- 稀疏表格高效存储（只存储有值的单元格）
- 非传统的寻址（例如字符串、负数、特殊标识符）
- 动态扩展表格范围而无需重新分配数组

### 分步API设计

系统采用分步API设计，将对话与附属流程分离：

```javascript
// 示例: 分步API
async function processUserInput(input) {
  // 1. 对话核心处理
  const dialogueResult = await processDialogue(input);
  
  // 2. 记忆提取（可并行）
  const memoryResult = await extractMemory(input);
  
  // 3. 表格更新（可延迟执行）
  scheduleTask(() => updateSheets(dialogueResult, memoryResult));
  
  // 4. 立即返回对话结果
  return dialogueResult;
}
```

## 贡献指南

### 代码规范

- 使用ESLint和Prettier保持代码一致性
- 遵循函数式编程原则，减少副作用
- 编写单元测试，保持测试覆盖率
- 使用JSDoc注释公共API

### 提交PR流程

1. 创建功能分支 `feature/your-feature-name`
2. 完成开发并通过测试
3. 提交PR到`dev`分支
4. 等待代码审查
5. 合并到`dev`分支进行集成测试
6. 稳定后合并到`main`分支发布

## 应用开发示例

### 基本使用示例

```javascript
// 创建一个新表格
const sheet = new Sheet('userPreferences');

// 存储简单数据
sheet.setCell('preferences', 'theme', 'dark');
sheet.setCell('preferences', 'fontSize', 16);

// 存储对象数据
sheet.setObjectCell('history', 'conversations', {
  list: ['Hello', 'How are you?'],
  addMessage(msg) {
    this.list.push(msg);
  }
});

// 使用存储的对象方法
const conversations = sheet.getCell('history', 'conversations').value;
conversations.addMessage('I am fine, thanks!');
```

### 事件监听示例

```javascript
// 监听单元格更新事件
sheet.on('cellUpdate', ({ row, col, value }) => {
  console.log(`Cell ${row}:${col} updated to ${value}`);
  // 更新UI或同步到其他系统
});

// 监听表格结构变化事件
sheet.on('structureChange', (changeInfo) => {
  console.log('Table structure changed:', changeInfo);
  // 重新渲染表格或更新相关组件
});
```

## 再次感谢您的参与和支持！

如果您在开发过程中遇到任何问题，或者有任何建议和想法，欢迎随时通过GitHub Issues或其他渠道与我们联系。
