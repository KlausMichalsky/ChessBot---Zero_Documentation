###### ♟️ ChessBot---Zero
---

## 📘 Manual de communication.py

---

#### ⚙️ *** Módulo: *** communication.py

✏️ Descripción:
- Gestiona la comunicación UART con el RP2040 Zero.
- Centraliza el acceso al periférico UART.
- Evita múltiples inicializaciones del hardware.
- Proporciona funciones simples para enviar y recibir datos.

📌 Responsabilidades:
- Inicializar la UART una sola vez.
- Enviar datos al Zero.
- Verificar si hay datos disponibles.
- Leer líneas completas desde el buffer UART.

⚠️ Notas:
- La instancia UART se crea una sola vez al importar el módulo.
- La configuración (baudrate, pines, ID) se obtiene desde config.py.
- Este módulo no implementa protocolo, solo transporte UART.

🧩 Ejemplo de uso:
> `import communication`
>  
> `communication.write(b"HOME\n")`

---

<div style="page-break-before: always;"></div>

---

#### ⚙️ *** Inicialización: *** uart = UART(...)

✏️ Descripción:
- Crea la instancia UART usando los parámetros definidos en config.py.
- Se ejecuta automáticamente al importar el módulo.
- Configura:
  - ID del periférico UART
  - Baudrate
  - Pin TX
  - Pin RX

📌 Parámetros utilizados (desde config.py):
- UART_ID — identificador del periférico UART.
- UART_BAUD — velocidad de comunicación.
- UART_TX_PIN — pin de transmisión.
- UART_RX_PIN — pin de recepción.

⚠️ Notas:
- No debe crearse otra instancia UART fuera de este módulo.
- Toda la comunicación debe pasar por estas funciones.

---

#### ⚙️ *** Función: *** write(data)

✏️ Descripción:
- Envía datos a través de la UART.
- Transmite exactamente los bytes recibidos como parámetro.

📌 Parámetros:
- data — objeto tipo bytes a enviar.

⚠️ Notas:
- El parámetro debe ser bytes, no string.
- Si se envía texto, debe convertirse antes:
  - `f"{cmd}\n".encode()`
- Se recomienda usar `\n` como delimitador de comandos.

🧩 Ejemplo:
> `cmd = "MOVE 10 20"`
>  
> `communication.write(f"{cmd}\n".encode())`

---

#### ⚙️ *** Función: *** any()

✏️ Descripción:
- Indica si hay datos disponibles en el buffer UART.
- Permite saber si se puede leer sin bloquear.

📌 Retorno:
- Número de bytes disponibles en el buffer.
- 0 si no hay datos.

🧩 Ejemplo:
> `if communication.any():`
> `    print("Hay datos disponibles")`

---

#### ⚙️ *** Función: *** readline()

✏️ Descripción:
- Lee una línea completa desde la UART.
- La lectura termina cuando se recibe un salto de línea `\n`.
- Devuelve los datos en formato bytes.

📌 Retorno:
- Línea recibida como bytes.
- None si no hay línea completa disponible.

⚠️ Notas:
- El protocolo debe enviar `\n` al final de cada mensaje.
- Puede requerir decodificación:
  - `line.decode().strip()`
- decode convierte bytes a string
- strip() elimina: \n \r espacios al inicio y final

🧩 Ejemplo:
> `if communication.any():`
> `    line = communication.readline()`
> `    if line:`
> `        mensaje = line.decode().strip()`
> `        print(mensaje)`



---

## 🔄 Flujo típico de comunicación

1. La Raspberry Pi Zero 2 W envía un comando terminado en `\n`.
2. El RP2040 Zero procesa el comando.
3. El Zero responde también terminado en `\n`.
4. Este módulo recibe la línea mediante `readline()`.

---

## 🎯 Buenas prácticas

- Siempre usar delimitador `\n`.
- Siempre enviar datos como bytes.
- No bloquear el programa esperando datos sin verificar `any()`.
- Mantener este módulo como capa de transporte simple y limpia.

---

✅ Este módulo forma la base de la comunicación entre la Raspberry Pi Zero 2 W y el RP2040 Zero dentro del proyecto ChessBot---Zero.
