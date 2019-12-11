---
layout: post
title: Uso básico Git en Visual Studio
---

A raíz de mi anterior post, me gustaría tratar el uso de Git en el IDE que más utilizo como desarrollador .NET: <a href="https://visualstudio.microsoft.com/es/" target="_blank">Visual Studio</a>. Suelo recomendar el uso de un IDE para trabajar con Git, no sin antes haber entendido los conceptos que subyacen en las acciones que realiza el IDE (comentados en mi <a href="https://dielop101.github.io/CheatSheet-Git-Basic/" target="_blank">anterior post</a>).

Visual Studio proporciona una serie de herramientas para poder gestionar repositorios Git. No considero que sea una herramienta que proporcione todas las capacidades que ofrece Git (como podría ser <a href="https://gitextensions.github.io/" target="_blank">GitExtensions</a> o <a href="https://www.sourcetreeapp.com/" target="_blank">SourceTree</a>, pero para un uso cotidiano de Git es más que suficiente.

<div align="center">
  <img src="/images/git-vs.png"/>
</div>

Para los ejemplos, utilizaré <b>Visual Studio 2019</b>.

## Preparando entorno
Como vimos anteriormente, existen dos formas para iniciar un repositorio:

**Init** - Tras crear un proyecto nuevo, tendremos la opción de añadirlo al control de versiones "Add to Source Control". Tras seleccionar que queremos usar Git, Visual Studio generará todo lo necesario para tener el repositorio creado con la rama master por defecto.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/init-01.png"/>
  <i>Comando init aplicado desde visual studio</i>
</div>
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/init-02.png"/>
  <i>Estado final tras iniciar un repositorio</i>
</div>

**Clone** - En caso de querer clonar un repositorio existente, simplemente tendremos que seleccionar la opción de "Manage Connections" y pulsar en la opción "Clone". En el primer input, introduciremos el repositorio remoto. En el segundo input, dónde queremos crear en local el repositorio.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/clone-01.png"/>
  <i>Clonación de un repositorio existente</i>
</div>

## Iniciando desarrollo
De nuevo, siguiendo las acciones descritas en el post anterior:
**Status** - El estado lo tenemos siempre presente desde Visual Studio, en la barra inferior de la aplicación. De izquierda a derecha, los iconos inferiores indican: commits pendientes de realizar push, cambios pendientes que no se han llevado a un commit, repositorio actual y rama actual.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/status-01.png"/>
  <div><i>Barra inferior del Visual Studio</i></div>
</div>

**Branch -a** - El listado de ramas la podemos obtener desde la opción "Branches", accesible desde la "Home".
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/branch-01.png"/>
  <div><i>Team Explorer - Branches</i></div>
</div>

**Branch [branch name]** - Para crear una nueva rama, desde la vista anterior seleccionamos la opción "New Branch". En el primer input introduciremos el nombre de la nueva rama, mientras que en el segundo input seleccionaremos la rama de la cual se va a obtener la rama. El checkbox nos indicará si queremos movernos a la nueva rama tras su creación.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/branch-02.png"/>
  <div><i>Branches - formulario para crear una nueva rama</i></div>
</div>

**Checkout** - Desde la vista de "Branches" hacemos doble click a la rama que queremos cambiar. Se marcará en negro la rama en la cual estés actualmente.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/Checkout-01.png"/>
  <div><i>Haciendo doble click cambia a la rama dev </i></div>
</div>

## Actualizar rama
**Stash** - Accedemos desde la "Home", en "Changes" para entrar a la gestión de los cambios en la rama. Una vez dentro, veremos la opción de "stash". Prodecemos a apilar todo, lo que generará un elemento en la pila y no tener cambios pendientes.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/stash-01.png"/>
  <div><i>Acceso a "Changes"</i></div>
</div>
<br>
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/stash-02.png"/>
  <div><i>Apilando cambios</i></div>
</div>
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/stash-03.png"/>
  <div><i>Cambios apilados. No quedan cambios pendientes</i></div>
</div>
**Fetch** - Para ver los cambios que ha sufrido nuestra rama en el repositorio remoto, desde el "Team Explorer - Home" pulsamos en el botón "Sync" y nos llevará a las opciones de sincronización. Pulsamos en "Fetch" y nos traerá los commits que no tengamos en nuestro local.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/fetch-01.png"/>
  <div><i>Opción "Sync"</i></div>
</div>
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/fetch-02.png"/>
  <div><i>Acción "Fetch"</i></div>
</div>

**Pull** - Una vez realizado el fetch, pulsando sobre "Pull" bajará los cambios en nuestra rama local.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/pull-01.png"/>
  <div><i>Acción "Pull"</i></div>
</div>


**Stash pop** - Teniendo el contenido anteriormente mencionado apilado, procedemos a desapilarlo con la opción "Pop", seleccionando con el botón derecho en el elemento a desapilar. Dependiendo nuestra necesidad, elegiremos dejarlo o no en el stage. En nuestro caso, no será necesario.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/stash-04.png"/>
  <div><i>Opciones para desapilar</i></div>
</div>
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/stash-05.png"/>
  <div><i>No queda nada en la pila. Se restauran los cambios pendientes</i></div>
</div>

## Subir cambios
**Add** - Seleccionamos los ficheros que queremos insertar en el stage. Con botón derecho sobre los mismos, nos aparecerá la opción de "Stage". Lo que quede dentro del stage serán los ficheros que se empaquetarán en el commit que se cree. El resto, quedarán fuera del commit.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/add-01.png"/>
  <div><i>Opción "Stage"</i></div>
</div>
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/add-02.png"/>
  <div><i>Sección "Staged Changes"</i></div>
</div>

**Commit** - Hilando con el comando anterior, rellenando el input con el mensaje que queramos reflejar, tendremos el commit creado pulsando "Commit Staged". Como indica el comentario, sólo se hará commit de los ficheros que estén bajo "Staged Changes". En caso de no haber creado "Staged Changes", se realizaría commit de todos los cambios que se hubiesen realizado en la rama.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/commit-01.png"/>
  <div><i>Acción "Commit Staged"</i></div>
</div>
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/commit-02.png"/>
  <div><i>Listado de commits pendientes de realizar push</i></div>
</div>

**Push** - Por último, en la parte superior del listado de commits, tenemos la opción de realizar el "Push". Esto realizará la sincronización de los cambios locales al repositorio remoto.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/push-01.png"/>
  <div><i>Acción "Push"</i></div>
</div>


## Finalizando desarrollo
**Merge** - Para realizar un merge entre dos ramas, desde la sección de "Branches" tendremos la opción de "Merge". Una vez pulsada, nos preseleccionará la rama actual como rama destino del merge que se realice. En "Merge from branch" seleccionaremos la rama "dev". En este caso, dado que estamos en master, el merge se hará de dev a master. Con la opción "Commit changes after merging", en caso de no haber conflictos, se realizará un commit con el merge automático.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/GitVisualStudio/merge-01.png"/>
  <div><i>Acción "Merge"</i></div>
</div>
## Conclusiones
Los IDEs se contruyeron para ayudar en el desarrollo software, pero como programadores nuestro deber va más hayá del mero uso. Tenemos que ser conscientes y capaces de saber qué está realizando por detrás y sus consecuencias. De nuevo, en este post he tratado de resumir los comandos más usados en mi día a día, habiendo mucho más recorrido por explorar en la  propia herramienta de Visual Studio. Cualquier duda o sugerencia, estoy disponible en mis redes sociales :)