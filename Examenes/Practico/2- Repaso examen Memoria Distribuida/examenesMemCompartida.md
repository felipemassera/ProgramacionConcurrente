# MD del 08-11-22
## PMA
1. Resolver con PMA el siguiente problema. Se debe modelar el funcionamiento de una casa de venta de repuestos 
automotores, en la que trabajan *V vendedores* y que debe atender a *C clientes*. El modelado debe considerar que: 
(a) cada cliente realiza un pedido y luego espera a que se lo entreguen; 
(b) los pedidos que hacen los clientes son tomados por cualquiera de los vendedores.
Cuando no hay pedidos para atender, los vendedores aprovechan para 
controlar el stock de los repuestos (tardan entre 2 y 4 minutos para hacer esto). 

Nota: maximizar la concurrencia.
### para maximixar la concurrencia cuando tenemos V y C procesos y uno de ellos hace mas de una cosa SIEMPRE usamos COORDINADOR que pregunta si el canal esta vacio o no!
```SH                                       NxN uso COORDINADOR
chan vendedorLibre(int);
chan solicitudCliente(int,Pedido);
chan atender[V](int,Pedido)
chan listoPedido(Pedido)

process cliente[id: 1..C]{
    Pedido pedido;
    send solicitudCliente(id, new pedido());
    receive listoPedido(pedido);
}
process vendedor[id: 1..v]{
    integer idC;
    text pedido;
    while(true){
        send vendedorLibre(id);
        receive atender(idC,pedido);
        if(idC==-1){
            delay 120.0 / 240.0;
        }else{
            //PREPARO EL PEDIDO
            send listoPedido[idC](pedido);
        }
    }
}

process coordinador{
    integer idClient;
    while(true){
        idCient = -1;
        receive vendedorLibre(idV);
        if (not solicitudCliente.empty()){
            receive solicitudCliente(idC,pedido);
            idClient= idC;
        }
        send atender[idV](idC)
    }
}

```

## ADA
2. Resolver  con  ADA  el  siguiente  problema.  Se  quiere  modelar  el  funcionamiento  de  un  *banco*,  al  cual  llegan 
*clientes*  que  deben  realizar  un  pago  y  llevarse  su  comprobante.  Los  clientes  se  dividen  entre  los  *regulares*  y  los 
*premium*,  habiendo  R  clientes  regulares  y  P  clientes  premium.  
- Existe  *un  único  empleado*  en  el  banco,  el  cual atiende  de  acuerdo  al  orden  de  llegada,  pero  dando  prioridad  a  los  premium  sobre  los  regulares.
- Si  a  los  30 minutos de llegar un cliente regular no fue atendido, entonces se retira sin realizar el pago. 
- Los clientes premium siempre esperan hasta ser atendidos.
```SH
Procedure BANCO is

TASK empleado is
    ENTRY clientePremium(monto: IN Double; comprobante: OUT Text);
    ENTRY clienteRegular(monto: IN Double; comprobante: OUT Text);
end Empleado;

TASK TYPE clienteRegular;
TASK TYPE clientePremium;

TASK BODY empleado is
BEGIN
    LOOP    
        Select
            ACCEPT clientePremium(monto: IN Double; comprobante: OUT Text) do 
                comprobante= realizarPago(monto);
            END ACCEPT;
        OR  when (clientePremium`count ==0); ACCEPT clienteRegular(monto: IN Double; comprobante: OUT Text)do 
                comprobante= realizarPago(monto);
            END ACCEPT;
    END LOOP;
END BODY;

TASK BODY clienteRegular is
    montoPagar: Double; comprobante: Text;
BEGIN
    montoPagar = X;
    SELECT                                                  //espero 30 min o me voy re caliente.
        Empleado.clienteRegular(montoPagar,comprobante);
    OR  DELAY  1800.0;
END BODY;

TASK BODY clientePremium is
    montoPagar: Double; comprobante: Text;
BEGIN
    montoPagar = X;
    Empleado.clientePremium(montoPagar, comprobante);
END BODY;

BEGIN
    NULL;
END BANCO;

```

# MD del 13-11-23

## PMA
 3.  En  una  oficina  existen  100  empleados  que  envían  documentos  para  imprimir  en  5  impresoras  compartidas.  Los 
pedidos  de  impresión  son  procesados  por  orden  de  llegada  y  se  asignan  a  la  primera  impresora  que  se  encuentre 
libre. Implemente un programa que permita resolver el problema anterior usando PASAJE DE MENSAJES 
ASINCRÓNICO (PMA).

```cpp
chan enviarDoc(int, Object);
chan impresoraLibre(int);
chan imprimirDoc[5](int, Object);

Process empleado[id: 1..100] {
    Object impresion;
    while(true){
        //realiza trabajos;
        send enviarDoc(new Documento());
    }
}

process impresora[id : 1..5]{
    integer idp; 
    Object documento, impresion;
    while(true){
        send impresoraLibre(id);
        receive imprimirDoc(idP,documento);
        impresion= imprimir(documento);
    }
}
process admin {
    idImp, idE: integer;
    Object documento;
    while(true){
        receive impresoraLibre(idImp);
        receive enviarDoc(idE,documento);
        send imprimirDoc[idImp](idE,documento);
    }
}
```

## PMS
4)  Resuelva el mismo problema anterior pero ahora usando PASAJE DE MENSAJES SINCRÓNICO (PMS).
```cpp
Process empleado[id: 1..100] {
    Object impresion;
    while(true){
        //realiza trabajos;
        admin!enviarDoc(id, new Documento());
    }
}

