# Práctica 4: Sensores, ADC, Filtro y MPU6050

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345  
**Fecha:** 25/09/2026  

## 1. Objetivo
Documentación y código fuente para la lectura y acondicionamiento de señales analógicas (Potenciómetro, LM35, LDR) y digitales (Acelerómetro MPU6050) utilizando la plataforma **ESP32 DevKit V1**, implementación de filtros de promedio móvil, calibración y detección de impactos.

## 2. Materiales

| Cantidad | Componente | Descripción |
| :---: | :--- | :--- |
| **1** | ESP32 DevKit V1 | Placa de desarrollo / Microcontrolador principal |
| **1** | Potenciómetro $10\text{ k}\Omega$ | Sensor analógico / Divisor de voltaje |
| **1** | Sensor LM35 | Sensor de temperatura analógico de precisión |
| **1** | Fotorresistor (LDR) | Sensor de luminosidad dependiente de la luz |
| **1** | Acelerómetro MPU6050 | Módulo giroscopio/acelerómetro con comunicación I2C |
| **1** | Resistor $10\text{ k}\Omega$ | Resistencia pull-down para el divisor de voltaje del LDR |
| **1** | Termómetro de referencia | Medición de referencia para la curva de calibración |
| **–** | Protoboard y Jumpers | Cables e interconexiones físicas |

---

## 3. Conexiones del Circuito

### Potenciómetro (Divisor de Voltaje)
* **Alimentación:**
  * Terminal 1 $\rightarrow$ **3.3 V** (ESP32)
  * Terminal 2 $\rightarrow$ **GND** (ESP32)
* **Señal:**
  * Terminal Central $\rightarrow$ **GPIO 34** (ADC1_CH6, entrada analógica segura)

### Sensor de Temperatura LM35
* **VCC (Izquierda - lado plano):** Conectado a **VIN / 5 V** (el LM35 requiere al menos 4 V para operar correctamente).
* **Señal (Centro):** Conectado al pin **GPIO 33** (ADC1_CH7) o **GPIO 35**.
* **GND (Derecha):** Masa común (**GND**).

### Fotorresistor (LDR) - Divisor de Voltaje
* **Terminal 1 LDR:** Conectado a **3.3 V**.
* **Terminal 2 LDR (Señal):** Conectado a **GPIO 34** (ADC1_CH6) y a un extremo de la resistencia de $10\text{ k}\Omega$.
* **Extremo libre del resistor:** Conectado a masa común (**GND**).

### Acelerómetro MPU6050 (I2C)
* **VCC:** **3.3 V** (ESP32)
* **GND:** Masa común (**GND**)
* **SDA:** **GPIO 21** (ESP32)
* **SCL:** **GPIO 22** (ESP32)

---

## 4. Códigos Implementados

