# Bitácora Laboratorio Docker - Clase 2

## 1. Objetivo

El objetivo de este laboratorio fue trabajar con Docker como base para un entorno de laboratorio SOC, utilizando contenedores, redes, volúmenes y Docker Compose.

También se configuró un contenedor Linux con medidas de seguridad para limitar sus privilegios y consumo de recursos.

---

## 2. Entorno utilizado

- Sistema operativo: Windows 11
- Docker Desktop
- Docker Engine: 29.7.2
- Docker Compose: 5.5.1
- Git: 2.55.0.windows.5
- Imagen utilizada: `ubuntu:24.04`
- Imagen de pruebas: `alpine`

---

## 3. Fase 1: Comandos básicos de Docker

En esta fase se utilizó un contenedor liviano basado en Alpine Linux para comprobar los principales comandos de administración y consulta de Docker.

### 3.1 Ejecución del contenedor

```bash
docker run -d --name fase1-alpine alpine sleep 3600
```

El comando creó y dejó ejecutándose el contenedor `fase1-alpine` utilizando la imagen `alpine`. El contenedor ejecutó el proceso `sleep 3600`.

### 3.2 Verificación del contenedor

```bash
docker ps
```

Permitió comprobar que el contenedor `fase1-alpine` se encontraba activo y ejecutándose.

Resultado obtenido mientras el contenedor estaba activo:

```text
CONTAINER ID   IMAGE     COMMAND        CREATED          STATUS          PORTS     NAMES
3f5c2454faef   alpine    "sleep 3600"   41 minutes ago   Up 41 minutes             fase1-alpine
```

### 3.3 Consulta de logs

```bash
docker logs fase1-alpine
```

No se obtuvo salida, debido a que el proceso `sleep 3600` no genera mensajes de registro.

### 3.4 Ejecución de un comando dentro del contenedor

```bash
docker exec fase1-alpine hostname
```

El comando permitió ejecutar `hostname` dentro del contenedor.

Resultado obtenido:

```text
3f5c2454faef
```

Esto permitió comprobar que era posible ejecutar comandos directamente dentro del contenedor.

### 3.5 Consulta de estadísticas

```bash
docker stats --no-stream fase1-alpine
```

Se realizó una medición mientras el contenedor se encontraba en ejecución.

Resultado obtenido:

```text
CONTAINER ID   NAME           CPU %     MEM USAGE / LIMIT     MEM %     NET I/O        BLOCK I/O     PIDS
3f5c2454faef   fase1-alpine   0.00%     2.477MiB / 7.594GiB  0.03%     1.7kB / 126B   2.26MB / 0B   1
```

El comando permite conocer el consumo de CPU, memoria, red, entrada/salida y cantidad de procesos del contenedor.

### 3.6 Inspección del contenedor

```bash
docker inspect fase1-alpine
```

El comando permitió consultar información detallada del contenedor, incluyendo su estado, imagen utilizada, configuración, red y parámetros de ejecución.

Durante la inspección se comprobó que:

- La imagen utilizada era `alpine`.
- El comando ejecutado era `sleep 3600`.
- El contenedor utilizaba la red `bridge`.
- La dirección IP asignada mientras estaba en ejecución era `172.17.0.2`.
- El contenedor no utilizaba volúmenes.
- El contenedor no estaba configurado como privilegiado.

El contenedor posteriormente terminó de forma normal después de completar los 3600 segundos definidos en el comando `sleep`.

---

## 4. Fase 2: Red y persistencia

En esta fase se creó una red Docker propia para permitir la comunicación entre contenedores mediante sus nombres y se creó un volumen para comprobar la persistencia de información después de eliminar un contenedor.

### 4.1 Creación de la red

```bash
docker network create --subnet 172.28.0.0/16 soc-net
```

Se creó la red `soc-net` utilizando la subred `172.28.0.0/16`.

### 4.2 Creación de los contenedores

```bash
docker run -d --name alpine-a --network soc-net alpine sleep 3600
```

```bash
docker run -d --name alpine-b --network soc-net alpine sleep 3600
```

Los dos contenedores fueron conectados a la red `soc-net`.

### 4.3 Prueba de comunicación mediante nombre

```bash
docker exec alpine-b ping -c 2 alpine-a
```

Resultado obtenido:

```text
PING alpine-a (172.28.0.2): 56 data bytes
64 bytes from 172.28.0.2: seq=0 ttl=64 time=0.111 ms
64 bytes from 172.28.0.2: seq=1 ttl=64 time=0.086 ms
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.086/0.098/0.111 ms
```

