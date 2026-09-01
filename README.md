# ZyAudit · 字审

> **Revised for v0.0.8 — 2026-08-31**

[English](README.md) · [中文](README_ZH.md) · [Español](README_ES.md)

> A Zymbol code auditing and documentation generation tool, written in Zymbol.  
> Internal core uses Mandarin Chinese identifiers. Multilingual output via local Ollama.

> **Validation project for Zymbol v0.0.4** — built to stress-test the language
> at that milestone, validate Unicode/CJK identifiers as first-class citizens,
> and document every friction point encountered in real development.

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

- **Zymbol v0.0.7+** installed and in PATH — needs the native `std/net`, `std/json`,
  `std/io` standard library (build/install from `interpreter/install-zymbol.sh`).
- **A provider**, one of:
  - **`ollama`** (local): Ollama running at `http://localhost:11434` with at least one model
  - **`gemini`** (cloud): a free API key from <https://aistudio.google.com/apikey>, set in
    the `GEMINI_API_KEY` environment variable
- **jq** — only still used by `字审/国际化.zy` to read `i18n.json` (dynamic key lookup).
  HTTP and JSON for the model calls are now native (`curl` is no longer required).

```bash
# Verify
zymbol --version          # v0.0.7+
ollama list               # for the ollama provider
echo "$GEMINI_API_KEY"    # for the gemini provider
```

---

## Usage

```
zymbol run 主程.zy <file.zy> [--语言 ZH|ES|EN] [--提供者 ollama|gemini] [--模型 name]
```

| Argument | Description | Default |
|----------|-------------|---------|
| `<file.zy>` | Zymbol file to audit | — (required) |
| `--语言 ZH\|ES\|EN` | Report output language | `ZH` |
| `--提供者 ollama\|gemini` | Model provider: local Ollama or cloud Gemini | `ollama` |
| `--模型 name` | Model to use | `qwen2.5-coder` (ollama) / `gemini-flash-latest` (gemini) |

The `gemini` provider reads its key from the `GEMINI_API_KEY` environment variable.

### Examples

```bash
# Local Ollama (default provider), Chinese report
zymbol run 主程.zy 源文件/计算器.zy --模型 gemma3:4b

# Local Ollama, Spanish report
zymbol run 主程.zy 源文件/计算器.zy --语言 ES --模型 gemma3:4b

# Cloud Gemini, English report
GEMINI_API_KEY=your_key zymbol run 主程.zy 源文件/计算器.zy --语言 EN --提供者 gemini
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
| `字审/构提.zy` | `提` | 构=build, 提=propose | Builds prompts; generates docs **directly in the target language** (single call) |
| `字审/召模.zy` | `模` | 召=summon, 模=model | Model client (Ollama/Gemini) over the `标准库` layer: install/connectivity checks, send, retry |
| `字审/析答.zy` | `答` | 析=analyze, 答=answer | Cleans the model response (strips code fences) |
| `字审/译文.zy` | `译` | 译=translate, 文=text | Formats output with per-language labels |
| `字审/报告.zy` | `报` | 报=report, 告=notify | Terminal output + writes `docs/*.md` (per-function docs or program overview) |
| `字审/国际化.zy` | `国` | 国=country/language | Reads interface labels from `i18n.json` via `jq` |
| `字审/标准库/` | `网络·编解·文件·词典` | — | **Mandarin i18n layer for the stdlib**: `std/net·json·io` adapters + key glossary |
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
    ↓  模  provider readiness
         ollama: 模_已装() → 模_检查() → 模_存在()   ·   gemini: key present
    ↓                ┌─ has functions ──────────────────────────────┐
    ↓  提/模/答/报   │ per function: prompt in target language → 模_发送 │
    ↓                │ → clean response → write per-function docs     │
    ↓                └───────────────────────────────────────────────┘
    ↓                ┌─ no functions (entry point) ─────────────────┐
    ↓  提/模/报      │ program overview in the target language        │
    ↓                └───────────────────────────────────────────────┘
    ↓  报  write docs/<name>_<LANG>.md
```

> **Direct generation (v0.0.7):** docs are produced **directly in the target language**
> in a single call; there is no separate translation pass anymore. This removed the brittle
> "generate-in-Chinese → translate" pipeline and halves the number of model calls.

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

## Mandarin i18n layer for the stdlib (`字审/标准库/`)

ZyAudit consumes the standard library **without leaking any English name into the Mandarin
code**, applying the language's three-layer i18n pattern:

| File | Re-exports | Reads as |
|------|------------|----------|
| `标准库/网络.zy` | `std/net` | `网络::获取` (get) · `网络::发送数据` (post_json) |
| `标准库/编解.zy` | `std/json` | `编解::解码` (decode) · `编解::编码` (encode) |
| `标准库/文件.zy` | `std/io` | `文件::写入` (write) · `文件::建目录` (mkdir) |

**Data-level i18n.** The **keys** of external API JSON (Ollama/Gemini: `candidates`, `models`,
`response`…) would stay in English. `编解::解码` transparently applies a **single glossary**
(private `词典()` in `编解.zy`) that recursively renames those keys to Mandarin, backed by the
native `std/json::decode_map` function (v0.0.7). The logic then reads
`数据.候选[1].内容.片段[1].文本` instead of `数据.candidates[1].content.parts[1].text`.
The glossary is defined **once** and shared by every consumer.

---

## Program-level documentation

Entry-point files (orchestrators like `serpiente.zy`) have **no functions** to document.
For them ZyAudit generates a **program overview** (a "Program overview" section in the `.md`):
a paragraph in the target language describing the program's purpose, main flow, and the
modules it depends on, derived from the source.

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
│   ├── 国际化.zy
│   └── 标准库/              # Mandarin i18n layer for the stdlib
│       ├── 网络.zy          # std/net adapter
│       ├── 编解.zy          # std/json adapter + 词典() glossary
│       └── 文件.zy          # std/io adapter
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

## v0.0.7 · What's new

| Change | Detail |
|--------|--------|
| **Stdlib i18n layer** | `字审/标准库/` (网络·编解·文件) re-exports `std/net·json·io` under Mandarin names — zero English in the code |
| **Data-level i18n** | `编解::解码` applies a single glossary (`词典()`) that renames external-API keys via `std/json::decode_map` (new native function) |
| **Direct generation** | Docs are generated directly in the target language (1 call); the brittle translation pass is gone |
| **Program overview** | Files with no functions (entry points) get a program overview instead of per-function docs |
| **Robust client** | Fixed the `\n` in the Gemini API key (broke the HTTP header) + per-minute rate-limit retry |
| **Bounded source** | Each per-function prompt is limited to its own code (no documenting neighbors) |

> Quota note: the Gemini free tier allows ~20 requests/day per model. To audit several
> modules at once, prefer local Ollama or space the runs out.

---

## License

AGPL-3.0-or-later — same as the Zymbol language.
