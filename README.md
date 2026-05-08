# ZyAudit · 字审

[English](README.md) · [中文](README_ZH.md) · [Español](README_ES.md)

> A Zymbol code auditing and documentation generation tool, written in Zymbol.  
> Internal core uses Mandarin Chinese identifiers. Multilingual output via local Ollama.

---

## What is ZyAudit?

**ZyAudit** (字审, *zì shěn* — "character audit") is a command-line tool that analyzes `.zy` files and automatically generates documentation using a local language model (Ollama). It is the equivalent of `rustdoc` + static analyzer, but built entirely in Zymbol.

The project has two simultaneous goals:

**1. Demonstrate real development in Mandarin**  
Every internal identifier — variables, functions, modules, parameters — is named in Chinese. The goal is to validate that Zymbol supports Unicode as a first-class citizen, and that Chinese characters are more compact without losing expressiveness. Where you would write `calculate_metrics` in English, Chinese needs only `计量`.

**2. Document the current limits of the language**  
Every obstacle encountered during construction is recorded in [`HALLAZGOS_ES.md`](HALLAZGOS_ES.md) classified as BUG, GAP, ERROR, or IDEA, with a documented workaround and a proposed fix for the language.

---

## Installation and requirements

- **Zymbol** installed and in PATH (`zymbol run`, `zymbol check`)
- **Ollama** running locally (`http://localhost:11434`) with at least one model installed
- **jq** available in PATH (used to read `i18n.json` and process JSON responses)
- **curl** available in PATH (communication with Ollama)

```bash
# Verify requirements
zymbol --version
ollama list
which jq curl
```

---

## Usage

```
zymbol run 主程.zy <file.zy> [--语言 ZH|ES|EN] [--模型 model-name]
```

| Argument | Description | Default |
|----------|-------------|---------|
| `<file.zy>` | Zymbol file to audit | — (required) |
| `--语言 ZH\|ES\|EN` | Report output language | `ZH` |
| `--模型 name` | Ollama model to use | `qwen2.5-coder` |

### Examples

```bash
# English report with default model
zymbol run 主程.zy 源文件/计算器.zy --语言 EN

# Spanish report
zymbol run 主程.zy 源文件/计算器.zy --语言 ES

# Chinese report (default)
zymbol run 主程.zy 源文件/计算器.zy

# English report with a specific model
zymbol run 主程.zy 源文件/计算器.zy --语言 EN --模型 deepseek-coder
```

---

## Output example

```
─────────────────────────────────────────
  ZyAudit · Audit Report
─────────────────────────────────────────
  File          : 源文件/计算器.zy
  Lines         : 52
  Code lines    : 38
  Functions     : 8
  Nesting       : 3 levels (max)
  Complexity    : 5
  Unused        : 2  →  临时, _内部辅助
  Coverage      : 75%
─────────────────────────────────────────
  [qwen2.5-coder] Generating…  加法
  [qwen2.5-coder] Generating…  减法
  ...
─────────────────────────────────────────
  ✓  docs/计算器_EN.md  written
─────────────────────────────────────────
```

The generated `docs/计算器_EN.md` contains a metrics table and the Ollama-generated documentation for each function, translated to English.

---

## Module architecture

All modules live in the `字审/` directory (the project name as namespace). Each file has a two-character prefix that identifies its exports.

| File | Prefix | Characters | Responsibility |
|------|--------|------------|----------------|
| `字审/解析.zy` | `解` | 解=decompose, 析=analyze | Parses the `.zy` file structure: functions, parameters, line numbers |
| `字审/计量.zy` | `量` | 计=calculate, 量=quantity | Calculates code quality metrics |
| `字审/构提.zy` | `提` | 构=build, 提=propose | Builds Chinese prompts for Ollama |
| `字审/召模.zy` | `模` | 召=summon, 模=model | Ollama client: installation check, connectivity, model existence, request sending |
| `字审/析答.zy` | `答` | 析=analyze, 答=answer | Parses the Ollama JSON response, extracts doc fields |
| `字审/译文.zy` | `译` | 译=translate, 文=text | Multilingual formatting: ZH direct, ES/EN via Ollama translation |
| `字审/报告.zy` | `报` | 报=report, 告=notify | Terminal output + writes `docs/*.md` files |
| `字审/国际化.zy` | `国` | 国=country/language | Reads interface labels from `i18n.json` via `jq` |
| `主程.zy` | — | 主=main, 程=program | Entry point: parses CLI args, coordinates all modules |

---

## Execution flow

```
<file.zy>
    ↓  解  parse file structure
Function list · parameters · line numbers
    ↓  量  calculate metrics
Lines · nesting depth · complexity · unused symbols
    ↓  报  print header + metrics to terminal
    ↓  模  three-stage check
         模_已装() → 模_检查() → 模_存在()
    ↓  提  build Chinese prompt per function
    ↓  模  模_发送() → Ollama /api/generate
    ↓  答  parse response, extract function/params/return fields
    ↓  译  if ES or EN, translate then format
    ↓  报  write docs/<name>_<LANG>.md
```

---

## Internationalization