La prueba demostró que `alpine-b` pudo resolver el nombre `alpine-a` y comunicarse con él sin utilizar directamente su dirección IP.

Esto permitió comprobar la resolución de nombres entre contenedores dentro de la red `soc-net`.

### 4.4 Creación del volumen

```bash
docker volume create lab-datos
```

Se creó el volumen nombrado `lab-datos`.

### 4.5 Creación del contenedor para probar persistencia

```bash
docker run -d --name alpine-persist --network soc-net -v lab-datos:/datos alpine sleep 3600
```

El volumen `lab-datos` fue montado dentro del contenedor en la ruta `/datos`.

### 4.6 Escritura de un archivo

```bash
docker exec alpine-persist sh -c "echo 'persiste' > /datos/prueba.txt"
```

Se creó el archivo `prueba.txt` dentro del volumen.

### 4.7 Eliminación del contenedor

```bash
docker rm -f alpine-persist
```

El contenedor utilizado para la prueba fue eliminado.

### 4.8 Verificación de la persistencia

```bash
docker run --rm -v lab-datos:/datos alpine cat /datos/prueba.txt
```

Resultado obtenido:

```text
persiste
```

La prueba demostró que el archivo permaneció disponible después de eliminar el contenedor que lo había creado.

Por lo tanto, el dato quedó almacenado en el volumen `lab-datos` y no solamente en la capa de escritura temporal del contenedor.

### 4.9 Eliminación de los contenedores de prueba

```bash
docker rm -f alpine-a alpine-b
```

Los contenedores utilizados para la prueba de comunicación fueron eliminados después de finalizar la demostración.

---

## 5. Fase 3: Docker Compose con hardening

Para la víctima Linux se creó un archivo `docker-compose.yml` utilizando una imagen con etiqueta fija, límites de recursos, reducción de capacidades, protección contra aumento de privilegios y un volumen para conservar registros.

El archivo utilizado fue el siguiente:

```yaml
services:
  victima-linux:
    # Imagen con version fija para mantener un despliegue reproducible
    image: ubuntu:24.04

    # Nombre del contenedor
    container_name: victima-linux

    # Nombre del equipo dentro del contenedor
    hostname: victima-linux

    # Mantiene el contenedor ejecutandose para las pruebas
    command: ["sleep", "infinity"]

    # Conexion a la red creada para el laboratorio
    networks:
      - soc-net

    # Puerto publicado para el laboratorio
    ports:
      - "2222:22"

    # Elimina todas las capacidades para reducir privilegios
    cap_drop:
      - ALL

    # Se agregan solamente las capacidades necesarias
    cap_add:
      - CHOWN
      - SETUID
      - SETGID
      - AUDIT_CONTROL
      - AUDIT_WRITE

    # Impide que los procesos obtengan privilegios adicionales
    security_opt:
      - no-new-privileges:true

    # Limites de CPU y memoria
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 1g

    # Volumen para conservar los registros
    volumes:
      - victima-logs:/var/log

# Volumen nombrado
volumes:
  victima-logs:

# Red creada previamente
networks:
  soc-net:
    external: true
```

### 5.1 Imagen con etiqueta fija

Se utilizó:

```yaml
image: ubuntu:24.04
```

Se utilizó una etiqueta fija para trabajar con una versión conocida de la imagen y facilitar la reproducción del laboratorio.

### 5.2 Nombre y hostname

Se configuraron:

```yaml
container_name: victima-linux
hostname: victima-linux
```

Esto permite identificar de manera clara el contenedor y el nombre del equipo dentro de él.

### 5.3 Red

El servicio se conecta a:

```yaml
networks:
  - soc-net
```

La red `soc-net` fue creada previamente y es utilizada por el servicio para la configuración del laboratorio.

### 5.4 Publicación del puerto

Se configuró:

```yaml
ports:
  - "2222:22"
```

Esto publica el puerto 22 del contenedor mediante el puerto 2222 del equipo anfitrión para las pruebas definidas en el laboratorio.

### 5.5 Volumen de registros

Se configuró:

```yaml
volumes:
  - victima-logs:/var/log
```

El volumen permite conservar los registros ubicados en `/var/log` fuera del ciclo de vida del contenedor.

---

## 6. Fundamentación del hardening

Las medidas de seguridad se configuraron para reducir los privilegios del contenedor y limitar el impacto de un posible compromiso.

### 6.1 cap_drop: ALL

