---
layout: post  
title: Mejora de Follow Line (fuera de plazo)
date: 2025-04-04  
thumbnail: images/cheting.png
excerpt: "Optimización del algoritmo Follow Line con visión y control reactivo"  
---
![Imagen de seguimiento de línea](/images/cheting.png)  <!-- Imagen dentro del post -->

# Informe Técnico de optimización de controlador PID para seguimiento de línea

## Resumen

Este documento presenta una comparación detallada entre dos versiones de un controlador PID implementado para la tarea de seguimiento de línea mediante visión por computador. La versión inicial tenía un tiempo de ejecución promedio de 145 segundos, mientras que la versión optimizada ha logrado reducir dicho tiempo a 56.54 segundos, representando una mejora significativa en eficiencia y desempeño.

## Resultados

- **Versión inicial**: 145.00 segundos
- **Versión optimizada**: 56.54 segundos

### Video demostrativo

[![Ver en YouTube](https://img.youtube.com/vi/VIDEO_ID/hqdefault.jpg)](https://youtu.be/df5QsGYy5y0)
---
<h2>Comparación Técnica</h2>

<table style="width:100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="text-align:left; border-bottom: 2px solid #ccc; padding: 6px;">Categoría</th>
      <th style="text-align:left; border-bottom: 2px solid #ccc; padding: 6px;">Versión Inicial</th>
      <th style="text-align:left; border-bottom: 2px solid #ccc; padding: 6px;">Versión Optimizada</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 6px;">Tiempo de ejecución</td>
      <td style="padding: 6px;">145 s</td>
      <td style="padding: 6px;">56.54 s</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Preprocesamiento de imagen</td>
      <td style="padding: 6px;">HSV, median blur, morfología</td>
      <td style="padding: 6px;">Gaussian blur, HSV, ROI inferior</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Área de análisis</td>
      <td style="padding: 6px;">Imagen completa</td>
      <td style="padding: 6px;">Solo mitad inferior (ROI)</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Cálculo de cX</td>
      <td style="padding: 6px;">Centroide del contorno más grande</td>
      <td style="padding: 6px;">Punto más alto del contorno (topmost)</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Estrategia PID</td>
      <td style="padding: 6px;">Parámetros adaptativos según curvatura</td>
      <td style="padding: 6px;">PID con dt real y reinicio de integral</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Control anti-windup</td>
      <td style="padding: 6px;">Límite fijo de la integral</td>
      <td style="padding: 6px;">Reinicio de integral si error &gt; 100</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Suavizado de dirección</td>
      <td style="padding: 6px;">Exponencial (alpha = 0.6)</td>
      <td style="padding: 6px;">Exponencial más responsivo (alpha = 0.15)</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Evaluación de curvatura</td>
      <td style="padding: 6px;">Diferencia entre cX actuales</td>
      <td style="padding: 6px;">Media móvil del historial</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Anticipación a curvas</td>
      <td style="padding: 6px;">No implementada</td>
      <td style="padding: 6px;">Desviación de cX hacia vértice interior</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Control de velocidad</td>
      <td style="padding: 6px;">3 niveles básicos</td>
      <td style="padding: 6px;">Niveles adaptativos según curvatura</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Límite de dirección (W)</td>
      <td style="padding: 6px;">Constante (±35°)</td>
      <td style="padding: 6px;">Dinámico (20°, 35°, 45°)</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Visualización</td>
      <td style="padding: 6px;">Contorno, línea y centroide</td>
      <td style="padding: 6px;">Error, giro, velocidad y centroide ajustado</td>
    </tr>
  </tbody>
</table>

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
