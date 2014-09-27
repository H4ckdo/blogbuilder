---
layout: post
comments: true
title: Closures en Javascript

#Author
author: esneyder_palacios
---

Antes de comenzar con explicaciones muy técnicas acerca de los closures es importante repasar primero el concepto del "alcance" 
de las variables para que entiendas el por qué de los closures.

Javascript limita el alcance de las variables a el ámbito en donde fueron creadas
por ejemplo: una variable global tiene un alcance global por que fue creada en un ámbito global en cambio una variable local tiene un alcance local por que fue creada dentro de un ámbito local como lo es una función, para que esto no se haga tan abstracto y puedas entender bien a lo que me refiero, aquí hay una imagen que lo ilustra bien: 

![alt tag](https://pbs.twimg.com/media/BuzfdMiIIAA0yN_.jpg:large)

En esta imagen podemos apreciar que hay una función que tiene anidada otra función cada una tiene su respectiva variable y el alcance de cada variable esta limitado con las flechas hasta el cierre de su función padre y la variable global creada en lo mas alto del código tiene un alcance global y se puede acceder a su valor desde cualquier parte del código muy sencillo verdad ? entonces comencemos lo interesante los closures.

¿Entonces que es eso? ¿Para qué sirve? ¿Cómo se come?

Bien te lo explico con un ejemplo: imagina que necesitamos el valor de una variable por fuera de su función padre, es decir fuera de su alcance.

{%highlight javascript linenos%}
  function funcionPadre(){
     var local="soy local y solo exito aqui";//local
  }
  //necesito el valor de la variable local en esta linea 
{%endhighlight%}

para "extender el alcance" necesitamos un closure que se comporta como una función anidada dentro otra función que llamaremos función padre entonces la función anidada tiene acceso a las variables de su función padre, en este caso usaremos una función anónima como función anidada "closure"


{%highlight javascript linenos%}
  function funcionPadre(){
      var local="soy local y solo exito aqui";//local
      return function(){
         return local      
      }
  } 
  var resultado=funcionPadre()();
  //llama la funcionPadre que retorna una función anónima que a su vez retorna el valor que necesitamos y guardamos el resultado en la variable resultado
  console.log(resultado);//soy local y solo exito aqui
{%endhighlight%}

Con el código anterior "extendimos el alcance de la variable local" pero tal vez te preguntes ¿por qué necesitamos colocar dos paréntesis para llamar a la función?

{%highlight javascript%}
var resultado=funcionPadre()();
{%endhighlight%}

El primer paréntesis llama a la ```funcionPadre``` que retorna una función anónima y para llamar dicha función anónima es que colocamos se segundo paréntesis, puedes intentar lo colocando solo un paréntesis y te retornara algo como esto:

{%highlight javascript%}
var resultado=funcionPadre();
console.log(resultado);//[function]
{%endhighlight%}

Que tal si hacemos un pequeño experimento y le pasamos como parametro algo a la función anonima y lo adjutamos con la función local

{%highlight javascript linenos%}
  function funcionPadre(){
      var local="soy local y solo exito aqui";//local
      return function(Extra){
         return local+Extra       
      }
  }

  var resultado=funcionPadre()(" :esto es data extra :p");
  //llama la funcionPadre que retorna una función anónima que a su vez retorna el valor
  //que necesitamos y guardamos el resultado en la variable resultado.
  console.log(resultado);//soy local y solo exito aqui :esto es data extra :p
{%endhighlight%}

Hagamos otro ejemplo un poco mas interesante imagina que en un ciclo for necesitamos que entre cada iteracion  haya un retraso de 5 segundos

{%highlight javascript linenos%}
 for(var i=0;i<30;i++){
  console.log(i);
    //espera 5 segundos 
   }
{%endhighlight%}

Lo mas lógico seria utilizar un "setTimeout" para que se quede esperando por 5 segundos entre cada iteracion 

{%highlight javascript linenos%}
for(var i=0;i<30;i++){
  console.log(i);//0 1 2 3 4 ....29
    setTimeout(function(){
       console.log(i);//30 30 30 .... 
    },5000);
} 
  console.log("termina ciclo");
{%endhighlight%}

Entonces por qué no funciono como queríamos y simplemente mostró 30? Fíjate que la condición del ciclo dice que la iteración va hasta cuando i < 30 es decir debió  terminar en la iteración 29.

La manera mas fácil de resolver el problema es partirlo en 2 incisos:

* ¿Por qué "setTimeout" se comporta así?
* ¿Por qué el valor de i es 30 dentro de "setTimeout"?


##¿Por qué "setTimeout" se comporta así?

Para responder esto es necesario que comprendas como funciona el "setTimeout", en PHP existe una instrucción similar al "setTimeout" llamada "sleep" vamos a emularla en JS para mostrarte como es que funcionan estas funciones de tiempo.


{%highlight javascript linenos%}
 function  sleep(delay,cb){
   var T_futuro=new Date().getTime()+delay//toma la hora en milisegundos y le suma el 
   //retraso que queremos es decir los 5 segundos  
   while(  new Date().getTime() < T_futuro );//no hace nada hasta que pase los 5 segundos
   return cb();
 }
 sleep(5000,function(){
   console.log("terminaron los 5 segundos de espera");
 });
{%endhighlight%}

Entonces en una forma similar funciona el ```setTimeout()``` internamente, la razón por la cual 
decidí mostrarte este ejemplo es por esta parte del codigo:

{%highlight javascript%}
 var T_futuro=new Date().getTime()+delay;
 while(  new Date().getTime() < T_futuro );//no hace nada hasta que pase los 5 segundos
{%endhighlight%}

Este ciclo hace que mientras no se cumpla la condición el programa no va a avanzar y nada de lo que este abajo se va a ejecutar, fíjate que la condición dice que la hora debe ser menor a el tiempo de inicio más el retraso 'delay' la clave esta en la variable 'T_futuro'

{%highlight javascript%}
 var T_futuro=new Date().getTime()+delay;
{%endhighlight%}

Esto hace que siempre que se llama a la función 'sleep' va tomar la hora y le suma 'delay', entonces esto quiere decir que siempre van hacer 5 segundos de retraso en este caso.

Con esto ya podemos volver al codigo y encontrar una solucion a nuestro problema ```setTimeout()``` es invocado en cada iteración y cada iteración toma una hora diferente para solucionarlo, basta con multiplicar i por el 'delay' esto va ha generar un tiempo en el que siempre hay 5 segundos de retraso en este caso.

{%highlight javascript linenos%}
var delay=5000;
for(var i=0;i<30;i++){
  setTimeout(function(){
    console.log("espero 5s");
  },delay*i);
//5000*i por cada iteracion esta operacion retorna una secuencia como
//5000*0=0 , 5000*1=5000, 5000*2=10000 .... 
}
{%endhighlight%}

Antes no funcionaba como queriamos por que siempre habia un tiempo fijo con aumentar el tiempo de espera en cada iteración solucionamos el problema del comportamiento del ```setTimeout()```, pero ¿Todo parece ejecutarse en tiempos distintos?

Recuerdas esta variable 'T_futuro' 

{%highlight javascript linenos%}
 var T_futuro=new Date().getTime()+delay;
{% endhighlight %}

Es la culpable y se encarga de que el retraso sea siempre 5 segundos, esto es un poco complejo de entender así que si no me entiendes no te desesperes y repasa el código parte por parte.

Bien ya solucionamos un inciso falta el otro

##¿Por qué el valor de i es 30 dentro de "setTimeout"?

Es porque al momento en que queremos mostrar i dentro del ```setTimeout()``` el ciclo for a terminado, pero cuando se ejecuta el "setTimeout" el ciclo "revive" y le incrementa 1 por el momento en que se ejecuta. Notas que ahora cuando imprimimos i dentro del "setTimeout" muestra 29 y ya no muestra mas 30, ahora no lo hace por que el tiempo entre cada iteración aumenta como debe ser al ritmo del "setTimeout"

{%highlight javascript linenos%}
for(var i=0;i<30;i++){
  setTimeout(function(){
    console.log(i);//29 29 29 ...
  },5000*i);
}
{% endhighlight %}
Necesitamos mantener la existencia de i en cada iteración para eso podemos hacer lo siguiente

{%highlight javascript linenos%}
for (var i = 0; i < 30 ; i++) {
    setTimeout(function(e) { 
      return function(){
       console.log(e); //0 1 2 3 ....
      }; 
    }(i), 5000*i);
}
{%endhighlight%}

Podemos mantener el valor de i usando una función anónima que se auto invoca una ["IIFE"](http://bentoncoding.com/2012/09/18/using-the-immediately-invoked-function-expression/)
en este caso el problema no era que no tuviéramos a la variable i a nuestra disposición si no que la referencia de i no era igual a el valor de i en cada iteracion, por que como digimos antes el ciclo
habia terminado cuando intentabamos llamarla.

En conclusión un closure es una función que puede acceder a las variables que están dentro de su alcance y podemos hacer cosas como retornar esa variable o hacer lo que necesitamos, en este caso mantuvimos el valor de i intacto siempre con la ayuda de un closure para i.