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
