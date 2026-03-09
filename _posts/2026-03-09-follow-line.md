---
layout: post
title: "Follow Line – Control reactivo con estimación de curvatura"
date: 2026-03-09
thumbnail: "/images/coxe.png"
excerpt: "Seguimiento de línea mediante visión por computador y control PID adaptativo"
published: true
---

![Imagen de seguimiento de línea](/images/coxe.png)

# Follow Line – Control reactivo con estimación de curvatura

En esta práctica se desarrolla un sistema de **seguimiento de línea basado en visión artificial**, utilizando un controlador PID combinado con varias heurísticas que permiten anticipar curvas, mejorar la estabilidad y recuperar la trayectoria cuando la línea se pierde.

Tras varias iteraciones de mejora, el sistema actual ha conseguido completar el **circuito simple en aproximadamente 58 segundos**, manteniendo estabilidad incluso en tramos con curvatura pronunciada.

---

# Arquitectura del algoritmo

El sistema se divide en tres bloques principales:

1. **Procesamiento de imagen**
2. **Estimación de trayectoria**
3. **Control dinámico del vehículo**

Cada uno de estos bloques contribuye a mejorar la robustez del sistema frente a cambios de trayectoria o pérdidas momentáneas de la línea.

---

# Procesamiento de imagen

La detección de la línea se realiza mediante segmentación en el espacio de color **HSV**, lo que permite detectar de forma robusta el color rojo independientemente de cambios moderados de iluminación.

```python
hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)
mask1 = cv2.inRange(hsv, lower_red1, upper_red1)
mask2 = cv2.inRange(hsv, lower_red2, upper_red2)
mask = cv2.bitwise_or(mask1, mask2)
```

## Posteriormente: estimación de trayectoria y control del vehículo

Una vez obtenida la máscara binaria de la línea roja, el siguiente paso consiste en estimar la trayectoria de la línea y generar el control del vehículo.

Para ello, en lugar de calcular únicamente un centroide del contorno, el sistema analiza varias **filas horizontales de la imagen**:

```python
scan_rows = [30, 60, 90, 130, 165]
```

En cada una de estas filas se detectan los píxeles pertenecientes a la línea y se calcula su centro. Esto permite obtener una representación más robusta de la posición de la línea incluso si existen pequeñas discontinuidades.

Cada fila tiene además un peso diferente, dando mayor importancia a las filas más cercanas al vehículo:
```python
weights = [1.0, 1.2, 1.5, 2.0, 2.2]
```

Con estos valores se calcula un centro ponderado que representa la posición estimada de la línea en la imagen.
