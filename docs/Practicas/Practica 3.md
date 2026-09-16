# Práctica 3: Motor DC, Puente H, Servo y ADC

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345  
**Fecha:** 15/09/2026  
## 1. Descripción del Proyecto
Documentación y código fuente para la práctica de control de actuadores (Motor DC y Servomotor) y lecturas analógicas (ADC) mediante microcontrolador y el driver de puente H.

---

## 2. Materiales

| Cantidad | Componente | Descripción |
| :---: | :--- | :--- |
| **1** | Arduino Uno / ESP32 | Microcontrolador principal |
| **1** | Driver Puente H (TB6612 o L298N) | Driver para el control de motores DC |
| **1** | Motor DC TT | Motor con caja reductora |
| **1** | Servomotor SG90 | Actuador de posición (0°, 90°, 180°) |
| **1** | Potenciómetro 10 kΩ | Sensor analógico para control de velocidad (ADC) |
| **1** | Fuente de alimentación | Fuente externa para motores (GND común con el microcontrolador) |
| **1** | Multímetro | Medición de corriente en serie |
| **–** | Protoboard y Jumpers | Interconexiones físicas |

---

## 3. Esquemáticos y Conexiones del Circuito

### Driver Puente H y Motor DC
* **Pines de Control:**
  * `in1` $\rightarrow$ **Pin 8**
  * `in2` $\rightarrow$ **Pin 7**
  * `in3` $\rightarrow$ **Pin 2**
  * `in4` $\rightarrow$ **Pin 4**
* **Alimentación y Masa:**
  * **VCC / VM:** Alimentación externa para los motores.
  * **GND:** Masa común unida al `GND` del microcontrolador.

### Servomotor SG90
* **Señal (Naranja/Amarillo):** Conectado al **Pin 6**.
* **VCC (Rojo):** Alimentación ($5\text{ V}$).
* **GND (Marrón/Negro):** Masa común (`GND`).

### Potenciómetro (ADC)
* **Pata extrema 1:** $5\text{ V}$ / $3.3\text{ V}$.
* **Pata extrema 2:** Masa (`GND`).
* **Pata central (Señal):** Entrada analógica (`A0`).

---

## 4. Códigos Implementados

### 4.1. Control del Motor DC (Puente H)

```cpp
#define in1 8
#define in2 7
#define in3 2
#define in4 4

void setup() {
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  pinMode(in3, OUTPUT);
  pinMode(in4, OUTPUT);
}

void loop() {
  // Activa la primera dirección de giro
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  delay(1000);

  // Activa el segundo canal / sentido opuesto
  digitalWrite(in3, LOW);
  digitalWrite(in4, HIGH);
  delay(1000);
}
