---
layout: post  
title: Seguimiento de línea con visión artificial en fórmula 1  
date: 2025-04-04  
thumbnail: images/chetin.png
excerpt: "Optimización del algoritmo Follow Line con visión y control reactivo"  
---
![Imagen de seguimiento de línea](/images/chetin.png)  <!-- Imagen dentro del post -->

# Seguimiento de línea con visión artificial en fórmula 1

Este informe documenta el desarrollo y optimización de un sistema de control visual para un coche simulado de Fórmula 1. El objetivo de la práctica era permitir que el vehículo siguiera una línea roja trazada sobre el asfalto del circuito, completando una vuelta en el menor tiempo posible sin perder estabilidad ni salirse del recorrido.

Se parte de una versión inicial completamente funcional, pero poco eficiente, y se llega a una versión final que mejora el rendimiento significativamente, logrando reducir el tiempo por vuelta a 63.86 segundos mediante mejoras visuales, de control y de estrategia.

---

## enfoque inicial: versión funcional pero lenta

La primera versión utilizaba una estrategia clásica basada en visión por computador y control PID. El procesamiento visual consistía en convertir la imagen capturada por la cámara del coche a HSV, segmentar la línea roja mediante un doble rango de máscaras y aplicar morfología para eliminar ruido. A continuación, se extraía el contorno más grande y se calculaba su centroide como punto de referencia.

El error se calculaba como la diferencia entre el centro horizontal de la imagen y el valor de `cX`. Este error alimentaba un PID con ganancia proporcional, integral y derivativa, y se aplicaba un suavizado exponencial para evitar oscilaciones. Además, se introdujo una zona muerta para pequeños errores (menos de 3 píxeles).

En cuanto a la velocidad, se regulaba de forma sencilla en función de una estimación de curvatura basada en la diferencia entre los últimos dos valores de `cX`. Se definieron tres rangos: uno para rectas (velocidad alta), uno para curvas suaves y otro para curvas cerradas. El valor máximo alcanzado era de 7.5 unidades.

Aunque el sistema era estable y terminaba el circuito, el tiempo por vuelta rondaba los 85 segundos y mostraba poca capacidad de adaptación a curvas prolongadas o cambios suaves en la trayectoria.

---

## análisis de problemas

Al evaluar el rendimiento, se identificaron varios cuellos de botella:

- La detección del centroide no anticipaba curvas con suficiente antelación.
- El control PID no usaba el tiempo entre iteraciones (`dt`), lo que afectaba a la consistencia.
- La estimación de curvatura era poco robusta.
- El suavizado de dirección era excesivo (`alpha = 0.7`), dificultando maniobras rápidas.
- Las velocidades eran conservadoras, lo que limitaba el rendimiento incluso en tramos rectos.

Estas debilidades motivaron un rediseño más profundo del enfoque.

---

## mejoras aplicadas en la versión final

Tras un proceso iterativo de ajustes y pruebas, se incorporaron mejoras clave en distintas áreas del sistema.

### uso del vértice superior como punto de referencia

En lugar de usar el centroide del contorno, se pasó a utilizar el vértice superior del contorno más grande, es decir, el punto más alto (con menor coordenada Y). Este cambio permitió anticipar mejor las curvas, ya que ese punto representa con mayor fidelidad la dirección futura de la línea. Al utilizar este punto y suavizar su evolución, se logró una respuesta más ágil y coherente del vehículo.

### reducción del suavizado de dirección

El suavizado exponencial aplicado al comando de dirección (`alpha`) se redujo de 0.7 a 0.35. Este cambio permitió al coche reaccionar más rápidamente a cambios en la trayectoria de la línea sin llegar a producir oscilaciones bruscas. Se encontró un equilibrio entre estabilidad y agilidad, crucial en tramos técnicos.

### ajuste más preciso de la curvatura

La curvatura dejó de calcularse como la diferencia entre solo dos puntos de `cX`, y pasó a estimarse a partir de un historial de cinco valores, cuya variación media se usa como referencia. Este enfoque proporciona una medida más estable de cómo evoluciona la línea a lo largo del tiempo, permitiendo decisiones de velocidad más acertadas.

### redefinición de los rangos de velocidad

La lógica de velocidades fue modificada para adaptarse mejor a las características del circuito. Se elevaron los umbrales de curvatura, y se incrementó la velocidad máxima en recta. En concreto, la velocidad se definió en tres niveles:

- En tramos amplios (curvatura < 25), se alcanzan los 11.7.
- En curvas suaves (curvatura < 44), se reduce a 9.6.
- En curvas cerradas, se mantiene en 4.2.

Gracias a esta configuración, el coche es capaz de acelerar intensamente en tramos rectos sin comprometer la estabilidad en las zonas técnicas.

### limitación adaptativa del ángulo de giro

Se añadió una restricción dinámica al ángulo máximo de giro permitido. Cuando la velocidad supera 8 unidades, el ángulo máximo se reduce a 20 grados para evitar zigzags en rectas. En caso contrario, se permite hasta 45 grados, facilitando giros más agresivos. Esto contribuye a mantener la estabilidad a alta velocidad sin perjudicar la maniobrabilidad en curvas lentas.

### limpieza del término integral en errores grandes

Se añadió una condición que reinicia el término integral del PID si el error absoluto supera los 100 píxeles. Esto evita acumulaciones que puedan desestabilizar el sistema en caso de errores momentáneos o reinicios forzados.

---

## resultados comparativos

Las mejoras introducidas han permitido reducir el tiempo por vuelta desde aproximadamente 85 segundos a 63.86 segundos. Además de la mejora en velocidad, se ha mantenido una gran estabilidad, incluso en curvas prolongadas o tras transiciones rápidas. El comportamiento general del coche resulta ahora mucho más competitivo.

| métrica                  | versión inicial     | versión final (opt.)     |
|--------------------------|----------------------|----------------------------|
| Tiempo por vuelta        | ~85 s                | **63.86 s**                |
| Estabilidad general      | Media                | Alta                       |
| Punto de seguimiento     | Centroide            | Vértice superior del contorno |
| Suavizado dirección      | Alpha = 0.7          | Alpha = 0.35               |
| Curvatura                | Δ(cX último - anterior) | Media sobre historial de cX |
| Velocidad máxima         | 7.5                  | 11.7                       |

---

## conclusiones

La evolución del sistema demuestra cómo un control visual reactivo puede mejorar enormemente con ajustes bien enfocados. No ha sido necesario recurrir a técnicas de predicción ni planificación complejas; el rendimiento se ha optimizado únicamente mediante una mejor interpretación visual y un control más preciso y adaptativo.

Este trabajo pone en valor el impacto de detalles como la elección del punto de referencia, la forma de medir la curvatura o la parametrización del suavizado, que son aspectos a menudo subestimados pero con gran efecto sobre el comportamiento final.

---

## vídeo demostrativo

[ver vídeo del resultado en YouTube](https://www.youtube.com/watch?v=AQUI_TU_VIDEO)
