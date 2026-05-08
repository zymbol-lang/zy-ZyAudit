# ZyAudit · 字审

[English](README.md) · [中文](README_ZH.md) · [Español](README_ES.md)

> 用 Zymbol 编写的 Zymbol 代码审计与文档生成工具。  
> 内部以中文标识符构建，通过本地 Ollama 自动生成多语言文档。

---

## 项目简介

**ZyAudit**（字审）是一款命令行工具，分析 `.zy` 源文件的代码质量指标，并通过本地 Ollama 大语言模型自动生成多语言文档注释。整个核心逻辑以中文标识符编写，展示汉字在符号式编程语言中的紧凑表达能力。

本项目同时承担两个目标：

**目标一：验证中文开发的可行性**  
使用中文标识符构建完整的实用程序，验证 Zymbol 将 Unicode 作为一等公民的能力。在 Zymbol 中，语法已全为符号（`?` = if、`@` = 循环、`>>` = 输出），中文标识符是自然的延伸——`计量` 比 `calcular_metricas` 更紧凑，语义同样清晰。

**目标二：系统性记录 Zymbol 的边界**  
构建过程中遇到的所有障碍均按四类记录于 [`HALLAZGOS_ES.md`](HALLAZGOS_ES.md)：

| 类别 | 说明 |
|------|------|
| **BUG** | 现有功能在特定场景下行为不正确 |
| **GAP** | 语言缺少完成项目所需的构造或能力 |
| **ERROR** | 编译器或运行时产生的未记录错误 |
| **IDEA** | 受构建经验启发的语言改进提案 |

**无外部依赖** — 缺失的能力一律通过 BashExec 临时解决，并标记为 GAP。

---

## 安装与依赖

- **Zymbol** 已安装并在 PATH 中（`zymbol run`、`zymbol check`）
- **Ollama** 在本机运行（`http://localhost:11434`），并已安装至少一个模型
- **jq** 在 PATH 中（用于读取 `i18n.json` 和处理 JSON 响应）
- **curl** 在 PATH 中（与 Ollama 通信）

```bash
# 验证依赖
zymbol --version
ollama list
which jq curl
```

---

## 使用方法

```
zymbol run 主程.zy <文件.zy> [--语言 ZH|ES|EN] [--模型 名称]
```

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `<文件.zy>` | 待审计的 Zymbol 文件 | — （必填） |
| `--语言 ZH\|ES\|EN` | 报告输出语言 | `ZH` |
| `--模型 名称` | 使用的 Ollama 模型 | `qwen2.5-coder` |

### 示例

```bash
# 中文报告（默认）
zymbol run 主程.zy 源文件/计算器.zy

# 西班牙文报告
zymbol run 主程.zy 源文件/计算器.zy --语言 ES

# 英文报告，指定模型
zymbol run 主程.zy 源文件/计算器.zy --语言 EN --模型 deepseek-coder
```

---

## 输出示例

```
─────────────────────────────────────────
  字审 ZyAudit · 审计报告
─────────────────────────────────────────
  文件          : 源文件/计算器.zy
  行数          : 52
  代码行        : 38
  函数          : 8
  嵌套          : 3 层（最大）
  圈复杂度      : 5
  未使用        : 2  →  临时, _内部辅助
  覆盖率        : 75%
─────────────────────────────────────────
  [qwen2.5-coder] 生成中…  加法
  [qwen2.5-coder] 生成中…  减法
  ...
─────────────────────────────────────────
  ✓  docs/计算器_ZH.md  已写入
─────────────────────────────────────────
```

生成的 `docs/计算器_ZH.md` 包含指标表格以及每个函数的 Ollama 生成文档。

---

## 模块架构

所有模块位于 `字审/` 目录（项目名称作为命名空间）。每个文件有两字前缀标识其导出。

| 文件 | 前缀 | 职责 |
|------|------|------|
| `字审/解析.zy` | `解` | 解析 `.zy` 文件结构：函数、参数、行号 |
| `字审/计量.zy` | `量` | 计算代码质量指标 |
| `字审/构提.zy` | `提` | 构建中文 Ollama 提示词 |
| `字审/召模.zy` | `模` | Ollama 客户端：安装检查、连通性、模型存在、发送请求 |
| `字审/析答.zy` | `答` | 解析 Ollama JSON 响应，提取文档字段 |
| `字审/译文.zy` | `译` | 多语言格式化：ZH 直出，ES/EN 经 Ollama 翻译 |
| `字审/报告.zy` | `报` | 终端输出 + 写入 `docs/*.md` 文件 |
| `字审/国际化.zy` | `国` | 从 `i18n.json` 动态读取界面标签 |
| `主程.zy` | — | 入口：解析 CLI 参数，协调各模块 |

---

## 执行流程

```
<文件.zy>
    ↓  解  解析文件结构
函数列表 · 参数 · 行号
    ↓  量  指标计算
行数 · 嵌套深度 · 圈复杂度 · 未使用符号
    ↓  报  终端打印头部 + 指标
    ↓  模  三级检查
         模_已装() → 模_检查() → 模_存在()
    ↓  提  按函数构建中文提示词
    ↓  模  模_发送() → Ollama /api/generate
    ↓  答  解析响应，提取功能/参数/返回字段
    ↓  译  若 ES 或 EN，翻译后格式化
    ↓  报  写入 docs/<名称>_<语言>.md
```

