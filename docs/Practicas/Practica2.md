# Práctica 2: ESP32: Salida, Entrada & Antirrebote

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345  
**Fecha:** 11/09/2026  

---

## 1. Objetivo
Implementar y analizar el control de entradas y salidas digitales en el microcontrolador ESP32 DevKit V1. Se busca configurar una salida digital para conmutar un LED a $1\text{ Hz}$, hacer uso de la resistencia interna `INPUT_PULLUP` para la lectura de botones, implementar un algoritmo de conmutación de estado (*toggle*) con técnica de antirrebote (*debounce*) por software sin retardos bloqueantes (`delay()`), y comparar el comportamiento de un contador de pulsaciones con y sin filtrado de rebotes.

---

## 2. Materiales y Herramientas

| Cantidad | Componente / Herramienta | Especificación / Valor |
| :---: | :--- | :--- |
| 1× | Microcontrolador | ESP32 DevKit V1 (WROOM-32) |
| 1× | Cable de comunicación | USB a Micro-USB / USB-C (Datos) |
| 1× | Diodo Emisor de Luz | LED (Rojo / Estándar) |
| 1× | Resistor de limitación | $220\,\Omega$ o $330\,\Omega$ |
| 1× | Pulsador | Push button (4 pines) |
| 1× | Resistor opcional | $10\text{ k}\Omega$ (Pull-up / Pull-down externo) |
| 1× | Elementos de ensamble | Protoboard y Cables Jumper |

---

## 3. Códigos Implementados con Comentarios

### 3.1. Práctica Blink:

