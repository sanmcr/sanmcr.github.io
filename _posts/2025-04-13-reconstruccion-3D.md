---
layout: post
title: "3D Reconstruction"
date: 2025-04-13
thumbnail: "images/muñequin.png"
excerpt: "Un resumen de lo que trata el post."
---
![Imagen de seguimiento de línea](/images/muñequin.png)  <!-- Imagen dentro del post -->


# Informe: Reconstrucción 3D utilizando visión estereoscópica

## Introducción

La reconstrucción 3D es una de las áreas más fundamentales en visión por computadora y robótica, pues permite obtener la representación tridimensional de una escena a partir de dos o más imágenes tomadas desde diferentes puntos de vista. En este informe, se detalla el proceso de implementación de un algoritmo para realizar una reconstrucción 3D utilizando dos cámaras (izquierda y derecha) montadas en un robot Kobuki. El objetivo principal es usar la geometría epipolar para detectar puntos correspondientes en las imágenes y luego calcular sus coordenadas en el espacio 3D.

## Objetivos

El objetivo de esta práctica es desarrollar un sistema capaz de generar una reconstrucción 3D de la escena visualizada por el robot a través de sus cámaras. Para ello, se utilizará la visión estereoscópica, que se basa en las siguientes etapas:

1. **Captura de imágenes de dos cámaras**: Obtener las imágenes de la cámara izquierda y la cámara derecha.
2. **Detección de puntos de características**: Identificar puntos característicos en la imagen izquierda que luego serán emparejados con sus correspondientes en la imagen derecha.
3. **Cálculo de la línea epipolar**: Utilizar la geometría epipolar para limitar la búsqueda de puntos correspondientes en la imagen derecha a una línea específica.
4. **Triangulación 3D**: Utilizar los puntos correspondientes en ambas imágenes para calcular su posición en el espacio 3D.
5. **Visualización en el visor 3D**: Representar los puntos reconstruidos en un visor 3D para verificar el resultado.

## Metodología

### 1. Captura de imágenes

El primer paso en el proceso de reconstrucción 3D es la captura de las imágenes que se tomarán desde las dos cámaras del robot. Se utilizan las siguientes funciones de la API para acceder a las imágenes:

```python
l_img = HAL.getImage('left')  # Obtener imagen de la cámara izquierda
r_img = HAL.getImage('right')  # Obtener imagen de la cámara derecha
``

Estas imágenes se usan como entrada para la siguiente fase del proceso.

### 2. Detección de puntos de características

El siguiente paso es identificar los puntos de interés en las imágenes. Para este ejercicio, se usa el algoritmo de detección de bordes **Canny** (una técnica común para detectar contornos en imágenes) para detectar las características que se utilizarán en la reconstrucción. Los puntos detectados en la imagen izquierda se considerarán los puntos de interés.

El código utilizado para realizar la detección de bordes es el siguiente:

```python
img = cv2.Canny(l_img, 100, 200)  # Detección de bordes en la imagen izquierda
```

Esto genera una nueva imagen binaria donde los bordes de la escena están representados por píxeles blancos (valor 255) y el resto de la imagen está en negro (valor 0). A partir de esta imagen, se extraen los puntos de interés (píxeles blancos) y se guardan en la lista white_pixels.


```python
white_pixels = []
height = img.shape[0]
width = img.shape[1]
for x in range(width):
    for y in range(height):
        if img[y][x] == 255:  # Si el píxel es blanco (borde detectado)
            white_pixels.append([x, y])  # Añadir el punto a la lista
```

## 3. Cálculo de la línea epipolar

La geometría epipolar nos permite reducir la búsqueda de correspondencias de puntos entre las dos imágenes. Dado un punto en la imagen izquierda, la línea epipolar en la imagen derecha es la única línea donde podemos encontrar el punto correspondiente.

Para calcular la línea epipolar, primero necesitamos obtener el vector de proyección 3D de un punto en la imagen izquierda. Este vector se calcula mediante la función `getProjectionLine()`, que toma el centro de la cámara y el punto en la imagen y lo convierte en un vector en el espacio 3D:

```python
def getProjectionLine(camera_optical_center, pxl, side):
    new_pxl = [pxl[1], pxl[0], 1]  # Convertir el punto a coordenadas de imagen
    cam_2d_point = HAL.graficToOptical(side, new_pxl)  # Transformar a coordenadas ópticas
    pt_3d = HAL.backproject(side, cam_2d_point)  # Proyectar el punto 2D a 3D
    projection_vector = pt_3d[:3] - camera_optical_center  # Calcular el vector de proyección
    return projection_vector
``

Luego, utilizando la proyección 3D, calculamos la línea epipolar en la imagen derecha.


