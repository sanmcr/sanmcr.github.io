---
layout: post
title: "Follow Line – Evolución de un controlador visual"
date: 2026-03-09
thumbnail: "/images/coxe.png"
excerpt: "Seguimiento de línea mediante visión artificial y control PID, analizando estabilidad, velocidad y comportamiento en distintos circuitos."
published: true
---

![Imagen de seguimiento de línea](/images/coxe.png)

En esta práctica se desarrolla un sistema de **seguimiento de línea mediante visión artificial** cuyo objetivo es completar distintos circuitos de forma autónoma.

Aunque el problema puede parecer sencillo, el comportamiento del robot depende de muchos factores:  
la detección visual, el controlador de dirección, la velocidad y el tipo de curva del circuito.

A lo largo de la práctica se han desarrollado **varias versiones del controlador**, cada una con comportamientos distintos en términos de estabilidad y velocidad.

<!--more-->

---

# Arquitectura general del sistema

El algoritmo se basa en tres bloques principales:

- **Procesamiento de imagen**
- **Estimación de trayectoria**
- **Control del robot**

La línea se detecta mediante segmentación de color en el espacio **HSV**, lo que permite aislar de forma robusta el color rojo.

Para reducir el coste computacional, el análisis se realiza únicamente sobre la **parte inferior de la imagen (ROI)**, ya que es donde aparece la información relevante para el control inmediato.

La posición de la línea se estima utilizando **varias filas horizontales**, calculando el centro de la línea en cada una de ellas y combinándolos mediante una media ponderada.

Las filas más cercanas al robot tienen mayor peso, ya que influyen más directamente en la dirección del movimiento.

---

# Estimación de curvatura

Para anticipar curvas se utilizan dos puntos de la línea:

- un punto cercano al robot
- un punto más lejano en la imagen

La diferencia entre ambos permite estimar la curvatura del tramo.

Cuando esta diferencia aumenta, el algoritmo utiliza mayor información del punto lejano para **anticipar el giro antes de que la curva llegue al robot**, lo que mejora significativamente el comportamiento en curvas pronunciadas.

---

# Control PID

El error lateral se calcula como la diferencia entre el centro de la imagen y la posición estimada de la línea.

Este error se introduce en un **controlador PID**, que genera la velocidad angular del robot.

Cada término del controlador tiene un papel diferente:

- **Proporcional (Kp)** corrige el error actual  
- **Integral (Ki)** compensa errores acumulados  
- **Derivativo (Kd)** suaviza cambios bruscos  

Además, el error se filtra mediante un suavizado exponencial para reducir el efecto del ruido en la detección visual.

---

# Evolución del controlador

Durante el desarrollo se probaron varias configuraciones con distintos compromisos entre velocidad y estabilidad.

---

# Versión rápida (control nervioso)

La primera versión del controlador priorizaba la **velocidad del recorrido**.

En esta configuración el robot respondía de forma muy agresiva a los cambios de error, lo que permitía mantener una velocidad alta en rectas pero generaba **oscilaciones constantes en curvas**.

Esto provocaba un movimiento lateral continuo intentando corregir el error.

Aun así, el circuito simple se completa en aproximadamente:

**≈58 segundos**

### Vídeo

<div style="text-align:center;">
<iframe width="700" height="394"
src="https://www.youtube.com/embed/6KSY_JKuLIg"
title="Control nervioso"
frameborder="0"
allowfullscreen>
</iframe>
</div>

# Versión estable

Posteriormente se desarrolló una versión más refinada del controlador introduciendo:

- mayor filtrado del error
- limitación de cambios bruscos en la dirección
- ajuste fino de la velocidad según el error lateral

Esta versión reduce considerablemente las oscilaciones, produciendo un movimiento más suave y estable.

Sin embargo, esta mejora en estabilidad implica una ligera pérdida de velocidad.

El circuito simple se completa aproximadamente en:

**≈63 segundos**

### Vídeo

<div style="text-align:center;">
<iframe width="700" height="394"
src="https://www.youtube.com/embed/stXZXhcyrKI"
title="Control estable"
frameborder="0"
allowfullscreen>
</iframe>
</div>

---

# Circuito recorrido en sentido inverso

También se probó ejecutar el circuito en sentido contrario.

Aunque el algoritmo es exactamente el mismo, el comportamiento cambia debido a que:

- la geometría de las curvas se afronta desde el lado opuesto
- algunas curvas se vuelven más difíciles de anticipar

En este caso el tiempo total del circuito aumenta hasta aproximadamente:

**≈78 segundos**

### Vídeo

<div style="text-align:center;">
<iframe width="700" height="394"
src="https://www.youtube.com/embed/tnBgYsc1wCk"
title="Circuito inverso"
frameborder="0"
allowfullscreen>
</iframe>
</div>
---

# Prueba en circuito Montmeló

Además del circuito simple se probó el algoritmo en un circuito más complejo inspirado en **Montmeló**.


En este circuito el robot comienza siguiendo la línea correctamente, pero termina saliéndose en un punto concreto.

### Vídeo

<div style="text-align:center;">
<iframe width="700" height="394"
src="https://www.youtube.com/embed/7oBNXKtt-oE"
title="Montmeló"
frameborder="0"
allowfullscreen>
</iframe>
</div>

Este comportamiento se debe principalmente a dos factores:

- presencia de **curvas abiertas muy largas**, donde el error lateral crece lentamente  
- dificultad para anticipar cambios de dirección con un controlador puramente reactivo

En este tipo de curvas el robot tiende a oscilar ligeramente hasta que la corrección acumulada se vuelve demasiado grande.

Esto muestra una limitación típica de los controladores reactivos basados únicamente en error lateral.

---

# Modelo Ackermann

El modelo del vehículo utilizado en el simulador sigue una geometría **Ackermann**, típica de vehículos con dirección delantera.

En el simulador utilizado la visualización del modelo Ackermann no funciona correctamente, por lo que no es posible observar la geometría de giro en pantalla.

Sin embargo, el comportamiento del robot indica que la cinemática sigue aplicándose correctamente, ya que el vehículo responde a los comandos de velocidad lineal y angular como se espera.

![Circuito Ackerman](/images/ackermandos.png)

---

# Resultados

Los tiempos aproximados obtenidos fueron:

| Versión | Tiempo |
|-------|------|
| Control nervioso | ~58 s |
| Control estable | ~63 s |
| Circuito en sentido inverso | ~78 s |

Esto refleja el clásico compromiso entre **velocidad y estabilidad** en sistemas de control.

Un controlador más agresivo permite completar el circuito más rápido, pero genera mayores oscilaciones y riesgo de salida de la trayectoria.

---

# Conclusión

El seguimiento de línea mediante visión artificial es un problema aparentemente sencillo, pero que en la práctica requiere equilibrar múltiples factores:

- Percepción visual robusta  
- Estimación de trayectoria  
- Control dinámico del vehículo  
- Ajuste fino de parámetros  

El sistema desarrollado demuestra cómo pequeños cambios en el controlador pueden producir diferencias significativas en el comportamiento del robot.

Esta práctica ha permitido explorar la interacción entre **visión por computador y control reactivo**, dos elementos fundamentales en robótica móvil.
