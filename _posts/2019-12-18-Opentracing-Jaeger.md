---
layout: post
title: Introducción a OpenTracing y Jaeger en .NET
---

Es una realidad que los microservicios están entre nosotros y han venido para quedarse, pero como todas las arquitecturas, tienen sus pros y sus contras. En el punto de contras, vemos el pronunciado coste que exigen los sistemas de servicios distribuidos: <b>el tracing</b>.

Es por ello que no han tardado en aparecer soluciones para tratar de hacer más llevadero esta problemática. Por una parte, <a href="https://opentracing.io/" target="_blank">OpenTracing</a> se perfila como el estándar a seguir, mientras que <a href="https://www.jaegertracing.io/" target="_blank">Jaeger</a> se presenta como una alternativa de implementación <i>open source</i> del estándar que fija OpenTracing.

En este post, trataré de sintetizar los conceptos clave acerca de OpenTracing y su implementación a través de Jaeger para .NET.

<div align="center">
  <img src="/images/OpentracingJaeger/Jaeger-icon.png"/>
</div>

## OpenTracing
### Introducción
De la mano de Google, OpenTracing se ofrece como un estándar asociado a la trazabilidad distribuída. Aunque parezca que OpenTracing es un estándar nuevo, lleva existiendo y usándose en arquitecturas complejas basadas en servicios desde hace tiempo, pero no ha dado el salto a popularizarse hasta encontrar su espacio en los microservicios. 

### Conceptos
Cabe destacar que OpenTracing no se apoya en ningún lenguaje de programación específico, es decir, tiene sentido bajo cualquier lenguaje. A efectos prácticos, esto se traduce en ofrecer al desarrollador una interfaz con una serie de funcionalidades para cumplir su estándar. El listado de los lenguajes de programación soportados se puede consultar <a href="https://github.com/opentracing" target="_blank">aquí</a>. En nuestro caso, elegiremos la interfaz que se ofrece para <a href="https://github.com/opentracing/opentracing-csharp" target="_blank">C#</a>. La implementación de esta interfaz recaerá en manos del desarrollador.

#### Span
Se define como la representación de una unidad lógica de trabajo, pudiendo ser desde una llamada http hasta un acceso a base de datos. La parte clave de este concepto es que tiene referencia a otros spans, construyéndose a través de esta información un flujo temporal, es decir, una traza. 

Contiene los siguientes campos:
<ul>
<li>Nombre.</li>
<li><i>Timespan</i> de inicio y de fin.</li>
<li>Referencias a otros spans.</li>
<li><b>SpanContext.</b></li>
<li>Conjunto de <b>tags</b> (clave/valor).</li>
<li>Conjunto de <b>logs</b> (clave/valor).</li>
</ul>

En Opentracing, se entienden dos tipos de referencias entre spans: 
<ul>
<li><b>ChildOf</b>: el span padre depende del span hijo (síncrono). El span padre siempre espera respuesta del span hijo. Por ejemplo, una petición http la cual esperamos respuesta.</li>
<li><b>FollowsFrom</b>: el span padre no depende del span hijo (asíncrono). El span padre podría terminar antes que el hijo. Por ejemplo, en mensajería asíncrona.</li>
</ul>

OpenTracing proveerá una interfaz <b>ISpan</b> capaz de realizar todas las operaciones necesarias para obtener y alterar estos campos.

##### SpanContext
Contenido dentro del Span, encontramos el <b>SpanContext</b>. Representa el estado del span que se suministrará a hijos y a otros procesos que lo requieran. En esta entidad, podemos encontrar el identificador asociado al span y el identificador de la traza. Además, contiene una lista de <b>baggage items</b>, entendiéndose como un listado clave/valor de información extra que pueda servirnos como identificador extra. 

De nuevo, OpenTracing proveerá una interfaz <b>ISpanContext</b> para gestionarlos, aunque la mayor parte de las operaciones se podrán realizar a través de la funcionalidad que ofrece la interfaz ISpan.

