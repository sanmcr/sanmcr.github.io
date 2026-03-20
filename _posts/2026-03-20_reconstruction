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

El objetivo es generar una nube de puntos de la escena en tiempo real a partir de las imágenes izquierda y derecha, aplicando geometría epipolar para encontrar correspondencias y triangulación para obtener la posición en el espacio.

Aunque a nivel teórico el proceso está bastante claro, en la práctica aparecen varios problemas que afectan directamente al resultado:

- La selección de puntos de interés  
- La calidad del matching entre imágenes  
- El ruido introducido por correspondencias incorrectas  
- La falta de textura en algunas superficies  

A lo largo del desarrollo fui ajustando los parámetros del sistema para encontrar un equilibrio entre densidad de puntos y calidad de la reconstrucción.

El resultado final es una reconstrucción bastante consistente en zonas con textura, aunque con limitaciones en regiones más uniformes.

<!--more-->

---

## Enfoque

El sistema sigue tres etapas principales: selección de puntos, búsqueda de correspondencias y reconstrucción 3D.

### Selección de puntos

Se utilizan bordes detectados con Canny sobre la imagen izquierda.  
Para evitar concentraciones de puntos en ciertas zonas, la imagen se divide en una rejilla y se limita el número de puntos por celda.

Además, se aplica un pequeño mecanismo de cooldown para evitar reutilizar siempre los mismos píxeles.

Esto permite mantener una distribución bastante homogénea en toda la imagen.

---

### Búsqueda de correspondencias

Para cada punto seleccionado:

- Se calcula su rayo de proyección desde la cámara izquierda  
- Se restringe la búsqueda en la imagen derecha a una banda horizontal (línea epipolar)  
- Se realiza matching mediante correlación de parches  

El matching se implementa con `matchTemplate` y se filtra mediante:

- Umbral mínimo de confianza  
- Restricción de disparidad  
- Ventana de búsqueda limitada  

Este paso es el más crítico, ya que errores aquí se traducen directamente en puntos mal reconstruidos.

---

### Reconstrucción 3D

Una vez encontrada la correspondencia:

- Se obtienen los rayos de ambas cámaras  
- Se calcula el punto 3D como el punto medio entre los rayos más cercanos  

También se aplica un filtro geométrico para descartar puntos inconsistentes.

El color del punto se obtiene como la media de los píxeles en ambas imágenes.

---

## Resultados

La reconstrucción permite apreciar correctamente la estructura general de la escena:

- Los objetos principales están bien definidos  
- Se distinguen diferentes planos de profundidad  
- Las zonas con textura presentan mayor densidad de puntos  

Sin embargo, aparecen algunas limitaciones:

- Las superficies uniformes generan pocos puntos  
- Existen errores puntuales en el matching  
- Aparece ruido en determinadas zonas  

---

## Conclusiones

La práctica muestra que es posible reconstruir una escena en 3D utilizando únicamente dos cámaras y relaciones geométricas.

El rendimiento del sistema depende principalmente de:

- La calidad del matching  
- La selección de puntos de interés  
- El equilibrio entre cantidad de puntos y filtrado  

En general, se obtiene una reconstrucción coherente sin necesidad de técnicas complejas, aunque con limitaciones inherentes al método.

---

## Demo

Se incluye un vídeo mostrando el proceso de reconstrucción en tiempo real y el resultado final.
