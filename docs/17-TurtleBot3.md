# Entrada 17 - TurtleBot3

## Objetivo

En las entradas anteriores estudiamos varias herramientas fundamentales de ROS 2:

- TF2.
- URDF.
- RViz.
- Gazebo.

Hasta este punto analizamos principalmente cómo ROS 2 representa, describe y visualiza un robot.

Ahora podemos unir todos estos conceptos utilizando una plataforma robótica real:

> **TurtleBot3 Burger**

El objetivo de esta entrada es comprender cómo está organizado TurtleBot3, cuáles son sus principales componentes y cómo ROS 2 permite comunicarnos con el robot.

---

# ¿Por qué TurtleBot3?

Cuando empecé a trabajar con ROS 2 apareció una pregunta importante:

> **¿Cómo se aplican realmente todos estos conceptos en un robot físico?**

Hasta ahora vimos que:

```text
URDF
  ↓
describe el robot

TF2
  ↓
relaciona sus sistemas de coordenadas

RViz
  ↓
permite visualizar información

Gazebo
  ↓
permite simular el robot
```

Pero necesitábamos una plataforma donde todos estos elementos trabajaran juntos.

Aquí aparece **TurtleBot3**.

---

# ¿Qué es TurtleBot3?

TurtleBot3 es una plataforma de robot móvil utilizada para educación, investigación y desarrollo con ROS.

Existen diferentes versiones de TurtleBot3.

Entre ellas:

```text
TurtleBot3 Burger
TurtleBot3 Waffle
TurtleBot3 Waffle Pi
```

En nuestro caso trabajaremos con:

```text
TurtleBot3 Burger
```

Es un robot móvil de tipo diferencial.

Esto significa que utiliza principalmente dos ruedas motrices independientes.

```text
        Frente
          ↑

     ┌─────────┐
     │  LIDAR  │
     │         │
 O───│  ROBOT  │───O
     │         │
     └─────────┘

  rueda       rueda
 izquierda    derecha
```

Controlando la velocidad de estas dos ruedas podemos controlar el movimiento del robot.

---

# Movimiento diferencial

El TurtleBot3 Burger tiene:

```text
Motor izquierdo
Motor derecho
```

Si ambos motores giran aproximadamente a la misma velocidad:

```text
Izquierda → →
Derecha   → →

Robot → avanza
```

Si ambos giran en sentido contrario:

```text
Izquierda ← ←
Derecha   ← ←

Robot → retrocede
```

Si las velocidades son diferentes:

```text
Izquierda → →
Derecha   →
```

el robot comienza a realizar una curva.

También puede realizar giros sobre su propio eje haciendo que las ruedas giren en sentidos opuestos.

```text
Izquierda ←
Derecha   →

       ↻
```

Esta configuración recibe el nombre de:

> **Differential Drive**

---

# Componentes principales del TurtleBot3 Burger

Al observar el robot físicamente podemos identificar varios componentes importantes.

De forma simplificada:

```text
             ┌──────────────┐
             │    LIDAR     │
             └──────┬───────┘
                    │
             ┌──────▼───────┐
             │ Raspberry Pi │
             └──────┬───────┘
                    │
             ┌──────▼───────┐
             │    OpenCR    │
             └──────┬───────┘
                    │
           ┌────────┴────────┐
           │                 │
     ┌─────▼─────┐     ┌─────▼─────┐
     │ Dynamixel │     │ Dynamixel │
     │ izquierdo │     │  derecho  │
     └───────────┘     └───────────┘
```

Cada elemento tiene una función diferente.

---

# Raspberry Pi

La Raspberry Pi funciona como uno de los computadores principales del robot.

En nuestro caso estamos trabajando con:

```text
Raspberry Pi 4 Model B
```

En ella se encuentra instalado:

```text
Ubuntu
ROS 2
paquetes de TurtleBot3
```

Por esta razón podemos conectarnos remotamente al robot y ejecutar comandos de ROS 2.

---

# Conexión mediante SSH

Como la Raspberry Pi puede trabajar sin monitor ni teclado conectados directamente, podemos controlarla desde otro computador utilizando **SSH**.

La arquitectura queda aproximadamente así:

```text
┌─────────────────────┐
│      Computador     │
│                     │
│ Terminal / RViz     │
└──────────┬──────────┘
           │
           │ Wi-Fi
           │
           ▼
┌─────────────────────┐
│    Raspberry Pi     │
│    TurtleBot3       │
│                     │
│       ROS 2         │
└─────────────────────┘
```

Para conectarnos utilizamos:

```bash
ssh ubuntu@IP_DEL_ROBOT
```