process impresora[id : 1..5]{
    integer idp; 
    Object documento, impresion;
    boolean seguir=true;
    whiles(seguir){
        admin!impresoraLibre(id);
        admin?imprimirDoc(documento);
        if(documento!=NULL)
            impresion= imprimir(documento);
        }else{
             seguir=false
        }  
    }
}
process admin {
    idImp, idE: integer;
    Object documento;
    colaEspera: cola;
    do
        [] empleado[*]?enviarDoc(idE,documento); -> colaEspera.push(documento);
        [] !colaEspera.empy(); impresora[*]?impresoraLibre(idImp) -> impresora[idImp]imprimirDoc(colaEspera.pop());
    od
    for (i=1;i<=5;i++){
         impresora[*]?impresoraLibre(idImp)
         impresora[idImp]imprimirDoc(NULL);
    }
}
```

## ADA
5)  Resuelva con ADA el siguiente problema: la página web del Banco *Central* exhibe las diferentes cotizaciones del dólar 
oficial de 20 bancos del país, tanto para la compra como para la venta. Existe una *tarea programada* que se ocupa de 
actualizar la página en forma periódica y para ello consulta  la cotización de  cada uno de  los 20 bancos. Cada banco 
dispone  de  una  API,  cuya  única  función  es  procesar  las  solicitudes  de  aplicaciones  externas.  
La  *tarea  programada* consulta de a una API por vez, esperando a lo sumo 5 segundos por su respuesta. Si pasado ese tiempo no respondió, 
entonces se mostrará vacía la información de ese banco. 
```sh
procedure BancoCentral is

TASK Central;
TASK TYPE Sucursal is
    ENTRY solicitud(valorUSD:OUT Double,id:In integer);
END Sucursal;
sucursales: array 1..20 of Sucursal;

TASK BODY Central is
var
    valoresUSD: Array 1..20 of Double;
    i: Integer;
Begin
    LOOP
        FOR i in 1..20 LOOP
            select
                sucursal(i).solicitud(valor) do
                    valoresUSD(i)=valor;
                end solicitud;               
            or  delay 5.0;
        END LOOP;
    END LOOP;

END BODY;

TASK BODY Sucursal is
VAR
    valorUSD: DOUBLE;
Begin
    LOOP
        ACCEPT solicitud(valor:OUT DOUBLE) do
            valor=valorUSD;
        end ACCEPT;
    END LOOP;
END BODY;

BEGIN
    NULL;
END BancoCentral;

```

# Programación Concurrente ATIC – Redictado de Programación Concurrente 
##  Segunda Fecha - 01/07/2021 

### PMA
6. Resolver  el  siguiente  problema  con  Pasaje  de  Mensajes  Asincrónicos  (PMA).  En  una  empresa  de
software  hay  *3  prog ra madores*  que  deben  arreglar  errores  informados  por *N  clientes*. 
- Los  clientes continuamente están trabajando, y cuando encuentran un error envían un reporte a la empresa para que lo corrija (*no tienen que esperar a que se resuelva*).
- Los programadores resuelven los reclamos de acuerdo al orden de llegada, y si no hay reclamos pendientes trabajan durante una hora en otros programas.
  
 *Nota: los procesos  no  deben  terminar  (trabajan  en  un  loop  infinito);  suponga  que  hay  una  función ResolverError  que simula que un programador está resolviendo un reporte de un cliente, y otra Programar que simula que está trabajando en otro programa*
```SH
chan informarBUG(int);
chan reparar(int);
chan programadorLibre(int);

process Cliente[id:1..N]{
    while(true){
        //TRABAJANDO
        send informarBUG(new BUG());
    }
}

process programador[id: 1..3]{
    while(true){
        send programadorLibre(id);
        receive reparar(bug);
        if (bug==-1){
            programar();
        }else{
            ResolverError(bug);
        }
    }
}

process admin{
    idP:Integer;
    while(true){
        receive programadorLibre(idP);
        if(informarBUG.empty()){
            bug==-1;
        }else{
            receive informarBUG(bug);
        }
        send reparar(bug);

    }
}
```

### ADA
7. Resolver  el  siguiente  problema  con  ADA.  Hay  *un  sitio  web*  para  identificación  genética  que  resuelve
pedidos  de  *N  clientes*. Cada cliente trabaja continuamente  de  la  siguiente  manera: 
- genera la secuencia de ADN, la envía al sitio web para evaluar y espera el resultado, después de esto puede comenzar a generar la siguiente secuencia de ADN. (LOOP)
-  Para resolver estos pedidos el sitio web cuenta con 5 servidores idénticos que atienden los pedidos de acuerdo al orden de llegada (cada pedido es atendido por un único servidor).
   Nota:  maximizar  la  concurrencia.  Suponga  que  los  servidores  tienen  una  función  ResolverAnálisis  que  recibe  la
secuencia de ADN y devuelve un entero con el resultado
```SH
Procedure  WEB is
TASK admin is
    Entry solicitudCliente(idC: IN integer; adn: IN integer);
    ENTRY servidorLibre(idC: OUT integer; adn: OUT integer);
end Admin;

TASK TYPE Cliente is
    ENTRY identificar(id:IN integer);
    ENTRY resultadoCliente(res: In integer);
END cliente;

TASK TYPE servidor;

clientes: array (1..N) of cliente;
servidores: array (1..5) of servidor;

TASK BODY admin is
var
    cola: queue;
    id,muestra: integer;
