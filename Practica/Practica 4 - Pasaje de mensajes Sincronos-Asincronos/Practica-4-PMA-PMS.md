# Practica 4 PMA // PMS 

# PMA

## Ejercicio 1
Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados.
 Analice el problema y defina qué procesos, recursos y canales/comunicaciones serán necesarios/convenientes
 para resolverlo. Luego, resuelva considerando las siguientes situaciones:
a. Existe un único empleado, el cual atiende por orden de llegada.
```cpp
chan avisoLlegada(string);chan fin[N](string);

process Empleado{
    int idProx;
    for (i:1; i<N; i++){
        receive avisoLlegada(idProx);
        //atiende al cliente
        send fin[idProx]("finalizado");
    }
}
process CLiente [id: 1..N]{
    string fin;
    send avisoLlegada(id);
    receive fin[id](fin);
}

```
b. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe modificarse en la solución anterior?
    Se debe modificar: 
    - La declaracion sobre el siguiente, debe dejar el id del empleado que atenderá al cliente;
    - se usa un WHILE en vez de un for. ya que no se sabe cuantos clientes atendera cada uno, en el mejor de los casos
        cada uno atendera N/2. pero no se sabe. NOTA: se puede ir llevando un conteo entre procesos empleados, pero
        esto nos demandaria una cantidad continua de mensajes entre ellos, el ejercico no aclara que debe terminar.
    - quien envia el fin se identiieque para que el cliente sepa donde buscar la rta.
```Cpp
chan avisoLlegada(string);
chan fin[1..2](string);

process Empleado [id: 1..2]{
    int idProx;
    while (true){
        receive avisoLlegada(idProx);
        //atiende al cliente
        send fin[id]("finalizado");
    }
}
process CLiente [id: 1..N]{
    string rta; string fin;
    send avisoLlegada(id);
    receive fin[idPuesto](fin);
}
// si llegado el caso hay que terminarlo habria quue agregar un proceso extra para que lleve el conteo de los cliemntes.
//cuando no hay mas envia un mensaje a los procesos empleado para que finalicen
```
c.Ídem b) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas
 durante 15 minutos.
 ¿Se puede resolver sin usar procesos adicionales? no
 ¿Qué consecuencias implicaría? genera demora innecesaria
```Cpp
chan avisoLlegada(string);
chan fin[1..2](string);
chan buscoCliente(int);
chan rtaEmpleado[1..2];

process Empleado [id: 1..2]{
    int idCliente;
    send buscoCliente(id);
    receive rtaEmpleado(idCliente);
    while (idCliente<>-100){
        if(idCliente==-1){
            delay(15min);
        }else{
            //atiende al cliente
            send fin[id]("finalizado");
        }
        send buscoCliente(id);
        receive rtaEmpleado(idCliente);
    }
}
process CLiente [id: 1..N]{
    string fin;
    send avisoLlegada(id);
    receive fin[id](fin);
}

process coordinador{ 
    int id;                           //el coordinador evita que los falsos positivos de la cola vacia, evitando que nunca terminen los procesos. 
    for (i=0;i<N;i++){
        receive buscoCliente(idEmpleado);
        if(!avisoLlegada.empty()){
            receive avisoLlegada(id);
        }else{
            id = -1;
        }
        send rtaEmpleado[idEmpleado](id);
    }
    id=-100
    send rtaEmpleado[1](id);
    send rtaEmpleado[2](id);
}

```

