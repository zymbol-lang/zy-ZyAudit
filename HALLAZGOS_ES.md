# HALLAZGOS · ZyAudit

> Registro vivo de BUGs, GAPs, ERRORs e IDEAs descubiertos durante la construcción de ZyAudit.  
> Cada entrada incluye el contexto exacto donde apareció, el workaround actual y la propuesta de solución.  
> Este documento alimenta directamente el roadmap de Zymbol v0.0.5 y versiones futuras.

---

## Cómo leer este documento

| Campo | Significado |
|-------|-------------|
| **ID** | Identificador único. Formato: `TIPO-NNN` |
| **Módulo** | Archivo `.zy` donde se encontró el problema |
| **Contexto** | Construcción exacta de Zymbol involucrada |
| **Descripción** | Qué falla, qué falta o qué se propone |
| **Workaround** | Solución temporal utilizada en ZyAudit |
| **Propuesta** | Cómo debería resolverse en el lenguaje |
| **Estado** | `abierto` · `workaround` · `propuesto` · `resuelto vX.X.X` |

---

## BUG — Comportamiento incorrecto en funcionalidad existente

Funcionalidades que Zymbol declara soportar pero que fallan en condiciones específicas encontradas durante la construcción en mandarín.

| ID | Módulo | Contexto | Estado |
|----|--------|----------|--------|
| [BUG-001](#bug-001--variables-mutables-de-módulo-invisibles-en-re-exports) | `召模.zy` | Variables mutables de módulo + capas de re-exportación | workaround |
| [BUG-002](#bug-002--captura-de-cli-args--no-declara-la-variable-en-el-scope-semántico) | `主程.zy` | `>< identifier` — analizador semántico no registra la variable capturada | workaround |
| [BUG-003](#bug-003--lsp-url-encodea-directorios-unicode-al-resolver-rutas-de-módulos) | `源码/*.zy`, `源码/国际化/*.zy` | LSP URL-encodea directorios con caracteres chinos → `module-not-found` | abierto |

---

### BUG-001 · Variables mutables de módulo invisibles en re-exports

- **Módulo:** `召模.zy`
- **Contexto:** Funciones que leen variables mutables de nivel de módulo, llamadas a través de una capa de re-exportación i18n (patrón 3 capas de `I18N.md`).
- **Descripción:** Una función que accede a una variable mutable de módulo (`主机`, `临目`) funciona correctamente cuando se llama directamente (`模::模_检查()`), pero lanza `undefined variable` cuando se llama a través de un módulo de re-exportación (`es::verificar()`). La capa de traducción no preserva el scope del módulo original para las funciones re-exportadas.
- **Caso mínimo:**
  ```
  // 召模.zy
  # .召模 {
      #> { 模_检查 }
      主机 = "http://localhost:11434"
      模_检查() { <~ 主机 }   // accede a variable de módulo
  }

  // i18n/召模_ES.zy
  # .召模_ES {
      <# ../召模 <= 模
      #> { 模::模_检查 <= verificar }
  }

  // consumidor
  <# ./i18n/召模_ES <= es
  es::verificar()   // → Runtime error: undefined variable: '主机'
  ```
- **Workaround:** Diseño sin estado — la configuración viaja como tuple parámetro. Las funciones no acceden a variables de módulo sino a sus parámetros:
  ```
  模_检查(配置) { <~ 配置.主机 $+ "/" ... }
  cfg = 模::模_默认配置()
  es::verificar(cfg)   // ✓ funciona
  ```
- **Propuesta:** El sistema de re-exportación debería preservar el scope del módulo de origen para las funciones re-exportadas, igual que lo hace para las llamadas directas.
- **Estado:** workaround

---

### BUG-002 · Captura de CLI args `><` no declara la variable en el scope semántico

- **Módulo:** `主程.zy`
- **Contexto:** Construcción `>< identifier` — captura de argumentos de línea de comandos en el tree-walker.
- **Descripción:** El analizador semántico no reconoce `>< 参数列` como una declaración de variable. Todos los usos subsecuentes de `参数列` — incluyendo `总参 = 参数列$#` y accesos dentro de bloques `? {}` — generan `error: undefined variable '参数列'`. El programa ejecuta correctamente en el tree-walker, pero el LSP y `zymbol check` reportan errores.
- **Caso mínimo:**
  ```
  >< 参数列
  总参 = 参数列$#   // → error: undefined variable '参数列'
  ```
- **Workaround:** Pre-declarar la variable con un valor vacío antes de `><`. El semantic analyzer reconoce la asignación inicial y permite los usos posteriores:
  ```
  参数列 = []   // pre-declarar — el runtime sobreescribe con los args reales
  >< 参数列
  总参 = 参数列$#   // ✓ sin error
  ```
- **Propuesta:** `>< identifier` debería registrar `identifier` como variable definida en el scope del analizador semántico, igual que cualquier asignación ordinaria.
- **Estado:** workaround

---

### BUG-003 · LSP URL-encodea directorios Unicode al resolver rutas de módulos

- **Módulo:** `源码/*.zy`, `源码/国际化/*.zy` — cualquier módulo dentro de un directorio con nombre chino.
- **Contexto:** Importaciones relativas (`<# ./召模 <= 模`, `<# ../召模 <= 模`) dentro de directorios con nombres en caracteres Unicode.
- **Descripción:** El resolver de módulos del LSP URL-encodea los segmentos chinos de la ruta antes de buscar el archivo en el filesystem. El directorio `源码` se convierte en `%E6%BA%90%E7%A0%81`, produciendo un path que no existe como nombre de archivo literal. El resultado son errores `module-not-found` en el editor para todos los módulos dentro de `源码/` que importan otros módulos del mismo directorio, y para los adapters en `源码/国际化/` que importan desde `源码/`. **El CLI (`zymbol check`, `zymbol run`) resuelve los mismos paths correctamente — el bug es exclusivo del LSP.**
- **Caso mínimo:**
  ```
  // archivo: 源码/译文.zy
  <# ./召模 <= 模   // LSP busca %E6%BA%90%E7%A0%81/召模.zy → not found
                    // CLI busca 源码/召模.zy → found ✓
  ```
- **Workaround:** Ninguno disponible sin renombrar el directorio a ASCII. El tool funciona correctamente en ejecución; los errores son únicamente visuales en el editor.
- **Propuesta:** El resolver de módulos del LSP debe normalizar las rutas usando el path del filesystem directamente (sin URL-encoding) al construir rutas absolutas desde importaciones relativas.
- **Estado:** abierto

---

## GAP — Capacidad ausente en el lenguaje

Construcciones o comportamientos que otros lenguajes tienen, que se necesitan para completar ZyAudit pero que Zymbol aún no implementa. No se consideran bugs — son características pendientes.

| ID | Módulo | Capacidad ausente | Estado |
|----|--------|-------------------|--------|
| [GAP-001](#gap-001--bounds-aritméticos-en-slices) | `解析.zy` | Expresiones aritméticas como bounds en `$[start..end]` | workaround |
| [GAP-002](#gap-002--expresión-parentizada-como-ítem-de-) | `解析.zy` | Expresión parentizada como ítem de `$++` | workaround |
| [GAP-003](#gap-003--warning-ambiguous-lifetime-en-variables-de-iteración-) | todos | Warning `ambiguous lifetime` en toda variable de iteración `@ var:array` | propuesto |

---

### GAP-001 · Bounds aritméticos en slices

- **Módulo:** `解析.zy`
- **Capacidad ausente:** Usar expresiones aritméticas directamente como extremos del operador slice `$[start..end]`. Actualmente solo acepta literales enteros o identificadores simples.
- **Caso de uso en ZyAudit:** Parsear salida de `grep -n` requiere calcular posiciones dinámicas: tomar los caracteres antes del `:` (`$[1..p-1]`) y después (`$[p+1..-1]`), donde `p` es el índice encontrado con `$??`.
- **Workaround actual:** Pre-calcular los bounds en variables intermedias antes del slice:
  ```
  // FALLA — aritmética inline en bounds:
  // nlin = linea$[1..p-1]   ← parse error

  // CORRECTO — pre-calcular:
  pend = p - 1
  pst  = p + 1
  nlin  = linea$[1..pend]
  resto = linea$[pst..-1]
  ```
- **Propuesta:** Permitir expresiones aritméticas simples (`+`, `-`) dentro de los bounds de slice:
  ```
  nlin  = linea$[1..p-1]    // debería funcionar
  resto = linea$[p+1..-1]   // debería funcionar
  ```
- **Estado:** workaround

---

### GAP-002 · Expresión parentizada como ítem de `$++`

- **Módulo:** `解析.zy`
- **Capacidad ausente:** El operador `$++` (build string/array) no acepta expresiones parentizadas como ítems. Solo acepta literales, identificadores simples o llamadas a función directas.
- **Caso de uso en ZyAudit:** Convertir el resultado de un postfix operator (`arr$#`) a string directamente dentro de `$++`:
  ```
  // Intento: convertir longitud de array a string en una línea
  nparams = "" $++ (pparts$#)   ← parse error
  ```
- **Workaround actual:** Variable intermedia para el valor parentizado:
  ```
  // CORRECTO:
  cnt     = pparts$#
  nparams = "" $++ cnt
  ```
- **Propuesta:** Que `$++` acepte expresiones parentizadas como ítems, en línea con el comportamiento de `>>`:
  ```
  // >> ya acepta esto correctamente:
  >> "len=" (arr$#) ¶

  // $++ debería aceptar lo mismo:
  s = "len=" $++ (arr$#)
  ```
- **Estado:** workaround

---

### GAP-003 · Warning `ambiguous lifetime` en variables de iteración `@ var:array`

- **Módulo:** `主程.zy`, `test_解析.zy`, `test_计量.zy` y cualquier archivo con loops `@ var:array`
- **Capacidad ausente:** El analizador semántico emite `warning: ambiguous lifetime for 'X'` para **toda** variable de iteración en un loop `@ var:array { }`, independientemente de si la variable es usada o no, y sin importar si el nombre se repite.
- **Descripción:** El mensaje de ayuda dice `consider using explicit lifetime annotation`, pero en la práctica ningún mecanismo disponible (`\ var` para destrucción explícita, prefijo `_`, variable distinta) suprime el warning. El warning se dispara porque el analizador considera que la variable de iteración es "modificada dentro del loop" (la reasignación ocurre cada iteración), pero esto es comportamiento esperado del mecanismo de loop y no un error del programador.
- **Caso mínimo:**
  ```
  arr = ["a", "b", "c"]
  @ elem:arr {           // → warning: ambiguous lifetime for 'elem'
      >> elem ¶
  }
  ```
- **Workarounds intentados sin éxito:**
  ```
  \ elem      // destrucción explícita post-loop — no suprime el warning
  @ _elem:arr // prefijo _ — no suprime el warning
  ```
- **Propuesta — anotación de lifetime explícita en la cabecera del loop:**
  El warning ya sugiere `consider using explicit lifetime annotation`. La solución es introducir esa sintaxis directamente en el operador `@`, usando `\` (el operador de destrucción existente) adjunto a la variable de iteración para declarar que su lifetime queda acotado a cada iteración del loop:
  ```
  // Sintaxis actual (genera warning):
  @ elem:arr { >> elem ¶ }

  // Sintaxis propuesta (lifetime explícito — sin warning):
  @ elem\:arr { >> elem ¶ }
  //     ^ \ adjunto al nombre declara: "elem vive solo dentro de cada iteración"
  ```
  Esta sintaxis reutiliza el símbolo `\` ya existente en el lenguaje (`\ x` para destrucción explícita) sin introducir nueva vocabulario. El analizador semántico reconocería `var\` en la cabecera de loop como "variable de iteración con lifetime acotado" y no emitiría `ambiguous lifetime`. En loops donde la variable de iteración sí se necesite fuera del bloque (caso legítimo), se usaría la forma sin `\` y el warning actuaría como advertencia real.
- **Estado:** propuesto

---

## ERROR — Error de compilación o runtime no documentado

Errores producidos por el intérprete o compilador que apuntan a una limitación no documentada del lenguaje.

| ID | Módulo | Error producido | Estado |
|----|--------|-----------------|--------|
| — | — | — | — |

---

## IDEA — Propuestas de mejora al lenguaje

Mejoras al lenguaje Zymbol inspiradas directamente en la experiencia de construir en mandarín.

| ID | Área | Resumen | Estado |
|----|------|---------|--------|
| [IDEA-001](#idea-001--raw-strings-para-bashexec-complejo) | sintaxis | Raw strings para BashExec con `{` `}` literales | propuesto |

---

### IDEA-001 · Raw strings para BashExec complejo

- **Área:** sintaxis
- **Motivación:** Cualquier BashExec que contenga sintaxis con llaves — awk, jq, funciones de shell, Python inline — requiere escapar manualmente cada `{` como `\{` y cada `}` como `\}`. Esto crea fricción importante al portar comandos existentes y hace el código difícil de leer y comparar contra la documentación de las herramientas externas.

  Ejemplo real de `解_计算深度`:
  ```
  // Lo que se escribe en Zymbol:
  prof = <\ "awk 'BEGIN\{d=0;m=0\}\{for(i=1;i<=length($0);i++)\{c=substr($0,i,1);if(c==\"\{\")\{d++;if(d>m)m=d\}else if(c==\"\}\")\{d--\}\}\}END\{print m\}' '" 路径 "'" \>

  // Lo que el programador quiere escribir (awk nativo, sin escapar):
  // awk 'BEGIN{d=0;m=0}{for(i=1;i<=length($0);i++){...}}END{print m}' file
  ```
- **Propuesta:** Sintaxis de raw string para BashExec que deshabilite la interpolación `{var}` y trate `{` y `}` como literales. La interpolación se expresaría con una sintaxis alternativa:
  ```
  // Propuesta A — delimitador triple o backtick para raw BashExec:
  prof = <\ `awk 'BEGIN{d=0;m=0}{...}END{print m}' '` 路径 `'` \>

  // Propuesta B — bloque raw con ${var} para interpolación explícita:
  prof = <\raw "awk 'BEGIN{d=0;m=0}{...}END{print m}' '${路径}'" \>
  ```
- **Impacto estimado:** Afecta a todo proyecto Zymbol que use BashExec con herramientas como awk, jq, sed, python3 inline, o cualquier script de shell con bloques `{ }`. Reduce la barrera de entrada para integrar comandos existentes sin traducción manual.
- **Estado:** propuesto

---

## Resumen de estado

| Categoría | Total | Abiertos | Con workaround | Propuestos | Resueltos |
|-----------|-------|----------|----------------|------------|-----------|
| BUG | 3 | 1 | 2 | 0 | 0 |
| GAP | 3 | 0 | 2 | 1 | 0 |
| ERROR | 0 | 0 | 0 | 0 | 0 |
| IDEA | 1 | 0 | 0 | 1 | 0 |
| **Total** | **7** | **1** | **4** | **2** | **0** |

---

## Historial de resoluciones

Entradas movidas aquí cuando pasan a estado `resuelto`.

| ID | Título | Resuelto en | Cómo |
|----|--------|-------------|------|
| — | — | — | — |
