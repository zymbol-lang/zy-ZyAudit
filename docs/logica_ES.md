# ZyAudit · logica.zy

## Métricas

| Métrica | Valor |
|---|---|
| Líneas totales | 134 |
| Líneas de código | 105 |
| Funciones | 8 |
| Anidamiento máx. | 4 |
| Complejidad ciclomática | 24 |
| Cobertura de documentación | 100% |
| Sin usar |  |

---

## Funciones documentadas

### lcg_sig

// 功能：Genera el siguiente valor pseudoaleatorio utilizando un Generador Congruencial Lineal (LCG) con parámetros fijos: 1664525, 1013904223 y módulo 2147483647.
// 参数：semilla: La semilla actual o el último valor generado de la secuencia, que debe ser un número entero.
// 返回：Un número entero en el rango [0, 2147483646], representando el siguiente número pseudoaleatorio de la secuencia LCG.

---

### rango_aleatorio

// Función：Genera un número entero aleatorio dentro de un rango cerrado [min, max], utilizando la semilla proporcionada en el proceso LCG.
// Parámetros：semilla: La semilla actual utilizada para generar la secuencia pseudoaleatoria.; min: El límite inferior inclusivo del rango deseado.; max: El límite superior inclusivo del rango deseado.
// Retorno：Devuelve una tupla que contiene el valor aleatorio generado (val) y la nueva semilla resultante (sig). Se requiere que `min` no sea mayor que `max`.

---

### fruta_aleatoria

// 功能：Esta función selecciona aleatoriamente un fruto de una lista fija utilizando una semilla para asegurar la reproducibilidad del resultado.
// 参数：semilla: El valor actual de la semilla necesario para calcular el índice aleatorio y determinar el siguiente estado de la semilla.
// 返回：Una tupla (fruto, s1) donde 'fruto' es la cadena Unicode del fruto seleccionado y 's1' es el valor actualizado de la semilla.

---

### nueva_comida

// 功能：Genera y posiciona una nueva comida en el tablero de juego asegurando que la ubicación no colisione con ninguna parte del cuerpo de la serpiente.
// 参数：serpiente: La estructura actual de coordenadas ocupadas por la serpiente.; AN: El ancho máximo permitido del tablero (columna límite); AL: La altura máxima permitida del tablero (fila límite); semilla: Semilla inicial para la generación de números aleatorios.
// 返回：Una tupla que contiene la nueva posición coordinada ((r, c)) donde se colocará la comida y un valor actualizado relacionado con el estado o semilla utilizada para la generación.

---

### tick_comida

// 功能：Genera las nuevas coordenadas para los recursos de comida y fruta en un entorno simulado, ejecutando el paso o "tick" de la simulación.
// 参数：comio: Bandera booleana que determina si se debe proceder con la generación de nuevos recursos. Si es falso o nulo, se ejecuta una salida temprana.
// 
// serpiente: La posición o estado actual de la serpiente (la entidad principal).
// AN: Límite del área de juego en dirección Norte.
// AL: Límite del área de juego en longitud o dimensión lateral.
// semilla: Datos iniciales o "seed" utilizados para la generación pseudoaleatoria de los recursos.
// comida: Estado actual o parámetro de entrada relacionado con la comida (puede ser utilizado en caso de retorno temprano).
// fruta: Estado actual o parámetro de entrada relacionado con la fruta (puede ser utilizado en caso de retorno temprano).
// 
// 返回：Una tupla que contiene tres valores generados tras el cálculo del nuevo ciclo de vida: (npos, nfru, s2), donde npos es la nueva posición de comida, nfru es la información de fruta y s2 es el nuevo estado general del sistema.
// 
// 边界条件：Si el parámetro `comio` no es verdadero (es falso o nulo), la función retorna inmediatamente los valores iniciales pasados en los parámetros de entrada de comida, fruta y semilla, sin realizar ningún cálculo de nueva posición ni estado.

---

### nueva_dir

// 功能：Calcula la próxima dirección de movimiento válida basándose en una tecla pulsada y la dirección actual, previniendo el giro inmediato hacia atrás.
// 参数：tecla: La tecla que intenta mover el personaje ('w', '↑', etc.). dir: La dirección de movimiento actual del personaje ('↑', '↓', '←', '→').
// 返回：Una cadena que representa la nueva dirección válida. Si la intención es moverse en dirección opuesta a `dir` (giro invertido), devuelve `dir`; de lo contrario, devuelve la dirección deseada por `tecla`.

---

### _en_serpiente

// Función: Verifica si una posición dada (`pos`) está contenida en el cuerpo de la serpiente (`serpiente`).
// Parámetros:
//   - serpiente: El arreglo que representa las coordenadas del cuerpo de la serpiente.
//   - pos: La posición específica a buscar dentro del cuerpo de la serpiente.
// Retorno:
//   - #1: Si `pos` es encontrado en el cuerpo de la serpiente.
//   - #0: Si `pos` no se encuentra en el cuerpo de la serpiente.
// Condicionales: El manejo de un arreglo vacío (`serpiente$# = 0`) devuelve correctamente #0.

---

### mover

// Función：Calcula y ejecuta un paso de movimiento para la serpiente en el juego. Determina la nueva posición de la cabeza, comprueba colisiones (paredes o consigo misma), actualiza la puntuación si se come comida, y gestiona el crecimiento del cuerpo.
// Parámetros：
//   serpiente: Array/lista que representa el cuerpo de la serpiente. [1] es la cabecera.
//   dir: String que indica la dirección del movimiento ('↑', '↓', '←', '→').
//   comida: Par (fila, columna) donde se encuentra la fruta comestible.
//   puntos: Número entero que guarda la puntuación actual del jugador.
//   AN: Límite de columnas disponibles en el mapa.
//   AL: Límite de filas disponibles en el mapa.
//   crecer: Booleano (true/false) que indica si la cola debe permanecer fija (manejo del crecimiento diferido).
// Retorno：El nuevo estado del juego, un tuple #(0, nueva_serpiente, nuevos_puntos, 0), o nil en caso de colisión.
// Consideraciones：Si los límites AL o AN son insuficientes para el movimiento calculado, se considera una colisión con la pared y no hay retorno exitoso. Si ocurre auto-colisión (encuentra otra parte del cuerpo) también retorna nil.

---

