# Estructuras de datos – 3. Listas enlazadas en C (COMPLETO)

## Introducción

Se ha creado la estructura `Libro` para guardar algunos datos:

```c
typedef struct Libro {
    char titulo[50];
    char autor[50];
    char isbn[13];
} Libro;
```

Y construiremos una lista para guardar, insertar, eliminar y obtener en general información sobre los elementos de nuestra lista.

Lo primero que haremos es crear nuestra propia lista, en los archivos:

* `lista.c`
* `lista.h`

Y definir las estructuras que vamos a utilizar.

## Crear estructuras

En primer lugar, vamos a utilizar una estructura llamada `Nodo`, tal cual como vimos en la presentación.

Incluimos el archivo `libro.h` para que detecte lo que es un `Libro`.

Y definimos una estructura llamada `Lista` para guardar el **puntero** a la **cabeza** de esa lista. En la presentación, vimos que la ventaja de
usar `Lista` es que podemos mantener más elementos dentro de la estructura:

```c
#ifndef lista_h
#define lista_h

#include <stdio.h>
#include "libro.h"

typedef struct Nodo {
    Libro libro;
    struct Nodo* siguiente;
} Nodo;

typedef struct Lista {
    Nodo* cabeza;
} Lista;

#endif /* lista_h */
```

En la implementación, incluiremos las operaciones que podemos necesitar hacer con nuestra lista.

## Gestionar nodos

Una de las operaciones que nos puede hacer falta, es la de **crear un nuevo nodo** (`CrearNodo()`) a partir de los datos de un libro.

>En C, para usar memoria dinámica, tenemos que usar `stdlib` para poder utilizar las funciones `malloc()` y `free()`

En primer lugar, instanciamos un nuevo nodo con `malloc()` a la cual le tenemos que indicar el tamaño de nuestro `Nodo`.
Y después, usando la función `strncpy()`, vamos a copiar dentro de nuestro libro, los datos del libro que le pasamos como parámetro.
Y después, como no sabemos a qué va a apuntar el **siguiente**, vamos a hacer que apunte a `NULL` por si acaso, de este modo no tenemos punteros
sueltos.
Y finalmente, lo devolvemos.

Además, en el caso de C, como las cosas no se eliminan solas, tenemos que hacer una función llamada `DestruirNodo()` para destruir un nodo que
ya no queramos mantener en memoria. Y esa función, lo único que tiene que hacer es un `free()`:

```c
#include "lista.h"
#include <stdlib.h>
#include <string.h>

Nodo* CrearNodo(Libro* libro) {
    Nodo* nodo = (Nodo *) malloc(sizeof(Nodo));
    strncpy(nodo->libro.titulo, libro->titulo, 50);
    strncpy(nodo->libro.autor, libro->autor, 50);
    strncpy(nodo->libro.isbn, libro->isbn, 13);
    nodo->siguiente = NULL;
    return nodo
}

void DestruirNodo(Nodo* nodo) {
    free(nodo);
}
```

## Insertar elementos (principio, final, entre medias)

Una de las primeras operaciones de nuestro interés es **insertar al principio** de una lista un libro. Lo vimos en la presentación, es muy
sencillo:

Lo primero que tenemos que hacer, es crear nuestri nuevo nodo a partir de los datos de nuestro libro.
Y después, poner como **cola** del nodo que acabamos de crear la actual **cabeza** de nuestra lista. Así que hacemos que el `siguiente` de
nuestro nuevo nodo sea la `cabeza` de nuestra lista.
Y una vez hecho, lo único que tenemos que hacer es mover el puntero `cabeza` de sitio para que apunte a nuestro nuevo nodo.
Y con eso, ya lo tenemos insertado al principio.

```c
void InsertarPrincipio(Lista* lista, Libro* libro) {
    Nodo* nodo = CrearNodo(libro);
    nodo->siguiente = lista->cabeza;
    lista->cabeza = nodo;
}
```

