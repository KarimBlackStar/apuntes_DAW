# Contador de visitas — Despliegue con Docker Compose

## Descripción

**Contador de visitas** es una aplicación web que registra las visitas mediante *Flask* y guarda el contador en **Redis**. Un servidor **Nginx** actúa como proxy inverso para que la aplicación sea accesible desde el puerto `8080` de la máquina virtual.

El despliegue se define en `compose.yaml` y sus parámetros se configuran en `.env`.

## Arquitectura

```text
Navegador / curl
      |
      | localhost:8080 (VM) -> puerto 80
      v
+------------------+          red frontend
| proxy            |----------------------------+
| nginx:alpine     |                            |
+------------------+                            v
                                         +--------------------+
                                         | web                |
                                         | Flask :5000        |
                                         | sin puerto público |
                                         +--------------------+
                                                  |
                                                  | red backend
                                                  v
                                         +--------------------+
                                         | redis              |
                                         | redis:7-alpine     |
                                         | puerto interno 6379|
                                         | volumen redis-datos|
                                         +--------------------+
```

| Servicio | Imagen | Puerto publicado en VM | Redes |
|:---|:---|:---:|:---|
| `proxy` | `nginx:alpine` | `8080:80` | `frontend` |
| `web` | `contador-visitas:1.0` (build `./app`) | Ninguno | `frontend`, `backend` |
| `redis` | `redis:7-alpine` | Ninguno | `backend` |

El servicio `proxy` **no está conectado** a `backend` y no puede resolver `redis` por la red interna. `web` comunica las dos redes. Redis almacena sus datos en el volumen con nombre `redis-datos`.

## Requisitos

- Máquina Linux con **Docker Engine** y **Docker Compose** disponibles.
- Imágenes requeridas descargadas o accesibles desde el registro de imágenes.
- Archivos `compose.yaml`, `.env`, `app/` y `proxy/default.conf` en el proyecto.
- Puerto `8080` de la VM disponible.

## Puesta en marcha

1. **Situarse en la carpeta del proyecto.**

   ```bash
   cd ~/simulacro_examen/contador-visitas
   ```

2. **Preparar la configuración** de `.env` con los valores indicados en la siguiente sección; no incluir contraseñas reales en el README.

   ```bash
   nano .env
   ```

3. **Validar** la configuración final que interpreta Compose.

   ```bash
   docker compose config --quiet
   ```

4. **Construir y arrancar** los servicios en segundo plano.

   ```bash
   docker compose up -d --build
   ```

5. **Comprobar** los servicios y el estado de Redis.

   ```bash
   docker compose ps
   ```

6. **Probar** el contador mediante el proxy.

   ```bash
   curl http://localhost:8080
   curl http://localhost:8080
   ```

7. **Detener** el proyecto sin eliminar el volumen persistente cuando terminen las pruebas.

   ```bash
   docker compose down
   ```

## Configuración

Las variables necesarias en el archivo `.env` son:

- **Redis (`redis`)**
  - `REDIS_PASSWORD`: contraseña obligatoria para el acceso a Redis; sustituir el valor de ejemplo por uno seguro.
- **Aplicación (`web`)**
  - `APP_VERSION=1.0`: versión visible en la aplicación.
  - `REDIS_HOST=redis`: nombre del servicio Redis, configurado en `compose.yaml`.
  - `REDIS_PASSWORD`: se transmite a la aplicación desde la configuración de Compose.
- **Proxy (`proxy`)**
  - Publica `8080:80`; su archivo `proxy/default.conf` se monta en modo solo lectura.

Ejemplo **sin credenciales reales** de `.env`:

```dotenv
REDIS_PASSWORD=CAMBIAR_POR_UNA_CLAVE_SEGURA
APP_VERSION=1.0
```

> **Aviso de seguridad:** nunca publiques el archivo `.env` ni contraseñas en Git o en capturas. Usa `.gitignore` para excluir `.env` y comparte, en su lugar, un `.env.example` sin secretos. Si una clave se ha publicado, debe cambiarse.

## Comprobación

- [ ] `docker compose config --quiet` termina sin errores.
- [ ] `docker compose ps` muestra los tres servicios en ejecución.
- [ ] Redis aparece como `healthy` tras iniciar.
- [ ] `curl http://localhost:8080` devuelve una respuesta de la aplicación.
- [ ] El contador aumenta entre peticiones sucesivas.
- [ ] El puerto `5000` de `web` **no** se publica en la VM.
- [ ] El proxy está conectado a `frontend`, pero no a `backend`.
- [ ] Tras `docker compose down` y `docker compose up -d`, el contador conserva sus datos.
- [ ] El archivo `proxy/default.conf` está montado en modo solo lectura.

Captura del despliegue (guardar la imagen real en `capturas/compose-servicios.png`):

![Salida de docker compose ps con los servicios proxy, web y redis y el estado de salud](capturas/compose-servicios.png)

## Referencias

- [Documentación oficial de Docker Compose](https://docs.docker.com/compose/)
- [Documentación oficial de Redis](https://redis.io/docs/latest/)
- [Documentación oficial de Nginx](https://nginx.org/en/docs/)

---

*Documento técnico de despliegue para el ejercicio 1.2. Ajustar nombres, rutas, imágenes y puertos si cambia el enunciado del examen.*
