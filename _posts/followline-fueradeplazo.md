---
layout: post
title: Control visual un robot, Fórmula1 (fuera de plazo)
date: 2025-05-04
thumbnail: images/p3.jpg
excerpt: "Versión mejorada de Follow Line"
---


#  Práctica: Seguimiento de Línea en Fórmula 1 (Visión en Robótica)

##  Objetivo

Diseñar un sistema reactivo para un coche simulado tipo Fórmula 1 que siga una línea roja en el asfalto, con el objetivo de **completar una vuelta en el menor tiempo posible** sin salirse del circuito ni chocar.

---

##  Enfoque inicial (versión lenta)

###  Sistema visual:
- Conversión de imagen de la cámara a HSV.
- Filtrado del color rojo mediante dos rangos de máscara.
- Obtención del **contorno más grande** y cálculo del **centroide** (`cv2.moments`) como punto de referencia.

### Control de dirección:
- Error = diferencia entre el centro de la imagen y `cX` del centroide.
- Control PID con parámetros fijos.
- Suavizado exponencial de dirección con `alpha = 0.7`.

###  Velocidad:
- Asignación en función de curvatura simple (basada en cX actual vs anterior).
- Velocidad máxima: **7.5** unidades.
- Tres velocidades fijas: recta, curva suave y curva cerrada.

---

##  Mejoras aplicadas (versión optimizada – 63s)

###  Cambio de punto de referencia: vértice superior
- Se usa el **punto más alto del contorno** (`topmost`) para anticipar mejor curvas.
- Mayor anticipación → mejor control en curvas suaves.

###  PID real con `dt`
- Cálculo de tiempo entre iteraciones (`dt`) para aplicar un PID correcto:
  - Proporcional (Kp), Integral (Ki), Derivativo (Kd) real.
- Mayor estabilidad frente a oscilaciones y cambios de FPS.

###  Curvatura avanzada
- Uso de historial de `cX` y cálculo de media de diferencias para suavizar.
- Curvatura más robusta y predecible.

###  Suavizado de giro más reactivo
- Se reduce `alpha = 0.35` → respuesta más dinámica sin sobrecorregir.

###  Velocidad dinámica mejorada
- Hasta **11.7** unidades en recta.
- Disminución controlada en función de curvatura suavizada.
- Lógica:
  ```python
  if curv < 10:     v = 11.7
  elif curv < 25:   v = 9.6
  else:             v = 4.0
 ```
