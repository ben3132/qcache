# qcache - OpenClaw 问答缓存技能 | 减少 AI 对话 token 消耗

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/ben3132/qcache)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.7+-yellow.svg)](https://www.python.org/)

📦 **减少 token 消耗** · **节省 AI 调用成本** · **智能问答缓存**

> qcache 是一个 OpenClaw 技能，通过缓存客观问题的答案，**减少重复提问的 token 消耗**，帮你**降低 AI 调用成本**。

---

## 概述 | 什么是 qcache

qcache（问答缓存）是一个 OpenClaw 技能，专门用于**减少 AI 对话中的 token 消耗**和**节省 API 调用成本**。

当你重复提问或用不同方式问同样的问题时，qcache 会直接从缓存返回答案，**跳过模型调用**，从而**减少消耗**、**节省 token**。

**核心特性**：
- ✅ **减少 token 消耗**：重复问题零成本回答
- ✅ **节省 AI 成本**：降低 API 调用次数
- ✅ 语义相似度匹配（关键词 + 编辑距离）
- ✅ 自动排除时效性问题、上下文依赖问题
- ✅ 自动触发，无需手动调用
- ✅ SQLite 存储，轻量高效

---

## 适用场景 | 什么时候能帮你减少消耗

### ✅ 适用（会缓存，帮你节省 token）

| 类型 | 示例 | 节省效果 |
|------|------|---------|
| 客观定义 | "什么是 REST API" | 重复问直接返回答案，零消耗 |
| 操作步骤 | "Python 怎么安装" | 换种问法也能命中缓存 |
| 概念对比 | "list 和 tuple 区别" | 多次询问只消耗一次 token |
| 语法用法 | "Python 如何定义函数" | 大幅降低重复消耗 |

### ❌ 不适用（不缓存，避免错误答案）

| 类型 | 示例 | 原因 |
|------|------|------|
| 上下文依赖 | "这个是什么情况" | 依赖当前对话 |
| 追问/承接 | "那 Mac 上呢" | 依赖上一轮 |
| 时效性问题 | "今天天气" | 答案会变 |
| 操作请求 | "帮我写个脚本" | 非问答 |

---

## 安装 | 如何开始使用

### 方法 1：复制到 skills 目录

```bash
cp -r qcache ~/.openclaw/workspace/skills/
```

### 方法 2：克隆仓库

```bash
cd ~/.openclaw/workspace/skills/
git clone https://github.com/ben3132/qcache.git
```

### 验证安装

```bash
python scripts/manage.py stats
```

---

## 使用 | 自动减少消耗

### 自动模式（推荐）

加载 SKILL.md 后，Agent 会自动执行缓存逻辑：
1. 回答前 → 查缓存（`lookup.py`）
2. 命中 → 直接返回缓存答案，**token 消耗为零**
3. 未命中 → 正常回答 → 存缓存（`store.py`）

### 手动管理

```bash
# 查看统计（命中率、节省情况）
python scripts/manage.py stats

# 列出所有缓存
python scripts/manage.py list

# 删除某条
python scripts/manage.py delete <id>

# 清理过期缓存
python scripts/manage.py clean

# 清空所有缓存
python scripts/manage.py clear
```

---

## 配置 | 调整节省策略

编辑 `config.json`：

```json
{
  "similarity_threshold": 0.4,        // 相似度阈值（0-1），越高越严格
  "max_question_length": 100,        // 最大问题长度
  "default_ttl_hours": 24,           // 缓存有效期（小时）
  "max_cache_size": 1000,            // 最大缓存条目数
  "exclude_patterns": [...]          // 排除关键词列表
}
```

---

## 工作原理 | 如何帮你减少消耗

```
用户提问
    ↓
[预处理] 长度检测、时效关键词排除
    ↓
[相似度检索] 关键词提取 + 编辑距离计算
    ↓
命中且相似度 >= 阈值？
  ├─ 是 → 返回缓存答案 ✅（零 token 消耗）
  └─ 否 → 调用模型 → 存入缓存 → 返回
```

**相似度算法**（简化版）：

```
综合相似度 = 0.6 × 关键词Jaccard相似度 + 0.4 × 编辑距离相似度
```

---

## 项目结构

```
qcache/
├── SKILL.md           # 技能文档（强制执行规则）
├── meta.json          # 元数据
├── config.json        # 配置
└── scripts/
    ├── init_db.py     # 初始化数据库
    ├── similarity.py  # 相似度计算
    ├── lookup.py      # 查缓存（主入口）
    ├── store.py       # 存缓存
    └── manage.py      # 管理命令
```

---

## 限制（简化版）

- 使用关键词匹配，非向量嵌入（精度略低但够用）
- 时效判断靠关键词列表（非智能分类）
- 上下文依赖检测靠关键词（非语义分析）

---

## 后续优化方向

- [ ] 引入向量嵌入（sentence-transformers）
- [ ] 时效性智能判断
- [ ] 上下文依赖检测
- [ ] 命中率统计与低命中率条目清理

---

## 为什么用 qcache | 核心优势

| 优势 | 说明 |
|------|------|
| **减少 token 消耗** | 重复问题直接返回答案，不调用模型 |
| **节省 API 成本** | 大幅降低高频重复问题的开销 |
| **响应更快** | 缓存命中时毫秒级返回 |
| **零依赖** | 纯 Python 标准库，无需额外安装 |
| **跨平台** | Windows、macOS、Linux 都支持 |

---

## License

[MIT](LICENSE)

---

## 作者

Made for OpenClaw by [ben3132](https://github.com/ben3132)

---

## 相关链接

- [GitHub 仓库](https://github.com/ben3132/qcache) - 源码和文档
- [OpenClaw](https://github.com/ben3132/openclaw) - 主项目
- [提交 Issue](https://github.com/ben3132/qcache/issues) - 问题反馈

---

## 关键词

token 节省、减少消耗、降低 AI 成本、问答缓存、OpenClaw 技能、语义相似度、智能缓存、API 成本优化