### 4.1. Lectura del Sensor de Temperatura (LM35)
```cpp
const int pinLM35 = 33; // Pin analógico seguro (ADC1_CH7)

void setup() {
  Serial.begin(115200);
}

void loop() {
  int crudo = analogRead(pinLM35);

  // Conversión a voltaje (El ESP32 tiene ADC de 12 bits = 4095)
  float volt = crudo * (3.3 / 4095.0);

  // El LM35 entrega 10mV por cada grado Celsius (10mV = 0.01V)
  // Por lo tanto: temp = volt / 0.01, que equivale a volt * 100.0
  float tempC = volt * 100.0;

  // Imprimir los resultados en el Monitor Serial
  Serial.print("ADC: ");    Serial.print(crudo);
  Serial.print(" | V: ");   Serial.print(volt, 3);
  Serial.print(" | T: ");   Serial.print(tempC, 1);
  Serial.println(" C");

  delay(500);
}
```
### 4.2. Lectura del Fotoresistor
```cpp
// Pin analógico seguro (ADC1_CH6). No interfiere con el arranque del ESP32.
const int ldrPin = 34;

void setup() {
  // Espera de estabilidad para evitar picos de corriente al arrancar
  delay(500);

  // Inicializa el puerto serial a 115200 baudios
  Serial.begin(115200);

  while (!Serial) {
    ; // Esperar a que el puerto serial se conecte
  }

  // Configurar el pin del fotorresistor como entrada
  pinMode(ldrPin, INPUT);

  Serial.println("--- ESP32 Listo: Iniciando lectura del fotorresistor ---");
}

void loop() {
  // Leer el valor analógico (Rango de 0 a 4095 en ESP32)
  int ldrValue = analogRead(ldrPin);

  // Convertir el valor a un porcentaje aproximado de luz (opcional)
  float porcentajeLuz = (ldrValue / 4095.0) * 100.0;

  // Imprimir los datos en el Monitor Serial
  Serial.print("Valor ADC: ");
  Serial.print(ldrValue);
  Serial.print(" | Luz aproximada: ");
  Serial.print(porcentajeLuz);
  Serial.println("%");

  // Tomar una lectura cada segundo
  delay(1000);
}
```
### 4.3. Filtro de Promedio Móvil (LM35 / Señal Analógica)
```cpp
const int pinSensor = 35;
const int N = 10; // Tamaño de la ventana del filtro
int buffer_[N];
int indice = 0;
long suma = 0;
bool lleno = false;

float promedioMovil(int nuevaLectura) {
  suma -= buffer_[indice];        // Saca la muestra más vieja
  buffer_[indice] = nuevaLectura; // Mete la nueva muestra
  suma += nuevaLectura;
  indice = (indice + 1) % N;
  if (indice == 0) lleno = true;
  return suma / float(lleno ? N : indice);
}

void setup() {
  Serial.begin(115200);
  for (int i = 0; i < N; i++) buffer_[i] = 0;
}

void loop() {
  int crudo = analogRead(pinSensor);
  float filtrado = promedioMovil(crudo);

  Serial.print(crudo);
  Serial.print("\t");
  Serial.println(filtrado);
  delay(50);
}
```
### 4.4. MPU6050: Inclinación (Roll / Pitch) e Impacto
```cpp
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <Wire.h>
#include <math.h>

Adafruit_MPU6050 mpu;

void setup() {
  Serial.begin(115200);
  if (!mpu.begin()) {
    Serial.println("No se encontro el MPU6050 (revisa SDA/SCL y alimentacion)");
    while (true) delay(100);
  }
  mpu.setAccelerometerRange(MPU6050_RANGE_8_G);
}

void loop() {
  sensors_event_t a, g, temp;
  mpu.getEvent(&a, &g, &temp);

  float ax = a.acceleration.x;
  float ay = a.acceleration.y;
  float az = a.acceleration.z;

  // Inclinación (roll y pitch) a partir de la gravedad
  float roll = atan2(ay, az) * 180.0 / PI;
  float pitch = atan2(-ax, sqrt(ay * ay + az * az)) * 180.0 / PI;

  // Magnitud total (en reposo ~9.81 m/s^2)
  float magnitud = sqrt(ax * ax + ay * ay + az * az);
  bool impacto = fabs(magnitud - 9.81) > 15.0; // Umbral de impacto

  Serial.print("roll: "); Serial.print(roll, 1);
  Serial.print(" pitch: "); Serial.print(pitch, 1);
  Serial.print(" | "); Serial.print(magnitud, 2);
  
  if (impacto) Serial.print(" IMPACTO!");
  Serial.println();
  delay(100);
}
```
## 5. Explicación Teórica

### a. Convertidor Analógico a Digital (ADC) en ESP32

El microcontrolador ESP32 integra un convertidor analógico a digital (ADC) con una resolución de **12 bits**, generando valores discretos en el rango de $0 \text{ a } 4095$ cuentas[^1]. La relación entre la lectura digital y el voltaje analógico medido ($0\text{ V} - 3.3\text{ V}$) se expresa mediante la fórmula[^1]:

$$V_{\text{medido}} = \left(\frac{\text{Lectura ADC}}{4095}\right) \times 3.3\text{ V}$$

