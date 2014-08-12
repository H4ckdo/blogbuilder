---
layout: post
title:Clousures en javascript
---

#Clousures en javascript
Antes de comenzar con explicaciones muy tecnicas acerca de los clousures es importante a repasar primero el concepto del "alcance"
de las variables para que entiendas el por que de los clousures.

Javascript limita el alcance de las variables a el ambito en donde fueron creadas
por ejemplo: una variable global tiene un alcance global por que fue creada en un ambito global en cambio una variable local tiene un alcance local por que fue creada dentro de un ambito local como lo es una funcion, para que esto no se haga tan abstracto y puedas entender bien a lo que me refiero aqui hay una imagen que lo ilustra bien:   

![alt tag](https://pbs.twimg.com/media/BuzfdMiIIAA0yN_.jpg:large)

en esta imagen podemos apreciar que hay una funcion que tiene anidada otra funcion cada una tiene su respectiva variable y el alcance de cada variable esta limitado con las flechas hasta el cierre de su funcion padre y la variable global creada en lo mas alto del codigo tienen un alcance global y se puede accerder a su valor desde cualquier parte del codigo muy sencillo verdad ? entonces comencemos lo interesante los clousures.

Entonces que eso ? para que sirve? como se come ?

Bien te lo explico con un ejemplo imaginemos que necesitamos el valor de una variable fuera de funcion padre, es decir fuera de su alcance.

```javascript
  function funcionPadre (){
  	  var local="soy local y solo exito aqui";//local
  
  }
  
  //necesito el valor de la variable local en esta linea 

```
para "extender el alcance" necesitamos un closure que funciona como una funcion que tiene acceso a las variable de su funcion padre, en ese caso usaremos una funcion anonima retornando la 


```javascript
  function funcionPadre(){
  	  var local="soy local y solo exito aqui";//local
      return function(){
         return local      
      }
  }
  
  var resultado=funcionPadre()();
  //llama la funcionPadre que retorna una funcion anonima que a su vez retorna el valor  
  //que necesitamos y guardamos el resultado en la variable resultado.
  console.log(resultado);//soy local y solo exito aqui
```
Con el codigo anterios "extendimos el alcance de la variable local" pero tal vez te preguntes por que necesitamos colocar dos parentesis para llamar a la funcion ?

```javascript
var resultado=funcionPadre()();

```
El primer parentesis llama a la funcionPadre que retorna una funcion anonima y para llamar dicha funcion anonima es que colocamos se segundo parentesis, puedes intentar lo colocando solo un parentesis y te retornara veras algo como esto:

```javascript
var resultado=funcionPadre();
console.log(resultado);//[function]
```
Que tal si hacemos un pequeño experimento y le pasamos como parametro algo a la funcion anonima y lo adjutamos con la funcion local 


```javascript
  function funcionPadre(){
  	  var local="soy local y solo exito aqui";//local
      return function(Extra){
         return local+Extra       
      }
  }
  
  var resultado=funcionPadre()(" :esto es data extra :p");
  //llama la funcionPadre que retorna una funcion anonima que a su vez retorna el valor  
  //que necesitamos y guardamos el resultado en la variable resultado.
  console.log(resultado);//soy local y solo exito aqui :esto es data extra :p
```

Hagamos otro ejemplo un poco mas interesante, imagaina que en un ciclo for necesitamos

cada iteracion se demore 5 segundos

```javascript
  for(var i=0;i<30;i++){
 	console.log(i);
    //espera 5 segundos 
   }
   
```
Lo mas logico seria utilizar un "setTimeout" para que se quede esperando por 5 segundos a 

```javascript
  for(var i=0;i<30;i++){
 	console.log(i);//0 1 2 3 4 ....29
    setTimeout(function(){
       console.log(i);//30 30 30 .... 
    },5000);
  } 
  console.log("termina ciclo");
```
Entonces por que no funciono como queriamos y simplemente mostro 30, fijate que la condicion de el ciclo dice que la iteracion va a para cuando i < 30 es decir debio haber terminado en la  iteracion 29.

La manera mas facil de resolver el problema partirlo en 2 insisos

* Por que setTimeout se comporta asi ?
* Por que el valor de i es 30 en el setTimeout ?


Por que setTimeout se ejecuta despues de que termina el ciclo

para responder esto es necesario que comprendas como es que funciona un "setTimeout" en PHP existe una intruccion similar al "setTimeout" llamada "sleep" vamos a emularla en JS para mostrarte como es que funcionan esta funciones de timepo.


```javascript
 function  sleep(delay,cb){
   var T_futuro=new Date().getTime()+delay//toma la hora en milisegundos y le suma el 
   //retraso que queremos es decir los 5 segundos 
   
   while(  new Date().getTime() < T_futuro );//no hace nada hasta que pase los 5 segundos
   return cb();
 }

 sleep(5000,function(){
   console.log("terminaron los 5 segundos de espera");
 });

```
Entonces en una forma similar funciona el "setTimeout" y la razon por la cual 
decidi mostrarte este ejemplo es por esta parte 

```javascript
 var T_futuro=new Date().getTime()+delay;
 while(  new Date().getTime() < T_futuro );//no hace nada hasta que pase los 5 segundos

```
este ciclo hace que mientras no se cumpla la condicion de el ciclo el programa va a avanzar y nada de lo que este abajo se va a ejecutar, fitare que la condicion dice que que la hora debe ser mejor a el tiempo de incio mas el retraso 'delay' la clave esta en el la variable 

```javascript
 var T_futuro=new Date().getTime()+delay;
```

el "setTimeout()" dentro del ciclo es invocado en cada itereacion entonces en cada iteracion toma una hora diferente para solucionar lo basta con multiplicar i por el 'delay'



```javascript
var delay=5000;

for(var i=0;i<30;i++){
	setTimeout(function(){
		console.log("espero 5s");
	},delay*i);

//5000*i por cada iteracion esta operacion retorna una secuencia como
//5000*0=0 , 5000*1=5000, 5000*2=10000 .... 
}
```
con aumentar el tiempo de espera en cada iteracion solucionamos el problema del comportamiento del "setTimeout", pero por que en cada iteracion el tiempo aumenta pero 
todo parece ejecutarse cada 5 segundos ?

recuerdas esta variable esta es la culpable el toma la hora en milisegundos por cada iteracion entonces el tiempo futuro siempre va ha ser 5 segundos 

```javascript
 var T_futuro=new Date().getTime()+delay;
```
Esto es un poco complejo de entender asi que si no me entiendes no te desesperes y repasa el codigo parte por parte.

Bien ya solucionamos un insiso falta el otro

¿ Por que el valor de i es 30 en el setTimeout ?
  
  El problema reside en que al momento en que queremo mostrar i dentro del "setTimeout" el ciclo for a terminado y ese 
 

```javascript
for(var i=0;i<20;i++){
	setTimeout(function(){
		console.log(i);
	},5000*i);
}
```


ToDo TERMINA DE ESCRIBIR EL POST