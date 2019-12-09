---
layout: post
title: Consejos comandos para git básico
---

De todos es conocido que Git es el sistema de control de versiones más conocido y usado en el mundo. Aquí os dejo un pequeño <i>cheatsheet</i> de los comandos que personalmente más suelo usar. ¡Espero que os sirvan!

<div align="center">
  <img src="https://dielop101.github.io/images/gitCommands.png"/>
</div>

## Preparando entorno
Podemos utilizar 2 comandos para iniciar nuestro entorno:

**Init** - crea un nuevo repositorio con la rama master por defecto.
```git
$ git init
    Initialized empty Git repository in C:/myrepository/.git/
```
**Clone** - partiendo de un repositorio ya existente, clona y sincroniza un repositorio remoto en tu entorno local. Por defecto, te situará en la rama master. En el ejemplo, clonaremos el repositorio vacío "mydummyrepo" de github a nuestro local
```git
$ git clone https://github.com/dielop101/mydummyrepo.git
    Cloning into 'mydummyrepo'...
    warning: You appear to have cloned an empty repository.
```
## Iniciando desarrollo
Una vez ya tenemos el repositorio creado, necesitaremos saber el estado actual, ramas existentes y cómo poder crear una rama para empezar nuestro desarrollo.

**Status** - Devuelve el estado actual del repositorio. Sabremos identificar rama actual, si está actualizada frente al repositorio remoto y si tenemos algo pendiende de subir.
```git
$ git status
    On branch master
    Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
      (use "git pull" to update your local branch)

    nothing to commit, working tree clean
```
**Branch -a** - Lista las ramas tanto locales como remotas, indicando la rama en la cual te encuentras actualmente mediante un asterisco.
```git
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```
**Branch [branch name]** - Dado que estamos en rama master, crearemos una rama que dependa de master. Vamos a crear dev:
```git
$ git branch dev
```
**Checkout** - Permite cambiar de rama, en el caso de que se permita. Si detecta ficheros modificados, no te permitirá el cambio (tendrás que hacer commit o stash para no tener contenido pendiente de subir).
```git
$ git checkout dev
    Switched to branch 'dev'
```
**Checkout -b [branch name]** - Crea rama y cambia a dicha rama (es decir, hace un branch + checkout)
```git
$ git checkout -b dev
    Switched to a new branch 'dev'
```
## Actualizar rama
Parece que alguien ha actualizado master, creando un fichero "test3". Vamos a sincronizar nuestra rama master para que, si en el futuro volvemos a crear una rama a partir de master, tengamos los últimos cambios.

**Stash** - En caso de tener cambios pendientes, los apilaremos para poder cambiar de rama. En el siguiente ejemplo, tenemos modificado el fichero "test2" y apilaremos sus cambios. Tras apilar los cambios, podemos entrar de nuevo al fichero y veremos que ya no contiene los cambios.
```git
$ git stash
    Saved working directory and index state WIP on dev: 293160a Create test2
```
**Fetch** - Sincroniza los cambios que ha sufrido la rama. No realiza ninguna modificación en los ficheros de la rama actual, simplemente deja en "standby" los cambios a la espera de realizar el pull. Podemos ahorrar este comando directamente usando el pull.
```git
$ git fetch
    remote: Enumerating objects: 4, done.
    remote: Counting objects: 100% (4/4), done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
    Unpacking objects: 100% (3/3), done.
    From https://github.com/dielop101/mydummyrepo
       293160a..37662be  master     -> origin/master
```
**Pull** - Actualiza la rama local con los cambios que existan en la rama del repositorio remoto. Cambiaremos de rama a master y bajaremos los cambios existentes.
```git
$ git pull
    Updating 293160a..37662be
    Fast-forward
     test3 | 1 +
     1 file changed, 1 insertion(+)
     create mode 100644 test3
```
**Stash pop** - Desapilamos el contenido anteriormente apilado.
```git
$ git stash pop
    On branch dev
    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working directory)

            modified:   test2

    no changes added to commit (use "git add" and/or "git commit -a")
    Dropped refs/stash@{0} (0414e783e55f3950aecca0d6f3d9ca8a53e2e567)
```

## Subir cambios
Vamos a subir los cambios que hemos realizado en el fichero "test2".

**Add** - Identificamos los ficheros que queremos de los cuales queremos hacer commit. Dado que queremos hacer commit de todo lo existente, lo indicaremos mediante el carácter punto
```git
$ git add .
```

**Commit** - Preparamos el commit que se añadirá a la rama. Definimos un mensaje explicando el cambio realizado
```git
$ git commit -m "subiendo modificación fichero test2"
    [dev c6ce524] subiendo modificación fichero test2
     1 file changed, 1 insertion(+), 1 deletion(-)
```
**Push** - Sincronizamos los cambios en el respositorio remoto. Esto implica llevar todos los commits que tengamos pendientes en la rama al repositorio remoto. Dado que no existe la rama en el servidor, tendremos que indicarlo en el comando
```git
$ git push origin dev
    Enumerating objects: 5, done.
    Counting objects: 100% (5/5), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (3/3), 299 bytes | 299.00 KiB/s, done.
    Total 3 (delta 0), reused 0 (delta 0)
    remote:
    remote: Create a pull request for 'dev' on GitHub by visiting:
    remote:      https://github.com/dielop101/mydummyrepo/pull/new/dev
    remote:
    To https://github.com/dielop101/mydummyrepo.git
     * [new branch]      dev -> dev
```

## Finalizando desarrollo 
Una vez finalizado el desarrollo, procedemos a mergear los cambios de dev en la rama master. Para ello, cambiaremos de rama a master. Recuerda que, una vez mergeado el contenido de dev en master, tendrás que realizar el push al repositorio, dado que el merge se ha realizado en local.

**Merge** - Combina los cambios de una rama origen a una rama destino. En este caso, mergearemos dev en master.
```git
$ git merge dev
    Merge made by the 'recursive' strategy.
     test2 | 2 +-
     1 file changed, 1 insertion(+), 1 deletion(-)
```
## Conclusiones
Git tiene infinidad de comandos y de parámetros. Con este post, trato de documentar los conceptos y los comandos que considero más importantes, dado que son los que más uso en mi día a día. Cualquier duda o sugerencia, estoy disponible en mis redes sociales :)