## Ejercicio 2

Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. 
Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido.
En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante.
Nota: maximizar la concurrencia (para ello usamos el coordinador quien se encarga de asignar la caja a cada cliente).
```cpp
chan solicitudCoordinador(String,int), rtaCoord[P](int),
chan solicitarPago[5](int, int), recibirComprobante[P](int),  

process Empleado[id:1..5]{
    int nroComprobante;
    while(true){
        receive solicitarPago[id](idCliente,pago);
        nroComprobante = facturarPago(pago);                    //funcion que al realizar una factura devielve el nro de comprobante
        send recibirComprobante[idCliente](nroComprobante);
        send solicitudCoordinador("liberarFila", id);
    }
}

process cliente [id:1..P]{
    int idFila;
    send solicitudCoordinador("solicitarFila",id);
    receive rtaCoord[id](idFila);
    send solicitarPago[idFila](id, pago);
    receive recibirComprobante[id](comprobante);
}

process coordinador{
    int cantClientesEnCola[1..5]=([5]0), index; 
    String opcion;
    while(true){  
        receive solicitudCoordinador(opcion, id);      //opciones pueden ser: "solicitarFila" + id cliente. "liberarFila" + id Fila;
        if (opcion.equals("solicitarFila"))
            index= buscarMinimo(cantClientesEnCola);
            cantClientesEnCola[index]++;
            send rtaCoord[id](index);
        }else{
            cantClientesEnCola[id]--;
        }
    }

    function buscarMinimo(cantCola){
        int min = 9999, minIndex;
        for (int i = 0; i < 5; i++) {
            if (cantCola[i] < min) {
                min = cantCola[i];
                minIndex=i;
            }
        }
        return minIndex;
    }
}
```

## Ejercicio 3

Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes.
El modelado debe considerar que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato.
  Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.
Nota: maximizar la concurrencia.
```cpp
chan avisoCocinero(int,string), finPedido[C](Plato);
chan vendedorLibre(int), vendedor(int, string)
chan pedido(id, string), 
process cocinero[id: 1..2]{
    int idCliente; String pedido; Plato plato;
    while(true){
        receive avisoCocinero(idCliente, pedido);
        plato = cocinar(pedido);
        send finPedido[idCliente](plato);
    }
}
process vendedor[id: 1..3]{
    String pedidoSolicitado;
    while (true){
        send vendedorLibre(id)
        receive vendedor(idCliente,pedido);
        if(idCliente==-1){
            delay(1..3 min);
        }else{
            send avisoCocinero(idCliente,pedidoSolicitado);
        }
    }
}
process cliente[id: 1..C]{
    String pedido;pedidoCocinado;
    send pedido(id, pedido);                    //envio al coordinador mi pedido, se lo envio a este por que los vendedores al ser <1 no pueden consultar por EMPTY.
    receive finPedido[id](pedidoCocinado);
}
process coordinador{                            //EXPLICACION DEL USO DEL COORD: el proceso coordinador hace de pasamanos y regula el comportamiento de los vendedores, esto pasa por que 
    while(true){                                //los vendedores no pueden evaluar de forma correcta que un canal sea empty, ya que puede pasar, que 2 procesos pregunten por
        receive vendedorLibre(idVend);          //un canal que no sea vacio en un if(lo cual es verdad) y al entrar haber un mensaje solo pudiendo suceder que uno de los dos procesos lo   
        if (pedido.empty()){                    //toma y el otro se queda esperando en el receive. Este 2do proceso no hace el delay al quedarse trabado, pero si ademas nunca llegase
            id=-1;                              //otro mensaje el proceso nunca finalizaria. 
        }else{
            receive pedido(id,pedido);
        }
        send vendedor(id,pedido);
    }   
}

```

