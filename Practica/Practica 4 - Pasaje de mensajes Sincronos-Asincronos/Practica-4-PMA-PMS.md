# Practica 4 PMA // PMS 

# PMA

## Ejercicio 1
Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados.
 Analice el problema y defina qué procesos, recursos y canales/comunicaciones serán necesarios/convenientes
 para resolverlo. Luego, resuelva considerando las siguientes situaciones:
a. Existe un único empleado, el cual atiende por orden de llegada.
```cpp
chan siguiente[N](String); chan avisoLlegada(string);chan fin(string);

process Empleado{
    int idProx;
    for (i:1; i<N; i++){
        receive avisoLlegada(idProx);
        send siguiente[idProx]("Sos el siguiente");
        //atiende al cliente
        send fin("finalizado");
    }
}
process CLiente [id: 1..N]{
    string rta; string fin;
    send avisoLlegada(id);
    receive siguiente[id](rta);
    receive fin(fin);
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
chan siguiente[N](String, int); chan avisoLlegada(string);chan fin[1..2](string);

process Empleado [id: 1..2]{
    int idProx;
    while (true){
        receive avisoLlegada(idProx);
        send siguiente[idProx]("Sos el siguiente",id);
        //atiende al cliente
        send fin[id]("finalizado");
    }
}
process CLiente [id: 1..N]{
    string rta; string fin;
    send avisoLlegada(id);
    receive siguiente[id](rta,idPuesto);
    receive fin[idPuesto](fin);
}

```
c.Ídem b) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas
 durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?
```Cpp
chan siguiente[N](String, int); chan avisoLlegada(string);chan fin[1..2](string);

process Empleado [id: 1..2]{
    int idProx;
    while (true){
        if(avisoLlegada.empty()){
            delay(15min);
        }else{
            receive avisoLlegada(idProx);
            send siguiente[idProx]("Sos el siguiente",id);
            //atiende al cliente
            send fin[id]("finalizado");
        }
    }
}
process CLiente [id: 1..N]{
    string rta; string fin;
    send avisoLlegada(id);
    receive siguiente[id](rta,idPuesto);
    receive fin[idPuesto](fin);
}

```

## Ejercicio 2

Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizar la concurrencia (para ello usamos el coordinador quien se encarga de asignar la caja a cada cliente).
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

Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.
Nota: maximizar la concurrencia.

```cpp
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
        if (pedido.empty()){
            delay(1..3 min);
        else{
            receive pedido(idCliente,pedidoSolicitado);
            send avisoCocinero(idCliente,pedidoSolicitado);
        }
    }
}
process cliente[id: 1..C]{
    String pedido;pedidoCocinado;
    send pedido(id, pedido);
    receive finPedido[id](pedidoCocinado);
}
```

## Ejercicio 4

Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un ticket factura por la operación.
a)
Implemente una solución para el problema descrito.
```cpp
```
b)
Modifique la solución implementada para que el empleado dé prioridad a los que terminaron de usar la cabina sobre los que están esperando para usarla.
Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.
```cpp
```
## Ejercicio 5

Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada.
a)Implemente una solución para el problema descrito.
```cpp
```
b)Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos.
```cpp
```
c)Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución.
```cpp
```
d)Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.
```cpp
```
e)Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo.
Nota: ni los administrativos ni el director deben esperar a que se imprima el documento.
```cpp
```