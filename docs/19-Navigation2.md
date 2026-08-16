# Entrada 19 - Navigation2

## Objetivo

En la entrada anterior vimos cómo utilizar **SLAM** para construir un mapa del entorno con TurtleBot3.

Aprendimos que el robot puede combinar:

```text
LIDAR
Odometría
TF2
```

para generar:

```text
/map
```

Después guardamos el resultado en archivos como:

```text
.yaml
.pgm
```

Pero todavía movíamos el robot manualmente mediante teleoperación.

Entonces apareció una nueva pregunta:

> **¿Cómo hacemos para que el robot utilice ese mapa y se desplace de manera autónoma hasta un punto determinado?**

Aquí aparece:

# Navigation2

---

# ¿Qué es Navigation2?

**Navigation2**, normalmente conocido como:

```text
Nav2
```

es el conjunto de herramientas de navegación autónoma utilizado en ROS 2.

Su objetivo principal es permitir que un robot móvil pueda desplazarse desde su posición actual hasta una posición objetivo.

Podemos imaginar:

```text
Inicio
  ●
   \
    \
     \
      ─────────────● Objetivo
```

Pero para llegar al objetivo el robot necesita resolver varios problemas.

Por ejemplo:

```text
¿Dónde estoy?

¿Dónde quiero llegar?

¿Qué camino debo seguir?

¿Hay obstáculos?

¿Cómo controlo las ruedas?

¿Debo cambiar la ruta?
```

Navigation2 integra diferentes componentes para resolver estos problemas.

---

# De SLAM a Navigation2

En SLAM construimos un mapa.

Ahora Navigation2 puede utilizar ese mapa.

El flujo cambia de:

```text
SLAM
 │
 ▼
Construir mapa
```

a:

```text
Navigation2
 │
 ▼
Utilizar mapa
```

Podemos representarlo así:

```text
SLAM
 │
 ▼
mapa_laboratorio.yaml
 │
 ▼
Navigation2
 │
 ▼
Navegación autónoma
```

---

# ¿Qué necesita Navigation2?

Para navegar autónomamente necesitamos varios elementos.

Entre ellos:

```text
Mapa

Posición del robot

LIDAR

Odometría

TF2

Objetivo

Planificador

Controlador
```

Todo esto debe trabajar conjuntamente.

---

# Arquitectura general

Podemos imaginar la navegación como:

```text
                    OBJETIVO
                       │
                       ▼
                 Navigation2
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
        MAPA        LOCALIZACIÓN   LIDAR
          │            │            │
          └────────────┼────────────┘
                       │
                       ▼
                PLANIFICACIÓN
                       │
                       ▼
                    /cmd_vel
                       │
                       ▼
                  TurtleBot3
```

Navigation2 no es simplemente un único nodo.

Es un sistema compuesto por diferentes elementos.

---

# El mapa

Uno de los primeros elementos que necesitamos es el mapa.

En nuestro caso tenemos un archivo como:

```text
mapa_laboratorio.yaml
```

Este archivo hace referencia a la imagen del mapa y contiene parámetros adicionales.

Por ejemplo:

```yaml
image: mapa_laboratorio.pgm
resolution: 0.05
origin: [0.0, 0.0, 0.0]
occupied_thresh: 0.65
free_thresh: 0.25
```

Los valores exactos dependen del mapa generado.

---

# ¿Cómo sabe el robot dónde está?

Tener un mapa no es suficiente.

El robot necesita estimar:

```text
¿Dónde estoy dentro de ese mapa?
```

Este problema recibe el nombre de:

```text
localización
```

Podemos imaginar:

```text
MAPA

██████████████████████
█                    █
█         ?          █
█                    █
██████████████████████
```

El robot conoce el mapa, pero necesita determinar cuál es su posición dentro de él.

---

# AMCL

Una herramienta común para localización en Navigation2 es:

```text
AMCL
```

que significa:

```text
Adaptive Monte Carlo Localization
```

Su objetivo es estimar la posición del robot dentro de un mapa conocido.

Utiliza información como:

```text
Mapa

LIDAR

Odometría
```

Podemos representarlo así:

```text
/map
  │
  ▼

/scan ─────► AMCL ◄───── /odom
               │
               ▼
       posición estimada
```

---

# Pose inicial

Cuando iniciamos la navegación, en algunas situaciones necesitamos indicarle aproximadamente al sistema dónde se encuentra el robot.

En RViz podemos utilizar una herramienta como:

```text
2D Pose Estimate
```

Con ella seleccionamos una posición y orientación aproximadas.

Conceptualmente:

```text
MAPA
████████████████████████
█                      █
█         ●────→       █
█                      █
████████████████████████
```

El punto representa la posición.

La flecha representa la orientación.

---

# ¿Por qué debemos indicar la orientación?

No basta con saber:

```text
X
Y
```

También necesitamos saber hacia dónde está mirando el robot.

Por ejemplo:

```text
●────→
```

no representa lo mismo que:

```text
←────●
```

Aunque ambos robots estén ubicados en el mismo punto.

Por eso la pose contiene:

```text
posición
+
orientación
```

---

# Seleccionar un objetivo

Una vez el robot está localizado podemos indicarle dónde queremos que vaya.

En RViz podemos utilizar una herramienta como:

```text
2D Goal Pose
```

Seleccionamos:

```text
posición objetivo
+
orientación objetivo
```

Por ejemplo:

```text
Inicio
  ●


                     ● Objetivo
                     ↑
```

Entonces Navigation2 comienza a calcular cómo llegar.

---

# Planificación global

Una de las tareas principales es calcular una ruta desde el robot hasta el objetivo.

Podemos imaginar:

```text
Inicio
  ●
   \
    \
     \
      ─────────────● Objetivo
```

Pero en un entorno real pueden existir obstáculos.

Por ejemplo:

```text
████████████████████████
█                      █
█  ●                   █
█       ██████         █
█       ██████         █
█                 ●    █
████████████████████████
```

El robot no puede simplemente avanzar en línea recta.

Necesita encontrar una ruta alrededor del obstáculo.

---

# Planner

Navigation2 utiliza un componente relacionado con planificación.

Podemos simplificarlo como:

```text
Mapa
  │
  ▼
Planner
  │
  ▼
Ruta
```

La ruta puede verse conceptualmente como:

```text
Inicio
 ●
  \
   \
    └───────────────┐
                    │
                    │
                    └────● Objetivo
```

---

# Visualizar la ruta en RViz

Aquí vuelve a aparecer RViz.

Navigation2 puede mostrar la ruta calculada.

Entonces podemos observar:

```text
Robot
  │
  ▼
●─────────────┐
              │
              │
              └──────────● Objetivo
```

Esto permite comprobar si el plan generado tiene sentido.

---

# Planificación global y control local

Una ruta global no es suficiente.

El robot también debe controlar su movimiento en tiempo real.

Podemos separar conceptualmente:

```text
PLAN GLOBAL
    │
    ▼
¿por dónde debo ir?


CONTROL LOCAL
    │
    ▼
¿cómo debo moverme ahora?
```

Por ejemplo:

```text
Ruta global
-------------------------

● ───────────────────── ●


Pero aparece una persona:

● ───────── X ───────── ●
```

El sistema puede necesitar reaccionar localmente.

---

# Controller

El controlador transforma la trayectoria deseada en comandos de velocidad.

Podemos imaginar:

```text
Ruta
 │
 ▼
Controller
 │
 ▼
/cmd_vel
 │
 ▼
TurtleBot3
```

Esto conecta Navigation2 con lo aprendido en la entrada 17.

Recordemos:

```text
/cmd_vel
```

es el tópico utilizado para enviar comandos de velocidad al robot.

---

# Navigation2 no controla directamente los motores

Al igual que ocurría con teleoperación, Navigation2 no necesita enviar señales eléctricas directamente a cada motor.

La cadena es aproximadamente:

```text
Navigation2
     │
     ▼
  /cmd_vel
     │
     ▼
TurtleBot3
     │
     ▼
OpenCR
     │
     ▼
Dynamixel
     │
     ▼
Ruedas
```

Esto mantiene separada la navegación de la electrónica de bajo nivel.

---