## Ejercicio 4

Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. 
Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle.
El empleado atiende a los clientes en el orden en que hacen los pedidos.
A cada cliente se le entrega un ticket factura por la operación.
a) Implemente una solución para el problema descrito. (IMPLEMENTE DIRECTAMENTE EL B SIN DARME CUENTA).
b)Modifique la solución implementada para que el empleado dé prioridad a los que terminaron de usar la cabina sobre los que están esperando para usarla.
Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.
```cpp
chan solicitarCabina(int);
chan obtenerCabina[N](int);
chan pagarEmpleado(int,int);
chan recibirComprobante[N](int);

process cliente[id: 1..N]{
    int cabina, ticket;
    send solicitarCabina(id);
    receive obtenerCabina[id](cabina);
    //USO CABINA
    send pagarEmpleado(id, cabina);
    receive recibirComprobante[id](ticket);
}

process empleado{
    int idCliente, cantCabinas=10, ticketAct, cabinaUsada;
    cola colaCabina=(10 cabinas); 
    while (true){
        if(!pagarEmpleado.empty()){
            receive pagarEmpleado(idCliente, cabinaUsada);
            ticketAct = Cobrar(cabinaUsada);
            cantCabinas++;
            colaCabina.push(cabinaUsada);
            send recibirComprobante[idCliente](ticketAct);
        }else{
            if((!solicitarCabina.empty()) and (cantCabinas>0)){
                receive solicitarCabina(idCliente);
                cantCabinas--;
                send obtenerCabina[idCliente](colaCabina.pop());
            }
        }
    }
}

```

## Ejercicio 5

Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos,
los cuales están continuamente trabajando y cada tanto envían documentos a imprimir.
Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada.

a)Implemente una solución para el problema descrito.
```cpp
chan solicitarImpresion(Documento);

process empleado[id: 1..N]{
    Documento doc;
    while (true){
        //TRABAJA
        doc = new Documento();              //genera el doc
        send solicitarImpresion(doc);
    }
}

process impresora[id:1..3]{
    Documento documento;
    while (true){
        receive solicitarImpresion(documento);
        Imprimir(documento);
    }
}

```

b)Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas,
 el cual tiene prioridad sobre los administrativos.
```cpp
chan solicitarImpresion(Documento);
chan solicitarImpresionDirector(Documento);
chan hayQueImprimir(boolean);
chan impresoraOK(int);

process empleado[id: 1..N]{
    Documento doc;
    while (true){
        //TRABAJA
        doc = new Documento();              //genera el doc
        send solicitarImpresion(doc);
        send hayQueImprimir(TRUE);
    }
}

process director{
    Documento doc;
    while (true){
        //TRABAJA
        doc = new Documento();              //genera el doc
        send solicitarImpresionDirector(doc);
        send hayQueImprimir(TRUE);                      //EL AVISO QUE AHY QUE IMPRIMIR ES PARA QUE EL COORDINADOR NO HAGA BUSY WAITING AL PEDO.
    }
}

process impresora[id:1..3]{
    Documento documento;
    while (true){
        send impresoraOK(id);                       //aviso al coordinador que estoy libre. 
        receive imprimir[id](documento);
        Imprimir(documento);
    }
}

process coordinadorImpresora{
    while(true){
        receive impresoraOK(idImpresora);               //al tener una impresora libre se evalua que archivo tiene que imprimir. 
        receive hayQueImprimir(ok);                     //se cuelga en hay que imprimir en caso de que haya archivo.
        if(!solicitarImpresionDirector.empty()){
            receive solicitarImpresionDirector(documento);
        }else{
            receive solicitarImpresion(documento);
        }
        send imprimir[idImpresora](documento);
    }
}

```

c)Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución.
```cpp
chan hayQueImprimir(boolean);
chan paraImpresion(int);

process empleado[id: 1..N]{
    Documento doc;
    for(int i=0; i<10;i++){
        //TRABAJA
        doc = new Documento();              //genera el doc
        send paraImpresion(doc);
    }
}

process impresora[id:1..3]{                                                                   
    Documento documento;
    receive solicitarImpresion(documento);
    while (!documento.equals("fin")){
        imprimir(documento);
        receive solicitarImpresion(documento);
    }
}

process coordinador{
    for(int i=1; i<=10*N;i++){
        receive paraImpresion(documento);
        send solicitarImpresion(documento)
    }
    for(int i=0; i<3;i++){
        send solicitarImpresion("fin")
    }
}


```