```cpp
// Definición de la constante 'LED' asignada al pin GPIO 23 del ESP32 donde está cableado el circuito
#define LED 23

// Función setup(): se ejecuta una sola vez al encender o reiniciar el microcontrolador
void setup() {
  pinMode(LED, OUTPUT); // Configura el pin GPIO 23 como salida digital de voltaje (HIGH/LOW)
}

// Función loop(): se ejecuta en un ciclo infinito de manera continua
void loop() {
  digitalWrite(LED, HIGH); // Envía un nivel lógico ALTO (3.3 V) al GPIO 23 para encender el LED
  delay(1000);             // Detiene la ejecución durante 1000 ms (1 segundo) manteniendo el LED encendido
  digitalWrite(LED, LOW);  // Envía un nivel lógico BAJO (0 V / Tierra) al GPIO 23 para apagar el LED
  delay(1000);             // Detiene la ejecución durante 1000 ms (1 segundo) manteniendo el LED apagado
}
```
### 3.2. Práctica Blink con boton:
```cpp
// Define el pin GPIO 23 del ESP32 donde se encuentra conectado el LED
#define LED 23
// Define el pin GPIO 33 del ESP32 donde se encuentra conectado el pulsador
#define BUTTON 33

// Función de configuración inicial que se ejecuta una sola vez al encender el microcontrolador
void setup() {
  pinMode(LED, OUTPUT);          // Configura el GPIO 23 como salida digital de voltaje (HIGH/LOW)
  pinMode(BUTTON, INPUT_PULLUP); // Configura el GPIO 33 como entrada digital activando la resistencia Pull-Up interna (3.3 V en reposo)
}

// Función del bucle principal que se ejecuta continuamente en ciclo infinito
void loop() {
  // Comprueba si la lectura digital del botón es BAJA (LOW), lo que significa que el botón fue presionado cerrando el circuito a GND
  if (digitalRead(BUTTON) == LOW) {
    digitalWrite(LED, HIGH);     // Envía nivel lógico ALTO (3.3 V) al GPIO 23 para encender el LED mientras el botón permanezca presionado
  } else {                       // Si el botón no está presionado (la resistencia Pull-Up mantiene el pin en HIGH)
    digitalWrite(LED, LOW);      // Envía nivel lógico BAJO (0 V / GND) al GPIO 23 para mantener el LED apagado
  }
}
```
### 3.3. Práctica Toggle con antirrebote:
```cpp
// Define el pin GPIO 23 del ESP32 asignado al LED
#define LED 23
// Define el pin GPIO 33 del ESP32 asignado al pulsador
#define BUTTON 33

// Variable booleana para registrar el estado lógico del LED (false = apagado, true = encendido)
bool estadoLed = false;
// Almacena la lectura del botón en el ciclo anterior (inicializada en HIGH por la resistencia Pull-Up)
int lecturaAnterior = HIGH;
// Guarda la marca de tiempo (milisegundos) del último cambio de estado del botón
unsigned long ultimoCambio = 0;
// Tiempo del filtro antirrebote por software definido en milisegundos (30 ms)
const unsigned long DEBOUNCE_MS = 30;

// Función de configuración inicial que se ejecuta una sola vez al encender el microcontrolador
void setup() {
  pinMode(LED, OUTPUT);          // Configura el pin GPIO 23 como salida digital
  pinMode(BUTTON, INPUT_PULLUP); // Configura el pin GPIO 33 como entrada con resistencia Pull-Up interna
  Serial.begin(115200);          // Inicializa la comunicación serial a 115200 baudios para enviar datos a la consola
}

// Bucle principal que se ejecuta indefinidamente
void loop() {
  // Lee el nivel de voltaje actual en el pin del botón (HIGH o LOW)
  int lectura = digitalRead(BUTTON);

  // Comprueba si hubo un cambio de estado y si transcurrió el tiempo de filtrado antirrebote (30 ms)
  if (lectura != lecturaAnterior && millis() - ultimoCambio > DEBOUNCE_MS) {
    ultimoCambio = millis(); // Actualiza el registro de tiempo con el instante del último cambio detectado
    
    // Evalúa si el botón fue efectivamente presionado (nivel BAJO / masa)
    if (lectura == LOW) {
      estadoLed = !estadoLed;                            // Conmuta el estado lógico del LED (TOGGLE)
      digitalWrite(LED, estadoLed);                      // Envía la señal al GPIO 23 para encender o apagar el LED
      Serial.print("boton presionado:");                 // Imprime en el Monitor Serie la etiqueta de confirmación
      Serial.println(estadoLed ? "encendido" : "apagado"); // Imprime el texto "encendido" o "apagado" según el estado actual
    }
  }

  // Actualiza la variable con la lectura del estado actual para la siguiente iteración
  lecturaAnterior = lectura;
}
```

## 4. Esquematicos
### 4.1. Esquemático Blink:
<img width="480" height="270" alt="giphy" src="https://github.com/user-attachments/assets/45f4f970-2570-43cd-86d8-dea6168a8ee3" />

Circuito de salida digital básico.

* **Alimentación:** El ESP32 recibe energía y comunicación desde la computadora a través del cable USB.
* **Línea de Tierra (`GND`):** El cable blanco va desde un pin **GND** del ESP32 hacia el riel negativo (azul) de la protoboard para fijar la masa común.
* **Señal de Salida Digital:** El cable negro transporta la señal lógica de $3.3\text{ V}$ desde el pin **GPIO 23** del ESP32 directamente hacia el ánodo (terminal positiva) del LED.
* **Limitación de Corriente:** Una resistencia de $220\,\Omega$ (Rojo-Rojo-Marrón) se conecta en serie entre el cátodo (terminal negativa) del LED y la línea de masa (`GND`) para proteger el diodo contra sobrecorriente.

### 4.1. Esquemático Blink con boton:

<img width="931" height="474" alt="dedc5704-7632-4450-a758-91016bfd3632" src="https://github.com/user-attachments/assets/7da21e11-92ba-4858-bb75-39af9bd16669" />

### Circuito de Entrada Digital (Pulsador / Botón).

Circuito de entrada digital básico para la lectura de un pulsador mecánico mediante un microcontrolador ESP32.