Configuración:

```yaml
cap_drop:
  - ALL
```

Esta opción elimina las capacidades del kernel que el contenedor podría recibir por defecto.

El riesgo que se busca mitigar es que un servicio comprometido tenga capacidades excesivas para realizar acciones sobre recursos del sistema, como manipular dispositivos, red o propiedad de archivos.

Al retirar todas las capacidades se reduce la superficie de privilegios disponible para el contenedor.

### 6.2 cap_add selectivo

Configuración:

```yaml
cap_add:
  - CHOWN
  - SETUID
  - SETGID
  - AUDIT_CONTROL
  - AUDIT_WRITE
```

Después de eliminar todas las capacidades, solamente se agregaron las capacidades indicadas en el descriptor.

Esto permite aplicar el principio de mínimo privilegio, evitando entregar al contenedor capacidades que no necesita.

### 6.3 no-new-privileges

Configuración:

```yaml
security_opt:
  - no-new-privileges:true
```

Esta opción impide que los procesos obtengan privilegios adicionales.

El riesgo que se busca mitigar es que un proceso que inicialmente tenga pocos privilegios pueda utilizar mecanismos de elevación de privilegios, como programas `setuid`, para obtener mayores privilegios dentro del contenedor.

### 6.4 Límites de CPU y memoria

Configuración:

```yaml
deploy:
  resources:
    limits:
      cpus: "1.0"
      memory: 1g
```

El límite de CPU establece un máximo de 1 CPU para el servicio y el límite de memoria establece un máximo de 1 GB.

El riesgo que se busca mitigar es que un contenedor consuma todos los recursos disponibles del equipo anfitrión, provocando lentitud, bloqueo del sistema o pérdida del trabajo del laboratorio.

### 6.5 Volumen de registros

Configuración:

```yaml
volumes:
  - victima-logs:/var/log
```

El volumen permite mantener los registros aunque el contenedor sea recreado.

Esto evita perder información almacenada en `/var/log` cuando se elimina la capa de escritura temporal del contenedor.

---

## 7. Fase 4: Verificación del despliegue

### 7.1 Validación del archivo Compose

```bash
docker compose config
```

El comando se ejecutó correctamente.

La validación permitió comprobar que el archivo `docker-compose.yml` tenía una configuración válida para Docker Compose.

### 7.2 Levantamiento del servicio

```bash
docker compose up -d
```

Resultado obtenido:

```text
[+] up 5/5
 ✔ Image ubuntu:24.04 Pulled
 ✔ Volume victima-linux_victima-logs Created
 ✔ Container victima-linux Started
```

El servicio `victima-linux` se levantó correctamente.

### 7.3 Verificación del estado

```bash
docker compose ps
```

Resultado obtenido:

```text
NAME            IMAGE          COMMAND            SERVICE         CREATED              STATUS              PORTS
victima-linux   ubuntu:24.04   "sleep infinity"   victima-linux   About a minute ago   Up About a minute   0.0.0.0:2222->22/tcp, [::]:2222->22/tcp
```

La prueba permitió comprobar que el contenedor se encontraba ejecutándose correctamente.

También se verificó la publicación del puerto:

```text
0.0.0.0:2222->22/tcp
[::]:2222->22/tcp
```

### 7.4 Verificación de consumo de recursos

```bash
docker stats --no-stream victima-linux
```

Resultado obtenido:

```text
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT   MEM %     NET I/O        BLOCK I/O   PIDS
cb1762c08d30   victima-linux   0.00%     1.168MiB / 1GiB    0.11%     1.7kB / 126B   0B / 0B     1
```

La salida permite comprobar que el límite de memoria configurado para el servicio es de `1GiB`.

También se observa que el consumo de CPU en el momento de la medición era de `0.00%`.

### 7.5 Nueva validación del descriptor

```bash
docker compose config
```

El comando se ejecutó nuevamente de forma correcta después del despliegue.

Esto permitió verificar que el descriptor utilizado continuaba siendo válido.

---

## 8. Fase 5: Control de versiones y publicación

Se utilizó Git para mantener versionado el laboratorio.

### 8.1 Inicialización del repositorio

```bash
git init
```

Se inicializó el repositorio Git dentro del proyecto.

### 8.2 Archivo .gitignore

Se creó el archivo:

```text
.gitignore
```

El archivo forma parte del repositorio y permite controlar archivos que no deben ser incluidos en el control de versiones.

### 8.3 Incorporación de archivos

