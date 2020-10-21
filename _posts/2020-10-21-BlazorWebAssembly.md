---
layout: post
title: ¡Adiós Javascript, hola Blazor!
---

¿Y si te digo que existe un framework capaz de alcanzar toda la potencia de un SPA moderno, pero sin entrar en contacto como desarrollador con Javascript? Pues sí, como habrás intuido, esto es lo que nos ofrece Blazor. ¿Quiere decir entonces que ya podremos olvidar de Javascript? Bueno, pero vayamos por partes.

<div align="center">
  <img src="/images/blazor/blazor.jpg"/>
</div>

## ¿Qué es Blazor?
Blazor es un proyecto de Microsoft que nace bajo la necesidad de proveer un framework que permita desarrollar Single Page Applications (SPA) para los desarrolladores .NET, usando como lenguajes de programación C# y Razor.

El objetivo de Microsoft está claro: competir en el mundo de los SPA a través de un framework con una curva de aprendizaje prácticamente nula para los desarrolladores .NET, abstrayendo la complejidad que requiere lidiar con frameworks basados en Javascript. En consecuencia, se construirán aplicaciones web enriquecidas usando únicamente HTML, CSS y C# en lugar de Javascript. El sueño del desarrollador .Net.

¿Pero, cómo es posible esto? Primero vamos a ver los diferentes modelos que nos ofrece Blazor:

### Modelos de hospedaje de Blazor
Es conveniente conocer los dos grandes enfoques que presenta Blazor:

<ul>
	<li> Blazor Server: modelo más tradicional, en el cual se prepara en el servidor la interfaz a enviar al cliente. Permite una interacción sencilla entre cliente y servidor a través de SignalR (socket que permanecerá abierto entre el cliente y el servidor para la transmisión de información). Se enfoca sobre todo como sustituto de los Web Forms de .NET.</li>
	<li> Blazor WebAssembly:  modelo SPA basado en WebAssembly. Da la posibilidad de realizar operaciones en el lado del servidor o llamar a APIs para solicitar datos, para salvaguardar información que quizás no quieras que se ejecute en el cliente. Para entender esto, hay que entender en qué consiste WebAssembly. </li>
</ul>

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/blazor/blazor-webassembly.png"/>
  <i>Blazor WebAssembly</i>
</div>

### WebAssembly
En este concepto radica la clave del funcionamiento de Blazor WebAssembly, en el cual nos vamos a enfocar de ahora en adelante. WebAssembly (conocido también por su abreviatura Wasm) es un estándar que permite ejecutar código binario en un navegador web para ofrecer un rendimiento mayor que Javascript. Es decir, el servidor enviará al navegador web todos los compilados que necesite para ejecutar en la propia máquina local cliente la aplicación web. 

Ahora que ya entendemos Blazor WebAssembly, se sobreentiende la necesidad de mantener un back-end con el procesamiento de información que no quieras llevar al cliente, ¿verdad? Exacto, quedaría feo que te decompilaran las librerías y expusieras información o lógica sensible.

### JS Interop
¿Y esto quiere decir que ya no podemos usar Javascript? Nada más lejos de la realidad. Como sabemos, existen multitud de librerías de terceros escritas en Javascript, que podemos seguir utilizando también en Blazor. Para ello, ofrece la posibilidad de interoperar entre Javascript y C# de manera sencilla a través del servicio IJSRuntime.

## Blazor en funcionamiento: ejemplo real
Vamos a ver un ejemplo real en el que Blazor puede ser una solución más que válida. Vamos a plantear una aplicación web de preguntas y respuestas. Para ello, vamos a necesitar:
<ul>	
<li>Host donde alojar la aplicación.</li>
<li>BBDD donde almacenar las preguntas y respuestas.</li>
</ul>

Como opción de alojamiento seleccionaremos Azure, concretamente usaremos los servicios:
<ul>
<li>App Service para alojar la aplicación</li>
<li>SQL database (SQL server) para la BBDD</li>
</ul>

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/blazor/diagram.png"/>
  <i>Diagrama App Blazor en Azure</i>
</div>

Recordemos que Azure cobra por uso de recuersos (CPU, memoria, etc), así que… ¿qué mejor que Blazor WebAssembly para delegar estos procesamientos de cálculos al cliente y tratar de minimizar cómputos en servidor?

