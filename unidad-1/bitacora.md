# Unidad 1


## Escribe el programa.

@12

M=0

@13

M=1

(LOOP)

@13

D=M

@5

D=D-A

@END

D;JGT

@13

D=M

@12

M=D+M

@13

M=M+1

@LOOP

0;JMP

(END)

@END

0;JMP






## Simula paso a paso. Recuerda la metodología: predice, ejecuta, observa y reflexiona.

<img width="414" height="580" alt="Captura de pantalla 2026-01-29 172909" src="https://github.com/user-attachments/assets/e1cae010-b5c3-49bc-89a5-4e1ebd0b2467" />



<img width="413" height="678" alt="Captura de pantalla 2026-01-29 173724" src="https://github.com/user-attachments/assets/3c82d3f5-f05e-48b4-ace0-47a4da73be81" />




## Actividad 05



1.Describe con tus palabras las tres fases del ciclo Fetch-Decode-Execute. ¿Qué rol juega el Program Counter (PC) en este ciclo?


El ciclo Fetch–Decode–Execute es la forma en que la computadora ehecuta lo que tiene paso a paso.

Primero (Fetch), la computadora busca la siguiente instrucción.
Luego (Decode), trata de entender qué le están pidiendo que haga.
Por último (Execute), es el que ejecuta la accion.



2.¿Cuál es la diferencia fundamental entre una instrucción-A (que empieza con @) y una instrucción-C (que involucra D, M, A, etc.) en el lenguaje ensamblador de Hack? Da un ejemplo de cada una.


La instrucción A es la que empieza con @ y se usa para decirle a la computadora con qué numero se va a trabajar.

La instrucción C es la que hace acciones, como guardar datoso sumar..


3.Explica la función de los siguientes componentes del computador Hack: el registro D, el registro A y la ALU.


El registro D se usa para guardar datos por un tiempo.

El registro A se usa para guardar numeros que se van a utilizar

La ALU es la parte que hace las operacione.


4.¿Cómo se implementa un salto condicional en Hack? Describe un ejemplo (p. ej., saltar si el valor de D es mayor que cero).


Un salto condicional se hace cuando se revisa un valor y decide si saltarselo o no.

Por ejemplo, si quiero saltar cuando el valor en D es mayor que cero, primero se calcula ese valor y luego se usa una instrucción de salto como:D;JGT


5.¿Cómo se implementa un loop en el computador Hack? Describe un ejemplo (p. ej., un loop que decremente un valor hasta que llegue a cero).


Un loop se hace usando una etiqueta (LOOP) y un salto  0;JMP. Asi se repite una y otra vez hasta completar una funcion.



6.¿Cuál es la diferencia entre la instrucción D=M y la instrucción M=D?


La instrucción D=M significa que el valor de la memoria le da al registro D.

La instrucción M=D significa que el valor del registro D se guarda en la memoria.



7.Describe brevemente qué se necesita para leer un valor del teclado (KBD) y para “pintar” un pixel en la pantalla (SCREEN).


Para leer el teclado, se usa la dirección especial KBD, que guarda el valor de la tecla que se está presionando.

