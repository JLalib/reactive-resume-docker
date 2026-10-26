# 🚀 Reactive Resume Docker

[![GitHub](https://img.shields.io/badge/GitHub-Repo-blue?logo=github)](https://github.com/amruthpillai/Reactive-Resume)
[![Docker](https://img.shields.io/badge/Docker-amruthpillai%2Freactive--resume-blue?logo=docker)](https://hub.docker.com/r/amruthpillai/reactive-resume)
[![License](https://img.shields.io/badge/License-MIT-green?logo=opensourceinitiative)](https://github.com/amruthpillai/Reactive-Resume/blob/main/LICENSE)

## 📋 Descripción general

**Reactive Resume** es un creador de currículums y cartas de presentación de código abierto, autohospedado y libre de rastreadores, anuncios y planes de pago. Despliegas la aplicación con Docker Compose junto a una base de datos PostgreSQL y todos tus currículums permanecen en tu propio hardware. La aplicación sirve la web, la API, el servidor MCP y los archivos subidos desde un único puerto (3000), y genera los PDF internamente sin necesidad de servicios de impresión externos.

## ✨ Características principales

- **Editor con vista previa en vivo**: Modos de escritura, diseño y revisión con previsualización del PDF en tiempo real.
- **Gestión múltiple**: Varios currículums y cartas de presentación desde una misma cuenta.
- **Generación de PDF nativa**: La propia aplicación crea los PDF, sin dependencias externas.
- **Subida de imágenes**: Almacenamiento local en carpeta `/app/data` o en bucket compatible con S3.
- **Autenticación flexible**: Cuenta propia con verificación por email y proveedores OAuth (Google, GitHub, LinkedIn) opcionales.
- **Funciones IA opcionales**: Búsqueda de empleo y asistente con IA, desactivadas por defecto y configurables mediante proveedor externo.
- **Privacidad total**: Sin rastreadores, sin anuncios, sin telemetría; tus datos nunca salen de tu servidor.

## 📋 Requisitos del sistema

- **Docker Engine** y plugin **Docker Compose** (o Docker Desktop).
- **Hardware mínimo**: 1 vCPU y 1 GB RAM para la app; reservar 2 GB si PostgreSQL corre en el mismo host.
- **Disco**: ~10 GB iniciales (crece según imágenes subidas).
- **Arquitecturas**: `amd64` y `arm64` (incluye Raspberry Pi 4+ con SO de 64 bits).
- **Opcional**: Dominio propio y proxy inverso con HTTPS para acceso externo seguro.

## 🐳 Instalación

### Paso 1: Crea la carpeta del proyecto
```bash
mkdir reactive-resume && cd reactive-resume
```

### Paso 2: Crea el fichero `.env`
```bash
# Dirección que usarás en el navegador
APP_URL="http://localhost:3000"

# Contraseña de la base de datos (la misma en las dos líneas)
POSTGRES_PASSWORD="cambia-esta-contrasena"
DATABASE_URL="postgresql://postgres:cambia-esta-contrasena@postgres:5432/postgres"

# Genera con: openssl rand -hex 32
AUTH_SECRET=""
```
> **Importante**: Sustituye `cambia-esta-contrasena` por una contraseña segura y rellena `AUTH_SECRET` con un valor generado (`openssl rand -hex 32`). No dejes los valores de ejemplo.

### Paso 3: Crea el `compose.yml`
```yaml
services:
  postgres:
    image: postgres:18
    restart: unless-stopped
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d postgres"]
      interval: 10s
      timeout: 5s
      retries: 10

  reactive-resume:
    image: amruthpillai/reactive-resume:latest
    restart: unless-stopped
    ports:
      - "3000:3000"
    env_file: .env
    volumes:
      - reactive_resume_data:/app/data
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres_data:
  reactive_resume_data:
```
> **Registro alternativo**: Para usar GitHub Container Registry, cambia la imagen a `ghcr.io/reactive-resume/reactive-resume:latest`.

### Paso 4: Arranca los contenedores
```bash
docker compose up -d --wait --wait-timeout 180
```
Si el arranque falla o agota el tiempo:
```bash
docker compose ps
docker compose logs --tail=50 reactive-resume
```

### Paso 5: Crea tu cuenta
Abre `http://localhost:3000` y regístrate. Entrarás directamente; la app también envía un email de verificación. Sin SMTP configurado, el enlace de verificación aparece en los logs:
```bash
docker compose logs reactive-resume | grep verify-email
```

## ⚙️ Configuración

1. **Variables de entorno obligatorias** en `.env`:
   - `APP_URL`: URL exacta que escribirás en el navegador (incluye `https://` si usas TLS).
   - `POSTGRES_PASSWORD` y `DATABASE_URL`: Credenciales de PostgreSQL (deben coincidir).
   - `AUTH_SECRET`: Clave de 64 caracteres hex (`openssl rand -hex 32`).

2. **Desactivar registros públicos** (tras crear tu cuenta):
   ```bash
   FLAG_DISABLE_SIGNUPS="true"
   docker compose up -d
   ```

3. **HTTPS y dominio propio**: Configura un proxy inverso (Traefik, Caddy, nginx) y actualiza `APP_URL` a la URL pública (ej. `https://resume.tudominio.com`). **Debe coincidir exactamente** con la barra del navegador.

4. **Correo electrónico (SMTP)**: Añade variables `SMTP_*` según la [referencia oficial](https://rxresu.me/docs/self-hosting/environment-variables) para verificación y recuperación de contraseña.

5. **Almacenamiento S3 (opcional)**: Configura variables `S3_*` para guardar subidas en bucket compatible S3 en lugar de la carpeta local.

6. **Redis (opcional)**: Para compartir estado entre procesos; añade servicio Redis y variables `REDIS_*` correspondientes.

## 🚀 Primeros pasos

1. Accede a `http://localhost:3000` (o tu dominio) y crea tu cuenta.
2. Pulsa **"New Resume"** y elige plantilla o empieza desde cero.
3. Edita secciones en el **modo Escritura**; cambia a **Vista previa** para ver el PDF en vivo.
4. Usa el **modo Revisión** para comprobar ortografía y formato.
5. Exporta a **PDF** desde el botón superior derecho del editor.
6. (Opcional) Crea **cartas de presentación** vinculadas al mismo currículum.

## 💡 Casos de uso

- **Búsqueda de empleo**: Currículums y cartas versionados en tu propio servidor, sin límites ni suscripciones.
- **Entorno familiar/amigos**: Instancia privada multi-cuenta con registros desactivados tras alta inicial.
- **Homelab**: Servicio más en tu stack, integrado con tus copias de seguridad y monitorización habituales.
- **Portafolio técnico**: Mantén versiones históricas de tu CV y adapta plantillas por oferta.

## 🔒 Acceso remoto seguro

1. Despliega un **proxy inverso** (Traefik, Caddy, Nginx Proxy Manager) con certificado TLS válido (Let's Encrypt o propio).
2. Configura el proxy para forward a `http://reactive-resume:3000`.
3. Actualiza `.env`: `APP_URL="https://resume.tudominio.com"`.
4. Reinicia: `docker compose up -d`.
5. Verifica que el login, cookies y redirecciones OAuth funcionan correctamente.

## 🛠️ Gestión y mantenimiento

### Ver estado
```bash
docker compose ps
```

### Ver logs en tiempo real
```bash
docker compose logs -f reactive-resume
```

### Actualizar a la última versión estable
```bash
docker compose pull
docker compose up -d
```
> **Versionado**: `latest` = última estable. Para fijar mayor, usa `v6`. Etiqueta `nightly` = rama principal (solo pruebas).

### Comprobar salud de la instancia
```bash
curl http://localhost:3000/api/health
```
Devuelve `200` si BD, almacenamiento y Redis (si configurado) responden; `503` si alguno falla.

### Copias de seguridad (imprescindibles)
Haz backup de **ambos volúmenes**:
- `postgres_data` → Base de datos PostgreSQL (usuarios, currículums, cartas).
- `reactive_resume_data` → Archivos subidos (fotos, adjuntos).

Sin ambos no es posible restaurar la instancia completa.

### Detener la instancia
```bash
docker compose down
```

## 📝 Licencia

Código bajo licencia **MIT** — libre para uso personal, comercial, modificación y distribución.  
Consulta el [LICENSE](https://github.com/amruthpillai/Reactive-Resume/blob/main/LICENSE) del repositorio oficial.

---

> 📖 **Guía completa y vídeo tutorial**: [Cómo instalar Reactive Resume en Docker - Creador de currículums autohospedado](https://genbyte.blogspot.com/2026/10/como-instalar-reactive-resume-en-docker.html)