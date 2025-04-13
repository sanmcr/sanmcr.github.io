---
layout: post
title: "3D Reconstruction"
date: 2025-04-13
thumbnail: "images/muñequin.png"
excerpt: "Reconstrucción tridimensional de escenas."
---
![Imagen de seguimiento de línea](/images/muñequin.png)  <!-- Imagen dentro del post -->

Este proyecto implementa una reconstrucción 3D utilizando visión estereoscópica, donde se calcula la profundidad de una escena a partir del emparejamiento de puntos detectados en imágenes captadas desde dos cámaras.

# Informe: title: "Reconstrucción 3D con visión estereoscópica en Python"


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
```

Estas imágenes se usan como entrada para la siguiente fase del proceso.

## Capturas de las cámaras

Estas son las imágenes capturadas por las cámaras del robot Kobuki, que se usan para la reconstrucción 3D:

<img src="/images/camaras.png" alt="Captura de la cámara izquierda y derecha" style="display: block; margin-left: auto; margin-right: auto; width: 80%;"/>


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

### 3. Cálculo de la línea epipolar

La geometría epipolar nos permite reducir la búsqueda de correspondencias de puntos entre las dos imágenes. Dado un punto en la imagen izquierda, la línea epipolar en la imagen derecha es la única línea donde podemos encontrar el punto correspondiente.

Para calcular la línea epipolar, primero necesitamos obtener el vector de proyección 3D de un punto en la imagen izquierda. Este vector se calcula mediante la función `getProjectionLine()`, que toma el centro de la cámara y el punto en la imagen y lo convierte en un vector en el espacio 3D:

```python
def getProjectionLine(camera_optical_center, pxl, side):
    new_pxl = [pxl[1], pxl[0], 1]  # Convertir el punto a coordenadas de imagen
    cam_2d_point = HAL.graficToOptical(side, new_pxl)  # Transformar a coordenadas ópticas
    pt_3d = HAL.backproject(side, cam_2d_point)  # Proyectar el punto 2D a 3D
    projection_vector = pt_3d[:3] - camera_optical_center  # Calcular el vector de proyección
    return projection_vector
