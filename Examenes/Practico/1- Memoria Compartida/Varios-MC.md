MC del 11-10-22
1. Resolver con SEMÁFOROS el siguiente problema. En una planta verificadora de vehículos, existen 7 estaciones donde se dirigen 150 vehículos para ser verificados. Cuando un vehículo llega a la planta, el coordinador de la planta le indica a qué estación debe dirigirse. El coordinador selecciona la estación que tenga menos vehículos asignados en ese momento. Una vez que el vehículo sabe qué estación le fue asignada, se dirige a la misma y espera a que lo llamen para verificar. Luego de la revisión, la estación le entrega un comprobante que indica si pasó la revisión o no. Más allá del resultado, el vehículo se retira de la planta. Nota: maximizar la concurrencia.
```c
cola colaEstacion[7]; cola colaEntrada; 
sem mutexColaEstacion[7]=[(7)1]; sem mutexColaEntrada=1;sem esperaVerificacion[150]=([150]0]; sem avisoEstacion[7]=([7]0);
sem esperaCoord=0; int comprobante[150]=([150]0); sem mutexContador;
int irFila[150]=([150]0); int totalFinalizados=0; int contador[7]=([7]0);

process coordinador{
    for (int i=1;i<=150;i++){
        p(esperaCoord);                         //avisa al coord que hay auto en espera.
        p(mutexColaEntrada);                    
        vehiculo= colaEntrada.pop();
        V(mutexColaEntrada);
        p(mutexContador);                       //bloqueo vector contador
        fila = min(contador);
        contador[fila]++;
        v(mutexContador);
        irFila[vehiculo]=fila;
        V(esperaVerificacion[vehiculo]);
    }
    
}
process vehiculo[id:1..150]{
    int fila;
    p(mutexColaEntrada);
    colaEntrada.push(id);                   //pusheo mi ID y aviso al coordinador que estoy esperando en la cola gral.
    v(mutexColaEntrada);
    v(esperaCoord);
    p(esperaVerificacion[id]);              //me informan cuando tengo fila asignada.
    fila = irFila[id];                      
    p(mutexColaEstacion[fila]);
    colaolaEstacion[fila].push(id);         //me pusheo en la fila asignada
    v(mutexColaEstacion[fila]);
    v(avisoEstacion[fila]);                 //informo a la estacion que estoy esperando.
    p(esperaVerificacion[id]);              //espero a que me avisen que esta el comprobante
    comprobante= comprobante[id];           //saco el comprobante
    //se retira el vehiculo.
}
process estacion[id:1..7]{
    while(true){
        int sig; int comprobante; vehiculo;
        p(avisoEstacion[id]);               //espero que me avisen que hay alguien esperando.
        p(mutexColaEstacion[id]);
        vehiculo= colaolaEstacion[id].pop();      //saco al vehiculo que esta esperando a ser verificado
        v(mutexColaEstacion[id]);
        p(mutexContador);                       //bloqueo vector contador
        cotador[id]--;
        v(mutexContador);
        comprobante= verificar(vehiculo);   //genero el comprobante de la verificacion;
        comprobante[vehiculo]= comprobante; //entrego el comprobante.
        v(esperaVerificacion[vehiculo]);   //informo que tiene el comprobante y se puede retirar
    }
}

```

2. Resolver con MONITORES el siguiente problema. En un sistema operativo se ejecutan 20 procesos que periódicamente realizan cierto cómputo mediante la función Procesar(). Los resultados de dicha función son persistidos en un archivo, para lo que se requiere de acceso al subsistema de E/S. Sólo un proceso a la vez puede hacer uso del subsistema de E/S, y el acceso al mismo se define por la prioridad del proceso (menor valor indica mayor prioridad).
```c
process proceso[id: 1..20]{
    int prioridad=..;
    while(true){
        Procesar();
        SO.solicitudES(id,prioridad);
        //escribo el archivo
        SO.liberarES();
    }
}

Monitor SO{
    cond espera[20];
    cola colaOrdenada;
    boolean ESlibre=true;
    procedure solicitudES (id:in int;prioridad:in int){
        if(ESlibre){
            ESlibre=false;
        }else{
            colaOrdenada.insertarOrdenado(id,prioridad);
            wait(espera[id]);
        }
    }
    procedure liberarES(){
        if(colaOrdenada.empty()){
            ESlibre=true;
        }else{
            proximo= colaOrdenada.pop();
            signal(espera[proximo]);
        }
    }
}
```
### XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

