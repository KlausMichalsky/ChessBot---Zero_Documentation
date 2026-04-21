# **📘Manual de firmware**

Este documento define las reglas fundamentales de diseño del
firmware.
- Contenido de archivos.
- Definición de tipos de datos del sistema.
- Declaración de variables y constantes.
- Mantención de arquitectura limpia.

## ⭐ Convenciones de nomenclatura
| Tipo                             | Estilo                   | Ejemplo                 |
| -------------------------------- | ------------------------ | ----------------------- |
| Struct                           | PascalCase               | `HomingState`           |
| Enum                             | PascalCase               | `HomingStatus`          |
| Enum values                      | MAYÚSCULAS               | `HOMING_OK`             |
| Funciones                        | camelCase                | `updateHoming()`        |
| Funciones (convención extendida) | moduloAccionObjetoSufijo | `motorUpdatePosition()` |
| Variables                        | camelCase                | `motorSpeed`            |
| Constantes                       | MAYÚSCULAS               | `MOTOR1_ENABLE`         |
| Archivos                         | snake_case               | `homing_manager.cpp`    |
| Clases                           | PascalCase               | `HomingManager`         |
| Namespaces                       | snake_case               | `motor_control`         |
| Getters                          | get + camelCase          | `getPosition()`         |
| Setters                          | set + camelCase          | `setSpeed()`            |
| Variables globales               | prefijo g_ -> "esto es global"              | `g_motorSpeed`          |
| Miembros privados                | prefijo _ -> “esto NO se toca desde afuera”                | `_encoderCount`         |
| Booleanos                        | is / has / can           | `isHomed`, `hasError`   |
| ISR (interrupciones)             | prefijo ISR_             | `ISR_encoderA()`        |
| Macros                           | MAYÚSCULAS + _           | `MOTOR_MAX_SPEED`       |

## 🔎 Contenido de Archivos

| 📁 Contenido de archivos (headers)`*.h`                                                                                        |
| ---------------------------------------------------------------------------------------------------------------------- |
| Constantes, definiciones, direcciones, límites del sistema, parámetros ajustables y declaraciones de funciones/clases. |

| 📁 Contenido de archivos `*.cpp` / `*.ino`                                                                                                                   |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Implementación de funciones, lógica del sistema, variables de estado (si son internas del módulo), contadores, temporizadores, flags y código ejecutable. |


| Tipo             | Dónde va                       | Qué contiene                                                                                                                                                   |
| ---------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Config           | `config.h`                     | Pines, velocidades, microstepping, límites del sistema, estructuras de configuración (`MotorConfig`, `HomingConfig`, etc.), enums de comandos (`Command`)      |
| Hardware / motor | `homingXY.cpp` / `homingZ.cpp` | Lógica del homing, máquinas de estado (`HomingStateXY`, `HomingStateZ`), funciones `Start()`, `Step()/Update()`, lectura de sensores y control del motor       |
| Coordinación     | `core.cpp`                     | Secuencias globales (`HOME_ALL`), coordinación entre motores, máquinas de estado globales del sistema y llamadas a `Start()` / `Step()` con chequeo de estados |

| Archivo      | Qué haces                              | Resultado                                      |
| ------------ | -------------------------------------- | ---------------------------------------------- |
| `core.cpp`   | `HomingRunTimeXY homingMotor1;`        | Se **define** la variable (reserva memoria)    |
| `core.h`     | `extern HomingRunTimeXY homingMotor1;` | Se **declara** la variable para otros archivos |
| otros `.cpp` | `#include "core.h"`                    | Permite acceder a la variable sin redefinirla  |

## 📦 Arquitectura de modulos

| Módulo                                | Función                                                                                 |
| ------------------------------------- | --------------------------------------------------------------------------------------- |
| `ChessBot---Zero.ino`                 | Punto de entrada del firmware Arduino. Inicializa `core` y arranca el loop principal.   |
| `core.cpp / core.h`                   | Cerebro del sistema. Coordina todos los módulos y define el flujo general del robot.    |
| `config.h`                            | Configuración global: pines, límites, velocidades, constantes y parámetros del sistema. |
| `command.cpp / command.h`             | Interpretación de comandos (parser + definición de comandos del sistema).               |
| `communication.cpp / communication.h` | Comunicación con el exterior (Serial / UART / posibles interfaces futuras).             |
| `homing.cpp / homing.h`               | Algoritmo de homing: máquina de estados, detección de sensores y cálculo de referencia. |
| `motors.cpp / motors.h`               | Control físico de motores (STEP/DIR/ENABLE, velocidad, aceleración).                    |
| `sensors.cpp / sensors.h`             | Lectura de sensores (Hall, finales de carrera, inputs físicos).                         |
| `xy_plane.cpp / xy_plane.h`           | Control de cinemática/plano XY (movimientos coordinados en dos ejes).                   |
| `z_axis.cpp / z_axis.h`               | Control del eje Z (movimiento vertical independiente).                                  |
| `utils.cpp / utils.h`                 | Funciones auxiliares reutilizables (helpers, cálculos, herramientas generales).         |

## 🔁 Flujo del sistema

| Nivel                 | Qué hace                                            |
| --------------------- | --------------------------------------------------- |
| `ChessBot---Zero.ino` | Inicializa el sistema y entra en el loop principal  |
| `communication`       | Recibe datos/comandos desde el exterior             |
| `command`             | Interpreta los comandos recibidos                   |
| `core`                | Decide qué acción ejecutar según estado del sistema |
| `homing`              | Ejecuta calibración de ejes si es necesario         |
| `xy_plane / z_axis`   | Convierte órdenes en movimiento por eje             |
| `motors`              | Ejecuta movimiento físico real                      |
| `sensors`             | Retroalimentación del estado físico                 |
| `utils`               | Soporte transversal en cualquier etapa              |

