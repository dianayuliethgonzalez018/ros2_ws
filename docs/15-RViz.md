# Entrada 15 - RViz

## Objetivo

En la entrada anterior vimos **URDF** y entendimos cómo ROS 2 puede describir la estructura de un robot mediante `links` y `joints`.

Ahora sabemos que podemos tener una estructura como:

```text
                 base_link
                /    |    \
               /     |     \
              ▼      ▼      ▼
     wheel_left   base_scan   wheel_right
```

También vimos que `robot_state_publisher` utiliza la descripción del robot para publicar las transformaciones correspondientes.

Pero apareció una nueva pregunta:

> **¿Cómo puedo comprobar visualmente que todo esto realmente está funcionando?**

Hasta ahora hemos utilizado principalmente la terminal.

Podemos consultar:

```bash
ros2 topic list
```

Podemos observar transformaciones.

Podemos revisar nodos.

Podemos comprobar tópicos.

Pero cuando trabajamos con un robot, muchas veces necesitamos **ver lo que está ocurriendo**.

Aquí aparece una de las herramientas más importantes de ROS 2:

# RViz

---

# ¿Qué es RViz?

**RViz** es una herramienta de visualización para ROS.

Su función principal es permitirnos representar gráficamente información que está circulando dentro del sistema ROS.

Por ejemplo, podemos visualizar:

```text
Robot
LIDAR
Mapa
TF
Trayectorias
Odometry
PointCloud
Imágenes
Objetivos de navegación
```

Una forma sencilla de entenderlo sería:

```text
                 ROS 2
                   │
       ┌───────────┼───────────┐
       │           │           │
      /tf        /scan       /map
       │           │           │
       └───────────┼───────────┘
                   │
                   ▼
                  RViz
                   │
                   ▼
           Visualización 3D
```

RViz toma información que está siendo publicada en ROS y nos permite **observarla gráficamente**.

---

# Algo importante: RViz no es un simulador

Esta fue una de las primeras confusiones que tuve.

Al ver un robot dentro de RViz puede parecer que estamos utilizando un simulador.

Pero RViz **no simula la física del robot**.

RViz principalmente:

> **visualiza información de ROS 2.**

Por ejemplo, si vemos un TurtleBot3 desplazándose en RViz, RViz no necesariamente está calculando:

```text
masa
fricción
gravedad
fuerzas
colisiones físicas
```

Simplemente está representando la información que recibe.

Esto será diferente cuando lleguemos a:

```text
Gazebo
```

que estudiaremos en la siguiente entrada.

Podemos resumir inicialmente:

```text
RViz
  ↓
Visualización

Gazebo
  ↓
Simulación
```

---

# Ejecutando RViz

Una forma básica de iniciar RViz en ROS 2 es:

```bash
rviz2
```

También podemos utilizar:

```bash
ros2 run rviz2 rviz2
```

Al abrirlo aparece una interfaz con diferentes paneles.

Inicialmente puede parecer complicada, pero podemos dividirla en partes.

```text
┌───────────────────────────────────────────────┐
│                    RViz                       │
├──────────────┬────────────────────────────────┤
│              │                                │
│   Displays   │                                │
│              │       Visualización 3D         │
│              │                                │
│              │                                │
├──────────────┴────────────────────────────────┤
│                Información                    │
└───────────────────────────────────────────────┘
```

La zona central es donde observaremos nuestro robot y los diferentes datos.

---

# Displays

Uno de los conceptos más importantes de RViz es:

```text
Displays
```

Un **Display** permite indicarle a RViz qué tipo de información queremos visualizar.

Por ejemplo:

```text
RobotModel
TF
LaserScan
Map
Odometry
Path
PointCloud2
Image
```

Podemos imaginar RViz como una pantalla inicialmente vacía.

Luego vamos agregando capas:

```text
              RViz
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
 RobotModel    TF     LaserScan
      │         │         │
      └─────────┼─────────┘
                ▼
       Visualización final
```

Cada Display representa información diferente.

---

# RobotModel

Comencemos con uno que conecta directamente con la entrada anterior:

```text
RobotModel
```

En URDF aprendimos a describir:

```text
links
joints
geometrías
```

RViz puede utilizar esa descripción para mostrar gráficamente el robot.

La relación sería:

```text
             URDF
               │
               ▼
      robot_description
               │
               ▼
          RobotModel
               │
               ▼
              RViz
```

Entonces todo lo que aprendimos sobre URDF comienza a tener una representación visual.

Por ejemplo:

```text
base_link
wheel_left_link
wheel_right_link
base_scan
```

pueden aparecer ahora como partes visibles del robot.