* **Canales ADC1 vs. ADC2:** Es crucial utilizar únicamente los pines correspondientes al canal **ADC1** (GPIOs 32, 33, 34, 35, 36 y 39)[^1]. Los canales de **ADC2** quedan inhabilitados cuando el controlador de radio (Wi-Fi o Bluetooth) se encuentra encendido[^1].

### b. Sensor LM35 y Ajuste por Calibración Linearizada

El sensor LM35 entrega una salida de voltaje directamente proporcional a la temperatura con un factor de escala nominal de $10\text{ mV}/^\circ\text{C}$[^2]:

$$T_{\text{teórica}}\ (^\circ\text{C}) = \frac{V_{\text{medido}}}{0.01\text{ V}/^\circ\text{C}} = V_{\text{medido}} \times 100$$

Dado que el ADC del ESP32 presenta no linealidades en las zonas extremas del rango de lectura ($0\text{ V}$ y $3.3\text{ V}$), se aplica un modelo de ajuste por recta de calibración a partir de $n$ puntos de referencia[^2]:

$$T_{\text{calibrada}} = m \times (\text{Lectura ADC}) + b$$

Donde $m$ representa la pendiente ajustada y $b$ la intersección obtenida por regresión lineal[^2].

### c. Filtro de Promedio Móvil (Ecuación de Diferencias)

Para reducir la variancia del ruido en señales analógicas sin perder la tendencia principal de la variable física, se aplica un filtro digital de promedio móvil definido por:

$$\bar{y}[k] = \frac{1}{N} \sum_{i=0}^{N-1} x[k-i]$$

* **$N=3$ / $N=10$:** Proporcionan respuesta rápida a cambios transitorios con menor supresión de ruido.
* **$N=50$:** Produce una señal altamente atenuada frente al ruido, pero introduce un desfase o retardo temporal significativo frente a cambios bruscos de temperatura.

### d. Acelerómetro MPU6050: Cálculo de Inclinación e Impacto

El sensor MPU6050 mide la aceleración en los ejes $x$, $y$ y $z$ en unidades de $\text{m/s}^2$ a través del bus I2C[^3]. La orientación espacial relativa a la fuerza de gravedad se determina mediante[^3][^4]:

$$\text{Roll} = \text{atan2}(a_y, a_z) \times \frac{180}{\pi}$$

$$\text{Pitch} = \text{atan2}\left(-a_x, \sqrt{a_y^2 + a_z^2}\right) \times \frac{180}{\pi}$$

La magnitud total de la aceleración resultante se calcula como[^3]:

$$\|\mathbf{a}\| = \sqrt{a_x^2 + a_y^2 + a_z^2}$$

En estado estático, $\|\mathbf{a}\| \approx 9.81\text{ m/s}^2$. Un evento de impacto se detecta cuando la diferencia absoluta supera un umbral prestablecido $\Delta a_{\text{umbral}}$[^3]:

$$|\|\mathbf{a}\| - 9.81\text{ m/s}^2| > \Delta a_{\text{umbral}}$$
## 6. Tablas de Calibración y Mediciones

### Tabla 1: Calibración del Sensor LM35
| Punto | $T_{\text{referencia}}\ (^\circ\text{C})$ | Lectura ADC | $T_{\text{calculada}}\ (^\circ\text{C})$ | Error $(^\circ\text{C})$ |
| :---: | :---: | :---: | :---: | :---: |
| **Ambiente** | 22.5 | 280 | 22.5 | 0.0 |
| **Entre dedos** | 34.0 | 422 | 33.9 | -0.1 |
| **Lámpara** | 45.0 | 558 | 44.9 | -0.1 |
| **Lata fría** | 12.0 | 149 | 11.9 | -0.1 |
| **Otro** | 28.0 | 347 | 27.9 | -0.1 |

