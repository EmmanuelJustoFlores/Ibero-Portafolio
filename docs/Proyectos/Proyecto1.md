# Proyectos

## Proyecto 1: KickOff - Vehículo Controlado por Bluetooth (Fútbol RC)

### 1. Contexto y Objetivo del Proyecto
El objetivo principal del Proyecto 1 es diseñar, construir en equipo y programar un vehículo a control remoto operado mediante comunicación Bluetooth desde un celular o PC. Este proyecto integra los conocimientos adquiridos en el curso sobre electrónica, microcontroladores, actuadores, comunicación y mecanismos. El desarrollo culminará con la participación del vehículo en un torneo de fútbol RC 1v1 entre equipos.

### 2. Arquitectura del Sistema
El flujo de control y hardware del vehículo sigue el siguiente diagrama de bloques:
**Celular / PC (app de terminal BT)** $\rightarrow$ **ESP32** $\rightarrow$ **Driver TB6612** $\rightarrow$ **2 motores TT** $\rightarrow$ **Ruedas**.

### 3. Requisitos Funcionales y Restricciones
Para que el proyecto sea validado y admitido en el torneo, debe cumplir obligatoriamente con los siguientes requerimientos:

* **Movimiento y Control (R1, R2, R3):** El carro debe moverse adelante, atrás, izquierda, derecha y hacer alto total mediante comandos Bluetooth, integrando control de velocidad por PWM. El protocolo de comandos debe estar documentado en el portafolio.
* **Seguridad y Failsafe (R4):** Si se pierde el enlace o dejan de llegar comandos en un lapso $\le 1$ segundo, el carro debe detenerse automáticamente.
* **Alimentación de Energía (R5, R6):** El vehículo debe incluir un interruptor físico de encendido/apagado accesible sin necesidad de desarmar componentes. La alimentación de los motores debe ser independiente de la del ESP32, compartiendo únicamente la tierra (GND común).
* **Mecánica, Chasis y Cableado (R7, R8, Restricciones):**
  * Las dimensiones máximas de huella son $20 \times 20\text{ cm}$ para asegurar la maniobrabilidad en la cancha.
  * El cableado debe ser ordenado y sujeto, evitando piezas o cables arrastrando.
  * Es indispensable dejar una superficie plana y horizontal de al menos $6 \times 6\text{ cm}$ completamente despejada en la parte superior, donde vivirá el marcador ArUco en fases posteriores.
