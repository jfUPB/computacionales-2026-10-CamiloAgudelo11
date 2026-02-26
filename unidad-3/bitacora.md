# Unidad 3


## Bitácora de aplicación 


### 1.Diagnóstico del problema (análisis):

Compila y ejecuta el código.

Identifica y explica con detalle al menos dos errores críticos de gestión de memoria en la clase Personaje y su uso en simularEncuentro.

Para cada error, describe:

¿Cuál es el error?

#### Error 1

Hay una fuga de memoria en la parte del codigo   estadisticas = new int[3]; pero en ningun momento hay un destructor para "estadisticas" quedando asi memoria sin usar

#### Error 2

Hay una copia peligrosa en la parte del codigo heroe.estadisticas  
copiaHeroe.estadisticas  esto significa los dos apuntan al mismo arreglo en el heap

¿Por qué ocurre? Explica el mecanismo a nivel de memoria (stack, heap, punteros)

#### Error 1


Cuando usamos new la memoria se reserva en el heap pero en el heap no se libera automaticamente entonces cuando la funcion termina el heroe se destruye por que esta en el stack pero memoria no por que esta en heap entonces queda memoria reservada sin usar

#### Error 2

Por que "estadisticas" es un puntero y cuando se copia, no se copia el arreglo solo la direccion

¿Cuál es su consecuencia?

#### Error 1

Si esto ocurre muchas veces dentro de un videojuego cada vez se va a consumir mas RAM y despues crashear

#### Error 2

Si se agrega el destructor de "estadisticas" entonces se destruye la copiaHeroe y heroe lo que daria un crasheo


USA EL DEPURADOR:


### 2.Captura pantallas y luego explica inmediatamente lo que observas y analizas para evidenciar cada uno de los errores que identificaste.


#### Error 1

<img width="1011" height="360" alt="image" src="https://github.com/user-attachments/assets/1206ca04-fb96-4d1b-8e52-33e11d9b13d8" />

Se puede evidenciar que se reserva memoria dinamica en el heap mediante new, pero en la clase no existe un destructor que libere esta memoria generando una fuga de memoria .


#### Error 2

<img width="938" height="223" alt="image" src="https://github.com/user-attachments/assets/4c5bec66-7141-4535-9bdd-9ba6e175f906" />

Despues de realizar la copia del objeto se observa que tanto heroe y copiaHeroe poseen el mismo valor en el puntero estadisticas esto demuestra una copia superficial ya que ambos apuntan a la memoria heap


### 3.Induce los fallos:


Trata de quebrar el programa de manera controlada para evidenciar los errores. No olvides capturas de pantalla del DEPURADOR para evidenciar lo que ocurre en memoria.


<img width="1918" height="1014" alt="Captura de pantalla 2026-02-26 150528" src="https://github.com/user-attachments/assets/5a7410ea-51c3-4d6d-b025-fcfe37cda246" />



<img width="1915" height="1018" alt="Captura de pantalla 2026-02-26 150604" src="https://github.com/user-attachments/assets/ca0cef24-b5a5-495a-8ec3-934cf329d348" />



### 4.Solución y refactorización (síntesis y creación):

Re-escribe la clase Personaje para que sea segura en cuanto a memoria. Debes utilizar los conocimientos adquiridos en esta unidad y por tanto tu solución no debería usar la Regla de los tres que probablemente sea la solución que te ofrezca una IA.
Presenta el código completo de tu clase Personaje corregida.




```cpp

#include <iostream>
#include <string>

class Personaje {
public:
    std::string nombre;
    int estadisticas[3];  

    
    Personaje(std::string n, int vida, int ataque, int defensa) {
        nombre = n;
        estadisticas[0] = vida;
        estadisticas[1] = ataque;
        estadisticas[2] = defensa;
        std::cout << "Constructor: nace " << nombre << std::endl;
    }

    void imprimir() {
        std::cout << "Personaje " << nombre
            << " [Vida: " << estadisticas[0]
            << ", ATK: " << estadisticas[1]
            << ", DEF: " << estadisticas[2]
            << "]" << std::endl;
    }
};


```

### 5.Justificación de la Solución:

Explica por qué cada uno de los cambios que añadiste resuelven los problemas que diagnosticaste.

La clase original usaba una memoria dinamica lo que daba la fuga de memoria y la copia superficial que traia problemas 

int* estadisticas;


estadisticas = new int[3];



El cambio que realice fue eliminar esta memoria y se reemplazo el puntero por un arreglo interno, tambien se elimino el uso de new y delete, utilizando un almacenamineto automatico


int estadisticas[3];


## Bitácora de reflexión
