# Neurobi
# Practica servidor web
## 1. Titulo
Implementación y configuración estándar de un servidor web nginx en contenedores docker para el panel de monitoreo fisiológico (Proyecto PIENSA)

## 2. Tiempo de duración
 300 minutos.

## 3. Fundamentos:
El diseño de infraestructuras para aplicaciones de salud mental distribuidas que interactúan con el Internet de las Cosas Médicas (IoMT) exige el cumplimiento de estrictos criterios de estabilidad, concurrencia y tolerancia a fallos. En el ámbito de seguimiento clínico de pacientes con diagnósticos específicos, se capturan flujos constantes de telemetría provenientes de sensores biológicos dedicados a medir variables críticas como el ritmo cardíaco (LPM), las pulsaciones periféricas y la fuerza muscular de agarre. La persistencia y visualización inmediata de estas métricas analíticas se delegan a un frontend moderno (desarrollado con React, TypeScript y empaquetado mediante Vite). Para servir este contenido estático de manera óptima al personal médico y evitar pérdidas de sincronía o latencias perjudiciales, se requiere el uso de un servidor web industrial altamente eficiente como Nginx.

Nginx destaca por sobre los servidores tradicionales (basados en un esquema clásico de asignación de un hilo por cada conexión entrante) gracias a su arquitectura modular, asíncrona y puramente orientada a eventos. Un proceso maestro (master process) controla de forma coordinada múltiples procesos de trabajo (worker processes), los cuales son capaces de gestionar decenas de miles de conexiones concurrentes de forma simultánea mediante mecanismos de multiplexación de entrada/salida. Esta cualidad resulta indispensable para el Proyecto PIENSA, ya que mitiga el riesgo de caídas de la interfaz web cuando múltiples terminales de monitoreo transmiten señales en tiempo real o cuando varios especialistas consultan los reportes clínicos concurrentemente.

Por otro lado, la adopción del paradigma de contenerización mediante Docker permite aislar por completo este servidor web junto con los artefactos de compilación del frontend, garantizando el principio de inmutabilidad de la infraestructura. Al encapsular el servidor y el código en un contenedor ligero que comparte de manera segura el núcleo del sistema operativo anfitrión, se elimina la clásica problemática de configuración ("funciona en mi máquina, pero no en el servidor"). En un entorno real de microservicios, Nginx no solo actúa sirviendo los archivos HTML, CSS y JS compilados, sino que asume el rol de Proxy Inverso perimetral, actuando como un intermediario seguro que unifica el tráfico externo, cifra las conexiones mediante certificados y redirige de forma controlada las peticiones hacia la API REST (Axios) o los canales bidireccionales en tiempo real (Socket.io) pertenecientes a las capas de negocio internas.

## 4. Conocimientos previos.
Para realizar esta practica el estudiante necesita tener claro los siguientes temas:
- Comandos básicos de sistemas operativos Linux (navegación en la terminal mediante cd, manipulación de archivos y carpetas con ls, mkdir, touch y uso de editores empotrados como nano).
- Manejo y herramientas del desarrollador en el navegador web (inspección de tráfico de red, códigos de estado HTTP y análisis de logs en la consola de depuración JavaScript).
- Fundamentos de redes y protocolos (mapeo de puertos TCP/IP, direccionamiento local en loopback, y ciclo de vida de peticiones web).
- Conceptos clave de empaquetado Frontend (procesos de construcción estática en entornos Node.js utilizando Vite y estructuración modular en React).

## 5. Objetivos a alcanzar
- Implementar contenedores aislados con Nginx utilizando las imágenes oficiales optimizadas basadas en Alpine Linux de Docker Hub.
- Manipular archivos de configuración personalizados (nginx.conf) para redefinir el comportamiento del servidor web interno y habilitar el soporte correcto para aplicaciones de una sola página (SPA).
- Desplegar el entorno frontend de neuromonitoreo (frontedneuro) asegurando la correcta accesibilidad de los scripts de visualización gráfica (Recharts) y la preparación para las conexiones socket.
- Configurar el mapeo de puertos de comunicación de red entre la máquina anfitriona y el contenedor para exponer de forma pública la interfaz web del sistema.

## 6. Equipo necesario:
- Computador con sistema operativo Windows 10/11, GNU/Linux o macOS.
- Cuenta en la plataforma interactiva en la nube Play with Docker (PWD) o, en su defecto, el motor de Docker Desktop instalado y configurado de forma local.
- Docker Engine v20.10.0 o superior completamente funcional.
- Navegador web moderno actualizado (Google Chrome, Mozilla Firefox o Microsoft Edge).
- Código fuente de la interfaz del proyecto provisto, incluyendo sus archivos de metadatos (package.json, vite.config.ts).

