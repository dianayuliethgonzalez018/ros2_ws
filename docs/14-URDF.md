# Entrada 14 - URDF

## Objetivo

Comprender qué es **URDF**, por qué ROS 2 necesita una descripción del robot y cómo podemos representar su estructura mediante enlaces (**links**) y articulaciones (**joints**).

En la entrada anterior aprendimos que **TF2** permite conocer la relación entre diferentes sistemas de coordenadas del robot.

Ahora aparece una nueva pregunta:

> ¿Cómo sabe ROS 2 cuáles son las partes que componen físicamente al robot y cómo están conectadas entre ellas?

Para responder esta pregunta necesitamos conocer **URDF**.

---

# La pregunta que me hice

Cuando observamos un robot podemos identificar fácilmente diferentes componentes.

Por ejemplo:

* Cuerpo.
* Ruedas.
* Sensores.
* Cámara.
* LIDAR.
* Brazos.
* Articulaciones.

Podemos imaginar un robot móvil sencillo:

```text
              LIDAR
                ●
                │
        ┌───────────────┐
        │     ROBOT     │
        └───────────────┘
          O           O
      rueda izq.   rueda der.
```

Nosotros podemos ver que todas esas piezas pertenecen al mismo robot.

Pero ROS 2 necesita una descripción que indique:

```text
¿Qué partes tiene el robot?

¿Cómo se llaman?

¿Dónde están ubicadas?

¿Cómo están conectadas?

¿Qué partes pueden moverse?
```

Aquí aparece **URDF**.

---

# ¿Qué es URDF?

**URDF** significa:

```text
Unified Robot Description Format
```

Es un formato utilizado en ROS para describir la estructura de un robot.

Un archivo URDF permite representar elementos como:

* Cuerpo del robot.
* Ruedas.
* Sensores.
* Articulaciones.
* Forma geométrica.
* Posición de los componentes.
* Propiedades físicas.

Los archivos URDF utilizan sintaxis **XML**.

Por ejemplo:

```xml
<robot name="mi_robot">

</robot>
```

Todo lo relacionado con la descripción del robot estará dentro de la etiqueta:

```xml
<robot>
```

---

# La idea principal de URDF

Una de las cosas más importantes que aprendí es que un robot en URDF se puede entender principalmente mediante dos elementos:

```text
LINKS
+
JOINTS
```

Los **links** representan las partes físicas del robot.

Los **joints** indican cómo se conectan esas partes.

Por ejemplo:

```text
base_link
    │
    ├── joint izquierdo
    │       │
    │       └── left_wheel
    │
    └── joint derecho
            │
            └── right_wheel
```

Esto comienza a parecerse al árbol que vimos cuando estudiamos TF2.

La diferencia es que ahora estamos describiendo **la estructura del robot**.

---

# ¿Qué es un link?

Un **link** representa una parte rígida del robot.

Por ejemplo:

```text
base_link
```

puede representar el cuerpo principal.

También podemos tener:

```text
left_wheel_link
right_wheel_link
base_scan
```

Cada uno representa una parte diferente.

La forma más sencilla de declarar un link es:

```xml
<link name="base_link">
</link>
```

Con esto ya existe una parte del robot llamada:

```text
base_link
```

Pero todavía no hemos indicado cómo se ve.

---

# Describiendo visualmente un link

Dentro de un link podemos utilizar:

```xml
<visual>
```

para indicar cómo queremos visualizarlo.

Por ejemplo:

```xml
<link name="base_link">

  <visual>

    <geometry>
      <box size="0.4 0.3 0.1"/>
    </geometry>

  </visual>

</link>
```

Aquí estamos diciendo que `base_link` tendrá forma de caja.

Las dimensiones son:

```text
X = 0.4 m
Y = 0.3 m
Z = 0.1 m
```

Podemos imaginarlo como:

```text
        ┌─────────────────┐
        │                 │
        │    base_link    │
        │                 │
        └─────────────────┘
```

---

# Geometrías básicas

URDF permite utilizar diferentes geometrías.

## Caja

```xml
<box size="0.4 0.3 0.1"/>
```

## Cilindro

```xml
<cylinder radius="0.05" length="0.02"/>
```

## Esfera

```xml
<sphere radius="0.05"/>
```

Estas formas son útiles para crear modelos sencillos y entender cómo funciona URDF antes de utilizar modelos 3D más complejos.

---

# Agregando color

También podemos definir propiedades visuales como el color.

Por ejemplo:

```xml
<material name="blue">
  <color rgba="0 0 1 1"/>
</material>
```

Los valores corresponden a:

```text
R = rojo
G = verde
B = azul
A = transparencia
```

Por ejemplo:

```text
1 0 0 1
```

representaría rojo.

