Semáforos
1)Resolver los problemas siguientes:
a)En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE en la terminal disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo con el orden de llegada. Implemente una solución utilizando únicamente procesos Persona. Nota: la función UsarTerminal() le permite cargar la SUBE en la terminal disponible.
```c

sem semEspera[P]=([P]0);sem mutexSig=1;
cola colaEspera; Terminal terminal; int sig=-1;

process Personas[id: 1..P]{
    p(mutexSig);
    if (sig==-1){
        sig=id;
        v(mutexSig);
    }else{
        colaEspera.push(id);
        v(mutexSig);
        p(semEspera[id])
    }
    UsarTerminal();

    p(mutexSig);
    if(!colaEspera.empty()){
        sig= colaEspera.pop();
        v(semEspera[sig]);   
    }else{
        sig= -1;
    }
    v(mutexSig);
}

```

b)Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles. Las personas realizan una única fila y la carga la realizan en la primera terminal que se libera. Recuerde que sólo debe emplear procesos Persona. Nota: la función UsarTerminal(t) le permite cargar la SUBE en la terminal t.

XXXXXXXXXXXXXXXXCONSULTAR
```c
sem mutexAcceso = 1;  terminalesLibres = T;  cola colaEsperando<int>;
sem hayTerminal[P] = ([P], 0);
process Persona[id:1..P]{
    P(mutexAcceso)
    if(terminalesLibres == 0){                      //verifico acceso a var compartida
        colaEsperando.push(id);
        V(mutexAcceso);
        P(hayTerminal[id]);
    } else {
        asignadas[id] = terminales.pop();
        terminalesLibres--;
        V(mutexAcceso);
    }
    
    usarTerminal(asignadas[id]);                    //uso la terminal

    P(mutexAcceso)
    if(colaEsperando.isEmpty()){                    //devuelvo la terminal.
        terminales.push(asignadas[id]);
        terminalesLibres++;
        V(mutexAcceso);
    } else {
        int sig = colaEsperando.pop();
        V(mutexAcceso);
        asignadas[sig] = asignadas[id];
        V(hayTerminal[sig]);
    }
}

```  

2)
Implemente una solución para el siguiente problema. Un sistema debe validar un conjunto de 10000 transacciones que se encuentran disponibles en una estructura de datos. Para ello, el sistema dispone de 7 workers, los cuales trabajan colaborativamente validando de a 1 transacción por vez cada uno. Cada validación puede tomar un tiempo diferente y para realizarla los workers disponen de la función Validar(t), la cual retorna como resultado un número entero entre 0 al 9. Al finalizar el procesamiento, el último worker en terminar debe informar la cantidad de transacciones por cada resultado de la función de validación.
 Nota: maximizar la concurrencia.

```c
cola colaTransacciones;
sem mutexCola=1; sem mutexFin=1; sem mutexPuntajes[10]=([10]1);
int puntajes[10]=([10]0); int fin=0;

process worker [id:1..7]{
    Transaccion transaccionActual;
    p(mutexCola);
        while(!colaTransacciones.empty()){
            TransaccionActual= colaTransacciones.pop()
            v(mutexCola);
            puntajeActual=Validar(TransaccionActual);
            P(mutexPuntajes[puntajeActual])
            puntajes[puntajeActual]++;
            V(mutexPuntajes[puntajeActual])
            p(mutexCola);
        }
    v(mutexCola);
    p(mutexFin);
    fin++;
    if(fin==7){
        for (i=1; i<10; i++){
            write( puntajes[i]);
        }
    }
    v(mutexFin)
}
``` 
3) Implemente  una  solución  para  el  siguiente  problema.  Se  debe  simular  el  uso  de  una  máquina 
expendedora de gaseosas con capacidad para 100 latas por parte de U usuarios. Además, existe un 
repositor encargado de reponer las latas de la máquina. Los usuarios usan la máquina según el orden 
de llegada. Cuando les toca usarla, sacan una lata y luego se retiran. En el caso de que la máquina se 
quede sin latas, entonces le debe avisar al repositor para que cargue nuevamente la máquina en forma 
completa. Luego de la recarga, saca una botella y se retira. Nota: maximizar la concurrencia; mientras 
se reponen las latas se debe permitir que otros usuarios puedan agregarse a la fila.