d)Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.
```cpp
chan solicitarImpresion(Documento);
chan solicitarImpresionDirector(Documento);
chan hayQueImprimir(boolean);
chan impresoraOK(int);
chan imprimir(Documento)
process empleado[id: 1..N]{
    Documento doc;
    for(int i=0; i<10;i++){
        //TRABAJA
        doc = new Documento();                          //genera el doc
        send solicitarImpresion(doc);
        send hayQueImprimir(TRUE);                      //Aviso que hay que imprimir, ya que k aimpresora puede estar preparada, pero que no hayan archivos para imprimir. 
    }
}

process director{
    Documento doc;
    for(int i=0; i<10;i++){
        //TRABAJA
        doc = new Documento();                          //genera el doc
        send solicitarImpresionDirector(doc);
        send hayQueImprimir(TRUE);                      //Aviso que hay que imprimir, ya que k aimpresora puede estar preparada, pero que no hayan archivos para imprimir. 
    }
}

process impresora[id:1..3]{
    Documento documento;
    send impresoraOK(id);                               //aviso al coordinador que estoy libre. 
    receive imprimir(documento);
    while (!documento.equals("fin")){
        Imprimir(documento);
        send impresoraOK(id);                           //aviso al coordinador que estoy libre. 
        receive imprimir(documento);
    }
}

process coordinadorImpresora{
        for(int i=0; i<10*(N+1);i++){                   //FOR hasta 10 archivos * (N empleados + 1 director) = 110 archivos.
            receive impresoraOK(idImpresora);           //al tener una impresora libre se evalua que archivo tiene que imprimir. 
            receive hayQueImprimir(ok);                 //se cuelga en hay que imprimir hasta que haya archivo.
            if(!solicitarImpresionDirector.empty()){
                receive solicitarImpresionDirector(documento);
            }else{
                receive solicitarImpresion(documento);
            }
            send imprimir[idImpresora](documento);
        }
        for(i=1;i<=3;i++){
            receive impresoraOK(idImpresora);
            send imprimir[idImpresora]("fin");
        }
}
```

e)Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo.
Nota: ni los administrativos ni el director deben esperar a que se imprima el documento.
## al realizar un canal de "hay que imprimir" avisando que el canal

# PMS

## Ejercicio 1

Suponga  que  existe  un  antivirus  distribuido que  se  compone  de  R  procesos  robots Examinadores y 1 proceso Analizador.
 Los procesos Examinadores están buscando continuamente  posibles  sitios  web  infectados;
 cada  vez  que  encuentran  uno  avisan  la dirección y luego continúan buscando.
 El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados.

a) Analice  el  problema  y  defina  qué  procesos,  recursos  y  comunicaciones  serán 
necesarios/convenientes para resolverlo. 
b) Implemente una solución con PMS sin tener en cuenta el orden de los pedidos. 
```cpp
process Robot[id:1..R]{
    String url;
    while(true){
        urlVirus = analizar();
        Analizador!aviso(urlVirus);          // AL HACER ESTO SE QUEDA TODO TRABADO NO SE COMO HACERLO SIN TENER EN CUIENTA EL ORDEN. HARIA ALGO COMO EL C!
    }
}
process Analizador{
    String url;
    while(true){
        Robot[*]?aviso(url);
        analizar(url);
    }
}
```
c) Modifique  el  inciso  (b)  para  que  el  Analizador  resuelva  los  pedidos  en  el  orden en que se hicieron. 

```cpp
process Robot[id:1..R]{
    String url;
    while(true){
        urlVirus = analizar();
        Admin!aviso(urlVirus);              //encruentro una pag rari y se la mando al coord para que la mande a analizar.
    }
}
process Analizador{
    String web;
    while(true){
        Admin!pedido();
        Admin?aviso(url);
        analizar(url);
    }
}
proces admin{
    cola avisos; string url;
    do {
        () Robot[*]?aviso(url) -> avisos.push(url);
        (!avisos.empty()) Analizador?pedido() -> Analizador!aviso(avisos.pop());
    } od
}
```

## Ejercicio 2