begin
    LOOP
        ACCEPT servidorLibre(id: OUT integer; adn : OUT integer) do
            ACCEPT solicitudCLiente(idC: IN integer; muestra: IN integer)do
                id= idC;
                adn=muestra;
            end solicitudCLiente;
        end servidorLibre;
    END LOOP;
end BODY;

TASK BODY CLIENTE is
VAR
    id, muestra, resultado:integer;
BEGIN
    ACCEPT identificar(idC: IN integer) do
        id=idC
    end ACCEPT;
    LOOP    
        muestra = generarMuestra();
        admin.solicitudCliente(id,muestra);
        ACCEPT resultadoCliente(res : IN integer) do 
            resultado = res;
        end ACCEPT;
    END LOOP;
END BODY;

TASK BODY Servidor is
VAR
    idC, muestra, resultado: integer;
BEGIN
    LOOP
        admin.servidorLibre(idC, muestra);
        resultado= ResolverAnálisis(muestra);
        Cliente[idC].resultadoCliente(resultado);
    END LOOP;
END BODY;

BEGIN
    NULL;
END WEB;
```

# Programación Concurrente ATIC – Redictado de Programación Concurrente 
## Tercera Fecha - 15/07/2021

### PMS
8. Resolver el siguiente problema con Pasaje de Mensajes Sincrónicos (PMS). En una excursión hay una 
tirolesa que debe ser usada por *20 turistas*. Para esto hay un *guía* y un *empleado*. 
- El empleado espera a que  todos  los  turistas  hayan  llegado  para  darles  una  charla explicando  las  medidas  de  seguridad.Cuando  
termina  la  charla  los  turistas  piden  usar  la  tirolesa  y  esperan  a  que  el  guía  les  vaya  dando  el  permiso  de  
tirarse.
- El  guía  deja  usar  la  tirolesa  a  un  cliente a  la  vez y  de  acuerdo  al  orden  en  que  lo  van  solicitando. 
Nota: todos los procesos deben terminar;  suponga que el empleado tienen una función DarCharla() que 
simula que el empleado está dando la charla, y los turistas tienen una función UsarTitolesa() que simula que 
está usando la tirolesa. 
```CPP
process empleado{
    For(i=1;i<20;i++){
        cliente[*]?llegue();
    }
    darCharla();
    For(i=1;i<20;i++){
        cliente[*]?finCapacitacion();
    }
}
process CLiente[id:1..20]{
    empleado!llegue()
    empleado!finCapacitacion();
    coordinador!solicitarTirolesa(id);
    guia?acceder();
    usarTirolesa();
    guia!liberar();
}
process guia{               
    for(i=1;i<=20;i++){         
        coord!empleadoLibre();  
        coord?siguiente(id);    
        Cliente[id]!acceder();  
        Cliente[id]?liberar();  
    }                           
}                               
process coordinador{                   
    int cant=0; Cola cola;
    DO
        () cant<20; cliente[*]?solicitarUso(id)->
            cola.push(id); cant++;
        () !cola.empty(); guia?guiaLibre()->
            guia!siguiente(cola.pop());
    OD
}
```

### ADA
9. Resolver el siguiente problema con ADA. En un negocio hay un empleado para atender los pedidos de 
clientes  de  *N  clientes*;  algunos  clientes  son  *ancianos  o  embarazadas*.  El  empleado  atiende  los  pedidos  de  
acuerdo a la siguiente prioridad: *primero las embarazadas, luego los ancianos, y por último el resto*. Cada 
cliente  hace  sólo  un  pedido  de  la  siguiente  manera:  en  el  caso  de  las  embarazadas,  si  no  son  atendidas  
inmediatamente se retiran; en el caso de los ancianos, esperan a los sumo 5 minutos a ser atendidos, y si 
no se retira; cualquier otro cliente espera si o si hasta ser atendido. El empleado atiende a los clientes de 
acuerdo al orden de llegada pero manteniendo las siguientes prioridades: primero las embarazadas, luego 
los  ancianos  y  luego  el  resto.  Nota:  suponga  que  existe  una  función  AtenderPedido()  que  simula  que  el  
empleado está atendiendo a un cliente.
```SH
PROCEDURE NEGOCIO IS
TASK TYPE CLIENTE;
TASK EMPLEADO IS
    ENTRY colaEmbarazo();
    ENTRY colaAnciano();
    ENTRY colaNormal();
END EMPLEADO;
clientes: Array (1..N) of cliente;

