---
layout: post  
title: Control visual de un robot de Fórmula 1 (fuera de plazo)  
date: 2025-45-04  
thumbnail: images/p3.jpg  
excerpt: "Versión mejorada de Follow Line"  
---

# Seguimiento de línea en un coche de Fórmula 1 simulado

En el marco de la asignatura Visión en Robótica del Máster en Visión Artificial, se ha desarrollado una práctica cuyo objetivo es permitir que un coche de Fórmula 1 simulado sea capaz de seguir de forma autónoma una línea roja trazada sobre el asfalto de un circuito virtual. La tarea consiste en aplicar técnicas de visión por computador y control reactivo para que el vehículo complete una vuelta al circuito en el menor tiempo posible, sin salirse de la pista ni colisionar con los bordes del trazado.

El sistema se ha desarrollado de forma incremental, comenzando con un enfoque sencillo y funcional, que fue posteriormente mejorado y optimizado para alcanzar un comportamiento mucho más eficiente, estable y competitivo. En este informe se describen en detalle ambas versiones del sistema, las mejoras introducidas, los fundamentos técnicos y los resultados obtenidos.

## Versión inicial: implementación básica

En una primera etapa, se implementó un sistema de seguimiento de línea basado en la detección de contornos y el cálculo del centroide del contorno más grande detectado en la imagen binaria resultante de segmentar el color rojo. Esta imagen se obtenía tras convertir la imagen RGB de la cámara frontal del coche al espacio de color HSV, que es más robusto frente a cambios de iluminación.

Una vez detectada la línea roja, se encontraba su centroide (coordenada `cX`) y se calculaba el error como la diferencia entre `cX` y el centro horizontal de la imagen. Este error alimentaba un controlador PID que generaba el comando de giro. Para evitar oscilaciones, se implementó un suavizado exponencial con un factor `alpha` de 0.7. La velocidad del coche se regulaba en función de la curvatura estimada, que se aproximaba como la diferencia entre los últimos dos valores de `cX` almacenados.

El comportamiento resultante era funcional, permitiendo que el coche siguiera la línea y completara una vuelta al circuito. Sin embargo, el sistema presentaba diversas limitaciones. En particular, la anticipación de curvas era deficiente, la velocidad máxima alcanzada era baja (7.5 unidades en tramos rectos), y el control de dirección era excesivamente lento, lo que resultaba en un tiempo de vuelta elevado, superior a 80 segundos.

## Optimizaciones introducidas: versión mejorada

A partir del análisis de las carencias observadas, se introdujeron diversas mejoras que transformaron significativamente el rendimiento del sistema. Estas mejoras se centraron en los aspectos de detección visual, cálculo del error, estimación de curvatura, ajuste de la velocidad y refinamiento del control PID.

### Cambio del punto de referencia visual

En lugar de usar el centroide del contorno, se pasó a utilizar el vértice superior del contorno rojo, es decir, el punto más elevado en la imagen correspondiente a la línea roja. Este punto proporciona una estimación más adelantada de la trayectoria, permitiendo anticipar cambios de dirección antes de que el vehículo llegue a ellos. Gracias a este cambio, el sistema adquirió una capacidad predictiva más efectiva, especialmente útil en curvas suaves y prolongadas.

### Cálculo real del PID con delta temporal

Se incorporó el cálculo del tiempo entre iteraciones (`dt`), lo que permitió aplicar un PID realista y más eficaz. El término derivativo pudo así calcularse como la derivada del error respecto al tiempo, lo que resultó en un control de giro más suave y menos propenso a oscilaciones, especialmente en transiciones rápidas.

### Estimación de curvatura basada en historial

La curvatura de la trayectoria se pasó a estimar mediante un historial de posiciones del punto de referencia (`cX`), permitiendo obtener una media de las diferencias acumuladas. Esta curvatura suavizada proporcionó una estimación mucho más estable y representativa de la trayectoria real, lo cual mejoró la adaptación de la velocidad en función de la geometría de la pista.

### Mjora del suavizado de giro

El valor del coeficiente de suavizado exponencial se redujo de 0.7 a 0.35, lo cual hizo que el sistema fuera más reactivo a cambios en la trayectoria. Este cambio permitió una mejor respuesta en curvas cerradas, reduciendo la tendencia a sobrecorregir y facilitando transiciones más precisas.

### Control dinámico de la velocidad

En lugar de utilizar tres velocidades fijas, se estableció una relación más detallada entre la curvatura estimada y la velocidad máxima permitida. Así, en tramos rectos el coche podía alcanzar hasta 11.7 unidades, mientras que en curvas suaves se reducía a 9.6 y en curvas cerradas hasta 4.0. Esta adaptación progresiva permitió aumentar considerablemente la velocidad media sin comprometer la estabilidad.

### Límite dinámico del ángulo de giro

Se implementó un sistema de limitación del ángulo de giro máximo, dependiente de la velocidad. A velocidades altas, el giro se limitaba a 20 grados para evitar movimientos bruscos, mientras que en curvas lentas se permitía un giro de hasta 45 grados. Esto evitó el zigzagueo en rectas y mejoró la estabilidad general.

## Resultados y comparativa

Con todas las mejoras introducidas, el sistema consiguió completar una vuelta al circuito en aproximadamente 63.8 segundos, reduciendo en más de 20 segundos el tiempo obtenido con la versión inicial. Además, la estabilidad del vehículo aumentó notablemente, tanto en tramos rectos como en curvas cerradas, y la respuesta ante cambios de dirección fue mucho más natural.

| métrica                  | versión inicial     | versión optimizada     |
|--------------------------|----------------------|--------------------------|
| Tiempo de vuelta         | ~80–85 segundos      | ~63.8 segundos           |
| Estabilidad en curvas    | Regular              | Alta                     |
| Velocidad máxima         | 7.5                  | 11.7                     |
| Punto de referencia      | Centroide            | Vértice superior         |
| Cálculo de PID con dt    | No                   | Sí                       |

## Conclusión

La práctica de seguimiento de línea mediante visión ha permitido implementar y afinar un sistema de control reactivo eficiente. Gracias a la mejora en la selección del punto de referencia, el uso de un PID realista, la adaptación de la velocidad a la curvatura y el refinamiento del control de giro, el vehículo ha demostrado ser capaz de seguir la línea roja con una gran precisión y velocidad.

Este trabajo evidencia que, con un tratamiento cuidadoso de la información visual y un diseño adecuado del sistema de control, es posible alcanzar un comportamiento muy competitivo sin necesidad de estrategias complejas de planificación o aprendizaje.

## vídeo demostrativo

[Ver vídeo del resultado en YouTube](https://www.youtube.com/watch?v=AQUI_TU_VIDEO)