---

## 国际化

界面标签（标题、指标名称、进度消息）通过 `字审/国际化.zy` 从 `i18n.json` 动态读取，使用 `jq` 查询。新增语言只需在 JSON 中添加一个块——无需修改代码：

```json
{
  "ZH": { "头标题": "字审 ZyAudit · 审计报告", ... },
  "ES": { "头标题": "字审 ZyAudit · Informe de Auditoría", ... },
  "EN": { "头标题": "ZyAudit · Audit Report", ... }
}
```

若请求的语言不存在，自动回退到 `ZH`。

---

## Ollama 集成

生成文档前，`主程.zy` 依次执行三级检查：

1. **`模_已装()`** — `ollama` 二进制文件在 PATH 中
2. **`模_检查()`** — 服务在 `http://localhost:11434` 响应
3. **`模_存在(模型)`** — 指定模型已安装

任意检查失败时，报告正常生成但文档字段替换为占位符 `—`，并在终端打印原因。**无需 API 密钥，代码不离开本机。**

默认主机为 `http://localhost:11434`，存储于 `字审/召模.zy` 的模块级变量中。若需连接其他主机，在检查前调用 `模::模_设主机("http://其他主机:11434")`。

### 推荐模型

| 模型 | 优势 |
|------|------|
| `qwen2.5-coder` | 中文代码理解最强——首选 |
| `deepseek-coder` | 代码推理能力强 |
| `llama3.1` | 通用多语言 |

---

## 文件结构

```
ZyAudit/
├── 主程.zy                  # 入口
├── i18n.json               # 界面标签 ZH / ES / EN
├── 字审/                    # 项目模块
│   ├── 解析.zy
│   ├── 计量.zy
│   ├── 构提.zy
│   ├── 召模.zy
│   ├── 析答.zy
│   ├── 译文.zy
│   ├── 报告.zy
│   └── 国际化.zy
├── 测试/                    # 各模块独立测试
│   ├── test_解析.zy
│   ├── test_计量.zy
│   ├── test_构提.zy
│   ├── test_召模.zy
│   ├── test_析答.zy
│   ├── test_译文.zy
│   ├── test_报告.zy
│   └── test_solo_ES.zy     # 验证 i18n.json 查询
├── 源文件/                  # 待审计的 .zy 示例文件
│   └── 计算器.zy
├── HALLAZGOS_ES.md         # BUG · GAP · ERROR · IDEA 发现记录（西班牙文）
├── README.md               # 英文文档
├── README_ES.md            # 西班牙文文档
└── README_ZH.md            # 本文件（中文）
```

---

## 测试

```bash
zymbol run 测试/test_解析.zy
zymbol run 测试/test_计量.zy
zymbol run 测试/test_报告.zy
zymbol run 测试/test_召模.zy
zymbol run 测试/test_析答.zy
zymbol run 测试/test_译文.zy
zymbol run 测试/test_solo_ES.zy   # 验证 i18n.json 查询系统
```

---

## v0.0.5 · ZyAudit 验证的语言修复

ZyAudit 是发现 Zymbol 语言问题的实战测试平台：在构建过程中共记录 6 个问题——3 个 BUG 和 3 个 GAP。所有问题均在 Zymbol **v0.0.5** 中解决，ZyAudit 源代码同步更新，移除了全部临时方案，直接使用已修复的语言特性。

| ID | 问题描述 | v0.0.5 修复方式 |
|----|----------|-----------------|
| BUG-001 | 模块级可变变量在再导出层中不可见 | `FunctionDef` 携带 `origin_module_path`，调用时恢复原模块上下文 |
| BUG-002 | `>< identifier` 不在语义分析 scope 中注册变量 | 在 `type_check.rs` 中新增 `Statement::CliArgsCapture` |
| BUG-003 | LSP 对 Unicode 目录名进行 URL 编码 → 模块未找到 | 在 `workspace.rs` 中添加 `percent_decode` |
| GAP-001 | slice 边界 `$[p-1..p+1]` 不支持算术表达式 | 在 `collection_ops.rs` 中新增 `parse_slice_bound()` |
| GAP-002 | `$++` 不接受括号表达式作为项 | 在 `string_ops.rs` 的 `can_start` 中添加 `TokenKind::LParen` |
| GAP-003 | `@ var:array` 循环产生 `ambiguous lifetime` 警告 | 循环变量使用 `_` 前缀即可抑制警告 |

**IDEA-001**（BashExec 的原始字符串）经评估后被放弃——修改 `{var}` 插值语法属于破坏性变更。完整说明见 [`HALLAZGOS_ES.md`](HALLAZGOS_ES.md)。

**端到端确认：** `zymbol run 主程.zy 源文件/计算器.zy --语言 ES --模型 codegemma:latest` 成功完成——9 个函数全部生成文档，`docs/计算器_ES.md` 已写入——证明所有修复在生产场景中均正确运行。

---

## 许可证

AGPL-3.0-or-later — 与 Zymbol 语言保持一致。

---

> 西班牙文文档：[README_ES.md](README_ES.md)  
> English documentation: [README.md](README.md)
