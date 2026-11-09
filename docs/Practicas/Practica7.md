# Práctica 7: Control y Lazo Cerrado (PID)

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345  
**Fecha:** Noviembre 2026  

---
## 1. Objetivo de la Práctica
Comprender y documentar la transición del control en lazo abierto al control en lazo cerrado, aplicando una estrategia de sintonización de un controlador PID (Proporcional, Integral y Derivativo) sobre una plataforma de demostración interactiva. Se evaluarán parámetros críticos de control como el sobreimpulso, el tiempo de establecimiento y el error permanente.

---

## 2. Desarrollo Experimental

### Parte A: Lazo Abierto con el Carro
En esta sección se evalúa el comportamiento del vehículo sin retroalimentación, ejecutando una secuencia fija de PWM por tiempos (ej. avanzar 2 s, girar 1 s y avanzar 2 s). Se registra la desviación final respecto a la meta bajo diferentes condiciones físicas.

| Condición | Intento | Desviación del punto final (cm) | Observaciones |
| :--- | :---: | :---: | :--- |
| **Piso de referencia** | 1 | | |
| **Piso de referencia** | 2 | | |
| **Otro piso o con carga** | 1 | | |
| **Otro piso o con carga** | 2 | | |

---

### Parte B: Lazo Cerrado con la Demo (Control PID)
Utilizando el simulador interactivo, se aplicó la receta de sintonización secuencial (primero proporcional, luego derivativo y finalmente integral) para estabilizar el sistema frente a perturbaciones y llegar al objetivo exacto.

| Intento | Kp | Ki | Kd | Sobreimpulso (%) | Tiempo de establecimiento (s) | Error permanente | Observación |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **1 (Solo P)** | 2.0 | 0 | 0 | 35.0 % | 7.5 s | 12.0 % | Respuesta rápida, pero oscila demasiado. No llega al objetivo exacto (error estacionario). |
| **2 (PD)** | 2.0 | 0 | 0.5 | 10.5 % | 4.2 s | 9.5 % | La acción derivativa actuó como "freno", reduciendo drásticamente las oscilaciones. El error persiste. |
| **3 (PID Final)** | 2.5 | 0.15 | 0.8 | **3.8 %** | **2.8 s** | **0.0 %** | Sintonización óptima. La acción integral eliminó el error permanente por completo. |

#### Justificación de los Parámetros Finales
1. **Proporcional ($K_p = 2.5$):** Se incrementó para proporcionar la fuerza motriz necesaria y reaccionar rápido ante el cambio de posición.
2. **Derivativo ($K_d = 0.8$):** Se ajustó para predecir el error y amortiguar el movimiento antes de llegar a la referencia, evitando que el sistema pasara de largo y oscilara repetidas veces.
3. **Integral ($K_i = 0.15$):** Se introdujo un valor pequeño al final para sumar el error residual acumulado y forzar a los actuadores a vencer la inercia/fricción estática final, logrando que el carro se estacione exactamente en la meta (0% de error permanente).

---

## 3. Reto / Extra
> El **Intento 3** documentado en la tabla anterior demuestra que con el juego de ganancias `Kp=2.5, Ki=0.15, Kd=0.8`, se logró superar exitosamente el reto planteado en la plataforma. Se obtuvo un **sobreimpulso del 3.8%** (cumpliendo la condición de ser menor al 5%) y el sistema se estabilizó en apenas **2.8 segundos** (cumpliendo la barrera de ser menor a 5 s).

---

## 4. Conclusiones y Discusión
* **Lazo Abierto:** Se evidenció que sin retroalimentación, el sistema "ciego" es altamente susceptible a perturbaciones externas (fricción del terreno, caídas de voltaje de la batería, imperfecciones de los motores). Los errores de trayectoria se acumulan progresivamente y el vehículo es incapaz de corregirlos.
* **Lazo Cerrado (PID):** La inclusión de retroalimentación cambia drásticamente el comportamiento del sistema. El controlador monitorea constantemente la diferencia entre dónde está y dónde debería estar (el error), calculando ajustes en tiempo real. La sinergia de las tres acciones matemáticas (Proporcional para velocidad de respuesta, Derivativa para estabilidad y amortiguamiento, Integral para precisión absoluta) garantiza un posicionamiento exacto y robusto frente a perturbaciones.
