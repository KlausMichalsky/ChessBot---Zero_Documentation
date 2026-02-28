###### ♟️ ChessBot---Zero
---

## 📘 Manual de motorsXY.cpp

---

#### ⚙️ *** Módulo: *** motorsXY.cpp

Módulo responsable del control del sistema **XY** del robot de ajedrez. Este módulo gestiona **energía, movimiento y estado** del sistema XY. El proceso de referencia (homing) se realiza en el módulo `homingXY`.

---

### ⚙️ *** Sección: *** INSTANCIAS DE MOTORES

✏️ Descripción:
Crea las instancias de los motores utilizando la librería AccelStepper.

🧩 Implementación:
> `AccelStepper motor1(AccelStepper::DRIVER, MOTOR1_STEP, MOTOR1_DIR);`  
> `AccelStepper motor2(AccelStepper::DRIVER, MOTOR2_STEP, MOTOR2_DIR);`

---

### ⚙️ *** Sección: *** ESTADO INTERNO DE MOTORES

✏️ Descripción:
Esta variable controla si los motores están habilitados (true) o deshabilitados (false).

- ▸ static: hace que la variable sea **local al archivo motor.cpp**,  
  es decir, **no es visible desde otros módulos**.  
- ▸ Se inicializa en false para que al arrancar el sistema  
  los motores estén apagados por seguridad.  
- ▸ Se actualiza en motorsXY_Enable() y motorsXY_Disable() para que  
  otras funciones del módulo sepan si se puede mover el motor.

🧩 Implementación:
> `static bool motorsEnabled = false;`

---

### ⚙️ *** Sección: *** API PÚBLICA (Application Programming Interface)

✏️ Descripción:
Inicializa los motores del sistema y prepara el hardware para operar de manera segura.

📌 Funcionalidad:
- Configura los pines ENABLE de cada motor como salida.  
- Deshabilita los motores al arrancar para evitar movimientos inesperados (estado seguro).  
- Establece parámetros básicos de cada motor:  
  • Velocidad máxima (fastSpeed)  
  • Aceleración (acceleration)  
- Prepara los motores para recibir comandos de movimiento  

⚠️ Nota:
Esta función debe llamarse **antes de cualquier intento de mover los motores**, preferentemente al inicio del setup().

---

### ⚙️ Función: `motorsXY_Init()`

✏️ **Descripción:**
Inicializa el módulo de motores XY. Configura pines, drivers, variables internas y deja el sistema listo para operar.

📌 **Parámetros:**

* Ninguno.

⚠️ **Notas:**

* Debe llamarse **una sola vez** al inicio del programa.
* No habilita los motores.
* No realiza homing.

🧩 **Ejemplo:**

```cpp
motorsXY_Init();
```

---

### ⚙️ Función: `motorsXY_Enable()`

✏️ **Descripción:**
Habilita eléctricamente los motores XY.

📌 **Parámetros:**

* Ninguno.

⚠️ **Notas:**

* Requerido antes de cualquier movimiento.
* No inicia movimiento.

🧩 **Ejemplo:**

```cpp
motorsXY_Enable();
```

---

### ⚙️ Función: `motorsXY_Disable()`

✏️ **Descripción:**
Deshabilita eléctricamente los motores XY.

📌 **Parámetros:**

* Ninguno.

⚠️ **Notas:**

* Libera los motores.
* Útil para reposo o seguridad.

🧩 **Ejemplo:**

```cpp
motorsXY_Disable();
```

---

### ⚙️ Función: `motorsXY_Move(long x, long y)`

✏️ **Descripción:**
Ordena un movimiento relativo del sistema XY.

📌 **Parámetros:**

* `x` → Pasos a mover en el eje X.
* `y` → Pasos a mover en el eje Y.

⚠️ **Notas:**

* Movimiento **no bloqueante**.
* Requiere motores habilitados.

🧩 **Ejemplo:**

```cpp
motorsXY_Move(2000, -500);
```

---

### ⚙️ Función: `motorsXY_SetSpeed(long speed)`

✏️ **Descripción:**
Configura la velocidad del sistema XY.

📌 **Parámetros:**

* `speed` → Velocidad en pasos por segundo.

⚠️ **Notas:**

* Debe llamarse antes del movimiento.
* Afecta a ambos ejes.

🧩 **Ejemplo:**

```cpp
motorsXY_SetSpeed(1200);
```

---

### ⚙️ Función: `motorsXY_Run()`

✏️ **Descripción:**
Servicio del módulo XY. Actualiza el estado y ejecuta los movimientos pendientes.

📌 **Parámetros:**

* Ninguno.

⚠️ **Notas:**

* Debe llamarse continuamente en `loop()`.
* No bloquea.

🧩 **Ejemplo:**

```cpp
void loop()
{
    motorsXY_Run();
}
```

---

### ⚙️ Función: `motorsXY_Stop()`

✏️ **Descripción:**
Detiene inmediatamente el movimiento del sistema XY.

📌 **Parámetros:**

* Ninguno.

⚠️ **Notas:**

* No deshabilita los motores.
* Pensada para situaciones de emergencia.

🧩 **Ejemplo:**

```cpp
motorsXY_Stop();
```

---

### ⚙️ Función: `motorsXY_Done()`

✏️ **Descripción:**
Indica si el último movimiento del sistema XY finalizó.

📌 **Parámetros:**

* Ninguno.

📤 **Retorna:**

* `true` → movimiento finalizado.
* `false` → movimiento en curso.

⚠️ **Notas:**

* No inicia ni detiene movimientos.
* Devuelve true si **ambos motores han llegado** a su posición objetivo
* distanceToGo() devuelve la distancia restante que le falta
  al motor para alcanzar el objetivo definido por moveTo().
* Se verifica que motor1 y motor2 hayan terminado su movimiento.
* Esto permite que otras funciones (por ejemplo, homing o secuencias  de movimiento) sepan cuándo se completó el desplazamiento.

🧩 **Ejemplo:**

```cpp
if (motorsXY_Done())
{
    // siguiente acción
}
```

---

### 📌 Flujo típico de uso

```cpp
motorsXY_Init();
motorsXY_Enable();
motorsXY_SetSpeed(1000);

motorsXY_Move(3000, 1500);

while (!motorsXY_Done())
{
    motorsXY_Run();
}
```

---

### 📎 Relación con otros módulos

* `homingXY` → referencia del sistema XY
* `motorsZ` → control del eje Z
* `communication` → comandos UART

---

### ⚠️ Estado del módulo

* Control de energía ✔️
* Movimiento relativo ✔️
* Máquina de estados no bloqueante ✔️

---
