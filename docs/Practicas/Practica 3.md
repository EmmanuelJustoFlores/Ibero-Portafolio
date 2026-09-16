# Práctica 3: Motor DC, Puente H, Servo y ADC

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345  
**Fecha:** 15/09/2026  
## 1. Descripción del Proyecto
Documentación y código fuente para la práctica de control de actuadores (Motor DC y Servomotor SG90) y lectura de entradas analógicas utilizando la plataforma **Arduino Uno** y un driver de **Puente H**.

## 2. Materiales

| Cantidad | Componente | Descripción |
| :---: | :--- | :--- |
| **1** | Arduino Uno | Placa de desarrollo / Microcontrolador principal |
| **1** | Driver Puente H | Módulo de control de potencia para motores DC |
| **1** | Motor DC TT | Motor con caja reductora |
| **1** | Servomotor SG90 | Actuador de posición angular ($0^\circ$, $90^\circ$, $180^\circ$) |
| **1** | Potenciómetro $10\text{ k}\Omega$ | Sensor analógico para lecturas ADC |
| **1** | Fuente de alimentación | Fuente externa de energía para motores (GND común con Arduino) |
| **1** | Multímetro | Medición de corriente en serie |
| **–** | Protoboard y Jumpers | Cables e interconexiones físicas |

---

## 3. Conexiones del Circuito

### Driver Puente H y Motor DC
* **Entradas de Control:**
  * `in1` $\rightarrow$ **Pin 8** (Arduino)
  * `in2` $\rightarrow$ **Pin 7** (Arduino)
  * `in3` $\rightarrow$ **Pin 2** (Arduino)
  * `in4` $\rightarrow$ **Pin 4** (Arduino)
* **Alimentación y Tierra:**
  * **VCC / VM:** Positivo de la fuente externa ($5\text{ V} - 12\text{ V}$).
  * **GND:** Masa común (unida al pin `GND` de Arduino y al negativo de la fuente externa).

### Servomotor SG90
* **Señal (Naranja / Amarillo):** Conectado al **Pin 6** (Arduino).
* **VCC (Rojo):** Alimentación ($5\text{ V}$).
* **GND (Marrón / Negro):** Masa común (`GND`).

### Potenciómetro (Entrada Analógica ADC)
* **Terminal 1:** $5\text{ V}$.
* **Terminal 2:** Masa (`GND`).
* **Terminal Central (Señal):** Conectada a la entrada analógica **A0** (Arduino).

---

## 4. Códigos Implementados

### 4.1. Control del Motor DC (Puente H)
```cpp
#define in1 8
#define in2 7
#define in3 2
#define in4 4

void setup() {
  // Configura los pines de control del Puente H como salidas digitales
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  pinMode(in3, OUTPUT);
  pinMode(in4, OUTPUT);
}

void loop() {
  // Activa la primera dirección de giro en el motor
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  delay(1000); // Mantiene el giro por 1 segundo

  // Activa la segunda dirección de giro / canal opuesto
  digitalWrite(in3, LOW);
  digitalWrite(in4, HIGH);
  delay(1000); // Mantiene el giro por 1 segundo
}
```
### 4.2. Control del Servomotor SG90
```cpp
#include <Servo.h>

Servo miServo;

void setup() {
  // Asigna el pin 6 de Arduino para el control del servomotor
  miServo.attach(6);
}

void loop() {
  // Posiciona el servo en ángulos de 0°, 90° y 180°
  miServo.write(0);
  delay(1000);
  
  miServo.write(90);
  delay(1000);
  
  miServo.write(180);
  delay(1000);
}
```
## 5. Explicación Teórica

### a. Prueba de Carga: Corriente en Arranque vs. Giro Libre

* **Giro libre:** Cuando el motor gira sin carga ni resistencia mecánica, el consumo de corriente es bajo debido a que solo vence la fricción interna de sus componentes.
* **Arranque / Carga:** La corriente demandada es significativamente mayor (de 3 a 5 veces más alta) porque en el instante inicial el motor está detenido, no existe fuerza electromotriz opuesta y requiere la máxima potencia para romper la inercia.

### b. Cálculo del Ciclo de Trabajo (Duty Cycle) del Servo

Los servomotores operan con una señal PWM de frecuencia fija de **50 Hz**, lo que representa un periodo total de **20 ms**.

$$Duty\ Cycle\ (\%) = \left(\frac{Tiempo\ en\ alto\ (ms)}{20\ ms}\right) \times 100$$

* **Posición 0° (~0.5 ms en alto):**
  $$\left(\frac{0.5\ ms}{20\ ms}\right) \times 100 = 2.5\%$$

* **Posición 90° (~1.5 ms en alto):**
  $$\left(\frac{1.5\ ms}{20\ ms}\right) \times 100 = 7.5\%$$

* **Posición 180° (~2.5 ms en alto):**
  $$\left(\frac{2.5\ ms}{20\ ms}\right) \times 100 = 12.5\%$$

---

## 6. Videos de Funcionamiento y Explicación por Voz

A continuación se presentan los enlaces a los videos demostrativos del funcionamiento físico de los circuitos, con explicación por voz de cada prueba realizada:

* **Video 1: Control de Motor DC con Puente H**
  * **Descripción:** Explicación por voz del cambio de sentido de giro del motor DC mediante las entradas de control (`in1` a `in4`) y la alimentación compartida.
  * **Enlace al video:** [Ver demostración en video](URL_DEL_VIDEO_AQUI)

* **Video 2: Posicionamiento del Servomotor SG90**
  * **Descripción:** Demostración comentada del movimiento angular del servo a $0^\circ$, $90^\circ$ y $180^\circ$ mediante señales PWM.
  * **Enlace al video:** [Ver demostración en video](URL_DEL_VIDEO_AQUI)


    
## 7. Reporte de Fallas y Soluciones

#### Falla 1: Inactividad en el Motor DC
* **Síntoma:** El motor DC no realizaba ningún movimiento al ejecutar el código.
* **¿Cómo la encontré?:** Se comprobó con un multímetro la falta de diferencia de potencial en los terminales del motor.
* **Solución:** Se conectó la tierra (`GND`) de la fuente de alimentación externa directamente con el `GND` de la placa Arduino para unificar la referencia lógica.

#### Falla 2: Inestabilidad y reinicios por el Servomotor
* **Síntoma:** El servomotor producía un zumbido, vibraciones erráticas o causaba el reinicio del Arduino Uno.
* **¿Cómo la encontré?:** Se detectó una caída de voltaje en la línea de 5V al ejecutar la función de movimiento.
* **Solución:** Se separó la alimentación de potencia del servomotor utilizando una fuente externa dedicada y manteniendo la masa común.
