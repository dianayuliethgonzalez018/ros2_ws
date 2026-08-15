# Entrada 16 - Gazebo

## Objetivo

En la entrada anterior aprendimos a utilizar **RViz** como herramienta de visualización dentro de ROS 2.

Entendimos que RViz puede mostrarnos información como:

```text
RobotModel
TF
LaserScan
Map
Odometry
Path
```

Pero también descubrimos algo importante:

> **RViz visualiza información, pero no es un simulador físico.**

Entonces apareció una nueva pregunta:

> **¿Cómo puedo probar un robot sin tener que utilizar siempre el robot físico?**

Por ejemplo, queremos comprobar qué ocurre cuando el robot:

```text
se mueve
gira
detecta una pared
choca con un objeto
utiliza su LIDAR
recorre una habitación
```

Para hacer estas pruebas necesitamos crear un entorno virtual.

Aquí aparece:

# Gazebo

---

# ¿Qué es Gazebo?

Gazebo es un simulador utilizado ampliamente en robótica.

Nos permite crear un mundo virtual donde podemos colocar robots, sensores y diferentes objetos.

Por ejemplo:

```text
┌─────────────────────────────────────┐
│                                     │
│       █████                         │
│       █   █                         │
│       █   █            ● Robot      │
│       █████                         │
│                                     │
│                     █████████       │
│                                     │
└─────────────────────────────────────┘
```

Dentro de este entorno podemos simular el comportamiento del robot antes de utilizar el hardware real.

---

# La diferencia entre RViz y Gazebo

Esta fue una de las diferencias más importantes que tuve que entender.

Podemos resumirla así:

```text
RViz
 │
 ▼
Visualiza datos de ROS 2


Gazebo
 │
 ▼
Simula un entorno físico
```

RViz puede mostrarnos dónde se encuentra el robot.

Gazebo puede simular qué ocurre físicamente cuando ese robot intenta moverse.

Por ejemplo:

```text
                 ROS 2
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
        RViz              Gazebo
          │                 │
          ▼                 ▼
   Visualización       Simulación
```

Las dos herramientas pueden utilizarse juntas.

---

# ¿Qué puede simular Gazebo?

Dentro de Gazebo podemos trabajar con elementos como:

```text
robots
paredes
mesas
cajas
obstáculos
sensores
cámaras
LIDAR
ruedas
```

También puede representar fenómenos físicos.

Por ejemplo:

```text
gravedad
colisiones
movimiento
contacto
masa
inercia
```

Esto significa que un robot dentro de Gazebo no es simplemente una imagen.

El simulador intenta representar su comportamiento dentro de un mundo físico.

---

# Un ejemplo sencillo

Imaginemos que tenemos un robot frente a una pared.

```text
Robot                     Pared

  ●   ───────────────→      █
                            █
                            █
                            █
```

Si simplemente visualizamos el robot en RViz, podemos representar su posición.

Pero Gazebo puede simular qué ocurre cuando el robot intenta avanzar.

Cuando llega a la pared:

```text
Robot → █ Pared
```

se produce una colisión.

El robot no debería atravesarla.

Esta es una de las principales diferencias entre visualización y simulación.

---

# El mundo de Gazebo

Gazebo utiliza el concepto de:

```text
World
```

Un **World** representa el entorno donde ocurre la simulación.

Dentro del mundo podemos tener:

```text
World
 │
 ├── Robot
 │
 ├── Pared
 │
 ├── Mesa
 │
 ├── Caja
 │
 └── Otros objetos
```

Por ejemplo:

```text
┌────────────────────────────────────┐
│              WORLD                 │
│                                    │
│     ┌───────┐                      │
│     │ Mesa  │                      │
│     └───────┘                      │
│                                    │
│                 ● TurtleBot3       │
│                                    │
│                         ██████     │
│                         Pared      │
└────────────────────────────────────┘
```

Esto nos permite construir diferentes escenarios para probar nuestros robots.

---

# El robot dentro de Gazebo

Para colocar un robot dentro del simulador necesitamos describir su estructura.

Aquí aparece nuevamente lo que estudiamos en:

```text
14 - URDF
```

Recordemos que URDF puede describir:

```text
links
joints
geometría
masa
inercia
```

Entonces empezamos a conectar los conceptos:

```text
URDF
 │
 ▼
Descripción del robot
 │
 ▼
Gazebo
 │
 ▼
Robot simulado
```

Por eso aprender URDF antes de Gazebo tenía sentido.

---

# Links y Joints nuevamente

Supongamos que nuestro robot tiene:

```text
              base_link
              /       \
             /         \
            ▼           ▼
     wheel_left     wheel_right
```

URDF describe estas relaciones.