TASK BODY EMPLEADO IS
BEGIN
    LOOP
        SELECT
            ACCEPT colaEmbarazo()do
                AtenderPedido();
            end colaEmbarazo;
        OR  
            WHEN (colaEmbarazo`count==0) ACCEPT colaAnciano()do
                AtenderPedido();
            END colaAnciano
        OR
            WHEN (colaEmbarazo`count==0 and colaAnciano`count==0) ACCEPT colaNormal() do
                AtenderPedido();
            end colaNormal;
        end select;
    END LOOP;
END EMPLEADO;

TASK BODY Cliente is
Var
    embarazada, anciano: boolean ;
Begin
    embarazada=X; anciano=X;
    If (embarazada) then
        Select 
            empleado.colaEmbarazo();
        else
            null;
        end select;
    elif (anciano) then
        Select 
            empleado.colaAnciano();
        or
            delay 300.0;  
        end select;
    else then
        empleado.colaNormal();
    end if;
end Cliente;

BEGIN 
    NULL;
END NEGOCIO;
```

# parcial práctico del 18-12-23

### PMA
10. Resolver con PASAJE DE MENSAJES ASINCRÓNICO (PMA) el siguiente problema. En un negocio de cobros digitales 
hay *P personas* que deben pasar por la única *caja* de cobros para realizar el pago de sus boletas. Las personas son 
atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de *5 boletas* de 
los que pagan más. Adicionalmente, las personas embarazadas tienen prioridad sobre los dos casos anteriores. Las 
personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los recibos de pago.
```SH
chan colaEmbarazo(int,double, text);
chan colaMenosCinco(int,double, text);
chan colaNormal(int,double, text);
chan rta[p](double, text);
chan hayCliente();

procedure Persona[id:1..P]{
    boolean embarazada= X;
    integer cantBoleta= X;
    double monto, vuelto;
    if (embarazada){
        send colaEmbarazo(id,monto,boleta);
    }elif (cantBoleta<5){
        send colaMenosCinco(id,monto,boleta);
    }else{
        send colaNormal(id,monto,boleta);
    }
    send hayCliente();                          //primero me encolo y luego informo que estoy para ser atendido.
    receive rta(vuelto, comprobante);
}

procedure empleado{
    double monto, vuelto;
    for (i=1;i<=P;i++){
        receive hayCliente();                   //me avisa que hay cliente esperanco en alguna cola de atencion;
        if (not colaEmbarazo.empty()){
         receive colaEmbarazo(id,monto, boleta);
        }else{
            if (not colaMenosCinco.empty()){
                receive colaMenosCinco(id,monto, boleta);
            }else{
                receive colaNormal(id,monto, boleta);
            }
        }   
        vuelto, comprobante= CobrarBoletas(monto,boletas);
        send rta[id](vuelto, comprobante);
    }
}

```

### ADA                 YA LO HICE
11) Resolver con  ADA el siguiente problema. La oficina *central* de  una empresa de  venta de  indumentaria debe calcular 
cuántas  veces  fue  vendido  cada  uno  de  los  artículos de  su  catálogo. La empresa  se  compone  de  *100  sucursales* y  cada 
una de ellas maneja su propia base de datos de ventas. La oficina central cuenta con una herramienta que funciona de la 
siguiente  manera:  ante  la  consulta  realizada  para  un  artículo  determinado,  la  herramienta  envía  el  identificador  del 
artículo a cada una de las sucursales, para que cada uno de éstas calcule cuántas veces fue vendido en ella. Al final del 
procesamiento,  la  herramienta  debe  conocer  cuántas  veces  fue  vendido  en  total,  considerando  todas  las  sucursales. 
Cuando ha terminado de procesar un artículo comienza con el siguiente (suponga que la herramienta tiene una función 
generarArtículo  que  retorna  el  siguiente  ID  a  consultar).  Nota:  maximizar  la  concurrencia.  Supongo  que  existe  una 
función  ObtenerVentas(ID) que  retorna la cantidad de veces  que fue vendido el artículo con identificador ID en la base 
de datos de la sucursal que la llama.
```SH

task body central

    loop
        modelo= QUEMODELO()
        for i in 1..200 loop
            select 
                ACCEPT servLibre(mod:out integer) do
                    mod=modelo;
                end servlibre
            or
                ACCEPT Resultado(cant): in integer) do
                    total= total+cant;
                end resultado;
            end select
        end loop;
        print("modelo:"+ modelo+ "cantidad: " + total)
        for i in 1..100 loop
            ACCEPT FINVUELTA();
        endLOOP

    end loop;
end central
task sede
    loop
        central.servLibre(modelo)
        cant= sumar(modelo);
        central.resultado(cant)
        central.finVuelta();
    end Loop;
end sede;    

```
#  parcial práctico del 13-12-22

### PMS
12. Resolver con Pasaje de Mensajes Sincrónicos (PMS) el siguiente problema. En un comedor estudiantil hay *un horno microondas* 
que debe ser usado por *E estudiantes de acuerdo con el orden de llegada*. Cuando el estudiante accede al horno, lo usa y luego se 
retira para dejar al siguiente. Nota: cada Estudiante una sólo una vez el horno.
#### AGREGO YO EL QUE TODOS LOS PROCEDIMIENTOS DEBEN TERMINAR
```SH                         passing the baton
procedure estudiante[id:1..E]{
    coord!solicitarUso(id);
    coord?usar()
    //USAR MICROONDAS
    coord!fin();
}

procedure coord{
    boolean libre=true;seguir= true;
     cola cola;
    integer contador=0;
    do
        () contador<E ; Estudiante[*]?solicitarUso(id) ->
            if (libre){
                libre=false;
                estudiante[id]!usar();
            }else{
                cola.push(id);
            }
        () contador<E; Estudiante[*]?fin() ->           //solose cuenta cuando se libera el recurso
            contador++;
            if(!cola.empty()){
                estudiante[cola.pop()]!usar();
            }else{
                libre=true;
            }
    od
}
```

### ADA     hecha en papel
13. Resolver  con  ADA  el  siguiente  problema.  Se  debe  controlar  el  acceso  a  una  base  de  datos.  Existen  *L  procesos  Lectores  y  E procesos Escritores*
    que trabajan indefinidamente de la siguiente manera: 