Mientras que:

```text
0 0 1 1
```

representaría azul.

---

# ¿Qué es un joint?

Hasta ahora podemos crear varias partes del robot.

Por ejemplo:

```text
base_link

left_wheel_link

right_wheel_link
```

Pero todavía están completamente separadas.

Necesitamos indicar cómo se conectan.

Para eso utilizamos un:

```text
joint
```

Un joint relaciona dos links.

Uno será:

```text
parent
```

y el otro:

```text
child
```

Por ejemplo:

```xml
<joint name="left_wheel_joint" type="continuous">

  <parent link="base_link"/>

  <child link="left_wheel_link"/>

</joint>
```

Aquí estamos diciendo:

```text
base_link
    │
    │ left_wheel_joint
    │
    └── left_wheel_link
```

---

# Parent y child otra vez

En la entrada anterior vimos los conceptos:

```text
Parent
Child
```

cuando estudiamos TF2.

Ahora vuelven a aparecer.

Por ejemplo:

```xml
<parent link="base_link"/>
<child link="left_wheel_link"/>
```

Esto permite construir una estructura jerárquica.

```text
             base_link
             /       \
            /         \
           /           \
left_wheel_link   right_wheel_link
```

Esta relación posteriormente puede convertirse en transformaciones que ROS 2 puede utilizar mediante TF2.

Aquí comenzamos a ver cómo **URDF y TF2 están relacionados**.

---

# Tipos de joints

No todas las articulaciones se comportan igual.

URDF permite definir diferentes tipos.

Algunos de los más importantes son:

## fixed

No permite movimiento.

```xml
type="fixed"
```

Es útil para sensores que están instalados permanentemente sobre el robot.

Por ejemplo:

```text
base_link
    │
    │ fixed
    │
    └── base_scan
```

El LIDAR está unido al cuerpo y no debería moverse respecto al robot.

---

## continuous

Permite rotación continua.

```xml
type="continuous"
```

Es muy útil para ruedas.

```text
base_link
    │
    │ continuous
    │
    └── wheel_link
```

La rueda puede girar continuamente.

---

## revolute

Permite rotación, pero con límites.

```xml
type="revolute"
```

Puede utilizarse, por ejemplo, para una articulación que solo puede girar cierto número de grados.

---

## prismatic

Permite movimiento lineal.

```xml
type="prismatic"
```

En lugar de girar, la articulación se desplaza en una dirección.

---

# Posición de los componentes

También necesitamos indicar dónde está ubicado cada componente.

Para esto podemos utilizar:

```xml
<origin>
```

Por ejemplo:

```xml
<origin xyz="0 0 0.1" rpy="0 0 0"/>
```

Aquí encontramos dos conceptos:

```text
xyz
rpy
```

`xyz` representa la posición:

```text
x
y
z
```

Mientras que `rpy` representa la orientación:

```text
roll
pitch
yaw
```

Podemos pensar en:

```text
xyz → dónde está

rpy → cómo está orientado
```

---

# Agregando una rueda

Vamos a crear una rueda sencilla.

Primero declaramos el link:

```xml
<link name="left_wheel_link">

  <visual>

    <geometry>
      <cylinder radius="0.05" length="0.02"/>
    </geometry>

  </visual>

</link>
```

Ahora necesitamos conectarla al cuerpo.

```xml
<joint name="left_wheel_joint" type="continuous">

  <parent link="base_link"/>

  <child link="left_wheel_link"/>

  <origin xyz="0 0.15 0" rpy="1.5708 0 0"/>

  <axis xyz="0 1 0"/>

</joint>
```

Ahora nuestro modelo comienza a tener una estructura real.

```text
base_link
    │
    │
    └── left_wheel_link
```

---

# El eje de movimiento

Cuando una articulación puede moverse necesitamos indicar alrededor de qué eje ocurre el movimiento.

Para eso utilizamos:

```xml
<axis xyz="0 1 0"/>
```

Por ejemplo:

```text
X → 1 0 0
Y → 0 1 0
Z → 0 0 1
```

La elección depende de cómo esté orientado el componente.

Esto es especialmente importante para las ruedas.

---

# Construyendo un robot sencillo

Podemos reunir lo aprendido en un modelo pequeño:

```xml
<?xml version="1.0"?>

<robot name="mi_robot">

  <link name="base_link">

    <visual>

      <geometry>
        <box size="0.4 0.3 0.1"/>
      </geometry>

    </visual>

  </link>


  <link name="left_wheel_link">

    <visual>

      <geometry>
        <cylinder radius="0.05" length="0.02"/>
      </geometry>

    </visual>

  </link>


  <joint name="left_wheel_joint" type="continuous">

    <parent link="base_link"/>

    <child link="left_wheel_link"/>

    <origin xyz="0 0.16 0" rpy="1.5708 0 0"/>

    <axis xyz="0 1 0"/>

  </joint>

</robot>
```