Gazebo puede utilizar esta información para representar físicamente el robot.

Por ejemplo:

```text
base_link
    │
    ├── joint izquierdo
    │       │
    │       ▼
    │   rueda izquierda
    │
    └── joint derecho
            │
            ▼
        rueda derecha
```

Cuando las ruedas giran, Gazebo puede calcular el movimiento resultante del robot.

---

# Propiedades físicas

Para realizar una simulación necesitamos más información que simplemente la apariencia del robot.

Por ejemplo:

```text
masa
inercia
colisiones
```

Estas propiedades permiten que el simulador determine cómo debe comportarse el objeto.

Podemos pensar:

```text
Visual
   │
   └── ¿Cómo se ve?


Collision
   │
   └── ¿Con qué puede chocar?


Inertial
   │
   └── ¿Cómo responde físicamente?
```

Estas tres partes son muy importantes cuando utilizamos URDF para simulación.

---

# Visual

La propiedad:

```text
visual
```

describe cómo queremos representar gráficamente una parte del robot.

Por ejemplo:

```xml
<visual>
    <geometry>
        <box size="1 1 1"/>
    </geometry>
</visual>
```

Conceptualmente:

```text
visual
  ↓
¿Cómo se ve el objeto?
```

---

# Collision

La propiedad:

```text
collision
```

define la geometría utilizada para detectar colisiones.

Por ejemplo:

```xml
<collision>
    <geometry>
        <box size="1 1 1"/>
    </geometry>
</collision>
```

Esto permite que Gazebo determine cuándo dos objetos están entrando en contacto.

Por ejemplo:

```text
Robot ─────────→ Pared

      COLISIÓN
          ↓
Robot █████████ Pared
```

Sin una descripción adecuada de colisión, el comportamiento de la simulación podría ser incorrecto.

---

# Inertial

También encontramos:

```text
inertial
```

Esta sección contiene propiedades físicas.

Por ejemplo:

```text
masa
momento de inercia
```

Conceptualmente:

```text
             Robot
               │
       ┌───────┼────────┐
       │       │        │
       ▼       ▼        ▼
    Visual  Collision  Inertial
       │       │        │
       ▼       ▼        ▼
   apariencia choque   física
```

Esto demuestra nuevamente que Gazebo necesita más información que RViz.

---

# Sensores simulados

Una de las cosas más interesantes de Gazebo es que también podemos simular sensores.

Por ejemplo:

```text
LIDAR
Cámara
IMU
```

Esto es especialmente importante para nuestro TurtleBot3.

Podemos tener:

```text
                 Gazebo
                    │
                    ▼
             LIDAR simulado
                    │
                    ▼
                  /scan
                    │
                    ▼
                   ROS 2
```

Desde el punto de vista de ROS, podemos recibir datos del sensor aunque el robot físico no esté conectado.

---

# LIDAR simulado

Imaginemos un TurtleBot3 dentro de una habitación virtual.

```text
████████████████████████
█                      █
█                      █
█        ● Robot       █
█                      █
█                      █
████████████████████████
```

El LIDAR simulado puede medir las distancias hasta las paredes.

Gazebo genera estas mediciones.

Después ROS puede publicarlas en:

```text
/scan
```

Entonces podemos visualizar esos datos en RViz.

Aquí comenzamos a conectar las dos herramientas.

---

# Gazebo + RViz

Gazebo y RViz no tienen que competir entre sí.

En realidad pueden trabajar juntos.

Por ejemplo:

```text
             Gazebo
                │
                │ simula
                ▼
              Robot
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
      /scan   /odom     /tf
        │       │        │
        └───────┼────────┘
                ▼
               RViz
                │
                ▼
          Visualización
```

Gazebo puede generar el comportamiento simulado.

RViz puede mostrar los datos resultantes.

---

# Un ejemplo con el LIDAR

Dentro de Gazebo tenemos:

```text
Pared
████████████████

        ●
      Robot
```

El LIDAR virtual detecta la pared.

Gazebo genera las mediciones.

```text
Gazebo
   │
   ▼
LIDAR virtual
   │
   ▼
/scan
```

Después RViz puede suscribirse:

```text
/scan
  │
  ▼
LaserScan
  │
  ▼
RViz
```

Así podemos observar el sensor virtual exactamente como haríamos con un sensor real desde el punto de vista de ROS.

---

# TurtleBot3 y Gazebo

TurtleBot3 dispone de paquetes preparados para trabajar con simulación.

Primero debemos indicar qué modelo estamos utilizando.

En nuestro caso:

```bash
export TURTLEBOT3_MODEL=burger
```

Podemos comprobarlo:

```bash
echo $TURTLEBOT3_MODEL
```

Deberíamos obtener:

```text
burger
```

