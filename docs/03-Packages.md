# Entrada 03 - que es un Packages en ROS 2?

## Objetivo

Comprender qué es un **Package** en ROS 2, por qué existe, cómo se crea y cuál es la función de cada uno de los archivos que lo componen.

---

# Introducción

Después de entender qué es un **Workspace**, el siguiente concepto con el que me encontré fue el de **Package**.

En prácticamente todos los tutoriales aparecía un comando parecido a este:

```bash
ros2 pkg create ...
```

Lo ejecutaba y automáticamente aparecían varias carpetas y archivos.

Mi primera reacción fue preguntarme:

> **¿Por qué ROS 2 crea tantos archivos?**
>
> ¿No sería suficiente con crear una carpeta usando `mkdir` y comenzar a programar?


Un Package es mucho más que una carpeta con código.

---

# El problema

Imaginemos que queremos comenzar un proyecto.

Podríamos hacer simplemente:

```bash
mkdir mi_robot
```

y empezar a escribir archivos dentro.

Para nosotros eso puede ser suficiente.

Pero ROS 2 necesita responder preguntas como:

- ¿Cómo se llama este proyecto?
- ¿Qué versión tiene?
- ¿Quién es el autor?
- ¿Qué dependencias necesita?
- ¿Cómo debe compilarse?
- ¿Dónde debe instalarse?
- ¿Qué programas puede ejecutar?

Una carpeta común no responde ninguna de esas preguntas.

Por eso ROS 2 necesita algo más organizado.

---

# ¿Qué es un Package?

Un **Package** es la unidad básica de organización dentro de ROS 2.

Puede imaginarse como un pequeño proyecto independiente que agrupa todos los archivos relacionados con una funcionalidad específica del robot.

Dentro de un Package pueden encontrarse:

- Nodes
- Launch Files
- Archivos de configuración
- Librerías
- Mensajes
- Servicios
- Acciones
- Recursos
- Documentación

Todo aquello que pertenece a una misma funcionalidad suele mantenerse dentro del mismo Package.

Por ejemplo:

```text
Robot

├── navegación
├── cámara
├── control de motores
├── sensores
└── interfaz gráfica
```

En ROS 2 normalmente cada una de esas funcionalidades sería un Package diferente.

---

# ¿Por qué dividir el robot en Packages?

Imaginemos un robot grande.

Si todo el código estuviera mezclado en una sola carpeta terminaríamos con cientos o miles de archivos.

Sería muy difícil:

- encontrar errores;
- reutilizar código;
- trabajar en equipo;
- actualizar solamente una parte del sistema.

Dividir el robot en Packages permite que cada componente tenga una responsabilidad específica.

Por ejemplo:

```text
robot/

├── robot_camera
├── robot_navigation
├── robot_control
├── robot_sensors
└── robot_gui
```

Cada Package puede evolucionar de manera independiente.

---

# Creando mi primer Package en Python

El primer Package que creé fue:

```bash
cd ~/ROS2Dev/ros2_ws/src

ros2 pkg create --build-type ament_python mi_primer_paquete
```

Analizando el comando:

```text
ros2
│
├── pkg
│      Trabajar con Packages
│
├── create
│      Crear un nuevo Package
│
├── --build-type
│      Tipo de construcción
│
├── ament_python
│      Utilizar Python
│
└── mi_primer_paquete
       Nombre del Package
```

ROS 2 generó toda la estructura necesaria.

---

# La estructura creada

```text
mi_primer_paquete/

├── package.xml
├── setup.py
├── setup.cfg
├── resource/
├── test/
└── mi_primer_paquete/
    └── __init__.py
```

Cuando la vi por primera vez pensé que había demasiados archivos.

Sin embargo, cada uno cumple una función muy específica.

---

# package.xml

Este archivo puede considerarse la identidad del Package.

No contiene código.

Contiene información sobre el proyecto.

Por ejemplo:

```xml
<name>mi_primer_paquete</name>

<version>0.0.0</version>

<description>...</description>

<maintainer>...</maintainer>

<license>...</license>
```

Aquí ROS 2 puede conocer:

- nombre
- versión
- descripción
- mantenedor
- licencia
- dependencias
- tipo de construcción

Sin este archivo ROS 2 deja de reconocer la carpeta como un Package.

---

# ¿Qué ocurre si eliminamos `package.xml`?

Escribiendo este repositorio surgió una pregunta interesante.

y si eliminamos package.xml, que pasaria?:

```text
package.xml
```

y luego ejecutamos:

```bash
colcon build
```

¿Qué ocurre?

La respuesta es que ROS 2 ya no identifica esa carpeta como un Package válido.

En consecuencia:

- no podrá construirlo;
- no aparecerá instalado;
- no podrá ejecutarse posteriormente con `ros2 run`.

Esto demuestra que `package.xml` es uno de los archivos más importantes del Package.