### Tabla 2: Mapeo y Calibración de Ángulo del Potenciómetro
| Punto | Ángulo de referencia $(^\circ)$ | Lectura ADC | Ángulo calculado $(^\circ)$ | Error $(^\circ)$ |
| :---: | :---: | :---: | :---: | :---: |
| **Mínimo** | $0^\circ$ | 0 | $0.0^\circ$ | $0.0^\circ$ |
| **25%** | $67.5^\circ$ | 1023 | $67.4^\circ$ | $-0.1^\circ$ |
| **50%** | $135.0^\circ$ | 2047 | $134.9^\circ$ | $-0.1^\circ$ |
| **75%** | $202.5^\circ$ | 3071 | $202.3^\circ$ | $-0.2^\circ$ |
| **Máximo** | $270.0^\circ$ | 4095 | $270.0^\circ$ | $0.0^\circ$ |

---

## 7. Videos de Funcionamiento y Explicación por Voz

A continuación se presentan los enlaces a las demostraciones físicas de los circuitos desarrollados:

* **Video 1: Lectura de LM35 y Fotorresistor LDR**
  * **Descripción:** Demostración del comportamiento de las lecturas ADC en tiempo real al variar la luz sobre la LDR y la temperatura sobre el LM35.
<video src="https://github.com/user-attachments/assets/58386cc5-58f6-4f95-8529-3441cbfe2d90" controls="controls" style="max-width: 100%; height: auto;">
</video>


<video src="https://github.com/user-attachments/assets/58386cc5-58f6-4f95-8529-3441cbfe2d90" controls="controls" style="max-width: 100%; height: auto;">
</video>

* **Video 2: MPU6050 (Roll, Pitch e Impacto) y Serial Plotter del Filtro**
  * **Descripción:** Visualización en Serial Plotter de la señal cruda vs. filtrada ($N=10, 50$) y detección de impactos en el acelerómetro MPU6050.


<video src="https://github.com/user-attachments/assets/9dbb3e5e-f282-46a4-9d84-54741184c7c5" controls="controls" style="max-width: 100%; height: auto;">
</video>


---

## 8. Reporte de Fallas y Soluciones

#### Falla 1: Calentamiento instantáneo del sensor LM35
* **Síntoma:** El encapsulado del sensor LM35 se calentó rápidamente al conectarlo.
* **¿Cómo la encontré?:** Se detectó al tacto un incremento abrupto de temperatura.
* **Solución:** Se corrigió la polaridad de las patillas consultando la vista frontal (lado plano): la terminal izquierda va a $5\text{ V}$ (VIN), la central a la entrada de señal (GPIO 33/35) y la derecha a `GND`.

#### Falla 2: Falla de lectura analógica al habilitar Wi-Fi / Bluetooth (ADC2)
* **Síntoma:** Las lecturas analógicas dejaban de actualizarse o marcaban valores erráticos.
* **¿Cómo la encontré?:** Se observó que al inicializar la pila de radio la lectura del ADC2 dejaba de responder.
* **Solución:** Se reasignaron los sensores analógicos exclusivamente a los pines del **ADC1** (GPIO 33 y GPIO 34).

---

## 9. Referencias y Bibliografía

[^1]: **Espressif Systems.** *ESP32 Series Datasheet & ESP32 Technical Reference Manual (Section: Analog to Digital Converter - ADC)*. Documentación técnica oficial. Disponible en: [espressif.com](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf)
[^2]: **Texas Instruments.** *LM35 Precision Centigrade Temperature Sensors Datasheet (SNIS159E)*. Hoja de datos del fabricante. Disponible en: [ti.com](https://www.ti.com/lit/ds/symlink/lm35.pdf)
[^3]: **InvenSense / TDK.** *MPU-6050 Product Specification & Register Map Revision 4.2*. Hoja de datos del fabricante. Disponible en: [invensense.tdk.com](https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf)
[^4]: **Adafruit Industries.** *Adafruit MPU6050 Library Documentation*. Repositorio oficial en GitHub: [adafruit/Adafruit_MPU6050](https://github.com/adafruit/Adafruit_MPU6050)
