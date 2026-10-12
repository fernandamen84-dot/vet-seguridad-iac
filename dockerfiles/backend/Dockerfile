# ============================================================
# BACKENDBD-1 - Backend Flask (Veterinaria Huellitas)
# Proyecto: Vet_Seguridad
# Autor: Fernanda Mendoza
# Fecha: Octubre 2026
# ============================================================
FROM alpine:latest

# Metadatos
LABEL maintainer="Fernanda Mendoza"
LABEL description="Backend Flask para el proyecto Vet_Seguridad"
LABEL version="1.0"

# Instalar dependencias del sistema
RUN apk update && apk add --no-cache \
    git \
    python3 \
    py3-pip \
    mariadb-client \
    nano \
    curl \
    bash

# Clonar el repositorio de la aplicación desde GitHub
RUN git clone https://github.com/fernandamen84-dot/veterinaria-app.git /veterinaria-app

# Crear entorno virtual e instalar dependencias de Python
WORKDIR /veterinaria-app
RUN python3 -m venv venv && \
    . venv/bin/activate && \
    pip install --no-cache-dir flask mysql-connector-python

# Modificar db.py para deshabilitar SSL
# (Necesario porque MariaDB en Alpine usa certificados autofirmados)
RUN sed -i "s/database=os.getenv('DB_NAME', 'veterinaria_db')/database=os.getenv('DB_NAME', 'veterinaria_db'),\n        ssl_disabled=True/" db.py

# Verificar que la modificación se aplicó correctamente
RUN grep "ssl_disabled" db.py

# Copiar script de inicio
COPY setup.sh /setup.sh
RUN chmod +x /setup.sh

# Exponer puerto de Flask
EXPOSE 80

# Comando de inicio
CMD ["/setup.sh"]