1. Resolver  con  SEMÁFOROS  el  siguiente  problema.  En  un  restorán  trabajan  C  cocineros  y  M  mozos.  De 
forma repetida, los cocineros preparan un plato y lo dejan listo en la bandeja de platos terminados, mientras 
que los mozos toman los platos de esta bandeja para repartirlos entre los comensales. Tanto los cocineros 
como los mozos trabajan de a un plato por vez. Modele el funcionamiento del restorán considerando que la 
bandeja de platos listos puede almacenar hasta P platos. No es necesario modelar a los comensales ni que 
los procesos terminen.  
### NOTA: es el problema de productores/consumidores con tamaño de buffer limitado visto en la página 14 de la teoría 4

```c
int libre=0;int ocupado=0;
sem hayEspacio=P; sem hayPlato=0;
sem mutexMozo=1; mutexCocinero=1;
Buffer bandeja[p];

process cocinero[id:1..C]{
    while(true){
        plato=produzcoPlato();
        p(hayEspacio);
        p(mutexCocinero);
        bandeja[libre]=plato;
        libre = (libre +1)mod P;
        v(mutexCocinero);
        v(hayPlato);
    }
}
process mozo[id:1..M]{
    while(true){
        p(hayPlato);
        p(mutexMozo);
        plato= bandeja[ocupado];
        ocupado= (ocupado+1)mod P;
        v(mutexMozo);
        v(hayEspacio);
        repartir(plato);
    }
}
```
2. Resolver  con  MONITORES  el  siguiente  problema.  En  una  planta  verificadora  de  vehículos  existen  5 
estaciones de verificación. Hay 75 vehículos que van para ser verificados, cada uno conoce el número de 
estación a la cual debe ir. Cada vehículo se dirige a la estación correspondiente y espera a que lo atiendan. 
Una vez que le entregan el comprobante de verificación, el vehículo se retira. Considere que en cada estación 
se atienden a los vehículos de acuerdo con el orden de llegada. Nota: maximizar la concurrencia.
```C

process vehiculo[id:1..75]{
    int nroEstacion=...;Comprobante comprobante;
    planta[nroEstacion].solicitoAtencion(id,comprobante);
    //me retiro
}

process estacion[id:1..5]{
    int siguiente;
    while (true){
        planta[id].verificar(siguiente);
        comprobante= verificar(siguiente);
        planta[id].entregarComprobante(comprobante)
    }
}

Monitor planta[id:1..5]{
    cond estacion; cond seRetira; cond colaEspera;
    boolean estacionLibre=true; cola espera;
    comprobante comprobanteActual;
    
    procedure solicitoAtencion(id:in int;comprobante:out Comprobante){
        espera.push(id);                        //me encolo para esperar
        signal(estacion);                           //aviso que estoy esperando;
        wait(colaEspera);                       //espero que me avisen para pasar.
        comprobante= comprobanteActual;
        signal(seRetira);
    }
    procedure verificar(siguiente=out int){
        if(espera.empty()){                     //si no hay nadie me duermo, sino llamo al siguiente y lo verifico
            wait(estacion);
        }
        siguiente= colaEspera.pop();
    }
    procedure entregarComprobante(compr : in int){
        comprobanteActual=compr;
        signal(colaEspera);
        wait(seRetira);
    }
}


```
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