---

# TF dentro de RViz

Aquí volvemos a conectar con la entrada 13.

RViz también tiene un Display llamado:

```text
TF
```

Al activarlo podemos visualizar los sistemas de coordenadas del robot.

Por ejemplo:

```text
                   Z
                   ↑
                   │
                   │
                   ●──────→ X
                  /
                 /
                Y
```

Y podemos observar diferentes frames:

```text
base_link
base_scan
wheel_left_link
wheel_right_link
odom
map
```

Esto es muy útil porque podemos comprobar visualmente si las transformaciones tienen sentido.

Por ejemplo:

```text
                   base_scan
                       ↑
                       │
                       │
                  base_link
```

Si el LIDAR aparece debajo del robot cuando físicamente debería estar encima, probablemente existe algún problema en la transformación o en la descripción del robot.

---

# Fixed Frame

Uno de los primeros parámetros que encontramos en RViz es:

```text
Fixed Frame
```

Este concepto es muy importante.

RViz necesita un sistema de coordenadas que utilizará como referencia para representar todos los demás datos.

Por ejemplo:

```text
Fixed Frame: base_link
```

significa que visualizaremos la información tomando `base_link` como referencia.

También podemos encontrar:

```text
odom
```

o:

```text
map
```

dependiendo de lo que estemos haciendo.

Podemos imaginarlo así:

```text
                    map
                     │
                     ▼
                    odom
                     │
                     ▼
                 base_link
                     │
                     ▼
                  sensor
```

Dependiendo de nuestra aplicación, podemos seleccionar diferentes referencias.

---

# Un error muy común en RViz

En algunas ocasiones RViz muestra mensajes como:

```text
Fixed Frame
No tf data
```

o algún Display aparece en rojo.

Esto no significa necesariamente que RViz esté dañado.

Puede significar que RViz está intentando utilizar un frame que actualmente no existe.

Por ejemplo:

```text
Fixed Frame: map
```

pero todavía nadie está publicando el frame:

```text
map
```

Entonces RViz no sabe cómo posicionar la información.

Esto me permitió entender que cuando RViz muestra un error, muchas veces debemos revisar primero:

```text
TF
Frames
Topics
Fixed Frame
```

antes de pensar que el programa está fallando.

---

# LaserScan

Ahora llegamos a algo especialmente importante para un robot móvil.

Nuestro TurtleBot3 utiliza un LIDAR.

El LIDAR realiza mediciones de distancia alrededor del robot.

En ROS esa información normalmente puede encontrarse en un tópico como:

```text
/scan
```

Podemos comprobarlo desde la terminal:

```bash
ros2 topic list
```

y también:

```bash
ros2 topic echo /scan
```

Pero observar cientos de números en la terminal no es demasiado intuitivo.

RViz puede representar esos datos mediante:

```text
LaserScan
```

La relación sería:

```text
             LIDAR
                │
                ▼
              /scan
                │
                ▼
           LaserScan
                │
                ▼
              RViz
```

En lugar de números podemos observar puntos alrededor del robot.

Por ejemplo:

```text
       • • • • • • • •
     •               •
   •                   •

          ROBOT

   •                   •
     •               •
       • • • • • • •
```

Esos puntos representan objetos detectados por el sensor.

---

# ¿Por qué esto es tan útil?

Imaginemos que el robot está frente a una pared.

Físicamente tenemos:

```text
ROBOT                 PARED
  ●                    █
  ●                    █
  ●                    █
```

El LIDAR detecta diferentes distancias.

RViz puede mostrarlas gráficamente.

Entonces podemos comprobar rápidamente:

> ¿El sensor realmente está detectando la pared?

Esto resulta mucho más sencillo que analizar directamente los datos numéricos de `/scan`.

---

# Map

Otro Display extremadamente importante es:

```text
Map
```

Más adelante utilizaremos **SLAM** para construir un mapa del entorno.

Ese mapa normalmente será publicado en:

```text
/map
```

RViz puede visualizarlo.

La relación será:

```text
              SLAM
                │
                ▼
              /map
                │
                ▼
               Map
                │
                ▼
              RViz
```

Por ejemplo, podríamos terminar observando algo parecido a:

```text
██████████████████████
█                    █
█                    █
█       ROBOT        █
█         ▲          █
█                    █
██████████████████████
```

Esto será fundamental cuando lleguemos a las entradas:

```text
18 - SLAM
19 - Navigation2
```

---

# Odometry

Otro dato que podemos visualizar es:

```text
Odometry
```

Normalmente relacionado con:

```text
/odom
```

La odometría permite estimar cómo se está desplazando el robot.