##### Tags y logs
Los <b>tags</b> sirven para anotar consultas, filtros y de más información necesaria para comprender la traza. En cambio, los <b>logs</b> son útiles para capturar mensajes de log siguiendo una línea temporal, facilitando entre otras cosas el debug de la ejecución del span.

Existe una convención de nombrado de las claves que podemos encontrar <a href="https://github.com/opentracing/specification/blob/master/semantic_conventions.md" target="_blank">aquí</a>. 

##### Ejemplo de span

```code
    t=0            operation name: db_query               t=x

Tags:
- db.instance:"customers"
- db.statement:"SELECT * FROM mytable WHERE foo='bar'"
- peer.address:"mysql://127.0.0.1:3306/customers"

Logs:
- message:"Can't connect to mysql server on '127.0.0.1'(10061)"

SpanContext:
- trace_id:"abc123"
- span_id:"xyz789"
- Baggage Items:
  - special_id:"vsid1738"
```

#### Tracer
Es el elemento encargado de la creación de spans. Además, tiene la capacidad de propagar a través de operaciones de inyección y extracción la información entre diferentes procesos. 

Todas estas operaciones las proveerá OpenTracing a través de la interfaz <b>ITracer</b>.

<div align="center">
  <img src="/images/OpentracingJaeger/Trace-inject-extract.png"/>
</div>

### DAG
Como se intuye a raíz de los conceptos, OpenTracing establece su base en que una traza sea un <i>Gráfico Acíclico Dirigido (DAG)</i> de spans. En el siguiente ejemplo, vemos una traza de 8 spans:
```code
        [Span A]  ←←←(la raíz)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C es un `ChildOf` Span A)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F] >>> [Span G] >>> [Span H]
                                       ↑
                                       ↑
                                       ↑
                         (Span G `FollowsFrom` Span F)

```

También lo podemos ver como una línea temporal:

```code
––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> tiempo

 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··] [Span G··] [Span H··]
```


## Jaeger

### Definición
Jaeger se ofrece como una implementación al estándar de OpenTracing, luego su principal objetivo será monitorizar sistemas distribuidos complejos. La implementación, definida y utilizada hoy día en los sistemas distribuidos de Uber, se basa en <a href="https://research.google/pubs/pub36356/" target="_blank">Dapper</a> y <a href="https://zipkin.io/" target="_blank">OpenZipkin</a>. 

La principal característica que difiere de otras implementaciones es que las trazas no se envían al recolector, sino que es un agente el encargado de recolectar esta información. En la siguiente imagen podemos ver la arquitectura:
<div align="center">
  <img src="/images/OpentracingJaeger/Opentracing-Jaeger-arq.png"/>
</div>

### All-in-one
Como hemos visto en la arquitectura, Jaeger se basa en 5 componentes: cliente, agente, recolector, query, y Jaeger UI, con un componente adicional de almacenamiento de memoria. 

La librería cliente se implementará en cada servicio, pero para levantar el resto de los servicios que forman la infraestructura, Jaeger nos ofrece un ejecutable que denomina <b>all-in-one</b>.

## Ejemplo práctico

### OpenTracing en acción
Para nuestro ejemplo, vamos a usar con una aplicación de consola. Comenzaremos importando la dependencia de OpenTracing que nos ofrecerá entre otras cosas las interfaces:
<div align="center">
  <img src="/images/OpentracingJaeger/OpenTracing-01.png"/>
</div>

Lo primero de todo será obtener a través del constructor una traza de tipo <b>ITracer</b>. El cómo se implemente de momento no nos importa. Sabremos que alguien nos va a inyectar la implementación cuando se requiera. Tendremos una propiedad privada de sólo lectura <b>_trace</b> la cual tendrá valor en el constructor para usarse más adelante.

