---
layout: post
title: "Repaso Comandos Git y Docker"
---
En clase usamos Git para guardar versiones del proyecto, GitHub para revisar y compartir los cambios y Docker Compose para probar el blog de Jekyll en local.

1. Empezar un repositorio

Si empiezo con una carpeta nueva, abro Git Bash dentro de ella. Con estos comandos preparo el primer archivo y guardo su primera versión. Sustituyo el nombre y el correo por los que quiero que aparezcan en mis commits.

git init
ls -la
git config --global user.name "Tu nombre"
git config --global user.email "tu-correo@example.com"
git config user.name
git config user.email
echo "# gitprueba-iaw" >> README.md
git add README.md
git commit -m "first commit"

ls -la muestra también la carpeta oculta .git, donde Git guarda el historial. Las dos consultas de git config muestran la identidad que Git utilizará en este repositorio. El operador >> añade el título al final del README y crea el archivo si hace falta.

Conectar con GitHub o clonar un proyecto

Para subir el proyecto local, primero creo un repositorio vacío en GitHub. En el siguiente ejemplo sustituyo TU-USUARIO y TU-REPOSITORIO por mis datos.

git branch -M main
git remote add origin https://github.com/TU-USUARIO/TU-REPOSITORIO.git
git remote -v
git push -u origin main

El primer comando renombra la rama actual a main. El segundo guarda la dirección de GitHub con el nombre origin. Con git remote -v compruebo las direcciones de descarga y subida. La opción -u vincula mi rama local con la remota para poder usar git push y git pull en adelante.

Si el repositorio ya existe en GitHub y quiero trabajar con él, empiezo desde la carpeta donde quiero descargarlo. clone crea la copia local y configura origin automáticamente.

git clone https://github.com/TU-USUARIO/gitpr-iaw.git
cd gitpr-iaw
git status

2. Revisar cambios y elegir qué entra en el commit

Comando

Para qué lo uso

git status

Ver la rama actual, los archivos nuevos y qué cambios están preparados.

git diff

Leer los cambios de archivos en seguimiento que aún tengo pendientes de preparar.

git add index.md

Preparar la versión actual de un archivo.

git add fichero1.md fichero2.md

Preparar solo los archivos indicados.

git add .

Preparar todos los cambios de la carpeta actual y sus subcarpetas.

git diff --cached

Revisar los cambios preparados para el siguiente commit.

git rm --cached index.md

Quitar el archivo del índice y conservar su copia en el disco. Si ya estaba en un commit, queda preparada su retirada del repositorio.

git rm -r --cached .

Aplicar esa retirada a toda la carpeta de forma recursiva. En los apuntes aparece como git rm --cached . -r.

git commit -m "added index.md"

Guardar lo que esté preparado con un mensaje. Si antes retiré index.md del índice, vuelvo a ejecutar git add index.md para incluirlo.

git log

Consultar el identificador, el autor, la fecha y el mensaje de cada commit.

git log --oneline

Ver el historial resumido en una línea por commit.

Los apuntes también mencionan git stash. Sirve para guardar temporalmente cambios en archivos en seguimiento si necesito cambiar de tarea. Con git stash pop puedo recuperarlos.

3. Trabajar con ramas y resolver un conflicto

En el ejercicio, dos ramas modifican la misma línea del README. Creo ambas desde main para que cada una tenga su propia versión del cambio.

git branch
git switch -c rama-uno
# Edito la línea 2 de README.md y guardo el archivo.
git add README.md
git commit -m "cambios rama-uno"

git switch main
git switch -c rama-dos
# Escribo algo distinto en la misma línea y guardo.
git add README.md
git commit -m "cambios de rama dos"

git switch main
git merge rama-uno
git merge rama-dos

git branch marca con un asterisco la rama activa. En los apuntes también usamos git checkout -b rama-uno y git checkout -b rama-dos como alternativas a git switch -c. Para volver a la principal puedo usar git checkout main o git switch main.

Si el segundo merge da un conflicto, abro el README. En este ejercicio, Accept Current Change conserva la línea de main que llegó de rama-uno, y Accept Incoming Change conserva la de rama-dos. También puedo escribir una versión que combine ambas. Guardo el resultado sin marcas de conflicto y termino la fusión.