Para insertar al final de una lista un libro en concreto, es más complicado porque tenemos que recorrer nuestra lista en busca del último
elemento, para que su **siguiente** apunte al nodo recién creado.

Creamos el nodo.

Entonces, mantenemos un nodo especial al que vamos a llamar `puntero`, que incialmente va a ser la **cabeza** de la lista. Y tenemos que hacer
que ese puntero vaya avanzando de una posición a otra hasta que haya alcanzado el final de nuestra lista. Lo vamos a saber porque su
`siguiente` va a valer nulo.
Vamos a avanzar `puntero` mientras tenga `siguiente` haciendo `puntero = puntero->siguiente;`. De este modo, en cada iteración, el puntero va
avanzando una posición y cuando hayamos llegado al último, como ya no tiene `siguiente`, se sale del bucle, y sabemos que `puntero` va a
apuntar al último elemento.
Así que, simplemente hacemos que ahora apunte al nodo recién creado.

Existe una situación en la que esto no va a funcionar y es cuando la lista esté vacía, ya que no vale nada por lo que nisiquiera tendrá puntero.
Así que tenemos que exminar el caso en el que la **cabeza** de la lista esté vacía, si esto pasa, lo único que tenemos que hacer es directamente
posicionarlo como **cabeza** de la lísta.
Por lo demás, en el caso de que la lista no este vacía, evidentemente tenemos que hacer lo que hicimos antes:

```c
void InsertarFinal(Lista* lista, Libro* libro) {
    Nodo* nodo = CrearNodo(libro);
    if (lista->cabeza == NULL) {
        lista->cabeza = nodo;
    } else {
        Nodo* puntero = lista->cabeza;
        while (puntero->siguiente) {
            puntero = puntero->siguiente;
        }
        puntero->siguiente = nodo;
    }
}
```

Ahora, vamos a interntar **insertar después del elemento n**, de nuestra `Lista* lista`, el `Libro* libro`. Es decir, cómo podemos insertar un
libro después del quinto libro o cómo podemos insertar un elemento después del tercer libro.

En primer lugar, instanciar un nuevo nodo.
Después, considerar el caso de que la lista esté vacía, porque en ese caso no hay nada que podamos buscar. Simplemente lo agregamos y ya está.

En caso contrario, tenemos que hacer dos pasos. Uno es buscar precisamente el libro número **n** y otro es insertarlo después.

Para buscar el libro **n**, tenemos que hacer una vez más un recorrido de lista. Sin embargo, esta vez tenemos que mantener una variable llamada
`posicion`, que va a decir que número de libro es el que hay ahora mismo apuntado por `puntero`. Por ejemplo, ahora mismo `puntero` es el libro
número 0, lo que tenemos que hacer es que cada vez que se recorra la lista, tenemos que incrementar la variable `posicion` una vez más. Y esto
lo hacemos hasta que hayamos alcanzado el libro número **n**, por lo que hacemos que **mientras** `posicion < n`.
Existe otro caso que tenemos que verificar también y es que hayamos alcanzado el final de la lista. Si hemos llegado al final de la lista,
pero aún no hemos llegado al libro que nos han pedido, simplemente no podemos seguir avanzando.

Una vez tengamos el libro **n** en la posicion `puntero`, tenemos que insertarlo después, pero no basta hacer `puntero->siguiente = nodo`,
porque seguramente ya esté apuntando a algo. Así que por si acaso, haremos primero es `nodo->siguiente = puntero->siguiente`. De este modo,
primero lo conectamos por la derecha y después por la izquierda.
En la presentación, vimos que esto era un paso necesario.

```c
void InsertarDespues(int n, Lista* lista, Libro* libro) {
    Nodo* nodo = CrearNodo(libro);
    if (lista->cabeza == NULL) {
        lista->cabeza = nodo;
    } else {
        Nodo* puntero = lista->cabeza;
        int posicion = 0;
        while (posicion < n && puntero->siguiente) {
            puntero = puntero->siguiente;
            posicion++;
        }
        nodo->siguiente = puntero->siguiente;
        puntero->siguiente = nodo;
    }
}
```

