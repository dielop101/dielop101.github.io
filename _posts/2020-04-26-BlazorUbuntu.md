---
layout: post
title: Desplegando Blazor WebAssembly a coste cero
---

¿Es posible hoy en día desplegar una aplicación .NET a coste 0? Pues claro que sí. Hoy os traigo los pasos para poder desplegar exactamente una aplicación Blazor WebAssembly en un servidor Ubuntu y que sea accesible a través del navegador de manera pública.

<br />
<br />
<div align="center">
  <img src="/images/BlazorUbuntu/00-dotnet.png"/>
</div>
<br />

## Virtualización
Lo primero que tendremos que hacer será conseguir el software que nos permitirá virtualizar nuestro servidor en nuestra propia máquina. Para ello, yo utilizo <a 
href="https://www.vmware.com/es.html" target="_blank">VMware</a>. VMware es un programa que simula un sistema físico (un computador, un hardware) con unas características de hardware determinadas. Como el objetivo es conseguirlo a coste 0, podemos descargar la versión gratuita desde el siguiente <a 
href="https://www.vmware.com/es/products/workstation-player/workstation-player-evaluation.html" target="_blank">link</a>.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/01-VMware.png"/>
  <div><i>VMware recién instalado</i></div>
</div>

## Servidor Ubuntu
Una vez tenemos el sistema necesario para virtualizar el servidor, vamos a descargar una imagen del sistema operativo de Ubuntu Server, también gratuitamente. He optado por Ubuntu, pero podría ser cualquier distribución Linux que queráis. En mi caso, descargué la versión <a 
href="https://releases.ubuntu.com/18.04.4/ubuntu-18.04.4-live-server-amd64.iso" target="_blank">Ubuntu 18.04.4</a>, que es la más reciente y estable hasta la fecha. Es una imagen ISO que vamos a montar sobre VMware.
Crearemos una nueva máquina virtual y, tras seleccionar la imagen ISO que acabamos de descargar, nos detectará automáticamente el servidor.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/02-Ubuntu-Creacion.png"/>
  <div><i>Creación del servidor en VMware</i></div>
</div>

Seguimos el proceso de creación de la máquina virtual, con los parámetros que necesitemos, y de manera automática VMware nos encenderá la máquina virtual. 
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/03-Ubuntu-Instalacion.png"/>
  <div><i>Instalación Ubuntu Server</i></div>
</div>

Continuamos la instalación del sistema operativo como si de cualquier sistema operativo se tratase, hasta su finalización.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/03-Ubuntu-Instalacion.png"/>
  <div><i>Imagen de login en Ubuntu Server</i></div>
</div>

De manera opcional, recomiendo en el paso de instalación marcar el acceso por SSH, para acceder a la máquina de manera más cómoda desde cualquier terminal.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/04-Ubuntu-ssh.png"/>
  <div><i>Configurar acceso SSH</i></div>
</div>

## Instalación dotnet SDK

Una vez tengamos el servidor levantado, podemos acceder desde nuestro propio terminal, ya que tenemos configurado el acceso a través de SSH. En caso de no tenerlo configurado, utilizaremos el acceso por usuario/contraseña desde la propia ventana de VMware.

Para acceder a través de SSH, previamente has de conocer la IP de la máquina virtual. Para ello, accederemos desde VMware la primera vez para ejecutar el comando <i>ifconfig</i>, que nos dará la IP asignada:

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/05-ifconfig.png"/>
  <div><i>ifconfig</i></div>
</div>

Con ello, desde un terminal que esté dentro de nuestra red, podremos acceder remotamente a la máquina, ejecutando <i>ssh NOMBREUSUARIO@IP</i>

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/06-cmd-ssh.png"/>
  <div><i>Acceso remoto al servidor a través de SSH</i></div>
</div>

Instalamos dotnet SDK, en mi caso será la versión 3.1. Para ello, seguimos los pasos descritos en este <a 
href="https://docs.microsoft.com/es-es/dotnet/core/install/linux-package-manager-ubuntu-1804" target="_blank">link</a> de la propia página de Microsoft.

```csharp
sudo add-apt-repository universe
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1
```
El último paso, si falla, deberemos ejecutar:
```csharp
sudo dpkg --purge packages-microsoft-prod && sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1
```

