# Guía de Comandos Específica para la OVA — Examen T1 (DAW)

> **Entorno:** OVA real del examen · Ejercicios en orden · Sin Internet · 24 noviembre 2026

## Resumen del entorno detectado

- **CONFIRMADO (Git — 1.1):** Repositorio `agenda-cultural` con ramas `main`, `develop`, `feature/filtro-categorias` y `hotfix/correo-contacto`, y etiquetas **SemVer** (`v1.0.0`, `v1.1.0`, `v1.2.0`).
- **CONFIRMADO (Docker — 2.1):** Imágenes `centro/panel-avisos` con etiquetas `1.0`, `1.1`, `2.0-beta` y `latest` (mismo ID que `2.0-beta`).
- **CONFIRMADO (Dockerfile — 2.2):** Proyecto Python `horario-app` con tests unitarios (`unittest`) y un archivo sensible `notas_dev.md`.
- **HIPÓTESIS (Compose — 2.3):** WordPress + MariaDB + Adminer (`mariadb:11`, `wordpress:php8.3-apache`, `adminer:4`) por las imágenes precargadas en la OVA. *El enunciado definitivo podría variar.*

> **REGLAS CRÍTICAS DE SEGURIDAD:**
> - Trabaja **siempre** dentro de tu copia en `~/examen`.
> - **NO** publiques ni pegues en entregables los datos internos ni el token de `notas_dev.md`.
> - **NO** ejecutes borrados (`--delete`, `down -v`) ni `git push` salvo cuando el PDF del examen lo exija expresamente.

---

## 0 · Preparación

### 0.1 · Comprobar rutas, imágenes y preparar carpetas

```bash
pwd
ls -ld /materiales_examen ~/materiales_examen ~/examen
ls -lah /materiales_examen
docker image ls
docker ps -a
docker compose version
mkdir -p ~/examen/entrega/capturas
cd ~/examen
```

> **ADAPTAR:** `~/materiales_examen` NO existe en la OVA; la ruta real es **`/materiales_examen`**.

---

## 1.1 · Git: `agenda-cultural`

### 1 · Desbloquear clon local (`dubious ownership`) y clonar

```bash
git config --global --add safe.directory /materiales_examen/repos/agenda-cultural.git
cd ~/examen
git clone /materiales_examen/repos/agenda-cultural.git agenda-cultural
cd agenda-cultural
```

> **ADAPTAR:** El error real en la OVA es `dubious ownership` porque el repo en `/materiales_examen` pertenece a otro usuario. Añade exclusivamente la ruta de este remoto de confianza.

### 2 · Configurar identidad local y alias `grafo`

```bash
git config --local user.name "TU NOMBRE COMPLETO"
git config --local user.email "dawserver@examen.local"
git config --local alias.grafo "log --oneline --graph --all --decorate"
git config --local --list
git grafo
```

> **ADAPTAR:** Usa el correo exacto que pida el enunciado del examen si especifica otro. Añadir `--local` garantiza que `git config --local --list` muestre solo la configuración de este repo.

### 3 · Crear ramas locales con seguimiento (*tracking*)

```bash
git fetch origin
git branch -a
git branch --track develop origin/develop
git branch --track feature/filtro-categorias origin/feature/filtro-categorias
git branch --track hotfix/correo-contacto origin/hotfix/correo-contacto
git branch -vv
```

> **ADAPTAR:** Omite cualquier rama que ya exista localmente (como `main`). Nunca crees una rama para `origin/HEAD`.

### 4 · Auditar historial y etiquetas

```bash
git grafo
git tag -n
git show v1.2.0
git log --oneline v1.2.0..main
git log --oneline v1.2.0..develop
git log --oneline --first-parent --no-merges v1.2.0..main
git show 0a574ed
git show dd39e98
```

> **ADAPTAR (Teoría clave — SemVer):** La OVA usa **Versionado Semántico (SemVer)** `vMAYOR.MENOR.PARCHE` (`v1.0.0`, `v1.1.0`, `v1.2.0`), **NO CalVer** como el simulacro:
> - **MAYOR (`1`):** cambios incompatibles con versiones anteriores.
> - **MENOR (`2`):** nuevas funcionalidades compatibles hacia atrás.
> - **PARCHE (`0`):** correcciones de errores (*hotfix* / *bugfix*) compatibles hacia atrás. Un *hotfix* sobre `v1.2.0` sería **`v1.2.1`**; una nueva funcionalidad sería **`v1.3.0`**.

### 5 · Ver ramas integradas y pendientes

