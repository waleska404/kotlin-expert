# 2. CONCEPTOS BÁSICOS

- En Kotlin, todos los tipos son clases. Todos los tipos son objetos, todos extienden de `Any` que sería equivalente al `Object` de Java. Por tanto todos los tipos se comportan igual en Kotlin, no como en Java. 

	Como desventaja, si quiero asignar un integer a una variable de tipo Long, no podré. No ocurren las conversiones automáticas que ocurrían entre tipos básicos en Java. Para esto disponemos la funciones de tipo `toLong()`, que nos permiten hacer esas conversiones entre tipos básicos.

- Las funciones son elementos de primer nivel. Igual que las variables, podemos definir funciones fuera de cualquier tipo de objeto o de clase. Incluso se pueden definir tipos del lenguaje que representan funciones.

## Clases:
- Hay dos tipos de clases muy diferenciadas: Objetos y Estructuras de Datos:
	- Objetos: Tienen información privada en forma de variable, y métodos públicos que nos permiten trabajar con esa información.
	- Estructuras de datos: Modelan información, estado y solemos poder acceder a esas variables de forma pública, ya sea mediante 
				setters o como en Kotlin, properties.

- Por defecto las clases en Kotlin son cerradas y para abrirlas tenemos que utilizar la palabra reservada `open`. Con el `open` podemos crear clases que hereden de la clase indicada como `open`. Por ejemplo:
			`open class Person(name: String, age: Int)` en vez de `class Person(name: String, age: Int)`

- Para crear una clase abstracta utilizamos la palabra reservada `abstract` en lugar de `open`. 

- Las clases abstractas y abiertas se diferencian en que, de una clase abstracta no se puede crear una instancia, de una abierta sí.

- Kotlin solo permite herencia simple, no herencia multiple, eso quiere decir que solo podemos heredar de una clase, no de varias a la vez.

- Ejemplo:
	```
	open class Person(name: String, age: Int)

	class Developer(name: String) : Person(name, 30) // Developer extiende de Person

	// llamando a los constructores para crear instancias
	Person("Frida", 30)
	Developer("Tom")
	```

## Properties:
- En Kotlin la forma de almacenar estado dentro de las clases, se realiza mediante propiedades. Estas propiedades vendrían a ser equivalente a un campo + un getter + un setter en Java.

- Es obligatorio asignar un valor inicial a las properties durante la construcción del objeto. Ejemplo:
	```
	open class Person(name: String, age: Int) {
		val name: String = name
		val age: Int = name
	}
	```
	Pero si el valor de la propertie lo vamos a igualar al parámetro del constructor, lo podemos indicar de la siguiente manera:

	```
	open class Person(val name: String, val age: Int)
	```
	* `val` si no vamos a modificar el valor de la property, en caso contrario la declariamos con `var`

- Tienen un getter y un setter por defecto que se puede sobreescribir. Cuando los sobreescribimos para acceder al valor de la property se utiliza la palabra reservada `field`. Por ejemplo:

	```
	open class Person(name: String, val age: Int) {
		var name = name
		get() = "Hello $field"
		set(value) {
			if(value != field) {
				field = value
			}
		}
	}
	```

- Properties sin backing field: Podemos tener properties sin backing field, esto lo hacemos dandole su valor solo a partir del getter. Esto nos puede servir para que el valor de esta propertie sea calculado cada vez que se pida. Lo podríamos hacer también con una función, pero la solución de property sin backing field es más avanzada:

	```
	class AppState {
    	val text = mutableStateOf("")
    	val buttonEnabled: Boolean
        	get() = text.value.isNotEmpty() 
	}
	```	

## Interfaces:
- Nos permiten definir un comportamiento que luego tendrán que implementar otros componentes. Con la salvedad de que en las interfaces de Kotlin podemos añadir código, aunque con ciertas restricciones. 

- En una interfaz podemos definir comportamiento pero en ningún caso podemos almacenar estado. Esto quiere decir que no podemos definir una property que necesite un backing field. Pero si que podemos definir properties que luego tengan que implementar las clases que extiendan esta interfaz, y usar esas properties en el código de la propia interfaz.

	```
	interface CanWalk {
		fun doStep()

		fun walk(steps: Int) {
			repeat(steps) { doStep() }
		}
	}

	class Dog : CanWalk {
		override fun doStep() {
			TODO("Not implemented yet")
		}
	}

	val dog = Dog()
	dog.walk(20)
	```	

## Data Classes:
- Son las que conforman una estructura de datos (vs las otras clases que son las conformaban un objeto). Estas nos permitirán trabajar con estructuras de datos de forma mucho más sencilla. Si añadimos la palabra reservada `data` delante de nuestra clase, añadirá funcionalidad extra, que nos va a permitir trabajar mejor con estos tipos de clases de datos.

- En Kotlin, igual que en Java, todos los objetos tienen su función `equals` que podemos sobreescribit en cualquier momento.

- El comparador equals de una clase, comprobará si los atributos de esa clase son iguales, y retornará `true` en ese caso, SI la class la hemos declarado con la palabra reservada `data`.

	```
	// en este caso la variable sonIguales tendrá valor false
	class Person(val name: String, val age: Int)

	val p1 = Person("Maik", 30)
	val p2 = Person("Maik", 30)

	val sonIguales = p1 == p2

	// en este caso la variable sonIguales tendrá valor true
	data class Person(val name: String, val age: Int)

	val p1 = Person("Maik", 30)
	val p2 = Person("Maik", 30)

	val sonIguales = p1 == p2
	```

- Las Data Classes tienen sobreescrito el método `toString`. Es más usable.
		
- Con el triple igual `===` se compara si los objetos son el mismo (si son la misma instancia). Esto sirve para clases y para data clases.

- En las Data Classes tenemos un método `copy`. Lo podemos utilizar para copiar una instancia y cambiar sólo uno de los valores:

	```
	val p3 = p2.copy(name = "Lina")
	```	
	
	La función `copy` la necesitamos porque si estamos trabajando con inmutabilidad, realmente no podríamos modificar un valor solo dentro de una estructura de datos, sino que la única manera sería hacer una copia igual, menos el valor que queremos cambiar.

- En las Data Classes también tenemos la desestructuración. Podemos partir el objeto Data Class en variables que se corresponden con los que recibe el constructor como argumento.

	```
	data class Person(val name: String, val age: Int)

	val p1 = Person("Maik", 30)

	val (a, b) = p1 // a = "Maik" y b = 30
	val (_, c) = p1 // _ si queremos omitir algún valor
	val (d) = p1	// podemos omitir valores desde el final: d = "Maik"

	```		

- En una Data Class no puede haber argumentos en el constructor que a su vez no sean properties. Esto porque sinó no puede calcular correctamente el `equals` o el `toString`.