## 🔖 Responsabilidad de cada módulo
| Módulo                | Responsabilidad                                                        | Qué NO debe hacer                                           |
| --------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------- |
| `core`                | Control central del sistema, estados globales, coordinación de módulos | No debe manejar detalles eléctricos ni lógica de bajo nivel |
| `command`             | Definir y parsear comandos del sistema                                 | No debe controlar hardware directamente                     |
| `communication`       | Entrada/salida de datos (Serial, protocolos)                           | No debe interpretar lógica del robot                        |
| `homing`              | Algoritmo de referencia (máquina de estados, detección de sensores)    | No debe decidir movimientos globales del sistema            |
| `motors`              | Control físico de motores                                              | No debe contener lógica de homing o planificación           |
| `sensors`             | Lectura de sensores físicos                                            | No debe tomar decisiones                                    |
| `xy_plane`            | Movimiento coordinado XY (cinemática / planificación local)            | No debe tocar drivers directamente                          |
| `z_axis`              | Control del eje Z                                                      | No debe coordinar otros ejes                                |
| `utils`               | Funciones auxiliares reutilizables                                     | No debe depender del estado del robot                       |
| `config`              | Parámetros globales                                                    | No debe contener lógica                                     |
| `ChessBot---Zero.ino` | Arranque del sistema                                                   | No debe contener lógica compleja                            |

## 🧩 Idea clave de la arquitectura
| Capa                | Rol                            |
| ------------------- | ------------------------------ |
| `communication`     | 🌐 Entrada de datos            |
| `command`           | 🧾 Interpretación              |
| `core`              | 🧠 Decisión central            |
| `homing`            | 🔍 Calibración                 |
| `xy_plane / z_axis` | 🧭 Planificación de movimiento |
| `motors`            | 🦾 Ejecución física            |
| `sensors`           | 👁 Feedback del mundo real     |
| `utils`             | 🧰 Soporte general             |

## **🧠 Resumen mental del sistema**
* communication → recibe información
* command → la traduce
* core → decide qué hacer
* homing → calibra si hace falta
* xy_plane / z_axis → convierten decisiones en movimiento
* motors → mueven el robot
* sensors → confirman la realidad


## 🌍 Variables globales y `extern`

### 1️⃣ ¿Qué hace `extern`?

`extern` le dice al compilador:

> “Esta variable existe en otro archivo, solo quiero usarla aquí, no la crees otra vez.”

---

### 🧠 Diferencia clave

| Tipo | Significado |
|------|-------------|
| **Declaración (`extern`)** | Solo informa que la variable existe |
| **Definición** | Crea la variable y reserva memoria |

---


### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> ChessBot---Zero.ino**

...


--------------------------------------------------------------------------

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> command.h**

- Declaración de funciones para la gestión de comandos  recibidos por UART

```c
#pragma once

#include <Arduino.h>

#include "config.h"
```
```c
bool commandAvailable();
void processCommand(const String &command);
String readCommand();
void commandSendStatusReport();
```


### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> command.cpp**

...


--------------------------------------------------------------------------

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> communication.h**

- Declaracion de la interfaz de comunicación UART

```c
#pragma once
#include <Arduino.h>
#define DEBUG_UART 0 // 1 = debug activado, 0 = debug desactivado

void debug(const String &msg); // función para debug condicional
void communicationInit();
```

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> communication.cpp**

- Inicializar UART
- Comunicación UART con el Raspberry
- Gestionar la recepción de datos de forma no bloqueante

```c
#include <Arduino.h>

#include "communication.h"
#include "config.h"
```

```c
// 🔹 CONFIGURACIÓN DE DEPURACIÓN UART
// =======================================================================
void debug(const String &msg) {
#if DEBUG_UART
    Serial1.println("Comando recibido: " + msg);
#endif
}

void communicationInit() {
    // USB para debug (opcional)
    Serial.begin(115200);

    // UART hardware en pines 0 y 1
    Serial1.setTX(0);
    Serial1.setRX(1);
    Serial1.begin(115200);
    while (Serial1.available())
        Serial1.read(); // limpia buffer UART
}
```

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> config.h**

- Declaracion de pines fisicos, constantes y tipos de datos.
- Declaracion de parametros globales.
- Declaracion de enums de estados.
- Declaracion de estructuras de configuración de motores.
- Centralizar parámetros mecánicos dependientes del hardware.

