# Ejemplo de Migración Completa

Este ejemplo demuestra la migración completa de una aplicación desde Docker Compose (desarrollo local) a Cloud Run (producción) con Secret Manager.

## Estructura

```
ejemplo-migracion-completa/
├── app.py                 # Aplicación Flask
├── Dockerfile            # Dockerfile (mismo para local y producción)
├── docker-compose.yml    # Configuración para desarrollo local
├── cloudbuild.yaml       # Configuración para CI/CD con Cloud Build
├── requirements.txt     # Dependencias Python
├── guia-migracion.md    # Guía detallada de migración
└── README.md            # Este archivo
```

## Desarrollo Local

### Ejecutar con Docker Compose

```bash
# Iniciar servicios
docker compose up --build

# Verificar funcionamiento
curl http://localhost:8080/health
curl http://localhost:8080/secret
```

### Variables de Entorno Locales

Las variables se definen en `docker-compose.yml` para desarrollo local.

## Producción en Cloud Run

### Requisitos Previos

1. Proyecto GCP con APIs habilitadas
2. Artifact Registry configurado
3. Secret Manager con secretos creados
4. Cloud Build configurado

### Despliegue Manual

```bash
# Construir y publicar imagen
gcloud builds submit --config cloudbuild.yaml

# O desplegar directamente
gcloud run deploy app-info-service \
  --source . \
  --region europe-west1 \
  --allow-unauthenticated
```

### Despliegue Automático

Configurar trigger en Cloud Build para ejecutar automáticamente en cada push.

## Migración Paso a Paso

Ver `guia-migracion.md` para instrucciones detalladas sobre:

- Preparación de la aplicación
- Migración de secretos
- Configuración de Cloud Build
- Despliegue en Cloud Run
- Configuración de CI/CD

## Conceptos Demostrados

- Migración de desarrollo local a producción
- Gestión de secretos con Secret Manager
- CI/CD con Cloud Build
- Despliegue automatizado en Cloud Run
- Mejores prácticas de seguridad

