# Manual de Docker

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> Docker permite empaquetar una aplicación con todo lo que necesita (sistema base, librerías, configuración) en un **contenedor** que se ejecuta igual en cualquier equipo. En Big Data es la forma más rápida de levantar herramientas como Spark, Kafka, bases de datos o Jupyter sin instalarlas "a mano". En este módulo lo usaremos sobre **WSL 2**, siguiendo el [manual de WSL](../wsl/manual_wsl.md).

---

## Índice

1. [¿Qué es Docker?](#1-qué-es-docker)
2. [Conceptos clave](#2-conceptos-clave)
3. [Instalación: dos opciones](#3-instalación-dos-opciones)
4. [Opción A: Docker Desktop + WSL 2](#4-opción-a-docker-desktop--wsl-2)
5. [Opción B: Docker Engine dentro de WSL (Ubuntu)](#5-opción-b-docker-engine-dentro-de-wsl-ubuntu)
6. [Imágenes](#6-imágenes)
7. [Contenedores](#7-contenedores)
8. [Puertos, variables de entorno y recursos](#8-puertos-variables-de-entorno-y-recursos)
9. [Persistencia de datos: volúmenes y *bind mounts*](#9-persistencia-de-datos-volúmenes-y-bind-mounts)
10. [Redes](#10-redes)
11. [Crear imágenes propias: el `Dockerfile`](#11-crear-imágenes-propias-el-dockerfile)
12. [Docker Compose: varios contenedores a la vez](#12-docker-compose-varios-contenedores-a-la-vez)
13. [Docker para Big Data: ejemplos del módulo](#13-docker-para-big-data-ejemplos-del-módulo)
14. [Mantenimiento y limpieza](#14-mantenimiento-y-limpieza)
15. [Solución de problemas frecuentes](#15-solución-de-problemas-frecuentes)
16. [Chuleta rápida](#16-chuleta-rápida)
17. [Referencias](#17-referencias)

---

## 1. ¿Qué es Docker?

Docker es una plataforma de **contenedores**. Un contenedor es un proceso aislado que ve su propio sistema de archivos, su propia red y sus propios procesos, pero **comparte el kernel** del sistema anfitrión.

### Contenedores frente a máquinas virtuales

| Característica | Máquina virtual | Contenedor |
|---|---|---|
| Qué virtualiza | El hardware completo (incluye un SO entero) | Solo el espacio de usuario; comparte el kernel |
| Tamaño | Gigabytes | Megabytes |
| Arranque | Minutos | Segundos o menos |
| Aislamiento | Muy alto | Alto (suficiente para la mayoría de usos) |
| Densidad | Pocas por equipo | Decenas o cientos por equipo |

```
      Máquinas virtuales                      Contenedores
┌────────┐┌────────┐┌────────┐     ┌────────┐┌────────┐┌────────┐
│ App A  ││ App B  ││ App C  │     │ App A  ││ App B  ││ App C  │
│ Libs   ││ Libs   ││ Libs   │     │ Libs   ││ Libs   ││ Libs   │
│ SO inv.││ SO inv.││ SO inv.│     └────────┘└────────┘└────────┘
└────────┘└────────┘└────────┘     ┌──────────────────────────┐
┌──────────────────────────┐       │      Docker Engine       │
│        Hipervisor        │       ├──────────────────────────┤
├──────────────────────────┤       │   SO anfitrión (kernel)  │
│      SO anfitrión        │       ├──────────────────────────┤
├──────────────────────────┤       │         Hardware         │
│         Hardware         │       └──────────────────────────┘
└──────────────────────────┘
```

### ¿Por qué Docker en Big Data?

- Levantar un **clúster de Spark, Kafka o Hadoop** en minutos, en un solo portátil.
- **El mismo entorno para todo el alumnado:** se acabó el "en mi equipo funciona".
- Probar versiones distintas de una herramienta sin romper nada.
- Es la base de los despliegues reales en la nube (Kubernetes).

---

## 2. Conceptos clave

- **Imagen:** plantilla de solo lectura con todo lo necesario para ejecutar una aplicación (p. ej., `postgres:16`). Se compone de **capas**.
- **Contenedor:** una instancia en ejecución de una imagen. De una misma imagen se pueden lanzar muchos contenedores.
- **Registro (*registry*):** almacén de imágenes. El público por defecto es **Docker Hub** (<https://hub.docker.com>); hay otros, como `quay.io` o `ghcr.io`.
- **Etiqueta (*tag*):** versión de una imagen: `nombre:etiqueta` (p. ej., `python:3.12-slim`). Si no se indica, se usa `latest`.
- **Dockerfile:** receta de texto para construir una imagen propia.
- **Volumen:** almacenamiento persistente gestionado por Docker; los datos sobreviven aunque se borre el contenedor.
- **Docker Compose:** herramienta para definir y levantar **varios contenedores** con un archivo YAML.
- **Docker Engine:** el servicio (*daemon* `dockerd`) que ejecuta los contenedores. El comando `docker` es el cliente que habla con él.

**Analogía:** la imagen es la **clase** y el contenedor es el **objeto**. El Dockerfile es el **código fuente** de la clase.

---

## 3. Instalación: dos opciones

| | **Opción A: Docker Desktop** | **Opción B: Docker Engine en WSL** |
|---|---|---|
| Qué es | Aplicación de Windows con interfaz gráfica que usa WSL 2 por debajo | Docker instalado directamente en Ubuntu, como en un servidor Linux |
| Facilidad | ✅ Muy sencilla | Algo más de pasos |
| Interfaz gráfica | Sí | No (solo terminal) |
| `docker` desde PowerShell | Sí | No (solo dentro de WSL) |
| Consumo de recursos | Mayor | Menor |
| Licencia | Gratis para uso personal, educativo y empresas pequeñas; las grandes requieren suscripción | Software libre (Apache 2.0) |

> ⚠️ **Elige solo una.** Tener las dos a la vez provoca conflictos (dos *daemons* distintos). Si ya usas Docker Desktop, no instales Docker Engine en Ubuntu, y al revés.

Requisito común: tener **WSL 2 con Ubuntu** instalado y actualizado (`wsl --update`).

---

## 4. Opción A: Docker Desktop + WSL 2

1. Descarga **Docker Desktop for Windows** desde <https://www.docker.com/products/docker-desktop/>.
2. Instálalo y deja marcada la opción **"Use WSL 2 instead of Hyper-V"**. Reinicia si lo pide.
3. Abre Docker Desktop y acepta las condiciones.
4. En **Settings → General**, comprueba que está marcado **"Use the WSL 2 based engine"**.
5. En **Settings → Resources → WSL Integration**, activa la integración con **Ubuntu** y pulsa *Apply & restart*.
6. Abre la terminal de Ubuntu y comprueba:

   ```bash
   docker --version
   docker compose version
   docker run hello-world
   ```

Notas:

- Docker Desktop tiene que estar **abierto** para que `docker` funcione.
- La memoria y CPU que usa Docker Desktop se limitan desde el `.wslconfig` de WSL (ver el manual de WSL, sección 10). Docker recomienda activar `autoMemoryReclaim`.

---

## 5. Opción B: Docker Engine dentro de WSL (Ubuntu)

> Todos los comandos se ejecutan **en la terminal de Ubuntu**. Hace falta tener **systemd activado** en WSL (`/etc/wsl.conf` → `[boot]` `systemd=true`; viene activado en las versiones recientes de Ubuntu).

### 5.1 Quitar paquetes antiguos o no oficiales

```bash
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 \
  docker-doc docker-buildx podman-docker containerd runc | cut -f1)
```

(Si dice que no hay nada que desinstalar, no pasa nada.)

### 5.2 Añadir el repositorio oficial de Docker

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<FIN
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
FIN

sudo apt update
```

### 5.3 Instalar Docker

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 5.4 Usar Docker sin `sudo`

```bash
sudo usermod -aG docker $USER
```

Cierra la terminal y vuelve a abrirla (o ejecuta `wsl --shutdown` desde PowerShell y vuelve a entrar) para que se aplique el grupo.

> ⚠️ Pertenecer al grupo `docker` equivale a tener permisos de administrador en la máquina. Es aceptable en un equipo de prácticas, no en un servidor compartido.

### 5.5 Arranque del servicio y verificación

```bash
sudo systemctl enable --now docker   # Arranca Docker y lo activa al iniciar
systemctl status docker              # Debe indicar "active (running)"
docker run hello-world
```

Si todo va bien, aparece el mensaje **"Hello from Docker!"**.

---

## 6. Imágenes

| Comando | Descripción |
|---|---|
| `docker search postgres` | Busca imágenes en Docker Hub |
| `docker pull postgres:16` | Descarga una imagen |
| `docker images` (o `docker image ls`) | Lista las imágenes locales |
| `docker image inspect postgres:16` | Información detallada (capas, variables, puertos…) |
| `docker history postgres:16` | Muestra las capas de la imagen |
| `docker rmi postgres:16` | Borra una imagen |
| `docker tag miapp:1.0 usuario/miapp:1.0` | Añade otra etiqueta a una imagen |
| `docker push usuario/miapp:1.0` | Sube una imagen a un registro (requiere `docker login`) |

**Consejos para elegir imágenes:**

- Usa preferentemente **imágenes oficiales** (marcadas como *Docker Official Image* o *Verified Publisher*).
- **Fija siempre la versión** (`postgres:16`, no `postgres:latest`): `latest` cambia con el tiempo y puede romper las prácticas.
- Las variantes `-slim` o `-alpine` ocupan mucho menos.

---

## 7. Contenedores

### 7.1 Crear y ejecutar: `docker run`

```bash
docker run hello-world                         # Ejecuta y termina
docker run -it ubuntu:24.04 bash               # Interactivo: abre una shell dentro
docker run -d --name web -p 8080:80 nginx      # En segundo plano, con nombre y puerto
docker run --rm -it python:3.12 python         # Se borra solo al salir
```

Opciones más usadas de `docker run`:

| Opción | Significado |
|---|---|
| `-d` | *Detached*: en segundo plano |
| `-it` | Interactivo con terminal |
| `--name <nombre>` | Nombre del contenedor (si no, Docker inventa uno) |
| `--rm` | Borra el contenedor al pararlo |
| `-p <host>:<contenedor>` | Publica un puerto |
| `-e VAR=valor` | Variable de entorno |
| `-v <origen>:<destino>` | Monta un volumen o una carpeta |
| `--network <red>` | Conecta a una red concreta |
| `--restart unless-stopped` | Reinicia el contenedor automáticamente |

### 7.2 Gestionar contenedores

| Comando | Descripción |
|---|---|
| `docker ps` | Contenedores en ejecución |
| `docker ps -a` | Todos, también los parados |
| `docker stop web` | Para un contenedor |
| `docker start web` | Arranca un contenedor parado |
| `docker restart web` | Reinicia |
| `docker rm web` | Borra un contenedor parado (`-f` para forzar) |
| `docker logs -f web` | Muestra los *logs* en directo (`Ctrl + C` para salir) |
| `docker exec -it web bash` | Abre una shell **dentro** de un contenedor en marcha (si no hay `bash`, usa `sh`) |
| `docker cp fichero.csv web:/tmp/` | Copia archivos entre el equipo y el contenedor |
| `docker inspect web` | Información detallada (IP, volúmenes, variables…) |
| `docker stats` | Consumo de CPU y memoria en directo |
| `docker top web` | Procesos del contenedor |

### 7.3 Ciclo de vida

```
            docker run
 imagen ───────────────▶ en ejecución ──docker stop──▶ parado ──docker rm──▶ (eliminado)
                              ▲                            │
                              └───────docker start─────────┘
```

> **Importante:** los cambios hechos dentro de un contenedor **se pierden al borrarlo**. Para conservar datos, usa volúmenes (sección 9).

---

## 8. Puertos, variables de entorno y recursos

### 8.1 Publicar puertos

```bash
docker run -d --name web -p 8080:80 nginx
```

`-p 8080:80` significa: **puerto 8080 del equipo** → **puerto 80 del contenedor**. Abre <http://localhost:8080> en el navegador de Windows.

### 8.2 Variables de entorno

Muchas imágenes se configuran con variables:

```bash
docker run -d --name bd \
  -e POSTGRES_USER=alumno \
  -e POSTGRES_PASSWORD=abc123 \
  -e POSTGRES_DB=ventas \
  -p 5432:5432 \
  postgres:16
```

También se pueden leer de un archivo: `--env-file .env`.

### 8.3 Limitar recursos

```bash
docker run -d --name spark --memory 2g --cpus 2 apache/spark:4.1.3-python3 sleep infinity
```

---

## 9. Persistencia de datos: volúmenes y *bind mounts*

| Tipo | Sintaxis | Dónde están los datos | Uso típico |
|---|---|---|---|
| **Volumen** | `-v datos_pg:/var/lib/postgresql/data` | Gestionados por Docker | Datos de bases de datos |
| **Bind mount** | `-v $(pwd)/notebooks:/home/jovyan/work` | Una carpeta tuya | Código, *notebooks*, datasets que editas |
| **tmpfs** | `--tmpfs /tmp` | En memoria (se pierden) | Datos temporales |

### 9.1 Volúmenes

```bash
docker volume create datos_pg
docker run -d --name bd -e POSTGRES_PASSWORD=abc123 -v datos_pg:/var/lib/postgresql/data postgres:16
docker volume ls
docker volume inspect datos_pg
docker volume rm datos_pg          # Solo si ningún contenedor lo usa
```

Aunque borres el contenedor `bd` y crees otro con el mismo volumen, los datos siguen ahí.

### 9.2 *Bind mounts*

```bash
mkdir -p ~/bigdata/datos
docker run --rm -it -v ~/bigdata/datos:/datos ubuntu:24.04 ls /datos
```

> **Rendimiento en WSL:** monta carpetas del **sistema de archivos de Linux** (`~/...`), **no** de `/mnt/c/...`. Con carpetas de Windows el acceso es mucho más lento.

---

## 10. Redes

Los contenedores de una misma red definida por el usuario **se encuentran por su nombre** (Docker hace de DNS).

```bash
docker network create red_bd
docker run -d --name bd --network red_bd -e POSTGRES_PASSWORD=abc123 postgres:16
docker run --rm -it --network red_bd postgres:16 psql -h bd -U postgres
```

En el segundo contenedor, `-h bd` funciona porque ambos están en `red_bd`.

| Comando | Descripción |
|---|---|
| `docker network ls` | Lista las redes |
| `docker network create <red>` | Crea una red |
| `docker network inspect <red>` | Contenedores conectados, subred… |
| `docker network connect <red> <contenedor>` | Conecta un contenedor ya creado |
| `docker network rm <red>` | Borra una red |

Tipos de red principales: `bridge` (por defecto, aislada), `host` (usa directamente la red del anfitrión) y `none` (sin red).

---

## 11. Crear imágenes propias: el `Dockerfile`

### 11.1 Instrucciones principales

| Instrucción | Para qué sirve |
|---|---|
| `FROM` | Imagen base (siempre la primera) |
| `WORKDIR` | Directorio de trabajo dentro de la imagen |
| `COPY` | Copia archivos del equipo a la imagen |
| `RUN` | Ejecuta un comando **al construir** la imagen (instalar paquetes…) |
| `ENV` | Variable de entorno |
| `EXPOSE` | Documenta el puerto que usa la aplicación |
| `USER` | Usuario con el que se ejecuta |
| `CMD` | Comando por defecto **al arrancar** el contenedor |
| `ENTRYPOINT` | Ejecutable fijo del contenedor (`CMD` pasa a ser sus argumentos) |

### 11.2 Ejemplo: script de análisis con pandas

Estructura del proyecto:

```
analisis/
├── Dockerfile
├── requirements.txt
├── .dockerignore
└── analisis.py
```

`requirements.txt`:

```
pandas==2.2.3
```

`analisis.py`:

```python
import pandas as pd

df = pd.DataFrame({"ciudad": ["Vigo", "Lugo", "Vigo", "Ourense"], "ventas": [120, 80, 200, 50]})
print(df.groupby("ciudad")["ventas"].sum().sort_values(ascending=False))
```

`Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Primero las dependencias: esta capa se reutiliza (caché) si no cambian
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Después el código
COPY analisis.py .

CMD ["python", "analisis.py"]
```

`.dockerignore` (lo que no se copia a la imagen):

```
__pycache__/
*.pyc
.venv/
datos/
```

Construir y ejecutar:

```bash
cd analisis
docker build -t analisis:1.0 .
docker run --rm analisis:1.0
```

Salida esperada:

```
ciudad
Vigo       320
Lugo        80
Ourense     50
Name: ventas, dtype: int64
```

### 11.3 Buenas prácticas

- Imágenes base **pequeñas y con versión fija** (`python:3.12-slim`).
- Ordena las instrucciones de **lo que menos cambia a lo que más cambia**, para aprovechar la caché de capas.
- Junta comandos `RUN` relacionados y limpia en la misma capa (`apt-get install ... && rm -rf /var/lib/apt/lists/*`).
- Usa `.dockerignore`.
- **Nunca** metas contraseñas ni claves en la imagen; pásalas con variables de entorno o secretos.

---

## 12. Docker Compose: varios contenedores a la vez

Compose describe una aplicación completa (servicios, redes, volúmenes) en un archivo `compose.yaml` (también vale `docker-compose.yml`). Se usa con `docker compose` (con espacio; la versión antigua `docker-compose` con guion está obsoleta).

### 12.1 Ejemplo: PostgreSQL + Adminer

`compose.yaml`:

```yaml
services:
  bd:
    image: postgres:16
    environment:
      POSTGRES_USER: alumno
      POSTGRES_PASSWORD: abc123
      POSTGRES_DB: ventas
    volumes:
      - datos_pg:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  adminer:
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - bd

volumes:
  datos_pg:
```

```bash
docker compose up -d
```

Abre <http://localhost:8081> e inicia sesión con servidor `bd`, usuario `alumno` y contraseña `abc123`. Compose crea automáticamente una red común, así que `adminer` encuentra a `bd` por su nombre.

### 12.2 Comandos de Compose

> Se ejecutan en la carpeta donde está el `compose.yaml`.

| Comando | Descripción |
|---|---|
| `docker compose up -d` | Crea y arranca todos los servicios en segundo plano |
| `docker compose ps` | Estado de los servicios |
| `docker compose logs -f <servicio>` | *Logs* en directo |
| `docker compose exec <servicio> bash` | Shell dentro de un servicio |
| `docker compose stop` / `start` | Para / arranca sin borrar |
| `docker compose down` | Para y borra contenedores y red (los volúmenes se conservan) |
| `docker compose down -v` | ⚠️ Además borra los volúmenes (se pierden los datos) |
| `docker compose up -d --scale worker=3` | Lanza varias réplicas de un servicio |
| `docker compose pull` | Descarga las versiones nuevas de las imágenes |
| `docker compose config` | Valida y muestra el archivo final |

---

## 13. Docker para Big Data: ejemplos del módulo

### 13.1 Jupyter con PySpark en un solo contenedor

El proyecto Jupyter publica imágenes con Spark ya instalado (en `quay.io`):

```bash
mkdir -p ~/bigdata/notebooks
docker run -d --name jupyter-spark \
  -p 8888:8888 -p 4040:4040 \
  -v ~/bigdata/notebooks:/home/jovyan/work \
  quay.io/jupyter/pyspark-notebook
docker logs jupyter-spark 2>&1 | grep "127.0.0.1:8888"
```

Copia la URL con el *token* en el navegador. Los *notebooks* que guardes en `work/` quedan en `~/bigdata/notebooks`. Con un trabajo en marcha, la interfaz de Spark está en <http://localhost:4040>.

### 13.2 Clúster de Spark (1 *master* + N *workers*) con Compose

`~/bigdata/spark-cluster/compose.yaml`:

```yaml
services:
  spark-master:
    image: apache/spark:4.1.3-python3
    container_name: spark-master
    command: /opt/spark/bin/spark-class org.apache.spark.deploy.master.Master
    ports:
      - "8080:8080"   # Interfaz web del master
      - "7077:7077"   # Puerto del clúster

  worker:
    image: apache/spark:4.1.3-python3
    command: /opt/spark/bin/spark-class org.apache.spark.deploy.worker.Worker spark://spark-master:7077
    environment:
      SPARK_WORKER_CORES: "1"
      SPARK_WORKER_MEMORY: "1g"
    depends_on:
      - spark-master
```

Levantar el clúster con **dos *workers***:

```bash
cd ~/bigdata/spark-cluster
docker compose up -d --scale worker=2
```

Abre <http://localhost:8080>: deben aparecer **2 workers** en estado *ALIVE*.

Lanzar un trabajo de ejemplo (cálculo de π) al clúster:

```bash
docker exec spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/examples/src/main/python/pi.py 10
```

Al final de la salida aparece algo como `Pi is roughly 3.14...`. En la interfaz web se ve la aplicación en *Completed Applications*.

Parar el clúster:

```bash
docker compose down
```

> Fija la **misma versión de Spark** en todos los servicios (y en el cliente, si te conectas desde fuera): mezclar versiones da errores de serialización difíciles de diagnosticar.

### 13.3 Otras imágenes útiles en el módulo

| Herramienta | Imagen de partida | Uso |
|---|---|---|
| PostgreSQL | `postgres` | Base de datos relacional de origen/destino |
| MongoDB | `mongo` | Base de datos NoSQL documental |
| Apache Kafka | `apache/kafka` | Ingesta de datos en *streaming* |
| Apache Spark | `apache/spark` | Procesamiento distribuido |
| Jupyter + Spark | `quay.io/jupyter/pyspark-notebook` | Análisis interactivo |
| MinIO | `minio/minio` | Almacenamiento de objetos compatible con S3 (*data lake*) |

> Consulta siempre en Docker Hub la etiqueta de versión vigente y fíjala en tus `compose.yaml`.

---

## 14. Mantenimiento y limpieza

Docker acumula imágenes, contenedores parados y volúmenes que ocupan mucho disco.

```bash
docker system df                 # Cuánto ocupa cada cosa
docker container prune           # Borra contenedores parados
docker image prune               # Borra imágenes "colgantes" (sin etiqueta)
docker image prune -a            # Borra todas las imágenes sin contenedor que las use
docker volume prune              # ⚠️ Borra volúmenes no usados (¡datos!)
docker system prune              # Contenedores parados + redes + imágenes colgantes + caché
docker system prune -a --volumes # ⚠️ Limpieza total
```

> **En WSL:** aunque borres imágenes, el disco virtual de WSL no se reduce solo. Activa `sparseVhd=true` en `.wslconfig` (manual de WSL, sección 10).

**Desinstalar Docker Engine (opción B):**

```bash
sudo apt purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm -rf /var/lib/docker /var/lib/containerd
sudo rm /etc/apt/sources.list.d/docker.sources /etc/apt/keyrings/docker.asc
```

**Desinstalar Docker Desktop (opción A):** *Configuración de Windows → Aplicaciones → Docker Desktop → Desinstalar*.

---

## 15. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `Cannot connect to the Docker daemon at unix:///var/run/docker.sock` | El servicio no está arrancado | Opción A: abrir Docker Desktop. Opción B: `sudo systemctl start docker` |
| `permission denied while trying to connect to the Docker daemon socket` | El usuario no está en el grupo `docker` | `sudo usermod -aG docker $USER` y reabrir la terminal |
| `docker: command not found` en Ubuntu con Docker Desktop | Falta la integración con WSL | *Settings → Resources → WSL Integration → Ubuntu* |
| `port is already allocated` / `address already in use` | Otro proceso o contenedor usa ese puerto | Cambiar el puerto del host (`-p 8082:80`) o parar el otro contenedor |
| `Conflict. The container name "/web" is already in use` | Ya existe un contenedor con ese nombre (aunque esté parado) | `docker rm web` o usar otro nombre |
| El contenedor se para nada más arrancar | El proceso principal termina o falla | `docker logs <contenedor>` para ver el error |
| Los datos desaparecen al recrear el contenedor | No se usó volumen | Montar un volumen (sección 9) |
| `no space left on device` | Imágenes y caché acumuladas | `docker system prune`, `docker system df` |
| Todo va muy lento con *bind mounts* | Carpeta en `/mnt/c` | Mover el proyecto a `~/` dentro de Linux |
| WSL consume demasiada RAM con Docker | Sin límite de memoria | Configurar `memory` y `autoMemoryReclaim` en `.wslconfig` |
| Error al instalar: conflicto con `docker.io` o `containerd` | Paquetes no oficiales instalados | Repetir el paso 5.1 |

Diagnóstico general:

```bash
docker version        # Cliente y servidor (si el servidor no aparece, el daemon no responde)
docker info
docker ps -a
docker logs <contenedor>
```

---

## 16. Chuleta rápida

```bash
# ---------- Imágenes ----------
docker pull <imagen>:<tag>              # Descargar
docker images                           # Listar
docker build -t <nombre>:<tag> .        # Construir desde un Dockerfile
docker rmi <imagen>                     # Borrar

# ---------- Contenedores ----------
docker run -d --name <n> -p 8080:80 <imagen>   # Crear y arrancar
docker run --rm -it <imagen> bash              # Interactivo y temporal
docker ps -a                            # Listar
docker logs -f <n>                      # Ver logs
docker exec -it <n> bash                # Entrar en un contenedor
docker stop <n> && docker rm <n>        # Parar y borrar

# ---------- Datos y redes ----------
docker volume ls                        # Volúmenes
docker network create <red>             # Crear red

# ---------- Compose ----------
docker compose up -d                    # Levantar
docker compose ps                       # Estado
docker compose logs -f                  # Logs
docker compose down                     # Parar y borrar (conserva volúmenes)

# ---------- Limpieza ----------
docker system df                        # Espacio ocupado
docker system prune                     # Limpiar lo que no se usa
```

---

## 17. Referencias

- Documentación oficial de Docker: <https://docs.docker.com/>
- Instalar Docker Engine en Ubuntu: <https://docs.docker.com/engine/install/ubuntu/>
- Pasos posteriores a la instalación en Linux: <https://docs.docker.com/engine/install/linux-postinstall/>
- Docker Desktop con WSL 2: <https://docs.docker.com/desktop/features/wsl/>
- Referencia del `Dockerfile`: <https://docs.docker.com/reference/dockerfile/>
- Referencia de Docker Compose: <https://docs.docker.com/reference/compose-file/>
- Docker Hub: <https://hub.docker.com/>
- Imagen oficial de Apache Spark: <https://hub.docker.com/r/apache/spark>
- Imágenes de Jupyter (Docker Stacks): <https://jupyter-docker-stacks.readthedocs.io/>
- Licencia de Docker Desktop: <https://docs.docker.com/subscription/desktop-license/>