```c
#pragma once
#include <Arduino.h>
```
```c
// 🔹 PINES DE CONFIGURACIÓN
// =======================================================================
// Sensor1, Encoder1, Motor1
#define HALL_1 3
#define AS5600_1_SDA 4
#define AS5600_1_SCL 5
#define MOTOR1_ENABLE 6
#define MOTOR1_DIR 7
#define MOTOR1_STEP 8
// Sensor2, Encoder2, Motor2
#define HALL_2 15
#define AS5600_2_SDA 26
#define AS5600_2_SCL 27
#define MOTOR2_ENABLE 12
#define MOTOR2_DIR 13
#define MOTOR2_STEP 14
// Sensor3, Encoder3, Motor3
#define HALL_3 29
#define MOTOR3_ENABLE 9
#define MOTOR3_DIR 10
#define MOTOR3_STEP 11
// Pines de Status-LEDs y electroimán
#define LED 2
#define MAGNET 28
```
```c
// 🔹 PARAMETROS DE CONFIGURACIÓN DEL PLANO XY
// =======================================================================
// BASE_SPEED: velocidad máxima que tendrá el motor que recorre la mayor distancia
// ✔ suficientemente alta para que el movimiento no sea lento
// ✔ suficientemente baja para que ningún motor pierda pasos

#define AS5600_ADDR 0x36
#define SEND_INTERVAL 33 // ms -> ~30Hz
#define DELTA_DEG 0.5f   // Enviar si el ángulo cambia más de DELTA_DEG
#define BASE_SPEED 1500  // usada para igualar tiempo de llegada de motor1 y 2
```
```c
// 🔹 PARAMETROS DE CONFIGURACIÓN DEL EJE Z
// =======================================================================
#define Z_STEPS_DOWN 12600 // cantidad de pasos para bajar
#define Z_DELAY 100        // delay entre movimientos para darle tiempo al iman

```
```c
// 🔹 NIVELES LÓGICOS DE ENABLE TMC2209
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
enum class Command {
    STATUS,
    RESET,
    HOME1,
    HOME2,
    HOME3,
    HOME_ALL,
    ANGLE1,
    ANGLE1_STREAM,
    ANGLE2,
    ANGLE2_STREAM,
    MOVE,
    PICK,
    PLACE,
    STOP_STREAM,
    UNKNOWN
};

// Estados de homing, motor1-2 (CW = ClockWise, CCW = CounterClockWise)
// EDGE = Flanco del imán
// Reverse = invertir dirección
enum class HomingStateXY {
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
// Estados de homing, motor3
enum class HomingStateZ {
    INACTIVE,
    FIND_EDGE_DOWNWARD,
    FIND_EDGE_UPWARD,
    MOVE_TO_REFERENCE,
    OK,
    ERROR
};

// Máquina de estado para HOME_ALL
enum class HomeAllState {
    IDLE,
    MOTOR1,
    MOTOR2,
    MOTOR3,
    DONE
};

// Maquina de estado para Homing de Motores individuales
enum class HomeSingleState {
    IDLE,
    RUNNING,
    DONE
};

enum class MotorID {
    NONE,
    J1,
    J2,
    Z
};
```
```c
// 🔹 ESTRUCTURAS DE CONFIGURACIÓN DE MOTORES
// =======================================================================
// Se utiliza cuando necesitamos agrupar variables relacionadas.
// Incluye todos los parámetros mecánicos y de velocidad necesarios para cada motor.
// El punto . delante de cada nombre de campo dentro de la inicialización
// de la estructura se llama “designated initializer” o inicializador designado.
// Significa que le estás diciendo explícitamente a qué campo de la estructura
// va cada valor, sin importar el orden.
// `struct` → miembros públicos por defecto.
// `class` → miembros privados por defecto.

struct MotorConfig {
    // Mecanica
    int microstepping;      // Microstepping del driver
    int reduction;          // Relación de reducción mecánica
    int stepsPerRevolution; // Pasos por revolución del motor

    // Homing
    float slowSpeed;       // Velocidad lenta durante homing
    float fastSpeed;       // Velocidad rápida durante homing
    long steps90Deg;       // Pasos equivalentes a 90°
    long stepsLimit;       // Límite máximo de pasos
    unsigned long timeout; // Tiempo máximo permitido en homing

    // Movimiento a coordenadas
    float baseSpeed;    // Velocidad máxima de referencia
    float acceleration; // Aceleración máxima

    // Pines
    int enablePin;
};

// Configuracion de Homing para cada motor, con parámetros mecánicos específicos
// inline para funciones le dice al compilador:
// “en vez de llamar a la función, copiá su código directamente acá” (mas rapido)
// inline par variables (C++17+, C++ moderno) const constexpr:
// Permite que la misma variable se defina en varios .cpp 
// sin generar error de “multiple definition”.
// El linker trata todas las definiciones como una sola instancia.

// Cada .cpp que incluya config.h ve la variable, pero el linker no se queja de duplicados.
// Si lo ponés en un .h incluido en varios .cpp → 💥 error de multiple definition

// Regla práctica
// Constantes, structs pequeños o configuraciones como motor1Config → inline en config.h
// Variables grandes, objetos dinámicos, funciones complejas → .cpp + extern en .h
// para mejor estructura -> declarar en .h y definir en .cpp usando extern
// ✅ mejor opcion -> inline constexpr calcula todo en tiempo de compilacion y no en runtime del codigo
// ⚡️ mucho mas rapido que const o inline const

// Configuracion de Homing para cada motor, con parámetros mecánicos específico
inline constexpr MotorConfig motor1Config = {
    .microstepping = 16,
    .reduction = 9,
    .stepsPerRevolution = 200,
    .slowSpeed = 800.0,
    .fastSpeed = 1500.0,
    .steps90Deg = 16 * 200 / 4,
    .stepsLimit = 0, // no existe para motor1
    .timeout = 15000,
    .baseSpeed = BASE_SPEED,
    .acceleration = 1000.0,
    .enablePin = MOTOR1_ENABLE};

inline constexpr MotorConfig motor2Config = {
    .microstepping = 16,
    .reduction = 6,
    .stepsPerRevolution = 200,
    .slowSpeed = 533.0,
    .fastSpeed = 1000.0,
    .steps90Deg = 16 * 200 / 4,
    .stepsLimit = 0, // no existe para motor2
    .timeout = 15000,
    .baseSpeed = BASE_SPEED,
    .acceleration = 1000.0,
    .enablePin = MOTOR2_ENABLE};

inline constexpr MotorConfig motor3Config = {
    .microstepping = 8,
    .reduction = 1,
    .stepsPerRevolution = 200,
    .slowSpeed = 2500.0,
    .fastSpeed = 4000.0,
    .steps90Deg = 0,    // no existe para motor3
    .stepsLimit = -100, // pasos máximos si arranca fuera del imán (solo motor3)
    .timeout = 12000,
    .baseSpeed = BASE_SPEED,
    .acceleration = 1000.0,
    .enablePin = MOTOR3_ENABLE};
```

--------------------------------------------------------------------------


### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> core.h**
...

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> core.cpp**

...


--------------------------------------------------------------------------

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> homing.h**

- Declarar la estructura de estado del homing
- Proporcionar funciones públicas para inicializar y ejecutar la
  rutina de homing.
- Permitir consultar el estado actual y detectar errores.
- Servir como interfaz para otros módulos del robot que requieran
  funcionalidad de homing.