1.  Resolver  con  SEMÁFOROS  el  siguiente  problema.  En  una  fábrica  de  muebles  trabajan  50  empleados.  A  llegar,  los  empleados 
forman 10 grupos de 5 personas cada uno, de acuerdo al orden de llegada (los 5 primeros en llegar forman el  primer grupo, los 5 
siguientes el segundo grupo, y así sucesivamente). Cuando un grupo se ha terminado de formar, todos sus integrantes se ponen a 
trabajar. Cada grupo debe armar M muebles (cada mueble es armado por un solo empleado); mientras haya muebles por armar en 
el  grupo  los  empleados  los  irán  resolviendo  (cada  mueble es  armado  por  un  solo  empleado).  Nota:  Cada  empleado  puede  tardar 
distinto  tiempo  en  armar  un  mueble.  Sólo  se  pueden  usar  los  procesos  “Empleado”,  y  todos  deben  terminar  su  ejecución. 
Maximizar la concurrencia
```c
int totalGrupo[10]=([10]0); int Grupo=1; int llegada=0; 
sem mutextotalGrupo[10];([10]1); sem mutexLlegada=1; sem barrera[10]=([10]0);

Process empleado[1..50]{
    int miGrupo;
    p(mutexLlegada);
    miGrupo= llegada div 5;
    llegada++;
    if (llegada mod 5 ==0){
        v(mutexLlegada);
        for (i=1;i<=5;i++){
            v(barrera[miGrupo]);
        }
    }else{
        v(mutexLlegada);
    }
    p(barrera[miGrupo]);                //fin barrera y arranca a laburar;

    p(mutextotalGrupo[miGrupo]);
    while (totalGrupo[miGrupo]< M){
        totalGrupo[miGrupo]++;
        v(mutextotalGrupo[miGrupo]);
        //armo mueble;
        p(mutextotalGrupo[miGrupo]);
    }
    v(mutextotalGrupo[miGrupo]);
}

```
2. Resolver con MONITORES el siguiente problema. En un comedor estudiantil hay un horno microondas que debe ser usado por E 
estudiantes  de  acuerdo  con  el  orden  de  llegada.  Cuando  el  estudiante  accede  al  horno,  lo  usa  y  luego  se  retira  para  dejar  al 
siguiente. Nota: cada Estudiante una sólo una vez el horno; los únicos procesos que se pueden usar son los “estudiantes”.
```C
process estudiante [id:1..E]{
    monitor.solicitarMicroondas();
    usarMicroondas();
    monitor.liberar();
}

Monitor monitor{
    cond espera; int esperando=0;
    boolean libre=true;

    procedure solicitarMicroondas(){
        if(libre){
            libre=false;
        }else{
            esperando++;
            wait(espera);
        }
    }
    procedure liberar(){
        if(esperando==0){
            libre=true;
        }else{
            esperando--;
            signal(espera);
        }
    }
}

```
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

MC del 10-10-23
 1.  Resolver con SEMÁFOROS los problemas siguientes: 
a)  En  una  estación  de  trenes,  asisten  P  personas  que  deben  realizar  una  carga  de  su  tarjeta  SUBE  en  la  terminal 
disponible.  La  terminal  es  utilizada  en  forma  exclusiva  por  cada  persona  de  acuerdo  con  el  orden  de  llegada. 
Implemente una solución utilizando sólo emplee procesos Persona. Nota: la función UsarTerminal() le permite cargar 
la SUBE en la terminal disponible. 
```c

```
b)  Resuelva  el  mismo  problema  anterior  pero  ahora  considerando  que  hay  T  terminales  disponibles.  Las  personas 
realizan  una  única  fila  y  la  carga  la  realizan  en  la  primera  terminal  que  se  libera.  Recuerde  que  sólo  debe  emplear 
procesos Persona. Nota: la función UsarTerminal(t) le permite cargar la SUBE en la terminal t.
```c

```
2. Resolver  con  MONITORES  el  siguiente  problema:  En  una  elección  estudiantil,  se  utiliza  una  máquina  para  voto 
electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el 
orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser 
usada por una persona a la vez. Nota: la función Votar() permite usar la máquina.
```c

```
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
MCyMD 01-07-2021