Como versión de .NET usaremos .NET Core 3.1, ya que está tanto soportado en App Service como Blazor.

Una vez clara la infraestructura, vamos a ver cómo organizamos la arquitectura de la aplicación:

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/blazor/sln.png"/>
  <i>Diagrama App Blazor en Azure</i>
</div>

Como vemos, contamos con un proyecto para el lado servidor, que nos va a mantener los procesamientos de lógicas que no queremos llevar al cliente (por ejemplo, comprobar qué respuesta es correcta), mientras que otros quedarán del lado del cliente (por ejemplo, saber qué pregunta toca mostrar al usuario). Esto también nos permite desacoplar parte front-end de la parte back-end, con los beneficios que ello conlleva.

### Frond-end
Planteamos 3 bloques principales:
<ul>
<li>Pages/Components/Shared: ficheros razor (estructurado en: páginas, componentes para compartir entre diferentes páginas y layouts genéricos). La lógica está escrita en C# (en mi caso usando code-behind, creando un archivo razor.cs, pero puede estar en la misma página razor) para comunicar con los servicios que veremos en el siguiente punto.</li>
<li>Services: servicios donde reside la lógica de la aplicación del cliente. Por hacer la analogía, estos serían ficheros Javascript en cualquier otro escenario, pero en Blazor serán ficheros cs y por tanto escritos en el lenguaje C#.</li>
<li>wwwroot: ficheros css, js, images, etc. Contiene el fichero index.html, que será el punto de entrada de la aplicación por parte del cliente web. En el fichero se establecerá el tag HTML “app”, que será el componente raíz del cual se cargarán el resto de componentes Blazor.</li>
<li>Model: modelo de datos que se usará entre los servicios y los diferentes ficheros de razor. </li>
</ul>

### Back-end
Se basará en una arquitectura organizada mediante DDD (driven domain design) y para el acceso a la base de datos utilizaremos Entity Framework Core. Para ello, tendremos 3 proyectos:
<ul>
<li>Server: punto de entrada de nuestra aplicación, donde estarán alojados los controladores.</li>
<li>Domain: lógica de negocio de nuestra aplicación, donde residen a su vez las entidades del dominio.</li>
<li>Infrastructure: recursos necesarios para recuperar la información de la base de datos a través de Entity Framework Core.</li>
</ul>

### Shared
Además, planteo un proyecto compartido entre el cliente y el servidor, que serán los DTOs de comunicación entre los servicios del proyecto del front-end y los controladores del back-end.

### Despliegue
El despliegue con Visual Studio y Azure se vuelve de lo más trivial. Usaremos el asistente que ya nos ofrece el IDE, establecemos nuestra cuenta de Azure ¡y listo! Asociaremos la aplicación con el servicio de App Service (si no lo tenemos creado, podemos crearlo desde el mismo asistente) y lo mismo con la instancia de base de datos.

Para la conexión a la base de datos, desde el servicio de Azure App Service se ofrece la posibilidad de crear configuraciones. Podemos coger la ruta de conexión a la base de datos del servicio de Azure SQL Server y establecerla como configuración en App Service.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/blazor/publish.png"/>
  <i>Publicación de la aplicación en App Service y SQL Server</i>
</div>

### Funcionando, que es gerundio
En la primera petición al servidor, vemos cómo nos devuelve todas las dlls necesarias para la correcta ejecución:
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/blazor/dlls.png"/>
</div>

Si seleccionamos una respuesta, vemos cómo va al servidor a guardar la respuesta que ha seleccionado el usuario, enviando la mínima información indispensable:
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/blazor/post.png"/>
</div>

Con ello, ofrecemos al usuario una interacción más enriquecida con nuestra aplicación.

## Conclusiones
Blazor y en especial Blazor WebAssembly ha venido para quedarse. Es un framework muy completo que Microsoft está apostando fuerte para desarrollos en los que se requiera una alta carga de desarrollo frond-end, en especial si se requieren interfaces muy enriquecidas e interactivas para el usuario. Si triunfará o no, sólo el tiempo lo dirá,. Lo que es seguro es que, a cada nuevo proyecto que se os plantee, tendréis por obligación que meter en la ecuación de frameworks a usar, la variable "Blazor".

Y como siempre, cualquier duda o sugerencia, estoy disponible en mis redes sociales :)
