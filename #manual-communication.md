# **📘 Manual de communication.cpp**

- Inicializar la comunicación UART.
- Configurar los pines físicos TX/RX.
- Proveer una función de depuración opcional.
- Es el puente de comunicación entre el Zero (RP2040) y el Raspberry Pi.

## **⚙️ Función debug()**

Envía un mensaje por UART solo si la depuración está activada.

**🧩 Implementación:**
```c
void debug(const String &msg)
{
#if DEBUG_UART
    Serial1.println("Comando recibido: " + msg);
#endif
}
```
Si DEBUG_UART = 1 → imprime mensaje.
Si DEBUG_UART = 0 → no imprime nada.
El compilador elimina el código cuando está desactivado.
Sirve para monitorear comandos sin afectar el sistema cuando no se necesita.

## **⚙️ Función UART_Init()**

Inicializa la comunicación serial del sistema.

**🧩 Implementación:**
```c
void UART_Init()
{
    Serial.begin(115200);   // USB debug
    Serial1.setTX(0);       // Pin TX
    Serial1.setRX(1);       // Pin RX
    Serial1.begin(115200);  // UART hardware
}
```
Serial → comunicación USB con PC.
Serial1 → UART hardware hacia Raspberry.
Velocidad: 115200 baudios.

## **🧠 Resumen conceptual**

1️⃣ Inicialización de UART → UART_Init()
2️⃣ Comunicación USB para debug → Serial.begin(115200)
3️⃣ Comunicación UART hardware → Serial1.begin(115200) con pines TX=0, RX=1
4️⃣ Depuración opcional → debug(msg) controlada por DEBUG_UART
5️⃣ Comunicación no bloqueante → permite que motores y sensores sigan funcionando mientras llegan comandos

Con esto se logra una comunicación confiable entre Zero y Raspberry Pi, con posibilidad de depuración sin afectar el rendimiento del sistema.