Esto es importante porque existen diferentes modelos de TurtleBot3.

Por ejemplo:

```text
Burger
Waffle
Waffle Pi
```

Nuestro trabajo está centrado en:

```text
TurtleBot3 Burger
```

---

# Iniciar una simulación de TurtleBot3

Una forma habitual de iniciar un entorno de simulación de TurtleBot3 es mediante un archivo `launch`.

Por ejemplo:

```bash
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

Antes debemos asegurarnos de tener:

```bash
export TURTLEBOT3_MODEL=burger
```

Entonces:

```bash
export TURTLEBOT3_MODEL=burger

ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

Al ejecutarlo debería iniciarse el entorno de simulación con el TurtleBot3.

---

# ¿Qué hace un launch?

Cuando ejecutamos:

```bash
ros2 launch ...
```

no necesariamente estamos iniciando un único programa.

Un archivo `launch` puede iniciar varios componentes necesarios para la simulación.

Conceptualmente:

```text
ros2 launch
     │
     ▼
Launch file
     │
     ├── Gazebo
     ├── Robot
     ├── sensores
     ├── nodos
     └── configuración
```

Esto hace mucho más sencillo iniciar sistemas complejos.

---

# Comprobar los tópicos

Una vez ejecutada la simulación podemos abrir otra terminal.

Configuramos nuevamente:

```bash
export TURTLEBOT3_MODEL=burger
```

Y podemos comprobar:

```bash
ros2 topic list
```

Podríamos encontrar tópicos relacionados con:

```text
/cmd_vel
/scan
/odom
/tf
/tf_static
```

Esto es muy interesante.

Aunque estamos trabajando con un robot virtual, ROS sigue utilizando tópicos.

---

# El tópico /cmd_vel

Uno de los tópicos más importantes para un robot móvil es:

```text
/cmd_vel
```

Este tópico se utiliza para enviar comandos de velocidad.

Conceptualmente:

```text
Comando
   │
   ▼
/cmd_vel
   │
   ▼
Robot
   │
   ▼
Movimiento
```

Podemos enviar:

```text
velocidad lineal
velocidad angular
```

Esto permite controlar el movimiento del robot.

---

# Teleoperación

Una forma sencilla de comprobar la simulación es utilizar teleoperación.

Podemos ejecutar:

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

Entonces podemos controlar el robot mediante el teclado.

Conceptualmente:

```text
             Teclado
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
             Gazebo
```

Cuando presionamos una tecla, se publica un comando.

El robot simulado recibe ese comando y se mueve dentro del mundo virtual.

---

# Algo muy importante

El comando:

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

no mueve directamente las ruedas.

Lo que hace es publicar mensajes de velocidad.

Podemos representarlo:

```text
TECLADO
   │
   ▼
teleop_keyboard
   │
   ▼
/cmd_vel
   │
   ▼
controlador
   │
   ▼
ruedas
```

Esta separación es una de las características importantes de ROS 2.

---

# Robot virtual y robot real

Aquí apareció algo que me pareció especialmente interesante.

Desde el punto de vista de nuestros programas, muchas veces podemos trabajar de manera parecida con:

```text
Robot simulado
```

y:

```text
Robot real
```

Por ejemplo, ambos pueden utilizar:

```text
/cmd_vel
/scan
/odom
/tf
```

Podemos imaginar:

```text
                    ROS 2
                      │
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
          Gazebo          TurtleBot3 real
             │                 │
             ▼                 ▼
           /scan             /scan
           /odom             /odom
           /tf               /tf
```

Esto permite desarrollar y probar algoritmos primero en simulación.

Después podemos probarlos en el robot físico.

---

# ¿Por qué simular antes?

Trabajar primero en simulación tiene varias ventajas.

Por ejemplo:

```text
No descargamos la batería.

No necesitamos tener siempre el robot disponible.

Podemos repetir experimentos.

Podemos crear diferentes entornos.

Podemos probar errores sin arriesgar el hardware.

Podemos trabajar desde nuestro computador.
```

Esto resulta especialmente útil durante el desarrollo.

---

# Pero la simulación no es perfecta

También debemos recordar algo importante:

> **Un robot simulado nunca representa perfectamente al robot real.**

En el mundo real existen factores como:

```text
deslizamiento de ruedas
ruido de sensores
errores mecánicos
irregularidades del piso
batería
fricción real
interferencias
```

Una simulación intenta aproximarse a estos fenómenos.

Por eso una estrategia común es:

```text
Desarrollar
    │
    ▼
Simular
    │
    ▼
Comprobar
    │
    ▼
Robot real
```

---

# Gazebo dentro de nuestro aprendizaje

Ahora podemos observar cómo se conectan las últimas entradas.