En el caso de haber alcanzado el final de la lista pero no hayamos alcanzado la posición que nos están pidiendo, no pasa nada, ya que al
`nodo->siguiente` se le va a asociar la lista vacía `puntero->siguiente`, ya que como es el último elemento lo que le sigue
es un **nulo**. Y después lo ponemos al final `puntero->siguiente = nodo`.

## Obtener elementos (longitud, obtener elemento n)

Hablando de acceder a elementos **n**, haremos una función para obtener información de nuestra lista que nos devuelva el libro en la posición **n**.

Nos va a devolver el `Libro*` en la posición `n` de nuestra `Lista* lista`. Devolvemos el libro, no el nodo porque se supone que el nodo es
una cosa interna que solamente le interesa a nuestra lista.

Lo primero que tenemos que hacer es verificar si nuestra lista está vacía, porque si está vacía es que no hay libros, por lo que no podemos
obtener nada. Así que si está vacía, devolvemos un valor falso para indicar que no tenemos nada, aunque en este caso devolvemos un **nulo**.

En el caso de que la lista si tenga cosas, tenemos que hacer algo a lo que hicimos anteriormente. Tenemos que mantener un contador que vaya
vigilando a qué número de libro hay en `puntero` y vaya aumentando hasta que encuentre el libro **n**. Cuando lo encuentre, vamos a devolver
directamente su mismo libro `&puntero->libro`, en este caso un puntero.

Pero, ¿y si sólo tenemos 4 libros y queremos devolver el número 10? Evidentemente nos vamos a pasar. Por eso mantenemos la condición
`puntero->siguiente` para mientras no hayamos llegado al final de la lista.
Si hemos llegado al final de la lista, no podemos pasar al siguiente elemento porque ya no hay más, además, cuando salgamos del bucle `while`, `posicion` va a ser distinto de `n`.
Es decir, tenemos que comprobar que cuando hemos salido, estamos en el elemento `n`.

Así que después del bucle, `posicion != n`, también vamos a devolver `NULL` para indicar que no hemos encontrado tantos libros. Y si
`posicion` vale `n`, entonces estamos en la posición correcta.

```c
Libro* Obtener(int n, Lista* lista) {
    if (lista->cabeza == NULL) {
        return NULL;
    } else {
        Nodo* puntero = lista->cabeza;
        int posicion = 0;
        while (posicion < n && puntero->siguiente) {
            puntero = puntero->siguiente;
            posicion++;
        }
        if (posicion != n) {
            return NULL;
        } else {
            return &puntero->libro;
        }
    }
}
```

Hablando de posiciones, haremos otra función para saber cuántos libros tenemos en nuestra lista.

Podríamos hacer lo mismo de antes, "tener un contador que se incremente en cada iteración del bucle". No está mal, pero vamos a aprovechar algo
que vimos en la presentación, y es que **podemos tener mas elementos en nuestra lista**, y podríamos tener variable extra o campo más bien
llamado `longitud` que nos diga cuántos elementos tenemos en nuestra lista. Cada vez que insertemos un elemento, vamos a incrementar está
variable, y cada vez que eliminemos un elemento la vamos a decrementar:

```c
typedef struct Lista {
    Nodo* cabeza;
    int longitud;
} Lista;
```

De este modo, `longitud` siempre va a mantener cuántos elementos hay en nuestra lista y a cambio, no tenemos que recorrer tantas veces nuestra
lista. Así que lo que tenemos que hacer, es que cada vez que insertemos un elemento, tenemos que incrementar la `longitud` en 1.

Y luego, `Contar()` un elemento es tan fácil como devolver la `longitud` de la lista:

