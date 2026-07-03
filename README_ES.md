# ZyAudit · 字审

> **Revisado para v0.0.7 — 2026-06-19**

[English](README.md) · [中文](README_ZH.md) · [Español](README_ES.md)

> Herramienta de auditoría y generación de documentación para Zymbol, escrita en Zymbol.  
> Núcleo interno con identificadores en chino mandarín. Salida multilingüe vía Ollama local.

> **Proyecto de validación de Zymbol v0.0.4** — construido para poner a prueba el lenguaje
> en ese milestone, validar los identificadores Unicode/CJK como ciudadanos de primera clase
> y documentar cada punto de fricción encontrado en el desarrollo real.

---

## ¿Qué es ZyAudit?

**ZyAudit** (字审, *zì shěn* — "auditoría de caracteres") es una herramienta de línea de comandos que analiza archivos `.zy` y genera documentación automática usando un modelo de lenguaje local (Ollama). Es el equivalente a `rustdoc` + analizador estático, pero construido completamente en Zymbol.

El proyecto tiene dos objetivos simultáneos:

**1. Demostrar desarrollo real en mandarín**  
Cada identificador interno — variables, funciones, módulos, parámetros — está nombrado en chino. El objetivo es validar que Zymbol soporta Unicode como ciudadano de primera clase y que los caracteres chinos son más compactos sin perder expresividad. Donde en español escribirías `calcular_metricas`, en chino basta `计量`.

**2. Documentar los límites del lenguaje**  
Cada obstáculo encontrado durante la construcción se registra en [`HALLAZGOS_ES.md`](HALLAZGOS_ES.md) clasificado como BUG, GAP, ERROR o IDEA, con workaround documentado y propuesta de solución al lenguaje.

---

## Instalación y requisitos

- **Zymbol v0.0.7+** instalado y en PATH — usa la librería estándar nativa
  `std/net`, `std/json`, `std/io` (compila/instala desde `interpreter/install-zymbol.sh`).
- **Un proveedor**, uno de:
  - **`ollama`** (local): Ollama en `http://localhost:11434` con al menos un modelo
  - **`gemini`** (nube): API key gratuita de <https://aistudio.google.com/apikey>,
    en la variable de entorno `GEMINI_API_KEY`
- **jq** — ya solo lo usa `字审/国际化.zy` para leer `i18n.json` (lookup por clave dinámica).
  El HTTP y el JSON de las llamadas al modelo ahora son nativos (`curl` ya no es necesario).

```bash
# Verificar
zymbol --version          # v0.0.7+
ollama list               # para el proveedor ollama
echo "$GEMINI_API_KEY"    # para el proveedor gemini
```

---

## Uso

```
zymbol run 主程.zy <archivo.zy> [--语言 ZH|ES|EN] [--提供者 ollama|gemini] [--模型 nombre]
```

| Argumento | Descripción | Por defecto |
|-----------|-------------|-------------|
| `<archivo.zy>` | Archivo Zymbol a auditar | — (requerido) |
| `--语言 ZH\|ES\|EN` | Idioma del reporte | `ZH` |
| `--提供者 ollama\|gemini` | Proveedor: Ollama local o Gemini en la nube | `ollama` |
| `--模型 nombre` | Modelo a usar | `qwen2.5-coder` (ollama) / `gemini-flash-latest` (gemini) |

El proveedor `gemini` lee su clave de la variable de entorno `GEMINI_API_KEY`.

### Ejemplos

```bash
# Ollama local (proveedor por defecto), reporte en español
zymbol run 主程.zy 源文件/计算器.zy --语言 ES --模型 gemma3:4b

# Gemini en la nube, reporte en inglés
GEMINI_API_KEY=tu_clave zymbol run 主程.zy 源文件/计算器.zy --语言 EN --提供者 gemini

# Auditoría en chino (por defecto)
zymbol run 主程.zy 源文件/计算器.zy
```

---

## Salida de ejemplo