```c
#pragma once

#include <Arduino.h>

#include <AccelStepper.h>

#include "config.h"
```
```c
// 🔹 ESTRUCTURA DE ESTADO DEL HOMING
// =======================================================================
// Esta estructura guarda TODO el estado necesario para ejecutar
// una rutina de homing no bloqueante.
// Se pasa por referencia entre homingInit, homingStart,
// homingStep y las funciones de consulta.
// Permite manejar múltiples motores con la misma lógica.
struct HomingXY {
    HomingStateXY state;     // Estado actual de la máquina de estados de homing// <-- aquí usamos el enum
    unsigned long startTime; // Tiempo (millis) en el que comenzó el homing
    long firstEdge;          // Primer flanco detectado por el sensor
    long secondEdge;         // Segundo flanco detectado por el sensor
    long centerPosition;     // Posición calculada a partir de los flancos -> referencia absoluta
    bool fault;              // Flag de error latcheado, permanece activo hasta que el usuario lo resetea
};

struct HomingZ {
    HomingStateZ state;      // Estado actual de la máquina de estados de homing// <-- aquí usamos el enum
    unsigned long startTime; // Tiempo (millis) en el que comenzó el homing
    long initialPosition;    // Posición calculada a partir de los flancos -> referencia absoluta
    long edge;               // Flanco de salida detectado por el sensor
    long reference;          // Posiciónde homing calculada
    bool fault;              // Flag de error latcheado, permanece activo hasta que se resetee
};

extern HomingXY motor1Homing;
extern HomingXY motor2Homing;
extern HomingZ motor3Homing;

extern HomeAllState homeAllState;
extern HomeSingleState homeSingleState;
extern MotorID motorToHome;
extern bool homeAllActive;
```
```c
// 🔹 API PÚBLICA (Application Programming Interface)
// =======================================================================
void homingInitXY(HomingXY &st);

void homingInitZ(HomingZ &st);

void homingStartXY(AccelStepper &motor,
                   const MotorConfig &cfg,
                   HomingXY &st,
                   int hallPin);

void homingStartZ(AccelStepper &motor,
                  const MotorConfig &cfg,
                  HomingZ &st,
                  int hallPin);

bool homingXYisActive(const HomingXY &st);

bool homingZisActive(const HomingZ &st);

void homingStepXY(AccelStepper &motor,
                  const MotorConfig &cfg,
                  HomingXY &st,
                  int hallPin);

void homingStepZ(AccelStepper &motor,
                 const MotorConfig &cfg,
                 HomingZ &st,
                 int hallPin);

bool homingXYhasError(const HomingXY &st);

bool homingZhasError(const HomingZ &st);

HomingStateXY homingGetStateXY(const HomingXY &st);

HomingStateZ homingGetStateZ(const HomingZ &st);
```



### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> homing.cpp**

- Ejecutar la máquina de estados de homing
  para motor1-2 (XY) y motor3 (Z).
- Detectar flancos del imán mediante sensor Hall.
- Calcular el centro/referencia y posicionar motores.
- Gestionar errores y timeouts de homing.