# El LIDAR durante la navegación

El mapa contiene obstáculos conocidos.

Pero el entorno puede cambiar.

Por ejemplo, durante el mapeo no había una caja:

```text
MAPA ORIGINAL

████████████████████
█                  █
█                  █
█                  █
████████████████████
```

Ahora alguien coloca una caja:

```text
ENTORNO ACTUAL

████████████████████
█                  █
█        ███       █
█        ███       █
████████████████████
```

El LIDAR puede detectar ese nuevo obstáculo.

Esto permite al sistema reaccionar ante elementos que no estaban en el mapa original.

---

# Costmaps

Navigation2 utiliza representaciones conocidas como:

```text
costmaps
```

Un costmap asigna diferentes costos a las zonas del entorno.

Podemos imaginar:

```text
Libre            Bajo costo

Cerca obstáculo  Costo mayor

Obstáculo        Costo muy alto
```

Conceptualmente:

```text
██████████████████████
█....................█
█....++++++++++......█
█....+████████+......█
█....++++++++++......█
█....................█
██████████████████████
```

Aquí:

```text
████ → obstáculo

++++ → zona cercana al obstáculo

.... → espacio libre
```

Esto ayuda al robot a mantener cierta distancia de paredes y objetos.

---

# Global Costmap

Podemos tener un:

```text
Global Costmap
```

que representa información utilizada para planificación a mayor escala.

Conceptualmente:

```text
Mapa completo
      │
      ▼
Global Costmap
      │
      ▼
Planner
```

---

# Local Costmap

También podemos tener un:

```text
Local Costmap
```

que se concentra alrededor del robot.

Por ejemplo:

```text
      entorno local

      ┌───────────┐
      │           │
      │     ●     │
      │   robot   │
      │           │
      └───────────┘
```

Este costmap puede actualizarse continuamente con información del LIDAR.

---

# Obstáculos dinámicos

Supongamos que el robot está avanzando.

```text
Robot
  ●──────────────→
```

De repente aparece un obstáculo:

```text
Robot

  ●─────────█
            █
```

El sensor detecta el obstáculo.

```text
LIDAR
  │
  ▼
/scan
  │
  ▼
Local Costmap
```

El controlador puede necesitar modificar el movimiento.

Esto demuestra que Navigation2 no depende únicamente del mapa guardado.

---

# Behavior Trees

Navigation2 también utiliza un concepto llamado:

```text
Behavior Tree
```

o árbol de comportamiento.

Podemos verlo inicialmente como una estructura que organiza decisiones de navegación.

Por ejemplo:

```text
             Ir al objetivo
                   │
          ┌────────┴────────┐
          │                 │
        ¿Ruta?          ¿Obstáculo?
          │                 │
          ▼                 ▼
      avanzar          recuperación
```

No es necesario entender todos sus detalles para comenzar a utilizar Nav2, pero es importante saber que ayuda a organizar el comportamiento general.

---

# Recuperación

¿Qué ocurre si el robot queda bloqueado?

Por ejemplo:

```text
████████████
█     ●    █
█    ███   █
█    ███   █
████████████
```

Navigation2 puede ejecutar comportamientos de recuperación.

Dependiendo de la configuración puede:

```text
girar

retroceder

recalcular la ruta

esperar
```

El objetivo es intentar salir de una situación problemática.

---

# Flujo completo de navegación

Podemos representar el sistema de forma simplificada:

```text
                  Objetivo
                     │
                     ▼
                Navigation2
                     │
         ┌───────────┼───────────┐
         │           │           │
         ▼           ▼           ▼
       Planner    Controller   Costmaps
         │           │           ▲
         │           │           │
         │           │         /scan
         │           │
         └──────┬────┘
                │
                ▼
             /cmd_vel
                │
                ▼
            TurtleBot3
```

---

# Iniciar Navigation2

Primero debemos tener disponible el mapa generado anteriormente.

Por ejemplo:

```text
$HOME/maps/mapa_laboratorio.yaml
```

También debemos configurar el modelo:

```bash
export TURTLEBOT3_MODEL=burger
```

Y si estamos trabajando con varios equipos:

```bash
export ROS_DOMAIN_ID=30
```

---

# Robot físico

En la Raspberry Pi debemos tener funcionando el robot.

Por ejemplo:

```bash
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30

ros2 launch turtlebot3_bringup robot.launch.py
```

Esta terminal debe permanecer abierta.

---

# Iniciar Navigation2 con el mapa

Desde el computador podemos cargar el mapa utilizando:

```bash
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30
```

Después:

```bash
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
map:=$HOME/maps/mapa_laboratorio.yaml
```

La ruta exacta dependerá del lugar donde se haya guardado el mapa.

---

# ¿Qué ocurre al iniciar Navigation2?

Conceptualmente:

```text
mapa_laboratorio.yaml
          │
          ▼
       Map Server
          │
          ▼
         /map
          │
          ▼
     Navigation2
```

Al mismo tiempo el robot continúa proporcionando:

```text
/scan
/odom
/tf
```

Toda esta información se combina.

---

# RViz durante Navigation2

RViz vuelve a ser una herramienta fundamental.

Podemos visualizar:

```text
Mapa

Robot

LIDAR

TF

Costmaps

Ruta global

Objetivo
```

Esto hace que RViz sea una de las principales herramientas para comprobar el comportamiento del sistema.

---

# Primer paso: localización

Después de iniciar Navigation2 necesitamos comprobar que el robot está correctamente ubicado.

Podemos utilizar:

```text
2D Pose Estimate
```

y seleccionar aproximadamente:

```text
posición
+
orientación
```

del robot.

La representación en RViz debe corresponder con la posición física.

---

# ¿Cómo verificar la localización?

Una prueba sencilla consiste en observar el LIDAR sobre el mapa.

Supongamos que físicamente tenemos:

```text
Robot frente a pared
```

En RViz los puntos del LaserScan deberían coincidir aproximadamente con la pared del mapa.

```text
MAPA
████████████████████

          ● Robot
          ↑
          │
       LaserScan
```

Si los puntos aparecen desplazados, la localización puede ser incorrecta.

---

# Segundo paso: seleccionar objetivo

Después podemos utilizar:

```text
2D Goal Pose
```

Seleccionamos un punto libre del mapa.

Por ejemplo:

```text
████████████████████████
█                      █
█   ●                  █
█                      █
█                ★     █
████████████████████████
```

Donde:

```text
● = robot

★ = objetivo
```

---

# ¿Qué hace Navigation2 después?

El sistema recibe el objetivo.

Entonces:

```text
Objetivo
   │
   ▼
Planner
   │
   ▼
Ruta
   │
   ▼
Controller
   │
   ▼
/cmd_vel
   │
   ▼
Robot
```

Mientras el robot avanza, los sensores continúan actualizando la información.

---

# El robot no sigue simplemente una línea fija

Esto es importante.

Navigation2 no genera necesariamente una ruta y la sigue ciegamente hasta el final.

Mientras el robot se desplaza puede recibir nuevas observaciones.

Por ejemplo:

```text
Ruta original

●────────────────────★
```

Aparece un obstáculo:

```text
●────────█───────────★
         █
```

Dependiendo de la configuración, el sistema puede:

```text
detenerse

modificar localmente el movimiento

recalcular la ruta
```

---

# Comprobar /cmd_vel

Podemos observar los comandos que Navigation2 está generando.

```bash
ros2 topic echo /cmd_vel
```

Mientras el robot navega deberían aparecer valores de velocidad.

Por ejemplo:

```text
linear:
  x: ...

angular:
  z: ...
```

Esto permite comprobar que Navigation2 realmente está enviando órdenes de movimiento.

---

# Comprobar /odom

También podemos observar:

```bash
ros2 topic echo /odom
```

Esto permite verificar que el robot está actualizando su odometría mientras se mueve.

---

# Comprobar /scan

Podemos revisar:

```bash
ros2 topic echo /scan
```

para comprobar que el LIDAR continúa publicando información.

Esto es importante porque Navigation2 necesita detectar obstáculos.

---

# Comprobar TF

Podemos revisar:

```bash
ros2 topic echo /tf
```

o utilizar las herramientas de TF2 estudiadas anteriormente.