1. Resolver  el  siguiente  problema  con  MONITORES.  En  un  examen  de  la  secundaria  hay  un  preceptor  y
una  profesora  que  deben  tomar  un  examen  escrito  a  45  alumnos.  El  preceptor  se  encarga  de  darle  el
enunciado  del  examen  a  los  alumnos  cundo  los  45  han  llegado  (es  el  mismo  enunciado  para  todos).  La
profesora  se  encarga  de  ir  corrigiendo  los  exámenes  de  acuerdo  al  orden  en  que  los  alumnos  van
entregando.  Cada  alumno  al  llegar  espera  a  que  le  den  el  enunciado,  resuelve  el  examen,  y  al    terminar  lo
deja  para  que  la  profesora  lo  corrija  y  le  dé  la  nota.  Nota:  maximizar  la  concurrencia;  todos  los  procesos
deben  terminar  su  ejecución;  suponga  que  la  profesora  tienen  una  función  corregirExamen  que  recibe  un
examen y devuelve un entero con la nota.

```C

```

1. Resolver  el  siguiente  problema  con  SEMÁFOROS.  En  un  vacunatorio  hay  un  empleado  de  salud  para  
vacunar a 50 personas. El empleado de salud atiende a las personas de acuerdo al orden de llegada y de a 5 
personas a la vez, es decir que cuando está libre debe esperar a que haya al menos 5 personas esperando, 
luego  vacuna  a  las  5  primeras  personas,  y  al  terminar  las  deja  ir  para  esperar  por  otras  5.  Cuando  ha  
atendido  a  las  50  personas  el  empleado  de  salud  se  retira.  Nota:  todos  los  procesos  deben  terminar  su  
ejecución; asegurarse de no realizar Busy Waiting; suponga que el empleado tienen una función 
VacunarPersona() que simula que el empleado está vacunando a UNA persona. 
```C

```



1- Resolver con SEMAFOROS el siguiente problema. En el examen de una materia hay un docente y 50 alumnos. Cuando todos los alumnos han llegado comienza el examen. A medida que los alumnos van terminando le entregan el examen al docente, y esperan a que este le devuelva la nota del examen. El docente debe corregir los exámenes de acuerdo al orden en que los alumnos entregaron. Nota: maximizar la concurrencia.
```C
sem barrera=0;
if(tot==50){
    for 50{
        v(barrera)
    }
}
p(barrera);
//haces ezamen
cola.push(id,examen);
v(profe);
p(espera[id]);
notaExamen= nota[id];
}
profesora{
    for 50{
        p(profesora);
        id,ezamen= cola.pop();
        nota[id]= corregir(ezamen);
        v(espera[id]);
    }
}

```



