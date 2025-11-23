# Dockerfile para App Info Service
# Mismo Dockerfile usado en desarrollo local y producción

FROM python:3.11-slim

WORKDIR /app

# Copiar requirements e instalar dependencias
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código de la aplicación
COPY app.py .

# Cloud Run requiere que la aplicación escuche en el puerto definido por PORT
ENV PORT=8080

EXPOSE 8080

CMD ["python", "-m", "flask", "run", "--host=0.0.0.0", "--port=8080"]

