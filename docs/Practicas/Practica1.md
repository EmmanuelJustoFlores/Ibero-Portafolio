# Práctica 1: 555 en Astable (LED Parpadeante)

**Asignatura:** Introducción a la Mecatrónica  
**Autor:** Emmanuel Justo Flores -208048, Romina Velarde Mata -207345    
**Fecha:** 05/09/2026  

---

## 1. Objetivo
Construir un oscilador astable utilizando el temporizador NE555 para hacer parpadear un LED, calcular la frecuencia y el ciclo de trabajo (*duty cycle*) teóricos, medirlos experimentalmente y analizar las diferencias.

---

## 2. Materiales y Herramientas

| Cantidad | Componente / Herramienta | Especificación / Valor |
| :---: | :--- | :--- |
| 1× | Circuito Integrado | NE555 (DIP-8) |
| 1× | Diodo Emisor de Luz | LED (Rojo / Estándar) |
| 1× | Resistor de limitación para LED | $330\,\Omega$ o $470\,\Omega$ |
| 1× | Resistor temporizador $R_A$ | $1\text{ k}\Omega$ |
| 1× | Resistor temporizador $R_B$ | $10\text{ k}\Omega$ |
| 1× | Capacitor de temporización $C$ | $100\,\mu\text{F}$ (Electrolítico) / $100\text{ nF}$ (Cerámico) |
| 1× | Capacitor de desacoplo | $10\text{ nF}$ (para pin 5 CTRL) |
| 1× | Fuente de alimentación | Fuente regulada de 5 V DC / Protoboard / Cables |

---

## 3. Diseño y Fórmulas Teóricas

El capacitor $C$ se carga a través de $R_A + R_B$ y se descarga únicamente a través de $R_B$:

* **Tiempo en ALTO ($t_{ALTO}$):**  
  $$t_{ALTO} = 0.693 \cdot (R_A + R_B) \cdot C$$

* **Tiempo en BAJO ($t_{BAJO}$):**  
  $$t_{BAJO} = 0.693 \cdot R_B \cdot C$$

* **Frecuencia teórica ($f$):**  
  $$f = \frac{1.44}{(R_A + 2R_B) \cdot C} = \frac{1.44}{(1000 + 20000) \cdot 100 \times 10^{-6}} \approx 0.69\text{ Hz}$$

* **Ciclo de trabajo teóricamente calculado ($Duty$):**  
  $$Duty = \frac{R_A + R_B}{R_A + 2R_B} \times 100 = \frac{11000}{21000} \times 100 \approx 52.4\%$$

---

## 4. Entregables

### 4.1. Esquemático Anotado
![Demostración circuito 555](https://media1.tenor.com/m/ZLt09yCH5cQAAAAC/5555-led-working.gif)

El diagrama muestra la configuración del temporizador NE555 operando en modo astable (oscilador libre). La función de cada nodo del circuito se detalla a continuación:

* **Alimentación y Control de Reset (Pines 8 y 4):** Se conectan a la línea positiva de alimentación ($V_{CC} = 5\text{ V}$).
* **Referencia de Tierra (Pin 1):** Conectado a la masa común ($GND$).
* **Red de carga y oscilación ($R_A$, $R_B$, $C$):**
  * La corriente inicial fluye desde $V_{CC}$ hacia el capacitor $C$ atravesando las resistencias $R_A$ y $R_B$ en serie.
  * El **Pin 7 (Descarga)** se conecta al punto medio entre $R_A$ y $R_B$. Durante la fase de descarga, el transistor interno del 555 conmuta a tierra y vacía la energía del capacitor $C$ únicamente a través de $R_B$.
  * Los pines **Pin 2 (Disparo)** y **Pin 6 (Umbral)** están unidos al polo positivo del capacitor $C$. Esta conexión permite al integrado sensar la tensión en el capacitor y alternar la salida del voltaje.
* **Estabilización de Referencia (Pin 5):** Se conecta a $GND$ mediante un capacitor cerámico de $10\text{ nF}$ para filtrar variaciones de tensión en el divisor resistivo interno del integrado.
* **Etapa de Salida (Pin 3):** Entrega la señal de onda cuadrada resultante, conectada en serie a la resistencia limitadora de $330\,\Omega$ y al LED indicador.
### 4.2. Montaje en Protoboard
[Foto del circuito en protoboard]


<img width="1599" height="1200" alt="a104b64c-33ca-4d31-bb6c-eac111d8b00b" src="https://github.com/user-attachments/assets/d5eb2563-eb73-4b36-82d5-5d27e02771e6" />


### 4.3. Mediciones y Comparación (Teoría vs. Práctica)

El porcentaje de error se calcula mediante la fórmula:

$$\% \text{ error} = \frac{|\text{Valor Teórico} - \text{Valor Medido}|}{\text{Valor Teórico}} \times 100$$

| Magnitud | Teórico (Calculado) | Medido (Experimental) | % de Error | Instrumento Utilizado |
| :--- | :---: | :---: | :---: | :--- |
| **$V_{CC}$ (V)** | 5.0 V | 4.99 V | 0.20% | Multímetro (V en paralelo) |
| **$V_{salida}$ en ALTO (V)** | $\approx V_{CC} - 1.5\text{ V}$ (3.5 V) | 3.2 V | 8.57% | Multímetro / Osciloscopio |
| **Frecuencia (Hz)** | 0.69 Hz | 358.9 mHz | 47.98% | Osciloscopio / DMM con Hz |
| **Duty (%)** | 52.4 % | 52.78 % | 0.73% | Osciloscopio (Measure) |
| **$I_{LED}$ (mA)** | 4.55 mA | 3.64 mA | 20.0% | Multímetro (A en serie) |

### 4.4. Demostración en Video
[Demostración del circuito]


https://github.com/user-attachments/assets/ddad14eb-2296-4cb1-b93c-edd6ea065297

---

## 5. Bitácora de Errores y Explicación

### ¿Por qué no dio exacto el valor medido respecto al teórico?
El desfase entre el valor teórico ($0.69\text{ Hz}$) y el medido se debe principalmente a las tolerancias físicas de los componentes:
* **Resistencias ($R_A, R_B$):** Presentan una tolerancia típica de $\pm 5\%$.
* **Capacitor Electrolítico ($C$):** Es el componente que **domina el error**, ya que los capacitores electrolíticos tienen variaciones de capacitancia reales que oscilan entre $-20\%$ y $+80\%$ respecto a su valor nominal impreso.

### Registro de fallas durante el ensamble
1. **¿Qué falló?:** luz led mal colocada.
2. **¿Cómo se encontró?:** pensábamos en un error en el acomodado de los jumpers así que probamos diferentes versiones y fue cuando invertimos los polos del led que descubrimos el error.
3. **¿Cómo se resolvió?:** poniendo el polo negativo del led al jumper que lo conecta a la salida del temporizador 555


.