2- Resolver con SEMAFOROS el siguiente problema. Hay una carrera donde compiten 15 Autos; cuando todos han llegado se larga la carrera. Al terminar cada auto debe saber en qué posición terminó. Nota: maximizar la concurrencia.
```C
    int pos=1; int cantBarrera=0;
    sem mutexCantBarrera=1;sem mutexPos=1; sem barrera=0
process vehiculo[id:1..15]{
    p(mutexCantBarrera);
    cantBarrera++;
    if (cantBarrera==15){
        for (i=1;i<=15;i++){
            v(barrera);
        }
    }
    v(mutexCantBarrera);
    p(barrera);                     //espero a que todos hayan llegado para correr
    //corre la carrera y se termina;
    p(mutexPos);
    miPosicion=pos;
    pos++;
    v(mutexPos);
    //me fui
}

```
3- Resolver con MONITORES el siguiente problema. Hay un teléfono público que debe ser utilizado por N personas de acuerdo al orden de llegada (de a una persona a la vez). Nota: maximizar la concurrencia.
```C
process persona[id:1..N]{
    monitor.solicitoUso();
    //uso recurso
    monitor.liberar();
}
Monitor monitor{
    boolean libre=true; cond espera;
    
    procedure solicitarUso(){
        if(libre){
            libre=false;
        }else{
            cantEspera++;
            wait(espera);
        }
    }
    procedure liberar(){
        if(cantEspera==0){
            libre=true;
        }else{
            cantEspera--;
            signal(espera);
        }
    }
}
```
4- Resolver con MONITORES el uso de un equipo de videoconferencia que puede ser usado por una única persona a la vez. Hay P Personas que utilizan este equipo (una única vez cada uno) para su trabajo de acuerdo a su prioridad. La prioridad de cada persona está dada por un número entero positivo. Además existe un Administrador que cada 3 hs. incrementa en 1 la prioridad de todas las personas que están esperando por usar el equipo. Nota: maximizar la concurrencia.
```C
process persona[id:1..P]{
    int prioridad=..;
    monitor.solicitarUso(id,prioridad);
    //uso equipo
    monitor.libero();
}

process administrador{
    while(true){
        delay(3hs);
        monitor.aumentarPrioridad();
    }
}

Monitor monitor{
    int cantEspera=0; cond espera[p]; cola colaEspera; boolean libre=true;
    procedure solicitarUso(id:in int; prioridad:in int){
        if(libre){
            libre=false;
        }else{
            colaEspera.insertarOrdenado(id,prioridad);
            cantEspera++;
            wait(espera[id])
        }
    }
    procedure libero(){
        int sig;
        if(cantEspera==0){
            libre=true;
        }else{
            sig= colaEspera.pop();      //al hacer pop me daria tb la prioridad pero no la uso;
            cantEspera--;
            signal(espera[sig]);
        }
    }
    procedure aumentarPrioridad(){
        for (i=1;i<=cantEspera;i++){
            id,prioridad= colaEspera.pop();
            prioridad++;
            colaEspera.insertarOrdenado(id,prioridad);
        }
    }
}

```
5- Resolver con MONITORES el siguiente problema. En un Crucero por el Mediterráneo hay 200 personas que deben subir al barco por medio de 10 lanchas con 20 lugares cada una. Cada persona sube a la lancha que le corresponde. Cuando en una lancha han subido sus 20 personas durante 5 minutos navega hasta el barco. Recién cuando han llegado las 10 lanchas al barco se les permite a las 200 personas subir al barco. Nota: suponga que cada persona llama a la función int NúmeroDeLancha() que le devuelve un valor entre 0 y 9 indicando la lancha a la que debe subir. Maximizar la concurrencia.
```C
process persona[id:1..200]{
    int num= NúmeroDeLancha();
    manejoLancha[num].abordar();
    Barco.abordar();
}
process lancha[id:1..10]{
    manejoLancha[id].navegar();
    delay(5 min);
    Barco.llegolancha();
    manejoLancha[id].desembarcar();
}

monitor manejoLancha[id:1..10]{
    int cantPasajeros=0;
    cond espera;

    procedure abordar(){
        cantPasajeros++;
        if(cantPasajeros==20){
            signal(lancha);
        }
        wait(espera);
    }

    procedure navegar(){
        if(cantPasajeros<>20){
            wait(lancha);
        }
    }
    procedure desembarcar(){
        signal_all(espera);
    }
}

monitor Barco{
    int totalLancha=0;
    cond subirBarco;
    int totalPersona=0;

    procedure llegolancha(){
        totalLancha++;
        if(totalLancha==10){
            signal_all(subirBarco);
        }else{
            wait(subirBarco);
        }
    }

    procedure abordar(){
        totalPersona++;
    }

}

```

