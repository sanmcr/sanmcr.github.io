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

Aunque el problema puede parecer sencillo, en la práctica el comportamiento del robot depende de varios factores:

- La detección visual de la línea
- El controlador de dirección
- La velocidad del vehículo
- La geometría de las curvas del circuito

A lo largo del desarrollo fui probando distintas configuraciones del controlador, buscando un equilibrio entre velocidad y estabilidad.

El resultado final es un controlador reactivo que funciona bien en el circuito simple, aunque presenta limitaciones en circuitos más complejos.

<!--more-->

---

## Arquitectura general del sistema

El sistema se basa en tres bloques principales:

- **Procesamiento de imagen**
- **Estimación de trayectoria**
- **Control del robot**

La línea se detecta mediante segmentación de color en el espacio HSV, lo que permite aislar de forma robusta el color rojo incluso cuando hay pequeñas variaciones de iluminación.

Para reducir el coste computacional, el análisis se realiza únicamente en la parte inferior de la imagen, donde se encuentra la información relevante para el control inmediato del vehículo.

Esta región se denomina **ROI (Region of Interest)**.

---

## Procesamiento de imagen

La detección de la línea comienza aplicando un pequeño suavizado a la imagen para reducir ruido.

Después se convierte la imagen al espacio de color **HSV**.

El rojo se detecta utilizando dos rangos de color, ya que este color aparece en dos zonas distintas dentro del espacio HSV.

Posteriormente se aplican operaciones morfológicas para:

- eliminar ruido
- cerrar pequeños huecos en la detección

Esto permite obtener una máscara más limpia de la línea.

---

## Estimación de trayectoria

En lugar de calcular un único punto de la línea, el algoritmo analiza varias filas horizontales de la imagen.

En cada una de ellas se detectan los píxeles correspondientes a la línea y se calcula su centro.

Estos centros se combinan mediante una media ponderada, donde las filas más cercanas al robot tienen mayor peso.

Esto tiene sentido porque:

- Las filas cercanas influyen directamente en el movimiento inmediato
- Las filas lejanas ayudan a anticipar la trayectoria

---

## Estimación de curvatura

Para anticipar curvas utilizo dos puntos característicos de la línea:

- Un punto lejano
- Un punto cercano

La diferencia horizontal entre ambos permite estimar si la trayectoria comienza a girar.

Si la diferencia aumenta significa que el robot se aproxima a una curva.

En ese caso el algoritmo utiliza más información del punto lejano para **anticipar el giro antes de que la curva llegue al robot**.

Esto ayuda a reducir correcciones bruscas.

---

## Control PID

El error lateral se calcula como la diferencia entre el centro de la imagen y la posición estimada de la línea.

Este error se introduce en un **controlador PID**, que genera la velocidad angular del robot.

Cada término del controlador tiene un papel diferente:

- **Proporcional (Kp)** corrige el error actual  
- **Integral (Ki)** compensa errores acumulados  
- **Derivativo (Kd)** suaviza cambios bruscos  

Además, el error se filtra mediante un suavizado exponencial, lo que ayuda a reducir el efecto del ruido en la detección de la línea.

---

## Evolución del controlador

Durante el desarrollo fui probando distintas configuraciones del controlador.

El principal objetivo era encontrar un equilibrio entre:

- Recorrer el circuito lo más rápido posible
- Mantener un movimiento estable sin oscilaciones excesivas

Esto dio lugar a dos versiones principales.

---

## Versión más rápida (control nervioso)

La primera versión del controlador estaba optimizada principalmente para velocidad.

El robot reaccionaba de forma muy agresiva a los cambios de error, lo que permitía mantener velocidades altas en rectas.

Sin embargo, esto generaba oscilaciones constantes en curvas, ya que el robot intentaba corregir continuamente su posición respecto a la línea.

El resultado era un movimiento lateral continuo intentando volver al centro de la trayectoria.

A pesar de esto, el circuito simple se completa aproximadamente en:

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

## Versión final ajustada

Posteriormente ajusté el controlador introduciendo varios cambios:

- Modificación de los umbrales que separan rectas y curvas
- Redistribución de la velocidad según el error lateral
- Limitación de cambios bruscos en la velocidad angular

En la versión final el robot:

- Es más rápido en rectas
- Reduce la velocidad antes en curvas
- Mantiene un movimiento más controlado

Esto reduce las oscilaciones respecto a la versión anterior.

Sin embargo, siguen apareciendo pequeñas oscilaciones, sobretodo en curvas abiertas largas, ya que el controlador sigue siendo puramente reactivo.

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

## Circuito recorrido en sentido inverso

También probé ejecutar el mismo circuito **en sentido contrario**.

Aunque el algoritmo es exactamente el mismo, el comportamiento cambia porque las curvas se afrontan desde el lado opuesto.

Esto hace que algunas curvas que antes eran suaves se vuelvan más difíciles de anticipar.

Como resultado, el robot necesita realizar más correcciones para mantenerse sobre la línea.

El tiempo total del circuito aumenta hasta aproximadamente:

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

## Prueba en circuito Montmeló

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

Este comportamiento se debe principalmente a dos factores:

- La presencia de curvas abiertas muy largas
- La dificultad de anticipar cambios de trayectoria utilizando únicamente un controlador reactivo

En estas curvas el error lateral crece lentamente y el controlador tarda más en reaccionar, lo que provoca oscilaciones hasta que la corrección se vuelve demasiado grande.

Esto muestra una limitación típica de los controladores reactivos basados únicamente en el error lateral.

---

## Modelo Ackermann

El modelo de vehículo utilizado en el simulador sigue una cinemática de tipo **Ackermann**, típica de vehículos con dirección en las ruedas delanteras.

En este modelo las ruedas delanteras no giran con el mismo ángulo, sino que cada una describe una circunferencia distinta para evitar deslizamientos laterales.

Durante la práctica intenté observar este comportamiento en el simulador. Sin embargo, la visualización del modelo Ackermann no funciona correctamente en la interfaz utilizada, por lo que no es posible observar directamente la geometría de giro de las ruedas.

Por este motivo no puedo verificar visualmente cómo se aplica el modelo en el simulador.

Aun así, el comportamiento dinámico del robot sugiere que la cinemática se está aplicando correctamente, ya que el vehículo responde de forma coherente a los comandos de velocidad lineal y angular.

![Modelo Ackermann](/images/ackermandos.png)

---

## Resultados

Los tiempos aproximados obtenidos fueron:

| Versión | Tiempo |
|-------|------|
| Control nervioso | ~58 s |
| Control final ajustado | ~63 s |
| Circuito sentido inverso | ~78 s |

Estos resultados muestran el compromiso clásico entre **velocidad y estabilidad** en sistemas de control.

Un controlador más agresivo permite completar el circuito más rápido, pero introduce mayores oscilaciones y mayor riesgo de salida de la trayectoria.

---

## Conclusión

El seguimiento de línea mediante visión artificial es un problema aparentemente sencillo, pero en la práctica requiere equilibrar varios aspectos:

- percepción visual robusta  
- estimación de trayectoria  
- control dinámico del vehículo  
- ajuste fino de parámetros  

A lo largo de esta práctica pude comprobar cómo pequeños cambios en el controlador producen diferencias significativas en el comportamiento del robot.

Este proyecto me ha permitido explorar la interacción entre visión por computador y control reactivo, dos elementos importantes en visiónn robótica.
