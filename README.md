# SwarmBalanceLab 🚀🐳⚖️

¡Bienvenido a **SwarmBalanceLab**!  
Aquí aprenderás (y sufrirás un poco) a balancear carga con Docker Swarm y HAProxy. Este proyecto es ideal para estudiantes, frikis de los contenedores y para aquellos que disfrutan de un toque de humor negro mientras lidian con problemas informáticos. 😈💻

---

## Tabla de Contenidos
- [Resumen del Taller](#resumen-del-taller)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Pasos Realizados](#pasos-realizados)
- [Problemas y Humor Negro](#problemas-y-humor-negro)
- [Requisitos Previos](#requisitos-previos)
- [Uso y Ejecución](#uso-y-ejecución)
- [Créditos](#créditos)

---

## Resumen del Taller 📝
En este taller, hicimos lo siguiente:
1. **Iniciamos Docker Swarm** para orquestar nuestros contenedores.
2. **Creamos una red overlay llamada `proxy`** para que nuestros servicios se comuniquen.
3. **Desplegamos un servicio web** (con 3 réplicas) usando la imagen `hashicorp/http-echo` para simular varios servidores.
4. **Configuramos HAProxy** para balancear la carga entre las réplicas usando un archivo de configuración.
5. **Nos topamos con Apache** que ocupaba el puerto 80, ¡como el fastidioso vecino que nunca se muda! 😡🚪
6. **Intentamos detener Apache** (con humor negro, "¡vete a freír espárragos, Apache!") y recibimos el error _"Access is denied"_ porque no teníamos permisos de administrador.
7. **Finalmente, redirigimos el tráfico o liberamos el puerto**, y pudimos ver el mensaje "Hola desde" en nuestras pruebas, confirmando que HAProxy hacía su magia. ✨

---

## Estructura del Proyecto 📁
La estructura de carpetas de nuestro proyecto es la siguiente:

```
SwarmBalanceLab/
├── docker-compose.yml      # Define el servicio web con 3 réplicas
├── haproxy.cfg             # Configuración de HAProxy para el balanceo de carga
└── README.md               # Este archivo, con humor negro y todo el proceso
```

---

## Pasos Realizados 🏃‍♂️

### 1. Inicialización del Swarm y Creación de la Red 🐳➡️🌐
- Ejecutamos:
  ```powershell
  docker swarm init
  docker network create -d overlay proxy
  ```
  _Nota: ¡Docker Swarm nos convierte en jefes de contenedores, o al menos eso nos gusta creer!_

### 2. Despliegue del Servicio Web 💻📢
- Creamos un archivo `docker-compose.yml` con este contenido:
  ```yaml
  version: "3.8"

  services:
    web:
      image: hashicorp/http-echo
      command: ["-text=Hola desde $HOSTNAME", "-listen=:80"]
      environment:
        - HOSTNAME
      deploy:
        replicas: 3
        restart_policy:
          condition: on-failure
      networks:
        - proxy

  networks:
    proxy:
      external: true
  ```
- Desplegamos el servicio:
  ```powershell
  docker stack deploy -c docker-compose.yml web_stack
  ```
  _Al principio, las réplicas se negaban a levantarse... ¡como yo en lunes por la mañana!_

### 3. Configuración y Despliegue de HAProxy ⚖️🔀
- Creamos el archivo `haproxy.cfg`:
  ```
  frontend http_front
      bind *:80
      default_backend web_servers

  backend web_servers
      balance roundrobin
      server web1 web:80 check
  ```
- En PowerShell, con el uso de backticks (`` ` ``) para continuar líneas, lanzamos el servicio:
  ```powershell
  docker service create --name haproxy `
    --publish 80:80 `
    --mount type=bind,source="${PWD}\haproxy.cfg",target=/usr/local/etc/haproxy/haproxy.cfg `
    --network proxy `
    haproxy:latest
  ```
  _¡HAProxy se puso en marcha, listo para repartir tráfico como un jefe!_

### 4. Problemas con Apache 🚫🔌
- Al probar:
  ```powershell
  while ($true) {
    curl http://localhost
    Start-Sleep -Seconds 1
  }
  ```
  Recibimos el mensaje: **"Forbidden"**.  
  _¿Por qué? Porque Apache, ese incordio, estaba usando el puerto 80 y bloqueaba el acceso. ¡El clásico invasor en nuestra fiesta digital!_

### 5. Intento (Fallido) de Detener Apache 😡🔐
- Se intentó ejecutar:
  ```cmd
  net stop Apache2.4
  ```
  pero obtuvimos:
  ```
  System error 5 has occurred.
  Access is denied.
  ```
  _El mensaje decía "Access is denied" – en resumen, no teníamos permisos para mandar a la basura a Apache. ¡Otra batalla ganada por el sistema!_

### 6. Solución Final y Pruebas ✅🔄
- Finalmente, se decidió redirigir el tráfico o liberar el puerto. Tras algunos ajustes y usar el puerto alternativo (o conseguir que Apache se haga a un lado), las pruebas con `curl http://localhost` comenzaron a mostrar "Hola desde", confirmando que HAProxy ya tomaba el control.
  _¡Victoria al fin! Y así, nuestros contenedores finalmente se comunicaron sin la interferencia de Apache._

---

## Requisitos Previos 🛠️
- **Docker**: [Instalación](https://docs.docker.com/get-docker/)
- **Docker Compose**: [Instalación](https://docs.docker.com/compose/install/)
- Una terminal (preferiblemente PowerShell) **ejecutada como Administrador** para ciertos comandos.

---

## Uso y Ejecución 🚀
1. **Inicializa Docker Swarm y crea la red**:
   ```powershell
   docker swarm init
   docker network create -d overlay proxy
   ```
2. **Despliega el stack del servicio web**:
   ```powershell
   docker stack deploy -c docker-compose.yml web_stack
   ```
3. **Despliega HAProxy** (recuerda usar backticks en PowerShell):
   ```powershell
   docker service create --name haproxy `
     --publish 80:80 `
     --mount type=bind,source="${PWD}\haproxy.cfg",target=/usr/local/etc/haproxy/haproxy.cfg `
     --network proxy `
     haproxy:latest
   ```
4. **Verifica el funcionamiento** haciendo peticiones:
   ```powershell
   while ($true) {
     curl http://localhost
     Start-Sleep -Seconds 1
   }
   ```
   Si Apache interfiere, deténlo (ejecuta la terminal como administrador o usa el Administrador de Servicios).

---

## Créditos 🤓🎓
- **Profesor:** Heberth Martinez  
- **Herramientas:** Docker, Docker Swarm, HAProxy  
- **Humor:** Para todos aquellos que saben que a veces, en el mundo IT, el humor negro es la única medicina para los errores y frustraciones diarias. 😈💊

________________________________________________
caesar
________________________________________________