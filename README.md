# Ejercicio 1

La función `combineOutputs` tiene como propósito devolver un valor de tipo `CustomResult`. Se comprueba que el resultado es exitoso (`CustomResult.RSuccess`) y además si es el mismo producto a través del `.id`. Si es el mismo producto, se hace una copia con `.copy` del producto para no modificar el objeto original y mantenerlo inalterable. La diferencia que hay entre `.apply` y `.also` es que `apply` se utiliza para modificar los valores del objeto y poder devolver el mismo objeto, y `also` devolvería solo el objeto original sin los valores modificados. `toMutableList` se utiliza para que se pueda añadir a la lista (`addAll`) todos los `productsProviders` del segundo resultado (`secondaryResult`).

### 3 tests unitarios:

```kotlin
    @Test
    fun `test any critical error received`(){
        val result1 = CustomResult.RSuccess(ProductData("id1", 10, listOf("provider1")))
        val result2 = CustomResult.RCriticalError
        val combinedResult = combineOutputs(result1, result2)
        assertEquals(CustomResult.RCriticalError, combinedResult)
    }

    @Test
    fun `test successful results, return only priority result`(){
        val result1 = CustomResult.RSuccess(ProductData("id1", 10, listOf("provider2")))
        val result2 = CustomResult.RSuccess(ProductData("id2", 5, listOf("provider3")))
        val combinedResult = combineOutputs(result1, result2)
        assertEquals(result1, combinedResult)
    }


    @Test
    fun `test successful results, combine output`(){
        val product1 = ProductData("id1", 4, listOf("provider4"))
        val product2 = ProductData("id1", 3, listOf("provider5", "provider6"))
        val result1 = CustomResult.RSuccess(product1)
        val result2 = CustomResult.RSuccess(product2)
        val combinedResult = combineOutputs(result1, result2)
        val expectedProduct = product1.copy(
            amount = product1.amount + product2.amount,
            providers = (product1.providers + product2.providers).distinct()
        )
        val expectedResult = CustomResult.RSuccess(expectedProduct)
        assertEquals(expectedResult, combinedResult)
    }
```
### Código alternativo:

```kotlin
fun combineOutputsSimplified(priorityResult: CustomResult, secondaryResult: CustomResult): CustomResult {
    return when {
        priorityResult is CustomResult.RSuccess && secondaryResult is CustomResult.RSuccess &&
                priorityResult.product.id == secondaryResult.product.id -> {
            priorityResult.copy(
                product = priorityResult.product.copy(
                    amount = priorityResult.product.amount + secondaryResult.product.amount,
                    providers = (priorityResult.product.providers + secondaryResult.product.providers).distinct()
                )
            )
        }
        else -> CustomResult.RCriticalError
    }
}
```

# Ejercicio 2
```kotlin
fun getMoviesInPlatforms(movies:List<Movie>, platforms:List<Platform>):List<MovieInPlatform>{
        return movies
            .filter { movie -> platforms.any { it.movieId == movie.movieId && (it.inService1 || it.inService2) } }
            .map { movie ->
                val platform = platforms.firstOrNull { it.movieId == movie.movieId }
                MovieInPlatform(
                    movieId = movie.movieId,
                    title = movie.title,
                    inService2 = platform?.inService2 ?: false
                )
            }
    }

    @Test
    fun `test get movies in platforms result`(){
        val moviesList = listOf(
            Movie(movieId="Movie1",title="The movie 1"),
            Movie(movieId="Movie2",title="The movie 2"),
            Movie(movieId="Movie3",title="The movie 3"),
            Movie(movieId="Movie4",title="The movie 4")
        )
        val platformsList = listOf(
            Platform(movieId = "Movie1", inService1 = false, inService2 = false),
            Platform(movieId = "Movie2", inService1 = true, inService2 = false),
            Platform(movieId = "Movie4", inService1 = true, inService2 = true)
        )

        val result = getMoviesInPlatforms(moviesList,platformsList)

        //Assert on how many movies are from the service 2
        val countService2 = result.count { it.inService2 }
        Assertions.assertEquals(2, countService2)
    }
```
