# Entrada 18 - SLAM

## Objetivo

En la entrada anterior vimos cómo trabajar con un **TurtleBot3 Burger real** dentro de ROS 2.

Aprendimos que el robot puede:

- recibir comandos de velocidad;
- mover sus ruedas;
- publicar odometría;
- obtener información del LIDAR;
- publicar transformaciones;
- visualizar información mediante RViz.

Ahora aparece una nueva pregunta:

> **¿Cómo puede el robot conocer el entorno donde se encuentra si todavía no tiene un mapa?**

Para resolver este problema utilizamos una técnica conocida como:

# SLAM

---

# ¿Qué significa SLAM?

SLAM significa:

```text
Simultaneous Localization And Mapping
```

En español podemos interpretarlo como:

```text
Localización y Mapeo Simultáneos
```

La idea principal es que el robot intenta hacer dos cosas al mismo tiempo:

```text
1. Construir un mapa del entorno.

2. Estimar dónde se encuentra dentro de ese mapa.
```

Podemos representarlo así:

```text
              ROBOT
                │
       ┌────────┴────────┐
       │                 │
       ▼                 ▼
   ¿Dónde estoy?     ¿Cómo es el lugar?
       │                 │
       └────────┬────────┘
                ▼
               SLAM
```

---

# ¿Por qué SLAM es necesario?

Imaginemos que colocamos el TurtleBot3 en una habitación que nunca ha visto.

El robot no conoce:

```text
las paredes

los pasillos

los obstáculos

el tamaño del lugar

su posición exacta
```

Sin un mapa, el robot puede moverse manualmente, pero no tiene una representación estructurada del entorno.

SLAM permite construir esa representación.

---

# El problema del huevo y la gallina

Una de las ideas más interesantes de SLAM es que aparecen dos problemas que dependen uno del otro.

Para construir un mapa necesitamos saber:

```text
¿Dónde está el robot?
```

Pero para saber dónde está el robot sería muy útil tener:

```text
un mapa
```

Entonces tenemos:

```text
Necesito un mapa
para saber dónde estoy.

Pero necesito saber dónde estoy
para construir el mapa.
```

Este es precisamente el problema que SLAM intenta resolver.

---

# ¿Qué información utiliza SLAM?

En un robot móvil como TurtleBot3 podemos utilizar principalmente:

```text
LIDAR
Odometría
TF2
```

La información del LIDAR puede llegar mediante:

```text
/scan
```

La odometría puede llegar mediante:

```text
/odom
```

Y TF2 permite relacionar sistemas de coordenadas como:

```text
odom
base_link
base_scan
map
```

Podemos imaginar:

```text
             /scan
               │
               ▼
             LIDAR
               │
               │
               ▼
              SLAM
               ▲
               │
               │
             /odom
               │
               ▼
           Odometría
```

---

# El papel del LIDAR

El LIDAR es uno de los sensores más importantes para construir el mapa.

Imaginemos una habitación:

```text
█████████████████████████
█                       █
█                       █
█         ● Robot       █
█                       █
█                       █
█████████████████████████
```

El LIDAR mide distancias alrededor del robot.

Conceptualmente:

```text
             ↑
          distancia

       ↖     ↑     ↗

          [LIDAR]

       ↙     ↓     ↘
```

Cada medición indica aproximadamente dónde existe una superficie.

Con muchas mediciones podemos comenzar a reconstruir las paredes del entorno.

---

# El papel de la odometría

El robot también necesita una estimación de su movimiento.

Por ejemplo:

```text
Posición inicial
      ●

      │
      │ avanza
      ▼

             ●
       nueva posición
```

La odometría nos ayuda a estimar este desplazamiento.

En TurtleBot3 esta información puede consultarse mediante:

```bash
ros2 topic echo /odom
```

SLAM puede utilizar esta información junto con las mediciones del sensor.

---

# Pero la odometría no es perfecta

Existe un problema importante.

Las ruedas pueden:

```text
deslizarse

girar ligeramente diferente

tener errores mecánicos

recorrer superficies irregulares
```

Por eso la odometría acumula error con el tiempo.

Podemos imaginar:

```text
Posición real
      ●

Posición estimada
          ●
```

Al principio ambas pueden estar muy cerca.

Después de recorrer una distancia mayor:

```text
Real
 ●


                  ●
               estimada
```

puede aparecer una diferencia.

SLAM intenta corregir parte de este error utilizando las observaciones del entorno.

---

# Cuando el robot reconoce un lugar

Supongamos que el robot recorre una habitación y después vuelve a un lugar por donde ya había pasado.

El LIDAR puede detectar una geometría conocida.

Por ejemplo:

```text
Primera vez:

██████
█
█   ● Robot
█
██████████


Después:

██████
█
█      ● Robot
█
██████████
```

