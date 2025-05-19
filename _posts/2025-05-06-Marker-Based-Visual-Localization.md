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

## Estimación de la pose