```csharp
using System;
using OpenTracing;

namespace OpenTraceProgram
{
    class Program
    {
        private readonly ITracer _tracer;

        public Program(ITracer tracer)
        {
            _tracer = tracer;
        }

        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

Una vez tenemos la traza, ya podemos crear nuestro primer <b>span</b>. 

Vamos a crear una función <i>HelloWord()</i> que únicamente saque por consola el mensaje <i>Hello World!</i>. En ella, iniciaremos un espacio donde vivirá el <i>scope</i> del span a través de la instancia <i>_tracer</i>. Debemos indicar al crear el span el nombre que va a tener. Si indicamos <i>finishSpanOnDispose = true</i>, cuando se haga el <i>dispose</i> del objeto scope, se finalizará el span (luego no será necesario que hagamos un <i>scope.Span.Finish()</i>). Vamos a registrar en el log del span que el mensaje "<i>Hello World!</i>" se ha mostrado correctamente.
```csharp
private void HelloWord()
{
	using (IScope scope = _tracer.BuildSpan("HelloWord Method")
		.StartActive(finishSpanOnDispose: true))
	{
		Console.WriteLine("Hello World!");
		scope.Span.Log("Hello World done");
	}
}
```

Con sólo esto, ya tenemos en juego un pequeño <b>DAG</b> que constará de un único span.

Para probar nuestra función, vamos a instanciar la clase <i>Program</i> y vamos a inyectar una implementación. Tenemos 2 opciones:
<ol>
<li>Inyectar <b>GlobalTracer</b>: es una instancia que nos prové OpenTracing para poder realizar pruebas sin necesidad de que el tracing influya. Es un objeto sin funcionalidad.</li>
<li>Inyectar <b>MockTracer</b>: instancia mock para realizar <b>UnitTest</b>, ofrecida también por OpenTracing.</li>
</ol>

Si usamos GlobalTracer:
```csharp
using OpenTracing.Util;
...
	static void Main(string[] args)
	{
		var tracer = GlobalTracer.Instance;
		new Program(tracer).HelloWord();
	}
```

Si usamos MockTracer:
```csharp
using OpenTracing.Mock;
...
	static void Main(string[] args)
	{
		var tracer = new MockTracer(); 
		new Program(tracer).HelloWord();
	}
```

Vamos a realizar una prueba con MockTracer para asegurar que el span que se crea tiene el log correcto:
```csharp
static void Main(string[] args)
{
	var tracer = new MockTracer(); 
	new Program(tracer).HelloWord();

	var spans = tracer.FinishedSpans();
	foreach (var span in spans)
	{
		Console.WriteLine(span.OperationName);
		Console.WriteLine(span.LogEntries.First()
		                  .Fields.First().Value);
	}
}
```

Resultado de la consola:
<div align="center">
  <img src="/images/OpentracingJaeger/OpenTracing-02.png"/>
</div>


### Jaeger en acción
Como veíamos en la arquitectura de Jaeger, necesitamos instanciar el cliente para que pueda posteriormente enviar las trazas al receptor. Para ello, vamos a instalar la dependencia de Jaeger:
<div align="center">
  <img src="/images/OpentracingJaeger/Jaeger-01.png"/>
</div>

Una vez instalado, quitaremos la inyección realizada anteriormente con MockTracer y usaremos Jaeger.

Lo primero que necesitaremos será instanciar la clase <b>Configuration</b>, que será la encargada de configurar y crear el objeto trace. Para instanciar el objeto, pedirá 2 parámetros: el nombre que asociaremos a la traza y un objeto que implemente la interfaz <i>ILoggerFactory</i>. De nombre, pondremos "Program-Main", y de log usaremos el que ofrece .Net Core a través del <i>ServiceCollection</i>. Este punto es interesante, ya que Jaeger obtiene también información a través del log que vayamos a usar en nuestra aplicación.

La configuración básica partirá de la necesidad de instanciar 2 objetos más: <b>SamplerConfiguration</b> y <b>ReporterConfiguration</b>, los cuales también requerirán del objeto <i>loggerFactory</i> creado.

Con el método <i>GetTracer()</i>, obtendremos un objeto de tipo ITracer acorde a la configuración que le hayamos indicado. Realizaremos un <i>cast</i> del objeto ITracer de OpenTracing a la clase Tracer de Jaeger para usarlo.
```csharp
using Jaeger;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.DependencyInjection;
...

