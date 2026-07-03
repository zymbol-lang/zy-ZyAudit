# ZyAudit · serpiente.zy

## Métricas

| Métrica | Valor |
|---|---|
| Líneas totales | 92 |
| Líneas de código | 65 |
| Funciones | 0 |
| Anidamiento máx. | 4 |
| Complejidad ciclomática | 10 |
| Cobertura de documentación | 0% |
| Sin usar |  |

---

## Resumen del programa

Este programa es una implementación del clásico juego de la serpiente para entornos de consola. Su objetivo principal es proporcionar una experiencia de juego interactiva en terminal, con soporte para controles WASD o flechas, además de funcionalidades para pausar y salir. El flujo inicia consultando las dimensiones de la terminal y generando una semilla aleatoria potente a partir de múltiples fuentes de entropía. La lógica del juego se ejecuta en un bucle principal que gestiona el movimiento de la serpiente, colisiones con comida/fruta, e incluye detección automática de redimensionamiento de pantalla para mantener el tablero centrado. El programa depende críticamente de dos módulos: `l` (para toda la gestión y reglas lógicas del juego) y `d` (responsable de renderizar todo en la interfaz gráfica).