• Escritor:  intenta  acceder  para  escribir,  si  no  lo  logra  inmediatamente,  espera  1  minuto  y  vuelve  a  intentarlo  de  la  misma 
manera. 
• Lector: intenta acceder para leer, si no lo logro en 2 minutos, espera 5 minutos y vuelve a intentarlo de la misma manera. 
Un proceso Escritor podrá acceder si no hay ningún otro proceso  usando la base de datos; al acceder escribe y sale de la BD. Un 
proceso Lector podrá acceder si no hay procesos Escritores usando la base de datos; al acceder lee y sale de la BD. Siempre se le 
debe dar prioridad al pedido de acceso para escribir sobre el pedido de acceso para leer. 
```SH
procedure ej13 is
task type escritor
task type lector
task bd is
    entry accesoLector();
    entry accesoEscritor();
    entry finEscritor();
    entry finLector();
end bd;
lectores ; array(1..E) of lector
escritores ; array(1..E) of escritor

task body escritor is
    loop
        select 
            bd.accesoEscritor
            escribirBD();
            bd.finEscritor();
        else
            delay 60;
    ebd loop
end escritor
task body Lector is
    loop
        select 
            bd.accesoLector();
            leerBD();
            finLector();
        or  delay 120;
            delay 300;
    ebd loop
end escritor
task body bd is
    integer lector=0;
    loop        
        select
            when (lectores==0 ) ACCEPT accesoEscritor();
            ACCEPT finEscritor();
        OR  
            when (accesoEscritor`count ==0) ; ACCEPT accesoLector();
                lector ++;
        OR
            ACCEPT finLector();
                lector --;
        end select;
    end loop;
end bd;
begin
    null
end ej13;
```
 
# primer recuperatorio del parcial práctico

### PMS //coord entre comp y supervisores?
14. Resolver con Pasaje de Mensajes Sincrónicos (PMS) el siguiente problema. En un torneo de programación 
hay *1 organizador, N competidores y S supervisores*. 
- El organizador comunica el desafío a resolver a cada competidor. 
- Cuando un competidor cuenta con el desafío a resolver, lo hace y lo entrega para ser evaluado. 
A continuación, espera a que alguno de *los supervisores lo corrija y le indique si está bien*. En caso de tener  errores, el competidor debe corregirlo y volver a entregar,
 repitiendo la misma metodología hasta que llegue a la solución esperada.
- Los supervisores corrigen las entregas respetando el orden en que los competidores van entregando.
 Nota: maximizar la concurrencia y no generar demora innecesaria.
```SH
process competidor [id: 1..N]{
    organizador!pedirDesafio(id);
    organizador?recibirDesafio(desafio);
    examen= resolver(desafio);
    corrdinador.corregir(examen,id);
    supervisor[*]?correccion(cumple);
    while (!cumple){
        examen= resolver(desafio);
        corrdinador.corregir(examen,id);
        supervisor[*]?correccion(cumple);
    }
}
process supervisor[id:1..S]{
    while(true){
        coordinador!libre(id);
        coordinador?siguiente(idA,examen);
        cumple= corregir(examen);
        competidor[idA]!correccion(cumple);
    }
}
process organizador{
    for(i=1;i<=N;i++){
        competidor[*]?pedirDesafio(idC);
        competidor[idC]!recibirDesafio(New desafio());
    }
}
process coordinador {
    cola cola;
    do
        () competidor[*]?corregir(examen, id) -> cola.push(id,examen);
        () !cola.empty(); supervisor[*]?libre(idS)-> supervisor[idS]!siguiente(cola.pop());
    od
}

```
////////////////////////////////
### ADA
15. Resolver con ADA el siguiente problema. Una empresa de venta de calzado cuenta con *S sedes*. En la *oficina central* 
de la empresa se utiliza un sistema que permite controlar el stock de los diferentes modelos, ya que 

*cada sede tiene una base de datos propia*. El sistema de control de stock funciona de la siguiente manera: 
dado un modelo determinado, lo envía a las sedes para que cada una le devuelva la cantidad disponible en 
ellas; al final del procesamiento, el sistema informa el total de calzados disponibles de dicho modelo. Una 
vez que se completó el procesamiento de un modelo, se procede a realizar lo mismo con el siguiente modelo. 
Nota: suponga que existe una función DevolverStock(modelo,cantidad) que utiliza cada sede donde recibe 
como parámetro de entrada el modelo de calzado y retorna como parámetro de salida la cantidad de pares 
disponibles. Maximizar la concurrencia y no generar demora innecesaria
```SH
```

# Programacion_Concurrente___Resolucion_Examenes_MD
## Parcial MC - 2020 - 1 - Tema 6

### PMA
16. Resolver con PASAJE DE MENSAJES ASINCRÓNICOS (PMA) el siguiente problema. Se debe simular la atención en un banco con 3 cajas para atender
a N clientes que pueden ser especiales (son las embarazadas y los ancianos) o regulares. Cuando el cliente llega al banco se dirige a la caja con menos
personas esperando y se queda ahí hasta que lo terminan de atender y le dan el comprobante de pago. Las cajas atienden a las personas que van a ella de
acuerdo al orden de llegada pero dando prioridad a los clientes especiales; cuando terminan de atender a un cliente le debe entregar un comprobante de
pago. Nota: maximizar la concurrencia. Respuesta:
```SH
process cliente [id:1..N]{
    boolean especial=X;

        send llegue(id);
        send atender();
        recieve irCaja[id](idCaja);
        if(especial){
            cajaEspecial[idCaja](id);
        }else
            cajaNormal[idCaja](id);
        }
        send tenesPersona[idCaja]();
        recive resultado[id](comprobante);
}
process caja[id:0..2]{
    while (true){
        receive tenesPersona[id]();
        if (!cajaEspecial[id].empty()){
            receive cajaEspecial[id](idC);
        [] (cajaEspecial[id].empty() and !cajaNormal[id].empty())
            receive cajaNormal[id](idC);
    }
        comprobante=AtenderCliente(idC);
        send resultado[idC](comprobante);
        send finCaja(id);
        send atender();
    }
}