```
─────────────────────────────────────────
  字审 ZyAudit · Informe de Auditoría
─────────────────────────────────────────
  Archivo       : 源文件/计算器.zy
  Líneas        : 52
  Cód. líneas   : 38
  Funciones     : 8
  Anidamiento   : 3 niveles (máx.)
  Complejidad   : 5
  Sin usar      : 2  →  临时, _内部辅助
  Cobertura     : 75%
─────────────────────────────────────────
  [qwen2.5-coder] Generando…  加法
  [qwen2.5-coder] Generando…  减法
  ...
─────────────────────────────────────────
  ✓  docs/计算器_ES.md  escrito
─────────────────────────────────────────
```

El archivo `docs/计算器_ES.md` contiene una tabla de métricas y la documentación de cada función generada y traducida por Ollama.

---

## Arquitectura de módulos

Todos los módulos viven en la carpeta `字审/` (el nombre del proyecto como namespace). Cada archivo tiene un prefijo de dos caracteres que identifica sus exportaciones.

| Archivo | Prefijo | Caracteres | Responsabilidad |
|---------|---------|------------|-----------------|
| `字审/解析.zy` | `解` | 解=descomponer, 析=analizar | Parsea la estructura del archivo `.zy` |
| `字审/计量.zy` | `量` | 计=calcular, 量=cantidad | Calcula métricas de calidad |
| `字审/构提.zy` | `提` | 构=construir, 提=proponer | Arma los prompts; genera la doc **directamente en el idioma destino** (una sola llamada) |
| `字审/召模.zy` | `模` | 召=convocar, 模=modelo | Cliente de modelo (Ollama/Gemini) sobre la capa `标准库`: instalación, conectividad, envío, reintento |
| `字审/析答.zy` | `答` | 析=analizar, 答=respuesta | Limpia la respuesta del modelo (quita cercas de código) |
| `字审/译文.zy` | `译` | 译=traducir, 文=texto | Formatea la salida con etiquetas por idioma |
| `字审/报告.zy` | `报` | 报=reportar, 告=notificar | Terminal + escritura de `.md` (doc por función o resumen de programa) |
| `字审/国际化.zy` | `国` | 国=país/idioma | Lee etiquetas desde `i18n.json` vía `jq` |
| `字审/标准库/` | `网络·编解·文件·词典` | — | **Capa i18n de la stdlib en mandarín**: adaptadores de `std/net·json·io` + glosario de claves |
| `主程.zy` | — | 主=principal, 程=programa | Punto de entrada, coordina todos los módulos |

---

## Flujo de ejecución

```
<archivo.zy>
    ↓  解  parseo de estructura
Lista de funciones · parámetros · número de línea
    ↓  量  cálculo de métricas
Líneas · profundidad · complejidad · símbolos sin usar
    ↓  报  imprime cabecera + métricas en terminal
    ↓  模  verifica disponibilidad del proveedor
         ollama: 模_已装() → 模_检查() → 模_存在()   ·   gemini: clave presente
    ↓                ┌─ con funciones ──────────────────────────────┐
    ↓  提/模/答/报   │ por función: prompt en idioma destino → 模_发送 │
    ↓                │ → limpia respuesta → escribe docs por función  │
    ↓                └───────────────────────────────────────────────┘
    ↓                ┌─ sin funciones (entry point) ────────────────┐
    ↓  提/模/报      │ resumen de programa en idioma destino         │
    ↓                └───────────────────────────────────────────────┘
    ↓  报  escribe docs/<nombre>_<LANG>.md
```

> **Generación directa (v0.0.7):** la doc se genera **directamente en el idioma destino**
> en una sola llamada; ya no hay paso de traducción posterior. Esto eliminó la fragilidad
> de "generar en chino → traducir" y reduce a la mitad las llamadas al modelo.

---

## Internacionalización

Las etiquetas de la interfaz (cabeceras, nombres de métricas, mensajes de progreso) se leen dinámicamente desde `i18n.json` en la raíz del proyecto a través del módulo `字审/国际化.zy`. Para agregar un nuevo idioma basta añadir un bloque al JSON — sin tocar código.

```json
{
  "ZH": { "头标题": "字审 ZyAudit · 审计报告", ... },
  "ES": { "头标题": "字审 ZyAudit · Informe de Auditoría", ... },
  "EN": { "头标题": "ZyAudit · Audit Report", ... }
}
```

