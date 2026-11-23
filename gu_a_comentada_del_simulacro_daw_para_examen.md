# Guía Comentada del Simulacro T1 — DAW (Apuntes de Examen)

Este documento reúne todo lo trabajado hoy en tu simulacro (**Ejercicios 1.1 a/b/c, 2.1 y 2.2**), con **cada comando explicado parámetro a parámetro**, las **respuestas teóricas listas y pulidas**, y los **avisos de errores reales** que aparecieron en tus capturas para no repetirlos el día del examen.

---

## ⚠️ Checklist anti-despistes (errores de hoy a evitar en el examen)

Antes de entregar tu `respuestas.md` real, revisa estos puntos que salieron en tus pruebas:

1. **Sustituye siempre los marcadores de ejemplo:** nunca escribas literalmente `HASH` ni `2026.11.X`; cámbialos por el hash real (ej. `54849aa`) o la versión real (`2026.11.0`).
2. **Entra en la carpeta del repo antes de `git config`:** ejecutar `git config` fuera de la carpeta clonada da `fatal: not in a git directory`.
3. **Usa `--local` y el formato pedido por el enunciado:** pon tu nombre completo, el correo `TU_USUARIO@simulacro.local` y muestra solo la config local con `git config --local --list` (no `git config -l`, que mezcla la global).
4. **No crees ramas con `origin/HEAD`:** al hacer `git branch -r`, ignora siempre `origin/HEAD` y `origin/main`.
5. **No dejes frases de guía pegadas en la entrega:** borra textos como *"Qué poner en respuestas.md:"* o *"Son los que te salgan en el paso 4"*; escribe los hashes y nombres concretos.
6. **Incluye todos los bloques de código que pida el enunciado:** hoy faltó pegar el contenido de `.gitignore`, `CHANGELOG.md` y el `Dockerfile` del 2.2 como bloques de código en `respuestas.md`.
7. **Numeración exacta:** usa los mismos números que el PDF (`1.1 a)`, `1.1 b)`, `1.1 c)`...) para facilitar la corrección.

---

# Bloque 1: Git y GitHub Flow (UD1)

> **Regla de oro del enunciado:** usa siempre `git merge --no-ff` al integrar una rama en `main` (simula una *Pull Request* aprobada dejando constancia del *merge commit* en el historial).

---

## Ejercicio 1.1 a) — Clonado y configuración local

### Comandos correctos paso a paso

```bash
# 1. Clonar el repositorio remoto (ORIGEN -> DESTINO) y entrar en él
mkdir -p ~/simulacro_examen
git clone ~/materiales_simulacro/repos/biblioteca-web.git ~/simulacro_examen/biblioteca-web
cd ~/simulacro_examen/biblioteca-web
```
* **Nota de hoy:** `git clone` siempre recibe primero el **origen** (el repo `.git`) y opcionalmente el **destino**. Si pones primero el destino vacío, Git dirá `repository ... does not exist`.

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
* **Corrección respecto a tu PDF:** usaste `git config -l`, que sacó también la configuración global (`http.proxy`, `gavkops@gmail.com`). Usa `--local --list` para cumplir exactamente lo pedido.

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
* **Alternativa válida (la que usaste):** `git checkout -b feature/novedades origin/feature/novedades`. Funciona igual, pero además te cambia a esa rama; con `git branch --track` te quedas en `main` sin moverte.
* **Tip si creas una rama por error** (como te pasó con `origin/HEAD`): se borra con `git branch -D <nombre>` (no existe `git branch rm`).

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

> ⚠️ **En tu PDF te saltaste este comando; asegúrate de ejecutarlo en el examen.**

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

---

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

4. **Ramas remotas: cuál puede borrarse y cuál no debe integrarse** *(faltaba en tu PDF)*
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
  ```gitignore
  .env
  ```
* **¿Ha dejado de ser accesible la clave de `.env`? ¿Qué debe hacer el equipo?**
  **No.** El archivo sigue guardado en los commits anteriores del historial (cualquiera puede verlo con `git show 156a2ce`). El equipo debe **revocar y rotar inmediatamente esa clave** por una nueva que nunca haya pasado por Git (y opcionalmente purgar el historial con `git filter-repo`, aunque rotar la contraseña es obligatorio siempre).

### Paso 3: Actualizar `CHANGELOG.md` *(faltó en tu práctica de hoy)*

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
* **Truco de rescate (por si te equivocas en el nombre de una etiqueta, como `2026.11.X`):**
  ```bash
  git tag -d 2026.11.X                  # Borrar etiqueta en local
  git push origin --delete 2026.11.X    # Borrar etiqueta en remoto
  ```

---

> 📌 **Pendiente del Bloque 1:** recuerda practicar mañana el **1.1 d)** (enmienda con `git commit --amend`, segundo clon de la compañera y resolución de conflicto con `git pull --no-rebase`) y adjuntar el **`README.md` del 1.2** usando la plantilla reutilizable.

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

---

### Respuestas modelo pulidas para el 2.2

* **Ventaja de construir en varias etapas (*multi-stage build*):**
  La imagen final es mucho más ligera (en este caso pasa de 71,5 MB a 20,7 MB, más de tres veces menos), lo que acelera las descargas y los despliegues. Además, **reduce la superficie de ataque** al dejar fuera el intérprete de Python, herramientas de compilación y código fuente que no hacen falta en producción.
* **¿Por qué hay que declarar `ARG VERSION` en las dos etapas?**
  Porque cada instrucción `FROM` inicia una etapa nueva e independiente y el ámbito (*scope*) de `ARG` termina al cambiar de etapa; sin repetirlo tras el segundo `FROM`, `VERSION` quedaría vacía al evaluar `LABEL`.
* **¿Qué diferencia hay entre `ARG` y `ENV`?**
  * `ARG` solo existe **durante la construcción** (`docker build --build-arg`) y no llega al contenedor en ejecución.
  * `ENV` existe en **construcción y ejecución**, queda guardada en la imagen y puede sobrescribirse al arrancar con `docker run -e`.
  *(Ninguna de las dos debe usarse para contraseñas o secretos).*

---

## 📌 Pendiente para mañana (sesión 2)
- [ ] **Ejercicio 1.1 d):** `git commit --amend`, clon de compañera, error de `push` y resolución de conflicto con `git pull --no-rebase`.
- [ ] **Ejercicio 2.3:** Docker Compose con `redis`, `web` (Flask) y `proxy` (Nginx), redes aisladas (`frontend`/`backend`), `healthcheck`, persistencia y escalado (`--scale web=3`).
- [ ] **Ejercicio 1.2:** Captura final para el `README.md` y comprobación en vista previa de VS Code.