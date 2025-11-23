# Materiales de Ejemplo - S12

Esta carpeta contiene archivos de ejemplo para las prácticas de la sesión S12 sobre Docker en CI/CD y Cloud Run.

## Estructura

- **practica-1/**: Ejemplos para la Práctica 1 (Publicación de Imágenes Docker)
  - `ejemplo-app-info-service/`: Aplicación Flask de ejemplo utilizada en todas las prácticas
  - Incluye Dockerfile, código fuente y documentación completa

- **practica-2/**: Ejemplos para la Práctica 2 (Automatización con Cloud Build)
  - `ejemplo-cloudbuild-basico/`: Configuración básica de Cloud Build
  - `ejemplo-cloudbuild-avanzado/`: Configuración avanzada con múltiples pasos

- **practica-3/**: Ejemplos para la Práctica 3 (Introducción a Cloud Run)
  - Los ejemplos utilizan la aplicación de `practica-1/`
  - Configuraciones de despliegue en Cloud Run

- **practica-4/**: Ejemplos para la Práctica 4 (Cloud Run Avanzado y Migración)
  - `ejemplo-migracion-completa/`: Migración completa de Docker Compose a Cloud Run
  - Incluye guía detallada de migración y configuración de Secret Manager

## Uso por Práctica

### Práctica 1: Publicación de Imágenes Docker

La aplicación `app-info-service` se utiliza para:
- Construir imágenes Docker localmente
- Publicar manualmente en Docker Hub
- Publicar manualmente en Artifact Registry

### Práctica 2: Automatización con Cloud Build

Los ejemplos de `cloudbuild.yaml` demuestran:
- Construcción automática de imágenes
- Publicación automática en Artifact Registry
- Configuración de triggers para CI/CD

### Práctica 3: Introducción a Cloud Run

La aplicación se despliega en Cloud Run para demostrar:
- Despliegue básico con `gcloud run deploy`
- Configuración de variables de entorno
- Health checks y logging

### Práctica 4: Cloud Run Avanzado y Migración

El ejemplo completo muestra:
- Migración de Docker Compose local a Cloud Run
- Integración con Secret Manager
- CI/CD completo con Cloud Build
- Mejores prácticas de seguridad

## Aplicación de Ejemplo: App Info Service

La aplicación `app-info-service` es una aplicación Flask simple diseñada específicamente para S12. Incluye:

- Endpoints informativos (`/`, `/info`, `/config`)
- Health check (`/health`) requerido por Cloud Run
- Gestión de secretos (`/secret`) para demostrar Secret Manager
- Estadísticas simples (`/stats`) que muestran escalado horizontal

Esta aplicación es intencionalmente simple para mantener el foco en los conceptos de CI/CD y Cloud Run, sin la complejidad adicional de bases de datos (que se cubrirá en S13).

## Notas

- Todos los ejemplos están diseñados para ser educativos y seguir mejores prácticas
- Los ejemplos son funcionales y han sido probados
- Consulta la guía del estudiante para el contexto completo de cada práctica
- La guía de migración en `practica-4/` es especialmente útil para entender el proceso completo

