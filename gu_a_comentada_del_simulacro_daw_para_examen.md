# Guía Comentada del Simulacro T1 — DAW (Apuntes de Examen)

Este documento reúne todo lo trabajado en tu simulacro (**Ejercicios 1.1 a/b/c, 1.2, 2.1, 2.2 y 2.3**), con **cada comando explicado parámetro a parámetro**, las **respuestas teóricas listas y pulidas**, la **plantilla completa de `README.md`** y los **avisos de errores reales** para no repetirlos el día del examen.

---

## ⚠️ Checklist anti-despistes (errores a evitar en el examen)

Antes de entregar tu `respuestas.md` real, revisa estos puntos:

1. **Sustituye siempre los marcadores de ejemplo:** nunca escribas literalmente `HASH` ni `2026.11.X`; cámbialos por el hash real (ej. `54849aa`) o la versión real (`2026.11.0`).
2. **Entra en la carpeta del repo antes de `git config`:** ejecutar `git config` fuera de la carpeta clonada da `fatal: not in a git directory`.
3. **Usa `--local` y el formato pedido por el enunciado:** pon tu nombre completo, el correo `TU_USUARIO@simulacro.local` y muestra solo la config local con `git config --local --list` (no `git config -l`, que mezcla la global).
4. **No crees ramas con `origin/HEAD`:** al hacer `git branch -r`, ignora siempre `origin/HEAD` y `origin/main`.
5. **No dejes frases de guía pegadas en la entrega:** borra textos como *"Qué poner en respuestas.md:"*; escribe los hashes, resultados y nombres concretos.
6. **Incluye todos los bloques de código que pida el enunciado:** pega el contenido de `.gitignore`, `CHANGELOG.md`, `Dockerfile`, `.dockerignore`, `.env` y `compose.yaml` como bloques de código en `respuestas.md`.
7. **Numeración exacta:** usa los mismos números que el enunciado (`1.1 a)`, `1.1 b)`, `1.1 c)`, `2.1`, `2.2`, `2.3`...) para facilitar la corrección.

---

# Bloque 1: Git, GitHub Flow y Documentación (UD1)

> **Regla de oro del enunciado:** usa siempre `git merge --no-ff` al integrar una rama en `main` (simula una *Pull Request* aprobada dejando constancia del *merge commit* en el historial).

## Ejercicio 1.1 a) — Clonado y configuración local

### Comandos correctos paso a paso

```bash
# 1. Clonar el repositorio remoto (ORIGEN -> DESTINO) y entrar en él
mkdir -p ~/simulacro_examen
git clone ~/materiales_simulacro/repos/biblioteca-web.git ~/simulacro_examen/biblioteca-web
cd ~/simulacro_examen/biblioteca-web
```

* **Nota:** `git clone` siempre recibe primero el **origen** (el repo `.git`) y opcionalmente el **destino**. Si pones primero el destino vacío, Git dirá `repository ... does not exist`.

```bash
# 2. Configurar identidad y alias SOLO para este repositorio (--local)
git config --local user.name "Karim Hanouni Otero"
git config --local user.email "dawserver@simulacro.local"
git config --local alias.grafo "log --oneline --graph --all --decorate"
```

* `--local`: guarda la configuración únicamente en `.git/config` de este proyecto, sin tocar tu configuración global.
* `alias.grafo`: crea el atajo `git grafo`, que combina:
  * `--oneline`: un commit por línea (hash corto + mensaje).
  * `--graph`: dibuja el árbol ASCII de ramas y fusiones.
  * `--all`: muestra todas las ramas, no solo la actual.
  * `--decorate`: enseña los punteros (`HEAD`, ramas y `tag:`).

```bash
# 3. Mostrar SOLO la configuración local y el historial gráfico
git config --local --list
git grafo
```

---

## Ejercicio 1.1 b) — Auditoría del repositorio

### Paso 1: Crear en local una rama por cada rama remota (siguiéndola)

```bash
# Ver todas las ramas (locales y remotas)
git branch -a

# Crear ramas locales vinculadas (tracking) a sus remotas
# (Ignora siempre origin/HEAD y origin/main)
git branch --track feature/novedades origin/feature/novedades
git branch --track feature/prestamos origin/feature/prestamos

# Comprobar que siguen a su remota (debe salir [origin/...])
git branch -vv
```

