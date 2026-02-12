# Unidad 2

## Actividad 08


### Implementa ambos programas en lenguaje ensamblador.


#### Problema 1

(start)


@10

D=A

@16

M=D

@20

D=A

@17

M=D


@16

D=A

@R0

M=D

@17

D=A

@R1

M=D


@returnFromSwap

D=A

@R15

M=D


@swap

0;JMP


(returnFromSwap)

@fin

(fin)

0;JMP




(swap)


@R0

A=M

D=M

@R13

M=D


@R1

A=M

D=M

@R0

A=M

M=D



@R13

D=M

@R1

A=M

M=D


@R15

A=M

0;JMP



#### Problema 2


(start)



@10

D=A

@16

M=D


@15

D=A

@17

M=D

@2

D=A

@18

M=D

@3

D=A

@19

M=D

@50

D=A

@20

M=D



@16

D=A

@R0

M=D

@5

D=A

@R1

M=D


@returnFromCalSum

D=A

@R15

M=D


@calSum

0;JMP


(returnFromCalSum)

@R0

D=M

@21

M=D


@fin

(fin)

0;JMP




(calSum)


@0

D=A

@R13

M=D


@0

D=A

@R14

M=D

(loop)

@R14

D=M

@R1

D=D-M

@finLoop

D;JGE


@R0

D=M

@R14

A=D+M

D=M

@R13

M=D+M



@R14

M=M+1

@loop

0;JMP

(finLoop)


@R13

D=M

@R0

M=D

@R15

A=M

0;JMP



### Capturas de pantalla donde muestres una parte de la simulación y expliques qué se ve esa captura.


<img width="1546" height="809" alt="image" src="https://github.com/user-attachments/assets/cbe65fb2-cf1f-48ea-9993-d491412044e5" />


 La instruccion M=D esta a punto de guardar el valor 10 que esta en el registro D en la direccion de la RAM 16 que esta registrado en A


### Captura de pantalla del simulador mostrando el resultado final correcto.


<img width="1552" height="737" alt="image" src="https://github.com/user-attachments/assets/75885174-9c74-4f74-a9ef-0a7c16f79da8" />




<img width="1550" height="731" alt="image" src="https://github.com/user-attachments/assets/4ed1a1be-9d5b-44fe-be1b-9a36574c0822" />




##  Reflect


### Muestra el diseño que hiciste en Bitmap Editor.


### Incluye el código en ensamblador generado por Bitmap Editor.


### Incluye el programa completo en ensamblador que llama a la función generada por Bitmap Editor y que lee las teclas d y e para dibujar y borrar respectivamente el mapa de bits.


### Construye tu programa PASO A PASO mediante pruebas utilizando el simulador.


### Incluye capturas de pantalla donde muestres el resultado final de la aplicación.

