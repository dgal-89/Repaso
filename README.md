# Apuntes de Implantación Web

Resumen de los comandos vistos en clase: Git, GitHub, Docker Compose, Jekyll y GitHub Pages.

Donde pone `URL`, `rama`, `archivo` o `"mensaje"`, hay que poner el valor de cada caso.

---

## Terminal

- `mkdir carpeta` → crea una carpeta
- `cd carpeta` → entra en ella
- `cd ..` → vuelve a la carpeta anterior
- `ls -la` → muestra todo lo que hay, también los archivos ocultos (como `.git`)

Cuando estás dentro de un repositorio, la terminal muestra la rama entre paréntesis, por ejemplo `(main)`.

---

## Git en local

Antes del primer commit hay que configurar el nombre y el email del autor:

```bash
git config --global user.name "Nombre"
git config --global user.email "correo@ejemplo.com"
```

Comandos principales:

- `git init` → convierte la carpeta en un repositorio (crea `.git`)
- `git add archivo` → prepara un archivo para el commit
- `git add .` → prepara todos los archivos
- `git status` → muestra el estado de los archivos (rojo = sin preparar, verde = preparado)
- `git commit -m "mensaje"` → guarda una "foto" de lo preparado
- `git log` → muestra el historial de commits (se sale con `q`)
- `git diff` → muestra los cambios línea a línea (`-` lo que había, `+` lo nuevo)

Para quitar archivos del área de preparación sin borrarlos:

- `git rm --cached archivo` → quita un archivo
- `git rm --cached . -r` → quita todos

Recorrido de los cambios:

```text
archivos --add--> preparados --commit--> historial --push--> GitHub
```

---

## Ramas

- `git branch` → lista las ramas (la actual lleva `*`)
- `git switch -c rama` → crea una rama y se cambia a ella
- `git checkout -b rama` → lo mismo, con el comando antiguo
- `git switch main` → cambia a la rama main
- `git merge rama` → trae los cambios de esa rama a la rama actual
- `git branch -M main` → renombra la rama actual a `main`

Para incorporar una rama a main, primero hay que situarse en main:

```bash
git switch main
git merge rama-uno
```

---

## Conflictos

Aparecen cuando dos ramas modifican la misma línea:

```text
<<<<<<< HEAD
contenido de la rama actual
=======
contenido de la otra rama
>>>>>>> rama-dos
```

Opciones de VS Code:

- **Accept Current Change** → se queda con el contenido de la rama actual
- **Accept Incoming Change** → se queda con el contenido de la otra rama
- **Accept Both Changes** → se queda con los dos

Una vez elegido, se guarda el archivo y se cierra el conflicto:

```bash
git add README.md
git commit -m "Conflicto resuelto"
```

---

## GitHub

- `git clone URL` → descarga un repositorio (ya viene enlazado con `origin`)
- `git remote add origin URL` → enlaza el repositorio local con GitHub
- `git remote -v` → muestra a qué repositorio está enlazado
- `git push -u origin main` → sube los commits (con `-u`, las siguientes veces basta con `git push`)
- `git pull` → descarga los cambios de GitHub

Subir un proyecto nuevo a un repositorio vacío de GitHub:

```bash
git init
git add .
git commit -m "Primer commit"
git branch -M main
git remote add origin URL
git push -u origin main
```

---

## Pull Requests

Una Pull Request sirve para proponer cambios de una rama a otra y que se revisen antes de unirlos.

```bash
git clone URL
git switch -c mejora-readme
# se modifica el archivo
git add README.md
git commit -m "Mejora del README"
git push -u origin mejora-readme
```

Después, en GitHub:

1. **Compare & pull request**
2. base: **main** ← compare: **mejora-readme**
3. Título, descripción y **Create pull request**
4. Revisión y comentarios
5. **Merge pull request**

---

## Docker Compose

- `docker compose config --quiet` → comprueba que el YAML está bien (si no muestra nada, es correcto)
- `docker compose up -d` → arranca el servicio en segundo plano
- `docker compose logs -f` → muestra lo que va pasando en el contenedor
- `docker compose restart` → reinicia el servicio
- `docker compose down` → para y elimina el contenedor

Es necesario tener Docker Desktop abierto. El blog se ve en http://localhost:4000

Archivo `docker-compose.yml`:

```yaml
services:
  jekyll:
    image: ruby:3.3-bookworm
    working_dir: /proyecto
    user: "${JEKYLL_UID:-1000}:${JEKYLL_GID:-1000}"
    environment:
      BUNDLE_USER_HOME: /tmp/jekyll-bundler
    volumes:
      - .:/proyecto
      - gemas:/usr/local/bundle
    ports:
      - "127.0.0.1:4000:4000"
      - "127.0.0.1:35729:35729"
    command:
      - sh
      - -ec
      - |
        if [ ! -f docs/Gemfile ]; then
          gem install jekyll --version 3.10.0 --no-document
          jekyll _3.10.0_ new docs --skip-bundle
        fi
        cd docs
        bundle check || bundle install
        exec bundle exec jekyll serve --host 0.0.0.0 --port 4000 \
          --watch --force_polling --livereload --livereload-port 35729

volumes:
  gemas:
```

Partes principales:

- `ruby:3.3-bookworm` → imagen oficial de Ruby; lo que va después de `:` es la etiqueta
- `.:/proyecto` → la carpeta del proyecto se comparte con el contenedor
- `gemas` → volumen donde se guardan las gemas para no reinstalarlas
- `127.0.0.1:4000:4000` → IP : puerto del equipo : puerto del contenedor

---

## Jekyll

Contenido de la carpeta `docs/`:

- `_config.yml` → configuración del sitio (título, tema, url…)
- `_posts/` → entradas del blog
- `_site/` → web generada (se crea automáticamente)
- `index.md` → portada

Las entradas van en `_posts/` con el formato `AAAA-MM-DD-titulo.md`, por ejemplo `2026-11-17-mi-primer-post.md`:

```markdown
---
layout: post
title: "Mi primer post"
---

Contenido de la entrada.
```

---

## GitHub Pages

Archivo `.gitignore`, con lo que no se sube al repositorio:

```text
/docs/_site/
/docs/.jekyll-cache/
/docs/.jekyll-metadata/
/docs/.sass-cache/
/docs/Gemfile.lock
```

Activación: **Settings → Pages → Deploy from a branch → main → /docs → Save**

Rutas en `docs/_config.yml`:

```yaml
url: "https://USUARIO.github.io"   # dominio
baseurl: "/blog-iaw"               # nombre del repositorio
```