```c
void InsertarPrincipio(Lista* lista, Libro* libro) {
    Nodo* nodo = CrearNodo(libro);
    nodo->siguiente = lista->cabeza;
    lista->cabeza = nodo;
    lista->longitud++;
}

void InsertarFinal(Lista* lista, Libro* libro) {
    Nodo* nodo = CrearNodo(libro);
    if (lista->cabeza == NULL) {
        lista->cabeza = nodo;
    } else {
        Nodo* puntero = lista->cabeza;
        while (puntero->siguiente) {
            puntero = puntero->siguiente;
        }
        puntero->siguiente = nodo;
    }
    lista->longitud++;
}

void InsertarDespues(int n, Lista* lista, Libro* libro) {
    Nodo* nodo = CrearNodo(libro);
    if (lista->cabeza == NULL) {
        lista->cabeza = nodo;
    } else {
        Nodo* puntero = lista->cabeza;
        int posicion = 0;
        while (posicion < n && puntero->siguiente) {
            puntero = puntero->siguiente;
            posicion++;
        }
        nodo->siguiente = puntero->siguiente;
        puntero->siguiente = nodo;
    }
    lista->longitud++;
}

int Contar(Lista* lista) {
    return lista->longitud;
}
```

Evidentemente, cualquiera puede acceder a la `longitud` y ya está. También se puede acceder a la variable `longitud` y modificarla, así que tenemos que tener cuidado con este sistema, pero nos ahorramos la
posibilidad de hacer tantas cosas.

## Eliminar elementos (principio, final, entre medias)

Por último, veremos cómo hacer para eliminar elementos.

Por ejemplo, vamos a ver cómo eliminar el primer elemento de una lista.

Lo que tenemos que hacer primero, es obtener el nodo que vamos a eliminar, que en este caso va a ser `lista->cabeza`.
Recordemos que tenemos que guardar su variable porque tenemos que destruir este nodo manualmente porque sino C no lo va a eliminar.

Ahora, lo único que tenemos que hacer es actualizar la `cabeza` de la lista haciendo que ahora apunte al segundo elemento. De este modo, ahora
el segundo va a ser la `cabeza`.

Una vez que lo tenemos, hacemos `DestruirNodo(eliminado)`, y puesto que tenemos que mantener la referencia a la vez que mantenemos la `longitud`,
lo que hacemos es decrementar la `longitud` de la lista en 1.

Existe un caso en el que no podemos eliminar el principio de la lista, y es cuando esté vacía.
Esto sólo lo vamos a poder hacer si nuestra lista no está vacía, es decir, si nuestra lista tiene `cabeza`:

```c
void EliminarPrincipio(Lista* lista) {
    if (lista->cabeza) {
        Nodo* eliminado = lista->cabeza;
        lista->cabeza = lista->cabeza->siguiente;
        DestruirNodo(eliminado);
        lista->longitud--;
    }
}
```

Hablando de listas vacías, podríamos hacer un método que nos devuelva **verdadero** o **falso** (en C sólo hay 1 y 0) según si la lista está vacía o no. Por ejemplo, podemos devolver `lista->cabeza == NULL`:

```c
int EstaVacia(Lista* lista) {
    return lista->cabeza == NULL;
}
```

Además, vamos a ver, por ejemplo, para eliminar el último elemento de la lista.

Una vez más, tenemos que recorrer la lista hasta colocarnos en el elemento que nos interesa.
Primero, tenemos que comprobar que la lista no esté vacía.

Tenemos que colocarnos al final de la lista, no en el último elemento porque para eso, luego tenemos que hacer (como vimos en la presentación),
es eliminar el puntero que hay en el penúltimo elemento, para que el penúltimo sea el nuevo último.
Así que lo que tenemos que hacer es recorrer nuestra lista, pero en vez de detenernos en el último elemento, detenernos en el penúltimo, lo
sabremos cuando el `puntero-siguiente->siguiente` valga nulo. Pero mientras no hayamos llegado, continuamos avanzando.

Una vez hayamos llegado, eliminamos el nodo. Ya que estamos en el penúltimo, hacemos `puntero->siguiente = NULL`, recordando que tenemos
que mantener la referencia al nodo que vamos a eliminar, por ejemplo, con `Nodo* elimnado = puntero->siguiente`, para poder hacer la operación
`DestruirNodo(eliminado)` y eliminar el nodo que estamos eliminando.