```bash
git branch -r --merged main
git branch -r --merged develop
git branch -r --no-merged main
git branch -r --no-merged develop
git log --oneline main..hotfix/correo-contacto
git log --oneline main..develop
```

> **ADAPTAR (Teoría clave — Git Flow vs GitHub Flow):**
> Justifica con el **grafo**, no solo por los nombres de rama:
> 1. Existen **dos ramas permanentes** (`main` para producción y `develop` para integración), a diferencia de GitHub Flow que solo tiene `main`.
> 2. Las ramas `feature/*` nacen de `develop` y se integran en `develop`.
> 3. Las ramas `hotfix/*` nacen de `main` para corregir un fallo urgente en producción y se integran **tanto en `main` (con etiqueta de parche) como en `develop`**.

### 6 · Integrar el `hotfix` (solo si lo pide el enunciado)

```bash
git switch main
git merge --no-ff hotfix/correo-contacto -m "Integrar corrección de correo"
git tag -a v1.2.1 -m "Versión 1.2.1"

git switch develop
git merge --no-ff hotfix/correo-contacto -m "Propagar hotfix a develop"
git grafo
```

> **ADAPTAR:** `v1.2.1` es un ejemplo SemVer (verifica la versión exacta que pida el PDF). Si al fusionar en `develop` aparece un conflicto, resuélvelo con `nano`, haz `git add` y `git commit` antes de continuar.

### 7 · Publicar y limpiar (solo si lo pide el enunciado)

```bash
git switch main
git push origin main
git push origin v1.2.1

git switch develop
git push origin develop

git branch -r --merged main
git branch -d hotfix/correo-contacto
git push origin --delete hotfix/correo-contacto
```

> **ADAPTAR:** `push` modifica el remoto y `--delete` elimina la rama remota: ejecútalos únicamente si el enunciado lo exige.

### 8 · Variante: crear e integrar una `feature` en Git Flow

```bash
git switch develop
git switch -c feature/nueva-funcion
nano ARCHIVO
git add ARCHIVO
git commit -m "feat: nueva función"

git switch develop
git merge --no-ff feature/nueva-funcion -m "Integrar feature"
git branch -d feature/nueva-funcion
```

> **ADAPTAR:** En **Git Flow**, las `feature/` salen de `develop` y vuelven a `develop` (en GitHub Flow usan `main`).

### 9 · Variantes: deshacer (`revert`), `--amend` y conflicto en `push`

```bash
# Deshacer un commit YA publicado de forma segura
git log --oneline -10
git show HASH
git revert HASH

# Corregir el último commit SOLO si todavía NO se ha hecho push
git add ARCHIVO
git commit --amend --no-edit

# Si un push se rechaza por cambios nuevos en el remoto
git pull --no-rebase origin main
git status
nano ARCHIVO
git add ARCHIVO
git commit -m "Resolver conflicto"
git push origin main
```

> **ADAPTAR:** Usa `git revert` para commits ya publicados. Nunca uses `git reset --hard` ni `git push --force` salvo petición expresa.

### 10 · Comprobaciones finales de Git

```bash
git status
git branch -a
git branch -vv
git tag -n
git grafo
```

---

## 1.2 · Markdown y evidencias

### 1 · Preparar `README.md` y carpeta de entrega

```bash
cd ~/examen
mkdir -p entrega/capturas
nano README.md
cp README.md entrega/README.md
cat entrega/README.md
```

> **ADAPTAR:** Mantén las secciones habituales de la rúbrica: `Descripción`, `Arquitectura`, `Requisitos`, `Puesta en marcha`, `Configuración`, `Comprobación` y `Referencias`.

### 2 · Fragmentos Markdown que suelen puntuarse

````markdown
# Título principal

## Descripción

Texto con **negrita**, *cursiva* y `código en línea`.

## Arquitectura

```text
navegador -> web -> db
```

| Servicio | Imagen | Puerto | Redes |
| :--- | :--- | ---: | :--- |
| `web` | `wordpress` | `8081` | `frontend`, `backend` |

## Puesta en marcha

1. Validar la configuración:
   ```bash
   docker compose config
   ```

## Comprobación

- [ ] Servicio sano (`healthy`).

> **Aviso de seguridad:** No incluir `.env` en Git.

![Evidencia del estado de los servicios](capturas/estado.png)

## Referencias

