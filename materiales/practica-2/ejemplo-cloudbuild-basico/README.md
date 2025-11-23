# Ejemplo Cloud Build Básico

Configuración básica de Cloud Build para construir y publicar imágenes Docker automáticamente.

## Estructura

```
ejemplo-cloudbuild-basico/
├── cloudbuild.yaml      # Configuración de Cloud Build
└── README.md           # Este archivo
```

## Configuración

Este `cloudbuild.yaml` realiza dos pasos:

1. **Construir la imagen**: Usa Docker para construir la imagen desde el Dockerfile
2. **Publicar la imagen**: Sube la imagen a Artifact Registry con dos tags (SHORT_SHA y latest)

## Variables de Sustitución

- `_REGION`: Región de GCP (por defecto: europe-west1)
- `_REPO_NAME`: Nombre del repositorio en Artifact Registry (por defecto: docker-repo)

## Variables Automáticas de Cloud Build

- `PROJECT_ID`: ID del proyecto de GCP
- `SHORT_SHA`: Primeros 7 caracteres del commit SHA

## Uso

Ver la guía del estudiante para instrucciones detalladas sobre cómo configurar y ejecutar este build.

