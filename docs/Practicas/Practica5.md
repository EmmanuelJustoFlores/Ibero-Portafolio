# Práctica 5: Comunicación ESP32 por Bluetooth Classic

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** [Tu Nombre / Tu Equipo]  
**Fecha:** [Fecha actual]  

## 1. Objetivo
Establecer un enlace de comunicación serial inalámbrica mediante Bluetooth Classic utilizando el ESP32 para recibir comandos desde un celular o PC, controlar un LED y visualizar los datos en el Monitor Serial[cite: 11, 15]. Adicionalmente, documentar el protocolo de comandos y evaluar la latencia del sistema[cite: 15].

## 2. Materiales

| Cantidad | Componente | Descripción |
| :---: | :--- | :--- |
| **1** | ESP32 DevKit V1 | Placa de desarrollo / Microcontrolador principal (clásico, no S3/C3)[cite: 15, 19]. |
| **1** | LED | Diodo emisor de luz para indicador visual[cite: 15]. |
| **1** | Resistor $220\ \Omega$ | Resistencia limitadora de corriente para el LED[cite: 15]. |
| **1** | Cable USB | Cable de datos para programar el ESP32[cite: 15]. |
| **1** | Celular Android (o PC) | Dispositivo con Bluetooth y la aplicación "Serial Bluetooth Terminal"[cite: 15]. |
| **–** | Protoboard y Jumpers | Cables e interconexiones físicas para el armado[cite: 15]. |

---

## 3. Conexiones del Circuito

* **LED:**
  * Ánodo (pata larga) $\rightarrow$ Conectado al pin **GPIO 23** del ESP32.
  * Cátodo (pata corta) $\rightarrow$ Conectado a la resistencia de $220\ \Omega$, y esta a la masa común (**GND**).

---

## 4. Código Implementado: Control de LED por Bluetooth

El siguiente código permite emparejar el ESP32 con el celular y controlar el encendido/apagado del LED enviando texto. Es crucial configurar la app de terminal para que agregue un salto de línea (`Newline = LF`) al final de cada envío.

```cpp
#include "BluetoothSerial.h"

// Crea el objeto para la comunicación Bluetooth
BluetoothSerial SerialBT;

// Pin donde está conectado el LED
#define LED 23

void setup() {
  // Inicializa el monitor serial tradicional
  Serial.begin(115200);
  
  // Nombre con el que aparecerá el dispositivo Bluetooth
  SerialBT.begin("ESP32");
  
  // Timeout corto para que readStringUntil no bloquee el loop
  SerialBT.setTimeout(20);
  
  // Configura el pin del LED como salida
  pinMode(LED, OUTPUT);
}

void loop() {
  // Revisa si hay datos entrantes por Bluetooth
  if (SerialBT.available()) {
    // Lee hasta encontrar el salto de línea (\n)
    String mensaje = SerialBT.readStringUntil('\n');
    
    // Quita espacios y retornos de carro (\r). ¡Sin esto la comparación falla!
    mensaje.trim(); 
    
    Serial.println("Recibido: " + mensaje);
    
    // Compara el mensaje recibido para ejecutar una acción
    if (mensaje == "ON") {
      digitalWrite(LED, HIGH);
    } 
    else if (mensaje == "OFF") {
      digitalWrite(LED, LOW);
    }
  }
  
  // NOTA: Nunca usar delay() en este loop, ya que causaría retrasos 
  // al recibir y procesar los comandos.
}