git add README.md
git commit -m "Resuelve el conflicto en README"
git status

4. Proponer una mejora con una pull request

Desde main, creo una rama, modifico el README y envío la propuesta a GitHub. También puedo crear esa rama con git checkout -b mejora-readme.

git switch -c mejora-readme
# Edito README.md y guardo los cambios.
git diff
git add README.md
git commit -m "cambios realizados en el archivo"
git log
git push -u origin mejora-readme

En GitHub abro una pull request desde mejora-readme hacia main. Si me piden una corrección, hago otro commit en esa misma rama y ejecuto git push. Cuando se aprueba la propuesta, Merge pull request integra los cambios. Después actualizo mi copia local.

git switch main
git pull

5. Arrancar y comprobar el blog con Docker Compose

Primero creo la carpeta de trabajo y entro en ella. Aquí guardo el archivo docker-compose.yml de la práctica. Si ya tengo el proyecto, abro su carpeta directamente.

mkdir blog-iaw
cd blog-iaw

Con Docker Desktop en marcha, ejecuto los comandos de Compose desde la carpeta que contiene el YAML.

Comando

Qué compruebo o consigo

docker compose config --quiet

Valida la configuración. Si es correcta, termina sin mostrar nada. Es la sintaxis corregida del comando de validación de los apuntes.

docker image ls

Muestra las imágenes descargadas. La práctica utiliza ruby:3.3-bookworm.

docker compose up -d

Crea o arranca los servicios en segundo plano. En el primer arranque, el script de la práctica genera el blog en docs/.

docker compose ps

Permite comprobar el estado de Jekyll y el puerto publicado.

docker compose logs -f jekyll

Muestra los mensajes de Jekyll en directo. Con Ctrl+C salgo de los mensajes y el servicio sigue en marcha.

docker ps -a

Lista todos los contenedores del equipo, incluidos los detenidos.

docker compose down

Detiene y elimina los contenedores del proyecto al terminar. La carpeta local docs/ se conserva.

Qué ejecuta el contenedor por dentro

El archivo de Compose ejecuta este bloque con sh -ec. La opción -c hace que el intérprete lea el bloque como un script y -e hace que termine si falla un paso necesario. Estos comandos ya se ejecutan al arrancar el servicio.

if [ ! -f docs/Gemfile ]; then
  gem install jekyll --version 3.10.0 --no-document
  jekyll _3.10.0_ new docs --skip-bundle
fi

cd docs
bundle check || bundle install
exec bundle exec jekyll serve --host 0.0.0.0 --port 4000 \
  --watch --force_polling --livereload --livereload-port 35729

[ ! -f docs/Gemfile ] comprueba si falta el archivo de dependencias. En ese caso, gem install instala Jekyll 3.10.0 sin generar su documentación y jekyll ... new docs --skip-bundle crea el sitio dejando la instalación de dependencias para el siguiente paso.

cd docs entra en el sitio. bundle check comprueba sus dependencias y || bundle install las instala si la comprobación falla.

exec bundle exec jekyll serve arranca Jekyll con las versiones del proyecto como proceso principal del contenedor. Escucha en el puerto 4000. Las opciones --watch y --force_polling detectan cambios en los archivos, y --livereload actualiza el navegador mediante el puerto 35729.

6. Comprobar y publicar esta entrada

Para que Jekyll reconozca este archivo como entrada, lo guardo en docs/_posts/2026-11-24-repaso-git-y-docker.md, con el título y layout: post en la cabecera. Abro la portada local y compruebo que puedo entrar al repaso. Si el proyecto tiene configurado baseurl: /blog-iaw, uso la ruta local del blog.

Desde la raíz del repositorio, en la rama main, publico la entrada con estos comandos.

git status
git add docs/_posts/2026-11-24-repaso-git-y-docker.md
git diff --cached
git commit -m "Añade el repaso de comandos de Git y Docker"
git push origin main

Para la entrega, el repositorio debe ser público. Si publico también el blog en GitHub Pages como en la práctica, selecciono la rama main y la carpeta /docs en la configuración de Pages. En docs/_config.yml, url contiene el dominio del sitio y baseurl la ruta del repositorio, por ejemplo /blog-iaw.