Y, si aún así vuelve a fallar:
```csharp
sudo apt-get install -y gpg
wget -O- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o microsoft.asc.gpg
sudo mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
wget https://packages.microsoft.com/config/ubuntu/18.04/prod.list
sudo mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
sudo chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg
sudo chown root:root /etc/apt/sources.list.d/microsoft-prod.list
sudo apt-get install -y apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1
```

Comprobamos que se ha instalado correctamente, ejecutando <i> dotnet --version</i>.

##Creación aplicación Blazor WebAssembly
Una vez tenemos dotnet instalado, tocará instalar la plantilla de WebAssembly de Blazor WebAssembly, que será la que utilicemos para nuestro ejemplo. Para ello, seguiremos los acciones descritas en el siguiente <a 
href="ttps://docs.microsoft.com/es-es/aspnet/core/blazor/get-started?view=aspnetcore-3.1&tabs=visual-studio" target="_blank">link</a>.

Ejecutamos el siguiente comando y ya tendremos la plantilla disponible para crearla:
```csharp
dotnet new -i Microsoft.AspNetCore.Components.WebAssembly.Templates::3.2.0-preview2.20160.5
```

Veremos cómo en las plantillas disponibles nos aparecerá blazorwasm.

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/07-template-Blazor-webAssembly.png"/>
  <div><i>Listado plantillas instaladas</i></div>
</div>

Creamos el proyecto blazorwasm a través de su plantilla:
```csharp
dotnet new blazorwasm --hosted
```

Con esto, ya tenemos la aplicación creada. Si accedemos a la carpeta creada con nombre "Server" y ejecutamos <i>dotnet run</i>, tendremos la aplicación levantada.
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/08-dotnet-run.png"/>
  <div><i>dotnet run</i></div>
</div>

##Modificación acceso aplicación Blazor WebAssembly
En primera instancia, la aplicación se levanta sobre localhost. Esto quiere decir que sólo es accesible desde nuestra propia máquina virtual. Como esto no nos vale, tendremos que cambiar el despliegue para que se lance sobre la IP 0.0.0.0:

En el fichero de configuración appsettings.json que tenemos en la propia carpeta "Server", tenemos que añadir la siguiente configuración:
```csharp
"Kestrel": { 
	"EndPoints": { 
		"Http": { 
			"Url": "http://0.0.0.0:5000" 
		} 
	} 
}
```

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/09-appsettings.json.png"/>
  <div><i>Modificación fichero appsettings.json a través del comando nano</i></div>
</div>


Esto hará que, cuando ejecutemos dotnet run, se levante en la IP deseada:
<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/10-dotnet-run_2.png"/>
  <div><i>dotnet run 0.0.0.0:5000</i></div>
</div>

Ahora sí, desde cualquier navegador dentro de la red, podremos acceder a través de la IP/puerto a la aplicación Blazor:

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/11-blazor-app.png"/>
  <div><i>Blazor app</i></div>
</div>

##ngrok
Para dar acceso a nuestra máquina a cualquier navegador de internet, utilizaremos la aplicación <a 
href="https://ngrok.com/" target="_blank">ngrok</a>. Esta aplicación nos facilitará un túnel para exponer nuestra aplicación web a internet y un nombre para poder acceder a la misma. Todo ello, de manera gratuita.

Para ello, instalaremos la aplicación y ejecutaremos lo siguiente:
```csharp
ngrok.exe http http://IP:5000/
```

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/12-ngrok.png"/>
  <div><i>ngrok en acción</i></div>
</div>

Si nos fijamos, ngrok ya nos habrá proveído de una URL de acceso. Si la usamos desde cualquier navegador externo a nuestra red, veremos cómo es posible acceder a la aplicación.

<div align="center" style="margin-bottom: 25px;">
  <img src="/images/BlazorUbuntu/13-ngrok-acceso.png"/>
  <div><i>ngrok en acción desde el navegador</i></div>
</div>


## Conclusiones
Como hemos visto, hemos conseguido levantar y acceder desde cualquier navegador web a una aplicación web, a coste 0. Ya habréis imaginado que esto tiene una pequeña trampa, ya que obviamente pagarás tu propio mantenimiento (coste de tener tu ordenador encendido), pero para crear servicios de .NET es una buena solución.

Cualquier duda o sugerencia, estoy disponible en mis redes sociales :)