Si el idioma solicitado no existe en el JSON, la cadena de fallback cae automáticamente a `ZH`.

---

## Capa i18n de la stdlib en mandarín (`字审/标准库/`)

ZyAudit consume la librería estándar **sin que ningún nombre en inglés se filtre al código
mandarín**, aplicando el patrón de tres capas de i18n del lenguaje:

| Archivo | Reexporta | Lee como |
|---------|-----------|----------|
| `标准库/网络.zy` | `std/net` | `网络::获取` (get) · `网络::发送数据` (post_json) |
| `标准库/编解.zy` | `std/json` | `编解::解码` (decode) · `编解::编码` (encode) |
| `标准库/文件.zy` | `std/io` | `文件::写入` (write) · `文件::建目录` (mkdir) |

**i18n de datos.** Las **claves** del JSON de las APIs externas (Ollama/Gemini: `candidates`,
`models`, `response`…) seguirían en inglés. `编解::解码` aplica de forma transparente un
**glosario único** (función privada `词典()` en `编解.zy`) que renombra esas claves a mandarín
de forma recursiva, apoyándose en la función nativa `std/json::decode_map` (v0.0.7). Así la
lógica lee `数据.候选[1].内容.片段[1].文本` en vez de `数据.candidates[1].content.parts[1].text`.
El glosario se define **una sola vez** y lo comparten todos los consumidores.

---

## Documentación a nivel de programa

