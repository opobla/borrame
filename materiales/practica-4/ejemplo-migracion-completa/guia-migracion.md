# Guía de Migración: Docker Compose → Cloud Run

Esta guía documenta el proceso de migración de una aplicación desde Docker Compose (desarrollo local) a Cloud Run (producción).

## Comparación: Desarrollo Local vs Producción

### Desarrollo Local (Docker Compose)

- **Orquestación**: Docker Compose gestiona múltiples contenedores
- **Secretos**: Variables de entorno en `.env` o `docker-compose.yml`
- **Red**: Red Docker interna entre servicios
- **Persistencia**: Volúmenes Docker locales
- **Escalado**: Manual (docker compose up --scale)

### Producción (Cloud Run)

- **Orquestación**: Cloud Run gestiona el ciclo de vida del servicio
- **Secretos**: Secret Manager de GCP
- **Red**: Red pública de Internet (con opciones de VPC)
- **Persistencia**: Servicios gestionados (Cloud SQL, etc.)
- **Escalado**: Automático según tráfico

## Pasos de Migración

### 1. Preparar la Aplicación

- Asegurar que la aplicación escucha en `0.0.0.0`
- Usar variable `PORT` para el puerto (Cloud Run la establece)
- Eliminar dependencias de servicios locales (volúmenes, redes Docker)

### 2. Migrar Secretos a Secret Manager

```bash
# Crear secreto en Secret Manager
gcloud secrets create secret-message \
  --data-file=- \
  --replication-policy="automatic"

# O desde línea de comandos
echo -n "Mi mensaje secreto" | gcloud secrets create secret-message \
  --data-file=-
```

### 3. Configurar Cloud Build

Crear `cloudbuild.yaml` que:
- Construya la imagen Docker
- Publique en Artifact Registry
- Despliegue en Cloud Run con secretos

### 4. Desplegar en Cloud Run

```bash
# Despliegue manual inicial
gcloud run deploy app-info-service \
  --image europe-west1-docker.pkg.dev/PROJECT_ID/docker-repo/app-info-service:latest \
  --region europe-west1 \
  --platform managed \
  --allow-unauthenticated \
  --set-env-vars "APP_NAME=App Info Service,ENVIRONMENT=production" \
  --update-secrets "SECRET_MESSAGE=secret-message:latest"
```

### 5. Configurar CI/CD Automático

- Crear trigger en Cloud Build
- Configurar para ejecutarse en cada push a la rama principal
- Verificar despliegue automático

## Checklist de Migración

- [ ] Aplicación escucha en `0.0.0.0` y usa variable `PORT`
- [ ] Secretos migrados a Secret Manager
- [ ] Cloud Build configurado y probado
- [ ] Artifact Registry configurado
- [ ] Cloud Run desplegado y funcionando
- [ ] Health checks funcionando correctamente
- [ ] Variables de entorno configuradas
- [ ] Secretos montados correctamente
- [ ] CI/CD automático configurado
- [ ] Documentación actualizada

## Consideraciones Importantes

1. **Puerto**: Cloud Run establece `PORT` automáticamente, la aplicación debe usarlo
2. **Secretos**: Nunca hardcodear secretos en código o imágenes
3. **Health Checks**: Implementar endpoint `/health` para Cloud Run
4. **Timeouts**: Cloud Run tiene límites de timeout (por defecto 300s)
5. **Escalado**: Cloud Run escala automáticamente, no gestionar manualmente

## Troubleshooting

### La aplicación no inicia

- Verificar que escucha en `0.0.0.0` y puerto correcto
- Revisar logs: `gcloud run services logs read app-info-service`

### Secretos no disponibles

- Verificar permisos del servicio de Cloud Run
- Comprobar nombre del secreto y versión
- Verificar formato de `--update-secrets`

### Build falla en Cloud Build

- Verificar que `cloudbuild.yaml` está en la raíz del repositorio
- Comprobar rutas relativas en los pasos
- Revisar permisos del servicio de Cloud Build