En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos 
continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo 
empleado  y  vuelve  a  su  trabajo.  El  segundo  empleado  toma  cada  muestra  de  ADN 
preparada,  arma  el  set  de  análisis  que  se  deben  realizar  con  ella  y  espera  el  resultado  para 
archivarlo.  Por  último,  el  tercer  empleado  se  encarga  de  realizar  el  análisis  y  devolverle  el 
resultado al segundo empleado.  
```cpp
procedure EmpleadoUNO{
    Muestra muestra;
    while (true){
        muestra = new muestra();
        Admin!muestra(muestra);
    }
}

procedure EmpleadoDOS{
    Muestra muestraAct;
    cola archivo;
    while(true){
        Admin!solicitarMuestra();
        Admin?muestra(muestraAct);
        set =ArmarSetAnalisis(muestraAct);
        EmpleadoTRES!SetAnalisis(set);
    	EmpleadoTRES?resultado(resultado);
        archivo.push(resultado);
    }
}

procedure EmpleadoTRES{
    while(true){
        EmpleadoDOS?SetAnalisis(set);
        resultado= AnalizarSet(set);
        EmpleadoDOS!resultado(resultado);
    }
}

procedure Admin{
    cola colaMuestras;
    do{
        EmpleadoUNO?muestra(muestra) -> colaMuestras.push(muestra);
        (!colaMuestras.empty()) EmpleadoDOS?solicitarMuestra() -> EmpleadoDOS!muestra(colaMuestras.pop());
    } od
}
```

## Ejercicio 3

En  un  examen  final  hay  N  alumnos  y  P  profesores.  Cada  alumno  resuelve  su  examen,  lo 
entrega  y  espera  a  que  alguno  de  los  profesores  lo  corrija  y  le  indique  la  nota.  Los 
profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.  
### Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben terminar su ejecución 

a) Considerando que P=1. 
```cpp
process Alumno[id: 1..N]{
    int nota;
    Examen examen= HacerExamen();
    Admin!finExamen(examen,id);
    Profesor?nota(nota)
}
process Profesor{
    Examen examen; int nota;
    for(i=1; i<=N;i++){
        Admin!solicitud();
        Admin?solicitarExamen(examen,idAlumno);
        nota= corregir(examen);
        Alumno[idAlumno]!nota(nota);
    }
}
process Admin{
    cola examenes; int cont
    do{
        (cont<N)Alumno[*]?finExamen(examen,id) -> cont++ ; examenes.push(examen,id);
        (!examenes.empy()) Profesor?solicitud()-> Profesor!solicitarExamen(examenes.pop());
    }od
}

```

b) Considerando que P>1. 
```cpp
process Alumno[id: 1..N]{
    int nota;
    Examen examen= HacerExamen();
    Admin!finExamen(examen,id);
    Profesor[*]?nota(nota)
}
process Profesor[id:1..P]{
    Examen examen; int nota;
    Admin!solicitud(id);
    Admin?solicitarExamen(examen,idAlumno);
    while(idAlumno != -1){
        nota= corregir(examen);
        Alumno[idAlumno]!nota(nota);
        Admin!solicitud(id);
        Admin?solicitarExamen(examen,idAlumno);
    }
}
process Admin{
    cola examenes;
    do{
        (cont<N) Alumno[*]?finExamen(examen,id) -> cont++; examenes.push(examen,id);
        (!examenes.empy()) Profesor[*]?solicitud(idProfesor)-> Profesor[idProfesor]!solicitarExamen(examenes.pop());
    }od

    for(i=1; i<=P;i++){
        Profesor[*]?solicitud(idProfesor);
        Profesor[idProfesor]!solicitarExamen("fin",-1)
    }
}
```

