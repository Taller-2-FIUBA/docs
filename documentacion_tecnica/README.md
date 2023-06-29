# Documentación técnica

## Arquitectura

![arquitectura](./arquitecture.png)

Para el proyecto se empleo una arquitectura de microservicios, diagramada a continuación:

Se procede a listar los microservicios y detalles relevantes de cada uno,
incluyendo sus principales dependencias, el lenguaje y framework con el que
fueron diseñados:

### Users

[Repositorio](https://github.com/Taller-2-FIUBA/users).  
Responsable de todas las operaciones relacionadas con los usuarios y
administradores: creación, log-in, edición y visualización.  
También es el punto de acceso para el manejo de las billeteras de cada usuario, aunque no es responsable de efectivizar las mismas.  
Utiliza PostgreSQL para almacenamiento de los datos de los usuarios, MongoDB
para almacenamiento de sus respectivas posiciones y Redis queue de métricas.
Escrito en Python y utiliza FastAPI.

### Payment

[Repositorio](https://github.com/Taller-2-FIUBA/payment).  
Responsable de efectivizar las transacciones efectuadas entre billeteras de la
plataforma. Se utiliza la red de testeo Goerli. Escrito en Javascript.

### Goals

[Repositorio](https://github.com/Taller-2-FIUBA/goals).  
Se utiliza para las operaciones relacionadas con las metas del usuario:
creación, edición y visualización. También provee información sobre el progreso
en las métricas en general. Utiliza PostgreSQL como base de datos. Escrito en Python utilizando FASTApi.

### Authentication

[Repositorio](https://github.com/Taller-2-FIUBA/authentication).  
Encargado de regular la seguridad de la plataforma a través de
la creación y decodificación de los JWT (JSON Web Tokens) utilizados
para el acceso a los distintos recursos. También provee acceso a Firebase, para
el registro y log-in efectivo de cada usuario, y una carga
y descarga básica de imágenes a través de Firebase Storage. Escrito en Go y utiliza Gin.

### Trainings

[Repositorio](https://github.com/Taller-2-FIUBA/trainings).  
Maneja los distintos entrenamientos: su creación, edición y visualización.
También comprende las distintas interacciones
posibles de los usuarios con estos: marcarlos como favoritos, asignarles un
puntaje, etc. Utiliza PostgreSQL como base de datos. Escrito en Python y utiliza
FastAPI

### Metrics

[Repositorio](https://github.com/Taller-2-FIUBA/metrics).
Escrito en Python, tiene dos componentes.

#### HTTP API

Lee métricas de MongoDB utilizando pipelines de agregación para mostrar las
métricas de aplicación.  

#### Consumer

Aplicación CLI que consume las métricas escritas en Redis y las guarda
en MongoDB.  
El servicio utiliza operaciones bloqueantes para esperar por nuevas métricas.

## Despliegue

El despliegue se realiza en Kubernetes. Para ello se crean imágenes de docker.

### Kubernetes

En Kubernetes se crean los siguientes objetos:

- `Deployment`: Usado para desplegar los distintos micro-servicios que conforma la aplicación.
Cada uno de estos micro-servicios es desplegado en un pod.
- `ConfigMap`: Usado para crear la configuración de la aplicación.
- `Service`: Usado para agrupar distintos pods y así el ingress controller puede acceder a los puertos expuestos por ellos. Hay un `service` por cada HTTP API de la aplicación. 

Las definiciones de cada uno de ellos se encuentra dentro de cada repositorio bajo la carpeta `kubernetes`.

- `Ingress`: Usado para conectar un load balancer con los distintos pods agrupados por cada servicio.

Dentro de la definición del `ingress` se encuentra la configuración del load balancer.
Esta definición es centralizada y se encuentra dentro del repositorio
[k8s](https://github.com/Taller-2-FIUBA/k8s)

## Configuración

Todos los micro-servicios se configuran mediante variables de entorno.  
Cada variable de entorno está prefijada por el nombre de la aplicación.
e.g. `USER_MONGO_USER`.  

Las variables de entorno se especifican usando un `configmap` que es utilizado
en el `deployment` para crear variables de entorno en el pod con la `data`
configurada.  

Cada repositorio contiene lo mínimo y necesario para que funcione el
micro-servicio dentro de `kubernetes/configmap.yaml`.  
En estos archivos, se utilizan placeholders para los datos sensibles, e.g.:
`$USERS_MONGODB_PASSWORD`. Esos placeholders son reemplazados por los valores
reales en el runtime de `Okteto` utilizando el comando `env`.

## Cloud providers

### Okteto

Usamos `Okteto` como cloud provider. En `Okteto` desplegamos todos los pods
usando los repositorios de github directamente.  
En la raíz de cada repositorio hay un archivo llamado `okteto.yml` que le 
instruye al controlador de `Okteto` que donde búscar los archivos con las
definiciones de los objetos de Kubernetes.  
Los pods son creados usando los `development environments` que proporciona la
plataforma.

### CI/CD

Utilizamos pipelines de gitlab para hacer CI/CD.
Se encarga de correr los unit e integration tests, como también analizadores
estáticos de código. Luego, el pipeline construye la imagen de docker y la sube
a Dockerhub para que esté disponible en Okteto.

## Post mortem

Precondiciones:

- Linea de tiempo con todo lo intentado desde la perdida de servicio hasta la
restauración de servicio.
- Logs/Traces/Métricas de la aplicación durante este periodo. 

Análisis:

De la linea de tiempos y Logs/Traces/Métricas:

- ¿Qué servicio/s fueron afectados?
- ¿Cuál fue la causa de la perdida de servicio?
- ¿Los Logs/Traces/Métricas muestran degradación de servicio o ataques?
- ¿Cómo fue restaurado el servicio?
- ¿Qué modificaciones son necesarias para que no vuelva a ocurrir?

## Entorno local

En el repositorio [overlays](https://github.com/Taller-2-FIUBA/overlays) se
encuentra documentación sobre como crear un entorno local que incluye las
dependencias.
