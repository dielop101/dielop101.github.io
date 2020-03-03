---
layout: post
title: Salvaguarda la arquitectura con ArchUnitNet
---

Es probable que en más de una ocasión te hayas saltado de manera accidental la arquitectura de tu solución, o estándares de programación fijados dentro de la empresa que, con suerte, lograrás detectarlos y pararlos manualmente gracias a procesos ya conocidos como las *Pull Request*.

Para nuestra suerte, existe una solución automática que, a través de test unitarios, podría ahorraros mucho tiempo a la hora de salvaguardar la arquitectura. Esta solución se llama <a href="https://www.archunit.org/" target="_blank">ArchUnit</a>, disponible para Java. No obstante, ya existe una solución OpenSource denominada <a href="https://github.com/TNG/ArchUnitNET" target="_blank">ArchUnitNET</a>, de la cual hablaré en este post.

<br />
<br />
<div align="center">
  <img src="/images/ArchUnitNET/ArchUnit-Logo.png"/>
</div>
<br />

## ¿Qué es ArchUnitNET?
Primero, tenemos que conocer qué es ArchUnit. Tal y como define en su web:

*ArchUnit es una librería que sirve para verificar la arquitectura del código Java utilizando cualquier marco de test unitario. ArchUnit es capaz de verificar dependencias entre paquetes, clases, capas, verificar dependencias cíclicas y más.*

Una vez sabido esto, ArchUnitNET no es más que la implementación en C# de ArchUnit. Aunque está todavía en fase inicial, para un uso sencillo y siendo que está orientado al testing, creo que su uso es una apuesta acertada.

## Probando ArchUnitNET
Lo primero que tenemos que hacer es instalar el nuget que indica en la <a href="https://github.com/TNG/ArchUnitNET" target="_blank">web</a>. En mi caso, voy a utilizar *NUnit* para los tests, pero podría servir *XUnit*, dado que también lo soporta.

Como primer ejemplo, vamos a suponer que queremos asegurar que todas las clases de nuestra librería implementan una interfaz base. Para ello, tenemos que indicar a ArchUnitNET qué librería o conjunto de librerías queremos observar a través del *assembly*. Obtendremos el *assembly* a partir de una clase que resida en la librería (en el ejemplo, la clase es *Class1*):

```csharp
private Architecture Architecture =
	new ArchLoader().LoadAssembly(typeof(Class1).Assembly)
	.Build();
```

Posteriormente, identificaremos a través del objeto *Architecture* nuestra interfaz base:

```csharp
private Interface _baseInterface = 
	Architecture.GetInterfaceOfType(typeof(Contracts.IBaseInterface));
```

Ya solo nos quedará recorrer las clases y verificar que todas implementan la interfaz:

```csharp
Assert.IsTrue(Architecture.Classes.All(archClass => archClass.ImplementsInterface(_baseInterface)));
```

Otro ejemplo sencillo: vamos a verificar que todas las propiedades de una clase cumplen que, si son públicas, el primer carácter vaya en mayúscula:

```csharp
Assert.IsTrue(Architecture.Classes.All(archClass => GetPropertiesByType(archClass).All(prop => char.IsUpper(prop.Name.First()))));
```

Siendo que la función *GetPropertiesByType* devuelve las propiedades filtradas por visibilidad pública:

```csharp
return archClass.GetPropertyMembers().Where(prop => prop.Visibility == Visibility.Public);
```

Con este ejemplo, se nos puede venir otra comprobación bastante útil en el mundo .NET: verificar que todas las interfaces empiezan por "I":

```csharp
Assert.IsTrue(Architecture.Interfaces.All(i => i.Name.StartsWith("I")));
```

Otra de las características que tiene ArchUnit es que maneja una sintaxis muy descriptiva, que veremos en el siguiente ejemplo: vamos a suponer que queremos comprobar que todos los contratos estuviesen metidos en su respectiva carpeta "Contracts". Para ello, comprobaremos que el namespace de todas las interfaces contiene la cadena de texto "Contracts".

El primer paso será obtener un *ObjectProvider* de tipo *Type* que nos defina todos los tipos que cumplen la característica de contener en el namespace el caracter "Contracts":

```csharp
private IObjectProvider<IType> InterfaceLayer = ArchRuleDefinition.Types().That().ResideInNamespace("Contracts");
```

Después, definiremos una regla, la cual va a indicarnos si las interfaces cumplen la característica fijada por el *ObjectProvider* *InterfaceLayer*:

```csharp
IArchRule interfacesShouldBeInContractsLayer =
                   ArchRuleDefinition.Interfaces().Should().Be(InterfaceLayer);
```

Sólo nos quedará ejecutar esta regla sobre nuestra arquitectura:

```csharp
Assert.IsTrue(interfacesShouldBeInContractsLayer.HasNoViolations(Architecture));
```

Al igual que definimos reglas, las reglas también se pueden combinar y ejecutar sobre la arquitectura. Supongamos que tenemos 2 reglas ya creadas, classesHaveCorrectNamespace e interfacesHaveCorrectNamespace, para comprobar que las clases estén en un namespace correcto, así mismo como las interfaces. La combinación se plantearía tal que así:

```csharp
IArchRule combinedArchRule = classesHaveCorrectNamespace.And(interfacesHaveCorrectNamespace);
```

Una vez tengamos la regla combinada, procedemos a ejecutarla:

```csharp
Assert.IsTrue(combinedArchRule.HasNoViolations(Architecture));
```

## ArchUnitNET para tu arquitectura
La librería con lo ya hemos visto, se intuye muy potente y fácil de usar, pero donde he encontrado su punto diferenciador es en la comprobación de dependencias.

Supongamos que tenemos el siguiente escenario: una solución con proyectos que basan su arquitectura en *Driven Domain Design* (DDD). Dicha solución consta de los siguientes proyectos:
<ul>
	<li>Dummy.Domain: no debe depender de nadie</li>
	<li>Dummy.Infrastructure: depende del dominio</li>
	<li>Dummy.Application: depende del dominio</li>
	<li>Dummy.Controllers: depende de la capa de aplicación</li>
</ul>

Con ArchUnitNET podemos asegurar que nadie rompa esta decisión de dependencias. 

¿Cómo lo haríamos? Algo tan sencillo como lo siguiente:

Definimos la arquitectura, cargando todos los assemblies:

```csharp
 private static readonly Architecture Architecture =
            new ArchLoader().LoadAssemblies(
                typeof(Controller).Assembly, 
                typeof(Application).Assembly,
                typeof(Domain).Assembly,
                typeof(Infrastructure).Assembly)
            .Build();

```

Definimos las capas:

```csharp
private readonly IObjectProvider<IType> ControllerLayer =
	ArchRuleDefinition.Types().That().ResideInNamespace("Controllers").As("Controllers Layer");
private readonly IObjectProvider<IType> ApplicationLayer =
	ArchRuleDefinition.Types().That().ResideInNamespace("Application").As("Application Layer");
private readonly IObjectProvider<IType> DomainLayer =
	ArchRuleDefinition.Types().That().ResideInNamespace("Domain").As("Domain Layer");
private readonly IObjectProvider<IType> InfrastructureLayer =
	ArchRuleDefinition.Types().That().ResideInNamespace("Infrastructure").As("Infrastructure Layer");

```

Para cada test, definimos sus reglas, las combinamos y ejecutamos sobre la arquitectura:

```csharp
public void Check_Controller_Dependencies()
{
	IArchRule controllerLayerShouldNotAccessInfrastructureLayer =
		ArchRuleDefinition.Types().That().Are(ControllerLayer).Should().NotDependOnAny(InfrastructureLayer);

	IArchRule controllerLayerShouldNotAccessDomainLayer =
		ArchRuleDefinition.Types().That().Are(ControllerLayer).Should().NotDependOnAny(DomainLayer);

	IArchRule controllerLayerShouldAccessDomainLayer =
		ArchRuleDefinition.Types().That().Are(ControllerLayer).Should().DependOnAny(ApplicationLayer);

	IArchRule combinedArchRule =
		controllerLayerShouldNotAccessInfrastructureLayer
			.And(controllerLayerShouldNotAccessDomainLayer)
			.And(controllerLayerShouldAccessDomainLayer);

	Assert.IsTrue(combinedArchRule.HasNoViolations(Architecture));
}
```

Todos los ejemplos que hemos visto los podréis encontrar en <a href="https://github.com/dielop101/UsingArchUnitNet" target="_blank">mi repositorio de GitHub</a>.

## Conclusiones
Como hemos visto, ArchUnitNET nos da un amplio acceso a clases, interfaces, capas, clases abstractas, etc.; así como métodos para manipularlos con bastante libertad y sencillez de uso. Su puesta a punto es sencilla y los beneficios que da son bastante altos.

Cualquier duda o sugerencia, estoy disponible en mis redes sociales :)