* **Alternativa válida:** `git checkout -b feature/novedades origin/feature/novedades` (crea y cambia a la rama).
* **Si creas una rama por error:** se borra con `git branch -D <nombre>` (no existe `git branch rm`).

### Paso 2: Mostrar historial completo y etiquetas con sus mensajes

```bash
git grafo
git tag -n
```

* `git tag -n` muestra cada etiqueta junto a su mensaje:
  * `2026.09.0` — Versión 2026.09.0: catálogo de libros destacados
  * `2026.10.0` — Versión 2026.10.0: horario de apertura
  * `2026.10.1` — Versión 2026.10.1: corrección del enlace al catálogo

### Paso 3: Mostrar todos los commits en `main` desde la última versión publicada

```bash
git log --oneline 2026.10.1..main
```

* La sintaxis `A..B` muestra todos los commits alcanzables desde `B` (`main`) que **no** estaban todavía en `A` (`2026.10.1`).

### Paso 4: Detectar commits hechos **directamente** en `main` (saltándose el flujo)

```bash
git log --oneline --first-parent --no-merges 2026.10.1..main
```

* `--first-parent`: recorre solo la línea principal de `main` (sin meterse dentro de las ramas fusionadas).
* `--no-merges`: oculta los commits de fusión (`Merge branch ...`).
* **Resultado real del repo:**
  * `54849aa` — `style: rediseño de la cabecera`
  * `156a2ce` — `chore: configuración del catálogo en línea`

```bash
# Inspeccionar qué tocó cada commit sospechoso (usando el hash REAL)
git show 54849aa
git show 156a2ce
```

### Respuestas modelo pulidas para el 1.1 b)

1. **¿Qué evidencias hay de que el proyecto usa GitHub Flow y no Git Flow? (al menos dos)**
   * Existe una **única rama permanente** (`main`); no hay rama `develop` ni ramas `release/` propias de Git Flow.
   * Todas las ramas de trabajo (`feature/catalogo`, `feature/horario`, `fix/enlace-catalogo`, `feature/novedades`, `feature/prestamos`) **nacen directamente de `main` y se integran de vuelta en `main`** mediante *merge* `--no-ff`.
   * Las etiquetas de versión (`2026.09.0`, `2026.10.0`, `2026.10.1`) se crean directamente sobre commits de `main`.

2. **Significado de `2026.10.1` y numeración de hoy (23/11/2026)**
   * Sigue versionado de calendario (**CalVer**) `AAAA.MM.N`: **`2026`** es el año, **`10`** el mes (octubre) y **`1`** el número de publicación dentro de ese mes (empezando en `0`, como se ve en `2026.09.0` y `2026.10.0`).
   * Si se publicara una versión hoy (noviembre de 2026), le correspondería **`2026.11.0`**.
   * Si se publicara otra más este mismo mes, sería **`2026.11.1`**.

3. **Commits que se han saltado el flujo y problema que introduce cada uno**
   * **`54849aa` (`style: rediseño de la cabecera`):** hecho directamente en `main` sin rama ni revisión; introduce un rediseño roto/no aprobado en la cabecera (`css/style.css` e `index.html`).
   * **`156a2ce` (`chore: configuración del catálogo en línea`):** hecho directamente en `main`; sube al repositorio un archivo `.env` con credenciales/configuración sensible que nunca debe versionarse.

4. **Ramas remotas: cuál puede borrarse y cuál no debe integrarse**
   * **Ya integrada (puede borrarse):** `origin/feature/novedades` (commit `b93aebd`), porque ya fue fusionada en `main` en el merge `83e65b3`.
   * **En desarrollo (NO debe integrarse todavía):** `origin/feature/prestamos` (commit `1b89532`, *"docs: borrador de la funcionalidad de préstamos"*), porque sigue en estado borrador y sin terminar.

---

## Ejercicio 1.1 c) — Deshacer errores y publicar

### Paso 0: Crear la rama de limpieza desde `main`

```bash
git switch main
git switch -c fix/limpieza-repositorio
```

### Paso 1: Deshacer el rediseño de la cabecera (`54849aa`)

```bash
git revert 54849aa
```

* **¿Por qué `git revert` y NO `git reset`?**
  Porque el commit `54849aa` ya está publicado en el repositorio remoto (`origin/main`). Usar `git reset` reescribiría el historial compartido, obligando a hacer `git push --force` y provocando conflictos y desincronización al resto del equipo. `git revert` crea un commit nuevo que aplica los cambios inversos de forma segura sin alterar la historia previa.