```c
#include <Arduino.h>

#include <AccelStepper.h>

#include "config.h"
#include "homing.h"
#include "sensors.h"
```
```c
// 🔹 CONSTANTES INTERNAS DEL MÓDULO
// =======================================================================
// static → visibles solo dentro de este archivo (.cpp)
static const int8_t CW = 1;   // ClockWise plano XY
static const int8_t CCW = -1; // Counter-ClockWise plano XY
static const int8_t dir = 1;  // Dirección inicial eje Z
```
```c
// 🔹 VARIABLES INTERNAS DEL MODULO
// =======================================================================
// Instancias del controlador de homing
// Estos objetos se encarga de ejecutar la secuencia completa de homing
// incluyendo movimiento hacia el endstop, detección y ajuste fino.
HomingXY motor1Homing;
HomingXY motor2Homing;
HomingZ motor3Homing;

// Estado global del proceso de homing de TODOS los motores.
// Se inicializa en IDLE (reposo), indicando que no hay homing en curso.
// Este estado controla la máquina de estados general (secuencial o paralela).
HomeAllState homeAllState = HomeAllState::IDLE;

// Estado del proceso de homing de UN SOLO motor.
// Permite ejecutar homing individual sin afectar a los demás motores.
// También inicia en IDLE, esperando que se le asigne una acción.
HomeSingleState homeSingleState = HomeSingleState::IDLE;

// Identificador del motor que se va a hacer homing de forma individual.
// Se usa junto con homeSingleState para saber a qué motor aplicar la secuencia.
// Se inicializa en NONE para indicar que ningún motor está seleccionado.
MotorID motorToHome = MotorID::NONE;
```
```c
// 🔹 INICIALIZACIÓN DEL ESTADO DE HOMING
// =======================================================================
// Inicializa la estructura de homing dejando todos los parámetros 
// en un estado seguro y conocido antes de iniciar el proceso.
void homingInitXY(HomingXY &st) {
    st.state = HomingStateXY::INACTIVE; // estado inicial: homing apagado
    st.startTime = 0;                   // timestamp de inicio de homing
    st.firstEdge = 0;                   // primer flanco (entrada o salida)
    st.secondEdge = 0;                  // segundo flanco detectado
    st.centerPosition = 0;              // centro calculado entre flancos
    st.fault = false;                   // no hay error latcheado
}

void homingInitZ(HomingZ &st) {
    st.state = HomingStateZ::INACTIVE;
    st.startTime = 0;
    st.initialPosition = 0;
    st.edge = 0;      // flanco de salida
    st.reference = 0; // referencia calculada
    st.fault = false;
}
```
```c
// 🔹 INICIO DEL PROCESO DE HOMING
// =======================================================================
// Inicia la secuencia de homing plano XY
// Configura el motor, habilita el driver y coloca la máquina de estados
// en el punto inicial del proceso de búsqueda de referencia.
// Parámetros de la función de inicio de homing para eje XY:

// Referencia al objeto que controla el motor paso a paso.
// Permite configurar velocidad, aceleración y posición durante el homing.
// Se pasa por referencia para operar directamente sobre el motor real.
AccelStepper &motor,

// Estructura de configuración del motor.
// Contiene parámetros como:
// - enablePin: pin de habilitación del driver
// - fastSpeed: velocidad de búsqueda rápida
// - acceleration: aceleración del motor
// Se usa para aplicar settings específicos durante el homing.
const MotorConfig &cfg,

// Estructura de estado del homing.
// Almacena toda la información del proceso:
// - estado actual (state)
// - tiempos (startTime)
// - detección de flancos (firstEdge, secondEdge)
// - errores (fault)
// Se pasa por referencia porque se modifica durante la ejecución.
HomingXY &st,

// Pin de entrada del sensor Hall (o endstop).
// Se utiliza para detectar la posición de referencia del eje.
// A través de este pin se leen los flancos (activación/desactivación)
// necesarios para calcular el punto central con precisión.
int hallPin

void homingStartXY(AccelStepper &motor,
                   const MotorConfig &cfg,
                   HomingXY &st,
                   int hallPin) {
    // Evita reentradas: si el homing ya está activo, no hace nada
    if (st.state != HomingStateXY::INACTIVE)
        return;

    pinMode(cfg.enablePin, OUTPUT);
    digitalWrite(cfg.enablePin, ENABLE_ACTIVE);
    digitalWrite(LED, LOW); // apaga el LED indicador al iniciar homing

    // Configuración dinámica del motor para homing y referencia de 
    // posicion actual temporal al iniciar homing
    motor.setMaxSpeed(cfg.fastSpeed);
    motor.setAcceleration(cfg.acceleration);
    motor.setCurrentPosition(0);

    // Inicializa variables internas del estado
    // Guarda el tiempo de inicio del homing.
    // Se utiliza para timeouts, diagnósticos o control de etapas.
    st.startTime = millis();

    // Limpia cualquier error previo latcheado.
    // A partir de aquí se asume que el proceso comienza en condiciones correctas.
    st.fault = false;

    // Establece el primer estado de la máquina de homing:
    // SEARCH_FAST_CW = movimiento rápido en sentido horario (ClockWise)
    // para encontrar el sensor/endstop por primera vez.
    st.state = HomingStateXY::SEARCH_FAST_CW;
}

// Analogo con homingStartXY
void homingStartZ(AccelStepper &motor,
                  const MotorConfig &cfg,
                  HomingZ &st,
                  int hallPin) {
    if (st.state != HomingStateZ::INACTIVE)
        return;

    pinMode(cfg.enablePin, OUTPUT);
    digitalWrite(cfg.enablePin, ENABLE_ACTIVE);
    digitalWrite(LED, LOW);

    motor.setMaxSpeed(cfg.fastSpeed);
    motor.setAcceleration(cfg.acceleration);
    motor.setCurrentPosition(0);

    st.startTime = millis();
    st.fault = false;
    st.state = HomingStateZ::FIND_EDGE_DOWNWARD;
}
```
```c
// 🔹 HOMING ACTIVO?
// =======================================================================
// Retorna true si la máquina de estados NO está en inactiva,
// lo que significa que hay un proceso de homing en curso.
bool homingXYisActive(const HomingXY &st) {
    return st.state != HomingStateXY::INACTIVE;
}
bool homingZisActive(const HomingZ &st) {
    return st.state != HomingStateZ::INACTIVE;
}
```
```c
// 🔹 PREGUNTAR ESTADO ACTUAL DEL HOMING
// =======================================================================
// Se usa para consultar en qué fase del proceso está el homing
// (ej: INACTIVE, SEARCH_FAST_CW, SEARCH_SLOW, CENTERING, etc.).
// No modifica el estado, solo lo expone en modo lectura.
HomingStateXY homingGetStateXY(const HomingXY &st) {
    return st.state; // devuelve el estado actual
}
HomingStateZ homingGetStateZ(const HomingZ &st) {
    return st.state;
}
```
```c
// 🔹 COMPROBAR SI HUBO ERROR EN HOMING
// =======================================================================
// Retorna true si el flag fault está activo, lo que significa
// que ocurrió una falla durante el proceso de homing (timeout,
// fallo de sensor, movimiento inválido, etc.).
bool homingXYhasError(const HomingXY &st) {
    return st.fault; // devuelve true si hubo error
}
bool homingZhasError(const HomingZ &st) {
    return st.fault;
}
```
```c
// 🔹 MÁQUINA DE ESTADOS DE HOMING
// =======================================================================
// Ejecuta solo UN paso de la máquina de estados del homing
// Esta función debe llamarse de forma cíclica para avanzar
// progresivamente en la secuencia de búsqueda y calibración.
void homingStepXY(AccelStepper &motor,
                  const MotorConfig &cfg,
                  HomingXY &st,
                  int hallPin) {

    // Lectura del sensor Hall:
    // La lógica está invertida porque se usa pull-up interno.
    // Invierte la logica del HAll
    // LOW  = imán presente (sensor activado)
    // HIGH = sin imán (sensor inactivo)
    bool imanPresente = (digitalRead(hallPin) == LOW); // activo con pull-up

    // ⏱️ Timeout de homing XY
    // Timeout global del proceso de homing XY:
    // Si el proceso tarda más que el límite definido en cfg.timeout,
    // se considera un error para evitar bloqueos infinitos.
    if (st.state != HomingStateXY::OK && st.state != HomingStateXY::ERROR) {
        // Compara el tiempo actual con el tiempo de inicio del homing.
        // Si se supera el timeout, se fuerza el estado de ERROR.
        if (millis() - st.startTime > cfg.timeout) {
            st.state = HomingStateXY::ERROR;
        }
    }

    switch (st.state) {
        case HomingStateXY::SEARCH_FAST_CW:
            // Mover rápido en sentido horario hasta detectar imán
            // o hasta alcanzar límite de 90° (cfg.steps90Deg)
            motor.setSpeed(CW * cfg.fastSpeed);
            motor.runSpeed();
            if (imanPresente)
                st.state = HomingStateXY::FIND_FIRST_EDGE_CW; // imán detectado → buscar primer flanco
            else if (motor.currentPosition() > cfg.steps90Deg)
                st.state = HomingStateXY::SEARCH_FAST_CCW; // no detectó imán → revertir dirección
            break;

        case HomingStateXY::SEARCH_FAST_CCW:
            // Mover rápido en sentido antihorario hasta detectar imán
            // o hasta límite de -90° (cfg.steps90Deg)
            motor.setSpeed(CCW * cfg.fastSpeed);
            motor.runSpeed();
            if (imanPresente)
                st.state = HomingStateXY::FIND_FIRST_EDGE_CCW; // imán detectado → buscar primer flanco
            else if (motor.currentPosition() < -cfg.steps90Deg)
                st.state = HomingStateXY::ERROR; // límite alcanzado en ambas direcciones → error
            break;

        case HomingStateXY::FIND_FIRST_EDGE_CW:
            // Detectar el primer flanco de salida horario
            motor.setSpeed(CW * cfg.slowSpeed);
            motor.runSpeed();
            if (!imanPresente) {
                st.firstEdge = motor.currentPosition(); // guardar primer flanco horario
                st.state = HomingStateXY::REVERSE_EDGE_CW;
            }
            break;

        case HomingStateXY::FIND_FIRST_EDGE_CCW:
            // Detectar primer flanco de salida antihorario
            motor.setSpeed(CCW * cfg.slowSpeed);
            motor.runSpeed();
            if (!imanPresente) {
                st.firstEdge = motor.currentPosition(); // guardar primer flanco antihorario
                st.state = HomingStateXY::REVERSE_EDGE_CCW;
            }
            break;

        case HomingStateXY::REVERSE_EDGE_CW:
            // Invertir dirección para encontrar flanco de entrada horario
            motor.setSpeed(CCW * cfg.slowSpeed);
            motor.runSpeed();
            if (imanPresente)
                st.state = HomingStateXY::FIND_SECOND_EDGE_CW;
            break;

        case HomingStateXY::REVERSE_EDGE_CCW:
            // Invertir dirección para encontrar flanco de entrada antihorario
            motor.setSpeed(CW * cfg.slowSpeed);
            motor.runSpeed();
            if (imanPresente)
                st.state = HomingStateXY::FIND_SECOND_EDGE_CCW;
            break;

        case HomingStateXY::FIND_SECOND_EDGE_CW:
            motor.setSpeed(CCW * cfg.slowSpeed);
            motor.runSpeed();
            if (!imanPresente) {
                st.secondEdge = motor.currentPosition(); // guardar segundo flanco
                st.state = HomingStateXY::CALC_CENTER;
            }
            break;

        case HomingStateXY::FIND_SECOND_EDGE_CCW:
            motor.setSpeed(CW * cfg.slowSpeed);
            motor.runSpeed();
            if (!imanPresente) {
                st.secondEdge = motor.currentPosition();
                st.state = HomingStateXY::CALC_CENTER;
            }
            break;

        case HomingStateXY::CALC_CENTER:
            st.centerPosition = (st.firstEdge + st.secondEdge) / 2;
            motor.moveTo(st.centerPosition);
            st.state = HomingStateXY::MOVE_TO_CENTER;
            break;

        case HomingStateXY::MOVE_TO_CENTER:
            motor.run();
            if (motor.distanceToGo() == 0) {
                motor.setCurrentPosition(0);
                digitalWrite(cfg.enablePin, ENABLE_INACTIVE);
                st.state = HomingStateXY::OK;
            }
            break;

        case HomingStateXY::OK:
            digitalWrite(LED, HIGH); // indicar éxito
            digitalWrite(cfg.enablePin, ENABLE_INACTIVE);
            break;

        case HomingStateXY::ERROR:
            digitalWrite(cfg.enablePin, ENABLE_INACTIVE);
            st.fault = true;
            break;

        default:
            break;
    }
}

// Analogo a homingStepXY
void homingStepZ(AccelStepper &motor,
                 const MotorConfig &cfg,
                 HomingZ &st,
                 int hallPin) {
    // Invierte la logica del HAll (imán presente = LOW)
    bool imanPresente = (digitalRead(hallPin) == LOW); // activo con pull-up

    // ⏱️ Timeout de homing Z
    if (st.state != HomingStateZ::OK && st.state != HomingStateZ::ERROR) {
        if (millis() - st.startTime > cfg.timeout) {
            st.state = HomingStateZ::ERROR;
        }
    }

    switch (st.state) {
        case HomingStateZ::FIND_EDGE_DOWNWARD:
            motor.setSpeed(dir * cfg.fastSpeed);
            motor.runSpeed();
            if (!imanPresente) {
                st.edge = motor.currentPosition();
                motor.moveTo(st.edge + 500); // avanza 500 pasos para alejarse un poquito del imán
                st.state = HomingStateZ::FIND_EDGE_UPWARD;
            }
            break;

        case HomingStateZ::FIND_EDGE_UPWARD:
            motor.setSpeed(-cfg.slowSpeed);
            motor.runSpeed();
            if (imanPresente) {
                st.edge = motor.currentPosition();
                st.state = HomingStateZ::MOVE_TO_REFERENCE;
            } else if (motor.currentPosition() <= cfg.stepsLimit) {
                st.state = HomingStateZ::ERROR;
            }
            break;

        case HomingStateZ::MOVE_TO_REFERENCE:
            motor.setSpeed(-cfg.slowSpeed);
            motor.runSpeed();
            if (motor.currentPosition() <= st.edge - 500) {
                motor.stop();
                motor.setCurrentPosition(0);
                digitalWrite(cfg.enablePin, ENABLE_INACTIVE);
                st.state = HomingStateZ::OK;
            }
            break;

        case HomingStateZ::OK:
            digitalWrite(LED, HIGH);
            digitalWrite(cfg.enablePin, ENABLE_INACTIVE);
            break;

        case HomingStateZ::ERROR:
            digitalWrite(cfg.enablePin, ENABLE_INACTIVE);
            st.fault = true;
            break;

        default:
            break;
    }
}
```