```

Luego, utilizando la proyección 3D, calculamos la línea epipolar en la imagen derecha.


### 4. Emparejamiento de puntos entre las imágenes

Una vez calculadas las líneas epipolares, el siguiente paso es buscar la correspondencia de los puntos entre las imágenes izquierda y derecha. Se utiliza la función `cv2.matchTemplate()` para realizar la correlación entre un bloque de la imagen izquierda (que corresponde a un punto de interés) y una región de la imagen derecha:

```python
match = cv2.matchTemplate(croped, template, cv2.TM_CCOEFF_NORMED)
```

El resultado de esta operación es una matriz de correlación que nos indica qué tan bien coincide la región de la imagen izquierda con la región de la imagen derecha. El punto con la mayor correlación será considerado como el punto correspondiente.

### 5. Triangulación 3D

Con los puntos correspondientes en ambas imágenes, podemos proceder a calcular las coordenadas 3D de los puntos utilizando la técnica de triangulación. En la triangulación, se utilizan las proyecciones de los puntos en ambas cámaras para determinar su ubicación en el espacio 3D.

```python
m, c, _ = np.linalg.lstsq(A.T, b, rcond=None)[0]  # Resolver el sistema de ecuaciones para la triangulación
pt_3d = (m * l_projection_vector) + ((c / 2) * n)
```

### 6. Visualización en el visor 3D
Finalmente, los puntos reconstruidos se visualizan en un visor 3D. Esto se realiza mediante la función GUI.ShowNewPoints(), que acepta un conjunto de puntos en formato [x, y, z, R, G, B]:

```
point = drawPoint(pxl, match, l_cam_pos, r_cam_pos, l_img, r_img, l_projection_vector)
GUI.ShowNewPoints([point])
```

Esto permite ver la reconstrucción 3D de la escena en tiempo real.

## Implementación y ejecución

El código fue desarrollado en Python y ejecutado en el entorno de simulación Unibotics, utilizando funciones proporcionadas por las interfaces HAL (Hardware Abstraction Layer) y GUI para la captura, procesamiento y visualización de datos. La reconstrucción se basa en emparejamientos válidos con umbral de correlación > 0.80 y utiliza triangulación para calcular posiciones 3D de los puntos detectados.


## Resultados

Durante la ejecución del algoritmo, el sistema es capaz de detectar puntos de interés en la imagen izquierda y encontrar sus correspondencias en la imagen derecha utilizando las líneas epipolares. A partir de estas correspondencias válidas (con una correlación mayor a 0.80), se realiza una triangulación para estimar la posición de cada punto en el espacio 3D.

En la siguiente imagen se muestra una representación 2D de los resultados obtenidos. En ella se puede ver claramente cómo se han detectado los contornos de los objetos de la escena, y cómo los puntos reconstruidos se visualizan superpuestos en azul. Esta imagen no muestra la nube de puntos 3D como tal, sino una proyección de los puntos sobre la vista de ambas cámaras, lo cual permite verificar visualmente la precisión del emparejamiento y de la reconstrucción:

<img src="/images/2d.png" alt="Reconstrucción 2D con puntos azules" style="width: 80%; display: block; margin: auto;" />

Puede observarse que los puntos en azul siguen los bordes de los objetos principales de la escena, como los personajes, los cubos de letras y el patito de goma. Esto indica que el sistema ha sido capaz de identificar correctamente las zonas con mayor información visual y realizar una reconstrucción precisa en esas regiones.

Este resultado visual es especialmente útil para validar de forma cualitativa el comportamiento del algoritmo: una buena alineación de los puntos azules sobre los objetos originales confirma que el sistema ha encontrado correspondencias correctas y que la triangulación ha sido coherente. Por el contrario, si los puntos azules estuvieran dispersos o mal alineados, indicaría posibles errores en el emparejamiento o ruido en los datos.

Además, esta representación previa a la nube de puntos 3D final también permite comparar cómo se distribuyen los puntos reconstruidos desde la cámara izquierda y la derecha cuando se realiza una reconstrucción bidireccional.

## Desafíos y Consideraciones

Algunos de los principales desafíos que se enfrentaron durante el desarrollo de este ejercicio fueron:

- **Correspondencia de puntos**: El algoritmo de `cv2.matchTemplate()` es sensible a las condiciones de iluminación y a las variaciones de los puntos entre las imágenes izquierda y derecha. Las condiciones ideales son aquellas en las que las imágenes están bien alineadas y con buena iluminación.
  
- **Precisión de la triangulación**: Los errores en la correspondencia de los puntos pueden llevar a una triangulación incorrecta, lo que afecta la precisión de la reconstrucción.

- **Tiempo de procesamiento**: A medida que aumenta el número de puntos a reconstruir, el tiempo de procesamiento también aumenta. Esto puede ser un factor limitante si se desea procesar grandes cantidades de puntos en tiempo real.

## Conclusiones

La reconstrucción 3D utilizando visión estereoscópica es un proceso complejo que involucra varios pasos, como la detección de puntos, el emparejamiento de correspondencias y la triangulación. El algoritmo desarrollado fue capaz de reconstruir puntos 3D de la escena con una precisión razonable, aunque se pueden mejorar algunos aspectos, como la precisión del emparejamiento de puntos y la optimización del tiempo de procesamiento.

Para futuras mejoras, se recomienda implementar métodos más avanzados de detección de características y correspondencias, así como técnicas de calibración para corregir posibles distorsiones en las imágenes.

En general, la práctica me permitió comprender mejor cómo se combinan conceptos geométricos, visión por computadora y estructuras 3D, y cómo influyen parámetros como la confianza en la calidad de la reconstrucción.

## Video de la reconstrucción 3D

Para visualizar el proceso de la reconstrucción en tiempo real, se ha subido un video a YouTube. En él, se muestra cómo se reconstruyen los puntos y cómo se visualizan en el visor 3D.

> En el siguiente video se aprecia cómo se visualiza progresivamente la reconstrucción de la escena en el visor 3D.


<iframe width="560" height="315" src="https://www.youtube.com/embed/EQguswWDk90" frameborder="0" allowfullscreen></iframe>