El sistema puede reconocer que ambas observaciones corresponden a una zona similar.

Esto puede ayudar a corregir errores acumulados.

A este concepto se le suele relacionar con:

```text
loop closure
```

o cierre de ciclo.

---

# El mapa de ocupación

Uno de los resultados de SLAM puede ser un:

```text
Occupancy Grid Map
```

Es decir, un mapa dividido en pequeñas celdas.

Cada celda representa información sobre el espacio.

Podemos tener conceptualmente:

```text
Blanco
↓
Espacio libre


Negro
↓
Obstáculo


Gris
↓
Zona desconocida
```

Por ejemplo:

```text
████████████████████
█                  █
█                  █
█       ●          █
█                  █
████████████████████
```

La zona interna puede representar espacio libre y los bordes representan las paredes detectadas.

---

# El tópico /map

Cuando SLAM está funcionando podemos obtener un tópico como:

```text
/map
```

Podemos comprobarlo mediante:

```bash
ros2 topic list
```

y buscar:

```text
/map
```

También podemos inspeccionarlo:

```bash
ros2 topic echo /map
```

Aunque, al igual que ocurrió con el LIDAR, observar directamente la información numérica no suele ser lo más intuitivo.

Para eso utilizamos RViz.

---

# SLAM + RViz

Aquí se conecta directamente la entrada 15.

RViz puede mostrar el mapa generado por SLAM.

La relación sería:

```text
             TurtleBot3
                 │
                 ▼
               /scan
                 │
                 ▼
                SLAM
                 │
                 ▼
               /map
                 │
                 ▼
                RViz
```

Así podemos observar cómo el mapa va apareciendo mientras movemos el robot.

---

# ¿Qué ocurre mientras movemos el robot?

Supongamos que comenzamos en una habitación desconocida.

Al principio podríamos tener:

```text
?????????????????????
?????????????????????
?????????????????????
????????●????????????
?????????????????????
```

A medida que el LIDAR detecta el entorno:

```text
████████?????????????
█      █?????????????
█  ●   █?????????????
█      █?????????????
████████?????????????
```

Luego seguimos avanzando:

```text
████████████████?????
█              █?????
█              █?????
█       ●      █?????
████████████████?????
```

Es decir, el robot va descubriendo progresivamente el entorno.

---

# SLAM Toolbox

En ROS 2 podemos trabajar con una herramienta llamada:

```text
slam_toolbox
```

Esta herramienta permite realizar SLAM utilizando información del robot.

Podemos iniciar SLAM mediante un archivo `launch`.

Por ejemplo:

```bash
ros2 launch turtlebot3_cartographer cartographer.launch.py
```

o, dependiendo de la configuración utilizada, mediante SLAM Toolbox.

En nuestras pruebas utilizamos **SLAM Toolbox** para construir el mapa del entorno con el TurtleBot3.

---

# Antes de iniciar SLAM

Primero necesitamos tener funcionando el robot físico.

En la Raspberry Pi podemos iniciar:

```bash
export TURTLEBOT3_MODEL=burger
```

y luego:

```bash
ros2 launch turtlebot3_bringup robot.launch.py
```

Esto permite que ROS 2 tenga disponibles los sensores, la odometría y las transformaciones del robot.

---

# Comunicación entre el robot y el computador

Podemos ejecutar los componentes del robot en la Raspberry Pi y visualizar el mapa desde nuestro computador.

Conceptualmente:

```text
COMPUTADOR
----------------
RViz
SLAM
Teleoperación


       Wi-Fi
         │
         ▼


RASPBERRY PI
----------------
TurtleBot3 Bringup
LIDAR
Odometría
TF
```

Para que ambos equipos formen parte del mismo sistema ROS 2 deben tener una configuración compatible.

Por ejemplo:

```bash
export ROS_DOMAIN_ID=30
```

---

# Iniciar SLAM

En el computador podemos preparar el entorno:

```bash
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30
```

Después podemos iniciar el proceso de SLAM.

Dependiendo del paquete utilizado, el comando puede variar.

Con SLAM Toolbox podemos trabajar con los archivos `launch` disponibles para mapeo.

La idea general es:

```text
Bringup del robot
        │
        ▼
       /scan
       /odom
       /tf
        │
        ▼
       SLAM
        │
        ▼
       /map
```

---

# Teleoperación durante el mapeo

Para construir un buen mapa necesitamos mover el robot por el entorno.

Podemos abrir otra terminal y ejecutar:

```bash
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30
```

Luego:

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

Ahora podemos mover manualmente el robot mientras SLAM construye el mapa.

---

# Flujo de trabajo durante el mapeo

Podemos organizarlo de esta manera:

