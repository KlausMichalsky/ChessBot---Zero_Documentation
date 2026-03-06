# **📘Manual de firmware**

Este documento define las reglas fundamentales de diseño del
firmware.
- Contenido de archivos
- Definición de tipos de datos del sistema.
- Declaración de variables y constantes.
- Mantención de arquitectura limpia.
- Evitar colisiones y errores por tipos inseguros.

## **➡️ Contenido de Archivos**

| Qué poner en `*.h`                                      |
| ------------------------------------------------------- |
| Constantes, direcciones, límites, parámetros ajustables |

| Qué poner en `*.cpp` / `*.ino`                          |
| ------------------------------------------------------- |
| Variables de estado, contadores, temporizadores, flags  |

| Tipo             | Dónde va                 | Qué contiene                                                                                  |
|-----------------|--------------------------|------------------------------------------------------------------------------------------------|
| Config           | `config.h`               | Comandos (`Command`), estructuras de configuración (`HomingConfig`), pines, velocidades, microstepping, límites |
| Hardware / motor | `homingXY.cpp` / `homingZ.cpp` | Variables de estado (`HomingStateXY`, `HomingStateZ`), `Start()`, `Update()`, lógica de homing |
| Coordinación     | `core.cpp`               | Secuencias globales (`HOME_ALL`), máquinas de estado globales (no internas de motor), llamadas a `Start()` y chequeo de `state` |

| Archivo      | Qué haces                              | Resultado                                               |
| ------------ | -------------------------------------- | ------------------------------------------------------- |
| core.cpp     | `HomingRunTimeXY homingMotor1;`        | Reserva memoria y crea la variable                      |
| core.h       | `extern HomingRunTimeXY homingMotor1;` | Permite que otros archivos accedan a la variable        |
| otro archivo | `#include "core.h"`                    | Puede leer/escribir `homingMotor1` sin definirla otra vez |

📦 Arquitectura correcta
main_zero.ino
      │
      ▼
     core
      │
 ┌────┴─────┐
 │          │
motors   homingXY / homingZ
 │          │
drivers   sensores Hall
core decide qué hacer y cuando hacerlo
homing ejecuta el algoritmo
motors mueve el hardware

🧠 Responsabilidad de cada módulo
motors
Debe encargarse solo del hardware del motor:
AccelStepper
setSpeed()
setAcceleration()
run()
pines STEP / DIR / ENABLE
Es decir: control físico del motor.
homingXY / homingZ
Se encargan de la lógica del algoritmo de homing:
máquina de estados
detección de flancos
cálculo del centro
control del sensor Hall
Pero no deberían poseer las variables globales del sistema.
core
core es el cerebro del robot físico.
Ahí:
coordinás motores
ejecutás secuencias
controlás HOME_ALL
actualizás estados
decidís qué sistema corre en cada loop
Entonces las variables runtime del sistema deben vivir aquí.

# ⭐ Convenciones de nomenclatura
| Tipo        | Estilo        | Ejemplo              |
|--------------|--------------|----------------------|
| Struct       | PascalCase   | `HomingState`        |
| Enum         | PascalCase   | `HomingStatus`       |
| Enum values  | MAYÚSCULAS   | `HOMING_OK`          |
| Funciones    | camelCase    | `updateHoming()`     |
|              | (moduloAccionObjetoSufijo)          |
| Variables    | camelCase    | `motorSpeed`         |
| Constantes   | MAYÚSCULAS   | `MAX_SPEED`          |
| Archivos     | snake_case   | `homing_manager.cpp` |


### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> ChessBot---Zero.ino**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> calc.h / calc.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> commands.h / commands.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> communication.h / communication.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> config.h / config.h**
```c
#pragma once
#include <Arduino.h>
```
```c
// 🔹 DEFINICIÓN DE PINES FISICOS
// =======================================================================
// Sensor1, Encoder1, Motor 1
#define HALL_1 3
#define AS5600_1_SDA 4
#define AS5600_1_SCL 5
#define MOTOR1_ENABLE 6
#define MOTOR1_DIR 7
#define MOTOR1_STEP 8

// Sensor2, Encoder2, Motor 2
#define HALL_2 15
#define AS5600_2_SDA 26
#define AS5600_2_SCL 27
#define MOTOR2_ENABLE 12
#define MOTOR2_DIR 13
#define MOTOR2_STEP 14

// Sensor3, Encoder3, Motor 3
#define HALL_3 29
#define MOTOR3_ENABLE 9
#define MOTOR3_DIR 10
#define MOTOR3_STEP 11

// Pines de LEDs y electroimán
#define LED 2
#define IMAN 28
```
```c
// 🔹 DEFINICIÓN PARAMETROS DE CONFIGURACIÓN DEö AS5600
// =======================================================================
#define AS5600_ADDR 0x36
#define SEND_INTERVAL 33 // ms -> ~30Hz
#define DELTA_DEG 0.5f   // Enviar si el ángulo cambia más de este DELTA_DEG
```
```c
// 🔹 NIVELES LÓGICOS DE ENABLE
// =======================================================================
// constexpr bool es ideal para:
// Flags de configuración
// Habilitar/deshabilitar módulos
// Pines fijos
// Parámetros que nunca cambian
// Este valor se conoce y se calcula en tiempo de compilación, no en tiempo de ejecución.
// Aqui ponemos el valor LOW (Enable activo por defecto en el TMC2209)
// a una variable de tipo bool o uint8_t cualquiera sirve. 

constexpr bool ENABLE_ACTIVE = LOW; // Nivel lógico (LOW=ON, HIGH=OFF)
constexpr bool ENABLE_INACTIVE = HIGH;
```
```c
// 🔹 TIPOS DE DATOS
// =======================================================================
// Define un conjunto cerrado de valores posibles.
// Se utiliza para:
// Estados de máquina (Homing, Motor, Sistema).
// Comandos UART.
// Flags lógicos del sistema.
// Comandos recibidos por UART
enum class Command
{
    STATUS,
    RESET_ERRORS,
    HOME_MOTOR1,
    HOME_MOTOR2,
    GET_ANGLE1,
    GET_ANGLE1_START,
    GET_ANGLE2,
    GET_ANGLE2_START,
    GET_ANGLE_STOP,
    UNKNOWN
};

// Estados de la rutina de homing 
// CW = ClockWise, CCW = CounterClockWise
// EDGE = Flanco del imán
// Reverse = invertir dirección
enum class HomingStateXY
{
    INACTIVE,
    FIND_FIRST_EDGE_CW,
    FIND_SECOND_EDGE_CW,
    FIND_FIRST_EDGE_CCW,
    FIND_SECOND_EDGE_CCW,
    SEARCH_FAST_CW,
    SEARCH_FAST_CCW,
    REVERSE_EDGE_CW,
    REVERSE_EDGE_CCW,
    CALC_CENTER,
    MOVE_TO_CENTER,
    OK,
    ERROR
};
```
```c
// 🔹 ESTRUCTURAS DE CONFIGURACIÓN
// =======================================================================
// Se utiliza cuando necesitamos agrupar variables relacionadas.
// Incluye todos los parámetros mecánicos y de velocidad necesarios para cada motor.
// El punto . delante de cada nombre de campo dentro de la inicialización
// de la estructura se llama “designated initializer” o inicializador designado.
// Significa que le estás diciendo explícitamente a qué campo de la estructura
// va cada valor, sin importar el orden.

// Ideal para:
// Runtime de estados.
// Configuraciones.
// Estructuras de datos simples.

// `struct` → miembros públicos por defecto.
// `class` → miembros privados por defecto.

struct HomingConfig
{
    int microstepping;
    int reduction;
    int stepsPerRevolution;

    float fastSpeed;
    float slowSpeed;
    float acceleration;

    long steps90Deg;
    unsigned long timeout;
    int enablePin;
};

// Configuracion de Homing para cada motor, con parámetros mecánicos específicos
// Para funciones:
// Originalmente inline se usaba para funciones.
// Le dice al compilador: “podés insertar el código de la función directamente donde se llama, en vez de hacer una llamada normal”.
// Ventaja: menos overhead de llamadas pequeñas.
// Para variables (C++17+)
// Ahora también sirve para variables const o constexpr.
// Permite que la misma variable se defina en varios .cpp sin generar error de “multiple definition”.
// El linker trata todas las definiciones como una sola instancia.

// ☝🏻 Cada .cpp que incluya config.h ve la variable, pero el linker no se queja de duplicados.

// Regla práctica
// Constantes, structs pequeños o configuraciones como motor1Config → inline en config.h ✅
// Variables grandes, objetos dinámicos, funciones complejas → .cpp + extern en .h ✅

inline const HomingConfig motor1Config = {
    .microstepping = 16,
    .reduction = 9,
    .stepsPerRevolution = 200,
    .fastSpeed = 1500.0,
    .slowSpeed = 800.0,
    .acceleration = 1000.0,
    .steps90Deg = motor1Config.microstepping * motor1Config.stepsPerRevolution / 4,
    .timeout = 15000, // 15 seg
    .enablePin = MOTOR1_ENABLE};

inline const HomingConfig motor2Config = {
    .microstepping = 16,
    .reduction = 6,
    .stepsPerRevolution = 200,
    .fastSpeed = 1000.0,
    .slowSpeed = 533.0,
    .acceleration = 1000.0,
    .steps90Deg = motor2Config.microstepping * motor2Config.stepsPerRevolution / 4,
    .timeout = 15000, // 15 seg
    .enablePin = MOTOR2_ENABLE};

inline const HomingConfig motor3Config = {
    .microstepping = 8,
    .reduction = 1,
    .stepsPerRevolution = 200,
    .fastSpeed = 4000.0,
    .slowSpeed = 500.0,
    .acceleration = 1000.0,
    .stepsLimit = -100, // pasos máximos si arranca fuera del imán
    .timeout = 12000,
    .enablePin = MOTOR3_ENABLE};
```


--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> homingXY.h / homingXY.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> motorsXY.h / motorsXY.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> sensors.h / sensors.cpp**

...

--------------------------------------------------------------------------




