---
name: prodoc-format
description: ProDoc 文档格式规范 —— 基于 Markdown 的文档组织约定，定义文件结构、Frontmatter 元数据、文档树构建规则、排序规则和流程图扩展语法
---

# ProDoc 文档格式规范

ProDoc 是一种基于 Markdown 的文档组织约定，定义了如何将一组 Markdown 文件组织成可供 ProDoc 渲染器识别和渲染的文档结构。

## 一、文件格式

所有 ProDoc 文档必须是 **`.md` 扩展名** 的 Markdown 文件，使用 UTF-8 编码。

## 二、文件系统映射

ProDoc 的文档树由**文件系统目录结构**直接映射而来。目录层级关系即文档的父子关系。

### 基本结构示例

```
document/
├── index.md              # 文档首页
├── guide/
│   ├── index.md          # "使用指南" 目录节点
│   ├── getting-started.md
│   └── configuration.md
├── api/
│   └── index.md
└── reference/
    └── changelog.md
```

- 每个 `.md` 文件对应一个文档节点
- 目录本身不直接产生节点，而是通过目录内的文件产生

## 三、元数据（Frontmatter）

每个文档文件可以在开头使用 YAML frontmatter 定义元数据：

```yaml
---
title: "文档标题"
order: 1
---
```

### 内置字段

| 字段 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `title` | `string` | 文档标题，用于导航树和页面显示 | 从正文第一个 H1 提取，否则为 `"Untitled"` |
| `order` | `number` | 排序权重，同级节点按此值升序排列 | `9999` |

### 自定义字段

Frontmatter 中除 `title` 和 `order` 以外的所有字段都会被解析并存入节点的 `meta` 对象中，可用于扩展。例如：

```yaml
---
title: "API 参考"
order: 3
author: "张三"
tags: ["api", "reference"]
---
```

### 值类型

Frontmatter 字段支持以下类型：

| 类型 | 示例 | 说明 |
|------|------|------|
| 整数 | `order: 1` | 纯数字，自动解析为 `number` |
| 浮点数 | `version: 1.5` | 带小数点的数字 |
| 布尔值 | `draft: true` | `true` 或 `false` |
| 字符串 | `title: "文档标题"` | 可带双引号或单引号 |
| 裸字符串 | `author: 张三` | 不带引号的字符串 |

## 四、文档树构建规则

### `index.md` 的作用

目录中的 `index.md` 文件充当**目录节点**，是该目录下其他文件的父节点。

例如，`guide/install.md` 的父节点优先匹配 `guide/index.md`（如果存在）。

### 父节点回退

如果目录中**没有 `index.md`**，则尝试将该目录的**同名 `.md` 文件**作为父节点：

- `guide/install.md` → 优先找 `guide/index.md`
- 若不存在，则尝试找 `guide.md`

### 根级文件

直接位于文档根目录下的文件（路径中不含 `/`）作为根节点的子节点。通常应包含一个 `index.md` 作为文档首页。

## 五、排序规则

同级节点按以下优先级排序：

1. **`order` 值升序** — 值越小越靠前
2. **`order` 相同时** — 按 `title` 字典序升序排列

建议为需要特定排序的文档显式设置 `order`，其他文档可省略（默认 `9999`）。

## 六、流程图扩展语法

ProDoc 支持一种特殊的 Markdown 代码块来定义流程图。

> ⚠️ 流程图渲染功能当前尚未实现。`prodoc-flow` 代码块在渲染时按普通代码块显示。

### 基本语法

使用 ````prodoc-flow` 代码块：

```prodoc-flow
graph LR
  A[开始] --> B{判断}
  B -->|是| C[处理1]
  B -->|否| D[处理2]
  C --> E[结束]
  D --> E
```

### 节点类型

| 语法 | 说明 |
|------|------|
| `[文本]` | 矩形节点 |
| `{文本}` | 菱形节点（判断） |
| `(文本)` | 圆形节点 |
| `[/文本/]` | 圆角矩形节点 |

### 文档链接

节点可以关联到文档，使用 `|` 分隔：

```prodoc-flow
graph LR
  A[首页|/index.md] --> B{有权限?}
  B -->|是| C[用户中心|/api/user.md]
  B -->|否| D[登录页]
```

## 七、完整示例

以下是一个符合 ProDoc 规范的完整文档目录：

```
document/
├── index.md
├── guide/
│   ├── index.md
│   ├── getting-started.md
│   └── flow-chart.md
└── api/
    └── index.md
```

`document/index.md`：

```markdown
---
title: "ProDoc 文档系统"
order: 1
---

# ProDoc 文档系统

欢迎使用 ProDoc。
```

`document/guide/index.md`：

```markdown
---
title: "使用指南"
order: 2
---

# 使用指南

本章节介绍如何使用 ProDoc。
```

`document/guide/getting-started.md`：

```markdown
---
title: "快速入门"
order: 1
---

# 快速入门

创建 ProDoc 项目...
```

## 八、内容规范建议

编写 ProDoc 文档内容时，建议遵循以下规范：

1. **标题层级**：每个文件以单一 H1（`#`）开头作为页面标题，正文从 H2（`##`）开始
2. **文件名**：使用小写字母 + 连字符命名（如 `getting-started.md`），保持 URL 友好
3. **图片资源**：将图片放在文档目录的 `images/` 或 `assets/` 子目录中
4. **相对链接**：使用相对路径链接到同文档集内的其他 `.md` 文件
5. **代码块**：始终为代码块指定语言标识，以便获得正确的语法高亮

## 九、验证方法

创建或修改 ProDoc 文档后，可通过以下方式验证：

```bash
# 启动文档查看服务器验证文档是否能正确渲染
npm run view -- ./your-doc-directory

# 启动编辑模式进行交互式编辑
npm run edit -- ./your-doc-directory
```

验证清单：
- [ ] 所有 `.md` 文件使用 UTF-8 编码
- [ ] Frontmatter 格式正确，无 YAML 语法错误
- [ ] `order` 字段为数字类型
- [ ] 目录中的 `index.md` 存在且包含有效的 frontmatter
- [ ] 导航树正确反映目录层级
- [ ] Markdown 内容渲染正常，无格式错误
