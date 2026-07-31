# Entrada 01 - ¿Por qué existe ROS 2?

## Objetivo

Antes de escribir código, quiero entender qué problema intenta resolver ROS 2
y por qué su arquitectura está dividida en componentes pequeños.

---

## La pregunta que me hice

Cuando empecé a leer sobre ROS 2 encontré términos como:

- Node
- Topic
- Message
- Service
- Action

Pero antes de aprender esos conceptos necesitaba responder una pregunta más
básica:

> ¿Por qué necesito ROS 2 para construir un robot?

---

## El problema de un programa gigante

Imagina que quieres construir un robot con:

- Dos motores.
- Un LIDAR.
- Una cámara.
- Una IMU.
- Navegación.
- Creación de mapas.

Una primera solución podría ser escribir un único programa:

```text
robot.py

- Leer el LIDAR
- Leer la cámara
- Controlar los motores
- Calcular la odometría
- Crear el mapa
- Planificar una ruta
- Evitar obstáculos
```

Al principio puede funcionar.

El problema aparece cuando el proyecto empieza a crecer. Todo termina
dependiendo de todo y cualquier cambio puede afectar partes que no esperabas.

Por ejemplo, si el LIDAR se daña y lo reemplazas por otro modelo, podrías
tener que revisar:

- Cómo se conecta.
- Qué datos entrega.
- Qué unidades utiliza.
- Qué frecuencia de actualización tiene.
- Qué partes del programa dependen directamente de él.

El problema no es solamente cambiar el sensor.

**Piensa un momento**

Si cambiar un sensor obliga a modificar medio proyecto...

¿el problema será el sensor o la arquitectura?

---

## La idea detrás de ROS 2

ROS 2 propone dividir el robot en programas pequeños que colaboran entre sí.

Cada programa se encarga de una responsabilidad concreta.

Por ejemplo:

```text
Programa del LIDAR
Programa de motores
Programa de odometría
Programa de navegación
Programa de teleoperación
```

Más adelante aprendí que estos programas reciben el nombre de **Nodes**.

Por ahora, la idea importante es esta:

> Un problema grande resulta más fácil de construir y mantener cuando se
> divide en problemas pequeños con responsabilidades claras.

---

## ¿Qué ventaja tiene esta separación?

Si cada componente está bien separado, cambiar un sensor no debería obligar a
reescribir todo el robot.

Por ejemplo:

```text
LIDAR A ──┐
          ├── datos de escaneo ──> navegación
LIDAR B ──┘
```

El programa que se comunica con el hardware puede cambiar, mientras que el
resto del sistema sigue trabajando con una estructura conocida.

Esto también ayuda a:

- Probar cada parte por separado.
- Reutilizar componentes.
- Detectar errores con mayor facilidad.
- Trabajar con varias personas.
- Evitar que todo el sistema dependa de un único programa.

---

## Lo que yo pensaba

Al principio pensé que ROS 2 era un sistema operativo completo para robots.

Después entendí que es un conjunto de herramientas, librerías y mecanismos de
comunicación que ayudan a construir software robótico de forma organizada.

---

## Lo que aprendí

ROS 2 no existe solamente para ejecutar comandos o lanzar simulaciones.

Su propósito principal es ayudar a construir sistemas robóticos modulares,
donde cada componente tenga una responsabilidad clara y pueda comunicarse con
los demás.

---

## Siguiente entrada

Ahora que entiendo el problema, la siguiente pregunta es:

> Si voy a construir varios programas para mi robot...
>
> **¿Dónde debería organizar todo ese código?**

En la siguiente entrada conoceremos el concepto de **Workspace**.

[Entrada 02 - Packages](docs/02-Workspace.md)