Interface labels (headers, metric names, progress messages) are read dynamically from `i18n.json` at the project root through the `字审/国际化.zy` module using `jq`. Adding a new language only requires adding a block to the JSON — no code changes needed:

```json
{
  "ZH": { "头标题": "字审 ZyAudit · 审计报告", ... },
  "ES": { "头标题": "字审 ZyAudit · Informe de Auditoría", ... },
  "EN": { "头标题": "ZyAudit · Audit Report", ... }
}
```

If the requested language does not exist in the JSON, the lookup automatically falls back to `ZH`.

---

## Ollama integration

Before generating documentation, `主程.zy` runs three checks in sequence:

1. **`模_已装()`** — the `ollama` binary is in PATH
2. **`模_检查()`** — the service responds at `http://localhost:11434`
3. **`模_存在(模型)`** — the requested model is installed

If any check fails, the report is generated anyway with `—` placeholders instead of documentation, and the reason is printed to the terminal. **No API key required. Code never leaves your machine.**

The default host is `http://localhost:11434`, stored as a module-level variable in `字审/召模.zy`. To point to a different host, call `模::模_设主机("http://other-host:11434")` before the checks.

### Recommended models

| Model | Advantage |
|-------|-----------|
| `qwen2.5-coder` | Best Chinese code understanding — first choice |
| `deepseek-coder` | Strong code reasoning |
| `llama3.1` | General multilingual |

---

## File structure

```
ZyAudit/
├── 主程.zy                  # Entry point
├── i18n.json               # Interface labels ZH / ES / EN
├── 字审/                    # Project modules
│   ├── 解析.zy
│   ├── 计量.zy
│   ├── 构提.zy
│   ├── 召模.zy
│   ├── 析答.zy
│   ├── 译文.zy
│   ├── 报告.zy
│   └── 国际化.zy
├── 测试/                    # Per-module tests
│   ├── test_解析.zy
│   ├── test_计量.zy
│   ├── test_构提.zy
│   ├── test_召模.zy
│   ├── test_析答.zy
│   ├── test_译文.zy
│   ├── test_报告.zy
│   └── test_solo_ES.zy     # Validates the i18n.json lookup system
├── 源文件/                  # Sample .zy files to audit
│   └── 计算器.zy
├── HALLAZGOS_ES.md         # BUG · GAP · ERROR · IDEA findings log (ES)
├── README.md               # English documentation (this file)
├── README_ES.md            # Spanish documentation
└── README_ZH.md            # Chinese documentation
```

---

## Tests

```bash
zymbol run 测试/test_解析.zy
zymbol run 测试/test_计量.zy
zymbol run 测试/test_报告.zy
zymbol run 测试/test_召模.zy
zymbol run 测试/test_析答.zy
zymbol run 测试/test_译文.zy
zymbol run 测试/test_solo_ES.zy   # validates i18n.json query system
```

---

## Why Mandarin Chinese identifiers?

In Zymbol, all syntax is already symbolic (`?` = if, `@` = loop, `>>` = print, `¶` = newline). Chinese identifiers are a natural extension: each character compresses semantics that in English would require a full word.

| English | Chinese | Characters |
|---------|---------|------------|
| `calculate_metrics` | `量` | 1 character |
| `parse_response` | `答_解析` | 3 characters |
| `generate_report` | `报_生成` | 3 characters |

`README_ES.md` and `README_ZH.md` are included so you can follow the project's logic regardless of which language you are most comfortable with.

---

## v0.0.5 · Language fixes validated in ZyAudit

ZyAudit was the real-world test bed that surfaced 6 Zymbol language issues — 3 BUGs and 3 GAPs — during development. All were resolved in Zymbol **v0.0.5**, and the ZyAudit source code was updated to remove every workaround and use the corrected language features directly.

| ID | Description | Fix in v0.0.5 |
|----|-------------|---------------|
| BUG-001 | Module-level mutable vars invisible through re-export layers | `FunctionDef` now carries `origin_module_path`; module context is restored on call |
| BUG-002 | `>< identifier` not registered in semantic scope | `Statement::CliArgsCapture` added in `type_check.rs` |
| BUG-003 | LSP URL-encoded Unicode directory names → module-not-found | `percent_decode` added in `workspace.rs` |
| GAP-001 | Arithmetic expressions not allowed as slice bounds `$[p-1..p+1]` | New `parse_slice_bound()` in `collection_ops.rs` |
| GAP-002 | Parenthesized expressions not accepted as `$++` items | `TokenKind::LParen` added to `can_start` in `string_ops.rs` |
| GAP-003 | `@ var:array` loops emitted `ambiguous lifetime` warning | `_` prefix in loop variable now suppresses the warning |

**IDEA-001** (raw strings for BashExec) was evaluated and discarded — changing the `{var}` interpolation syntax would be a breaking change. See [`HALLAZGOS_ES.md`](HALLAZGOS_ES.md) for full details and reasoning.

**End-to-end confirmation:** `zymbol run 主程.zy 源文件/计算器.zy --语言 ES --模型 codegemma:latest` completed successfully — all 9 functions documented, `docs/计算器_ES.md` written — confirming all fixes work correctly in production use.

---

## License

AGPL-3.0-or-later — same as the Zymbol language.