```text
13 - TF2
     │
     ▼
Sistemas de coordenadas
     │
     ▼
14 - URDF
     │
     ▼
Descripción del robot
     │
     ▼
15 - RViz
     │
     ▼
Visualización
     │
     ▼
16 - Gazebo
     │
     ▼
Simulación
```

Ya tenemos varias piezas fundamentales para comenzar a trabajar directamente con TurtleBot3.

---

# El flujo completo hasta ahora

Podemos representar lo aprendido de esta manera:

```text
                    ROS 2
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
         TF2         URDF       Topics
          │           │           │
          └──────┬────┴────┬──────┘
                 │         │
                 ▼         ▼
               RViz      Gazebo
                 │         │
                 ▼         ▼
          Visualización  Simulación
```

Cada herramienta cumple una función diferente.

---

# Prueba básica de TurtleBot3 en Gazebo

Una prueba básica puede dividirse en dos terminales.

## Terminal 1 - Simulación

```bash
export TURTLEBOT3_MODEL=burger

ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

Esta terminal debe permanecer ejecutándose.

---

## Terminal 2 - Teleoperación

Abrimos otra terminal.

```bash
export TURTLEBOT3_MODEL=burger

ros2 run turtlebot3_teleop teleop_keyboard
```

Ahora podemos enviar comandos al robot.

La estructura es:

```text
Terminal 1
    │
    ▼
Gazebo + TurtleBot3


Terminal 2
    │
    ▼
Teleoperación
    │
    ▼
/cmd_vel
    │
    ▼
Robot simulado
```

---

# Una tercera terminal para observar ROS

También podemos abrir otra terminal y ejecutar:

```bash
ros2 topic list
```

Podemos observar los tópicos disponibles.

Por ejemplo:

```bash
ros2 topic echo /scan
```

o:

```bash
ros2 topic echo /odom
```

Entonces podemos comprobar que el robot simulado realmente está generando información.

---

# Gazebo, RViz y terminal

Ahora podemos tener tres formas diferentes de observar nuestro sistema.

```text
              ROS 2
                │
     ┌──────────┼──────────┐
     │          │          │
     ▼          ▼          ▼
  Gazebo      RViz      Terminal
     │          │          │
     ▼          ▼          ▼
 Simulación  Visualizar   Datos
 físicos     información  ROS
```

Las tres herramientas se complementan.

---

# Lo que aprendí

Antes de estudiar Gazebo pensaba que RViz y Gazebo realizaban prácticamente la misma función.

Ahora entiendo que son herramientas diferentes.

```text
RViz
↓
Representa información de ROS.


Gazebo
↓
Simula el robot y su entorno.
```

También entendí que Gazebo puede generar información que después ROS publica mediante tópicos.

Por ejemplo:

```text
Gazebo
   │
   ▼
LIDAR virtual
   │
   ▼
/scan
   │
   ▼
RViz
```

Esto permite desarrollar sistemas robóticos sin depender siempre del hardware físico.

---

# Algo que considero importante

La simulación no reemplaza completamente al robot real.

Más bien funciona como una etapa intermedia.

```text
Idea
 │
 ▼
Código
 │
 ▼
Gazebo
 │
 ▼
Pruebas
 │
 ▼
Robot real
```

De esta manera podemos encontrar muchos problemas antes de realizar las pruebas físicas.

---

# Conclusión

En esta entrada aprendí que Gazebo nos permite crear un entorno virtual donde podemos probar robots.

También entendí que:

```text
URDF
```

describe la estructura del robot,

```text
TF2
```

relaciona sus sistemas de coordenadas,

```text
RViz
```

visualiza información,

y:

```text
Gazebo
```

permite simular el robot y su entorno.

Ahora tenemos las herramientas necesarias para dejar de estudiar los componentes de manera aislada y comenzar a trabajar directamente con nuestro robot móvil.

---

# Siguiente entrada

Hasta ahora hemos estudiado diferentes herramientas:

```text
TF2
URDF
RViz
Gazebo
```

Pero todas ellas comienzan a tener mucho más sentido cuando las utilizamos juntas en un robot real.

El robot que utilizaremos será:

```text
TurtleBot3 Burger
```

En la siguiente entrada veremos:

```text
qué es TurtleBot3
cómo está compuesto
cómo funciona dentro de ROS 2
qué tópicos utiliza
cómo iniciarlo
cómo controlarlo
cómo se relaciona con el LIDAR
cómo se relaciona con la OpenCR
```

Y comenzaremos a conectar definitivamente:

```text
Computador
     │
     ▼
ROS 2
     │
     ▼
Raspberry Pi
     │
     ▼
OpenCR
     │
     ▼
TurtleBot3
```

# Entrada 17 - TurtleBot3