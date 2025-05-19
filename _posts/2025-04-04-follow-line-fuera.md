---
layout: post  
title: Seguimiento de línea con visión artificial en fórmula 1  
date: 2025-04-04  
thumbnail: images/cheting.png
excerpt: "Optimización del algoritmo Follow Line con visión y control reactivo"  
---
![Imagen de seguimiento de línea](/images/cheting.png)  <!-- Imagen dentro del post -->

# Informe Técnico de Optimización de Controlador PID para Seguimiento de Línea

## Resumen

Este documento presenta una comparación detallada entre dos versiones de un controlador PID implementado para la tarea de seguimiento de línea mediante visión por computador. La versión inicial tenía un tiempo de ejecución promedio de 145 segundos, mientras que la versión optimizada ha logrado reducir dicho tiempo a 56.54 segundos, representando una mejora significativa en eficiencia y desempeño.

## Resultados

- **Versión inicial**: 145.00 segundos
- **Versión optimizada**: 56.54 segundos

> [Enlace al video demostrativo](https://www.youtube.com/insertar_enlace_aqui)

---

## Comparación Técnica

| Categoría                      | Versión Inicial                                   | Versión Optimizada                                    |
|-------------------------------|----------------------------------------------------|--------------------------------------------------------|
| Tiempo de ejecución           | 145 s                                              | 56.54 s                                                |
| Preprocesamiento de imagen    | Conversión HSV, median blur, morfología            | Gaussian blur, conversión HSV, ROI en zona inferior    |
| Área de análisis              | Imagen completa                                    | Solo mitad inferior (ROI)                             |
| Cálculo de cX                 | Centroide del contorno más grande                  | Punto más alto del contorno más grande (topmost)       |
| Estrategia PID                | Parámetros adaptativos según curvatura             | PID con cálculo de derivada e integral por tiempo real |
| Control anti-windup           | Límite fijo de acumulación de error                | Reinicio de integral si error > 100                    |
| Suavizado de dirección        | Suavizado exponencial (alpha = 0.6)                | Suavizado más responsivo (alpha = 0.15)               |
| Evaluación de curvatura       | Diferencia entre valores extremos de cX            | Media móvil con historial de curvatura                |
| Anticipación a curvas         | No implementada                                    | Desviación de cX hacia vértice interior en curvas     |
| Control de velocidad          | 3 niveles simples                                  | Niveles agresivos y adaptativos según curvatura       |
| Límite de dirección (W)       | Constante (±35°)                                   | Dinámico (20°, 35°, 45°) según curvatura               |
| Visualización en pantalla     | Contorno, centroide, línea de error                | Contorno, cX ajustado, error, giro y velocidad         |

---

## Principales Mejoras Aplicadas

1. **Reducción del área procesada**  
   Se limita el análisis a la mitad inferior de la imagen, donde se encuentra la línea de interés, lo que disminuye considerablemente el procesamiento innecesario.

2. **Referencia del punto de control más precisa**  
   Se utiliza el punto superior del contorno (topmost) como referencia para calcular el error lateral, permitiendo una mejor anticipación de la trayectoria.

3. **PID más eficiente**  
   - El uso de `dt` real mejora el cálculo del término derivativo.
   - Se implementa un mecanismo para reiniciar la acumulación de error (integral) en casos de desviación excesiva.

4. **Suavizado más reactivo**  
   Un suavizado exponencial con menor peso histórico (alpha = 0.15) permite que el sistema responda más rápidamente a cambios repentinos.

5. **Anticipación de curvas cerradas**  
   Se calcula la dirección de la curva y se ajusta el punto de referencia (`cX`) hacia el vértice interior, permitiendo un mejor trazado en curvas pronunciadas.

6. **Control adaptativo de velocidad y dirección**  
   Tanto la velocidad como el límite del ángulo de giro se ajustan dinámicamente en función de la curvatura detectada, permitiendo una conducción más agresiva y eficiente en tramos rectos y segura en curvas.

---

## Conclusión

La versión optimizada del sistema ha logrado mejorar de forma notable el desempeño general del seguimiento de línea. Las mejoras no solo reducen significativamente el tiempo total de recorrido, sino que también aportan mayor estabilidad, precisión y adaptabilidad del controlador PID ante escenarios variables.

El conjunto de optimizaciones implementadas permiten considerar esta versión como una solución más robusta y eficiente para entornos de navegación autónoma mediante visión artificial.

[Enlace al video demostrativo](https://youtu.be/df5QsGYy5y0)
