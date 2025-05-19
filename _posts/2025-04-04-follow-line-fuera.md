---
layout: post  
title: Mejora de Follow Line (fuera de plazo)
date: 2025-04-04  
thumbnail: images/cheting.png
excerpt: "Optimización del algoritmo Follow Line con visión y control reactivo"  
---
![Imagen de seguimiento de línea](/images/cheting.png)  <!-- Imagen dentro del post -->

# Informe Técnico de optimización de controlador PID para seguimiento de línea

---

# Circuito Simple
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


# Circuito Ackerman


Tras optimizar el comportamiento del sistema en el circuito simple, se realizó una adaptación específica del controlador PID para el circuito tipo **Ackermann**, que presenta características diferentes: curvas más amplias, menores radios de giro y necesidad de anticipación en la trayectoria.

El objetivo fue mantener la estabilidad de navegación y reducir el tiempo de recorrido frente a la versión original.

---

### Resultados

- **Versión inicial**: aproximadamente 10 minutos (600 segundos)
- **Versión optimizada**: 324.16 segundos

> [![Ver en YouTube](https://img.youtube.com/vi/VIDEO_ID/hqdefault.jpg)](https://youtu.be/NRz4K7V092s)  
> *Reemplazar `VIDEO_ID` por el ID real del video en YouTube*

---

### Comparación Técnica

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
      <td style="padding: 6px;">~600 s</td>
      <td style="padding: 6px;">324.16 s</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Preprocesamiento de imagen</td>
      <td style="padding: 6px;">HSV, median blur, morfología</td>
      <td style="padding: 6px;">Gaussian blur, HSV, ROI inferior</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Área de análisis</td>
      <td style="padding: 6px;">Imagen completa</td>
      <td style="padding: 6px;">Mitad inferior de la imagen (ROI)</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Punto de referencia (cX)</td>
      <td style="padding: 6px;">Centroide del contorno</td>
      <td style="padding: 6px;">Punto más alto del contorno (topmost)</td>
    </tr>
    <tr>
      <td style="padding: 6px;">PID</td>
      <td style="padding: 6px;">Ajustes proporcionales al error</td>
      <td style="padding: 6px;">PID con `dt` real y derivada suavizada</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Suavizado de dirección</td>
      <td style="padding: 6px;">Alpha = 0.6</td>
      <td style="padding: 6px;">Alpha_w = 0.3</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Anticipación a curvas</td>
      <td style="padding: 6px;">No implementada</td>
      <td style="padding: 6px;">Ajuste de `cX` hacia vértice interior</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Suavizado de velocidad</td>
      <td style="padding: 6px;">Discreto según error</td>
      <td style="padding: 6px;">Cambio progresivo con `dv_max`</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Evaluación de curvatura</td>
      <td style="padding: 6px;">No considerada</td>
      <td style="padding: 6px;">Media móvil del historial de `cX`</td>
    </tr>
    <tr>
      <td style="padding: 6px;">Visualización</td>
      <td style="padding: 6px;">Contorno, línea de error</td>
      <td style="padding: 6px;">Error, velocidad, giro y punto topmost</td>
    </tr>
  </tbody>
</table>

---

### Mejoras Implementadas

1. **Procesamiento localizado (ROI)**  
   Se reduce la zona de interés a la mitad inferior de la imagen, disminuyendo la carga computacional sin perder precisión.

2. **PID con tiempo real (`dt`) y derivada suavizada**  
   Mejora la estabilidad y permite una respuesta proporcional al contexto dinámico del vehículo.

3. **Curvatura basada en historial**  
   Se calcula la curvatura como la variación entre múltiples valores recientes de `cX`, permitiendo determinar si el tramo es recto, suave o cerrado.

4. **Anticipación en curvas amplias**  
   Implementación de un mecanismo que desplaza el centro de control hacia el vértice interior de la curva, lo que mejora la trazada en curvas abiertas del tipo Ackermann.

5. **Velocidad progresiva con control de aceleración (`dv_max`)**  
   En lugar de saltos bruscos entre velocidades, se limita la variación máxima por ciclo, logrando una navegación más natural.

---

### Conclusión del Conjunto

Las optimizaciones aplicadas tanto en el circuito simple como en el tipo Ackermann han permitido una reducción significativa de los tiempos de recorrido y una mejora cualitativa en la estabilidad del movimiento. Cada entorno requería ajustes particulares en cuanto a anticipación, suavizado y respuesta del controlador, los cuales fueron abordados en ambas versiones con un enfoque adaptativo y eficiente.

> Este informe continuará actualizándose con nuevas pruebas y ajustes conforme se implementen mejoras adicionales.


