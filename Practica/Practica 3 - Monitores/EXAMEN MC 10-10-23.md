Parcial Práctico de MC                  Primera Fecha                               10/10/2023
Tema 1
1. Resolver con SEMÁFOROS los problemas siguientes:
a) En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE en la terminal disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo con el orden de llegada. Implemente una solución utilizando únicamente procesos Persona.
  Nota: la función UsarTerminal() le permite cargar la SUBE en la terminal disponible.
b) Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles. Las personas realizan una única fila y la carga la realizan en la primera terminal que se libera. Recuerde que sólo debe emplear procesos Persona. Nota: la función Usar Terminal(t) le permite cargar la SUBE en la terminal t.

a)
```c
int P = ...;
cola llegadas; sem mutexManejoSig=1;
sem esperar[P] = (P, 0);
int sig = -1;

process Persona[id: 1..P]{
    P(mutexManejoSig);
    if(sig == -1){
        sig = id;
        V(mutexManejoSig);
    } else {
        llegadas.Push(id);
        V(mutexManejoSig);
        P(esperar[id])
    }

    UsarTerminal();

    P(mutexManejoSig);
    if(llegadas.isEmpty()){
        sig=-1;
    } else {
        sig = llegadas.pop();
        V(esperar[id])
    }
    V(mutexManejoSig);
}
```

b)
```c
int T = ...; int P = ...;
cola idTerminales[T] = ...; sem colaTerminales = 1;
cola personas; sem mutexCola=1;
int libres = T;
process Persona[id:1..P]{
    int idTerm;
    P(colaTerminales);
    if(libres == 0){
        V(colaTerminales);

        P(mutexCola);
        personas.push(id);
        V(mutexCola);

        P(esperar[id]);

        P(colaTerminales);
        idTerm = idTerminales.pop();
        V(colaTerminales);
    } else {
        libres--;
        idTerm = idTerminales.pop();
        V(colaTerminales);
    }

    Terminal(idTerm).UsarTerminal();

    P(colaTerminales);
    idTerminales.push(idTerm);
    V(colaTerminales);

    P(mutexCola);
    if(!personas.isEmpty()){
        V(esperar[personas.pop()]);
    } else {
        libres++;
    }
    V(mutexCola);
}
```



2. Resolver con MONITORES el siguiente problema. En una elección estudiantil, se utiliza una máquina para voto electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser usada por una persona a la vez. Nota: la función Votar() permite usar la máquina.

```c
process Persona[id:1..N]{
    bool prioridad = ...; //Depende de si es anciano o embarazada (seria true en ese caso)
    votacion.encolarse(id,prioridad);
    //EMITE VOTO
    votacion.irse();
}

process Autoridad{
    for (int i=1; i<N;i++){
        votacion.siguiente;
    }
}

Monitor Votacion{
    int esperando = 0;
    cond despertarPersona[N];
    cola personas;
    procedure Encolarse(id in int, prioridad in bool){
        esperando++;
        personas.push(id, prioridad);
        signal(despertarAutoridad);
        wait(despertarPersonas[id]);
    }

    procedure Irse(){
        signal(fin);
    }

    procedure Siguiente(){
        if(esperando == 0){
            wait(despertarAutoridad);
        }
        esperando--;
        id, _ = personas.pop();
        signal(despertarPersonas[id]);
        wait(fin);
    }
}
```










