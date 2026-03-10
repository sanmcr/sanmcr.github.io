---
layout: post
title: "Follow Line – Evolución de un controlador visual"
date: 2026-03-09
thumbnail: "/images/coxe.png"
excerpt: "Seguimiento de línea mediante visión artificial y control PID, analizando estabilidad, velocidad y comportamiento en distintos circuitos."
published: true
---

![Imagen de seguimiento de línea](/images/coxe.png)

En esta práctica he desarrollado un sistema de **seguimiento de línea mediante visión artificial** cuyo objetivo es completar distintos circuitos de forma autónoma.

Aunque el problema puede parecer sencillo, en la práctica el comportamiento del robot depende de muchos factores:  
la detección visual, el controlador de dirección, la velocidad y la geometría de las curvas.

A lo largo del desarrollo fui probando **distintas versiones del controlador**, cada una con un compromiso diferente entre velocidad y estabilidad.

<!--more-->

---

# Arquitectura general del sistema

El algoritmo se basa en tres bloques principales:

- **Procesamiento de imagen**
- **Estimación de trayectoria**
- **Control del robot**

La línea se detecta mediante segmentación de color en el espacio **HSV**, lo que permite aislar de forma robusta el color rojo incluso cuando existen pequeñas variaciones de iluminación.

Para reducir el coste computacional, el análisis se realiza únicamente sobre la **parte inferior de la imagen (ROI)**, ya que es donde aparece la información más relevante para el control inmediato del robot.

En lugar de calcular un único punto de la línea, el sistema analiza **varias filas horizontales de la imagen**, obteniendo el centro de la línea en cada una de ellas.  

Posteriormente estos puntos se combinan mediante una **media ponderada**, dando mayor peso a las filas más cercanas al robot, ya que influyen más directamente en la dirección del movimiento.

---

# Estimación de curvatura

Para mejorar el comportamiento en curvas añadí una pequeña estimación de curvatura.

La idea consiste en utilizar dos puntos de la línea:

- un punto **cercano al robot**
- un punto **más lejano en la imagen**

La diferencia entre ambos permite estimar si la trayectoria comienza a girar.

Cuando esta diferencia aumenta, el controlador utiliza más información del punto lejano para **anticipar el giro antes de que la curva llegue al robot**, lo que mejora significativamente la estabilidad en curvas pronunciadas.

---

# Control PID

El error lateral se calcula como la diferencia entre el centro de la imagen y la posición estimada de la línea.

Este error se introduce en un **controlador PID**, que genera la velocidad angular del robot.

Cada término del controlador tiene un papel diferente:

- **Proporcional (Kp)** corrige el error actual  
- **Integral (Ki)** compensa errores acumulados  
- **Derivativo (Kd)** suaviza cambios bruscos  

Además, el error se filtra mediante un **suavizado exponencial**, lo que reduce el efecto del ruido en la detección visual y evita correcciones excesivamente bruscas.

---

# Evolución del controlador

Durante el desarrollo fui probando varias configuraciones del controlador con distintos compromisos entre **velocidad de recorrido y estabilidad**.

Esto dio lugar a dos versiones principales.

---

# Versión rápida (control nervioso)

La primera versión del controlador estaba orientada principalmente a **completar el circuito lo más rápido posible**.

El robot respondía de forma muy agresiva a los cambios de error, lo que permitía mantener una velocidad alta en rectas.

Sin embargo, esta estrategia generaba **oscilaciones constantes en curvas**, ya que el robot corregía continuamente su posición respecto a la línea.

Este comportamiento producía un movimiento lateral continuo intentando volver al centro de la trayectoria.

A pesar de estas oscilaciones, el circuito simple se completa aproximadamente en:

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

---

# Versión estable

Posteriormente desarrollé una versión más refinada del controlador introduciendo varios cambios:

- mayor filtrado del error
- limitación de cambios bruscos en la velocidad angular
- ajuste más fino de la velocidad en función del error lateral

Estas modificaciones reducen considerablemente las oscilaciones, produciendo un movimiento más suave y estable.

El robot sigue la línea de forma más consistente, especialmente en curvas cerradas.

La contrapartida es que esta versión sacrifica algo de velocidad respecto a la versión anterior.

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

También probé ejecutar el mismo circuito **en sentido contrario**.

Aunque el algoritmo es exactamente el mismo, el comportamiento cambia porque las curvas se afrontan desde el lado opuesto.

Esto hace que algunas curvas que antes eran suaves se conviertan en curvas más difíciles de anticipar.

Como resultado, el robot necesita más correcciones para mantenerse sobre la línea.

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

Además del circuito simple, probé el algoritmo en un circuito más complejo inspirado en **Montmeló**.

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

Este comportamiento parece estar relacionado con dos factores principales:

- la presencia de **curvas abiertas muy largas**
- la dificultad de anticipar cambios de trayectoria utilizando únicamente un controlador reactivo

En este tipo de curvas el error lateral crece muy lentamente, por lo que el controlador tarda más en reaccionar y el robot puede empezar a oscilar hasta terminar saliéndose de la trayectoria.

Esto muestra una limitación típica de los controladores reactivos basados únicamente en el error lateral.

---

# Modelo Ackermann

El modelo de vehículo utilizado en el simulador sigue una cinemática de tipo **Ackermann**, típica de vehículos con dirección en las ruedas delanteras.

En este tipo de geometría las ruedas delanteras no giran con el mismo ángulo, sino que cada una describe una circunferencia distinta para evitar deslizamientos laterales.

Durante la práctica intenté observar este comportamiento en el simulador. Sin embargo, la visualización del modelo Ackermann no funciona correctamente en la interfaz utilizada, por lo que no es posible ver directamente la geometría de giro de las ruedas.

Por este motivo no puedo verificar visualmente cómo se aplica el modelo en el simulador.

Aun así, el comportamiento dinámico del robot sugiere que la cinemática se está aplicando correctamente, ya que el vehículo responde de forma coherente a los comandos de velocidad lineal y angular.

![Circuito Ackermann](/images/ackermandos.png)

---

# Resultados

Los tiempos aproximados obtenidos fueron:

| Versión | Tiempo |
|-------|------|
| Control nervioso | ~58 s |
| Control estable | ~63 s |
| Circuito en sentido inverso | ~78 s |

Estos resultados reflejan el compromiso clásico entre **velocidad y estabilidad** en sistemas de control.

Un controlador más agresivo permite completar el circuito más rápido, pero introduce mayores oscilaciones y mayor riesgo de salida de la trayectoria.

---

# Conclusión

El seguimiento de línea mediante visión artificial es un problema aparentemente sencillo, pero en la práctica requiere equilibrar varios elementos:

- percepción visual robusta  
- estimación de trayectoria  
- control dinámico del vehículo  
- ajuste fino de parámetros  

A lo largo de la práctica pude comprobar cómo **pequeños cambios en el controlador producen diferencias significativas en el comportamiento del robot**.

Este proyecto me ha permitido explorar la interacción entre **visión por computador y control reactivo**, dos componentes fundamentales en robótica móvil.