La dirección IP depende de la red a la cual se encuentre conectado el TurtleBot3.

---

# OpenCR

Otro componente fundamental es la tarjeta **OpenCR**.

Podemos pensar inicialmente en ella como el puente entre ROS 2 y parte del hardware del robot.

Una representación simplificada sería:

```text
ROS 2
 │
 ▼
Raspberry Pi
 │
 ▼
OpenCR
 │
 ├──────────► Motor izquierdo
 │
 └──────────► Motor derecho
```

La Raspberry Pi ejecuta los procesos de ROS 2 mientras OpenCR participa en el control del hardware de bajo nivel.

---

# Motores Dynamixel

El TurtleBot3 Burger utiliza motores Dynamixel.

Estos motores permiten controlar las ruedas del robot.

Tenemos principalmente:

```text
Motor izquierdo
Motor derecho
```

ROS 2 no necesita que nosotros controlemos directamente cada señal eléctrica del motor.

En lugar de eso podemos trabajar con comandos de velocidad.

Por ejemplo:

```text
/cmd_vel
```

---

# El LIDAR

Otro componente fundamental del TurtleBot3 es el LIDAR.

LIDAR significa:

```text
Light Detection and Ranging
```

Este sensor permite medir distancias alrededor del robot.

Conceptualmente realiza algo parecido a:

```text
             obstáculo
                █
                █
                █

        ← ← ←   │   → → →

              LIDAR
                ●
           ↙    ↓    ↘
```

El sensor realiza mediciones alrededor del robot y genera información sobre las distancias a los objetos.

En ROS 2 esta información normalmente puede encontrarse en un tópico como:

```text
/scan
```

Podemos comprobarlo utilizando:

```bash
ros2 topic list
```

y posteriormente:

```bash
ros2 topic echo /scan
```

---

# ¿Cómo inicia TurtleBot3 en ROS 2?

Una vez conectado al robot mediante SSH debemos cargar nuestro entorno.

Por ejemplo:

```bash
source /opt/ros/humble/setup.bash
```

También debemos indicar qué modelo de TurtleBot3 estamos utilizando:

```bash
export TURTLEBOT3_MODEL=burger
```

En nuestro caso:

```text
burger
```

Después podemos iniciar los nodos necesarios para trabajar con el robot.

```bash
ros2 launch turtlebot3_bringup robot.launch.py
```

---

# ¿Qué significa bringup?

Al principio el término **bringup** puede resultar extraño.

Una forma sencilla de entenderlo es:

> Bringup inicia los componentes necesarios para que ROS 2 pueda comunicarse con el robot físico.

Cuando ejecutamos:

```bash
ros2 launch turtlebot3_bringup robot.launch.py
```

se ponen en funcionamiento diferentes elementos relacionados con:

- motores;
- sensores;
- estado del robot;
- odometría;
- transformaciones;
- comunicación con el hardware.

Por eso `bringup` es uno de los primeros pasos cuando queremos utilizar el TurtleBot3 real.

---

# Verificar los nodos

Con el robot iniciado podemos abrir otra terminal y consultar:

```bash
ros2 node list
```

Esto permite observar los nodos que están funcionando.

La arquitectura comienza a verse así:

```text
             ROS 2

       ┌───────┼────────┐
       │       │        │
       ▼       ▼        ▼
   sensores  estado   control
       │       │        │
       └───────┼────────┘
               │
               ▼
           TurtleBot3
```

---

# Verificar los tópicos

También podemos consultar:

```bash
ros2 topic list
```

Aquí aparecen tópicos importantes.

Entre ellos podemos encontrar conceptos como:

```text
/cmd_vel
/odom
/scan
/joint_states
/tf
/tf_static
```

Cada uno transporta información diferente.

---

# /cmd_vel

Uno de los tópicos más importantes para un robot móvil es:

```text
/cmd_vel
```

Su nombre puede interpretarse como:

```text
command velocity
```

es decir:

> comando de velocidad.

Los mensajes enviados a este tópico permiten indicar cómo queremos que se mueva el robot.

Conceptualmente:

```text
            /cmd_vel
                │
                ▼
        ┌───────────────┐
        │   TurtleBot3  │
        └───────────────┘
                │
         ┌──────┴──────┐
         ▼             ▼
      rueda          rueda
    izquierda       derecha
```

---

# Velocidad lineal y angular

Para controlar un robot diferencial normalmente trabajamos con dos variables principales:

```text
velocidad lineal
velocidad angular
```

La velocidad lineal determina principalmente cuánto avanza o retrocede el robot.

```text
linear.x > 0

      ↑
      │
    ROBOT
```

Mientras que la velocidad angular determina cuánto gira.

