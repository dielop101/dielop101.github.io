---
layout: post
title: ObjectMother - Mejorando el testing
---

Seguro que más de una vez has puesto en duda la utilidad de los tests que realizas. Sientes que el tiempo dedicado es realmente alto en comparación con el coste del desarrollo y, para colmo, tras pasar toda una jornada laboral realizándolos, esa controvertida métrica que tantos dolores de cabeza da, como es el <i>code average</i>, no sube más del 20%... 

Con este post, voy a contaros un patrón que personalmente me ayudó a quitar esa sensación de "pereza" a la hora de realizar pruebas unitarias.

## Problemas en los tests
* No existe una estructura de código base preparada. Y con código incluyo ni siquiera existir proyecto creado
* Ficheros de código enormes donde no se modulariza nada
* No se aplican ningún tipo de estrategia ni arquitecturas conocidas
* Nombrado de los tests/variables sin contexto
* Duplicidad de código

La sensación es que los tests están un paso por detrás en cuanto a exigencia de calidad de código con respecto al desarrollo funcional en sí.

Un patrón de diseño que me ha ayudado a optimizar mis tiempos a la hora de realizar los tests y a mantener una limpieza y orden en el código es el patrón <b>ObjectMother</b>.

## ObjectMother 
Su definición es sencilla: se basa en un sistema de retorno de objetos con características específicas para pasar a nuestros componentes o servicios que vayamos a testear. Hablando en términos de implementación, se puede ver como una clase que retorna, normalmente mediante un patrón Factory, objetos específicos.

## Mejoras
* Centralizar los objetos
* Delegar en un tercero la responsabilidad de crear los objetos
* Reducir la posibilidad de código duplicado, ya que suele ser habitual compartir entre diferentes tests objetos con similares características
* Mejora la sintaxis en el nombrado de los objetos
* Tener una estructura de código base preparado, evitando el tener que crear de cero los datos

## Construcción
Dependiendo de lo complejo que sea nuestro entorno a testear, podemos tener:
* De 1 a N ObjectMothers
* Desde 1 clase a 1 proyecto o N proyectos

La decisión dependerá de cómo se intuya que vaya a crecer el dominio de la aplicación.

## Ejemplo
Supongamos que tenemos un servicio que calcule el riesgo de dar un crédito a una persona. Las especificaciones nos indican que:
* Si no trabajas, el sistema devolverá 50% de riesgo.
* En caso contrario, 0%

Además, la edad también afectará en el cálculo, ya que no trabajar teniendo una edad superior a 50 años, incrementará el riesgo a 75%.

El servicio recibirá una entidad *Person*, siendo uno de sus atributos *Age* y otro *HasJob*.

```csharp
public class Person
{
	public Person(bool hasJob, int age)
	{
		HasJob = hasJob;
		Age = age;
	}

	public int Age { get; set; }
	public bool HasJob { get; set; }
}
```

¿Cómo sería nuestro ObjectMother de la entidad *Person*?

Crearemos una clase llamada *PersonMother* que contendrá los métodos que devolverán objetos *Person*.

Una posibilidad sería tener un método que devuelva un objeto *Person* genérico y que reciba como parámetros si trabaja y su edad.

```csharp
public static Person CreatePerson(bool hasJob, int age)
{
	return new Person(hasJob, age);
}
```

A partir de este método, podemos tener una función que devuelva una persona con trabajo.

```csharp
public static Person CreatePersonWithJob()
{
	return CreatePerson(true, 30);
}
```

De nuevo, construiremos a partir de este segundo método otro que devuelva una persona sin trabajo y, sobre este método, otro nuevo que además tenga en cuenta una edad superior a 50.

```csharp
public static Person CreatePersonWithoutJob()
{
	var person = CreatePersonWithJob();
	person.HasJob = false;

	return person;
}

public static Person CreatePersonWithoutJobAndAgeRisk()
{
	var person = CreatePersonWithoutJob();
	person.Age = 60;

	return person;
}
```

## Conclusiones
Hay diferentes maneras de crear un ObjectMother, pero la esencia es la misma: centralizar y dar la funcionalidad necesaria para crear objetos con las características específicas que el negocio requiera. 

Cualquier duda o sugerencia, estoy disponible en mis redes sociales :)