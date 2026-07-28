# Entrada 02 - Workspace

## Objetivo

Entender qué es un Workspace, por qué existe y cómo crear nuestro primer espacio de trabajo en ROS 2.

---

## La pregunta que me hice

Después de entender por qué existe ROS 2, apareció una nueva duda.

> ¿Dónde voy a desarrollar mi proyecto?

¿Basta con crear una carpeta cualquiera en mi computador o ROS 2 necesita una estructura específica?

---

## El problema

Cuando desarrollamos un proyecto pequeño normalmente basta con crear una carpeta.

Por ejemplo:

```text
MiProyecto/
├── main.py
├── config.py
└── README.md
```

Pero un proyecto de robótica puede crecer mucho con el tiempo.

ROS 2 necesita un lugar donde organizar el código, los archivos generados durante la compilación y los registros del proyecto.

Para resolver ese problema utiliza un **Workspace**.

---

# ¿Qué es un Workspace?

Un **Workspace** es el directorio principal donde desarrollaremos un proyecto con ROS 2.

Podemos imaginarlo como nuestro escritorio de trabajo.

Dentro de él iremos organizando todo el proyecto.

Más adelante aprenderemos cómo ROS 2 organiza el código dentro de este espacio.

---

# Creando mi primer Workspace

Para crear un Workspace basta con crear una carpeta y, dentro de ella, otra carpeta llamada `src`.

```bash
mkdir -p ~/ROS2Dev/ros2_ws/src
```

Veamos qué significa este comando.

- `mkdir` → crea directorios.
- `-p` → crea también las carpetas intermedias si no existen.
- `~/ROS2Dev/ros2_ws/src` → ruta donde se creará el Workspace.

Después entramos al Workspace.

```bash
cd ~/ROS2Dev/ros2_ws
```

Y comprobamos dónde estamos.

```bash
pwd
```

Resultado:

```text
/home/brayan/ROS2Dev/ros2_ws
```

---

# Explorando el Workspace

Una vez creado podemos ver su contenido.

```bash
find . -maxdepth 2
```

Después de trabajar un tiempo con ROS 2, mi Workspace tiene la siguiente estructura.

```text
ros2_ws/
├── build/
├── docs/
├── install/
├── log/
├── src/
├── .git/
├── README.md
└── .gitignore
```

No todas estas carpetas las creé yo.

Algunas pertenecen a ROS 2 y otras forman parte de mi proyecto.

---

# ¿Para qué sirve cada carpeta?

## src/

Es la carpeta donde escribiré el código del proyecto.

Será la carpeta en la que trabajaré la mayor parte del tiempo.

---

## build/

ROS 2 utiliza esta carpeta durante la compilación.

Aquí genera archivos temporales necesarios para construir el proyecto.

No debemos modificar su contenido manualmente.

---

## install/

Después de compilar, ROS 2 coloca aquí los archivos preparados para ejecutarse.

Tampoco debemos modificar esta carpeta manualmente.

---

## log/

Aquí ROS 2 guarda los registros de cada compilación.

Si ocurre algún error durante la construcción del proyecto, esta carpeta suele contener información útil para encontrar la causa.

---

## docs/

Esta carpeta no pertenece a ROS 2.

La he creado para documentar mi proceso de aprendizaje.

---

## README.md

Es la portada del repositorio.

Describe el propósito del proyecto y sirve como punto de entrada para cualquier persona que quiera seguir esta bitácora.

---

# ¿Qué hace `colcon build`?

Una vez que tengamos código dentro del Workspace necesitaremos construir el proyecto.

Para eso utilizaremos:

```bash
colcon build
```

De forma simplificada ocurre lo siguiente.

```text
src/
    │
    │ colcon build
    ▼
build/
    │
    ▼
install/
```

Durante este proceso:

- Se toma el código del proyecto.
- Se construye.
- Se generan archivos temporales en `build/`.
- El resultado final se coloca en `install/`.
- Se registran los mensajes del proceso en `log/`.

Por eso normalmente solo escribiremos código dentro de `src/`.

---

# ¿Por qué ejecutar `source install/setup.bash`?

Después de construir el proyecto es habitual ejecutar:

```bash
source install/setup.bash
```

Este comando no vuelve a compilar el proyecto.

Lo que hace es actualizar el entorno de la terminal para que ROS 2 pueda encontrar los cambios que acabamos de construir.

Podemos imaginar el proceso así.

```text
Escribo código
      │
      ▼
colcon build
      │
      ▼
Los archivos quedan en install/
      │
      ▼
source install/setup.bash
      │
      ▼
La terminal ya conoce el proyecto
```

Si olvidamos ejecutar este comando, ROS 2 puede comportarse como si nuestro proyecto todavía no existiera.

Durante el desarrollo es muy común utilizar esta secuencia.

```bash
colcon build
source install/setup.bash
```

---

# Lo que aprendí

Antes pensaba que un Workspace era simplemente una carpeta.

Ahora entiendo que es el espacio de trabajo donde ROS 2 organiza todo el desarrollo del proyecto.

También comprendí que:

- Yo escribo el código principalmente dentro de `src/`.
- ROS 2 administra automáticamente `build/`, `install/` y `log/`.
- Después de compilar normalmente debo ejecutar `source install/setup.bash` para que la terminal conozca los cambios realizados.

---

# Siguiente entrada

Ahora ya sé dónde desarrollaré mi proyecto.

La siguiente pregunta es:

> Si `src/` contiene mi código...

**¿Cómo organiza ROS 2 ese código?**

En la siguiente entrada conoceremos el concepto de **Package** y crearemos el primero.