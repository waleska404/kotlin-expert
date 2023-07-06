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

	suspend fun doLogin(user: String, pass: String) {
		// Server request
		return User("Maikela")
	}

	suspend fun requestCurrentFriends(user: User) {
		// Server request
		return listOf(User("1"), User("2"))
	}

	suspend fun requestSuggestedFriends(user: User) {
		// Server request
		return listOf(User("3"), User("4"))
	}
}

// PSEUDOCODE SOLUCIÓN AL CALLBACK HELL
coroutine {
	println("Starting")
	val user = userService.doLogin("user", "pass") 
	val currentFriends = userService.requestCurrentFriends(user) 
	val finalUser = user.copy(friends = currentFriends)
	println(finalUser)
}
```
