c) Ídem  b)  pero  considerando  que  los  alumnos  no  comienzan  a  realizar  su  examen  hasta 
que todos hayan llegado al aula. 
```cpp
process Alumno[id: 1..N]{
    int nota;
    Admin!llegue();
    Admin?Empezar();
    Examen examen= HacerExamen();
    Admin!finExamen(examen,id);
    Profesor[*]?nota(nota)
}
process Profesor[id:1..P]{
    Examen examen; int nota;
    Admin!solicitud(id);
    Admin?solicitarExamen(examen,idAlumno);
    while(idAlumno!=-1){
        nota= corregir(examen);
        Alumno[idAlumno]!nota(nota);
        Admin!solicitud(id);
        Admin?solicitarExamen(examen,idAlumno);
    }
}
process Admin{
    cola examenes; int cont=0;
    for(i=1; i<=N;i++){                     //manejo la barrera
        Alumno[*]?llegue();
    }
    for(i=1; i<=N;i++){
        Alumno[i]!empezar();
    }

    do{
        (cont<N) Alumno[*]?finExamen(examen,id) -> cont++; examenes.push(examen,id);
        (!examenes.empy()) Profesor[*]?solicitud(idProfesor)-> Profesor[idProfesor]!solicitarExamen(examenes.pop());
    }od
}
```

## Ejercicio 4

En  una  exposición  aeronáutica  hay  un  simulador  de  vuelo  (que  debe  ser  usado  con exclusión  mutua)  y  un  empleado  encargado  de  administrar  su  uso.
Hay  P  personas  que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira. 
Nota: cada persona usa sólo una vez el simulador.   

a) Implemente  una  solución  donde  el  empleado  sólo  se  ocupa  de  garantizar  la  exclusión 
mutua. 
```cpp
process Empleado{
    for(i=1; i<=P;i++){
        Persona[*]?SolicitarUso(idPersona);
        Persona[idPersona]!darAcceso();
        Persona[idPersona]?fin();
    }
}

procces Persona[id: 1..P]{
    Empleado!solicitarUso(id);
    Empleado?darAcceso();
    //USO EL SIMULADOR UN "RATO"
    Empleado!fin();
}

```

b) Modifique la solución anterior para que el empleado considere el orden de llegada para 
dar acceso al simulador. 

## CHEQUEAR CON IVO
```cpp
process Empleado{
    for(i=1; i<=P;i++){
        Admin!solicitarPersona();
        Admin?siguiente(idPersona);
        Persona[idPersona]!darAcceso();
        Persona[idPersona]?fin();
    }
}

procces Persona[id: 1..P]{
    Admin!solicitarUso(id);
    Empleado?darAcceso();
    //USO EL SIMULADOR UN "RATO"
    Empleado!fin();
}
process Admin{
    cola colaEspera; int cant=0;
    do{
        (cant<P) Persona[*]!solicitarUso(id)->cant++; colaEspera.push(id);
        (!colaEspera.empty()) Empleado?solicitarPersona()-> Empleado!siguiente(colaEspera.pop());
    }od
}
```

## Ejercicio 5

En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E  Espectadores  de  acuerdo  con  el  orden  de  llegada.
Cuando  el  espectador  accede  a  la máquina  en  su  turno  usa  la  máquina  y  luego  se  retira  para  dejar  al  siguiente.
Nota:  cada Espectador una sólo una vez la máquina.
## CHEQUEAR CON IVO

```cpp
procedure Espectador [id:1..E]{
    Admin!solicitarUso(id);
    Admin?darAcceso();
    //USO LA MAQUINA EXPENDEDORA.
    Admin!fin();
}

procedure admin{
    cola colaEspera;
    boolean ocupado=false
    do{
        (cant<E) Espectador[*]?solicitarUso(idE) -> 
            if (ocupado){
                colaEspera.push(idE);
            }else{
                ocupado= true;
                Persona[idE]!darAcceso();
                cant++;
            }
        (cant<E) Espectador[*]?fin() ->                         //me falta regular el corte, por que de esta manera no aceptaria el ultimo fin. CREO
            if((!colaEspera.empty)){
                Espectador[colaEspera.pop()]!darAcceso();
                cant++;
            }else{
                ocupado=False;
            }
    }od
}

```
