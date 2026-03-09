---
layout: post
title: "Follow Line – Control reactivo con estimación de curvatura"
date: 2026-03-09
thumbnail: "/images/coxe.png"
excerpt: "Seguimiento de línea basado en visión artificial, estimación de curvatura y control PID adaptativo."
published: true
---


![Imagen de seguimiento de línea](/images/coxe.png)

En esta práctica se desarrolla un sistema de **seguimiento de línea basado en visión artificial**, utilizando un controlador PID combinado con varias heurísticas que permiten **anticipar curvas, mejorar la estabilidad y recuperar la trayectoria** cuando la línea se pierde.

Tras varias iteraciones de mejora, el sistema actual ha conseguido completar el **circuito simple en aproximadamente 58 segundos**, manteniendo un comportamiento estable incluso en tramos con curvatura pronunciada.

<!--more-->

---

## Arquitectura del algoritmo

El sistema se divide en tres bloques principales:

1. **Procesamiento de imagen**
2. **Estimación de trayectoria**
3. **Control dinámico del vehículo**

Esta estructura permite separar claramente la parte de percepción visual de la parte de decisión y control.

---

## Procesamiento de imagen

La detección de la línea se realiza en el espacio de color **HSV**, ya que resulta más robusto que trabajar directamente sobre la imagen en RGB cuando se quiere segmentar un color concreto.

Primero se aplica un pequeño suavizado para reducir ruido y después se convierte la imagen al espacio HSV.

```python
 blurred = cv2.GaussianBlur(frame, (5, 5), 0)  
 hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)
```

A continuación se segmenta el color rojo utilizando dos rangos de HSV, ya que el rojo se encuentra dividido en dos zonas del espacio de color.


```python
mask1 = cv2.inRange(hsv, lower_red1, upper_red1)  
mask2 = cv2.inRange(hsv, lower_red2, upper_red2)  
mask = cv2.bitwise_or(mask1, mask2)
```

Después se aplican operaciones morfológicas para eliminar ruido y cerrar pequeños huecos en la detección.

```python
mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel, iterations=1)  
mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=1)
```

Para reducir el coste computacional, el análisis se realiza únicamente en la **parte inferior de la imagen (ROI)**, ya que es donde se encuentra la información relevante para el control del vehículo.

```python
roi = mask[h - roi_h : h, :]
```

---

## Estimación de trayectoria

En lugar de calcular un único centroide de la línea, el algoritmo analiza varias filas horizontales de la imagen.


```python
scan_rows = [30, 60, 90, 130, 165]
```

En cada fila se detectan los píxeles de la línea y se calcula su centro. Posteriormente se calcula una media ponderada utilizando diferentes pesos.

```python
weights = [1.0, 1.2, 1.5, 2.0, 2.2]
```
Las filas más cercanas al vehículo tienen mayor peso, ya que influyen más directamente en el control inmediato de la dirección.

---

## Estimación de curvatura

Para anticipar curvas se utilizan dos puntos característicos de la línea:

- un punto lejano
- un punto cercano

```python
c_far = row_center(roi, 60)
c_near = row_center(roi, 165)
```

La diferencia entre ambos permite estimar la curvatura del tramo.

```python
curve_px = abs(c_near - c_far)
```

Si esta diferencia aumenta significa que el vehículo se aproxima a una curva pronunciada.

En función de este valor, el algoritmo desplaza ligeramente el punto de control hacia el punto lejano para **anticipar el giro**.

---

## Cálculo del error

El error lateral se calcula como la diferencia entre el centro de la imagen y la posición estimada de la línea.

```python
error = center - cX
```

Este error indica cuánto debe girar el vehículo para volver a situarse sobre la línea.

---

## Control PID

El error calculado se introduce en un controlador PID que genera la velocidad angular del robot.

```python
w_cmd = Kp * err_f + Ki * integral + Kd * derivative
```

Cada término del controlador tiene un papel diferente:

- **Proporcional (Kp)** corrige el error actual  
- **Integral (Ki)** compensa errores acumulados  
- **Derivativo (Kd)** suaviza cambios bruscos  

El cálculo del término derivativo utiliza el tiempo real entre iteraciones (`dt`), lo que mejora la estabilidad del sistema.

---

## Suavizado del error

Para evitar oscilaciones provocadas por pequeñas variaciones en la detección de la línea, el error se filtra mediante un suavizado exponencial.

```python
err_f = err_alpha * error + (1 - err_alpha) * err_f
```

Esto permite que el control sea más estable y menos sensible al ruido.

---

## Control adaptativo de velocidad

La velocidad del vehículo se adapta dinámicamente según la magnitud del error.

Cuando el error es pequeño el robot puede avanzar más rápido, mientras que en curvas o desviaciones grandes se reduce la velocidad.

```python
 if e < 18:  
     v = 12.5  
 elif e < 45:  
     v = 9.0  
```

De esta forma se consigue un comportamiento más eficiente en rectas y más estable en curvas.

---

## Recuperación cuando se pierde la línea

Si el sistema detecta menos de dos filas válidas, significa que la línea se ha perdido.

```python
if valid < 2:
```

En ese caso el robot entra en un modo de búsqueda y gira hacia el último lado donde se detectó la línea.

```python
w_cmd = 6.0 * last_seen_side
```

Esto permite recuperar la línea rápidamente sin detener completamente el movimiento.

---

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

Esta práctica me ha permitido profundizar en la integración entre **visión por computador y control reactivo en robots móviles**.