### Paso 2: Dejar de versionar `.env` sin borrarlo del disco

```bash
echo ".env" >> .gitignore
git rm --cached .env
git add .gitignore
git commit -m "chore: dejar de versionar .env y añadir .gitignore"

# Verificación doble
ls -la .env                 # Debe seguir existiendo en tu disco
git ls-files | grep .env    # NO debe devolver nada (ya no está rastreado)
```

* `--cached` elimina el archivo únicamente del **índice (staging)** de Git, dejando intacto el archivo físico en tu carpeta.
* **Contenido de `.gitignore` (pégalo en `respuestas.md`):**
  ```text
  .env
  ```
* **¿Ha dejado de ser accesible la clave de `.env`? ¿Qué debe hacer el equipo?**
  **No.** El archivo sigue guardado en los commits anteriores del historial (cualquiera puede verlo con `git show 156a2ce`). El equipo debe **revocar y rotar inmediatamente esa clave** por una nueva que nunca haya pasado por Git (y opcionalmente purgar el historial con `git filter-repo`, aunque rotar la contraseña es obligatorio siempre).

### Paso 3: Actualizar `CHANGELOG.md`

```bash
cat CHANGELOG.md   # Consultar el formato de entradas anteriores
nano CHANGELOG.md  # Añadir la entrada de 2026.11.0 arriba
```

Ejemplo de entrada siguiendo el formato estándar (*Keep a Changelog*):

```markdown
## [2026.11.0] - 2026-11-23
### Añadido
- Archivo `.gitignore` para excluir archivos de entorno (`.env`).
### Corregido
- Revertido el rediseño no aprobado de la cabecera (`54849aa`).
### Eliminado
- Retirado el archivo `.env` del control de versiones de Git.
```

```bash
git add CHANGELOG.md
git commit -m "docs: actualizar CHANGELOG para versión 2026.11.0"
```

### Paso 4: Integrar en `main` con `--no-ff`, borrar la rama y etiquetar

```bash
git switch main
git merge --no-ff fix/limpieza-repositorio -m "Merge branch 'fix/limpieza-repositorio'"
git branch -d fix/limpieza-repositorio
git tag -a 2026.11.0 -m "Versión 2026.11.0: limpieza del repositorio"
```

* `-a`: crea una **etiqueta anotada** (guarda autor, fecha y mensaje `-m`), obligatoria para publicar versiones.

### Paso 5: Subir `main`, la etiqueta y borrar la rama ya integrada del remoto

```bash
# Comprobar qué ramas remotas ya están integradas en main
git branch -r --merged main

# Subir main y la etiqueta
git push origin main
git push origin 2026.11.0

# Borrar la rama ya integrada (feature/novedades) en remoto y en local
git push origin --delete feature/novedades
git branch -d feature/novedades

# Grafo final
git grafo
```

* **Truco de rescate (si te equivocas en el nombre de una etiqueta):**
  ```bash
  git tag -d 2026.11.X                  # Borrar etiqueta en local
  git push origin --delete 2026.11.X    # Borrar etiqueta en remoto
  ```

---

## Ejercicio 1.2 — Documentación técnica en Markdown (`README.md`)

> **Recuerda:** en la entrega hay dos `.md` distintos: `respuestas.md` (evidencias de todos los ejercicios) y `README.md` (el manual técnico del proyecto del 2.3, que es el único donde se puntúa la sintaxis Markdown — **1,5 puntos**).

### Checklist de los 10 elementos obligatorios

1. **Encabezados:** `#` para el título principal y `##` para cada sección (`Descripción`, `Arquitectura`, `Requisitos`, `Puesta en marcha`, `Configuración`, `Comprobación`, `Referencias`).
2. **Diagrama de arquitectura:** esquema ASCII dentro de un bloque ```` ```text ````.
3. **Tabla con alineación explícita:** columnas con `:---`, `:---:`, `---:` listando servicios, imagen, puerto y redes.
4. **Lista numerada:** pasos de puesta en marcha (`1.`, `2.`...) cada uno con su bloque ```` ```bash ````.
5. **Lista anidada:** variables de `.env` agrupadas por servicio (sin contraseñas reales).
6. **Lista de tareas:** comprobaciones con `- [ ]`.
7. **Cita (`>`):** aviso de seguridad sobre no subir `.env` a Git ni a imágenes.
8. **Imagen con ruta relativa y texto alternativo:** `![Descripción](capturas/ej2_3_comprobacion.png)`.
9. **Enlaces externos:** `[Texto](https://...)` a documentación oficial.
10. **Formato en línea:** `` `código` `` para comandos/archivos/variables, **negrita** y *cursiva*.

