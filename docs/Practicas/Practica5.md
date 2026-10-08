# Práctica 5: Comunicación ESP32 por Bluetooth Classic

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345  
**Fecha:** 03/10/2026  

## 1. Objetivo
Establecer un enlace de comunicación serial inalámbrica mediante Bluetooth Classic utilizando la placa ESP32 para recibir comandos enviados desde un dispositivo móvil o PC, controlar el estado de un LED de forma inalámbrica y visualizar los datos de depuración en el Monitor Serial. Adicionalmente, documentar el protocolo de comandos, evaluar la latencia del sistema y documentar las fallas presentadas durante el armado y conexión del circuito.

## 2. Materiales

| Cantidad | Componente | Descripción |
| :---: | :--- | :--- |
| **1** | ESP32 DevKit V1 | Placa de desarrollo / Microcontrolador principal (clásico, no S3/C3). |
| **1** | LED | Diodo emisor de luz para indicador visual. |
| **1** | Resistor $220\ \Omega$ | Resistencia limitadora de corriente para el LED. |
| **1** | Cable USB | Cable de datos para programar y alimentar el ESP32. |
| **1** | Celular Android (o PC) | Dispositivo con Bluetooth y la aplicación "Serial Bluetooth Terminal". |
| **–** | Protoboard y Jumpers | Cables e interconexiones físicas para el armado del circuito. |

---

## 3. Conexiones del Circuito
* **Ánodo del LED (pata larga):** Conectado al pin **GPIO 5** del ESP32.
* **Cátodo del LED (pata corta):** Conectado a un extremo de la resistencia de $220\ \Omega$, y el otro extremo de la resistencia conectado a la masa común (**GND**).

---

## 4. Código Implementado: Control de LED por Bluetooth

El código siguiente inicializa el módulo Bluetooth del ESP32 bajo el nombre `ESP32_Emmanuel` y procesa comandos de texto mediante una lectura serial optimizada para evitar bloqueos en el bucle principal (`loop`).

```cpp
#include "BluetoothSerial.h"

BluetoothSerial SerialBT;
const int ledPin = 5;
String mensaje = "";

void setup() {
  Serial.begin(115200);
  
  SerialBT.begin("ESP32_Emmanuel");
  Serial.println("El dispositivo Bluetooth ha iniciado, listo para emparejar.");
  
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);
}

void loop() {
  if (SerialBT.available()) {
    mensaje = SerialBT.readStringUntil('\n');
    
    // Limpia la cadena de texto de saltos de línea y espacios extra
    mensaje.trim();
    
    // Muestra el comando en el Monitor Serial para depuración
    Serial.print("Comando recibido: ");
    Serial.println(mensaje);

    if (mensaje.indexOf("ON") != -1) {
      digitalWrite(ledPin, HIGH);
      SerialBT.println("Accion: LED Encendido");
    }
    else if (mensaje.indexOf("OFF") != -1) {
      digitalWrite(ledPin, LOW);
      SerialBT.println("Accion: LED Apagado");
    }
  }
}
```

---
<img width="1200" height="1600" alt="image" src="https://github.com/user-attachments/assets/80f5255e-237c-4500-8867-0403398d44da" />
<img width="1600" height="1200" alt="image" src="https://github.com/user-attachments/assets/f55dc266-0a4d-47b3-8e2a-98fa2fa24323" />


<video width="100%" controls>
  <source src="https://github.com/user-attachments/assets/1ae8af0f-b539-4387-b29c-b952f8a59888" type="video/mp4">
</video>

## 5. Explicación Teórica

### a. Comunicación Bluetooth Classic en ESP32 y Perfil Serial (SPP)
El ESP32 integra de forma nativa controladores físicos para Bluetooth de modo dual (Bluetooth Classic y BLE). Para esta práctica se utiliza el protocolo **Bluetooth Classic** empleando el perfil de puerto serie virtual (*Serial Port Profile* - SPP). Esto permite que el microcontrolador emule un puerto serial físico por el aire, haciendo posible que un dispositivo maestro (como un teléfono Android o una PC) se empareje y transmita cadenas de caracteres de la misma manera que si estuviera conectado mediante un cable USB.

### b. Recepción No Bloqueante y Saneamiento de Cadenas
Las comunicaciones inalámbricas suelen introducir caracteres de control no deseados (como retornos de carro `\r` o saltos de línea adicionales `\n`). 
* El uso de `readStringUntil('\n')` permite capturar tramas de texto completas delimitadas por el usuario.
* La función `mensaje.trim()` es indispensable para eliminar espacios en blanco y caracteres ocultos antes de realizar las comparaciones lógicas (`indexOf("ON")`), previniendo fallas de ejecución silenciosas.

---

## 6. Protocolo de Comandos
Para asegurar una comunicación clara y sin ambigüedades entre el controlador y el ESP32, se establece un protocolo de comandos basados en texto:

| Comando | Acción asociada |
| :--- | :--- |
| **ON** | Enciende el LED indicador conectado al GPIO 5. |
| **OFF** | Apaga el LED indicador conectado al GPIO 5. |

## 7. Reporte de Fallas y Soluciones

* **Falla 1: Conexión errónea del LED y polaridad invertida**
  * **Síntoma:** Al enviar el comando "ON" desde la terminal Bluetooth, el Monitor Serial indicaba que la acción se ejecutaba correctamente, pero el LED físico no encendía.
  * **¿Cómo la encontré?:** Se revisó el circuito físico y se detectó que el pin de salida estaba configurado erróneamente en el código o el diodo LED estaba polarizado de forma inversa (ánodo al cátodo).
  * **Solución:** Se corrigió la asignación del pin en el código hacia el **GPIO 5** y se verificó la orientación correcta del LED (ánodo al pin de control mediante la resistencia limitadora de $220\ \Omega$ y cátodo a la tierra común).

