# Tensorflow con y sin GPU (NVIDIA) - Entorno contenerizado y usable desde Visual Studio Code Dev Containers

Entorno reproducible para prácticas de Deep Learning con TensorFlow + GPU NVIDIA (si se tiene) usando:
* Docker
* Dev Containers
* Visual Studio Code
* CUDA (si se tiene GPU NVIDIA)

Este repositorio permite desarrollar notebooks y aprovechar la aceleración GPU, si se tiene, en:
* Linux con (o sin) NVIDIA
* Windows con (o sin) NVIDIA  (WSL2 recomendado)
* EC2 con GPU NVIDIA (mínimo g4dn.xlarge y una AMI Ubuntu 22.04 con soporte GPU o similar)

El entorno está completamente contenerizado y no depende del Python del sistema.

## Objetivo

Proporcionar:
* Entorno reproducible
* Aceleración CUDA real si se dispone del hardware
* Compatible con VSCode Dev Containers (el desarrollador utiliza VSCode en el host pero la ejecución ocurre dentro del contenedor)
* Independiente del sistema operativo host

## Requisitos del Host

### Hardware

* 8GB de RAM mínimo (16 GB recomendado)
* Si se quiere aceleración GPU: GPU NVIDIA compatible con CUDA (Compute Capability >= 7.0 recomendado)

### Sistema Operativo

**Linux (recomendado)**

Ubuntu 20.04/22.04

**Windows**

* Windows 11
* WSL2
* Para aceleración gráfica: Drivers NVIDIA compatibles con CUDA

### Software necesario

#### **Docker**
```bash
docker --version
```
#### **NVIDIA Container Toolkit** (si se tiene GPU NVIDIA)

Este paso es **imprescindible** para que Docker pueda exponer la GPU al contenedor.

Comprobar:
```bash
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```
Si aparece la GPU, todo correcto. Si no, hay que instalar como se indica [aquí](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

#### **Visual Studio Code**

Extensiones necesarias:
* `ms-python.python`
* `ms-toolsai.jupyter`

## Cómo funciona el entorno

El proyecto usa:
```
.devcontainer/devcontainer.json
.devcontainer/Dockerfile
```
VSCode:
1. Construye la imagen Docker
2. Lanza el contenedor con la siguiente opción si detecta la GPU:
```
--gpus all
```
3. Monta el directorio de tu copia local del repo en `/workspace` dentro del contenedor

El fichero `Dockerfile` crea un contenedor con las versión de tensorflow para gpu y todas las dependencias necesarias pero si no hay GPU utiliza las versiones para CPU.

## Uso del entorno

### 1. Clona este repositorio

```bash
git clone https://github.com/jolmedilla/tensorflow-env-gpu-dev.git
cd tensorflow-env-gpu-dev
```

### 2. Abre VSCode
```bash
code .
```

### 3. Reabre en Dev Container
Desde VS Code:
1. Ctrl + Shift + P
2. Elige "Dev Containers: Reopen in Container:"
![Reopen in Dev Container](./images/open-dev-container.png)

La primera vez que hagas esto tardará bastante porque se baja la imagen base y luego instala los paquetes de panda, numpy, tensorflow, jupyter, etc. 

### 4. Abre un notebook en VSCode y asócialo a un kernel
En el repo hay un notebook de ejemplo para que pruebes si hay o no GPU en tu máquina (vía Docker NVIDIA toolkit), así que puedes utilizarlo para este paso, aunque ésto lo vas a tener que repetir con otro notebook distinto que quieras utilizar en este proyecto.
1. Abre el notebook en VSCode (primero debe estar en el mismo directorio del repo clonado)
2. En la esquina superior derecha de la ventana de edición, presiona donde dice "Select Kernel"
![Select Kernel](./images/select-kernel.png)

3. Ahora elige "Python Environments"
![Python Environments](./images/python-environments.png)

4. Es muy importante que ahora elijas `Python 3.10.12 /usr/bin/python3`. No elijas ninguno de los otros porque no tendrán instaladas las dependencias apropiadas.
![Python 3.10.12](./images/python-3.10.12-usr-bin.png)

A partir de ahora ya puedes trabajar con el notebook desde VSCode y se estará ejecutando todo dentro del contenedor

## Cómo puedo saber si mi notebook está realmente utilizando la GPU

Lo primero que puedes hacer es comprobar que realmente tienes la GPU NVIDIA disponible en tu contenedor, para ello desde la opción "Terminal" de la barra de menú de VSCode elige "New Terminal" y eso te abrirá un shell, ejecuta:
```bash
nvidia-smi
````
Te debería salir una pantalla con las características de tu GPU, versiónd del driver y versión de CUDA.

Además, cuando estés ejecutando un notebook y veas la evolución de tus epoch, puedes ejecutar el siguiente comando en esa misma terminal, que te irá mostrando lo mismo que antes, pero con un refresco de un segundo:
```bash
watch -n 1 nvidia-smi
```
Y verás el porcentaje de uso de tu GPU así como la cantidad de memoria de la misma en uso:
![Uso de la GPU](./images/nvidia-smi.png)

## Nota sobre el entorno en EC2

Como se comenta más arriba, este entorno se ha probado en una máquina EC2 de AWS con soporte GPU. Explico muy brevemente como replicar los pasos que yo he seguido, aunque no me voy a extender mucho en ello:
1. Lanza tu instancia de EC2, creando un par de claves ssh y configurando `.ssh/config` para que las utilice automáticamente contra la IP pública (no me extiendo aquí en ésto)
3. Abre VSCode en tu copia local de este repo
2. Lanza la conexión remota con Ctrl+Shit+P y luego buscando "Remote-SSH: Connect to Host..."
3. Pon la IP pública o el alias que hayas usado para ella en `.ssh/config`
4. Ahora se abre una nueva ventana de VSCode y puedes copiar el repo desde la ventana local de VSCode o el explorador de ficheros a la ventana remota de VSCode
5. A partir de ahora puedes continuar en el paso 3 de la sección de "Uso del entorno" como si estuvieras en local.

Es importante que elijas una AMI que ya incluya, además del soporte GPU, la instalación de docker o que la instales tú en la instancia una vez arrancada. Yo he utilizado la AMI "Deep Learning OSS Nvidia Driver AMI GPU TensorFlow 2.18 (Ubuntu 22.04) 20241213" con id "ami-00dab7aa6afcd90b5" que ya viene con docker pre-instalado. Para el hardware he utilizado una instancia `g4dn.xlarge` que tiene un coste razonable, de 0,526 $/hora más el coste del disco, almacenamiento EBS, que por una unidad `gp3` es de 0,08 $/mes/GB (por defecto, a mí me puso una de 50GB). No te olvides de parar la instancia cuando no estés trabajando en ella y cuando haya acabado del todo, elimina la instancia y asegúrate de que la unidad EBS también se ha eliminado.