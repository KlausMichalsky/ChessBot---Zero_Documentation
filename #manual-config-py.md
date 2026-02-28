###### ♟️ ChessBot---Zero
---

## 📘 Manual de config.py

---

#### ⚙️ *** Módulo: *** config.py

✏️ Descripción:
- Contiene las constantes de configuración del sistema.
- Centraliza parámetros de hardware y comunicación.
- Permite modificar pines o velocidades sin tocar la lógica principal.
- Facilita mantenimiento y escalabilidad del proyecto.

📌 Responsabilidades:
- Definir parámetros UART.
- Definir pines utilizados por el Raspberry.
- Mantener constantes globales organizadas.
- Evitar valores "hardcodeados" en el código principal.

⚠️ Notas:
- Este archivo NO debe contener lógica.
- Solo debe contener constantes.
- Cualquier cambio de hardware debe hacerse aquí primero.

---

<div style="page-break-before: always;"></div>

---

## 🔌 Configuración UART

#### ⚙️ *** Constante: *** UART_ID

✏️ Descripción:
- Identificador del periférico UART utilizado en el Raspberry.
- Determina qué controlador hardware se usa (por ejemplo UART0 o UART1).

📌 Tipo:
- Entero

🧩 Ejemplo:
> `UART_ID = 0`

---

#### ⚙️ *** Constante: *** UART_BAUD

✏️ Descripción:
- Velocidad de comunicación en baudios.
- Debe coincidir exactamente con el dispositivo del otro lado.

📌 Tipo:
- Entero

⚠️ Notas:
- Si no coincide el baudrate, los datos se corrompen.
- Valores típicos: 9600, 57600, 115200.

🧩 Ejemplo:
> `UART_BAUD = 115200`

---

#### ⚙️ *** Constante: *** UART_TX_PIN

✏️ Descripción:
- Pin configurado como transmisión (TX) en el Raspberry.
- Envía datos hacia la RP2040-Zero.

📌 Tipo:
- Número de pin GPIO

🧩 Ejemplo:
> `UART_TX_PIN = 0`

---

#### ⚙️ *** Constante: *** UART_RX_PIN

✏️ Descripción:
- Pin configurado como recepción (RX) en el Raspberry.
- Recibe datos desde la RP2040-Zero.

📌 Tipo:
- Número de pin GPIO

🧩 Ejemplo:
> `UART_RX_PIN = 1`

---

## 🔄 Flujo de uso

1. `communication.py` importa config.py.
2. Se leen las constantes UART.
3. Se inicializa la UART con esos parámetros.
4. Todo el sistema usa esa configuración centralizada.

---

## 🎯 Buenas prácticas

- No modificar estos valores desde otros archivos.
- No redefinir pines en el código principal.
- Mantener nombres claros y consistentes.
- Agrupar futuras configuraciones (motores, sensores, etc.) aquí.

---

## 🚀 Ventajas de esta estructura

- Código más limpio.
- Cambios de hardware más rápidos.
- Menos errores por valores duplicados.
- Mejor organización del proyecto ChessBot---Zero.

---

✅ config.py es la base estructural de configuración del sistema y permite que el resto del código permanezca modular y ordenado.