```text
angular.z > 0

       ↺
     ROBOT
```

Combinando ambas podemos generar diferentes trayectorias.

---

# Teleoperación

Una forma sencilla de comprobar el movimiento del robot es mediante teleoperación.

Podemos utilizar:

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

Este nodo permite enviar comandos utilizando el teclado.

La comunicación queda aproximadamente así:

```text
TECLADO
   │
   ▼
turtlebot3_teleop
   │
   ▼
/cmd_vel
   │
   ▼
TurtleBot3
   │
   ▼
MOTORES
```

De esta forma podemos mover manualmente el robot.

---

# /odom

Otro tópico fundamental es:

```text
/odom
```

Este tópico está relacionado con la **odometría**.

La odometría permite estimar cómo se está desplazando el robot utilizando información relacionada con el movimiento de las ruedas.

Podemos imaginar:

```text
Posición inicial

      ●

      │
      │ movimiento
      ▼

             ●
       posición estimada
```

ROS 2 mantiene una estimación de variables como:

```text
x
y
orientación
velocidad
```

Esta información será especialmente importante cuando trabajemos posteriormente con navegación.

---

# /joint_states

También encontramos:

```text
/joint_states
```

Este tópico contiene información sobre las articulaciones del robot.

En TurtleBot3 las ruedas están representadas mediante joints.

Por ejemplo:

```text
wheel_left_joint
wheel_right_joint
```

Esta información puede ser utilizada junto con la descripción URDF.

Aquí podemos conectar lo aprendido anteriormente:

```text
URDF
 +
joint_states
 +
robot_state_publisher
        │
        ▼
       TF2
        │
        ▼
      RViz
```

Ahora podemos ver que los temas anteriores no estaban aislados.

Todos forman parte del funcionamiento del robot.

---

# /tf y /tf_static

También podemos encontrar:

```text
/tf
/tf_static
```

Estos tópicos están relacionados con TF2.

Recordemos que TF2 permite conocer la relación entre diferentes sistemas de coordenadas.

En TurtleBot3 podemos encontrar frames similares a:

```text
odom
 │
 ▼
base_footprint
 │
 ▼
base_link
 │
 ├────────► base_scan
 │
 ├────────► wheel_left_link
 │
 └────────► wheel_right_link
```

Esto permite que ROS 2 comprenda dónde se encuentran los diferentes componentes del robot.

---

# Conexión con RViz

Ahora podemos entender por qué RViz resulta tan útil.

RViz puede recibir información proveniente del TurtleBot3 y representarla gráficamente.

Por ejemplo:

```text
TurtleBot3 físico
      │
      ├──── /scan
      ├──── /odom
      ├──── /joint_states
      └──── /tf
             │
             ▼
            RViz
```

De esta manera podemos observar desde nuestro computador información generada por el robot físico.

---

# Ver el LIDAR en RViz

Una de las pruebas más interesantes consiste en visualizar las mediciones del LIDAR.

El sensor publica información aproximadamente mediante:

```text
/scan
```

RViz puede utilizar esta información mediante una visualización de tipo:

```text
LaserScan
```

Entonces tenemos:

```text
LIDAR
  │
  ▼
/scan
  │
  ▼
ROS 2
  │
  ▼
RViz
```

Los obstáculos detectados alrededor del robot pueden aparecer gráficamente.

---

# Comunicación completa del sistema

Después de analizar los componentes podemos representar el sistema completo.

```text
                    COMPUTADOR
                 ┌─────────────┐
                 │    RViz     │
                 │   Teleop    │
                 └──────┬──────┘
                        │
                       Wi-Fi
                        │
                        ▼
               ┌────────────────┐
               │  Raspberry Pi  │
               │                │
               │     ROS 2      │
               └───────┬────────┘
                       │
                       ▼
                 ┌──────────┐
                 │  OpenCR  │
                 └────┬─────┘
                      │
             ┌────────┴────────┐
             ▼                 ▼
        Motor izquierdo   Motor derecho

               Raspberry Pi
                      │
                      ▼
                    LIDAR
                      │
                      ▼
                    /scan
```

Esta arquitectura nos permite comprender mejor qué ocurre cuando enviamos una orden desde nuestro computador.

---

# ¿Qué ocurre cuando presiono una tecla?

Supongamos que ejecutamos:

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

y ordenamos al robot avanzar.

Conceptualmente ocurre:

```text
1. Presiono una tecla
          │
          ▼
2. teleop_keyboard
          │
          ▼
3. publica /cmd_vel
          │
          ▼
4. ROS 2 recibe el comando
          │
          ▼
5. sistema de control del TurtleBot3
          │
          ▼
6. OpenCR
          │
          ▼
7. motores Dynamixel
          │
          ▼
8. ruedas
          │
          ▼
9. el robot se mueve
```