2_Resolver este ejercicio con Semáforos o Monitores. En un centro oftalmológico hay 2 médicos con diferentes especialidades. Existen N pacientes que deben ser atendidos. Para esto algunos de los pacientes puede ser atendido indistintamente por cualquier médico y otros solo por uno delos médicos en particular. Cada paciente saca turno con cada uno de los médicos que lo puede atender y espera hasta que llegue el turno con uno de ellos, espera a que termine de atenderlo y se retira.
Nota: suponga que existe la función Elegir Medico() que retorna 1,2 o 3. (1 indica que solo se atiende con el medico 1, 2 que solo se atiende con el medico 2 o 3 que puede ser atendido indistintamente por cualquier medico)
```c
SEMAFOROS:
sem mutexCola[2]=1; cola colaEspera[2];int cantEspera[2]=([3]0);
sem medico[2]=0; sem espera[N]=([N]0);

process paciente[id:1..N]{
    nMedico=Elegir medico();                //elijo medico.
    if(nMedico==3){
        nMedico= randInt(2);
    }
    p(mutexCola[nMedico]);
    cantEspera[nMedico]++;
    colaEspera[nroMedico].push(id);
    v(mutexCola[nroMedico]);
    v(medico[nMedico]);
    p(espera[id]);                          //espero a ser atendido. 
}

Process medico[id:1..2]{
        int sig;
    while(true){
        p(medico[id]);                      //espero que me avisen
        p(mutexCola[id]);               
        cantEspera[id]--;
        sig=colaEspera[id].pop();
        v(mutexCola[id]);
        atenderPaciente(sig);
        v(espera[sig]);                 //aviso que se puede ir;
    }
}

```
### MONITORES
```c
process paciente[id:1..P]{
    nMedico=Elegir medico();
    if(nMedico==3){
        nMedico= randInt(2);
    }
    fila[nMedico].llegue();
    consultorio[nMedico].solicitarAtencion();

}
process medico[id:1..2]{
    while(true){
        fila[id].proximo();
        consultorio[id].atender();
        atenderPaciente();
        consultorio[id].liberar();
    }
}

Monitor fila[id:1..2]{
    cond espera; int enEspera=0;
    
    procedure llegue(){
        enEspera++;
        signal(medico);
        wait(espera);       //pasa a ser atendido
    }

    procedure proximo(){
        if(enEspera==0){
            wait(medico);
        }
        enEspera--;
        signal(espera);
    }

}

Monitor consultorio[id:1..2]{
    boolean llegue=false; 
    cond medico, finAtencion;

    procedure solicitarAtencion(){
        llegue=true;
        signal(medico)
        wait(finAtencion);
        signal(fin);
    }
    procedure atender(){
        if(!llegue){
            wait(medico)
        }
    }
    procedure liberar(){
        true=false;
        signal(finAtencion);
        wait(fin);
    }

}
```
6. SEMAFOROS Una empresa de turismo posee 4 combis con capacidad para 25 personas cada una y UN
vendedor que vende los pasajes a los clientes de acuerdo al orden de llegada. Hay C clientes
que al llegar intentan comprar un pasaje para una combi en particular (el cliente conoce este
dato); si aún hay lugar en la combi seleccionada se le da el pasaje y se dirige hacia la combi;
en caso contrario se retira. Cada combi espera a que suban los 25 pasajeros, luego realiza el
viaje, y cuando llega al destino deja bajar a todos los pasajeros. Nota: maximizar la
concurrencia; suponga que para cada combi al menos 25 clientes intentarán comprar pasaje.
```c
    sem boleteria=0; mutexColaBoleteria=1; esperaCombi[C]=([C]0);esperaEntrada[C]=([C]0); finViaje[4]=([4]0)
    cola colaBoleteria;
    boolean entrada[C]=([C]false);

    process combi[id:1..4]{
        for (i=1;i<=25;i++){
            p(esperaCombi[id]);
        }
        delay(Tiempo del viaje);
        for (i=1;i<=25;i++){
            v(finViaje[id]);
        }
    }
    process vendedor{
        int ticket[4]=([4]25);;int id,nroCombi;
        while(true){
            p(boleteria);
            p(mutexColaBoleteria);
            id,nroCombi= colaBoleteria.pop()
            v(mutexColaBoleteria);
            if(ticket[nroCombi]>0){
                ticket[nroCombi]--;
                entrada[id]=true;
            }else{
                entrada[id]=false;
            }

        }
    }
    process cliente[id:1..C]{
        int nroCombi= ..;
        p(mutexColaBoleteria);
        colaBoleteria.push(id,nroCombi);
        v(mutexColaBoleteria);
        v(boleteria);
        p(esperaEntrada[id]);
        if(entrada[id]){
            v(esperaCombi[nroCombi]);
            p(finViaje[nroCombi]);
        }
        //se retira
    }

```
7. SEMAFOROS En una herrería hay 15 empleados que forman 5 grupos de 3 personas; los grupos se forman
de acuerdo al orden de llegada (los 3 primeros pertenecen al grupo 1, los 3 siguientes al grupo
2, y así sucesivamente). Ni bien conoce el grupo al que pertenece el empleado comienza a
trabajar (no debe esperar al resto de grupo para comenzar). Cada grupo debe hacer
exactamente P unidades de un producto (cada unidad es hecha por un único empleado). Al
terminar de hacer las P unidades de un grupo, sus 3 empleados se retiran. Nota: maximizar
la concurrencia; ningún grupo puede hacer unidades de más.
```c
int nGrupo=1; int piezas[5]=([5]P);int termine[5]=([5]0); int cantEmpleado=1;
sem mutexPiezas[5]=([5]1); sem mutexCantEmpledo=1; sem mutexTermine[5]=([5]1); sem barreraIrse[5]=([5]0); 

process empleado[id 1..15]{
    int nroGrupo=..;
    p(mutexCantEmpledo);
    nroGrupo= cantEmpleado div 3;
    cantEmpleado++;
    v(mutexCantEmpledo);
    p(mutexPiezas[nroGrupo]);
    while (piezas[nroGrupo]>0){
        piezas[nroGrupo]--;
        v(mutexPiezas[nroGrupo]);
        //hago piezas
        p(mutexPiezas[nroGrupo]);
    }
    v(mutexPiezas[nroGrupo]);
    p(mutexTermine[nroGrupo]);
    termine[nrogrupo]++;
    if(termine[nrogrupo]==3){
        for (i=1;i<=3;i++){
            v(barreraIrse[nroGrupo]);
        }
    }
    v(mutexTermine[nroGrupo]);
    p(barreraIrse[nroGrupo];
    }
```

