---
layout: post
title: "Reconstrucción 3D – Unibotics"
date: 2026-03-20
thumbnail: "/images/3d.png"
excerpt: "Reconstrucción 3D de una escena a partir de un par estéreo utilizando geometría epipolar, matching y triangulación en tiempo real."
published: true
---

![Reconstrucción 3D](/images/3d.png)

En esta práctica he desarrollado un sistema de reconstrucción 3D a partir de dos cámaras utilizando la plataforma Unibotics.

El objetivo es generar una nube de puntos de la escena en tiempo real a partir de las imágenes izquierda y derecha, utilizando geometría epipolar para encontrar correspondencias y triangulación para obtener la posición en el espacio.

Aunque a nivel teórico el proceso es bastante directo, en la práctica me he encontrado con varios problemas que afectan mucho al resultado:

- La selección de puntos de interés  
- La calidad del matching entre imágenes  
- El ruido generado por correspondencias incorrectas  
- La falta de textura en algunas superficies  

A lo largo del desarrollo he ido ajustando distintos parámetros para encontrar un equilibrio entre densidad de puntos y calidad de la reconstrucción.

El resultado final es una reconstrucción bastante consistente en zonas con textura, aunque con limitaciones en regiones más uniformes.

<!--more-->

---

## Enfoque

He estructurado el sistema en tres etapas principales: selección de puntos, búsqueda de correspondencias y reconstrucción 3D.

---

### Selección de puntos

Para detectar puntos relevantes utilizo Canny sobre la imagen izquierda.

Para evitar que todos los puntos se concentren en la misma zona:

- Divido la imagen en una rejilla  
- Limito el número de puntos por celda  
- Aplico un pequeño cooldown para no reutilizar siempre los mismos píxeles  

Esto me permite mantener una distribución bastante homogénea de puntos en toda la imagen.

---

### Búsqueda de correspondencias

Para cada punto seleccionado:

- Calculo su rayo de proyección desde la cámara izquierda  
- Restrinjo la búsqueda en la imagen derecha a una banda horizontal (línea epipolar)  
- Busco la mejor correspondencia mediante correlación de parches  

El matching lo implemento con `matchTemplate`, y filtro resultados usando:

- Un umbral mínimo de confianza  
- Restricciones de disparidad  
- Una ventana de búsqueda limitada  

Este es claramente el paso más crítico del sistema: si el matching falla, la reconstrucción se degrada mucho.

---

### Reconstrucción 3D

Una vez tengo la correspondencia:

- Obtengo los rayos de ambas cámaras  
- Calculo el punto 3D como el punto medio entre los rayos más cercanos  

Además, aplico un filtro geométrico para descartar puntos inconsistentes.

El color del punto lo obtengo como la media de los píxeles en ambas imágenes.

---

## Resultados

La reconstrucción permite apreciar bastante bien la estructura de la escena:

- Los objetos principales están bien definidos  
- Se distinguen diferentes planos de profundidad  
- Las zonas con textura presentan mayor densidad de puntos  

Sin embargo, también aparecen algunas limitaciones:

- Las superficies uniformes generan pocos puntos  
- Hay errores puntuales en el matching  
- Aparece ruido en algunas zonas  

---

## Conclusiones

Esta práctica me ha servido para entender bien cómo funciona la reconstrucción 3D a partir de visión estéreo.

He visto que:

- La geometría epipolar reduce mucho el problema  
- El matching es el factor más crítico  
- La selección de puntos influye directamente en el resultado  

Al final, todo consiste en encontrar un equilibrio entre cantidad de puntos y calidad de la reconstrucción.

El sistema consigue una reconstrucción bastante coherente sin necesidad de técnicas complejas, aunque con limitaciones propias del enfoque.

---

## Demo

Incluyo un vídeo donde se puede ver el proceso de reconstrucción en tiempo real y el resultado final.

<div style="text-align:center;">
<iframe width="700" height="394"
src="[https://www.youtube.com/embed/6KSY_JKuLIg](https://youtu.be/Wdp6Zce26XQ)"
title="Control nervioso"
frameborder="0"
allowfullscreen>
</iframe>
</div>