---

# setup.py

Este archivo pertenece al sistema de empaquetado de Python.

Su función principal es indicar cómo debe instalarse el proyecto.

Aquí encontramos información como:

```python
setup(
    name=package_name,
    version='0.0.0',
)
```

También aparecen elementos importantes como:

```python
packages=find_packages()
```

que busca automáticamente los módulos de Python que deben instalarse.

---

# install_requires

Dentro de `setup.py` también aparece:

```python
install_requires=[
    'setuptools'
]
```

Esto indica qué herramientas necesita Python para poder instalar correctamente el proyecto.

En este caso solamente necesita `setuptools`.

No debe confundirse con las dependencias propias de ROS 2.

---

# entry_points

Probablemente la parte más importante del archivo sea:

```python
entry_points={
    'console_scripts': [
    ],
}
```

Aquí más adelante registraremos nuestros Nodes.

Por ejemplo:

```python
entry_points={
    'console_scripts':[
        'saludo = mi_primer_paquete.saludo:main'
    ],
}
```

Después podremos ejecutar:

```bash
ros2 run mi_primer_paquete saludo
```

Sin necesidad de buscar manualmente el archivo Python.

---

# setup.cfg

Este archivo contiene configuración adicional para la instalación.

Por ejemplo:

```ini
[develop]
script_dir=$base/lib/mi_primer_paquete

[install]
install_scripts=$base/lib/mi_primer_paquete
```

Aquí se define dónde serán instalados los ejecutables del Package.

No contiene código.

No contiene lógica.

Simplemente configura el proceso de instalación.

---

# resource/

Su función es permitir que ROS 2 registre correctamente el Package durante la instalación.

Normalmente nunca será necesario modificar su contenido manualmente.

---

# test/

Aquí aparecen varias pruebas automáticas.

Entre ellas:

- flake8
- pep257
- copyright

Inicialmente pensé que comprobaban si mi código estaba correctamente comentado.

Después entendí que realmente verifican reglas específicas.

Por ejemplo:

### flake8

Comprueba el estilo del código.

### pep257

Comprueba que la documentación siga las convenciones de Python.

### copyright

Verifica que existan los encabezados de licencia correspondientes.

Estas herramientas ayudan a mantener proyectos organizados cuando muchas personas trabajan sobre el mismo código.

---

# ¿Por qué existen dos carpetas con el mismo nombre?

Una de las cosas que más me confundió fue encontrar esto:

```text
mi_primer_paquete/

└── mi_primer_paquete/
```

La carpeta exterior representa el Package de ROS 2.

La carpeta interior representa el paquete de Python.

Dentro de ella aparecerán posteriormente todos nuestros archivos `.py`.

Además contiene:

```text
__init__.py
```

Este archivo indica que esa carpeta debe tratarse como un módulo de Python.

---

# Mi primer Package en C++

Después decidí crear un Package usando C++.

Utilicé:

```bash
ros2 pkg create --build-type ament_cmake mi_paquete_cpp
```

La estructura generada fue distinta.

```text
mi_paquete_cpp/

├── package.xml
├── CMakeLists.txt
├── include/
└── src/
```

> "Este Package tiene menos archivos."

---

# ¿Por qué tiene menos archivos?

Porque C++ utiliza otra herramienta para construir el proyecto.

Mientras Python utiliza:

```text
setuptools
```

C++ utiliza:

```text
CMake
```

Por eso desaparecen:

```text
setup.py

setup.cfg
```

y aparece:

```text
CMakeLists.txt
```

Este archivo será el encargado de indicar cómo debe compilarse el proyecto.

---

# include/

Otra diferencia importante es la carpeta:

```text
include/
```

Aquí normalmente se colocan los archivos de cabecera (`.hpp`).

Mientras que las implementaciones (`.cpp`) estarán dentro de:

```text
src/
```

Por ejemplo:

```text
include/
    motor.hpp

src/
    motor.cpp
```

Esta organización es propia de los proyectos escritos en C++.

---

# Python vs C++

Después de crear ambos Packages entendí algo importante.

No existen dos tipos de ROS 2 diferentes.

Existe un mismo concepto de Package.

Lo único que cambia es la tecnología utilizada para construir el código.

| Python | C++ |
|----------|------|
| ament_python | ament_cmake |
| setup.py | CMakeLists.txt |
| setup.cfg | CMakeLists.txt |
| Código Python | Código C++ |

Ambos siguen siendo Packages de ROS 2.

---

# ¿Cuándo utilizar Python?

Generalmente cuando buscamos:

- desarrollar rápido;
- crear prototipos;
- automatizar tareas;
- trabajar con inteligencia artificial;
- crear herramientas auxiliares.

Python permite avanzar muy rápido durante el desarrollo.

---

# ¿Cuándo utilizar C++?

Generalmente cuando necesitamos:

- máximo rendimiento;
- control de motores;
- procesamiento de sensores;
- algoritmos de alto rendimiento;
- aplicaciones donde el tiempo de respuesta es crítico.

C++ requiere más trabajo, pero ofrece un mejor rendimiento.

---

# ¿Se pueden mezclar?

Sí.

Y de hecho es lo más habitual.

Por ejemplo:

```text
Robot

├── Node de cámara (Python)
├── Node de IA (Python)
├── Node de navegación (C++)
├── Node de motores (C++)
└── Node de sensores (C++)
```

Todos ellos pueden comunicarse mediante Topics, Services o Actions sin importar el lenguaje en el que fueron escritos.

Esa es una de las mayores fortalezas de ROS 2.

---

# ¿Existen Packages para otros lenguajes?

Sí, aunque cuando se comienza con ROS 2 casi toda la documentación se centra en **Python** y **C++**, estos no son los únicos lenguajes con los que es posible desarrollar aplicaciones.

Lo importante es entender que un **Package** no depende del lenguaje de programación, sino de que exista una forma de integrarlo con el ecosistema de ROS 2.

Actualmente los lenguajes con mayor soporte son:

- **Python** (`rclpy`)
- **C++** (`rclcpp`)

Sin embargo, la comunidad también ha desarrollado bibliotecas para otros lenguajes como:

- Rust
- Java
- C#
- JavaScript (Node.js)
- Go

Cada uno dispone de diferentes niveles de soporte y mantenimiento.

---

## Entonces... ¿por qué casi todo el mundo usa Python o C++?

La respuesta es sencilla.

ROS 2 está diseñado principalmente alrededor de estos dos lenguajes.

Eso significa que:

- reciben nuevas características antes que otros;
- tienen la documentación oficial más completa;
- cuentan con la comunidad más grande;
- la mayoría de ejemplos están escritos en ellos;
- la mayoría de paquetes oficiales también los utilizan.

Por esta razón, aprender Python y C++ primero facilita muchísimo el aprendizaje de ROS 2.

---

## ¿Qué implicaciones tiene usar otro lenguaje?

Aunque técnicamente es posible crear Nodes en otros lenguajes, hay algunas desventajas que conviene conocer.

Por ejemplo:

- menos documentación disponible;
- menos ejemplos;
- menor soporte por parte de la comunidad;
- algunas bibliotecas oficiales pueden no estar disponibles;
- es posible que ciertas funcionalidades aparezcan más tarde o nunca lleguen.

En otras palabras, cuanto más nos alejamos de Python y C++, mayor será el trabajo que tendremos que realizar por nuestra cuenta.

---

# Bonus - ¿Cómo eliminar un Package?

Si ya no necesitas uno, puedes eliminarlo fácilmente.

## Paso 1. Ir a la carpeta `src`

```bash
cd ~/ROS2Dev/ros2_ws/src
```

## Paso 2. Eliminar el Package

Por ejemplo:

```bash
rm -rf mi_paquete_cpp
```

o

```bash
rm -rf mi_primer_paquete
```

> **Advertencia:** `rm -rf` elimina la carpeta y todo su contenido de forma permanente. Asegúrate de escribir correctamente el nombre del Package antes de ejecutar el comando.

## Paso 3. Limpiar el Workspace

Aunque el Package ya no exista en `src/`, todavía pueden quedar archivos de compilaciones anteriores.

Desde la raíz del Workspace ejecuta:

```bash
cd ~/ROS2Dev/ros2_ws

rm -rf build install log

colcon build

source install/setup.bash
```

Con esto se reconstruye el Workspace únicamente con los Packages que siguen existiendo.

---

# Lo que aprendí

Pensaba que un Package era simplemente una carpeta donde guardar código.

Ahora entiendo que un Package es la unidad de trabajo de ROS 2 y que existe para organizar el software del robot de una forma clara y reutilizable. También comprendí que cada archivo generado tiene una responsabilidad específica: unos describen el proyecto, otros indican cómo instalarlo y otros ayudan a que ROS 2 pueda reconocerlo y ejecutarlo correctamente.

Otra idea que me pareció muy interesante es que **ROS 2 mantiene el mismo concepto de Package sin importar el lenguaje de programación**. Ya sea Python o C++, la filosofía es la misma; únicamente cambia la herramienta encargada de construir el proyecto.

---

# Próxima entrada

En la siguiente entrada comenzaré a trabajar con el concepto más importante de ROS 2:

> **Los Nodes.**

Qué son, por qué ROS 2 divide un robot en múltiples Nodes en lugar de crear un único programa gigante y construiras tu primer Node tanto en Python como en C++, entendiendo cómo ROS 2 los ejecuta mediante el comando `ros2 run`.

Con esto empezaré a pasar de comprender la estructura de un proyecto a crear los primeros programa.

[Entrada 04 - Packages](docs/04-node.md)