Por ejemplo:

```text
Inicio
  ●
   \
    \
     ●
      \
       \
        ● Robot
```

RViz puede ayudarnos a observar cómo cambia la posición estimada del robot mientras se mueve.

Esto también se relaciona con los frames:

```text
odom
   │
   ▼
base_link
```

que vimos anteriormente.

---

# Path

RViz también puede mostrar trayectorias mediante:

```text
Path
```

Esto será especialmente útil en navegación.

Imaginemos:

```text
Robot
  ●
   \
    \
     \
      ──────────────● Objetivo
```

Cuando Navigation2 calcule una ruta, RViz podrá mostrarla.

Así podremos observar:

```text
posición actual
      ↓
      ●───────────────┐
                      │
                      │
                      └──────● objetivo
```

Esto nos permitirá ver qué camino está intentando seguir el robot.

---

# RViz no genera necesariamente los datos

Aquí entendí algo muy importante.

Cuando agregamos:

```text
LaserScan
```

RViz no está realizando la medición.

Cuando agregamos:

```text
Map
```

RViz no está construyendo necesariamente el mapa.

Cuando agregamos:

```text
RobotModel
```

RViz no está creando la descripción del robot.

Principalmente está **visualizando información generada por otros componentes**.

Por ejemplo:

```text
LIDAR ───────────────→ /scan
                         │
                         ▼
                       RViz


SLAM ────────────────→ /map
                         │
                         ▼
                       RViz


URDF ─────────→ robot_description
                         │
                         ▼
                       RViz
```

Por eso RViz es una herramienta de visualización.

---

# Topics y RViz

Ahora podemos conectar otro concepto de ROS 2.

Supongamos que tenemos:

```text
/scan
/map
/odom
```

Estos son tópicos.

RViz puede suscribirse a ellos para visualizar su información.

Podemos pensar:

```text
Publisher
    │
    ▼
  Topic
    │
    ▼
   RViz
```

Por ejemplo:

```text
LIDAR
  │
  ▼
/scan
  │
  ▼
RViz
```

Entonces muchos Displays de RViz necesitan que seleccionemos un:

```text
Topic
```

correcto.

---

# ¿Qué pasa si selecciono el tópico incorrecto?

Supongamos que agregamos:

```text
LaserScan
```

pero no seleccionamos:

```text
/scan
```

RViz no tendrá datos que representar.

Por eso cuando algo no aparece podemos comprobar:

```bash
ros2 topic list
```

y verificar qué tópicos existen.

También podemos comprobar si realmente llegan mensajes:

```bash
ros2 topic echo /scan
```

De esta manera podemos separar dos problemas diferentes:

```text
¿ROS está publicando datos?

           o

¿RViz está configurado incorrectamente?
```

---

# Status de los Displays

RViz utiliza estados para indicarnos si un Display está funcionando correctamente.

Podemos encontrar:

```text
OK
Warn
Error
```

De manera conceptual:

```text
OK
↓
Los datos están llegando correctamente.


Warn
↓
Existe alguna condición que debemos revisar.


Error
↓
RViz no puede representar correctamente la información.
```

Esto es muy útil para diagnosticar problemas.

---

# Guardar una configuración de RViz

Configurar RViz desde cero cada vez sería incómodo.

Podemos guardar una configuración.

Normalmente estos archivos utilizan la extensión:

```text
.rviz
```

Por ejemplo:

```text
turtlebot3.rviz
```

Dentro pueden almacenarse configuraciones como:

```text
Fixed Frame
Displays
Topics
Cámara
Opciones de visualización
```

Después podemos iniciar RViz utilizando esa configuración:

```bash
rviz2 -d turtlebot3.rviz
```

Esto explica por qué algunos paquetes de ROS incluyen archivos `.rviz`.

---

# RViz y nuestro TurtleBot3

Ahora podemos conectar todo esto con el robot real.

Cuando iniciamos el TurtleBot3 mediante su `bringup`, diferentes componentes comienzan a publicar información.

Podemos tener algo conceptualmente parecido a:

```text
                 TurtleBot3
                     │
          ┌──────────┼───────────┐
          │          │           │
          ▼          ▼           ▼
        /scan      /odom        /tf
          │          │           │
          └──────────┼───────────┘
                     │
                     ▼
                    RViz
```

Entonces desde nuestro computador podemos visualizar información que está generando el robot.

Esto nos permite comprobar sensores, transformaciones y posteriormente visualizar el mapa.

---

# RViz y ROS_DOMAIN_ID

Cuando trabajamos con ROS 2 en diferentes computadores, ambos deben poder encontrarse dentro de la comunicación de ROS 2.

