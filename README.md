# 本项目，包括AI辅助教学系统与OpenClaw的多Agent两部分
## AI辅助教学系统：由vue+node.js以传统的软件工程方式进行构建。


# OpenClaw的duoagent部分 ，举例：私域知识库 Agent之创建指南
## 概述

本文档介绍如何在 OpenClaw 中创建一个可以使用私域知识库的 Agent。该 Agent 能够：
1. 管理私域知识库（添加、删除文档）
2. 智能搜索知识库内容
3. 基于知识库内容回答用户问题

---

## 目录

1. [架构说明](#架构说明)
2. [快速开始](#快速开始)
3. [详细配置](#详细配置)
4. [使用方法](#使用方法)
5. [扩展功能](#扩展功能)
6. [常见问题](#常见问题)

---

## 架构说明

### 系统架构

```
┌─────────────────────────────────────────────────────────┐
│                    OpenClaw Gateway                      │
│                          │                               │
│         ┌────────────────┼────────────────┐              │
│         │                │                │              │
│         ▼                ▼                ▼              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ 主 Agent    │  │ 知识库 Agent│  │ 其他 Agent  │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
│                          │                               │
│                          ▼                               │
│              ┌─────────────────────┐                    │
│              │   知识库 (Memory)   │                    │
│              │  - metadata.json   │                    │
│              │  - doc_*.txt       │                    │
│              └─────────────────────┘                    │
└─────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 说明 |
|------|------|
| Agent 配置 | 定义 Agent 的名称、模型、提示词 |
| 知识库 Skill | 提供知识库的 CRUD 操作 |
| Memory 存储 | 持久化知识库文档 |
| System Prompt | 指导 Agent 如何使用知识库 |

---

## 快速开始

### 方式一：使用已有代码（推荐）

我们已经为您准备好了完整的代码，位于：
```
/root/.openclaw/workspace/knowledge-agent/
```

**目录结构：**
```
knowledge-agent/
├── config/
│   └── agent.json              # Agent 配置
├── skills/
│   └── knowledge-base/
│       ├── skill.json         # Skill 元数据
│       ├── SKILL.md           # 使用文档
│       └── knowledge_base.py  # 核心代码
└── workspace/
    └── memory/
        └── knowledge/          # 知识库存储
```

### 方式二：从头创建

按照以下步骤创建您自己的知识库 Agent：

#### Step 1: 创建 Agent 目录

```bash
mkdir -p /root/.openclaw/workspace/my-knowledge-agent/{config,skills/knowledge-base,workspace/memory}
```

#### Step 2: 创建 Agent 配置

创建 `/root/.openclaw/workspace/my-knowledge-agent/config/agent.json`：

```json
{
  "agentId": "my-knowledge-assistant",
  "name": "我的知识库助手",
  "description": "基于私域知识库的AI助手",
  "model": "qianfan-ls/qianfan-code-latest",
  "systemPrompt": "你是一个专业的知识库助手...",
  "channel": "webchat-knowledge",
  "enabled": true
}
```

#### Step 3: 创建知识库 Skill

创建 `/root/.openclaw/workspace/my-knowledge-agent/skills/knowledge-base/knowledge_base.py`，复制上面的代码。

#### Step 4: 注册 Agent

需要修改 OpenClaw 配置来注册新 Agent，详见下一节。

---

## 详细配置

### Agent 配置说明

```json
{
  "agentId": "knowledge-assistant",
  "name": "知识库助手",
  "description": "基于私域知识库的AI助手",
  "model": "qianfan-ls/qianfan-code-latest",
  "thinking": false,
  "systemPrompt": "你是一个专业的知识库助手。你拥有访问私域知识库的权限...",
  "channel": "webchat-knowledge",
  "enabled": true
}
```

**配置项说明：**

| 配置项 | 必填 | 说明 |
|--------|------|------|
| agentId | ✅ | Agent 唯一标识（英文） |
| name | ✅ | Agent 显示名称（中文） |
| description | ✅ | Agent 描述 |
| model | ✅ | 使用的模型 |
| systemPrompt | ✅ | 系统提示词，定义 Agent 行为 |
| channel | ✅ | 消息通道标识 |
| enabled | ✅ | 是否启用 |

### Channel 配置

在 OpenClaw 配置文件中添加 channel：

```json
{
  "channels": {
    "webchat-knowledge": {
      "type": "webchat",
      "enabled": true,
      "capabilities": {
        "inlineButtons": "allowlist"
      }
    }
  }
}
```

---

## 使用方法

### 1. 添加知识库文档

通过命令行添加：
```bash
cd /root/.openclaw/workspace/knowledge-agent/skills/knowledge-base
python knowledge_base.py add "文档标题" "文档内容" [分类] [标签]
```

示例：
```bash
# 添加产品文档
python knowledge_base.py add "产品功能介绍" "我们的产品包含以下核心功能：1. 用户管理 2. 权限控制 3. 数据分析" "产品" "功能,核心"

# 添加 FAQ
python knowledge_base.py add "常见问题" "Q: 如何重置密码？ A: 在登录页面点击忘记密码" "FAQ" "密码,登录"
```

### 2. 搜索知识库

```bash
python knowledge_base.py search "关键词" [返回数量]
```

示例：
```bash
python knowledge_base.py search "用户管理"
python knowledge_base.py search "密码" 5
```

### 3. 列出所有文档

```bash
python knowledge_base.py list
python knowledge_base.py list 产品  # 按分类筛选
```

### 4. 删除文档

```bash
python knowledge_base.py delete doc_1
```

---

## 在 Agent 对话中使用

当用户提问时，Agent 会自动：

1. **接收用户问题**
2. **搜索知识库** - 调用 knowledge_base.py search
3. **分析匹配内容** - 提取最相关的文档片段
4. **生成回答** - 结合知识库内容回答

### 示例对话

**用户：** 请介绍一下你们的产品

**Agent：** 根据我们的知识库，我为您介绍产品的核心功能：

1. **用户管理** - 支持多用户协作，精细的权限控制
2. **权限控制** - 基于角色的访问控制 (RBAC)
3. **数据分析** - 提供数据可视化和报表功能

如需了解更多详情，请告诉我具体想了解哪部分。

---

## 扩展功能

### 1. 集成向量搜索（高级）

当前版本使用关键词匹配，后续可升级为向量搜索：

```python
# 使用 embedding 模型进行语义搜索
from sentence_transformers import SentenceTransformer

def semantic_search(query, top_k=5):
    model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')
    query_embedding = model.encode(query)
    
    # 与知识库文档 embedding 进行相似度计算
    # ...
```

### 2. 支持更多文件格式

可以扩展支持 PDF、Word、Markdown 等格式：

```python
import PyPDF2

def add_pdf_document(title, file_path, category="general"):
    with open(file_path, 'rb') as f:
        reader = PyPDF2.PdfReader(f)
        content = "\n".join([page.extract_text() for page in reader.pages])
    
    return add_document(title, content, category)
```

### 3. 自动同步外部知识源

可以添加定时任务同步外部知识库：

```python
# 定时从 Confluence/Wiki 同步
def sync_external_knowledge():
    # 从外部 API 获取文档
    # 调用 add_document 添加到知识库
    pass
```

---

## 常见问题

### Q1: 如何批量导入文档？

```bash
# 遍历目录批量导入
for file in *.txt; do
    title=$(basename "$file" .txt)
    content=$(cat "$file")
    python knowledge_base.py add "$title" "$content"
done
```

### Q2: 知识库存储在哪里？

- 路径：`/root/.openclaw/workspace/knowledge-agent/workspace/memory/knowledge/`
- metadata.json: 文档索引
- doc_*.txt: 文档内容

### Q3: 如何备份知识库？

```bash
# 打包备份
cd /root/.openclaw/workspace/knowledge-agent/workspace/memory
tar -czvf knowledge-backup.tar.gz knowledge/
```

### Q4: Agent 回答不准确怎么办？

1. 检查知识库中是否有相关文档
2. 调整 systemPrompt 让 Agent 更准确地使用知识库
3. 添加更多相关文档到知识库

---

## 相关文件

| 文件 | 路径 |
|------|------|
| Agent 配置 | `/root/.openclaw/workspace/knowledge-agent/config/agent.json` |
| 知识库代码 | `/root/.openclaw/workspace/knowledge-agent/skills/knowledge-base/knowledge_base.py` |
| 知识库存储 | `/root/.openclaw/workspace/knowledge-agent/workspace/memory/knowledge/` |

---

*文档版本: 1.0.0*  
*更新日期: 2026-03-20*