```c
sem mutexMaquina=1; sem reponer=0; sem mutexReponiendo=1; sem mutexCola[U]=([U]1);
int latas=100; int sig=-1;
cola colaEspera; boolean reponiendo=false;

process Usuario [id:1..U]{

    p(mutexCola);                        //acceso al recurso 
    if (sig==-1){
        sig=id;
        v(mutexCola);
    }else{
        colaEspera.push(id);
        v(mutexCola);
        p(mutexCola[id]);
    }
    
    if (latas==0){                      //uso recurso
        v(reponer);
        p(esperoReposicion);
    }
    latas--;
    tomarLATA();
    
    p(mutexCola);                       //libero recurso;
    if(! colaEspera.empty()){
        sig= colaEspera.pop()
        v(mutexCola);
        v(mutexCola[sig]);
    }else{
        sig=-1;
        v(mutexCola);
    }
}

process repositor{
    for(i=1 ; i<U/100;i++){
        p(reponer);
        latas=100;
        v(esperoReposicion);
    }   
}

```
MONITORES 
1) Resolver  el  siguiente  problema.  En  una  elección  estudiantil,  se  utiliza  una  máquina  para  voto 
electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina 
de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. 
La máquina de voto sólo puede ser usada por una persona a la vez. Nota: la función Votar() permite 
usar la máquina.
```c
process Persona[id: 1..N]{
    boolean prioridad=..;
    monitor.llego(id,prioridad);
    votar();
    monitor.retirarse();
}

process Autoridad{
    for (i=1;i<N;i++){
        monitor.darAcceso();
    }
}

Monitor monitor{
    cond autoridad; cond esperaSP; cond esperaCP; cond fin;
    int cantEsperaSP=0; cantEsperaCP=0;

    procedure llego(id:in int; prioridad:in boolean){
        if(prioridad){
            cantEsperaCP++;
            signal(autoridad);
            wait(esperaCP);
        }else{
            cantEsperaSP++;
            signal(autoridad);
            wait(esperaSP);
        }
    }

    procedure retirarse(){
        signal(fin);
    }
    procedure darAcceso(){
        if (cantEsperaSP==0 and cantEsperaCP==0){
            wait(autoridad)
        }
        if (cantEsperaCP>0){
            cantEsperaCP--;
            signal(esperaCP);
        }else{
            cantEsperaSP--;
            signal(esperaSP);
        }
        wait(fin);
    }
}

```
2) Resolver el siguiente problema.  En una empresa trabajan 20 vendedores ambulantes que forman 5 
equipos de 4 personas cada uno (cada vendedor conoce previamente a qué equipo pertenece). Cada 
equipo se encarga de vender un producto diferente. Las personas de un equipo se deben juntar antes 
de  comenzar  a  trabajar.  Luego  cada  integrante  del  equipo  trabaja  independientemente  del  resto 
vendiendo  ejemplares  del  producto  correspondiente.  Al  terminar  cada  integrante  del  grupo  debe 
conocer la cantidad de ejemplares vendidos por el grupo. Nota: maximizar la concurrencia. 
```c
process vendedores[id:1..20]{
    int nroGrupo=..; int nroVentas=0; ventasTotales=0;
    monitor[nroGrupo].llego();
    //salimo a vender
    monitor[nroGrupo].darVentas(nroVentas,ventasTotales);

}

Monitor monitor[id: 1..5]{
    cond espera;
    int cantEspera=0; int ventasTotales=0;

    procedure llego(){
        cantEspera++;
        if(cantEspera<>4){
            wait(espera);
        }else{
            signal_all(espera);
        }
    }
    procedure darVentas(ventas:in int; ventasT:out int){
        ventasTotales= ventasTotales+ventas;
        cantEspera--;
        if(cantEspera<>0){
            wait(espera);
        }else{
            signal_all(espera);
        }
        ventasT=ventasTotales;
    }
}


```
3) Resolver el siguiente problema. En una montaña hay 30 escaladores que en una parte de la subida 
deben utilizar un único paso de a uno a la vez y de acuerdo con el orden de llegada al mismo. Nota: 
sólo se pueden utilizar procesos que representen a los escaladores; cada escalador usa sólo una vez 
el paso
```c
process escalador[id: 1..30]{
    monitor.llegoPaso(id);
    //cruzo el paso
    monitor.finCruce();
}

Monitor monitor{
    cond espera;
    int cantEspera=0;
    boolean libre=true;

    procedure llegoPaso(){
        if(libre){
            libre=false;
        }else{
            cantEspera++;
            wait(espera);
        }
    }
    procedure finCruce(){
        if(cantEspera==0){
            libre=true;
        }else{
            cantEspera--;
            signal(espera);
        }
    }
}
```