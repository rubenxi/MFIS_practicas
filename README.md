Práctica 2: Filósofos
El problema de los filósofos, o de los filósofos que cenan (dining philosophers), es un problema clásico
propuesto por E. Dijkstra en 1965 para modelar el problema de la sincronización de procesos en un
sistema operativo. Los filósofos están sentados a una mesa circular, y se pasan la vida comiendo y
pensando. Para poder comer, necesitan coger los dos palillos junto a su plato. El problema surge porque
estos palillos son recursos compartidos: el palillo que queda a la derecha de un filósofo es el palillo a la
izquierda de su vecino de la derecha.
Figura 1: Filósofos cenando
Ejercicio 1: Especificación del problema de los filósofos
Especifica el problema de los filósofos como una teoría de reescritura en Maude en un fichero
dining-philosophers.maude siguiendo las siguientes indicaciones.
Aunque normalmente el problema se plantea para cinco filósofos, en realidad podemos definirlo
para cualquier número de filósofos, siendo dicho número un parámetro de nuestra especificación.
Para ello definimos la siguiente teoría NAT*:
fth NAT* is
protecting NAT .
op k : -> NzNat .
endfth
En esta teoría se están protegiendo los naturales, de forma que sus posibles modelos son aquellos
en los que tenemos el modelo inicial de NAT y la constante k se interpreta como cualquier valor
de tipo NzNat.
Si tenemos N filósofos, utilizaremos números entre 0 y N-1 para identificar filósofos y palillos.
Y esto nos permitirá identificar los palillos que le corresponden a cada filósofo: el filósofo I
podrá usar los palillos I e I+1 módulo N. Para sistematizar esta operación, definimos un módulo
parametrizado NAT/{k :: NAT*} con un tipo Nat/{k} de naturales módulo k con una operación
[_] que transforma cualquier número natural a su correspondiente número módulo k.
fmod NAT/{k :: NAT*} is
sort Nat/{k} .
op [_] : Nat -> Nat/{k} .
var N : Nat .
ceq [N] = [sd(N, k)] if N >= k .
endfm
A continuación definiremos un módulo DINING-PHILOSOPHERS{P :: NAT*} que importa (en
modo protecting) el módulo NAT/{P} en el que especificaremos el problema de los filósofos para
cualquier número de filósofos.
• Define un tipo Status con tres posibles valores: thinking, hungry e eating.
1
• Crea tipos Philosopher y Chopstick para representar filósofos y palillos, con constructores
op philosopher : Nat/{P} Status Nat -> Philosopher .
op chopstick : Nat/{P} -> Chopstick .
Una mesa se representará como un conjunto de filósofos y palillos de tipo Configuration,
con none como conjunto identidad de la unión de conjuntos definida con el operador __. El
primer argumento de los operadores philosopher y chopstick representa su identificador.
Los otros dos argumentos del operador philosopher representan su estado y el número de
palillos que tiene cogidos en un momento dado (un valor entre 0 y 2). Cuando un palillo
está libre tenemos un objeto de tipo Chopstick en el conjunto; cuando un filósofo toma un
palillo libre, este desaparece del conjunto y el número de palillos del filósofo que lo toma se
incrementa en uno. Cuando lo suelta, el palillo vuelve a aparecer en la configuración y el
contador del filósofo se decrementa.
• Cuatro reglas especifican las acciones que pueden ocurrir en nuestro sistema:
◦ get-hungry: un filósofo en estado thinking pasa a estado hungry.
◦ grab-stick: un filósofo en estado hungry coge un palillo libre.
◦ eat: un filósofo en estado hungry con dos palillos pasa a estado eating.
◦ think: un filósofo en estado eating pasa a estado thinking y libera sus dos palillos.
• Define un operador initState (sin argumentos) que cree una mesa con k filósofos en estado
thinking y sus correspondientes palillos libres.
Para instanciar el módulo DINING-PHILOSOPHERS necesitamos una vista desde la teoría NAT*
hasta un módulo que importe NAT en modo protecting. Esta vista debe indicar cómo instanciamos
la constante k. Para hacerlo debemos asociar a k, no un operador, sino un término, y para ello
utilizamos la sintaxis op OP to term TERM ., para interpretar k como 5 como sigue:
view 5 from NAT* to INT is
op k to term 5 .
endv
Con esto ya estamos listos para crear un módulo en el que analizar el problema.
mod DINING-PHILOSOPHERS-5 is
pr DINING-PHILOSOPHERS{5} .
endm
[Q1] ¿Es la teoría de reescritura terminante? ¿Es coherente? ¿Es el espacio de búsqueda alcanzable a
partir de initState finito? Justifica tu respuesta. Prueba a utilizar el comando reduce y el comando
rewrite para reescribir el término initState usando ecuaciones, y ecuaciones y reglas, respectivamente.
[Q2] Utiliza el comando search para buscar un estado de bloqueo. Utiliza los comandos show path y
show path labels para conocer la secuencia de pasos seguida hasta dicho estado de bloqueo.
Crea un módulo DINING-PHILOSOPHERS-PREDS{P :: NAT*} que importe el módulo
DINING-PHILOSOPHERS{P} y defina proposiciones atómicas
op phil-status : Nat/{P} Status -> Prop .
op phil-sticks : Nat/{P} Nat -> Prop .
que se satisfagan si el filósofo con el identificador dado como primer argumento está en el estado
indicado como segundo argumento o tiene el número de palillos indicado, respectivamente.
Crea un módulo DINING-PHILOSOPHERS-5-CHECK con el que poder comprobar propiedades con
el comprobador de modelos para 5 filósofos.
[Q3] Utiliza el comprobador de modelos de Maude para encontrar un camino hasta el estado de bloqueo.
Como conocemos el estado de bloqueo (cada filósofo tiene un palillo), podemos encontrarlo verificando
2
la propiedad temporal que niega la existencia de dicho estado: siempre es verdad que no tenemos un
estado de bloqueo.
[Q4] Analiza el formato del contraejemplo devuelto como resultado y proporciona la secuencia de los
10 primeros pasos con el siguiente formato: el filósofo 1 coge el palillo 1, el filósofo 1 suelta el palillo 1,
el filósofo 2 coge el palillo 3, . . .
Que el sistema pueda bloquearse no es una buena noticia, y se han propuesto varias soluciones para
este problema. Especifiquemos a continuación tres de ellas.
Ejercicio 2: Jerarquía de recursos
Para evitar el problema del bloqueo, Dijstra propuso asignar un orden parcial a los palillos, estableciendo
la convención de que los palillos se cogerían en orden. Es decir, el filósofo [I] tomaría primero el palillo
con identificador menor entre [I] e [I + 1], y después el otro. Los palillos se sueltan a la vez. Esta
sencilla solución evita la situación de bloqueo de la solución original, pues, para cinco filósofos, sólo
cuatro de los cinco podrían coger su primer palillo. Observa que para N filósofos, los filósofos 0 y N-1
intentarían primero coger el palillo 0, rompiendo así el bloqueo.
Crea una copia del fichero del Ejercicio 1 en un fichero dining-philosophers-order.maude y modifí-
calo de forma que funcione según esta variación.
[Q5] Demuestra, utilizando el comando search, que la nueva especificación no tiene bloqueos.
[Q6] Demuestra, utilizando el comprobador de modelos, que la nueva especificación no tiene bloqueos.
Esta solución no es justa: Cualquiera de los filósofos puede quedarse sin comer aunque lo desee.
[Q7] Comprueba, utilizando el comprobador de modelos, que esta especificación no satisface la propiedad
de viveza débil ni la de viveza fuerte. Es posible que los contraejemplos de la comprobación de las
propiedades para los distintos filósofos tengan longitudes distintas, ¿por qué?
Ejercicio 3: Un gestor de procesos
Otra solución consiste en limitar el número de comensales sentados a la mesa. Si tenemos una mesa
para N filósofos, la solución consistiría en dejar que sólo haya N-1 filósofos como máximo sentados a la
mesa en cada momento, de forma que cualquiera de ellos puede comer sin temor a bloqueos. Para ello,
contamos con un portero que controla el acceso. Mientras un filósofo está pensando, este está fuera de
la mesa (no está en la configuración que representa la mesa). El portero se encarga de controlar el
número de filósofos a N-1, es decir, un filósofo sólo puede sentarse a la mesa si no se ha superado el
límite de comensales en ella, y abandona la mesa cuando deja de comer. Para garantizar que ningún
filósofo se queda sin sentarse en la mesa si lo desea, tenemos que garantizar que el portero deja entrar
a los filósofos de forma justa. Podemos garantizarlo implementando una cola para los filósofos que
desean pasar a la mesa, de forma que cuando un filósofo pasa a estar hambriento se pone en una cola,
que eventualmente le dará acceso a la mesa.
Para ello, se definirá un tipo System con constructor
op [_,_,_] : Configuration Queue Configuration -> System .
que representa la mesa (primer argumento), la cola de filósofos hambrientos en espera de pasar a la
mesa (segundo argumento) y el conjunto de filósofos fuera de la mesa. Inicialmente, los palillos estarán
en la mesa (primer argumento), sólo filósofos en la mesa podrán acceder a ellos, y estos serán devueltos
a la mesa cuando terminan de ser usados antes de abandonar la mesa. La cola la implementaremos
como una lista de valores de tipo Nat/{P}, en la que los elementos se añaden por un extremo y se sacan
por el otro. Los filósofos en espera se mantendrán en la configuración del tercer argumento mientras se
les da paso.
Crea una copia del fichero del Ejercicio 1 en un fichero dining-philosophers-rooms.maude y modifí-
calo de forma que funcione según esta variación con las siguientes reglas:
3
- `get-hungry`: un filósofo en estado `thinking` pasa a estado `hungry` y se pone en la cola para pasar a la mesa
- `enter`: el primer filósofo en la cola pasa a la mesa si el número de filósofos en ella es menor que *N-1*.
- `grab-stick`: un filósofo en estado `hungry` coge un palillo libre.
- `eat`: un filósofo con en estado `hungry` con dos palillos pasa a estado `eating`.
- `think`: un filósofo en estado `eating` pasa a estado `thinking`, libera sus dos palillos y abandona la mesa.
[Q8] Demuestra, utilizando el comando search, que esta especificación no tiene bloqueos.
Para poder utilizar el comprobador de modelos necesitamos adaptar las definiciones de las proposiciones
atómicas en el módulo DINING-PHILOSOPHERS-PREDS al nuevo tipo de estado.
[Q9] Demuestra, utilizando el comprobador de modelos, que esta especificación no tiene bloqueos.
Esta solución, sin embargo, no es justa. Cualquiera de los filósofos puede quedarse sin comer aunque lo
desee.
[Q10] Comprueba, utilizando el comprobador de modelos, que esta especificación no satisface la
propiedad de viveza débil ni la de viveza fuerte.
[Q11] Analiza el contraejemplo devuelto en cualquiera de las comprobaciones de [Q10] para propor-
cionar su secuencia de pasos como en [Q4].
Ejercicio 4: Un gestor de recursos
Alternativamente, además de filósofos y palillos, en esta solución añadimos un camarero al sistema.
En este sistema, cuando un filósofo quiere comer pide permiso al camarero, y este lo encola para ser
atendido. Cuando los palillos del filósofo que lleva más tiempo esperando en la cola están disponibles, el
filósofo coge sus palillos y empieza a comer. Para poder especificar este comportamiento, utilizaremos
el tipo Queue de colas de valores de tipo Nat/{P} del ejercicio 2, y un tipo Waiter con constructor
op waiter : Queue -> Waiter .
Este operador mantiene la cola de filósofos en estado hungry a la espera de poder coger los palillos.
Observa que este camarero funcionará como centinela de la sección crítica, similar al MUTEX, aunque
en este caso sin múltiples tokens.
Crea una copia del fichero del Ejercicio 1 en un fichero dining-philosophers-waiter.maude y
modifícalo de forma que funcione según esta variación:
Cuando un filósofo pasa de estado thinking a hungry se le añade a la cola de espera del camarero.
Si están libres sus palillos, el primer filósofo en la cola de espera puede coger sus dos palillos y
empezar a comer.
Cuando un filósofo termina de comer, libera sus palillos.
Asegúrate de modificar la definición del estado inicial (initState) adecuadamente.
[Q12] Demuestra, utilizando el comando search, si esta especificación tiene bloqueos o no.
[Q13] Demuestra, utilizando el comprobador de modelos, si esta especificación tiene bloqueos o no.
[Q14] Demuestra, utilizando el comprobador de modelos, si esta especificación satisface las propiedades
de viveza débil y fuerte o no.
[Q15] Una de las principales dificultades para implementar los algoritmos concurrentes está en la
atomicidad de las operaciones. La solución en el ejercicio 3 consigue evitar bloqueos garantizando viveza
a costa de realizar en una única acción la comprobación de la existencia de palillos libres, su cogida
y el empezar a comer. Reflexiona sobre las ventajas e inconvenientes de cada una de las soluciones
propuestas y sobre la dificultad de implementación de operaciones atómicas de tanta complejidad.
4
