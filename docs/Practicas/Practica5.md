# Práctica 5: Comunicación ESP32 por Bluetooth Classic

**Asignatura:** Introducción a la Mecatrónica
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345
**Fecha:** 03/10/2026

## 1. Objetivo
Establecer un enlace de comunicación serial inalámbrica mediante Bluetooth Classic utilizando el ESP32 para recibir comandos desde un celular o PC, controlar un LED y visualizar los datos en el Monitor Serial. Adicionalmente, documentar el protocolo de comandos y evaluar la latencia del sistema probando el impacto del uso de la función `delay()`.

## 2. Materiales

| Cantidad | Componente | Descripción |
| :---: | :--- | :--- |
| **1** | ESP32 DevKit V1 | Placa de desarrollo / Microcontrolador principal (clásico, no S3/C3). |
| **1** | LED | Diodo emisor de luz para indicador visual. |
| **1** | Resistor 220 Ω | Resistencia limitadora de corriente para el LED. |
| **1** | Cable USB | Cable de datos para programar el ESP32. |
| **1** | Celular Android (o PC) | Dispositivo con Bluetooth y la aplicación "Serial Bluetooth Terminal". |
| **–** | Protoboard y Jumpers | Cables e interconexiones físicas para el armado. |

---

## 3. Conexiones del Circuito
* **Ánodo del LED (pata larga):** Conectado al pin **GPIO 23** del ESP32.
* **Cátodo del LED (pata corta):** Conectado a la resistencia de 220 Ω, y el otro extremo de la resistencia a la masa común (GND).

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
    // Lee hasta el salto de línea (\n)
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
  
  // Sin delay(): el loop debe girar rápido para responder al instante
}
```

---

## 5. Protocolo de Comandos
Para garantizar que el ESP32 y el celular se comuniquen sin ambigüedades, se estableció el siguiente conjunto de comandos de texto, cada uno asociado a una acción clara:

| Comando | Acción |
| :--- | :--- |
| **ON** | Enciende el LED. |
| **OFF** | Apaga el LED. |
| **M,izq,der** | Fija velocidad de cada motor (ej. M,200,200 = adelante recto). |
| **S** | Alto total - failsafe. |

---

## 6. Reporte de Fallas, Latencia y Soluciones
* **Falla de lectura en la comparación de cadenas (`mensaje.trim()`):** Sin utilizar la instrucción `mensaje.trim()`, el texto recibido incluye caracteres invisibles como un retorno de carro (ej. "ON\r"). Esto provoca que la comparación `mensaje == "ON"` falle silenciosamente.
* **Experimento de Latencia (Uso de `delay`):** Si se coloca un `delay(1000)` dentro del `loop()`, el ESP32 atiende únicamente un comando por segundo. En un carro controlado bajo esta lógica, se presentaría un segundo de retraso, haciéndolo inmanejable. La solución es dejar el loop libre usando `readStringUntil('\n')` acompañado de un `setTimeout` corto.

***