Esto muestra una de las ventajas de ROS 2:

> No necesitamos controlar directamente los motores desde nuestra aplicación.

Podemos trabajar mediante interfaces de comunicación estandarizadas.

---

# ¿Qué ocurre mientras el robot se mueve?

Mientras enviamos comandos de movimiento también se genera información.

Por ejemplo:

```text
Motores / encoders
       │
       ▼
   odometría
       │
       ▼
     /odom
```

Simultáneamente:

```text
LIDAR
  │
  ▼
/scan
```

Y la estructura del robot continúa publicándose mediante:

```text
/joint_states
/tf
/tf_static
```

Por lo tanto el robot no solamente recibe información.

También publica continuamente información sobre su estado y su entorno.

---

# ROS 2 como sistema distribuido

Una de las cosas más interesantes de esta práctica es que ROS 2 no necesita ejecutarse completamente en una sola computadora.

Podemos tener:

```text
COMPUTADOR PERSONAL
-------------------
RViz
Teleoperación
Herramientas ROS 2


        Wi-Fi
          │
          ▼


RASPBERRY PI
-------------
ROS 2
TurtleBot3 Bringup
Sensores
Control del robot
```

Ambos equipos pueden formar parte del mismo sistema ROS 2.

Esta característica será fundamental para las siguientes aplicaciones.

---

# Comandos principales utilizados

Durante las pruebas con TurtleBot3 podemos utilizar comandos como:

```bash
export TURTLEBOT3_MODEL=burger
```

Iniciar el robot:

```bash
ros2 launch turtlebot3_bringup robot.launch.py
```

Consultar nodos:

```bash
ros2 node list
```

Consultar tópicos:

```bash
ros2 topic list
```

Observar el LIDAR:

```bash
ros2 topic echo /scan
```

Observar la odometría:

```bash
ros2 topic echo /odom
```

Teleoperar:

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

---

# Algo importante que aprendí

Antes de trabajar directamente con TurtleBot3 podía parecer que conceptos como:

```text
TF2
URDF
RViz
Gazebo
Topics
Nodes
```

eran herramientas independientes.

Pero trabajando con el robot físico podemos observar que todas están relacionadas.

```text
                    TurtleBot3
                        │
        ┌───────────────┼───────────────┐
        │               │               │
       URDF            TF2           Topics
        │               │               │
        └───────────────┼───────────────┘
                        │
                       RViz
```

URDF describe la estructura.

TF2 relaciona los sistemas de coordenadas.

Los tópicos transportan información.

RViz permite visualizarla.

Y TurtleBot3 integra todos estos elementos en una plataforma física.

---

# Del robot manual al robot autónomo

Hasta este punto podemos controlar el TurtleBot3 manualmente.

Por ejemplo:

```text
Persona
   │
   ▼
Teclado
   │
   ▼
/cmd_vel
   │
   ▼
Robot
```

Pero aparece una nueva pregunta:

> **¿Qué ocurre si queremos que el robot conozca el entorno y pueda desplazarse sin que una persona lo controle continuamente?**

Para lograrlo primero necesitamos que el robot pueda construir una representación del lugar donde se encuentra.

Aquí aparece uno de los conceptos más importantes de la robótica móvil:

```text
SLAM
```

---

# Conclusión

TurtleBot3 permite llevar a una plataforma física muchos de los conceptos estudiados anteriormente.

Ahora podemos comprender una arquitectura básica como:

```text
Computador
     │
     │ Wi-Fi
     ▼
Raspberry Pi
     │
    ROS 2
     │
     ▼
OpenCR
     │
     ▼
Motores
```

mientras los sensores proporcionan información:

```text
LIDAR
  │
  ▼
/scan
  │
  ▼
ROS 2
```

Además aparecen tópicos fundamentales como:

```text
/cmd_vel
/odom
/scan
/joint_states
/tf
/tf_static
```

Con esto ya tenemos un robot capaz de:

- comunicarse mediante ROS 2;
- recibir comandos de movimiento;
- controlar sus ruedas;
- obtener información del LIDAR;
- generar odometría;
- publicar transformaciones;
- visualizar información mediante RViz.

El siguiente paso será utilizar esta información para construir un mapa del entorno.

---

# Siguiente entrada

**Entrada 18 - SLAM**

En la siguiente entrada estudiaremos cómo TurtleBot3 puede utilizar el LIDAR, las transformaciones y la odometría para construir un mapa del entorno mientras se desplaza.