- [Documentación oficial de Docker Compose](https://docs.docker.com/compose/)
````

> **ADAPTAR:** Sustituye los servicios por los reales del ejercicio 2.3 y asegúrate de que la imagen referenciada exista en `capturas/`.

---

## 2.1 · Docker: `centro/panel-avisos`

### 1 · Inspeccionar las versiones reales

```bash
docker image ls centro/panel-avisos
docker image inspect -f '{{json .Config.ExposedPorts}}' centro/panel-avisos:1.1
docker image inspect -f '{{json .Config.Env}}' centro/panel-avisos:1.1
docker image inspect -f '{{json .Config.Labels}}' centro/panel-avisos:1.1
docker image inspect -f '{{index .Config.Labels "es.centro.panel.fichero-avisos"}}' centro/panel-avisos:1.1
```

> **ADAPTAR:**
> - Puerto interno: `80/tcp`.
> - `1.0` **no** admite `CENTRO_NOMBRE`; `1.1` **sí**; `2.0-beta` está marcada como *beta* y no apta para producción.
> - Si quieres ver el JSON formateado, añade `| jq` al final de los comandos con `{{json ...}}`.

### 2 · Comprobar etiquetas `estado` y la trampa de `latest`

```bash
docker image inspect -f '{{index .Config.Labels "es.centro.panel.estado"}}' centro/panel-avisos:1.0
docker image inspect -f '{{index .Config.Labels "es.centro.panel.estado"}}' centro/panel-avisos:1.1
docker image inspect -f '{{index .Config.Labels "es.centro.panel.estado"}}' centro/panel-avisos:2.0-beta

docker image inspect -f '{{.Id}}' centro/panel-avisos:latest
docker image inspect -f '{{.Id}}' centro/panel-avisos:2.0-beta
```

> **CLAVE DE EXAMEN:** `latest` y `2.0-beta` comparten exactamente el **mismo ID**. Por eso en producción nunca se debe usar `:latest` sin comprobar qué versión apunta: hay que fijar explícitamente la etiqueta estable (`centro/panel-avisos:1.1`).

### 3 · Arrancar, consultar y diagnosticar

```bash
docker run -d --name avisos \
  -p 8081:80 \
  -e CENTRO_NOMBRE="IES Armando Cotarelo Valledor" \
  centro/panel-avisos:1.1

docker ps -a
curl -i http://localhost:8081
docker logs avisos
docker inspect -f '{{.State.ExitCode}}' avisos
```

> **ADAPTAR:** Cambia `8081` por el puerto del host que pida el enunciado (y `80` solo si cambiara el puerto interno). `CENTRO_NOMBRE` es específica de `1.1` y `2.0-beta`.

### 4 · Copiar y consultar `avisos.txt`

```bash
docker exec avisos cat /usr/share/nginx/html/avisos.txt
docker cp avisos:/usr/share/nginx/html/avisos.txt ~/examen/avisos.txt
cat ~/examen/avisos.txt
```

> **ADAPTAR:** La ruta `/usr/share/nginx/html/avisos.txt` se obtiene de la etiqueta `es.centro.panel.fichero-avisos`.

### 5 · Persistencia con volumen nombrado (si lo pide el enunciado)

```bash
docker volume create avisos-datos
docker rm -f avisos

docker run -d --name avisos -p 8081:80 \
  -e CENTRO_NOMBRE="IES Armando Cotarelo Valledor" \
  -v avisos-datos:/usr/share/nginx/html \
  centro/panel-avisos:1.1

docker volume inspect avisos-datos
docker restart avisos

# Prueba de persistencia destruyendo el contenedor
docker rm -f avisos
docker run -d --name avisos -p 8081:80 \
  -e CENTRO_NOMBRE="IES Armando Cotarelo Valledor" \
  -v avisos-datos:/usr/share/nginx/html \
  centro/panel-avisos:1.1
```

> **ADAPTAR (Matiz importante):** Este volumen monta el directorio web **completo** (`/usr/share/nginx/html`), no solo `avisos.txt`. Al inicializar un volumen vacío copia los archivos de la imagen, pero si luego actualizas la imagen a otra versión montando el mismo volumen, los archivos viejos del volumen ocultarán el contenido nuevo de la imagen.

### 6 · Variante: *bind mount* de un solo fichero

```bash
# Requiere que ~/examen/avisos.txt exista previamente en el host
docker rm -f avisos
docker run -d --name avisos -p 8081:80 \
  -e CENTRO_NOMBRE="IES Armando Cotarelo Valledor" \
  --mount type=bind,src="$HOME/examen/avisos.txt",dst=/usr/share/nginx/html/avisos.txt \
  centro/panel-avisos:1.1
```

> **ADAPTAR:** Añade `,readonly` al final de `--mount` solo si el ejercicio exige que el contenedor no pueda modificar el archivo.

---

## 2.2 · Dockerfile: `horario-app`

### 1 · Copiar el proyecto y ejecutar los tests en local

```bash
cp -a /materiales_examen/proyectos/horario-app ~/examen/
cd ~/examen/horario-app
find . -maxdepth 3 -type f
python3 -m unittest discover -s tests -v
```

> **ADAPTAR:** `datos/` y `static/` son necesarios en ejecución; `tests/` solo se usa en la fase de pruebas; `notas_dev.md` **nunca** debe entrar en la imagen.

### 2 · Crear `.dockerignore`

*(Puedes crearlo con `nano .dockerignore` o con el bloque siguiente).*

```bash
cat > .dockerignore << 'EOF'
notas_dev.md
.git/
__pycache__/
*.pyc
.env
EOF
```

> **ADAPTAR:** **NO** excluyas `tests/` en `.dockerignore` si vas a ejecutar `unittest` en la primera etapa del `Dockerfile`. `notas_dev.md` sí debe excluirse siempre porque contiene datos internos/token.

### 3 · Crear el `Dockerfile` de dos etapas

*(Puedes crearlo con `nano Dockerfile` o con el bloque siguiente).*

```dockerfile
# --- ETAPA 1: tests (ejecuta pruebas unitarias) ---
FROM python:3.13-alpine AS tests
WORKDIR /app
COPY app.py ./
COPY datos/ ./datos/
COPY static/ ./static/
COPY tests/ ./tests/
RUN python -m unittest discover -s tests -v

# --- ETAPA 2: imagen final de ejecución ---
FROM python:3.13-alpine
ARG APP_VERSION=desarrollo
LABEL org.opencontainers.image.version="${APP_VERSION}"
WORKDIR /app
COPY --from=tests /app/app.py ./app.py
COPY --from=tests /app/datos/ ./datos/
COPY --from=tests /app/static/ ./static/
ENV APP_VERSION="${APP_VERSION}" GRUPO="2DAW" PUERTO="8000"
EXPOSE 8000
HEALTHCHECK --interval=10s --timeout=3s CMD python -c 'import os,urllib.request;urllib.request.urlopen("http://127.0.0.1:"+os.environ["PUERTO"],timeout=2)' || exit 1
CMD ["python", "app.py"]
```

> **ADAPTAR:**
> - **Diferencia clave respecto al simulacro:** aquí Python **SÍ** debe estar en la imagen final porque `app.py` es un servidor web en Python (no un generador estático). La ventaja del *multi-stage* aquí es ejecutar las pruebas en la etapa `tests` (si fallan, se aborta el `build`) y dejar `tests/` fuera de la imagen final.
> - Si el enunciado pidiera también usar `APP_VERSION` dentro de la etapa `tests`, recuerda declarar `ARG APP_VERSION=desarrollo` **en ambas etapas**.
> - Si el enunciado pidiera ejecutar sin `root`, añade `RUN adduser -D appuser` y `USER appuser` antes de `EXPOSE` (en Alpine se usa `adduser -D`, no `useradd`).

### 4 · Construir, ejecutar y verificar

```bash
docker build --build-arg APP_VERSION=1.3.0 -t horario-app:1.3.0 .

docker run -d --name horario -p 8083:8000 \
  -e GRUPO=1DAW \
  -e PUERTO=8000 \
  horario-app:1.3.0

docker ps
docker logs horario
curl -i http://localhost:8083/
curl -i http://localhost:8083/no-existe
docker inspect -f '{{.State.Health.Status}}' horario
```

> **ADAPTAR (Trampa de `curl`):** `1.3.0` y `8083` son ejemplos. **NO uses `curl -I`** con esta app: `app.py` implementa `GET`, no `HEAD`, por lo que `curl -I` puede devolver `501 Unsupported method ('HEAD')`. Usa `curl -i` (minúscula).

### 5 · Comprobar `LABEL`, ausencia de secretos/tests y etapa `tests`

```bash
docker image inspect -f '{{json .Config.Labels}}' horario-app:1.3.0 | jq
docker run --rm --entrypoint sh horario-app:1.3.0 -c 'ls -lah /app'
docker run --rm --entrypoint sh horario-app:1.3.0 -c 'test ! -f /app/notas_dev.md && test ! -d /app/tests && echo OK'

docker build --target tests -t horario-app:tests .
docker image ls horario-app
```

> **ADAPTAR:** Nunca incluyas el token de `notas_dev.md` en capturas ni en `respuestas.md`.

---

## 2.3 · Docker Compose (HIPÓTESIS: WordPress + MariaDB + Adminer)

> **AVISO:** Este apartado es una **plantilla hipotética** basada en las imágenes presentes en la OVA (`mariadb:11`, `wordpress:php8.3-apache`, `adminer:4`). Adapta nombres de servicios, puertos, variables y redes exactamente a lo que pida el PDF definitivo.

### 1 · Crear carpeta, `.env` y `.gitignore`

```bash
mkdir -p ~/examen/wordpress-compose
cd ~/examen/wordpress-compose

cat > .env << 'EOF'
DB_PASSWORD=CAMBIAR_CLAVE_LOCAL
DB_ROOT_PASSWORD=CAMBIAR_CLAVE_ADMIN
EOF

printf '.env\n' > .gitignore
```

> **ADAPTAR:** Cambia las contraseñas en `.env` y nunca subas `.env` a Git.

### 2 · Crear `compose.yaml` de ejemplo

```yaml
services:
  db:
    image: mariadb:11
    environment:
      MARIADB_DATABASE: wordpress
      MARIADB_USER: wordpress
      MARIADB_PASSWORD: ${DB_PASSWORD:?Falta DB_PASSWORD}
      MARIADB_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:?Falta DB_ROOT_PASSWORD}
    volumes:
      - db-datos:/var/lib/mysql
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 10

  wp:
    image: wordpress:php8.3-apache
    ports:
      - "127.0.0.1:8081:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD:?Falta DB_PASSWORD}
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp-datos:/var/www/html
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend
      - backend
    restart: unless-stopped

  adminer:
    image: adminer:4
    ports:
      - "127.0.0.1:8082:8080"
    environment:
      ADMINER_DEFAULT_SERVER: db
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend
      - backend
    restart: unless-stopped

volumes:
  db-datos:
  wp-datos:

networks:
  frontend:
  backend:
```

> **ADAPTAR:**
> - Si el enunciado pide solo `wp` y `db`, elimina el bloque `adminer`.
> - `db` **no publica el puerto `3306`** hacia el host (solo es accesible por la red interna `backend`).
> - Si vas a acceder mediante **reenvío de puertos de VirtualBox** en vez de túnel SSH, cambia `"127.0.0.1:8081:80"` por `"8081:80"` (si el enunciado no exige `127.0.0.1`).

### 3 · Validar, levantar y probar persistencia

```bash
# Validar sin exponer contraseñas en pantalla
docker compose config --quiet

# Levantar y comprobar estado y logs
docker compose up -d
docker compose ps
docker compose logs --tail 50 db
docker compose logs --tail 50 wp
docker volume ls
docker network ls
curl -i http://localhost:8081

# En PowerShell del anfitrión (si se permite túnel SSH):
# ssh -p 2222 -L 8081:127.0.0.1:8081 dawserver@localhost

# Probar persistencia (down conserva volúmenes)
docker compose down
docker compose up -d

# SOLO si el examen exige BORRAR datos y volúmenes:
docker compose down -v
```

> **ADAPTAR:** `docker compose config` completo muestra las contraseñas en texto claro; usa `docker compose config --quiet` si no quieres que salgan en capturas (o censúralas si el enunciado pide la salida completa).

---

## Extra · Errores habituales y diagnóstico rápido

### Git: sintaxis de `--no-ff`

```bash
git switch main
git merge --no-ff RAMA -m "Integración"
```

> No escribas `--no --ff` ni `--no--ff`.

### Git: `push` rechazado

```bash
git fetch origin
git pull --no-rebase origin main
git status
# Si hay conflicto: nano ARCHIVO -> git add ARCHIVO -> git commit
git push origin main
```

> Cambia `main` por `develop` si trabajas sobre `develop`. No uses `--force`.

### Docker: diagnóstico rápido y puertos ocupados

```bash
docker ps -a
docker logs NOMBRE
docker inspect -f '{{.State.ExitCode}}' NOMBRE
docker inspect -f '{{json .Mounts}}' NOMBRE
ss -tln | grep ':8081 '
```

> Cambia `NOMBRE` y el puerto según corresponda. No borres contenedores ajenos.

---

## Entrega final

```bash
cd ~/examen
find entrega -maxdepth 3 -type f
zip -r entrega_examen.zip entrega
```

> **ADAPTAR:** Usa exactamente el nombre de carpeta y archivo `.zip` que indique el PDF definitivo del examen.