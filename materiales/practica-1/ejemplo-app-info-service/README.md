# App Info Service - Ejemplo para S12

Aplicación Flask de ejemplo diseñada para demostrar conceptos de CI/CD, publicación de imágenes Docker y despliegue en Cloud Run.

## Características

Esta aplicación incluye:

- **Endpoints informativos**: Muestra información del contenedor y configuración
- **Health check**: Endpoint `/health` requerido por Cloud Run
- **Gestión de secretos**: Demuestra uso de variables de entorno y Secret Manager
- **Estadísticas simples**: Muestra escalado horizontal en Cloud Run

## Estructura

```
ejemplo-app-info-service/
├── app.py              # Aplicación Flask principal
├── Dockerfile          # Dockerfile para construir imagen
├── requirements.txt    # Dependencias Python
├── .dockerignore       # Archivos a ignorar en build
├── .env.example       # Plantilla de variables de entorno
└── README.md          # Este archivo
```

## Uso Local

### 1. Construir la imagen

```bash
docker build -t app-info-service:latest .
```

### 2. Ejecutar el contenedor

```bash
docker run -d \
  -p 8080:8080 \
  -e APP_NAME="Mi App" \
  -e ENVIRONMENT="development" \
  -e SECRET_MESSAGE="Mi secreto" \
  --name app-info \
  app-info-service:latest
```

### 3. Probar los endpoints

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

## Endpoints

- `GET /`: Información básica del servicio y endpoints disponibles
- `GET /health`: Health check para Cloud Run
- `GET /info`: Información del contenedor y entorno
- `GET /config`: Configuración sin exponer valores sensibles
- `GET /secret`: Endpoint que requiere secreto configurado
- `GET /stats`: Estadísticas simples (muestra escalado en Cloud Run)

## Variables de Entorno

- `APP_NAME`: Nombre de la aplicación
- `ENVIRONMENT`: Entorno (development, production, etc.)
- `SECRET_MESSAGE`: Mensaje secreto (desde Secret Manager en producción)
- `API_KEY`: Clave API opcional
- `PORT`: Puerto donde escucha la aplicación (Cloud Run lo establece automáticamente)

## Uso en Prácticas S12

Esta aplicación se utiliza en todas las prácticas de S12:

- **Práctica 1**: Publicación de imágenes en Docker Hub y Artifact Registry
- **Práctica 2**: Automatización con Cloud Build
- **Práctica 3**: Despliegue básico en Cloud Run
- **Práctica 4**: Cloud Run avanzado con Secret Manager

## Notas

- La aplicación está diseñada para ejecutarse en Cloud Run
- El puerto se configura automáticamente mediante la variable `PORT`
- Los secretos deben gestionarse mediante Secret Manager en producción
- El endpoint `/stats` demuestra escalado horizontal (cada instancia tiene su contador)

