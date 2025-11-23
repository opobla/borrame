# Guía del Estudiante - S12: Docker en CI/CD y Cloud Run

Esta guía te acompañará durante la sesión S12 para aprender sobre Docker en CI/CD (Continuous Integration/Continuous Deployment) y Cloud Run. Incluye conceptos teóricos esenciales, prácticas paso a paso y ejercicios para consolidar tu aprendizaje.

## Objetivos de Aprendizaje

Al finalizar esta sesión serás capaz de:

- Publicar imágenes Docker en registros públicos (Docker Hub) y privados (Artifact Registry)
- Configurar Cloud Build para automatizar la construcción y publicación de imágenes Docker
- Desplegar aplicaciones contenedorizadas en Cloud Run utilizando diferentes métodos
- Gestionar secretos de forma segura utilizando Secret Manager de GCP (Google Cloud Platform)
- Migrar aplicaciones desde Docker Compose local a Cloud Run en producción
- Implementar pipelines de CI/CD completos que integren construcción, publicación y despliegue automatizado
- Aplicar mejores prácticas de seguridad en la gestión de credenciales y secretos en entornos cloud

## Requisitos Previos

### Conocimientos Necesarios

- Conocimientos de Docker adquiridos en S11 (imágenes, contenedores, Dockerfiles, Docker Compose)
- Familiaridad con línea de comandos (terminal/consola)
- Conceptos básicos de CI/CD (Continuous Integration/Continuous Deployment)
- Conocimientos básicos de Python y Flask (para la aplicación de ejemplo)
- Familiaridad básica con servicios de Google Cloud Platform

### Software Requerido

Antes de comenzar, asegúrate de tener instalado:

- **Docker Desktop** (o Docker Engine + Docker Compose):
  - Debe estar instalado y funcionando desde S11
  - Versión mínima recomendada: Docker 24.0+
  