### Plantilla completa de `README.md` (adaptada al proyecto 2.3 `contador-visitas`)

Guárdala como `README.md` en tu carpeta de entrega y verifica el resultado en VS Code con `Ctrl + Shift + V`:

````markdown
# Contador de visitas

## Descripción

Aplicación web *multi-contenedor* desarrollada con **Flask** que contabiliza y persiste el número de visitas de cada petición, utilizando **Redis** para la persistencia de datos y **Nginx** como *proxy inverso* y único punto de entrada. El despliegue completo se gestiona de forma declarativa mediante `Docker Compose`.

## Arquitectura

Los servicios se distribuyen en las redes internas `frontend` y `backend` para aislar la capa de datos del acceso exterior y ocultar la aplicación detrás del proxy inverso:

```text
[Cliente / Navegador]
         |
         | HTTP (puerto 8080 de la VM -> 80 del contenedor)
         v
   +-------------+       red: frontend       +------------------+
   |    proxy    | ------------------------> |       web        |
   | (Nginx :80) |    http://web:5000        |  (Flask :5000)   |
   +-------------+                           +------------------+
                                                      |
                                                      | red: backend
                                                      v
                                             +------------------+
                                             |      redis       |
                                             |  (Redis :6379)   |
                                             |   vol: /data     |
                                             +------------------+
```

| Servicio | Imagen | Puerto publicado | Redes |
| :--- | :--- | :---: | ---: |
| `proxy` | `nginx:alpine` | `8080:80` | `frontend` |
| `web` | `contador-visitas:1.0` *(build `./app`)* | Ninguno *(interno `5000`)* | `frontend`, `backend` |
| `redis` | `redis:7-alpine` | Ninguno *(interno `6379`)* | `backend` |

## Requisitos

- **Docker Engine** y el plugin **Docker Compose** (`docker compose version`).
- Herramienta `curl` para comprobar las respuestas HTTP.
- Puerto `8080` libre en la máquina anfitriona.

## Puesta en marcha

1. Accede al directorio del proyecto:
   ```bash
   cd ~/simulacro_examen/contador-visitas
   ```

2. Crea el archivo `.env` con las variables de configuración necesarias:
   ```bash
   cat > .env << 'EOF'
   REDIS_PASSWORD=cambia-esta-clave
   APP_VERSION=1.0
   EOF
   ```

3. Valida la sintaxis de `compose.yaml` y la sustitución de variables:
   ```bash
   docker compose config
   ```

4. Construye la imagen de la aplicación y levanta todos los servicios en segundo plano:
   ```bash
   docker compose up -d --build
   ```

5. Comprueba el estado de los contenedores y verifica que `redis` figure como `healthy`:
   ```bash
   docker compose ps
   ```

## Configuración

Variables definidas en el archivo `.env` agrupadas por servicio *(sin incluir contraseñas reales)*:

- Servicio `redis`
  - `REDIS_PASSWORD`: contraseña obligatoria usada por `--requirepass` y por el *healthcheck*. Si no está definida, Compose detiene el arranque con error.
- Servicio `web`
  - `REDIS_PASSWORD`: misma contraseña para autenticarse contra Redis.
  - `APP_VERSION`: versión de la aplicación que se muestra en la respuesta (por defecto `1.0`).
  - `REDIS_HOST`: nombre del servicio de Redis en la red interna (`redis`, fijada en `compose.yaml`).

> **Aviso de seguridad:** No subas nunca el archivo `.env` a Git ni lo copies dentro de una imagen Docker. Añádelo siempre a `.gitignore` y comparte únicamente un archivo `.env.example` con valores ficticios.

## Comprobación

Lista de verificaciones que debe completar quien despliega el proyecto:

- [ ] Ejecutar `docker compose ps` y confirmar que todos los servicios están `Up` y `redis` figura como `healthy`.
- [ ] Lanzar varias peticiones con `curl http://localhost:8080` y verificar que el contador se incrementa.
- [ ] Confirmar que `curl http://localhost:5000` falla porque el puerto de `web` no está publicado al exterior.
- [ ] Verificar el aislamiento de redes: `docker compose exec proxy ping -c 1 web` debe responder y `docker compose exec proxy ping -c 1 redis` debe fallar.
- [ ] Comprobar la persistencia ejecutando `docker compose down` seguido de `docker compose up -d` y verificando que el contador conserva su valor.
- [ ] Probar el escalado con `docker compose up -d --scale web=3` y `docker compose restart proxy`, comprobando que cambia el identificador de `servido por` manteniendo un único contador compartido.

