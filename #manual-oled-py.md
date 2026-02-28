###### ♟️ ChessBot---Zero
---

## 📘 Manual OLED — Módulo de Visualización

---


⚙️ *** Bloque de Inicialización: *** OLED SSD1306  
> Descripción:  
Inicializa la pantalla OLED SSD1306 mediante el bus I2C.
• Introduce un pequeño retardo para asegurar el arranque del display.  
• Configura el bus I2C con los pines definidos.  
• Crea la instancia del controlador SSD1306.  

> Configuración:
• `OLED_I2C_ID` → ID del bus I2C utilizado.  
• `OLED_I2C_SCL` → Pin SCL del bus I2C.  
• `OLED_I2C_SDA` → Pin SDA del bus I2C.  
• `OLED_FRQ` → Frecuencia del bus I2C.  
• `OLED_ANCHO` → Ancho del display en píxeles.  
• `OLED_ALTO` → Alto del display en píxeles.

> Código:
```python
# OLED SSD1306
utime.sleep_ms(200)  # Pequeño delay para que el OLED termine de arrancar
i2c = I2C(
    OLED_I2C_ID,
    scl=Pin(OLED_I2C_SCL),
    sda=Pin(OLED_I2C_SDA),
    freq=OLED_FRQ
)
oled = SSD1306_I2C(OLED_ANCHO, OLED_ALTO, i2c)

---

⚙️ *** Función: *** `oled_init()`  
> Descripción:  
Inicializa la pantalla OLED y muestra un mensaje de arranque.
• Limpia completamente el display.  
• Muestra el texto **"Iniciando..."** en la primera línea.  
• Actualiza la pantalla.  
• Introduce un pequeño retardo para asegurar el refresco.

> Parámetros: Ninguno

> Notas:  
• Debe llamarse una sola vez al iniciar el programa.  
• Asume que el objeto global `oled` ya está inicializado.  

---

⚙️ *** Función: *** `mostrar_titulo(texto)`  
> Descripción:  
Muestra un título centrado en la primera línea del OLED.
• Limpia toda la pantalla.  
• Centra el texto horizontalmente en la línea 0.  
• Actualiza el display.

> Parámetros:  
• `texto` (`str`): texto a mostrar como título.

> Notas:  
• Se recomienda usar esta función para encabezados fijos.  
• La línea 0 se preserva en operaciones posteriores.

---

⚙️ *** Función: *** `mostrar_texto(linea, texto)`  
> Descripción:  
Muestra una línea de texto alineada a la izquierda.
• Borra la parte inferior del display (excepto el título).  
• Escribe el texto desde el borde izquierdo.  
• Actualiza el display.

> Parámetros:  
• `linea` (`int`): número de línea (0, 1, 2, …).  
• `texto` (`str`): texto a mostrar.

> Notas:  
• La posición vertical se calcula con `linea * CHAR_ALTO`.  
• Ideal para mostrar información dinámica.

---

⚙️ *** Función: *** `centrar_texto(linea, texto)`  
> Descripción:  
Muestra una línea de texto centrada horizontalmente.
• Borra la parte inferior del display.  
• Calcula automáticamente la posición X para centrar el texto.  
• Actualiza el display.

> Parámetros:  
• `linea` (`int`): número de línea.  
• `texto` (`str`): texto a mostrar centrado.

> Notas:  
• Utiliza `CHAR_ANCHO` para el cálculo del centrado.  
• Útil para mensajes destacados o estados.

---

⚙️ *** Función: *** `fill_rect(x, y, w, h, color=0)`  
> Descripción:  
Rellena un rectángulo en la pantalla OLED píxel por píxel.
• Dibuja o borra un área rectangular definida por coordenadas.  
• Funciona como base para operaciones de limpieza parcial.

> Parámetros:  
• `x` (`int`): coordenada horizontal inicial.  
• `y` (`int`): coordenada vertical inicial.  
• `w` (`int`): ancho del rectángulo en píxeles.  
• `h` (`int`): alto del rectángulo en píxeles.  
• `color` (`int`):  
  • `0` → borrar (negro)  
  • `1` → dibujar (blanco)

> Notas:  
• No actualiza la pantalla por sí sola (`oled.show()` externo).  
• Útil para borrar zonas específicas sin parpadeo.

---

⚙️ *** Función: *** `borrar_parte_inferior()`  
> Descripción:  
Borra la parte inferior del OLED manteniendo intacta la línea del título.
• Limpia desde la segunda línea hasta el final del display.  
• Conserva la línea 0 como encabezado.  
• Actualiza el display.

> Parámetros: Ninguno

> Notas:  
• Internamente usa `fill_rect()`.  
• Evita limpiar toda la pantalla para reducir parpadeo.  

---