--------------------------------------------------------------------------


### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> motors.h**
...

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> motors.cpp**


...

--------------------------------------------------------------------------


### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> sensors.h**

- Declaración de funciones y constantes para el manejo
  de sensores del robot de ajedrez.
- Proveer prototipos de funciones de inicialización y lectura.
- Definir constantes y macros para sensores.
- Servir de interfaz clara para el resto del proyecto.

```c
#pragma once

#include <Arduino.h>
#include <Wire.h>
```
```c
// 🔹 DECLARACIONE DE VARIABLES GLOBALES (extern -> sin duplicar instancias)
// =======================================================================

extern float lastSentAngle_1;
extern unsigned long lastSendTime_1;
extern float lastSentAngle_2;
extern unsigned long lastSendTime_2;
// lastSentAngle_1 / lastSentAngle_2
// Último ángulo enviado para cada sensor. 
// Se utiliza para evitar envíos redundantes cuando no hay cambios significativos.
// lastSendTime_1 / lastSendTime_2
// Marca de tiempo del último envío de datos de cada sensor (en milisegundos).
// Permite controlar la frecuencia de transmisión.

extern float sensor1Offset;
extern float sensor2Offset;
// sensor1Offset / sensor2Offset
// Offset angular calculado durante el proceso de homing. 
// Representa la diferencia entre la posición real del sensor y 
// la posición de referencia (cero mecánico).

extern float sensor1Angle;
extern float sensor2Angle;
// sensor1Angle / sensor2Angle
// Ángulo del sensor una vez aplicado el offset de calibración. 
// Este es el valor final utilizado por el sistema de control

void sensorsInit();
// Inicializa los sensores angulares y configura la comunicación (I2C). 
// Debe llamarse al inicio del sistema.

uint16_t sensorReadAngle(TwoWire &wire);
// Lee el valor crudo del ángulo desde el sensor a través del bus I2C. 
// Devuelve el dato sin procesar.

void sensorStreamAngle(TwoWire &wire, float &lastSentAngle, unsigned long &lastSendTime)
// Envía el ángulo del sensor de forma continua, 
// controlando cambios mínimos y tiempo entre envíos para optimizar la comunicación.

void sensorSendAngle(TwoWire &wire)
// Envía el ángulo actual del sensor sin aplicar 
// lógica de filtrado o control de frecuencia.

float sensorHomingOffset(TwoWire &wire)
// Calcula el offset del sensor durante el proceso de homing. 
// Este valor se usa como referencia para corregir lecturas futuras.

float sensorCorrectedAngle(TwoWire &wire, float offset)
// Devuelve el ángulo corregido aplicando el offset previamente calculado. 
// Este valor representa la posición real del sistema.
```

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> sensors.cpp**

