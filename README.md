\# Bitácora Laboratorio Docker - Clase 2



\## 1. Objetivo



El objetivo de este laboratorio fue trabajar con Docker como base para un entorno de laboratorio SOC, utilizando contenedores, redes, volúmenes y Docker Compose.



También se configuró un contenedor Linux con medidas de seguridad para limitar sus privilegios y consumo de recursos.



\---



\## 2. Entorno utilizado



\- Sistema operativo: Windows 11

\- Docker Desktop

\- Docker Engine: 29.7.2

\- Docker Compose: 5.5.1

\- Git: 2.55.0.windows.5

\- Imagen utilizada: `ubuntu:24.04`

\- Imagen de pruebas: `alpine`



\---



\## 3. Red Docker



Se creó una red propia para permitir la comunicación entre contenedores mediante sus nombres.



\### Comando utilizado



```bash

docker network create --subnet 172.28.0.0/16 soc-net