Durante navegación una estructura importante puede ser:

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

Si alguna transformación importante falta, Navigation2 puede presentar errores.

---

# Problema: el robot no se mueve

Un caso importante es cuando Navigation2 calcula una ruta pero el robot físico no se desplaza.

Podemos separar el diagnóstico.

Primero:

```bash
ros2 topic echo /cmd_vel
```

Si aparecen comandos:

```text
/cmd_vel funciona
```

pero las ruedas no se mueven, entonces el problema podría encontrarse más abajo en la cadena:

```text
Navigation2
     │
     ▼
/cmd_vel       ✅
     │
     ▼
TurtleBot3
     │
     X
Motores
```

Esto demuestra por qué es importante comprender la arquitectura completa del robot.

---

# Problema: el robot aparece mal ubicado

Si el robot aparece en una posición incorrecta dentro del mapa podemos revisar:

```text
Pose inicial

AMCL

TF

Odometría

LaserScan
```

También podemos volver a establecer:

```text
2D Pose Estimate
```

desde RViz.

---

# Problema: RobotModel en error

En RViz puede aparecer:

```text
RobotModel
Error
```

Esto puede deberse a problemas relacionados con:

```text
robot_description

TF

frames faltantes
```

Podemos comprobar:

```bash
ros2 topic list
```

y revisar:

```text
/tf
/tf_static
/joint_states
```

Esto conecta directamente con las entradas 13, 14 y 15.

---

# Problema: el robot virtual no coincide con el físico

Si la representación de RViz no coincide con la posición real debemos comprobar:

```text
Localización

TF

Odometría
```

No debemos asumir inmediatamente que el mapa está mal.

La posición representada depende de varias fuentes de información.

---

# Organización de terminales

Para trabajar con el robot físico podemos organizar varias terminales.

Por ejemplo:

```text
Terminal 1
----------
SSH a Raspberry
TurtleBot3 Bringup


Terminal 2
----------
Navigation2


Terminal 3
----------
RViz


Terminal 4
----------
Diagnóstico
ros2 topic list
ros2 topic echo ...
```

Esto ayuda a separar cada función.

---

# Flujo completo desde encender el robot

Podemos resumir el proceso:

```text
1. Encender TurtleBot3
          │
          ▼
2. Conectar Raspberry a la red
          │
          ▼
3. Iniciar TurtleBot3 Bringup
          │
          ▼
4. Verificar sensores
          │
          ▼
5. Cargar mapa
          │
          ▼
6. Iniciar Navigation2
          │
          ▼
7. Abrir RViz
          │
          ▼
8. Establecer pose inicial
          │
          ▼
9. Seleccionar objetivo
          │
          ▼
10. Robot navega autónomamente
```

---

# La evolución del proyecto

Al observar todo el proceso podemos ver cómo cada tema fue necesario para llegar hasta aquí.

```text
TF2
 │
 ▼
¿Cómo se relacionan los frames?


URDF
 │
 ▼
¿Cómo está construido el robot?


RViz
 │
 ▼
¿Cómo veo lo que ocurre?


Gazebo
 │
 ▼
¿Cómo lo pruebo en simulación?


TurtleBot3
 │
 ▼
¿Cómo lo llevo al robot real?


SLAM
 │
 ▼
¿Cómo construyo el mapa?


Navigation2
 │
 ▼
¿Cómo navego autónomamente?
```

Esta secuencia muestra que Navigation2 no es un tema aislado.

Depende de muchos conceptos anteriores.

---

# Del control manual a la autonomía

Al principio nuestro sistema era:

```text
PERSONA
  │
  ▼
TECLADO
  │
  ▼
/cmd_vel
  │
  ▼
ROBOT
```

Después de integrar SLAM y Navigation2 tenemos:

```text
PERSONA
  │
  ▼
OBJETIVO
  │
  ▼
NAVIGATION2
  │
  ▼
PLANIFICACIÓN
  │
  ▼
/cmd_vel
  │
  ▼
ROBOT
```

La persona ya no necesita controlar continuamente cada movimiento.

Solamente indica:

> **Quiero llegar aquí.**