Una vez eliminado, reducimos la `longitud` de la lista y listo.

```c
void EliminarUltimo(Lista* lista) {
    if (lista->cabeza) {
        Nodo* puntero = lista->cabeza;
        while (puntero->siguiente->siguiente) {
            puntero = puntero->siguiente;
        }
        Nodo* eliminado = puntero->siguiente;
        puntero->siguiente = NULL;
        DestruirNodo(eliminado);
        lista->longitud--;
    }
}
```

Ahora, solo nos falta una situación, y es el caso cuando la lista solo tiene un elemento. Porque si la lista solo tiene un elemento, no nos
podemos posicionar en el anterior, porque la lista es a la vez primero y último.

Así que tenemos que evaluar un caso especial, el de que la lista tenga solo un elemento, es decir, no tenga `siguiente` ya que no podemos pasar.
Tenemos que hacer una operación más sencilla, que es hacer que la lista vuelva a estar vacía eliminando la `cabeza` de la lista.
Hacemos que la lista valga **nulo**, después, destruimos la referencia a la `cabeza` y después decrementamos su `longitud`

```c
void EliminarUltimo(Lista* lista) {
    if (lista->cabeza) {
        if (lista->cabeza->siguiente) {
            Nodo* puntero = lista->cabeza;
            while (puntero->siguiente->siguiente) {
                puntero = puntero->siguiente;
            }
            Nodo* eliminado = puntero->siguiente;
            puntero->siguiente = NULL;
            DestruirNodo(eliminado);
            lista->longitud--;
        } else {
            Nodo* eliminado = lista->cabeza;
            lista->cabeza = NULL;
            DestruirNodo(eliminado);
            lista->longitud--;
        }
    }
}
```

De este modo, si la lista solo tiene un elemento, eliminamos ese último elemento.

Por último, veremos como eliminar un elemento cualquiera de la lista.

Vamos a eliminar el elemento número **n** de nuestra lista. En este caso, no se trata de ponernos ni al final ni al principio de la lista, se
trata de ponernos a eliminar un nodo cualquiera.

Como siempre, la lista tiene que tener una `cabeza`, porque si la lista está vacía, no hay nada que eliminar.
En la presentación, vimos que cuando eliminamos un elemento cualquiera de una lista, tenemos que establecer una conexión entre el **anterior**
que vamos a eliminar y el **siguiente** que vamos a eliminar. Si los conectamos saltándonos el elemento que pretendemos eliminar, ya está, ya
no forma parte de nuestra lista.

Y eso significa que tenemos que recorrer la lista, empezando por la `cabeza` y terminando en el anterior elemento al que queremos eliminar.
Así que una vez mas, vamos a recurrir a una variable que mantega el contador y esa variable va a tener que ir pasando de puntero a puntero.
Por ejemplo, `puntero = puntero->siguiente` y `posicion` va a ir incrementándose. Y nos debemos detener cuando hayamos llegado al nodo `n-1`, porque tenemos que colocarnos en el nodo anterior.
Si `posicion < (n - 1)`, no hemos llegado. En el momento que valga `n - 1` es que ya estamos en el anterior.

Una vez esté en el anterior, tenemos que eliminar su `siguiente`, así que lo primero que hacemos es marcar su nodo `siguiente` como nodo que
vamos a eliminar, y después conectar el `siguiente` del anterior, al `siguiente` del nodo que vamos a eliminar, de este modo, ya lo hemos
saltado.

Ahora, destruimos el nodo que marcamos para eliminar, y finalmente decrementamos la `longitud` de la lista.

```c
void ElimianrElemento(int n, Lista* lista) {
    if (lista->cabeza) {
        Nodo* puntero = lista->cabeza;
        int posicion = 0;
        while (posicion < (n - 1)){
            puntero = puntero->siguiente;
            posicion++;
        }
        Nodo* eliminado = puntero->siguiente;
        puntero->siguiente = eliminado->siguiente;
        DestruirNodo(eliminado);
        lista->longitud--;
    }
}
```