- **Google Cloud SDK (gcloud CLI)**:
  - Windows: [Instalador de Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
  - macOS: `brew install google-cloud-sdk` o [instalador](https://cloud.google.com/sdk/docs/install)
  - Linux: [Instalación según distribución](https://cloud.google.com/sdk/docs/install)
  - Versión mínima recomendada: 450.0.0+
  
- **Editor de código**:
  - VS Code o Cursor
  - Extensiones recomendadas: Docker, YAML
  
- **Git** para control de versiones

### Cuentas y Recursos GCP

- **Proyecto GCP activo** con facturación habilitada
  - Si no tienes proyecto, crear uno en [Google Cloud Console](https://console.cloud.google.com/)
  - Asegúrate de tener facturación habilitada (hay free tier generoso)
  
- **APIs habilitadas** en tu proyecto:
  - Artifact Registry API
  - Cloud Build API
  - Cloud Run API
  - Secret Manager API
  
- **Docker Hub** (cuenta gratuita opcional, para práctica de publicación manual)
  - Crear cuenta en [Docker Hub](https://hub.docker.com/)

### Verificación de Instalación

Verifica que todo está correctamente configurado:

```bash
# Verificar Docker
docker --version
docker compose version

# Verificar Google Cloud SDK
gcloud --version

# Autenticarse en GCP (si no lo has hecho)
gcloud auth login

# Configurar proyecto por defecto
gcloud config set project TU_PROJECT_ID

# Verificar que las APIs están habilitadas
gcloud services list --enabled
```

Si todos los comandos funcionan correctamente, estás listo para comenzar.

## Introducción Teórica

### Registros de Imágenes Docker

Los registros de imágenes Docker son repositorios centralizados donde se almacenan y distribuyen imágenes Docker. Permiten compartir imágenes entre diferentes entornos, equipos y servicios cloud, facilitando la distribución de aplicaciones contenedorizadas.

**Tipos de registros**:

Docker Hub constituye el registro público más utilizado a nivel mundial. Proporciona acceso gratuito a imágenes públicas y opciones de pago para imágenes privadas. Es ideal para proyectos open source y aprendizaje, aunque tiene limitaciones en planes gratuitos para imágenes privadas. El formato de imágenes en Docker Hub es `usuario/nombre-imagen:tag`.

Google Container Registry (GCR) fue el servicio original de GCP para almacenar imágenes Docker, utilizando buckets de Cloud Storage como backend. Aunque sigue funcionando, Google recomienda migrar a Artifact Registry, que es el servicio moderno y recomendado para nuevos proyectos.

Artifact Registry representa el servicio actual de GCP para almacenar artefactos de build, incluyendo imágenes Docker, paquetes Maven, npm y otros formatos. Proporciona mejor integración con otros servicios de GCP, control de acceso granular mediante IAM (Identity and Access Management), y soporte para múltiples formatos de artefactos en un solo servicio. El formato de imágenes en Artifact Registry es `REGION-docker.pkg.dev/PROJECT_ID/REPO_NAME/IMAGE_NAME:TAG`.

**Autenticación en registros**:

La autenticación en Docker Hub se realiza mediante `docker login`, utilizando credenciales de cuenta Docker Hub o tokens de acceso personal. Para automatización, los tokens de acceso personal proporcionan mayor seguridad que contraseñas.

En Artifact Registry, la autenticación utiliza credenciales de aplicación de GCP mediante `gcloud auth configure-docker`. Cloud Build y otros servicios de GCP se autentican automáticamente utilizando service accounts, eliminando la necesidad de credenciales manuales en pipelines automatizados.

**Tagging de imágenes**:

El tagging correcto de imágenes es esencial para gestión de versiones y despliegues. Las mejores prácticas incluyen usar tags semánticos como `v1.2.3` para releases, `latest` para la versión más reciente estable, y tags basados en commit SHA para trazabilidad. Los tags descriptivos facilitan la identificación de versiones específicas y permiten rollbacks controlados.

### Cloud Build - Automatización CI/CD

Cloud Build es un servicio completamente gestionado de GCP que ejecuta builds en la nube. Permite automatizar la construcción, testing y publicación de aplicaciones sin necesidad de mantener infraestructura propia de CI/CD.

**Conceptos de CI/CD**:

Continuous Integration (CI) se refiere a la práctica de integrar cambios de código frecuentemente en un repositorio compartido, donde cada integración se verifica mediante builds y tests automatizados. Esto permite detectar errores tempranamente y mantener el código en un estado siempre funcional.

Continuous Deployment (CD) extiende CI automatizando el despliegue de cambios que pasan todas las pruebas. En el contexto de Cloud Build, CD puede incluir la publicación de imágenes en registros y el despliegue automático en servicios como Cloud Run.

**Arquitectura de Cloud Build**:

Cloud Build ejecuta builds en contenedores efímeros que se destruyen al finalizar el build. Cada paso del build se ejecuta en un contenedor específico, permitiendo usar diferentes herramientas en cada paso. Los builds pueden ejecutarse manualmente mediante `gcloud builds submit` o automáticamente mediante triggers que responden a eventos de Git.

**Configuración de cloudbuild.yaml**:

El archivo `cloudbuild.yaml` define la configuración del build mediante una estructura YAML. La sección `steps` contiene los pasos a ejecutar, donde cada paso especifica la imagen del contenedor a usar y los argumentos del comando. Los pasos se ejecutan secuencialmente, y el build falla si cualquier paso falla.

Las `substitutions` permiten parametrizar builds mediante variables que pueden definirse en el archivo o pasarse externamente. Variables predefinidas como `PROJECT_ID` y `SHORT_SHA` están disponibles automáticamente, mientras que variables personalizadas se definen con el prefijo `_`.

La sección `images` especifica qué imágenes construir y publicar. Cloud Build automáticamente publica estas imágenes en el registro especificado después de construir exitosamente.

**Triggers de Cloud Build**:

Los triggers permiten ejecutar builds automáticamente ante eventos específicos, principalmente pushes a ramas o tags en repositorios Git. Los triggers pueden configurarse para ejecutarse en todas las ramas, ramas específicas, o basándose en patrones de nombres de archivos modificados.

### Cloud Run - Serverless Containers

Cloud Run es un servicio completamente gestionado de GCP que ejecuta contenedores de forma serverless. Permite ejecutar aplicaciones contenedorizadas sin gestionar servidores, con escalado automático y facturación por uso.

**Conceptos de serverless**:

Serverless se refiere a un modelo de ejecución donde el proveedor cloud gestiona completamente la infraestructura, escalado y disponibilidad. Los desarrolladores solo proporcionan el código o contenedor, y el servicio se encarga de ejecutarlo cuando hay peticiones. Cloud Run implementa este modelo para contenedores, permitiendo ejecutar cualquier aplicación contenedorizada sin configuración de servidores.

**Arquitectura de Cloud Run**:

Cloud Run ejecuta contenedores en instancias efímeras que se crean bajo demanda y se destruyen después de un período de inactividad. Cada instancia puede manejar múltiples peticiones concurrentes según la configuración de concurrency. Cloud Run gestiona automáticamente el escalado horizontal, creando nuevas instancias cuando aumenta la carga y eliminando instancias cuando disminuye.

Las aplicaciones en Cloud Run deben seguir el contrato de Cloud Run: escuchar en el puerto especificado por la variable de entorno `PORT`, responder a peticiones HTTP, y implementar un endpoint de health check. Cloud Run establece automáticamente la variable `PORT`, y la aplicación debe usar este valor en lugar de hardcodear un puerto.

**Despliegue en Cloud Run**:

El despliegue puede realizarse de múltiples formas. El método más directo es `gcloud run deploy`, que construye la imagen desde código fuente o despliega una imagen existente. Cloud Run también soporta despliegue desde Artifact Registry, permitiendo separar la construcción de la publicación y el despliegue.

Durante el despliegue, Cloud Run permite configurar variables de entorno, secretos, recursos (CPU y memoria), concurrency, timeouts y políticas de escalado. Estas configuraciones pueden especificarse mediante flags de `gcloud` o archivos de configuración YAML.

**Escalado automático**:

Cloud Run escala automáticamente desde cero instancias hasta el límite configurado según la demanda. El escalado a cero significa que no hay costos cuando no hay tráfico, pero la primera petición después de un período inactivo puede experimentar "cold start", un delay mientras Cloud Run inicia una nueva instancia.

### Secret Manager de GCP

Secret Manager es un servicio de GCP que proporciona almacenamiento seguro, gestión centralizada y control de acceso para secretos como contraseñas, API keys y certificados. Permite almacenar secretos de forma encriptada y acceder a ellos desde aplicaciones y servicios de GCP.

**Conceptos fundamentales**:

Los secretos en Secret Manager son objetos que contienen datos sensibles como contraseñas, tokens o claves de API. Cada secreto tiene un nombre único dentro de un proyecto y puede tener múltiples versiones. Las versiones permiten rotar secretos sin cambiar el código de la aplicación, simplemente actualizando qué versión se utiliza.

Secret Manager encripta automáticamente los secretos en reposito y proporciona control de acceso mediante IAM. Los permisos IAM determinan qué usuarios o servicios pueden acceder, leer o modificar secretos, proporcionando seguridad granular.

**Integración con Cloud Run**:

Cloud Run permite montar secretos como variables de entorno o archivos dentro del contenedor. Cuando se monta como variable de entorno, el valor del secreto está disponible como una variable de entorno estándar. Cuando se monta como archivo, Secret Manager crea un archivo en el sistema de archivos del contenedor con el contenido del secreto.

## Prácticas Paso a Paso

### Práctica 1: Publicación de Imágenes Docker

**Duración estimada**: 60 minutos  
**Objetivo**: Aprender a publicar imágenes Docker en registros públicos (Docker Hub) y privados (Artifact Registry), comprendiendo las diferencias y casos de uso de cada uno.

#### Paso Previo: Construcción y Prueba Local

**Objetivo**: Construir la imagen localmente y verificar que funciona correctamente antes de publicarla.

1. **Navegar al directorio de la aplicación**:
   ```bash
   cd materiales/practica-1/ejemplo-app-info-service
   ```

2. **Construir la imagen Docker**:
   ```bash
   docker build -t app-info-service:latest .
   ```

3. **Verificar que la imagen se construyó correctamente**:
   ```bash
   docker image ls | grep app-info-service
   ```

4. **Ejecutar el contenedor localmente para probarlo**:
   ```bash
   docker run -d \
     -p 8080:8080 \
     -e APP_NAME="Mi App" \
     -e ENVIRONMENT="development" \
     -e SECRET_MESSAGE="Mi secreto de prueba" \
     --name app-info \
     app-info-service:latest
   ```

5. **Probar los endpoints de la aplicación**:
   ```bash
   # Información del servicio
   curl http://localhost:8080/
   
   # Health check
   curl http://localhost:8080/health
   
   # Información del contenedor
   curl http://localhost:8080/info
   
   # Configuración (sin secretos)
   curl http://localhost:8080/config
   
   # Mensaje secreto
   curl http://localhost:8080/secret
   
   # Estadísticas
   curl http://localhost:8080/stats
   ```

6. **Verificar logs del contenedor**:
   ```bash
   docker container logs app-info
   ```

7. **Detener y eliminar el contenedor de prueba**:
   ```bash
   docker container stop app-info
   docker container rm app-info
   ```

**Ejercicio de verificación**:
- ¿Todos los endpoints responden correctamente?
- ¿El health check devuelve `{"status": "healthy"}`?
- ¿El endpoint `/secret` muestra el mensaje secreto configurado?

Una vez verificado que la aplicación funciona correctamente localmente, procederemos a publicarla en los registros.

#### Subtarea 1.1: Publicación Manual en Docker Hub

**Objetivo**: Publicar una imagen Docker en Docker Hub manualmente.

**Nota**: Si ya completaste el paso previo, la imagen `app-info-service:latest` ya está construida. Si no, asegúrate de estar en el directorio correcto y construir la imagen primero.

1. **Asegurarse de estar en el directorio correcto** (si no lo estás ya):
   ```bash
   cd materiales/practica-1/ejemplo-app-info-service
   ```

2. **Autenticarse en Docker Hub**:
   ```bash
   docker login
   # Introducir tu usuario y contraseña de Docker Hub
   ```

3. **Tagging de la imagen con formato Docker Hub**:
   El formato para Docker Hub es `usuario/nombre-imagen:tag`. Reemplaza `TU_USUARIO` con tu usuario de Docker Hub:
   ```bash
   docker tag app-info-service:latest TU_USUARIO/app-info-service:latest
   docker tag app-info-service:latest TU_USUARIO/app-info-service:v1.0.0
   ```

4. **Verificar los tags creados**:
   ```bash
   docker image ls | grep app-info-service
   ```
   Deberías ver la imagen original y las dos nuevas con formato Docker Hub.

5. **Publicar la imagen**:
   ```bash
   docker push TU_USUARIO/app-info-service:latest
   docker push TU_USUARIO/app-info-service:v1.0.0
   ```

6. **Verificar publicación**:
   - Acceder a https://hub.docker.com
   - Buscar tu repositorio `TU_USUARIO/app-info-service`
   - Verificar que las imágenes aparecen con los tags correctos

**Ejercicio de verificación**:
- ¿Qué diferencia hay entre publicar con tag `latest` y `v1.0.0`?
- ¿Cómo harías para que otra persona pueda descargar tu imagen desde Docker Hub?

#### Subtarea 1.2: Publicación Manual en Artifact Registry

**Objetivo**: Publicar una imagen Docker en Artifact Registry de GCP.

**Nota**: Asegúrate de tener la imagen `app-info-service:latest` construida localmente (del paso previo o Subtarea 1.1).

1. **Habilitar API de Artifact Registry** (si no está habilitada):
   ```bash
   gcloud services enable artifactregistry.googleapis.com
   ```

2. **Obtener tu PROJECT_ID**:
   ```bash
   export PROJECT_ID=$(gcloud config get-value project)
   echo "Tu PROJECT_ID es: ${PROJECT_ID}"
   ```

3. **Crear repositorio en Artifact Registry**:
   ```bash
   gcloud artifacts repositories create docker-repo \
     --repository-format=docker \
     --location=europe-west1 \
     --description="Repositorio Docker para prácticas S12"
   ```

4. **Configurar autenticación Docker para Artifact Registry**:
   ```bash
   gcloud auth configure-docker europe-west1-docker.pkg.dev
   ```

5. **Tagging de la imagen con formato Artifact Registry**:
   El formato es `REGION-docker.pkg.dev/PROJECT_ID/REPO_NAME/IMAGE_NAME:TAG`:
   ```bash
   docker tag app-info-service:latest \
     europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo/app-info-service:latest
   docker tag app-info-service:latest \
     europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo/app-info-service:v1.0.0
   ```

6. **Publicar la imagen**:
   ```bash
   docker push europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo/app-info-service:latest
   docker push europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo/app-info-service:v1.0.0
   ```

7. **Verificar publicación**:
   ```bash
   gcloud artifacts docker images list \
     europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo
   ```

**Ejercicio de verificación**:
- Compara el proceso de publicación en Docker Hub vs Artifact Registry. ¿Cuáles son las principales diferencias?
- ¿Por qué es importante el formato del tag en Artifact Registry?

#### Subtarea 1.3: Comparativa y Ejercicios

**Ejercicios adicionales**:

1. **Listar imágenes en ambos registros**:
   - Docker Hub: Acceder a la web y ver tus imágenes
   - Artifact Registry: Usar `gcloud artifacts docker images list`

2. **Descargar imagen desde Artifact Registry**:
   ```bash
   docker pull europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo/app-info-service:latest
   ```

3. **Reflexión**:
   - ¿En qué casos usarías Docker Hub vs Artifact Registry?
   - ¿Qué ventajas tiene Artifact Registry para servicios en GCP?

### Práctica 2: Automatización con Cloud Build

**Duración estimada**: 60 minutos  
**Objetivo**: Configurar Cloud Build para automatizar la construcción y publicación de imágenes Docker mediante triggers que responden a eventos de Git.

#### Subtarea 2.1: Configuración Básica de Cloud Build

**Objetivo**: Configurar Cloud Build para construir y publicar imágenes automáticamente.

1. **Habilitar API de Cloud Build**:
   ```bash
   gcloud services enable cloudbuild.googleapis.com
   ```

2. **Verificar permisos**:
   El service account de Cloud Build necesita permisos en Artifact Registry. Verificar que tienes los permisos necesarios:
   ```bash
   gcloud projects get-iam-policy ${PROJECT_ID} \
     --flatten="bindings[].members" \
     --filter="bindings.members:serviceAccount:*cloudbuild*"
   ```

3. **Crear archivo cloudbuild.yaml**:
   Crear archivo `cloudbuild.yaml` en la raíz del proyecto (no dentro de `materiales/`) con el siguiente contenido:
   ```yaml
   steps:
     - name: 'gcr.io/cloud-builders/docker'
       args:
         - 'build'
         - '-t'
         - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:${BUILD_ID}'
         - '-t'
         - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:latest'
         - '.'
       dir: 'materiales/practica-1/ejemplo-app-info-service'
   
     - name: 'gcr.io/cloud-builders/docker'
       args:
         - 'push'
         - '--all-tags'
         - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service'
   
   images:
     - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:${BUILD_ID}'
     - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:latest'
   
   substitutions:
     _REGION: 'europe-west1'
     _REPO_NAME: 'docker-repo'
     # Nota: BUILD_ID se usa directamente en los tags (no en substitutions) porque siempre está disponible.
     # Para triggers de Git, puedes cambiar BUILD_ID por SHORT_SHA en los tags si prefieres usar el commit SHA.
   
   options:
     machineType: 'E2_HIGHCPU_8'
     logging: CLOUD_LOGGING_ONLY
   
   timeout: '1200s'
   ```


`${BUILD_ID}` es una variable automática que proporciona Cloud Build y representa un identificador único para cada ejecución (build). Su propósito principal es asegurar que cada artefacto generado (por ejemplo, una imagen Docker) tenga un tag único y rastreable, aunque el contenido del código no haya cambiado. Esto permite:
- Diferenciar imágenes construidas en ejecuciones distintas, incluso si provienen del mismo commit.
- Facilitar la depuración y trazabilidad: puedes saber exactamente qué build originó una imagen específica.
- Evitar colisiones o sobreescritura accidental de imágenes cuando hay builds concurrentes.

En resumen, `${BUILD_ID}` asegura unicidad y trazabilidad en los artefactos que genera Cloud Build.

1. **Ejecutar build manualmente**:
   ```bash
   gcloud builds submit --config cloudbuild.yaml
   ```
   
   **Nota**: Si encuentras un error sobre `SHORT_SHA` vacío o tags inválidos, asegúrate de que el `cloudbuild.yaml` use `${BUILD_ID}` directamente en los tags (no dentro de una substitution) para ejecución manual. `BUILD_ID` siempre está disponible y se expande automáticamente cuando se usa directamente en los tags.

2. **Monitorear el build**:
   El build mostrará progreso en tiempo real. Al finalizar, verificar que la imagen se publicó correctamente:
   ```bash
   gcloud builds list --limit=5
   gcloud artifacts docker images list \
     europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo
   ```

**Ejercicio de verificación**:
- ¿Qué hace cada paso en el `cloudbuild.yaml`?
- ¿Qué son las `substitutions` y para qué sirven?
- ¿Qué variables automáticas usa Cloud Build (`PROJECT_ID`, `BUILD_ID`)?
- ¿Por qué usamos `BUILD_ID` en lugar de `SHORT_SHA` para ejecución manual?

#### Subtarea 2.2: Configuración de Triggers

**Objetivo**: Configurar trigger para ejecutar builds automáticamente en cada push.

**Nota**: Esta subtarea requiere tener un repositorio Git conectado a Cloud Build. Si no tienes repositorio, puedes usar Cloud Source Repositories o conectar un repositorio externo (GitHub, GitLab).

1. **Conectar repositorio Git** (si no está conectado):
   
   **Opción A: Cloud Source Repositories**:
   ```bash
   gcloud source repos create mi-repo
   git remote add google https://source.developers.google.com/p/${PROJECT_ID}/r/mi-repo
   git push google main
   ```
   
   **Opción B: Conectar repositorio externo** (GitHub, GitLab):
   - Ir a Cloud Build > Repositories en la consola de GCP
   - Click en "Connect Repository"
   - Seleccionar proveedor (GitHub, GitLab, Bitbucket)
   - Autorizar y seleccionar repositorio

2. **Crear trigger mediante consola**:
   - Ir a Cloud Build > Triggers en la consola de GCP
   - Click en "Create Trigger"
   - Configurar:
     - **Name**: `build-and-push-on-push`
     - **Event**: Push to a branch
     - **Branch**: `^main$` (o tu rama principal)
     - **Configuration**: Cloud Build configuration file
     - **Location**: `cloudbuild.yaml`
   - Click en "Create"

3. **Probar el trigger**:
   - Hacer un pequeño cambio en el código (por ejemplo, un comentario)
   - Hacer commit y push:
     ```bash
     git add .
     git commit -m "Test Cloud Build trigger"
     git push
     ```
   - Ir a Cloud Build > History y verificar que se inició un build automáticamente

4. **Verificar resultado**:
   ```bash
   gcloud builds list --limit=5
   ```

**Ejercicio de verificación**:
- ¿Qué ventajas tiene usar triggers vs ejecutar builds manualmente?
- ¿Cómo podrías configurar diferentes builds para diferentes ramas?

### Práctica 3: Introducción a Cloud Run

**Duración estimada**: 60 minutos  
**Objetivo**: Desplegar la aplicación en Cloud Run, configurar variables de entorno, y entender recursos y escalado.

#### Subtarea 3.1: Despliegue Básico

**Objetivo**: Desplegar la aplicación en Cloud Run manualmente.

1. **Habilitar API de Cloud Run**:
   ```bash
   gcloud services enable run.googleapis.com
   ```

2. **Desplegar desde imagen en Artifact Registry**:
   ```bash
   export PROJECT_ID=$(gcloud config get-value project)
   gcloud run deploy app-info-service \
     --image europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo/app-info-service:latest \
     --region europe-west1 \
     --platform managed \
     --allow-unauthenticated \
     --set-env-vars "APP_NAME=App Info Service,ENVIRONMENT=production"
   ```

3. **Obtener URL del servicio**:
   Cloud Run mostrará la URL después del despliegue. También puedes obtenerla con:
   ```bash
   gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'value(status.url)'
   ```

4. **Probar el servicio**:
   ```bash
   SERVICE_URL=$(gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'value(status.url)')
   curl ${SERVICE_URL}/health
   curl ${SERVICE_URL}/
   curl ${SERVICE_URL}/info
   ```

5. **Ver logs del servicio**:
   ```bash
   gcloud run services logs read app-info-service --region europe-west1 --limit=50
   ```

**Ejercicio de verificación**:
- ¿Qué significa `--allow-unauthenticated`?
- ¿Por qué Cloud Run establece automáticamente la variable `PORT`?
- Accede a la URL en tu navegador y prueba los diferentes endpoints.

#### Subtarea 3.2: Configuración de Variables de Entorno

**Objetivo**: Configurar variables de entorno en Cloud Run.

1. **Actualizar servicio con variables de entorno**:
   ```bash
   gcloud run services update app-info-service \
     --region europe-west1 \
     --set-env-vars "APP_NAME=App Info Service,ENVIRONMENT=production,API_KEY=test-key-123"
   ```

2. **Verificar configuración**:
   ```bash
   gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'yaml(spec.template.spec.containers[0].env)'
   ```

3. **Probar que las variables están disponibles**:
   ```bash
   SERVICE_URL=$(gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'value(status.url)')
   curl ${SERVICE_URL}/config
   curl ${SERVICE_URL}/info
   ```

**Ejercicio de verificación**:
- ¿Por qué no deberías poner secretos en variables de entorno?
- ¿Qué diferencia hay entre `--set-env-vars` y `--update-env-vars`?

#### Subtarea 3.3: Configuración de Recursos y Escalado

**Objetivo**: Configurar recursos (CPU, memoria) y políticas de escalado.

1. **Configurar recursos**:
   ```bash
   gcloud run services update app-info-service \
     --region europe-west1 \
     --cpu 2 \
     --memory 1Gi \
     --max-instances 10 \
     --min-instances 0 \
     --concurrency 80
   ```

2. **Verificar configuración**:
   ```bash
   gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'yaml(spec.template.spec.containers[0].resources)'
   ```

3. **Configurar timeout**:
   ```bash
   gcloud run services update app-info-service \
     --region europe-west1 \
     --timeout 300
   ```

4. **Probar escalado**:
   Generar múltiples peticiones para ver cómo Cloud Run escala:
   ```bash
   SERVICE_URL=$(gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'value(status.url)')
   for i in {1..20}; do curl ${SERVICE_URL}/stats & done
   wait
   ```

**Ejercicio de verificación**:
- ¿Qué significa `--min-instances 0` vs `--min-instances 1`?
- ¿Cómo afecta la concurrency al número de instancias necesarias?
- ¿Qué es un "cold start" y cómo se puede evitar?

### Práctica 4: Cloud Run Avanzado con Secret Manager

**Duración estimada**: 60 minutos  
**Objetivo**: Integrar Secret Manager con Cloud Run y migrar aplicación completa desde Docker Compose local a Cloud Run.

#### Subtarea 4.1: Configuración de Secret Manager

**Objetivo**: Crear y gestionar secretos en Secret Manager.

1. **Habilitar API de Secret Manager**:
   ```bash
   gcloud services enable secretmanager.googleapis.com
   ```

2. **Crear secreto desde línea de comandos**:
   ```bash
   echo -n "Este es un mensaje secreto para Cloud Run" | \
     gcloud secrets create secret-message \
     --data-file=- \
     --replication-policy="automatic"
   ```

3. **Verificar secreto creado**:
   ```bash
   gcloud secrets list
   gcloud secrets versions access latest --secret="secret-message"
   ```

4. **Otorgar permisos al servicio de Cloud Run**:
   ```bash
   PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format='value(projectNumber)')
   gcloud secrets add-iam-policy-binding secret-message \
     --member="serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com" \
     --role="roles/secretmanager.secretAccessor"
   ```

**Ejercicio de verificación**:
- ¿Por qué es necesario otorgar permisos IAM al servicio de Cloud Run?
- ¿Qué significa `--replication-policy="automatic"`?

#### Subtarea 4.2: Integración con Cloud Run

**Objetivo**: Montar secretos en Cloud Run como variables de entorno.

1. **Desplegar servicio con secreto como variable de entorno**:
   ```bash
   gcloud run deploy app-info-service \
     --image europe-west1-docker.pkg.dev/${PROJECT_ID}/docker-repo/app-info-service:latest \
     --region europe-west1 \
     --platform managed \
     --allow-unauthenticated \
     --set-env-vars "APP_NAME=App Info Service,ENVIRONMENT=production" \
     --update-secrets "SECRET_MESSAGE=secret-message:latest"
   ```

2. **Verificar que el secreto está montado**:
   ```bash
   gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'yaml(spec.template.spec.containers[0].env)'
   ```

3. **Probar endpoint que usa el secreto**:
   ```bash
   SERVICE_URL=$(gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'value(status.url)')
   curl ${SERVICE_URL}/secret
   ```

4. **Montar secreto como archivo** (opcional):
   ```bash
   gcloud run services update app-info-service \
     --region europe-west1 \
     --update-secrets "/etc/secrets/secret-message=secret-message:latest"
   ```

**Ejercicio de verificación**:
- ¿Cuál es la diferencia entre montar un secreto como variable de entorno vs como archivo?
- ¿Qué significa `:latest` en el nombre del secreto?

#### Subtarea 4.3: Migración Completa

**Objetivo**: Migrar aplicación de Docker Compose local a Cloud Run con Secret Manager.

1. **Revisar aplicación local**:
   ```bash
   cd materiales/practica-4/ejemplo-migracion-completa
   cat docker-compose.yml
   ```

2. **Preparar secretos en Secret Manager**:
   ```bash
   echo -n "Mensaje secreto de producción" | \
     gcloud secrets create secret-message \
     --data-file=- \
     --replication-policy="automatic"
   ```

3. **Configurar cloudbuild.yaml para despliegue automático**:
   Crear o actualizar `cloudbuild.yaml` en la raíz del proyecto con:
   ```yaml
   steps:
     - name: 'gcr.io/cloud-builders/docker'
       args:
         - 'build'
         - '-t'
         - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:${BUILD_ID}'
         - '-t'
         - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:latest'
         - '.'
       dir: 'materiales/practica-4/ejemplo-migracion-completa'
   
     - name: 'gcr.io/cloud-builders/docker'
       args:
         - 'push'
         - '--all-tags'
         - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service'
   
     - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
       entrypoint: gcloud
       args:
         - 'run'
         - 'deploy'
         - '${_SERVICE_NAME}'
         - '--image'
         - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:${BUILD_ID}'
         - '--region'
         - '${_REGION}'
         - '--platform'
         - 'managed'
         - '--allow-unauthenticated'
         - '--set-env-vars'
         - 'APP_NAME=${_APP_NAME},ENVIRONMENT=${_ENVIRONMENT}'
         - '--update-secrets'
         - 'SECRET_MESSAGE=${_SECRET_NAME}:latest'
   
   images:
     - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:${BUILD_ID}'
     - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/app-info-service:latest'
   
   substitutions:
     _REGION: 'europe-west1'
     _REPO_NAME: 'docker-repo'
     _SERVICE_NAME: 'app-info-service'
     _APP_NAME: 'App Info Service'
     _ENVIRONMENT: 'production'
     _SECRET_NAME: 'secret-message'
     # _IMAGE_TAG: Para ejecución manual usa BUILD_ID directamente (BUILD_ID siempre está disponible).
     # Para triggers de Git, puedes cambiar esto a '${SHORT_SHA}' o pasar --substitutions=_IMAGE_TAG=${SHORT_SHA}
     # IMPORTANTE: BUILD_ID se usa directamente, no dentro de una substitution porque no se expande
     _IMAGE_TAG: '${BUILD_ID}'
   
   options:
     machineType: 'E2_HIGHCPU_8'
     logging: CLOUD_LOGGING_ONLY
   
   timeout: '1200s'
   ```

4. **Ejecutar build completo**:
   ```bash
   cd ../..  # Volver a la raíz del proyecto
   gcloud builds submit --config cloudbuild.yaml
   ```

5. **Verificar despliegue**:
   ```bash
   SERVICE_URL=$(gcloud run services describe app-info-service \
     --region europe-west1 \
     --format 'value(status.url)')
   curl ${SERVICE_URL}/health
   curl ${SERVICE_URL}/secret
   ```

**Ejercicio de verificación**:
- Compara el proceso de desarrollo local (Docker Compose) vs producción (Cloud Run). ¿Cuáles son las principales diferencias?
- ¿Qué ventajas tiene usar Secret Manager vs archivos `.env`?
- Documenta el proceso de migración en un archivo `MIGRACION.md`.

## Ejercicios de Verificación

Al finalizar cada práctica, verifica que has comprendido los conceptos:

### Práctica 1
- [ ] Puedo construir imágenes Docker localmente
- [ ] Puedo publicar imágenes en Docker Hub con el formato correcto
- [ ] Puedo publicar imágenes en Artifact Registry con el formato correcto
- [ ] Entiendo las diferencias entre Docker Hub y Artifact Registry

### Práctica 2
- [ ] Puedo crear un `cloudbuild.yaml` funcional
- [ ] Entiendo qué son las substitutions y cómo usarlas
- [ ] Puedo ejecutar builds manualmente con `gcloud builds submit`
- [ ] Puedo configurar triggers para builds automáticos

### Práctica 3
- [ ] Puedo desplegar aplicaciones en Cloud Run
- [ ] Puedo configurar variables de entorno en Cloud Run
- [ ] Entiendo cómo funciona el escalado automático
- [ ] Puedo ver y entender los logs de Cloud Run

### Práctica 4
- [ ] Puedo crear y gestionar secretos en Secret Manager
- [ ] Puedo integrar secretos con Cloud Run
- [ ] Entiendo cómo migrar de Docker Compose a Cloud Run
- [ ] Puedo configurar un pipeline completo de CI/CD

## Troubleshooting Básico

### Problemas Comunes

**Error: "unauthorized: authentication required"**:
- Verificar que estás autenticado: `docker login` para Docker Hub, `gcloud auth configure-docker` para Artifact Registry

**Error: "permission denied" en Cloud Build**:
- Verificar que el service account de Cloud Build tiene permisos en Artifact Registry
- Otorgar rol `roles/artifactregistry.writer` si es necesario

**Error: "Container failed to start" en Cloud Run**:
- Verificar que la aplicación escucha en el puerto definido por `PORT`
- Revisar logs: `gcloud run services logs read SERVICE_NAME --region REGION`

**Secretos no disponibles en Cloud Run**:
- Verificar permisos IAM del service account de Cloud Run
- Verificar que el nombre del secreto es correcto
- Verificar que el secreto existe: `gcloud secrets list`

## Ejercicios Adicionales (Opcionales)

1. **Configurar diferentes builds para diferentes ramas**:
   - Crear triggers que ejecuten diferentes configuraciones según la rama
   - Usar diferentes tags de imagen para desarrollo y producción

2. **Implementar health checks avanzados**:
   - Modificar el endpoint `/health` para verificar dependencias
   - Configurar health checks en Cloud Run

3. **Configurar escalado personalizado**:
   - Experimentar con diferentes valores de `min-instances` y `max-instances`
   - Medir el impacto en cold starts y costos

4. **Rotación de secretos**:
   - Crear una nueva versión de un secreto
   - Actualizar Cloud Run para usar la nueva versión sin downtime

## Recursos de Referencia

### Documentación Oficial

- **Docker Hub**: https://docs.docker.com/docker-hub/
- **Artifact Registry**: https://cloud.google.com/artifact-registry/docs
- **Cloud Build**: https://cloud.google.com/build/docs
- **Cloud Run**: https://cloud.google.com/run/docs
- **Secret Manager**: https://cloud.google.com/secret-manager/docs

### Tutoriales

- **Cloud Run Tutorials**: https://cloud.google.com/run/docs/tutorials
- **Cloud Build Tutorials**: https://cloud.google.com/build/docs/tutorials

### Herramientas

- **gcloud CLI Reference**: https://cloud.google.com/sdk/gcloud/reference
- **Cloud Build Configuration**: https://cloud.google.com/build/docs/build-config