Resolver con SEMÁFOROS el siguiente problema. Simular un examen técnico para concursos Nodocentes en la Facultad, en el mismo participan 100 personas distribuidas en 4 concursos (25 personas en cada concurso) con un coordinador en cada una de ellos. Cada persona ya conoce en que concurso participa. El coordinador de cada concurso espera hasta que lleguen las 25 personas correspondientes al mismo, les entrega el examen a resolver (el mismo para todos los de ese concurso), y luego corrige los exámenes de esas 25 personas de acuerdo al orden en que van entregando. Cada persona al llegar debe esperar a que su coordinador (el que corresponde a su concurso) le dé el examen, lo resuelve, lo entrega para que su coordinador lo evalúe y espera hasta que le deje la nota para luego retirarse. Nota: maximizar la concurrencia; sólo usar los procesos que representes a las personas y a los coordinadores; todos los procesos deben terminar.
```c
sem barrera[4]=([4]0),mutexColaExamen[4]=([4]1), coordinador[4]=([4]0), esperoNota[4]=([4]0);mutexCont[4]=([4]1);
cola colaExamen[4];
int examen[100]=([100]0), cont[4]=([4]0);

process persona[id: 1..100]{
    concurso=..;
    p(mutexCont[concurso]);
    cont[concurso]++;
    v(mutexCont[concurso]);
    if(cont[concurso]==25){
        v(coordinador[concurso]);
    }
    p(barrera[concurso]);       //barrera
    int examen= examen[concurso];
    hacer(examen);
    p(mutexColaExamen[concurso]);
    colaExamen[concurso].push(id,examen);
    v(mutexColaExamen[concurso]);
    v(coordinador[concurso]);
    p(esperoNota[id]);
    nota=examen[id];
    //me fui
}
process coordinador[id:1..4]{
    p(coordinador[concurso]);
    examen[id]= new examen();
    for (i=1;i<=25;i++){
        v(barrera[concurso])
    }
    for (i=1;i<=25;i++){
        p(coordinador[concurso]);
        p(mutexColaExamen[concurso]);
        id,examen= colaExamen[concurso].pop();
        v(mutexColaExamen[concurso]);
        examen[id]= corregir(examen);
        v(esperoNota[id]);
    }
}
```

