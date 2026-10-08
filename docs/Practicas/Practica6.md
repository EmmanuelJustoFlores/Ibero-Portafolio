# Práctica 6: Mecanismos Aplicados

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345  
**Fecha:** 03/10/2026  

## 1. Objetivo
Analizar, identificar y comprender el funcionamiento de diferentes tipos de mecanismos mecánicos impresos en 3D (sistemas de transmisión, engranes, trenes compuestos, juntas universales y transformadores de movimiento). El objetivo incluye calcular relaciones de transmisión, identificar la reversibilidad o el autobloqueo de los sistemas y asociarlos con aplicaciones reales en la mecatrónica y en el diseño del vehículo del proyecto.

## 2. Materiales y Estaciones del Taller

| Cantidad | Componente | Descripción |
| :---: | :--- | :--- |
| **1** | Kit de Mecanismos de PLA | Modelos físicos impresos en 3D de uso didáctico (diferencial, reductor cicloidal, junta cardán, etc.). |
| **1** | Calibrador / Herramienta de conteo | Para el conteo manual de dientes en trenes de engranes y ranuras de la cruz de Ginebra. |
| **–** | Dispositivo móvil o cámara | Registro fotográfico propio de cada estación de mecanismos. |

---

## 3. Análisis de Mecanismos en Estación (Diferencial)

A continuación se documenta el análisis físico del mecanismo diferencial evaluado en el taller:

* **Estación analizada:** Diferencial mecánico de engranes cónicos.
* **Fotografía de la estación:**

<img width="1200" height="1600" alt="image" src="https://github.com/user-attachments/assets/6682e56d-26f5-4188-8ff2-8d2cf4d05720" />

* **Funcionamiento y observación:** El diferencial permite que un solo eje de entrada reparta el giro hacia dos salidas independientes a velocidades distintas. Al realizar una prueba física y detener una de las salidas con el dedo, se observa de inmediato que la otra salida gira al doble de velocidad. 
* **Aplicación en el proyecto:** Aunque el carro de la materia no utiliza un diferencial mecánico tradicional (sino una *dirección diferencial* controlada por software enviando velocidades distintas a cada motor por separado), este principio es la base fundamental de la tracción automotriz.

---

## 4. Ficha de Estaciones (Resumen de Práctica)

| Estación | ¿Qué transforma? (Vel/par, rot/trasl, etc.) | Relación estimada | ¿Reversible o autobloqueante? | ¿Dónde lo has visto / aplicación real? |
| :--- | :--- | :--- | :--- | :--- |
| **A. Diferencial** | Reparte el giro entre dos salidas a distinta velocidad. | Relación variable según carga ($1:1$ base) | Reversible | Eje trasero de automóviles. |
| **B. Reductor Cicloidal** | Reducción de velocidad muy grande en poco espacio. | Alta reducción (decenas a uno) | Autobloqueante por fricción interna | Articulaciones de robots industriales. |
| **C. Junta Cardán** | Transmite rotación entre ejes que forman un ángulo. | $1:1$ (con variación de velocidad angular instantánea) | Reversible | Flecha de transmisión de vehículos. |
| **D. Obturador de láminas** | Movimiento rotatorio de entrada a apertura coordinada. | Sincronizado | Reversible | Diafragma de cámaras fotográficas. |

---

## 5. Explicación Teórica

### a. Relación de Transmisión en Engranes
Para dos engranes acoplados con $Z_1$ dientes en la entrada y $Z_2$ dientes en la salida, la relación de transmisión $i$ se define como:

$$i = \frac{Z_2}{Z_1} = \frac{\omega_{\text{entrada}}}{\omega_{\text{salida}}} = \frac{\tau_{\text{salida}}}{\tau_{\text{entrada}}}$$

* **Reducción ($i > 1$):** El engrane de salida tiene más dientes, por lo que gira más lento pero con mayor par (fuerza de giro).
* **Multiplicación ($i < 1$):** La salida gira más rápido pero con menor par.

### b. Propiedad de Autobloqueo (Sinfín + Corona)
Algunos mecanismos, como el sistema de tornillo sinfín, presentan la propiedad de **autobloqueo**: el sinfín puede mover a la corona con una gran relación de reducción en una sola etapa, pero la corona no puede empujar ni hacer girar al sinfín. Esto resulta sumamente útil en robótica para sostener cargas en posición estática sin consumir energía constante en los motores.

---

## 6. Hoja de Ejercicios Prácticos

1. **Tren simple:** Un piñón de $10$ dientes mueve un engrane de $40$ dientes. Si el motor entrega $300\text{ rpm}$ y $0.1\text{ N}\cdot\text{m}$, la velocidad de salida es:
   $$\omega_{\text{salida}} = \frac{300}{4} = 75\text{ rpm}$$
   Y el par de salida aumenta a:
   $$\tau_{\text{salida}} = 0.1 \times 4 = 0.4\text{ N}\cdot\text{m}$$

2. **Velocidad teórica del carro:** Considerando que el motor TT cuenta con una reducción interna de $1:48$, gira a $200\text{ rpm}$ a $6\text{ V}$ sin carga, y utiliza ruedas de $D = 65\text{ mm}$ ($0.065\text{ m}$), la velocidad lineal teórica se calcula como:
   $$v = \pi \cdot D \cdot \frac{\text{rpm}}{60} = \pi \cdot 0.065 \cdot \frac{200}{60} \approx 0.68\text{ m/s}$$
   En un entorno real, esta velocidad disminuye debido a la fricción con el suelo, la resistencia al avance y la caída de tensión bajo carga de las baterías.

---

## 7. Reporte de Fallas y Observaciones del Taller

* **Falla 1: Atascamiento en mecanismos impresos en 3D**
  * **Síntoma:** Algunas piezas presentaban resistencia excesiva o se atoraban al intentar girarlas manualmente.
  * **¿Cómo la encontré?:** Al operar los modelos de PLA con demasiada rapidez.
  * **Solución:** Se recordó la regla fundamental del taller: los mecanismos de plástico deben manipularse con suavidad y sin forzar las piezas. Comprender la tolerancia de impresión y la alineación de los ejes ayudó a liberar el movimiento sin dañar los dientes.
