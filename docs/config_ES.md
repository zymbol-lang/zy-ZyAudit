# ZyAudit · config.zy

> **Revisado para v0.0.5 — 2026-05-12**

## Métricas

| Métrica | Valor |
|---|---|
| Líneas totales | 69 |
| Líneas de código | 56 |
| Funciones | 7 |
| Anidamiento máx. | 4 |
| Complejidad ciclomática | 8 |
| Cobertura de documentación | 14% |
| Sin usar |  |

---

## Funciones documentadas

### load

// Función: Leer y analizar parámetros de configuración del sistema desde el archivo de configuración especificado `~/.ZethyCLI/ZethyCLI.conf` (como modelo, modo, si está habilitado el pensamiento, etc.), y configurar las variables de estado internas correspondientes。
// Parámetros: Ninguno
// Retorna: Ninguno (Esta función produce efectos secundarios, utilizados para inicializar o cargar el estado de la configuración)

---

### save

// Función: Escribe los parámetros de configuración de la aplicación actual (modelo, modo, si hacer pensamiento, etc.) en el archivo de configuración local del usuario `~/.ZethyCLI/ZethyCLI.conf`.
// Parámetros: 
// Retorna: No tiene valor de retorno directo; la función completa la persistencia de la configuración escribiendo en un archivo (efecto secundario).

---

### get_model

// Función: Se utiliza para obtener el nombre o identificador del modelo configurado en tiempo de ejecución.
// Parámetros: Ninguno.
// Retorna: Devuelve el nombre del modelo leído de la constante de configuración `cfg_model` (generalmente una cadena).

---

### get_mode

// Función: Obtener el modo de configuración actual (cfg_mode).
// Parámetros: Ninguno
// Retorna: El valor del modo de configuración.

---

### get_think

// Función: Obtiene el valor de configuración cfg_think relacionado con el pensamiento almacenado.
// Parámetros: Ninguno.
// Retorna: El valor de configuración cfg_think recuperado.

---

### get_host

// Función: Obtener la dirección del Host predefinida, la cual se lee de la variable de configuración cfg_host.
// Parámetros: Sin parámetros.
// Retorna: Cadena de caracteres, devuelve la dirección del Host configurada.

---

### get_tmp_dir

// Función: Obtener la ruta del directorio temporal especificado en la configuración.
// Parámetros: Ninguno
// Retorna: La cadena de texto con la ruta del directorio temporal definida en el archivo de configuración.

---