```bash
git add docker-compose.yml .gitignore
```

Los archivos principales del proyecto fueron agregados al control de versiones.

### 8.4 Primer commit

```bash
git commit -m "Configura laboratorio Docker y endurecimiento"
```

Se realizó un commit descriptivo para registrar la configuración inicial del laboratorio y sus medidas de seguridad.

### 8.5 Documentación de la bitácora

Se creó el archivo:

```text
README.md
```

El README contiene la documentación del laboratorio, los comandos utilizados, las verificaciones realizadas, la configuración de Docker Compose y la fundamentación de las medidas de seguridad.

### 8.6 Segundo commit

```bash
git commit -m "Agrega bitácora de laboratorio Docker"
```

Se realizó un segundo commit descriptivo para registrar la documentación del laboratorio.

### 8.7 Publicación

El repositorio fue publicado en GitHub mediante la cuenta utilizada para este laboratorio.

Repositorio:

```text
Diego-byte907/victima-linux
```

La rama utilizada es:

```text
master
```

Los cambios fueron enviados mediante:

```bash
git push
```

La publicación permitió disponer del `docker-compose.yml`, `.gitignore` y `README.md` en el repositorio.

---

## 9. Archivos del proyecto

El proyecto contiene los siguientes archivos principales:

```text
victima-linux/
├── .gitignore
├── README.md
└── docker-compose.yml
```

### Descripción

- `.gitignore`: archivo utilizado para controlar archivos excluidos del repositorio.
- `README.md`: bitácora con el desarrollo, comandos, verificaciones y fundamentación.
- `docker-compose.yml`: descriptor de infraestructura utilizado para desplegar la víctima Linux con hardening.

---

## 10. Verificación respecto de los criterios de evaluación

### 10.1 Compose - 40 puntos

El servicio `victima-linux` fue levantado correctamente mediante Docker Compose utilizando una imagen con etiqueta fija, límites de CPU y memoria, reducción de capacidades, `no-new-privileges` y volumen declarado.

El descriptor está comentado y versionado en Git.

La configuración permite reproducir el servicio mediante Docker Compose utilizando el descriptor entregado y la red `soc-net` definida para el laboratorio.

### 10.2 Red y volumen - 20 puntos

Se creó y utilizó la red `soc-net`.

La comunicación entre `alpine-b` y `alpine-a` fue demostrada mediante `ping` utilizando el nombre del contenedor.

También se creó el volumen `lab-datos` y se demostró que el archivo `prueba.txt` permaneció disponible después de eliminar el contenedor que lo había creado.

### 10.3 Fundamentación - 20 puntos

Las opciones de seguridad utilizadas fueron explicadas indicando el riesgo concreto que busca mitigar cada una:

- `cap_drop: ALL`: reduce privilegios innecesarios.
- `cap_add`: permite solamente capacidades específicas.
- `no-new-privileges`: evita aumentos de privilegios.
- Límites de CPU y memoria: evitan el consumo excesivo de recursos del equipo.
- Volumen: permite conservar los registros fuera del ciclo de vida del contenedor.

### 10.4 Trazabilidad - 20 puntos

El proyecto fue versionado utilizando Git.

Se utilizaron commits descriptivos:

```text
Configura laboratorio Docker y endurecimiento
Agrega bitácora de laboratorio Docker
```

El repositorio fue publicado y contiene el descriptor Compose, el README y el `.gitignore`.

---

## 11. Conclusión

Durante el laboratorio se trabajó con los principales elementos de Docker necesarios para construir una base para un entorno SOC.

Se comprobó el ciclo básico de administración y diagnóstico de contenedores mediante `run`, `ps`, `logs`, `exec`, `stats` e `inspect`.

También se comprobó la comunicación entre contenedores mediante una red Docker propia y la persistencia de información mediante un volumen.

Finalmente, se creó un descriptor Docker Compose para una víctima Linux incorporando medidas de hardening, límites de recursos y almacenamiento persistente.

El proyecto fue validado mediante Docker Compose, versionado mediante Git y publicado en GitHub.

El resultado permite disponer de una configuración documentada para continuar con las actividades del laboratorio SOC.

---

## 12. Referencias

- Docker Docs — conceptos y primeros pasos.
- Docker Compose — especificación del archivo.
- Docker — seguridad del motor y del contenedor.
- CIS Docker Benchmark.
- Docker Desktop — backend WSL 2.
- Microsoft — configuración avanzada de WSL.
- MITRE ATT&CK — matriz Containers.