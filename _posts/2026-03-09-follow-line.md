---
layout: post
title: "Follow Line – Control reactivo con estimación de curvatura"
date: 2026-03-09
thumbnail: "/images/coxe.png"
excerpt: "Seguimiento de línea basado en visión artificial, estimación de curvatura y control PID adaptativo."
published: true
---

![Imagen de seguimiento de línea](/images/coxe.png)

En esta práctica se desarrolla un sistema de seguimiento de línea basado en visión artificial, utilizando un controlador PID combinado con varias heurísticas que permiten anticipar curvas, mejorar la estabilidad y recuperar la trayectoria cuando la línea se pierde.

Tras varias iteraciones de mejora, el sistema actual ha conseguido completar el circuito simple en aproximadamente 58 segundos, manteniendo estabilidad incluso en tramos con curvatura pronunciada.

<!--more-->

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

Estimación de curvatura

Para anticipar las curvas se utilizan dos puntos característicos:

un punto lejano (c_far)

un punto cercano (c_near)

La diferencia entre ambos permite estimar la curvatura del tramo:
```python
curve_px = abs(c_near - c_far)
```
Si la diferencia es grande, significa que el vehículo se aproxima a una curva pronunciada.

En función de este valor, el punto de control se ajusta mezclando el centro ponderado con el punto lejano. De esta forma el vehículo comienza a girar antes de llegar a la curva.

Cálculo del error

El error lateral se calcula como la diferencia entre el centro de la imagen y el punto estimado de la línea:
```python
error = center - cX
```
Este valor representa cuánto debe corregir el vehículo su dirección para volver a situarse sobre la línea.

Control PID

El error calculado se introduce en un controlador PID, encargado de generar la velocidad angular del robot:
```python
w_cmd = Kp * err_f + Ki * integral + Kd * derivative
```
Cada término del PID tiene una función diferente:

Proporcional (Kp) → corrige el error actual

Integral (Ki) → compensa errores acumulados

Derivativo (Kd) → suaviza cambios bruscos

El cálculo de la derivada utiliza el tiempo real entre iteraciones (dt), lo que mejora la estabilidad del controlador.

Suavizado del error

Para evitar oscilaciones provocadas por pequeñas variaciones en la detección de la línea, el error se filtra mediante un suavizado exponencial:
```python
err_f = err_alpha * error + (1 - err_alpha) * err_f
```
Esto reduce el ruido y hace que el sistema sea más estable.

Control adaptativo de velocidad

La velocidad del vehículo se adapta en función de la magnitud del error:

errores pequeños → mayor velocidad

errores grandes → reducción de velocidad
```python
if e < 18:
    v = 12.5
elif e < 45:
    v = 9.0
```
Esto permite circular rápidamente en rectas y disminuir la velocidad en curvas.

Limitador de giro

Para evitar cambios bruscos de dirección, se limita el valor máximo del giro permitido:
```python
w_cmd = clamp(w_cmd, -max_w, max_w)
```
Además, se introduce un limitador de variación por ciclo que evita cambios demasiado rápidos en la dirección.

Recuperación cuando se pierde la línea

Si el sistema detecta menos de dos filas válidas, significa que la línea se ha perdido. En ese caso el robot entra en un modo de búsqueda:
```python
if valid < 2:
```
El vehículo gira hacia el último lado donde se detectó la línea:
```python
w_cmd = 6.0 * last_seen_side
```
Esto permite recuperar la trayectoria sin detener completamente el movimiento.

## Resultados

Con esta versión del algoritmo, el sistema ha logrado completar el **circuito simple en aproximadamente 58 segundos**.

Este resultado supone una mejora significativa respecto a versiones anteriores del controlador, manteniendo además un comportamiento estable en curvas.

Actualmente se están realizando pruebas adicionales en circuitos más complejos para evaluar la robustez del sistema.

---

## Conclusión

El sistema final combina:

- detección robusta de la línea mediante HSV  
- análisis de múltiples filas de la imagen  
- estimación de curvatura de la trayectoria  
- control PID con suavizado  
- control adaptativo de velocidad  
- recuperación automática de la línea  

Todo ello permite obtener un comportamiento **rápido y estable en el seguimiento de línea**.

Esta práctica ha permitido profundizar en la integración entre **visión por computador y control reactivo en robots móviles**.