4.Monitores En una mesa de exámenes hay 3 profesores que les deben tomar un examen oral a 30 alumnos
de acuerdo al orden de llegada. Cada examen es tomado por un único profesor. Cuando un
alumno llega, espera a que alguno de los profesores (cualquiera) lo llame y se dirige al
escritorio correspondiente a ese profesor, donde le tomará el examen; al terminar el
profesor le da la nota y el alumno se retira. Cuando un profesor está libre llama al siguiente
alumno. Nota: todos los procesos deben terminar su ejecución.
```c
process profesor[id:1..3]{
    boolean hayAlumno=true;
    acceso.siguiente(id,hayAlumno);
    while (hayAlumno){
        escritorio.siguiente();
        int nota= tomarOral();
        escritorio.darNota(nota);
        acceso.siguiente(id,hayAlumno);
    }
}
process alumno[id:1..30]{
    int nProf; nota;
    acceso.llego(id,nProf);
    escritorio[nProf].darExamen(nota);
}
Monitor acceso{
    cond espera,profesor;
    cola cola;
    int canSiguientesProfe=30; profesor[30]=([30]0);

    procedure llego(id:in int; nProf: out int){
        cola.push(id);
        signal(profesor);
        wait(espera);
        nprof=profesor[id];
    }
    procedure siguiente(idProf:in int;hayAlumno:out boolean){
        int idAlumno;
        if(canSiguientesProfe>0)
            canSiguientesProfe--;
            if(cola.empty())){
                wait(profesor)
            }
            idAlumno=cola.pop();
            profesor[idAlumno]=idProf;
            signal(espera);
        }else{
            hayAlumno=false;
        }
    }
}
monitor escritorio[id: 1..3]{
    boolean libre=true;
    int notaAct;
    cond esperar, fin, profesor;

    procedure darExamen(notaFinal:out int){
        libre=false;
        signal(profesor)
        wait(esperar);
        notafinal= notaAct;
        signal(fin)
    }
    procedure siguiente(){
        if(libre){
            wait(profesor);
        }
    }
    procedure darExamen(nota: in int){
        notaAct=nota;
        signal (esperar);
        wait(fin)
    }
}
```

6. Monitores En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada
una conoce previamente a que equipo pertenece). Cuando las personas van llegando
esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4
integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que
cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser
de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al
finalizar cada persona debe conocer el monto total juntado por su grupo. Nota: maximizar
la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una
persona existe una función Moneda() que retorna el valor de la moneda encontrada
```c
process Jugadores[id:1..20]{
    equipo=..; int totalGrupo=0;
    juego[equipo].llego();
    for (i=1;i<=15;i++){
        total+=Moneda();
    }
    juego[equipo].terminar(totalMio,totalGrupo);
}
Monitor juego[id:1..4]{
    int cont=0; int totalEquipo=0;

    procedure llego(){
        cont++;
        if(cont==4){
            signal_all(espera);
        }else{
            wait(espera);
        }
    }

    procedure terminar(total:in int; totalE:out int){
        totalEquipo+=total;
        cont--;
        if(cont==0){
            signal_all(espera);
        }else{
            wait(espera);
        }
        totalE=totalEquipo;
    }
}

```