static void Main(string[] args)
{
	var loggerFactory = new ServiceCollection()
		.AddLogging()
		.BuildServiceProvider()
		.GetService<ILoggerFactory>();

	var samplerConfiguration = new Configuration
		.SamplerConfiguration(loggerFactory)
		.WithType(ConstSampler.Type)
		.WithParam(1);

	var reporterConfiguration = new Configuration
		.ReporterConfiguration(loggerFactory)
		.WithLogSpans(true);

	var tracer = new Configuration("Program-Main", loggerFactory)
		.WithSampler(samplerConfiguration)
		.WithReporter(reporterConfiguration)
		.GetTracer();

	using ((Tracer)tracer)
	{
		new Program(tracer).HelloWord();
	}
}
```

Una vez listo el cliente, vamos a pasar a configurar el resto de componentes de Jaeger. Como vimos anteriormente, Jaeger ofrece un ejecutable llamado <b>all-in-one</b> que nos permitirá lanzarlos:
<div align="center">
  <img src="/images/OpentracingJaeger/Jaeger-02.png"/>
</div>

Por defecto, el servidor estará en el puerto <b>16686</b>, así que podemos acceder al siguiente enlace para entrar a Jaeger-UI: <a href="http://localhost:16686/search" target="_blank">http://localhost:16686/search</a> 

<div align="center">
  <img src="/images/OpentracingJaeger/Jaeger-03.png"/>
</div>

Si volvemos a lanzar la aplicación de consola, Jaeger es capaz de obtener la información del tracing que acabamos de crear. En el listado de servicios, veremos "Program-Main", que es el nombre que le hemos dado desde la configuración. Si pulsamos el botón "Find Traces", obtendremos toda la información asociada a la traza. En nuestro ejemplo, vemos el span que hemos creado con nombre "HelloWord Method".

<div align="center">
  <img src="/images/OpentracingJaeger/Jaeger-04.png"/>
</div>

Si accedemos al span, vemos información sobre el mismo:

<div align="center">
  <img src="/images/OpentracingJaeger/Jaeger-05.png"/>
</div>

## Conclusiones
Sin entrar al detalle de toda la potencia que nos ofrece OpenTracing y Jaeger, hemos podido ver los conceptos claves y la rapidez con la que podemos levantar un entorno plenamente operativo. Cualquier duda o sugerencia, estoy disponible en mis redes sociales :)

## Enlaces de interés
Para entrar más en profundidad con OpenTracing y Jaeger, os dejo los siguientes links que me han servido para construir este post:
<ul>
<li><a href="https://opentracing.io/docs" target="_blank">https://opentracing.io/docs</a></li>
<li><a href="https://opentracing.io/specification/" target="_blank">https://opentracing.io/specification/</a></li>
<li><a href="https://www.jaegertracing.io/docs/1.16/" target="_blank">https://www.jaegertracing.io/docs/1.16/</a></li>
<li><a href="https://github.com/yurishkuro/opentracing-tutorial/tree/master/csharp" target="_blank">https://github.com/yurishkuro/opentracing-tutorial/tree/master/csharp</a></li>
<li><a href="https://www.paradigmadigital.com/dev/trazabilidad-distribuida-con-opentracing-y-jaeger/" target="_blank">https://www.paradigmadigital.com/dev/trazabilidad-distribuida-con-opentracing-y-jaeger/</a></li>
</ul>