![Comprobación del estado de los servicios y contador de visitas funcionando en contador-visitas](capturas/ej2_3_comprobacion.png)

## Referencias

- [Documentación oficial de Docker Compose](https://docs.docker.com/compose/)
- [Documentación oficial de Nginx](https://nginx.org/en/docs/)
- [Documentación oficial de Redis](https://redis.io/docs/latest/)
````

---

# Bloque 2: Docker y Docker Compose (UD2)

## Ejercicio 2.1 — Diagnóstico y persistencia

### Paso 1: Arranque inicial (provocar el error)

```bash
docker run -d --name asistencia -p 8084:80 centro/registro-asistencia:1.0
docker ps
```

* `-d`: segundo plano (*detached*).
* `--name asistencia`: asigna nombre fijo al contenedor.
* `-p 8084:80`: publica el puerto `80` interno del contenedor en el `8084` de la VM.
* **Resultado:** `docker ps` sale vacío porque el contenedor falló al arrancar y se detuvo.

### Paso 2: Diagnóstico del fallo

```bash
docker ps -a
docker inspect -f '{{.State.ExitCode}}' asistencia
docker logs asistencia
```

* `docker ps -a`: muestra **todos** los contenedores, incluidos los detenidos (`Exited (1)`).
* `docker inspect -f '{{.State.ExitCode}}'`: extrae solo el código de salida (`0` = éxito, distinto de `0` = error; aquí devuelve **`1`**).
* `docker logs asistencia`: *(ojo: es `logs` en plural, no `log`)* muestra la causa exacta:
  `ERROR: no se ha definido la variable de entorno CENTRO (nombre del centro).`

### Paso 3: Metadatos, borrado y recreación con volumen

```bash
docker image inspect centro/registro-asistencia:1.0
```

* En el bloque `Labels` de la imagen vemos dónde guarda sus datos:
  * `"es.centro.registro.datos": "/datos"`
  * `"es.centro.registro.fichero": "/datos/registro.log"`

```bash
docker rm asistencia
docker run -d --name asistencia -p 8084:80 \
  -e CENTRO="IES Armando Cotarelo Valedor" \
  -v asistencia-datos:/datos \
  centro/registro-asistencia:1.0
```

* `docker rm asistencia`: borra el contenedor detenido para poder reutilizar el nombre `asistencia`.
* `-e CENTRO="..."`: pasa la variable de entorno obligatoria.
* `-v asistencia-datos:/datos`: crea (si no existe) y monta el **volumen con nombre** `asistencia-datos` en `/datos`.

### Paso 4: Comprobar arranques y reinicio

```bash
curl http://localhost:8084     # Arranques registrados: 1
docker restart asistencia
curl http://localhost:8084     # Arranques registrados: 2
```

* `docker restart` detiene y vuelve a arrancar **el mismo contenedor** (mismo ID), sumando un segundo arranque.

### Paso 5: Probar la persistencia destruyendo el contenedor

```bash
docker rm -f asistencia
docker run -d --name asistencia -p 8084:80 \
  -e CENTRO="IES Armando Cotarelo Valedor" \
  -v asistencia-datos:/datos \
  centro/registro-asistencia:1.0
curl http://localhost:8084     # Arranques registrados: 3
```

* `-f`: fuerza la parada y borrado de un contenedor en ejecución.
* **Explicación de examen (¿qué habría pasado sin el volumen?):**
  El contador marca **3 arranques** porque el volumen con nombre `asistencia-datos` tiene un ciclo de vida independiente al del contenedor y conservó `/datos/registro.log` tras el borrado. **Sin el volumen**, el archivo se habría escrito en la capa escribible efímera del contenedor, se habría destruido con `docker rm -f` y el nuevo contenedor habría empezado otra vez en **1**.

### Paso 6: Extraer el registro al host y contar contenedores

```bash
docker cp asistencia:/datos/registro.log ~/simulacro_examen/registro.log
cat ~/simulacro_examen/registro.log
```

* Salida real obtenida:
  ```text
  2026-11-23 18:41:42 arranque  centro=IES Armando Cotarelo Valedor  contenedor=d797036bd141
  2026-11-23 18:44:09 arranque  centro=IES Armando Cotarelo Valedor  contenedor=d797036bd141
  2026-11-23 18:47:02 arranque  centro=IES Armando Cotarelo Valedor  contenedor=7d0a5dd2330b
  ```
* **Respuesta de examen:** aparecen **3 líneas** (3 arranques), pero solo **2 contenedores distintos** (`d797036bd141` y `7d0a5dd2330b`), porque los dos primeros arranques corresponden al mismo contenedor reiniciado y el tercero al contenedor recreado tras `docker rm -f`.

### Paso 7: Ruta física del volumen en la VM

```bash
docker volume inspect asistencia-datos
```

* Campo `Mountpoint`: **`/var/lib/docker/volumes/asistencia-datos/_data`**.

---

## Ejercicio 2.2 — Construcción en varias etapas (*Multi-stage build*)

### Paso 1: Copiar el proyecto

```bash
mkdir -p ~/simulacro_examen/guia-ciclo
cp -r ~/materiales_simulacro/proyectos/guia-ciclo/* ~/simulacro_examen/guia-ciclo/
cd ~/simulacro_examen/guia-ciclo
```

### Paso 2: `Dockerfile` comentado (pégalo siempre en `respuestas.md`)

```dockerfile
# --- ETAPA 1: build (genera el sitio estático con Python) ---
FROM python:3.13-alpine AS build
WORKDIR /src
COPY . .
ARG VERSION=desarrollo
RUN python generar.py --version "${VERSION}" --salida /src/dist

# --- ETAPA 2: imagen final ligera (solo Nginx + HTML generado) ---
FROM nginx:alpine-slim
ARG VERSION=desarrollo
LABEL org.opencontainers.image.version="${VERSION}"
COPY --from=build /src/dist/ /usr/share/nginx/html/
EXPOSE 80
HEALTHCHECK --interval=10s --timeout=3s CMD wget -q --spider http://localhost/ || exit 1
```

* `AS build`: pone nombre a la primera etapa para poder referenciarla luego con `--from=build`.
* `ARG VERSION=desarrollo`: define un argumento de construcción con valor por defecto si no se pasa `--build-arg`.
* `COPY --from=build`: copia **únicamente** la carpeta generada `/src/dist/` desde la etapa 1 hacia la raíz web de Nginx.
* `EXPOSE 80`: documenta el puerto interno (no lo publica; sigue haciendo falta `-p`).
* `HEALTHCHECK`: cada 10s (`--interval`) comprueba con `wget --spider` (sin descargar el cuerpo) que `http://localhost/` responde en menos de 3s (`--timeout`).

### Paso 3: `.dockerignore`

```text
dist/
borradores/
README.md
Dockerfile
.dockerignore
```

* **Por qué es vital aquí:** impide que la carpeta `dist/` antigua (que contiene `antigua.html`) y `borradores/` entren en el contexto de construcción.

### Paso 4: Construir la imagen pasando el argumento `VERSION`

```bash
docker build --build-arg VERSION=2026.11.0 -t guia-ciclo:2026.11.0 .
```

* `--build-arg VERSION=2026.11.0`: inyecta el valor de `VERSION` en los `ARG` de ambas etapas.
* `-t guia-ciclo:2026.11.0`: asigna nombre y etiqueta a la imagen.
* `.`: indica que el contexto de construcción es la carpeta actual.

### Paso 5: Arrancar el contenedor y comprobar con `curl`

```bash
docker run -d --name guia -p 8085:80 guia-ciclo:2026.11.0
curl http://localhost:8085                # Muestra "Guía del ciclo versión 2026.11.0"
curl -I http://localhost:8085/antigua.html # Devuelve "HTTP/1.1 404 Not Found"
```

* `curl -I` pide solo las cabeceras HTTP para verificar rápidamente el código `404 Not Found`.

### Paso 6: Las 3 comprobaciones obligatorias con Docker

```bash
# 1. Confirmar que el contenedor aparece como (healthy) tras ~10s
docker ps

# 2. Filtrar la etiqueta OCI de versión usando -f y jq
docker image inspect -f '{{json .Config.Labels}}' guia-ciclo:2026.11.0 | jq '."org.opencontainers.image.version"'
# Salida esperada: "2026.11.0"

# 3. Demostrar que la imagen final NO contiene Python
docker run --rm guia-ciclo:2026.11.0 python3 --version
# Salida esperada: python3: not found
```

* **Truco de sintaxis `jq`:** cuando una clave JSON contiene puntos (como `org.opencontainers.image.version`), debe ir entre comillas dobles dentro de las comillas simples: `jq '."org.opencontainers.image.version"'`.

### Paso 7: Construir solo la etapa `build` y comparar tamaños

```bash
docker build --target build -t guia-ciclo:build .
docker image ls guia-ciclo
```

* `--target build`: detiene la construcción al finalizar la etapa llamada `build`.
* **Comparativa real obtenida en tu VM:**
  * `guia-ciclo:2026.11.0` (imagen final): **20.7 MB** en disco (`5.81 MB` de contenido).
  * `guia-ciclo:build` (etapa con Python): **71.5 MB** en disco (`17.4 MB` de contenido).
  * La etapa `build` pesa **más del triple** que la imagen final.

### Respuestas modelo pulidas para el 2.2

* **Ventaja de construir en varias etapas (*multi-stage build*):**
  La imagen final es mucho más ligera (pasa de 71,5 MB a 20,7 MB, más de tres veces menos), lo que acelera las descargas y los despliegues. Además, **reduce la superficie de ataque** al dejar fuera el intérprete de Python, herramientas de compilación y código fuente que no hacen falta en producción.
* **¿Por qué hay que declarar `ARG VERSION` en las dos etapas?**
  Porque cada instrucción `FROM` inicia una etapa nueva e independiente y el ámbito (*scope*) de `ARG` termina al cambiar de etapa; sin repetirlo tras el segundo `FROM`, `VERSION` quedaría vacía al evaluar `LABEL`.
* **¿Qué diferencia hay entre `ARG` y `ENV`?**
  * `ARG` solo existe **durante la construcción** (`docker build --build-arg`) y no llega al contenedor en ejecución.
  * `ENV` existe en **construcción y ejecución**, queda guardada en la imagen y puede sobrescribirse al arrancar con `docker run -e`.
  *(Ninguna de las dos debe usarse para contraseñas o secretos).*

---

## Ejercicio 2.3 — Docker Compose con proxy inverso

### Reglas de oro de sintaxis YAML (para no fallar en `compose.yaml`)

* **Solo espacios, nunca tabuladores:** usa siempre **2 espacios** por nivel de sangría.
* **Espacio obligatorio tras `:` y tras `-`:** escribe `clave: valor` y `  - elemento`.
* **Comillas cuando el valor contiene `:`:** escribe siempre `"8080:80"` en `ports:`.
* **Variables `.env`:**
  * `${VAR:?mensaje}` → obliga a que exista en `.env` (si falta, detiene Compose con ese error).
  * `${VAR:-defecto}` → usa un valor por defecto si no está definida.
  * `$$VAR` → pasa un `$` literal al contenedor (útil en `CMD-SHELL`).
* **Valida siempre antes de levantar:** `docker compose config` te indica la línea exacta de cualquier error de sangría.

### Paso 1: Copiar el proyecto y entrar en la carpeta

```bash
cp -r ~/materiales_simulacro/proyectos/contador-visitas ~/simulacro_examen/
cd ~/simulacro_examen/contador-visitas
ls -la
```

### Paso 2: Crear `.env` (pégalo en `respuestas.md`)

```env
REDIS_PASSWORD=clave-simulacro-2026
APP_VERSION=1.0
```

### Paso 3: Crear `compose.yaml` comentado (pégalo en `respuestas.md`)

```yaml
services:
  redis:
    image: redis:7-alpine
    command: ["redis-server", "--requirepass", "${REDIS_PASSWORD:?Falta REDIS_PASSWORD en .env}", "--appendonly", "yes"]
    volumes:
      - datos-redis:/data
    healthcheck:
      test: ["CMD-SHELL", "redis-cli -a ${REDIS_PASSWORD} --no-auth-warning ping | grep -q PONG"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - backend
    restart: unless-stopped

  web:
    build: ./app
    image: contador-visitas:1.0
    environment:
      REDIS_HOST: redis
      REDIS_PASSWORD: ${REDIS_PASSWORD:?Falta REDIS_PASSWORD en .env}
      APP_VERSION: ${APP_VERSION:-1.0}
    depends_on:
      redis:
        condition: service_healthy
    networks:
      - frontend
      - backend
    restart: unless-stopped

  proxy:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./proxy/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - web
    networks:
      - frontend
    restart: unless-stopped

volumes:
  datos-redis:

networks:
  frontend:
  backend:
```

* **Claves críticas:**
  * `command:` en `redis` activa la contraseña (`--requirepass`) y la persistencia en disco (`--appendonly yes` sobre `/data`).
  * `web` **no tiene `ports:`** porque solo responde internamente en la red `frontend` (y además poder escalar a 3 réplicas prohíbe fijar un puerto único en el host).
  * `depends_on` con `condition: service_healthy` hace que `web` espere a que el `healthcheck` de `redis` responda `PONG`.
  * El *bind mount* de Nginx lleva `:ro` (solo lectura) para proteger la configuración.
  * `proxy` solo está en `frontend` y `redis` solo en `backend` para aislar la base de datos.

### Paso 4: Comprobaciones en orden exacto

```bash
# 1. Validar YAML y ver variables sustituidas
docker compose config

# 2. Construir imagen y levantar servicios en segundo plano + verificar estado e imágenes
docker compose up -d --build
docker compose ps
docker compose images

# 3. Probar contador a través del proxy (8080) y confirmar que el puerto 5000 NO está expuesto
curl http://localhost:8080
curl http://localhost:8080
curl http://localhost:8080
curl http://localhost:5000    # Debe fallar con Connection refused

# 4. Demostrar aislamiento de redes desde proxy
docker compose exec proxy ping -c 1 web    # Funciona (comparten red frontend)
docker compose exec proxy ping -c 1 redis  # Falla: bad address 'redis' (redes distintas)

# 5. Probar persistencia (down conserva volúmenes; down -v los destruiría)
docker compose down
docker compose up -d
curl http://localhost:8080    # El contador continúa donde lo dejaste

# 6. Escalar web a 3 réplicas, reiniciar proxy y hacer 6 peticiones
docker compose up -d --scale web=3
docker compose restart proxy
for i in 1 2 3 4 5 6; do curl -s http://localhost:8080; echo ""; done
```

### Respuestas modelo pulidas para el 2.3

* **¿Por qué `curl http://localhost:5000` no funciona?**
  Porque el servicio `web` escucha en el puerto `5000` **solo dentro de la red interna de Docker** y no tiene ningún puerto publicado (`ports:`) hacia la VM. El único punto de entrada expuesto al exterior es el servicio `proxy` en el puerto `8080`, que reenvía las peticiones a `http://web:5000` por la red `frontend`.
* **Explicación de los `ping` desde `proxy` (aislamiento de redes):**
  * `ping -c 1 web` **funciona** porque `proxy` y `web` comparten la red `frontend`, y el DNS interno de Docker Compose resuelve el nombre `web` a su IP interna.
  * `ping -c 1 redis` **falla** (`bad address 'redis'`) porque `proxy` solo pertenece a `frontend` y `redis` solo pertenece a `backend`; al no compartir red, el DNS de `frontend` no conoce a `redis`, aislando la capa de datos.
* **Persistencia tras `docker compose down` y `up -d`:**
  El contador conserva su valor porque `docker compose down` borra contenedores y redes pero **conserva los volúmenes nombrados** (`contador-visitas_datos-redis`), donde Redis guarda sus datos en `/data` gracias a `--appendonly yes`. Solo con `docker compose down -v` se perderían los datos.
* **Escalado (`--scale web=3`): por qué cambia `servido por` y por qué comparten contador:**
  * **Por qué cambia `servido por`:** al escalar `web` a 3 réplicas y reiniciar `proxy`, el DNS interno de Docker asocia el nombre `web` a las 3 IPs de las réplicas y reparte las conexiones entre ellas (balanceo *round-robin*), por lo que cada petición puede ser atendida por un contenedor (ID) distinto.
  * **Por qué comparten contador:** porque `web` no guarda el estado en memoria local, sino en un **único servicio centralizado (`redis`)** al que acceden las 3 réplicas por la red `backend`.

---

## 📌 Único fleco pendiente

- [ ] **Ejercicio 1.1 d):** `git commit --amend`, clon de compañera, error de `push` y resolución de conflicto con `git pull --no-rebase`.
- [ ] **Actualizar `kit-examen:1.3`:** sustituir este archivo `.md` actualizado en `kit-examen/apuntes/` y subir la versión `1.3` a Docker Hub.