process coord {
    contCajas= array(0..2) ([3]0);
    while(true){
        receive atender();
        if(not Empty(llegue)){
            receive llegue(idP);
            idC=min(contCajas);
            contCajas[idC]++;
            send irCaja[idP](idC);
        
        [] (not empty(finCaja))
            receive finCaja(idC);
            contCajas[idC]--;
        }
    }   
}   

```
### PMS
17. Resolver con PMS (Pasaje de Mensajes SINCRONICOS) el
siguiente problema. En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de
administrar el uso del mismo. A su vez hay P personas que van a la exposición y solicitan usar el simulador, cada una de ellas espera a que el empleado lo
deje acceder, lo usa por un rato y se retira para que el empleado deje pasar a otra persona. El empleado deja usar el simulador a las personas respetando
el orden en que hicieron la solicitud. Nota: cada persona usa sólo una vez el simulador. Respuesta:
```SH
```

### PMS
18. Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. Simular la atención de una estación de servicio con un único surtidor que
tiene un empleado que atiende a los N clientes de acuerdo al orden de llegada. Cada cliente espera hasta que el empleado lo atienda y le indica qué y
cuánto cargar; espera hasta que termina de cargarle combustible y se retira. Nota: cada cliente carga combustible sólo una vez; todos los procesos deben
terminar. Respuesta:
```SH
```

### PMS
19. En una carrera hay C corredores y 3 Coordinadores. Al llegar los corredores deben dirigirse a los coordinadores para que cualquiera de ellos le dé el número
de “chaleco” con el que van a correr y luego se va. Los coordinadores atienden a los corredores de acuerdo al orden de llegada (cuando un coordinador está
libre atiende al primer corredor que está esperando). Nota: maximizar la concurrencia. Respuesta
```SH
```

### ADA
20. Resolver el siguiente problema con ADA. Hay un sitio web para identificación genética que resuelve pedidos de N clientes . Cada cliente trabaja
continuamente de la siguiente manera: genera la secuencia de ADN, la envía al sitio web para evaluar y espera el resultado; después de esto puede
comenzar a generar la siguiente secuencia de ADN. Para resolver estos pedidos el sitio web cuenta con 5 servidores idénticos que atienden los pedidos de
acuerdo al orden de llegada (cada pedido es atendido por un único servidor). Nota: maximizar la concurrencia. Suponga que los servidores tienen una
función ResolverAnálisis que recibe la secuencia de ADN y devuelve un entero con el resultado.
Respuesta:
```SH

```

### ADA
Consigna:
21- Resolver con ADA la siguiente situación. En una obra social que tiene 15 sedes en diferentes lugares se tiene información de las enfermedades de cada
uno de sus clientes (cada sede tiene sus propios datos). Se tiene una Central donde se hacen estadísticas, y para esto repetidamente elige una enfermedad
y debe calcular la cantidad total de clientes que la han tenido. Esta información se la debe pedir a cada Sede. Maximizar la concurrencia. Nota: existe
una función ElegirEnfermos(e) que es llamada por cada Sede y devuelve la cantidad de clientes de esa sede que han tenido la enfermedad e.
Respuesta:
```SH
```

### ADA
22. En una *playa* hay *5 equipos* de *4 personas* cada uno (en total son 20 personas donde cada una conoce previamente a que equipo pertenece). [Cuando las
personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes)], [a partir de ese momento el equipo
comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y
se suman los montos de las 60 monedas conseguidas en el grupo.] [Al finalizar cada persona debe conocer el grupo que más dinero junto]. Nota: maximizar
la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la
moneda encontrada. Respuesta
```SH
procedure EJ22 is
task playa is
    ENTRY finEquipo(total:in integer; idE: in integer);
    ENTRY ganador(idM:out integer);
end playa;
task type equipo is
    ENTRY llegoBarrera();
    ENTRY finBarrera();
    Entry sumarMonedaE(cant:in integer);
    Entry identificar(idE:in integer);
task type Persona;
personas: array (1..20) of persona;
equipos: array(1..5) of equipo;

task body persona is
    eq, cant, idGanador:integer;
begin
    eq=X;
    cant=0;
    Equipo[eq].llegoBarrera();
    Equipo[eq].finBarrera();
    for i in (1..15) LOOP
        cant= cant+ Moneda();
    end loop;
    Equipo[eq].sumarMonedaE(cant);
    playa.Ganador(idGanador);
end persona;

Task body Equipo is
    id, totalE:integer
begin
    totalE=0;
    ACCEPT identificar(idE:in integer) do 
        id=idE;
    end identificar;
    for i in (1..4) loop
        ACCEPT llegobarrera()
    end loop;
    for i in (1..4) loop
        ACCEPT finbarrera()
    end loop;
    for i in (1..4) loop
        ACCEPT sumarMonedaE(cant: in integer)do
            total=total+cant;
        end sumarMonedaE;
    end loop;
    Playa.finEquipo(total,id);
end Equipo;
task body playa is
    idMax,cantMax:integer
begin
    cantMax=-1; idMax=-1;
    for i in (1..5) loop
        ACCEPT finEquipo(total:IN integer, idE:inEquipo) do
            if (total>cantMax){
                idMax=idE;
                cantMax=total;
            }
        end finEquipo;
    end loop;
    for i in (1..20) loop
        ACCEPT ganador(idG: OUT integer)do
            idG=idMax;
        end ganador;
    end loop;