Aunque todavía es un modelo muy sencillo, ya contiene los dos conceptos fundamentales:

```text
LINKS
+
JOINTS
```

---

# El árbol del robot

A medida que agregamos componentes podemos construir algo parecido a:

```text
                    base_link
                  /     |      \
                 /      |       \
                /       |        \
               /        |         \
left_wheel_link    base_scan    right_wheel_link
```

Cada componente puede tener su propio sistema de coordenadas.

Esto conecta directamente con lo que vimos en TF2.

URDF define:

```text
cómo está construido el robot
```

y TF2 permite trabajar con:

```text
las relaciones entre sus sistemas de coordenadas
```

---

# ¿Cómo utiliza ROS 2 el URDF?

ROS 2 necesita tener disponible la descripción del robot.

Normalmente esta descripción se publica mediante un parámetro llamado:

```text
robot_description
```

Uno de los nodos fundamentales para trabajar con esta información es:

```text
robot_state_publisher
```

Este nodo utiliza la descripción del robot para publicar las transformaciones correspondientes entre sus diferentes partes.

Podemos representar la idea así:

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
        /tf
     /tf_static
```

Aquí se conecta directamente la entrada anterior con esta.

---

# robot_state_publisher

Cuando tenemos un modelo URDF podemos utilizar:

```text
robot_state_publisher
```

para publicar el estado del robot.

Conceptualmente ocurre esto:

```text
URDF
 │
 │ describe links y joints
 ▼
robot_state_publisher
 │
 │ genera transformaciones
 ▼
TF2
```

Esto permite que otras herramientas de ROS 2 conozcan la estructura del robot.

---

# joint_state_publisher

Existe otra herramienta importante:

```text
joint_state_publisher
```

Su función está relacionada con el estado de las articulaciones.

Por ejemplo, si tenemos una rueda:

```text
left_wheel_joint
```

podemos necesitar conocer su posición o movimiento.

La información de las articulaciones normalmente se publica en:

```text
/joint_states
```

Esta información puede ser utilizada por `robot_state_publisher` para calcular las transformaciones correspondientes.

Podemos imaginar:

```text
/joint_states
      │
      ▼
robot_state_publisher
      │
      ▼
     TF2
```

---

# URDF en TurtleBot3

Cuando trabajamos con TurtleBot3 aparecen nombres que ahora tienen mucho más sentido.

Por ejemplo:

```text
base_footprint
base_link
base_scan
wheel_left_link
wheel_right_link
```

Estos nombres representan diferentes partes o referencias asociadas al robot.

Una estructura simplificada podría verse así:

```text
                 base_footprint
                       │
                       ▼
                   base_link
                 /     |      \
                /      |       \
               /       |        \
wheel_left_link    base_scan    wheel_right_link
```

`base_link` representa una referencia principal del cuerpo.

`base_scan` está relacionado con el sensor LIDAR.

`wheel_left_link` y `wheel_right_link` están relacionados con las ruedas.

Cuando anteriormente observamos mensajes relacionados con:

```text
robot_state_publisher
```

y nombres como:

```text
base_link
base_scan
wheel_left_link
wheel_right_link
```

en realidad ROS 2 estaba construyendo y publicando precisamente estas relaciones.

---

# Visual, collision e inertial

Hasta ahora nos hemos concentrado principalmente en:

```xml
<visual>
```

Pero un link puede contener más información.

Una estructura más completa puede ser:

```xml
<link name="base_link">

  <visual>
    ...
  </visual>

  <collision>
    ...
  </collision>

  <inertial>
    ...
  </inertial>

</link>
```

Cada sección tiene una función diferente.

## visual

Define cómo se ve el objeto.

```text
¿Cómo quiero visualizar esta pieza?
```

## collision

Define la geometría utilizada para detectar colisiones.

```text
¿Qué espacio físico ocupa?
```

## inertial

Define propiedades físicas relacionadas con:

```text
masa
inercia
centro de masa
```

Estas propiedades serán especialmente importantes cuando utilicemos simuladores.

---

# URDF y simulación

Un robot puede verse correctamente sin tener todas sus propiedades físicas bien configuradas.

Pero cuando queremos simularlo aparecen nuevas necesidades.

El simulador necesita saber cosas como:

```text
¿Cuánto pesa?

¿Cómo está distribuida su masa?

¿Dónde puede colisionar?