El sistema se encarga de calcular cómo hacerlo.

---

# De percepción a acción

También podemos observar otro flujo importante:

```text
ENTORNO
   │
   ▼
LIDAR
   │
   ▼
/scan
   │
   ▼
Navigation2
   │
   ▼
Decisión
   │
   ▼
/cmd_vel
   │
   ▼
Motores
   │
   ▼
MOVIMIENTO
```

Esto representa uno de los principios fundamentales de la robótica móvil:

```text
Percibir
   ↓
Procesar
   ↓
Decidir
   ↓
Actuar
```

---

# Navigation2 integra todo el sistema

La arquitectura completa puede verse así:

```text
                          MAPA
                            │
                            ▼
                       Navigation2
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
        AMCL              Planner          Costmaps
          ▲                 │                 ▲
          │                 ▼                 │
       /scan              Ruta              /scan
          ▲                 │
          │                 ▼
        LIDAR            Controller
                            │
                            ▼
                         /cmd_vel
                            │
                            ▼
                       Raspberry Pi
                            │
                            ▼
                          OpenCR
                            │
                   ┌────────┴────────┐
                   ▼                 ▼
             Motor izquierdo    Motor derecho
                   │                 │
                   └────────┬────────┘
                            ▼
                         Movimiento
```

Esta arquitectura demuestra cómo los diferentes niveles del sistema trabajan juntos.

---

# Lo que aprendí

Antes de trabajar con Navigation2 podía pensar que la navegación autónoma era simplemente:

> seleccionar un punto y hacer que el robot avance.

Ahora entiendo que detrás existen varios procesos.

El robot necesita:

```text
conocer el mapa

localizarse

detectar obstáculos

calcular una ruta

seguir esa ruta

generar velocidades

actualizar su posición

reaccionar a cambios
```

Todo esto debe ocurrir de manera coordinada.

---

# Conceptos principales

Los principales conceptos que aprendí en esta entrada son:

```text
Navigation2
      │
      ├── Localización
      │
      ├── AMCL
      │
      ├── Planner
      │
      ├── Controller
      │
      ├── Global Costmap
      │
      ├── Local Costmap
      │
      ├── Behavior Trees
      │
      └── Recovery
```

Cada elemento cumple una función diferente dentro del proceso de navegación.

---

# Flujo final

Podemos resumir toda la aplicación de navegación de TurtleBot3 así:

```text
Mapa guardado
      │
      ▼
Navigation2
      │
      ▼
Localización
      │
      ▼
Objetivo
      │
      ▼
Planificación
      │
      ▼
Control
      │
      ▼
/cmd_vel
      │
      ▼
TurtleBot3
      │
      ▼
Movimiento
      │
      ▼
LIDAR + Odometría
      │
      └──────────────► Navigation2
```

El sistema funciona como un ciclo continuo.

---

# Conclusión

Navigation2 permite transformar un TurtleBot3 controlado manualmente en una plataforma capaz de desplazarse de manera autónoma dentro de un mapa conocido.

Para lograrlo utiliza información proveniente de:

```text
Mapa

LIDAR

Odometría

TF2

Localización
```

y genera:

```text
rutas

decisiones

comandos de velocidad
```

Finalmente estos comandos llegan al robot mediante:

```text
/cmd_vel
```

Con esto completamos el recorrido desde los fundamentos de visualización hasta una aplicación completa de navegación autónoma.

---

# Recorrido completo

Las entradas desarrolladas permiten observar una evolución progresiva:

```text
13 - TF2
      │
      ▼
Sistemas de coordenadas

14 - URDF
      │
      ▼
Descripción del robot

15 - RViz
      │
      ▼
Visualización

16 - Gazebo
      │
      ▼
Simulación

17 - TurtleBot3
      │
      ▼
Robot físico

18 - SLAM
      │
      ▼
Construcción del mapa

19 - Navigation2
      │
      ▼
Navegación autónoma
```

De esta manera pasamos de comprender cómo ROS 2 representa un robot hasta utilizar esas herramientas para resolver un problema real de robótica móvil:

> **hacer que un robot conozca su entorno y se desplace autónomamente dentro de él.**