end playa;

begin

end ej22;

```

### ADA
23. En un sistema para acreditar carreras universitarias, hay *UN Servidor* que atiende pedidos de *U Usuarios* de a uno a la vez y de acuerdo con el orden en
que se hacen los pedidos. Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la respuesta de este que le indica si está
todo bien o hay algún error. Mientras haya algún error,vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que
está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 2 minutos a que sea recibido por el servidor, pasado ese tiempo
espera un minuto y vuelve a intentarlo (usando el mismo documento). Respuesta:
```SH
procedure universidad is

task type usuario is

task servidor is
    ENTRY pedido(trabajo: in txt, cumple:out boolean );
end servidor
usuarios: arrayt(1..U)of usuario;

task body usuario is

begin
    trabajo=hacerTrabajo();
    cumple=false;
    select
        servidor.pedido(trabajo, cumple);
    or  delay 120.0;
            delay 60.0;
    end select;
    while(!cumple)
        trabajo=hacerTrabajo();
        select
            servidor.pedido(trabajo, cumple);
        or  delay 120.0;
                delay 60.0;
        end select;
    end while;
end usuario;

task body servidor is
    loop
        ACCEPT pedido(trabajo: in txt, cumple:out boolean ) do
            cumple= verificar(trabajo);
        end pedido;
    end loop
end servidor;

begin 

end universidad

```

### ADA
24. Resolver con ADA el siguiente problema. Simular la venta de entradas a un evento musical por medio de un portal web. Hay N clientes que intentan
comprar una entrada para el evento; los clientes pueden ser regulares o especiales (clientes que están asociados al sponsor del evento). Cada cliente especial
hace un pedido al portal y espera hasta ser atendido; cada cliente regular hace un pedido y si no es atendido antes de los 5 minutos, vuelve a hacer el
pedido siguiendo el mismo patrón (espera a lo sumo 5 minutos y si no lo vuelve a intentar) hasta ser atendido. Después de ser atendido, si consiguió
comprar la entrada, debe imprimir el comprobante de la compra.
El portal tiene Z entradas para vender y atiende los pedidos de acuerdo al orden de llegada pero dando prioridad a los Clientes Especiales. Cuando
atiende un pedido, si aún quedan entradas disponibles le vende una al cliente que hizo el pedido y le entrega el comprobante.
Nota: no debe modelarse la parte de la impresión del comprobante, sólo llamar a una función Imprimir (comprobante) en el cliente que simulará esa
parte; la cantidad E de entradas es mucho menor que la cantidad de clientes (E « C); todas las tareas deben terminar.
Respuesta:
```SH
```

### ADA
25. Resolver con ADA el siguiente problema. Simular la atención en una oficina donde atiende un empleado. Hay N personas que deben hacer UN trámite
cada uno. Cuando una persona llega le da los datos al empleado y espera a que este resuelva el trámite y le entregue un comprobante. Por otro lado está el
director de la oficina, el cual necesita 5 informes del empleado; cada vez que necesita un informe se lo pide al empleado y si no lo atiende inmediatamente
sigue trabajando durante 10 minutos y luego lo vuelve a intentar siguiendo el mismo patrón (si no lo atiende inmediatamente trabaja durante 10 minutos
y luego lo vuelve a intentar) hasta que el empleado lo atienda y le dé el informe pedido. El empleado atiende los pedidos de acuerdo al orden de llegada
pero siempre dando prioridad a los pedidos de las personas. Nota:
todas las tareas deben terminar. Respuesta:
```SH
```

### ADA
26. Resolver con ADA el siguiente problema. Un programador contrató a 5 estudiantes para testear los sistemas desarrollados por él. Cada estudiante tiene
que trabajar con un sistema diferente y debe encontrar 10 errores (suponga que seguro existen 10 errores en cada sistema) que pueden ser: urgentes,
importantes, secundarios. Cada vez que encuentra un error se lo reporta al programador y espera hasta que lo resuelva para continuar. El programador
atiende los reportes de acuerdo al orden de llegada pero teniendo en cuenta las siguientes prioridades: primero los urgentes, luego los importantes y por
último los secundarios; si en un cierto momento no hay reportes para atender durante 5 minutos trabaja en un nuevo sistema que está desarrollando. De-
spués de resolver los 50 reportes de error (10 por cada estudiante) el programador termina su ejecución. Nota: todas las tareas deben terminar. Respuesta:
```SH
```

### PMA
27) Resolver con PMA el siguiente problema. Para una aplicación de venta de pasaje se tienen 3 servidores replicados para mejorar la eficiencia en la
atención. Existen N clientes que hacen alguna de estas dos solicitudes: compra de un pasaje o devolución de un pasaje. Las solicitudes se deben atender
dando prioridad a las solicitudes de compra. Nota: suponga que cada cliente llama a la función TipoSolicitud() que le devuelve el tipo de solicitud a
realizar. Maximizar la concurrencia. Respuesta:
```SH
chan comprar(int);
chan devolucion(int);
chan respuesta[N](txt);
chan hayCliente();
chan servidorLibre(int)
chan atender(int)

process cliente[id:1..N]{
    opcion= tipoSolicitud();
    if (opcion == "comprar"){
        send comprar(id);
    }else{
        send devolucion(id);
    }
    send hayCliente();
    receive restuesta[id](comprobante);
}
process servidor[id:1..3]{
    send servidorLibre(id);
    receive atender(idC);
    comprobante= atender(idC);
    send respuesta[idC](comprobante);
}