- Inicializar y configurar sensores.
- Implementación de funciones para la lectura y gestión de
  sensores AS5600.

#####  🔀 Una Direccion, dos buses I2C diferentes

    ✏️ Descripción:
    Direccion común pero perifericos I2C distintos
    Porque aunque ambos sensores tengan dirección 0x36, están en buses físicos diferentes. Es como tener dos calles distintas con la misma numeración

    🧩 Implementación:
    Sensor 1 → pines 4 y 5
    Sensor 2 → pines 26 y 27
    En el RP2040 eso significa que se están usando dos controladores I2C distintos:
    Wire (I2C0)
    Wire1 (I2C1)

    ⚠️ La direccion es la misma para ambos sensores:

```c
#include <Arduino.h>

#include "config.h"
#include "sensors.h"
#include "utils.h"
```
```c
// 🔹 VARIABLES INTERNAS DEL MÓDULO ❌ Mover estas variables a core❓
// ⚠️ Nota: Estas variables actualmente están dentro del módulo de sensores, 
// pero pueden ser movidas al core del sistema si se requiere centralizar 
// el estado global.
// =======================================================================

unsigned long lastSendTime_1 = 0;
float lastSentAngle_1 = -1000.0f; 
unsigned long lastSendTime_2 = 0;
float lastSentAngle_2 = -1000.0f;
// lastSendTime_1 / lastSendTime_2
// Almacenan el tiempo (en milisegundos) del último envío de datos de cada sensor.
// Se utilizan para limitar la frecuencia de transmisión y evitar saturar la comunicación.
// lastSentAngle_1 / lastSentAngle_2
// Guardan el último ángulo enviado.
// Se emplean para detectar cambios y evitar envíos redundantes 
// si el valor no varía significativamente.
// Inicializados en -1000.0 como valor imposible, 
// garantizando que el primer envío siempre se realice.
// ¿Por qué -1000?
// Porque es un valor imposible (el AS5600 solo va de 0 a 360).
// Entonces el primer envío siempre ocurre.

float sensor1Offset = 0;
float sensor2Offset = 0;
// sensor1Offset / sensor2Offset
// Offset angular calculado durante el proceso de homing.
// Representa la diferencia entre la posición física real y la referencia de cero del sistema.

float sensor1Angle = 0;
float sensor2Angle = 0;
// sensor1Angle / sensor2Angle
// Ángulo actual del sensor después de aplicar el offset.
// Este valor ya está listo para ser usado por el sistema de control de movimiento.
```
 ##### Este diagrama explica cómo se decide enviar un ángulo del sensor AS5600 solo cuando cambia:

<p align="center">
  <img src="img/diagrama_de_angulos.png" alt="Diagrama de envío de ángulo" width="400">
</p>