Pero existen 2 casos que no hemos tratado. Uno de ellos es el caso de que nos podan eliminar el elemento número 0, ya que no podemos
posicionarnos en el anterior, ya que es el primero, no tiene nada delante.

Tenemos que trata el caso especial de que `n == 0`, y eliminaremos el primer elemento, y es bastante sencillo, ya que es similiar a
`EliminarPrincipio()`. Entonces, si nos piden eliminar el primer elemento, simplemente lo eliminamos:

```c
void ElimianrElemento(int n, Lista* lista) {
    if (lista->cabeza) {
        if (n == 0) {
            Nodo* eliminado = lista->cabeza;
            lista->cabeza = lista->cabeza->siguiente;
            DestruirNodo(eliminado);
            lista->longitud--;
        } else {
            Nodo* puntero = lista->cabeza;
            int posicion = 0;
            while (posicion < (n - 1)){
                puntero = puntero->siguiente;
                posicion++;
            }
            Nodo* eliminado = puntero->siguiente;
            puntero->siguiente = eliminado->siguiente;
            DestruirNodo(eliminado);
            lista->longitud--;
        }
    }
}
```

El otro caso, es que nos pidan eliminar el elemento, por ejemplo, 15, de una lista que tenga 5 libros.

Si nos piden eliminar un libro que no existe porque no tenemos tantos, tampoco podemos hacerlo. Así que tenemos que establecer siempre una
comprobación para asegurarnos de que `n` está en el rango de nuestra lista.

Es muy sencillo, tenemos que asegurarnos de que `n < lista->longitud`:

```c
void EliminarElemento(int n, Lista* lista) {
    if (lista->cabeza) {
        if (n == 0) {
            Nodo* eliminado = lista->cabeza;
            lista->cabeza = lista->cabeza->siguiente;
            DestruirNodo(eliminado);
            lista->longitud--;
        } else if (n < lista->longitud) {
            Nodo* puntero = lista->cabeza;
            int posicion = 0;
            while (posicion < (n - 1)){
                puntero = puntero->siguiente;
                posicion++;
            }
            Nodo* eliminado = puntero->siguiente;
            puntero->siguiente = eliminado->siguiente;
            DestruirNodo(eliminado);
            lista->longitud--;
        }
    }
}
```

Esto es porque, si por ejemplo, tenemos 4 libros en nuestra lista (0, 1, 2, 3), si nos piden eliminar el libro `n = 4`, no podemos porque no
tenemos tantos libros. Así que siempre tenemos que asegurarnos de que `n` esté en los límites de nuestra lista.

## Conclusiones

Una vez hecho esto, lo único que tenemos que hacer es asegurarnos que todos los elementos están agregados en la cabecera:

```c
#ifndef lista_h
#define lista_h

#include <stdio.h>
#include "libro.h"

typedef struct Nodo {
    Libro libro;
    struct Nodo* siguiente;
} Nodo;

typedef struct Lista {
    Nodo* cabeza;
    int longitud;
} Lista;

Nodo* CrearNodo(Libro* libro);

void DestruirNodo(Nodo* nodo);

void InsertarPrincipio(Lista* lista, Libro* libro);

void InsertarFinal(Lista* lista, Libro* libro);

void InsertarDespues(int n, Lista* lista, Libro* libro);

Libro* Obtener(int n, Lista* lista);

int Contar(Lista* lista);

int EstaVacia(Lista* lista);

void EliminarPrincipio(Lista* lista);

void EliminarUltimo(Lista* lista);

void EliminarElemento(int n, Lista* lista);

#endif /* lista_h */
```

Para poder utilizar esta lista de forma útil fuera nuestra **clase**, más bien **módulo**, porque esto es un módulo de C, no es una clase de
C++ ni nada por el estilo.