process coordinador{
    while(true){
        receive servidorLibre(idS);
        receive hayCliente();
        if(not empty(comprar)){
            receive comprar(idC);
        }else{

            receive devolucion(idC);
        }
        send atender(idC);
    }
}

```

### ADA
28. Resolver con ADA el siguiente problema. Se debe simular un juego en el que participan *30 jugadores* que forman *5 grupos* de 6 personas. Al llegar cada
jugador debe buscar las instrucciones y el grupo al que pertenece en un cofre de cemento privado para cada uno; para esto deben usar un único martillo
gigante de a uno a la vez y de acuerdo al orden de llegada. Luego se debe juntar con el resto de los integrantes de su grupo y los 6 juntos realizan
las acciones que indican sus instrucciones. Cuando un grupo termina su juego le avisa a un *Coordinador* que le indica en qué orden término el grupo.
Nota 1: maximizar la concurrencia; suponer que existe una función Jugar() que simula que los 6 integrantes de un grupo están jugando juntos;
    el programa debe finalizar,, y cada untegrante del equipo debe saber en que posicion termino;
NOTA 2: suponga que existe una función Romper(grupo) que simula cuando un jugador está rompiendo su cofre con el martillo y le retorna el grupo al que pertenece. Respuesta
```SH
procedure Juego is
task type jugador;
task type grupo is
    ENTRY llegadaBarrera()
    ENTRY finBarrera()
end grupo
task admin is
    ENTRY finEquipo(pos: in integer);
end admin;
jugadores : array (1..30) of jugador;
equipos : array (1..5) of equipo;

task body jugador is

begin
    admin.solicitarMartillo();
    romper(grupo);
    admin.liberar();
    equipo[grupo].llegueBarrera();
    equipo[grupo].quieroPOS(pos);
end jugador;

task body equipo is

begin 
    for i in 1..6 loop
        ACCEPT llegoBarrera();
    end loop
    jugar();
    admin.finEquipo(pos);
    for i in 1..6 loop
        ACCEPT quieroPOS(posi:out integer) do
            posi=pos;
        end quieroPOS
    end LOOP;
end equipo;  

task body admin is
 integer pos=1;
begin
    for i in 1..65 loop
        select
            when(libre); ACCEPT solicitarMartillo();
                libre=false;
        OR      
            ACCEPT liberar();
                libre=true;
        OR 
            ACCEPT finEquipo(posicion:out integer)do
                posicion=pos;
                pos++;
            end finEquipo;
    end loop;
end admin;

begin
    null;
end juego;
```

### PMS
1.  Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. Hay *un teléfono público* que debe ser usado por *U usuarios* de acuerdo al
*orden de llegada* (se debe usar con exclusión mutua). El usuario debe esperar su turno, usa el teléfono y luego lo deja para que el siguiente lo use. Nota:
cada usuario usa sólo una vez el teléfono. Respuesta:
```sh
process usuario[id: 1..U]{
    buffer!solicitarUso(id);
    buffer?acceder(id);
    UsarTelefono();
    buffer!salir();
}

process buffer{
    boolean ocupado=false;
    int cont=0;
    do
        () cont<U; usuario[*]?soliciarUso(idU)-> 
            IF(ocupado){
                cola.push(idU);
            }else{
                ocupado=true;
                usuario[idU]!acceder();
            }
        () cont<U; usuario[*]?liberar() -> 
            cont++;
            IF(!cola.empty()){
                usuario[cola.pop()]!acceder();
            }else{
                ocupado=false;
            }
    od
}


```

### PMS
30. Resolver con PMS el siguiente problema: Se debe administrar el acceso para usar un determinado *Servidor* donde no se permite más de *10 usuarios* trabajando al mismo tiempo, por cuestiones de rendimiento.
 Existen N usuarios que solicitan acceder al Servidor, esperan hasta que se les de acceso para trabajar en él y luego salen del mismo. Nota: suponga que existe una función *TrabajarEnServidor()* que llaman los usuarios para representar que están trabajando en el Servidor.

```CPP                            
process usuarios[id:1..10]{
    while(true){
        admin!solicitarUso(id);
        admin?accesoAprobado();
        trabajarEnServidor();
        admin!salir();
    }
}

process admin[]{
    Cola colaEspera;
    integer cantLibre=10;
    do
        () usuario[*]?solicitarUso(id) ->
            if (cantLibre>0){
                cantLibre--;
                usuario[id]!accesoAprobado();
            }else{
                colaEspera.push(id);
            }
        () usuario[*]?salir( ) -> 
            if(!colaEspera.empty()){
                usuario[colaEspera.pop()]!accesoAprobado();
            }else{
                cantLibre++;
            }
    od
}
```
# ADA
31. Resolver con ADA el siguiente problema. Para un experimento se tiene una red con 150 controladores de temperatura y un módulo central. Cada controlador toma la temperatura cada 30 segundos y si está fuera de rango realiza lo siguiente: si está por encima del rango espera a que la central que le indique qué acción realizar; si está por debajo del rango espera a lo sumo 10 minutos a que la central le indique qué acción realizar; en cualquiera de los dos casos, realiza la acción indicada por la central (si fue atendido). La central atiende los pedidos de los controladores dando prioridad a aquellos que son por superar el rango permitido. Nota: suponga que existen las funciones medir() que retorna la temperatura al controlador; actualizar() que simula que el controlador está haciendo la acción indicada por la central; determinar() que es usado por la central para determinar qué acción debe hacer el controlador en base a la información que le envió; el experimento nunca finaliza.