Los archivos de entrada (orquestadores como `serpiente.zy`) **no tienen funciones** que
documentar. Para ellos ZyAudit genera un **resumen del programa** (sección "Resumen del
programa" en el `.md`): un párrafo en el idioma destino con el propósito, el flujo principal y
los módulos de los que depende, generado a partir del código fuente.

---

## Integración con Ollama

ZyAudit verifica tres condiciones antes de intentar generar documentación:

1. **`模_已装()`** — el binario `ollama` está en PATH
2. **`模_检查()`** — el servicio responde en `http://localhost:11434`
3. **`模_存在(模型)`** — el modelo solicitado está instalado

Si alguna condición falla, el reporte se genera igualmente con placeholders `—` en lugar de documentación, y se imprime el motivo en terminal.

El host por defecto es `http://localhost:11434`, almacenado como variable de módulo en `字审/召模.zy`. Para apuntar a otro host, se usa `模::模_设主机("http://otro-host:11434")` antes de las verificaciones.

### Modelos recomendados

| Modelo | Ventaja |
|--------|---------|
| `qwen2.5-coder` | Mejor comprensión de código en chino — primera opción |
| `deepseek-coder` | Fuerte en razonamiento de código |
| `llama3.1` | Multilingüe general |

---

## Estructura de archivos

```
ZyAudit/
├── 主程.zy                  # Punto de entrada
├── i18n.json               # Etiquetas de interfaz ZH / ES / EN
├── 字审/                    # Módulos del proyecto
│   ├── 解析.zy
│   ├── 计量.zy
│   ├── 构提.zy
│   ├── 召模.zy
│   ├── 析答.zy
│   ├── 译文.zy
│   ├── 报告.zy
│   ├── 国际化.zy
│   └── 标准库/              # Capa i18n de la stdlib en mandarín
│       ├── 网络.zy          # adaptador de std/net
│       ├── 编解.zy          # adaptador de std/json + glosario 词典()
│       └── 文件.zy          # adaptador de std/io
├── 测试/                    # Tests por módulo
│   ├── test_解析.zy
│   ├── test_计量.zy
│   ├── test_构提.zy
│   ├── test_召模.zy
│   ├── test_析答.zy
│   ├── test_译文.zy
│   ├── test_报告.zy
│   └── test_solo_ES.zy     # Test del sistema i18n
├── 源文件/                  # Archivos .zy de ejemplo para auditar
│   └── 计算器.zy
├── HALLAZGOS_ES.md         # BUG · GAP · ERROR · IDEA documentados
├── README.md               # Documentación en inglés
├── README_ES.md            # Este archivo (español)
└── README_ZH.md            # Documentación en chino
```

---

## Tests

Cada módulo tiene su test independiente en `测试/`:

```bash
zymbol run 测试/test_解析.zy
zymbol run 测试/test_计量.zy
zymbol run 测试/test_报告.zy
zymbol run 测试/test_召模.zy
zymbol run 测试/test_析答.zy
zymbol run 测试/test_译文.zy
zymbol run 测试/test_solo_ES.zy   # verifica lookup de i18n.json
```

---

## ¿Por qué chino mandarín?

En Zymbol, toda la sintaxis ya es simbólica (`?` = if, `@` = loop, `>>` = print, `¶` = newline). Los identificadores en chino son el complemento natural: cada carácter comprime semántica que en español requeriría una palabra entera.

| Español | Chino | Caracteres |
|---------|-------|------------|
| `calcular_metricas` | `量` | 1 carácter |
| `analizar_respuesta` | `答_解析` | 3 caracteres |
| `generar_reporte` | `报_生成` | 3 caracteres |

`README.md` (inglés) y `README_ZH.md` (chino) también están disponibles para seguir la lógica del proyecto en tu idioma preferido.

---

## v0.0.5 · Hallazgos resueltos y validados en ZyAudit

ZyAudit fue el banco de pruebas real que descubrió 6 problemas del lenguaje Zymbol — 3 BUGs y 3 GAPs — durante su construcción. Todos fueron resueltos en Zymbol **v0.0.5**, y el código fuente de ZyAudit fue actualizado para eliminar cada workaround y usar las características corregidas directamente.

| ID | Descripción | Fix en v0.0.5 |
|----|-------------|---------------|
| BUG-001 | Variables mutables de módulo invisibles a través de capas de re-exportación | `FunctionDef` lleva `origin_module_path`; contexto del módulo original restaurado en llamada |
| BUG-002 | `>< identifier` no registraba la variable en el scope semántico | `Statement::CliArgsCapture` añadido en `type_check.rs` |
| BUG-003 | LSP URL-encodea directorios Unicode → module-not-found | `percent_decode` añadido en `workspace.rs` |
| GAP-001 | Expresiones aritméticas no permitidas como bounds de slice `$[p-1..p+1]` | Nuevo `parse_slice_bound()` en `collection_ops.rs` |
| GAP-002 | Expresiones parentizadas no aceptadas como ítems de `$++` | `TokenKind::LParen` añadido a `can_start` en `string_ops.rs` |
| GAP-003 | Loops `@ var:array` emitían warning `ambiguous lifetime` | Prefijo `_` en la variable de iteración suprime el warning |

**IDEA-001** (raw strings para BashExec) fue evaluada y descartada — cambiar la sintaxis de interpolación `{var}` sería un breaking change. Ver [`HALLAZGOS_ES.md`](HALLAZGOS_ES.md) para detalles completos y razonamiento.

**Confirmación end-to-end:** `zymbol run 主程.zy 源文件/计算器.zy --语言 ES --模型 codegemma:latest` completó exitosamente — 9 funciones documentadas, `docs/计算器_ES.md` escrito — confirmando que todos los fixes funcionan correctamente en uso productivo.

---

## v0.0.7 · Novedades

| Cambio | Detalle |
|--------|---------|
| **Capa i18n de stdlib** | `字审/标准库/` (网络·编解·文件) reexporta `std/net·json·io` con nombres en mandarín — cero inglés en el código |
| **i18n de datos** | `编解::解码` aplica un glosario único (`词典()`) que renombra las claves de las APIs externas vía `std/json::decode_map` (función nativa nueva) |
| **Generación directa** | La doc se genera directamente en el idioma destino (1 llamada); se eliminó el paso de traducción frágil |
| **Resumen de programa** | Los archivos sin funciones (entry points) reciben un resumen del programa en vez de doc por función |
| **Cliente robusto** | Fix del `\n` en la API key de Gemini (rompía el header HTTP) + reintento ante rate-limit por minuto |
| **Fuente acotada** | El prompt por función se limita a su propio código (no documenta funciones vecinas) |

> Nota de cuota: el free tier de Gemini permite ~20 peticiones/día por modelo. Para
> auditar varios módulos a la vez, considera Ollama local o espaciar las corridas.

---

## Licencia

AGPL-3.0-or-later — igual que el lenguaje Zymbol.
