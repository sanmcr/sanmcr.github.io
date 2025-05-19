---
layout: post
title: Marker Based Visual Loc
date: 2025-05-17
thumbnail: images/p3.jpg
excerpt: "Marker Based Visual Loc"
---
![Imagen de seguimiento de línea](/images/p3.jpg)  <!-- Imagen dentro del post -->

# Marker Based Visual Loc

## Objetivo

El objetivo del proyecto es realizar la autolocalización de un robot en un entorno 2D mediante la detección de marcadores visuales (AprilTags) y la estimación de su posición y orientación a partir de ellos. Para ello se ha implementado un sistema robusto que combina visión artificial, transformaciones geométricas y odometría como respaldo.

---

## Inicialización

El robot inicia capturando imágenes de la cámara frontal y usando la librería `pyapriltags` para detectar los AprilTags presentes en la escena. La información de posición real de los tags se carga desde un archivo YAML proporcionado por el simulador:

```python
config = yaml.safe_load(Path("/resources/exercises/marker_visual_loc/apriltags_poses.yaml").read_text())
tags_info = config["tags"]
```

A partir de esta información, se realiza el pipeline completo de transformaciones para pasar de coordenadas de la imagen al sistema global del robot.


## Estimación de la Pose

La posición del robot respecto al mundo se obtiene aplicando una serie de transformaciones:

### `world2tag`
Matriz de transformación del mundo al tag, obtenida del archivo YAML. Contiene la posición y orientación conocidas de cada AprilTag.

### `tag2tag_optical`
Rotaciones necesarias para pasar del sistema de coordenadas del AprilTag al sistema de coordenadas ópticas de la cámara.

### `solvePnP()` (OpenCV)
A partir de los puntos 3D (coordenadas del tag) y sus proyecciones 2D en la imagen, se obtiene la transformación `cam_optical2tag_optical`, que se invierte para obtener la posición de la cámara con respecto al tag.

### `cam_optical2cam`
Matriz inversa de `tag2tag_optical` que permite transformar del sistema óptico al sistema físico de la cámara.

### `cam2robot`
Transformación fija entre el sistema de coordenadas de la cámara y el centro del robot, obtenida a partir del modelo del robot (archivo SDF).

### `world2robot`
Composición de todas las transformaciones anteriores. A partir de esta matriz se extraen las coordenadas `x`, `y` y el ángulo `yaw` que representan la pose del robot en el mundo.

```python
yaw = math.atan2(world2robot[1, 0], world2robot[0, 0])
```

## Composición de Transformaciones

La estimación de la pose del robot se basa en una serie de transformaciones encadenadas, que permiten convertir coordenadas del sistema visual (imagen) al sistema global del entorno.

### Tabla resumen

| Transformación               | Descripción                                                      |
|-----------------------------|------------------------------------------------------------------|
| `world2tag`                 | Pose conocida del tag en el mundo (extraída del archivo YAML)   |
| `tag2tag_optical`           | Rotación del marco del tag al marco óptico de la cámara          |
| `tag_optical2cam_optical`   | Inversa de la pose obtenida con `solvePnP()`                     |
| `cam_optical2cam`           | Conversión del sistema óptico al sistema físico de la cámara     |
| `cam2robot`                 | Desplazamiento entre la cámara y el centro del robot             |
| `world2robot`               | Resultado final: estimación de posición y orientación del robot  |

### Composición completa

La matriz `world2robot` se calcula aplicando la siguiente secuencia de transformaciones:

world2robot = world2tag × tag2tag_optical × tag_optical2cam_optical × cam_optical2cam × cam2robot


A partir de esta matriz, se obtiene:

- `x`, `y`: posición estimada del robot
- `yaw`: orientación estimada del robot

```python
yaw = math.atan2(world2robot[1, 0], world2robot[0, 0])
```

## Selección del AprilTag

Si se detectan múltiples AprilTags, se selecciona el que esté más cerca de la última posición estimada del robot.  
Esta decisión se toma para mejorar la fiabilidad de la estimación, ya que los tags lejanos tienden a introducir mayor error debido a la distorsión de perspectiva y al ruido visual.

---

## Odometría

Cuando no se detecta ningún AprilTag, se activa la estimación de pose por odometría:

- Se utilizan los datos de posición y orientación del robot proporcionados por `HAL.getOdom()`.
- Se implementa una lógica de búsqueda activa: el robot gira mientras avanza lentamente, con el objetivo de recuperar la visión de algún tag en su entorno.

```python
HAL.setV(0.1)
HAL.setW(0.4 if last_tag_side == 'left' else -0.4)
```
## Giro Adaptativo Inteligente

Una mejora clave del sistema es que, si el robot no detecta ningún AprilTag durante un tiempo prolongado (por ejemplo, 90 segundos), cambia automáticamente la dirección de giro para evitar quedarse atascado.

Esto se implementa mediante un contador llamado `no_tag_counter` que se incrementa en cada iteración del bucle principal.  
Cuando supera un umbral (`no_tag_limit = 900`), se invierte el valor de `last_tag_side` para modificar la dirección de búsqueda.


```python
if no_tag_counter >= no_tag_limit:
    last_tag_side = 'right' if last_tag_side == 'left' else 'left'
    no_tag_counter = 0
```


## Resultados

El sistema desarrollado permite:

- Detectar y seguir AprilTags de forma visual.
- Corregir automáticamente la orientación del robot hacia el tag más cercano.
- Estimar con precisión su posición y orientación en el mapa.
- Continuar navegando de forma coherente mediante odometría cuando no hay visión disponible.
- Cambiar dinámicamente la estrategia de búsqueda tras un periodo prolongado sin detección.

---

## Demostración en Vídeo

En el siguiente vídeo se puede observar el comportamiento final del sistema en el entorno simulado de Unibotics:

**https://www.youtube.com/watch?v=0MP5zJXQh24&t=3s&ab_channel=SandraMontejanoC%C3%A1novas**