## 7. Material de apoyo.
- Documentacion de docker y referencia de comandos para despliegues de red (https://docs.docker.com).
- Guia de asignatura de Arquitectura de Software y documentación oficial de configuración de Nginx (https://nginx.org).
- Cheat sheet de comandos Linux estructurados para administración básica de terminal.
- Código fuente base del frontend que integra librerías como Recharts para gráficas biométricas, Axios para peticiones HTTP y Socket.io-client para comunicación de sensores en tiempo real.

## 8. Procedimiento

Paso 1: Inicializar la interfaz de comandos o terminal en la máquina anfitriona. Navegar hasta la carpeta raíz donde se encuentran los archivos fuentes del proyecto frontend cargado ("frontedneuro"). Asegurar la presencia de los archivos clave ejecutando el listado en la consola:
ls -la

Paso 2: Compilar el código de la aplicación basada en React y Vite para generar los recursos estáticos puros optimizados para producción. Ejecutar de forma consecutiva los comandos:
npm install
npm run build
Este proceso leerá las configuraciones de 'vite.config.ts' y creará una carpeta en la raíz denominada 'dist/', la cual contiene los bundles optimizados de JavaScript, hojas de estilo CSS procesadas con Tailwind CSS y el archivo base index.html.

Paso 3: Crear el archivo de configuración personalizado para el servidor Nginx con la finalidad de gestionar las rutas internas de la aplicación. Crear un archivo llamado 'nginx.conf' en la raíz con el comando 'nano nginx.conf' e incluir la siguiente especificación técnica:
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
Guardar y cerrar el editor presionando Ctrl+O, Enter y luego Ctrl+X.

Paso 4: Crear un archivo de automatización de infraestructura denominado 'Dockerfile' ('nano Dockerfile') para empaquetar el servidor de manera definitiva. Añadir las siguientes instrucciones:
FROM nginx:alpine
COPY dist/ /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Paso 5: Construir la imagen Docker personalizada para el frontend de neuromonitoreo ejecutando la directiva de compilación:
docker build -t frontend-neuro:v1 .

Paso 6: Desplegar y poner en ejecución el contenedor en segundo plano, realizando un puenteo o mapeo desde el puerto físico 8080 del equipo host hacia el puerto interno 80 de escucha del servidor web contenerizado:
docker run -d --name servidor-neuro -p 8080:80 frontend-neuro:v1

Paso 7: Validar que el proceso se haya inicializado correctamente en el motor de ejecución docker mediante el comando de inspección:
docker ps

Paso 8: Abrir el cliente de navegación web de su preferencia y acceder a la dirección de red local asignada: http://localhost:8080. Presionar la tecla F12 para abrir las herramientas de desarrollo y corroborar el flujo de datos.


Diagrama de Flujo y Red de la Práctica (Mapeo de Infraestructura):

        [ CLIENTE EXTERNO: Terapeuta O FAMILIAR ]
                           │
                           ▼ (Petición HTTP a http://localhost:8080)
┌──────────────────────────────────────────────────────────────┐
│                    [ MÁQUINA ANFITRIONA ]                    │
│                                                              │
│       Puerto de Entrada: 8080 ─────────► Puerto Interno: 80  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │             [ CONTENEDOR DOCKER - NGINX ]              │  │
│  │                                                        │  │
│  │   Archivos de Configuración Internos:                  │  │
│  │   - Rutas y SPA (/etc/nginx/conf.d/default.conf)       │  │
│  │                                                        │  │
│  │   Directorio Raíz Web (Código Servido de React):       │  │
│  │   - Archivos de Producción (/usr/share/nginx/html)     │  │
│  │     ├─ index.html (Entrada y Montaje del DOM)          │  │
│  │     ├─ assets/ (Estructuras de Estilos e Interfaces)   │  │
│  │     └─ Lectura de Canales (Preparado para Sensores)    │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
Figura 1-1. Diagrama de contenedores. El servidor web Nginx procesa localmente las peticiones aisladas en el puerto 8080.

## 9. Resultados esperados:
Al finalizar de manera exitosa la práctica de laboratorio guiada, se obtendra una infraestructura funcional contenerizada
![alt text](<Captura de pantalla 2026-07-08 231653.png>)
![alt text](<Captura de pantalla 2026-07-08 231721.png>)
![alt text](<Captura de pantalla 2026-07-08 231733.png>)
![alt text](<Captura de pantalla 2026-07-08 231929.png>)
![alt text](<Captura de pantalla 2026-07-08 231948.png>)
![alt text](<Captura de pantalla 2026-07-08 231956.png>)

link: http://localhost:5173/
link: http://localhost:8761/eureka/apps
## 10. Bibliografía
- Docker Inc. (2026). Docker Containers and Virtualized Isolation Deployment Guide for Academic Environments. Docker Documentation. Recuperado de https://docs.docker.com
- Nginx International. (2025). Configuring High-Performance Web Servers for Single Page Applications (SPA) and IoT Systems. Nginx Core Docs. Recuperado de https://nginx.org/en/docs/
- Sommerville, I. (2011). Ingeniería de Software (9a ed.). Ciudad de México, México: Pearson Educación.