```c
// 🔹 INICIALIZACIÓN DEL MÓDULO DE SENSORES
// =======================================================================
void sensorsInit() {
    pinMode(HALL_1, INPUT_PULLUP);
    pinMode(HALL_2, INPUT_PULLUP);
    pinMode(HALL_3, INPUT_PULLUP);

    Wire.setSDA(AS5600_1_SDA);
    Wire.setSCL(AS5600_1_SCL);
    Wire.begin();
    Wire1.setSDA(AS5600_2_SDA);
    Wire1.setSCL(AS5600_2_SCL);
    Wire1.begin();
}
```
```c

// 🔹 LECTURA DE SENSORES (BAJO NIVEL)
// =======================================================================
// Lee el ángulo crudo desde el sensor AS5600 vía I2C
// TwoWire → le decimos “la función va a recibir un objeto de tipo TwoWire”.
// &wire → le decimos “pasalo por referencia, no por copia”.
uint16_t sensorReadAngle(TwoWire &wire) {
    // Inicia la comunicación con el sensor y selecciona el registro 0x0E, 
    // donde se encuentra el valor del ángulo.
    wire.beginTransmission(AS5600_ADDR);
    wire.write(0x0E);
    wire.endTransmission(false);

    // Solicita 2 bytes de datos desde el sensor 
    // (el ángulo está representado en 12 bits distribuidos en dos bytes).
    wire.requestFrom(AS5600_ADDR, (uint8_t)2);

    if (wire.available() < 2)
        return 0;
    // Verifica que la lectura fue exitosa.
    // Si no hay suficientes datos disponibles, devuelve 0 como valor de seguridad.

    uint8_t high = wire.read();
    uint8_t low = wire.read();
    // Lee los dos bytes recibidos:
    // high: parte alta del ángulo
    // low: parte baja del ángulo

    return ((high & 0x0F) << 8) | low;
    // Une los dos bytes recibidos para formar el valor del ángulo del sensor
    // El sensor envía el ángulo dividido en dos partes (alto y bajo)
    // Solo 12 bits contienen información útil del ángulo
    // Se descartan los bits sobrantes del byte alto
    // Luego se combinan ambos bytes en un solo número
}
```
```c
// 🔹 DEBUG / PRUEBAS (NO USAR EN PRODUCCIÓN)
// =======================================================================
// ⚠️ Solo para pruebas de lectura de angulo
// Las funciones de DEBUG usan Serial1 (bloqueante).
// No deben utilizarse durante el control de movimiento en tiempo real,
// ya que pueden introducir latencias y afectar la estabilidad del sistema.
void sensorSendAngle(TwoWire &wire) {
    uint16_t rawAngle = sensorReadAngle(wire); // Leer sensor AS5600
    float degrees = rawToDegrees(rawAngle);    // Convertir a grados
    degrees = round1Decimal(degrees);          // Redondear a 1 decimal
    Serial1.print(degrees, 1);                 // Asegurar envio de solo 1 decimal
    Serial1.print("\n");
}

// ⚠️ Solo para pruebas de lectura de angulo continuo
// ⚠️ Streaming continuo (puede afectar control de motores)
// -> El envio continuo bloquea movimiento de motores
void sensorStreamAngle(TwoWire &wire, float &lastSentAngle, unsigned long &lastSendTime) {
    uint16_t rawAngle = sensorReadAngle(wire);                                 // Leer sensor AS5600
    float degrees = rawToDegrees(rawAngle);                                    // Convertir a grados
    degrees = round1Decimal(degrees);                                          // Redondear a 1 decimal
    sendFilteredFloat(degrees, lastSentAngle, lastSendTime, 0.5, 33, Serial1); // Enviar angulo filtrado por UART
}
// 🧠 Resumen conceptual
// Estas 4 líneas hacen un filtro doble:

// 1️⃣ Filtro por cambio mínimo → evita microvariaciones
// 2️⃣ Filtro por tiempo → evita saturar el bus
// - Sin esto pasaba:
// AS5600 → 1000 lecturas por segundo
// UART → se llena
// Pico → recibe mezclado
// OLED → parpadea y números raros
// - Con esto pasa:
// AS5600 → 1000 lecturas
// Zero → filtra
// UART → 30 limpias por segundo
// Pico → feliz
// OLED → fluido
```
```c
// 🔹 CALCULO DE OFFSET PARA CODIFICADOR AS5600
// =======================================================================
// Si alguna vez tu homing mecanico cae cerca del 0 digital del sensor:
// entonces el promedio normal se rompe
// Mejor solucion:
// 30 lecturas + corrección si cruza 0° + promedio 👀💡
//         0°
//    330°     30°
//  300°         60°
//  270°         90°
//  240°        120°
//   210°      150°
//         180°
// Calcula el offset (cero mecánico) del sensor usando varias mediciones.
// Toma varias lecturas del ángulo cuando el sistema está 
// en la posición de referencia (homing)
// Luego calcula el valor promedio para definir ese punto como “cero”

float sensorHomingOffset(TwoWire &wire) {
    const uint8_t samples = 30;
    float sum = 0;
    // Define cuántas mediciones se van a tomar y una variable para acumular resultados

    float firstAngle = rawToDegrees(sensorReadAngle(wire));
    // Toma una primera lectura como referencia inicial

    // Repite la lectura varias veces para reducir ruido del sensor
    for (uint8_t i = 0; i < samples; i++) {
        float angle = rawToDegrees(sensorReadAngle(wire));
        // Corrige el problema de salto entre 0° y 360°
        // (Ejemplo: evita que 359° y 1° se interpreten como valores muy lejanos)
        if (fabs(angle - firstAngle) > 180) {
            if (angle < firstAngle)
                angle += 360;
            else
                angle -= 360;
        }
        sum += angle;
    }

    // Calcula el promedio final → este es el offset del sistema   
    // Asegura que el resultado quede siempre dentro de 0° a 360°
    float offset = sum / samples;
    if (offset >= 360)
        offset -= 360;
    if (offset < 0)
        offset += 360;
    return offset;
}

// Convierte la lectura del sensor en el ángulo real del sistema, 
// aplicando el offset de homing
// Ajusta el resultado para que siempre quede entre 0° y 360°
float sensorCorrectedAngle(TwoWire &wire, float offset) {
    float angle = rawToDegrees(sensorReadAngle(wire)) - offset;
    if (angle >= 360)
        angle -= 360;
    if (angle < 0)
        angle += 360;
    return angle;
}
```





--------------------------------------------------------------------------

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> utils.h**

...

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> utils.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> xy_plane.h**

...
### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> xy_plane.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> z_Axis.h**

...
### **<img src="img/c++.png" width="30" style="position: relative; top: 4px;"> z_Axis.cpp**

...

--------------------------------------------------------------------------

### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> utils.h**

...
### **<img src="img/c++.png" width="20" style="position: relative; top: 4px;"> utils.cpp**

...
