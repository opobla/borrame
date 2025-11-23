# S12: Docker en CI/CD y Cloud Run - Material para Estudiantes

Este repositorio contiene todo el material necesario para completar las prácticas de la sesión S12 sobre Docker en CI/CD (Continuous Integration/Continuous Deployment) y Cloud Run.

## 📚 Estructura del Repositorio

```
s12-estudiantes/
├── README.md                    # Este archivo - Introducción y guía rápida
├── guia-estudiante.md          # Guía completa con teoría y prácticas paso a paso
├── materiales/                 # Materiales de ejemplo para consultar
│   ├── practica-1/
│   │   └── ejemplo-app-info-service/
│   ├── practica-2/
│   │   └── ejemplo-cloudbuild-basico/
│   └── practica-4/
│       └── ejemplo-migracion-completa/
└── ejercicios/                 # Directorio para tu trabajo (crear según necesidad)
```

## 🚀 Inicio Rápido

1. **Lee este README** para entender la estructura del repositorio
2. **Consulta `guia-estudiante.md`** para la guía completa con teoría y prácticas
3. **Trabaja con los ejemplos** en `materiales/` según las instrucciones de cada práctica
4. **Crea tus archivos** en `ejercicios/` cuando sea necesario según las prácticas

## 📖 Contenido de las Prácticas

### Práctica 1: Publicación de Imágenes Docker
- Publicación manual en Docker Hub
- Publicación manual en Artifact Registry
- Comparativa de registros

**Material de ejemplo**: `materiales/practica-1/ejemplo-app-info-service/`

### Práctica 2: Automatización con Cloud Build
- Configuración de `cloudbuild.yaml`
- Ejecución manual de builds
- Configuración de triggers automáticos

**Material de ejemplo**: `materiales/practica-2/ejemplo-cloudbuild-basico/`

### Práctica 3: Introducción a Cloud Run
- Despliegue básico en Cloud Run
- Configuración de variables de entorno
- Recursos y escalado

**Material de ejemplo**: Usa la aplicación de `materiales/practica-1/`

### Práctica 4: Cloud Run Avanzado con Secret Manager
- Gestión de secretos con Secret Manager
- Integración con Cloud Run
- Migración completa de Docker Compose a Cloud Run

**Material de ejemplo**: `materiales/practica-4/ejemplo-migracion-completa/`

## ✅ Requisitos Previos

Antes de comenzar, asegúrate de tener:

### Software Instalado
- **Docker Desktop** (o Docker Engine + Docker Compose) - versión 24.0+
- **Google Cloud SDK (gcloud CLI)** - versión 450.0.0+
- **Editor de código** (VS Code o Cursor)
- **Git** para control de versiones

### Cuentas y Recursos
- **Proyecto GCP activo** con facturación habilitada
- **APIs habilitadas** en tu proyecto:
  - Artifact Registry API
  - Cloud Build API
  - Cloud Run API
  - Secret Manager API
- **Docker Hub** (cuenta gratuita opcional)

### Verificación Rápida

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

## 📝 Cómo Usar los Materiales de Ejemplo

Los materiales en `materiales/` son ejemplos funcionales que puedes:

1. **Consultar como referencia** para entender la estructura y configuración
2. **Copiar y adaptar** según las instrucciones de cada práctica
3. **Usar como base** para tus propias implementaciones

**Importante**: Los materiales de ejemplo están diseñados para ser educativos. Adapta los valores (PROJECT_ID, nombres de servicios, etc.) a tu propio proyecto.

## 🛠️ Flujo de Trabajo Típico

### Ejemplo: Práctica 1

```bash
# 1. Consultar el ejemplo
cat materiales/practica-1/ejemplo-app-info-service/app.py

# 2. Construir la imagen localmente (según guía)
cd materiales/practica-1/ejemplo-app-info-service
docker build -t app-info-service:latest .

# 3. Seguir los pasos de la guía para publicar
# (ver guia-estudiante.md para pasos detallados)
```

### Ejemplo: Práctica 2

```bash
# 1. Consultar ejemplo de cloudbuild.yaml
cat materiales/practica-2/ejemplo-cloudbuild-basico/cloudbuild.yaml

# 2. Crear tu propia configuración (si es necesario)
mkdir -p ejercicios/practica-2
# Crear o adaptar cloudbuild.yaml según práctica
```

## 📚 Recursos Adicionales

- **Guía completa**: Consulta `guia-estudiante.md` para teoría detallada y pasos completos
- **Documentación oficial**:
  - [Docker Hub](https://docs.docker.com/docker-hub/)
  - [Artifact Registry](https://cloud.google.com/artifact-registry/docs)
  - [Cloud Build](https://cloud.google.com/build/docs)
  - [Cloud Run](https://cloud.google.com/run/docs)
  - [Secret Manager](https://cloud.google.com/secret-manager/docs)

## ❓ Troubleshooting

Si encuentras problemas:

1. Consulta la sección "Troubleshooting Básico" en `guia-estudiante.md`
2. Verifica que todos los requisitos previos están cumplidos
3. Revisa los logs de errores con detalle
4. Consulta la documentación oficial de GCP

## 📧 Soporte

Para dudas específicas sobre las prácticas, consulta:
- La guía del estudiante (`guia-estudiante.md`)
- La documentación oficial de GCP
- Los comentarios y README.md en cada ejemplo de `materiales/`

---

**Última actualización**: Consulta el historial de commits para ver cambios recientes.