¿Cómo deben comportarse sus articulaciones?
```

Por eso las secciones:

```text
collision
inertial
```

se vuelven importantes.

Más adelante veremos esto cuando trabajemos con **Gazebo**.

---

# URDF y modelos 3D

No estamos limitados a cajas, cilindros y esferas.

También podemos utilizar modelos 3D mediante:

```xml
<mesh>
```

Por ejemplo:

```xml
<geometry>
  <mesh filename="package://mi_robot/meshes/base.stl"/>
</geometry>
```

Esto permite representar robots con geometrías mucho más cercanas al modelo físico real.

Sin embargo, para aprender URDF es mucho más sencillo comenzar con:

```text
box
cylinder
sphere
```

y después avanzar hacia modelos 3D.

---

# ¿Cómo comprobar un URDF?

Cuando escribimos un archivo URDF podemos cometer errores de sintaxis o de estructura.

Una herramienta útil es:

```bash
check_urdf robot.urdf
```

Si el archivo es válido podemos obtener información sobre el árbol del robot.

Por ejemplo:

```text
robot name is: mi_robot
---------- Successfully Parsed XML ---------------
root Link: base_link
```

Si existe algún error, esta herramienta puede ayudarnos a encontrarlo.

Dependiendo de la instalación puede ser necesario tener instaladas las herramientas correspondientes de URDF.

---

# Un error común: varios links sin joints

Podríamos escribir:

```xml
<link name="base_link"/>

<link name="left_wheel_link"/>

<link name="right_wheel_link"/>
```

Sintácticamente tenemos diferentes links.

Pero aparece un problema:

```text
base_link

left_wheel_link

right_wheel_link
```

No existe ninguna relación entre ellos.

Necesitamos los joints:

```text
             base_link
              /     \
             /       \
            /         \
left_wheel_link   right_wheel_link
```

Por eso los joints son fundamentales para construir el árbol del robot.

---

# Otro error común: parent y child incorrectos

Supongamos que queremos conectar:

```text
base_link
    │
    └── base_scan
```

Deberíamos tener conceptualmente:

```text
Parent = base_link
Child  = base_scan
```

Si invertimos incorrectamente estas relaciones podemos construir un árbol diferente al esperado.

Por eso es importante pensar siempre:

> ¿Cuál componente depende físicamente de cuál?

---

# Otro error común: orientación de las ruedas

Una rueda puede aparecer correctamente ubicada pero girar alrededor del eje equivocado.

Por ejemplo:

```xml
<axis xyz="1 0 0"/>
```

no representa el mismo eje que:

```xml
<axis xyz="0 1 0"/>
```

Por eso debemos tener en cuenta tanto:

```text
origin
```

como:

```text
axis
```

al configurar articulaciones móviles.

---

# De URDF a TF2

Ahora podemos conectar las dos últimas entradas.

En URDF definimos:

```text
base_link
    │
    └── base_scan
```

y especificamos la posición:

```xml
<origin xyz="0 0 0.1" rpy="0 0 0"/>
```

Después `robot_state_publisher` puede utilizar esa información para publicar la transformación.

Entonces TF2 puede conocer:

```text
base_link → base_scan
```

Esto explica de dónde salen muchas de las transformaciones que observamos anteriormente.

La relación completa puede verse como:

```text
URDF
 │
 ├── links
 │
 ├── joints
 │
 └── origin
       │
       ▼
robot_state_publisher
       │
       ▼
      TF2
       │
       ▼
 /tf y /tf_static
```

---

# Lo que aprendí

Antes veía archivos URDF como grandes archivos XML llenos de etiquetas.

Ahora entiendo que su idea principal es mucho más sencilla.

Un robot puede describirse principalmente mediante:

```text
LINKS
+
JOINTS
```

Los links representan las partes rígidas.

Los joints representan cómo están conectadas.

También aprendí que:

* `visual` define cómo se ve una pieza.
* `collision` define su geometría para colisiones.
* `inertial` contiene propiedades físicas.
* `origin` permite indicar posición y orientación.
* `axis` define el eje de movimiento de una articulación.
* `parent` y `child` construyen la jerarquía del robot.
* `robot_state_publisher` utiliza la descripción para publicar transformaciones.
* URDF y TF2 están directamente relacionados.

La idea que me queda es:

> URDF describe cómo está construido el robot y TF2 permite conocer la relación espacial entre sus diferentes partes.

---

# Siguiente entrada

Ahora tenemos una descripción del robot.

Sabemos cuáles son sus partes.

Sabemos cómo están conectadas.

Y sabemos que ROS 2 puede generar transformaciones entre ellas.

Pero aparece una nueva pregunta:

> ¿Cómo podemos ver todo esto gráficamente y comprobar que nuestro robot está realmente bien construido?

Para responder esta pregunta utilizaremos una de las herramientas de visualización más importantes de ROS 2:

**RViz**.

[Entrada 15 - RViz](15-RViz.md)