# 2. CORRUTINAS

- Las corrutinas nos permiten gestionar los hilos, y por otro lado gestionar la concurrencia: cuando ocurren las peticiones, como sincronizarlas, etc.

- Las corrutinas nos permiten escribir el código asíncrono pero de forma secuencial. Las peticiones que tengamos que requieran un callback, se pueden sustituir por peticiones que *parecen* secuenciales, y por tanto, el resultado de una linea lo podemos utilizar en la linea siguiente. (Solucionando el callback hell).

- Varias corrutinas se pueden ejecutar en un mismo hilo, los hilos se gestionan de manera eficiente para repartirse el trabajo. Mientras que los hilos concurrentes que se pueden ejecutar en una aplicación son limitados, la cantidad de corrutinas que se pueden utilizar en un mismo instante es muy grande.

- Las corrutinas están basadas en la idea de funciones suspendidas, estas son funciones que pueden suspender la ejecución de una corrutina en el punto donde se las llame, y una vez que esa función `suspend` ha terminado, informa a la corrutina para que pueda seguir ejecutandose en el punto donde lo dejó. Esa función `suspend` nos habrá devuelto un valor de retorno que podremos utilizar en las siguientes lineas.

- Las corrutinas son el lugar seguro donde las funciones `suspend` se pueden ejecutar sin llegar bloquear (por normal general) el hilo actual de ejecución.

	```
	data class User(val name: String, val friends: List<User>)

	class UserService {

		fun doLogin(user: String, pass: String, callback: (User) -> Unit) {
			// Server request
			callback(User("Maikela"))
		}

		fun requestCurrentFriends(user: User, callback: (List<User>) -> Unit) {
			// Server request
			callback(listOf(User("1"), User("2")))
		}

		fun requestSuggestedFriends(user: User, callback: (List<User>) -> Unit) {
			// Server request
			callback(listOf(User("3"), User("4")))
		}
	}

	// CALLBACK HELL EXAMPLE
	fun test() {
		val userService = UserService()
		println("Starting")
		userService.doLogin("user", "pass") { user ->
			userService.requestCurrentFriends(user) { currentFriends ->
				userService.requestSuggestedFriends(user) {
					val finalUser = user.copy(friends = currentFriends + suggestedFriends)
					println(finalUser)
				}
			}
		}
	}

	// PSEUDOCODE SOLUCIÓN AL CALLBACK HELL
	coroutine {
		val user = suspended { userService.doLogin("user", "pass") }
		val currentFriends = suspended { userService.requestCurrentFriends(user) }
		val finalUser = user.copy(friends = currentFriends)
		println(finalUser)
	}
	```

## Funciones Suspend:

- Funciones que nos permiten suspender el ciclo de ejecución de la corrutina, para que esta espere a que el resultado esté listo y poder utilizarlo en la siguiente linea. 

- Podemos convertir cualquier función en suspendida añadiendo delante la palabra reservada `suspend`.

- Una regla de las funciones suspendidas es que para que tenga sentido marcar una función como `suspend`, debe utilizar otra función suspend dentro.

- El ejemplo inicial con funciones suspend quedaría de la siguiente manera. (El interior de las funciones todavía es fake):
	```
	data class User(val name: String, val friends: List<User>)

	class UserService {

		suspend fun doLogin(user: String, pass: String): User {
			// Server request
			return User("Maikela")
		}

		suspend fun requestCurrentFriends(user: User): List<User> {
			// Server request
			return listOf(User("1"), User("2"))
		}

		suspend fun requestSuggestedFriends(user: User): List<User> {
			// Server request
			return listOf(User("3"), User("4"))
		}
	}

	fun test () {

		val userService = UserService()

		coroutine {
			println("Starting")
			val user = userService.doLogin("user", "pass") 
			val currentFriends = userService.requestCurrentFriends(user) 
			val finalUser = user.copy(friends = currentFriends)
			println(finalUser)
		}
	}
	```

## CoroutineContext y Dispatchers:

- El `CoroutineContext` es el que va a definir las condiciones en las que se ejecuta el bloque de código afectado por esa corrutina. Está formado por varios elementos: los dos más importantes son 
	- el `Dispatcher`: va a definir en qué hilo o conjunto de hilos se va a ejecutar y el código que afecta a ese contexto, y 
	- el `Job`: nos va a permitir controlar y realizar acciones sobre ese contexto, por ejemplo: esperar a que todas las corrutinas asociadas a ese job acaben, o cancelar todas esas corrutinas.

- La función `withContext` nos permite modificar el context de ejecución a partir del punto en el que se ejecute. Todo lo que añadamos a este contexto, sobreescribirá al contexto principal de la corrutina.

- En el ejemplo siguiente, tenemos una corrutina a la que le estamos diciendo que se ejecute en el hilo principal. Pero las funciones que se ejecutan son suspend, y utilizan `withContext` para cambiar de hilo.
	```
	data class User(val name: String, val friends: List<User>)

	class UserService {

		suspend fun doLogin(user: String, pass: String): User = withContext(Dispatchers.IO) {
			// Server request
			User("Maikela")
		}

		suspend fun requestCurrentFriends(user: User): List<User> = withContext(Dispatchers.IO) {
			// Server request
			listOf(User("1"), User("2"))
		}

		suspend fun requestSuggestedFriends(user: User) List<User> = withContext(Dispatchers.IO) {
			// Server request
			listOf(User("3"), User("4"))
		}
	}

	fun test () {

		val userService = UserService()

		coroutine(Dispatchers.Main) {
			println("Starting")
			val user = userService.doLogin("user", "pass") 
			val currentFriends = userService.requestCurrentFriends(user) 
			val finalUser = user.copy(friends = currentFriends)
			println(finalUser)
		}
	}	
	```

- Dispones de diferentes `Dispatchers` para indicar los hilos de ejecución: 
	- `Default`: Se utilizará muchas veces por defecto si no indicamos nada. Normalmente para operaciones que requieran un uso intensivo de la CPU. Algoritmos, tratamiento de listas muy grandes, etc. Va a utilizar tantos hilos como cores tenga la CPU de nuestro dispositivo.
	- `IO`: Cuando nos conectamos a otro sistema, y podemos quedarnos bloqueados esperando a que responda, pero no estamos empleando la CPU para nada. Acceso a base de datos, ficheros, peticiones a sensores, etc. Utiliza 64 hilos, o en caso de que la CPU del dispositivo tenga más cores que 64, utilizaría tantos hilos como cores del dispositivo.
	- `Main`: Hilo principal de las aplicaciones que utilizan UI.
	- `Uncofined`: Se usaba antes para testing pero ya casi no se utiliza. Lo que hace es que cuando vuelve de una petición, no necesariamente vuelve al hilo original de la corrutina sino que alomejor se queda en el hilo que la llamó. De esta forma no hay saltos entre hilos, ni entre dispatchers. 