* **Alimentación y Tierra (`GND`):** El cable blanco conecta el pin **GND** del ESP32 a la masa común en la protoboard.
* **Señal de Entrada Digital:** El cable morado conecta una de las terminales del pulsador al pin **GPIO 33** del ESP32 para registrar el cambio de estado.
* **Referencia de Masa:** El cable gris deriva la terminal del pulsador a la línea de tierra (`GND`) para cerrar el circuito al presionar.
* **Estabilización de Voltaje:** Resistencia de $10\text{ k}\Omega$ (Marrón-Negro-Naranja) configurada para fijar el nivel de voltaje de referencia y evitar falsos disparos por ruido eléctrico.

### 4.1. Esquemático Toggle con antirrebote:

<img width="931" height="474" alt="dedc5704-7632-4450-a758-91016bfd3632" src="https://github.com/user-attachments/assets/a2faa229-0ffb-4140-8111-60f110dbb1ac" />

### Circuito de Entrada y Salida Digital con Antirrebote.

Circuito de control digital que combina una entrada por pulsador (GPIO 33) y una salida por LED (GPIO 23) utilizando un microcontrolador ESP32.

* **Alimentación y Tierra (`GND`):** El cable blanco conecta el pin **GND** del ESP32 al riel negativo (azul) de la protoboard para fijar la masa común.
* **Etapa de Salida Digital (LED):** El cable negro conecta el pin **GPIO 23** del ESP32 al ánodo del LED, cuya corriente es limitada en serie por una resistencia de $220\,\Omega$ (Rojo-Rojo-Marrón) conectada a `GND`.
* **Etapa de Entrada Digital (Pulsador):** Los cables morado y gris conectan las terminales del pulsador mecánico al pin **GPIO 33** del ESP32 y a la referencia de masa (`GND`).
* **Estabilización de Voltaje:** Se incluye una resistencia de $10\text{ k}\Omega$ (Marrón-Negro-Naranja) en la etapa de entrada para evitar lecturas erráticas o estado flotante al presionar el botón.
  
### 4.3. Videos de Funcionamiento

https://github.com/user-attachments/assets/0de9aeb1-d5ff-4cfd-ab79-5b950de74a7e

https://github.com/user-attachments/assets/d41355eb-6ac8-4a3b-9520-77a86f3190ca

https://github.com/user-attachments/assets/882d7cd6-23f9-4a39-a691-d0f5e1822e2f

---

### 4.4. Explicación Teórica

#### a. ¿Qué es el rebote de un botón?
Cuando presionas o sueltas un botón, las piezas de metal en su interior chocan y rebotan un par de veces antes de quedarse quietas. 
Como el ESP32 es un procesador extremadamente rápido, detecta todos esos pequeños brincos en milisegundos y piensa que presionaste el botón muchas veces seguidas en lugar de una sola.

#### b. ¿Por qué con `INPUT_PULLUP` la lógica queda invertida?
Al activar `INPUT_PULLUP`, el ESP32 mantiene el pin alimentado con energía todo el tiempo. Por eso, cuando el botón **no está presionado**, la tarjeta detecta un nivel **ALTO (`HIGH` / `1`)**.
Cuando **presionas el botón**, la corriente se va a tierra (`GND`) y el voltaje cae a cero, así que la tarjeta detecta un nivel **BAJO (`LOW` / `0`)**.
En resumen: el botón funciona "al revés" (suelto es `1` y presionado es `0`).

---

## 5. Conclusiones

### 5.2. Conclusiones
* **Filtro por código (*antirrebote*):** Comprobamos que es necesario usar un filtro en el código para limpiar los brinquitos del botón y evitar que la tarjeta lea pulsaciones falsas.
* **Uso de `millis()`:** Usar `millis()` en lugar de `delay()` evita que el programa se congele esperando, lo que permite que el ESP32 siga haciendo otras tareas al mismo tiempo.