```text
Terminal 1
----------
Robot Bringup


Terminal 2
----------
SLAM


Terminal 3
----------
Teleoperación


RViz
----------
Visualización del mapa
```

Mientras movemos el TurtleBot3:

```text
Teleop
  │
  ▼
/cmd_vel
  │
  ▼
Robot
  │
  ├────► /odom
  │
  └────► /scan
             │
             ▼
            SLAM
             │
             ▼
            /map
             │
             ▼
            RViz
```

---

# ¿Cómo debemos mover el robot?

Durante el mapeo es conveniente recorrer el entorno de forma controlada.

Por ejemplo:

```text
Inicio
  ●───────→
          │
          │
          ↓
          ●───────→
```

Debemos permitir que el LIDAR observe correctamente las paredes y los objetos.

Movimientos demasiado rápidos pueden dificultar la construcción del mapa.

Por eso es mejor recorrer el entorno lentamente.

---

# Observar el mapa en RViz

En RViz podemos agregar un Display:

```text
Map
```

y seleccionar el tópico:

```text
/map
```

También podemos visualizar:

```text
LaserScan
RobotModel
TF
```

Entonces podemos observar al mismo tiempo:

```text
Mapa
Robot
LIDAR
Frames
```

Esto resulta muy útil para comprobar si SLAM está funcionando correctamente.

---

# Fixed Frame durante SLAM

Durante el mapeo podemos trabajar con un frame como:

```text
map
```

en RViz.

Por ejemplo:

```text
Fixed Frame: map
```

Entonces la estructura puede verse conceptualmente así:

```text
map
 │
 ▼
odom
 │
 ▼
base_footprint
 │
 ▼
base_link
 │
 ▼
base_scan
```

La aparición del frame `map` es una diferencia importante respecto a las pruebas básicas del robot.

---

# Map y odom

Aquí aparece una relación muy importante.

Podemos tener:

```text
map
 │
 ▼
odom
 │
 ▼
base_link
```

`odom` representa principalmente una referencia relacionada con la odometría local.

`map` representa una referencia global asociada al mapa.

SLAM puede mantener una transformación entre:

```text
map → odom
```

para corregir errores de localización acumulados.

---

# Guardar el mapa

Construir un mapa puede tomar varios minutos.

No queremos perderlo al cerrar SLAM.

Por eso podemos guardarlo.

En ROS 2 podemos utilizar herramientas relacionadas con `map_server`.

Por ejemplo:

```bash
ros2 run nav2_map_server map_saver_cli -f ~/maps/mapa_laboratorio
```

Esto puede generar archivos como:

```text
mapa_laboratorio.yaml
mapa_laboratorio.pgm
```

---

# ¿Qué contiene el archivo YAML?

El archivo:

```text
mapa_laboratorio.yaml
```

contiene información sobre el mapa.

Por ejemplo:

```text
imagen utilizada
resolución
origen
umbrales de ocupación
```

Conceptualmente puede verse así:

```yaml
image: mapa_laboratorio.pgm
resolution: 0.05
origin: [0.0, 0.0, 0.0]
occupied_thresh: 0.65
free_thresh: 0.25
```

Los valores exactos dependen del mapa generado.

---

# ¿Qué contiene el archivo PGM?

El archivo:

```text
mapa_laboratorio.pgm
```

contiene la imagen del mapa.

Podemos pensar:

```text
YAML
 │
 └── configuración del mapa


PGM
 │
 └── imagen del mapa
```

Ambos archivos trabajan juntos.

---

# Nuestro mapa del laboratorio

Durante nuestras pruebas con TurtleBot3 recorrimos el laboratorio utilizando teleoperación mientras el LIDAR detectaba el entorno.

El mapa se guardó en una ruta similar a:

```text
$HOME/maps/mapa_laboratorio.yaml
```

Esto nos permitió cerrar el proceso de SLAM y posteriormente volver a cargar el mismo mapa para navegación.

---

# ¿Cómo sabemos si un mapa quedó bien?

Un mapa útil debería representar claramente:

```text
paredes

pasillos

obstáculos permanentes

espacios libres
```

Por ejemplo, un buen resultado podría verse aproximadamente así:

```text
██████████████████████████
█                        █
█                        █
█      █████             █
█      █   █             █
█      █████             █
█                        █
██████████████████████████
```

Si aparecen paredes duplicadas o deformadas, puede existir algún problema.

---

# Problemas comunes durante SLAM

Durante el proceso pueden aparecer varios errores.

Por ejemplo:

```text
Mapa deformado

Paredes duplicadas

Robot mal ubicado

LIDAR desplazado

TF incorrecto

Odometría con demasiado error
```

No todos los errores significan que SLAM está fallando directamente.

Muchas veces debemos revisar primero los datos que recibe.

---

# Problema: no aparece el mapa

Si en RViz no aparece `/map`, podemos comprobar:

```bash
ros2 topic list
```

y verificar si existe:

```text
/map
```

También podemos revisar los nodos:

```bash
ros2 node list
```

Esto ayuda a comprobar si el nodo de SLAM realmente está ejecutándose.

---

# Problema: no aparecen datos del LIDAR

Podemos revisar:

```bash
ros2 topic echo /scan
```

Si `/scan` no publica información, SLAM no tendrá mediciones del entorno.

La cadena se rompería aquí:

```text
LIDAR
  │
  X
/scan
  │
  ▼
SLAM
```

Por eso comprobar los tópicos es una herramienta fundamental de diagnóstico.

---

# Problema: TF

También pueden aparecer errores como:

```text
No transform from ...
```

o:

```text
Frame does not exist
```

Aquí debemos recordar la entrada 13.

SLAM depende de transformaciones correctas.

Podemos revisar:

```bash
ros2 topic echo /tf
```

y:

```bash
ros2 topic echo /tf_static
```

También podemos utilizar herramientas de TF2 para comprobar relaciones específicas.

---

# Problema: el mapa se deforma

Si el robot se mueve demasiado rápido o la odometría presenta demasiado error, pueden aparecer paredes duplicadas.

Por ejemplo:

```text
Pared real:

████████████


Mapa:

████████████
   ████████████
```

Esto puede indicar que el sistema no está alineando correctamente las observaciones.

Mover el robot más lentamente puede ayudar.

---

# La importancia de cerrar recorridos

Cuando sea posible es útil volver a zonas ya exploradas.

Por ejemplo:

```text
Inicio ●───────────────┐
      │                │
      │                │
      │                │
      └───────────────●
                     regreso
```

Esto permite que el sistema vuelva a observar estructuras conocidas y puede ayudar a mejorar la consistencia del mapa.

---

# SLAM conecta todo lo aprendido

SLAM es uno de los primeros puntos donde prácticamente todos los conceptos anteriores se utilizan juntos.

```text
                 TurtleBot3
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
        /scan      /odom       TF2
          │          │          │
          └──────────┼──────────┘
                     ▼
                    SLAM
                     │
                     ▼
                    /map
                     │
                     ▼
                    RViz
```

Aquí aparecen:

```text
TurtleBot3
LIDAR
Odometría
TF2
RViz
Topics
Nodes
```

trabajando dentro de una sola aplicación.

---

# De teleoperación a autonomía

Hasta ahora seguimos moviendo manualmente el robot.

La estructura es:

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
  │
  ▼
SLAM
```

Pero una vez tenemos un mapa aparece una nueva posibilidad:

> **¿Podemos decirle al robot solamente dónde queremos que vaya y dejar que él calcule cómo llegar?**

Eso nos lleva al siguiente paso.

---

# Lo que aprendí

Antes de trabajar con SLAM pensaba que construir un mapa consistía simplemente en guardar las mediciones del LIDAR.

Pero el problema es mucho más interesante.

El robot debe combinar:

```text
LIDAR

Odometría

Transformaciones
```

mientras intenta estimar:

```text
dónde se encuentra
```

y simultáneamente:

```text
cómo es el entorno
```

Por eso SLAM significa:

```text
Simultaneous Localization And Mapping
```

---

# Flujo completo del mapeo

Podemos resumir nuestro proceso así:

```text
1. Encender TurtleBot3
          │
          ▼
2. Iniciar bringup
          │
          ▼
3. Comprobar /scan y /odom
          │
          ▼
4. Iniciar SLAM
          │
          ▼
5. Abrir RViz
          │
          ▼
6. Teleoperar el robot
          │
          ▼
7. Recorrer el entorno
          │
          ▼
8. Construir /map
          │
          ▼
9. Guardar el mapa
```

Este mapa será la base para la siguiente aplicación.

---

# Conclusión

SLAM permite que un robot móvil construya una representación del entorno mientras estima su posición dentro de ella.

En TurtleBot3 podemos combinar:

```text
/scan
/odom
/tf
```

para generar:

```text
/map
```

El mapa puede visualizarse en RViz y posteriormente guardarse en archivos como:

```text
.yaml
.pgm
```

Con esto dejamos de tener únicamente un robot capaz de moverse.

Ahora tenemos un robot que puede construir una representación de su entorno.

---

# Siguiente entrada

Ya tenemos:

```text
un robot

sensores

odometría

transformaciones

un mapa
```

Pero todavía lo hemos movido utilizando teleoperación.

Entonces aparece la siguiente pregunta:

> **¿Cómo hacemos para que el TurtleBot3 utilice ese mapa y se desplace de manera autónoma hasta un punto determinado?**

Para resolver este problema utilizaremos:

# Navigation2

**Entrada 19 - Navigation2**