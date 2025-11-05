# app/Dockerfile
FROM python:3.10-slim

LABEL maintainer="docente@example.org"

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# SOLUCIÓN: Configurar mirrors confiables y manejar reintentos
RUN echo 'deb http://deb.debian.org/debian bullseye main' > /etc/apt/sources.list && \
    echo 'deb http://security.debian.org/debian-security bullseye-security main' >> /etc/apt/sources.list

# SOLUCIÓN: Mejorar la instalación con reintentos y --fix-missing
RUN apt-get update --fix-missing || apt-get update --fix-missing

# SOLUCIÓN: Instalación con manejo de errores
RUN apt-get install -y --no-install-recommends \
    build-essential \
    default-libmysqlclient-dev \
    pkg-config \
    curl \
    bash \
    && rm -rf /var/lib/apt/lists/* || \
    (apt-get update --fix-missing && \
     apt-get install -y --no-install-recommends \
        build-essential \
        default-libmysqlclient-dev \
        pkg-config \
        curl \
        bash \
    && rm -rf /var/lib/apt/lists/*)

# crear usuario no-root
RUN useradd --create-home --shell /bin/bash appuser

# usar /app como WORKDIR para coincidir con docker-compose si usas ambos enfoques
WORKDIR /app

# Copiar requirements (para cache de Docker)
COPY --chown=appuser:appuser requirements.txt .

# Instalar dependencias de Python
RUN pip install --no-cache-dir --upgrade pip
RUN pip install --no-cache-dir -r requirements.txt
RUN pip install pytest pylint

# Copiar el resto de la aplicación
COPY --chown=appuser:appuser . .

# Dar permisos de ejecución
RUN chmod -R 755 /app && \
    mkdir -p /app/.pytest_cache && \
    chmod 777 /app/.pytest_cache

RUN chmod +x entrypoint.sh wait_for_db.py

USER appuser

EXPOSE 5000

CMD ["./entrypoint.sh"]