En nuestro caso podemos configurar:

```bash
export ROS_DOMAIN_ID=30
```

También debemos indicar el modelo del TurtleBot3:

```bash
export TURTLEBOT3_MODEL=burger
```

Por ejemplo:

```bash
export ROS_DOMAIN_ID=30
export TURTLEBOT3_MODEL=burger
```

Si los equipos no están comunicándose correctamente, podemos abrir RViz perfectamente pero no recibir los datos del robot.

Es decir:

```text
RViz abre
   ↓
pero
   ↓
no aparecen datos
```

No necesariamente significa que RViz tenga un problema.

También debemos revisar la comunicación ROS 2 entre los dispositivos.

---

# Una prueba sencilla

Una prueba útil antes de abrir RViz es ejecutar:

```bash
ros2 topic list
```

Si el robot está funcionando correctamente podríamos encontrar tópicos relacionados con:

```text
/scan
/odom
/tf
/tf_static
/joint_states
```

Después podemos abrir:

```bash
rviz2
```

Y comenzar agregando:

```text
TF
RobotModel
LaserScan
```

De esta manera vamos comprobando cada componente por separado.

---

# Relación entre TF2, URDF y RViz

Ahora las tres entradas empiezan a conectarse.

Primero estudiamos:

```text
13 - TF2
```

y aprendimos cómo ROS relaciona sistemas de coordenadas.

Después:

```text
14 - URDF
```

y aprendimos cómo describir las partes del robot.

Ahora:

```text
15 - RViz
```

nos permite observar esa información.

Podemos representarlo así:

```text
                  URDF
                    │
                    ▼
            robot_description
                    │
                    ▼
        robot_state_publisher
                    │
                    ▼
                   TF2
                    │
                    ▼
                   RViz
                    │
                    ▼
        ┌─────────────────────┐
        │      ROBOT 3D       │
        │                     │
        │   TF     LIDAR      │
        │   MAP    ODOM       │
        └─────────────────────┘
```

Ahora muchos conceptos que antes aparecían únicamente en la terminal empiezan a tener una representación visual.

---

# Algo que me ayudó a entender RViz

Podemos pensar en RViz como un **monitor para ROS 2**.

ROS puede estar ejecutando:

```text
sensores
nodos
transformaciones
mapas
algoritmos
```

y RViz nos permite observar parte de esa información.

Por eso es especialmente útil para:

```text
visualización
desarrollo
pruebas
depuración
navegación
```

---

# Lo que aprendí

Al principio pensaba que RViz era simplemente una aplicación para mostrar el robot en 3D.

Pero realmente puede visualizar muchos tipos de información.

Por ejemplo:

```text
RobotModel  → modelo del robot

TF          → sistemas de coordenadas

LaserScan   → mediciones del LIDAR

Map         → mapa

Odometry    → odometría

Path        → trayectorias
```

También entendí que:

> **RViz no es el encargado de generar necesariamente esos datos.**

Los datos vienen de otros componentes de ROS.

RViz los recibe y los representa.

---

# La diferencia que debemos recordar

Por ahora podemos quedarnos con esta diferencia:

```text
                RViz
                  │
                  ▼
         ¿Qué está pasando?
                  │
                  ▼
            VISUALIZAR
```

Mientras que en la siguiente entrada veremos:

```text
               Gazebo
                  │
                  ▼
       ¿Qué pasaría físicamente?
                  │
                  ▼
             SIMULAR
```

Y esta diferencia será fundamental.

---

# Conclusión

En esta entrada aprendí que RViz es una herramienta que permite visualizar información de ROS 2.

También entendí cómo se conecta con conceptos vistos anteriormente:

```text
TF2
 │
 ├── sistemas de coordenadas
 │
 ▼
URDF
 │
 ├── estructura del robot
 │
 ▼
RViz
 │
 └── visualización
```

Además, RViz será fundamental para las siguientes aplicaciones:

```text
TurtleBot3
SLAM
Navigation2
```

porque nos permitirá observar el robot, el LIDAR, el mapa y las rutas de navegación.

---

# Siguiente entrada

Ahora sabemos cómo:

```text
describir el robot     → URDF

relacionar sus frames  → TF2

visualizar sus datos   → RViz
```

Pero aparece una nueva pregunta:

> **¿Qué hacemos si no tenemos el robot físico disponible o queremos probar nuestro sistema sin arriesgar el hardware real?**

Necesitamos crear un entorno virtual donde el robot pueda:

```text
moverse
usar sensores
detectar obstáculos
interactuar con objetos
```

Para eso necesitamos un simulador.

# Entrada 16 - Gazebo