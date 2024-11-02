# PMA 
 Usar las propias colas de mensajes , para poder usarlas en las guardas del intermedio.
 Muchos usuarios y muchos admins, usar siempre un intermedio
 Muchos usuarios y un solo admin, no usar intermedio
 En el caso de que un process se tenga que quedar esperando, primero manda sus datos(id) y luego se queda esperando a ser atendido
 Manejar bien las prioridades solicitadas
 Maximizar concurrencia=QUE TODOS TRABAJEN A LA PAR(CASO TIPICO DE INTERMEDIO)

TENER EN CUENTA DECLARAR LOS CANALES "CHAN ..."
### EJEMPLO DE COORDINADOR PARA EL USO  de V vendedores y P personas: si no hay clientes el vendedor espera.
### EL USO DEL COORDINADOR EN ESTE CAOS ES POR QUE NO SE PUEDE CONSULTAR POOR COLAS.EMPTY() SIENDO MAS DE UN VENDEDOR.
```CPP
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

# PMS
 Usar bien el concepto de bloqueo , conjuntamente con las guardas.
 Saber lo tipos de estado de las guardas al tener la condición boolean en true o false, y si llego o no llego el mensaje receptor que se va usar.
 ## Muchos usuarios y muchos admins, usar un intermedio
 ## Muchos usuarios y un admin, no usar un intermedio.
 Usar bien las prioridades usando colas, conjuntamente con los mensajes solicitados
 ## Maximizar concurrencia=USAR INTERMEDIOS Y AVECES NO
 
 ## RECORDAR QUE PARA CORTAR LA EJECUCION DE UN DO OD HAY QIE PONER CONDICIONES 
    DO
        () COND; ACEPTAR -> RTA
        () COND; ACEPTAR -> RTA
    OD
 ## USAR UN ADMIN QUE PERMITA MAXIMIZAR CONCURRENCIA, USANDO CONDICIONALES Y COLAS PARA MANEJAR EL FLUJO.

```cpp
process admin{                    
    queue cola; 
    int idImp;                               //uso de admin PMS para poder realizar la administracion del orden de peticiones usando la cola y preguntando si esta esta vacia luego!
    do
        () empleado[*]?solicitud(documento); -> cola.push(documento);
        [] !cola.empty(); impresora[*]?impresoraLibre(idImp); -> impresora!imprimir[idImp](cola.pop());
    od
}
```
## Cuando hay un conjunto de personas queriendo usar un recurso, este esta manejado por una persona, pero necesita atenderlos en orden, entonces se usa un COORD que maneje la cola, y el empleado acvise cuando puede atender a una persona nueva.
process empleado{                           \|\ process guia{                       
    for(i=1;i<=20;i++){                     \|\     int cant=0; Cola cola;
        coord!empleadoLibre();              \|\     DO
        coord!siguiente(id);                \|\         ()cant<20; cliente[x]?solicitarUso(id)-> 
        Cliente[id]!acceder();              \|\             cola.push(id); cant++;
        Cliente[id]?liberar();              \|\         ()!cola.empty(); guia?guiaLibre()->
    }                                       \|\             guia!siguiente(cola.pop());
}                                           \|\     OD
                                            \|\ }

## cuando hay un solo grupo de procesos que tienen que compartir un recurso, lo que hay que hacer es usar un ADMIN que administre el passing the condition;
```cpp
process admin{                                                           //manejo el acceso a la terminal con un passing the condition ya que usamos un unico recurso comartido;
    cola cola; integer idP; boolean libre= true;
    do{
        () persona[*]?solicitarUso(idP) ->
            if(libre){
                libre=false;
                persona[idP]!usar();
            }else{
                cola.push(idP);
            }
        () persona[*]?terminalLibre() ->
            if(cola.empty()){
                libre=true
            }else{
                persona[cola.pop()]!usar();
            }
    }od
}
```

# ADA
 Muchas tareas de distinto tipo, que quieran usar una tarea en especial solo la podran usar cuando una tarea la deje libre.
 Siempre hacer el cuerpo del accept, para tomar variables y poder usarlas en la tarea una vez finalizada el accept
 Los llamados entre tareas pueden traer deadlock
 Caso especial de tarea Timer
 Usar bien rendevouz
 Maximizar concurrencia=(rendevouz+evitar deadlock+identificadores para las tareas+bloquear a los demas, para usar la seccion critica con el accept)
 Para el caso de prioridades se usan los when y muchos or (POR QUE PUEDE HABER MUCHAS TAREAS) ,pero si solo fueran dos tareas, un ELSE funciona tambien

# identificar CLASES DESDE EL MAIN
```sh
VAR 
    x: integer;
Begin
    For x:1 to N
        personas[x].identificacion(x);
    endfor
end PROCEDURE;
```
Casos
## Select con accept- PRIORIDAD
 ```cpp  
   SELECT 
            ACCEPT pagoAnciano(Boleta:In Text; Monto: In Real; Comprobante: Out Text; cambio: OUT real) do
                    comprobante,cambio = CobrarBoletas(Boleta, Monto);
            END ACCEPT;
        OR  when(pagoAnciano`count==0); pagoConPrioridad(Boleta:In Text; Monto: In Real; Comprobante: Out Text; cambio: OUT real) do
                    comprobante,cambio = CobrarBoletas(Boleta, Monto);
            END ACCEPT;
        OR  when(pagoAnciano`count==0 and pagoConPrioridad`count==0) pagoSinPrioridad(Boleta:In Text; Monto: In Real; Comprobante: Out Text; cambio: OUT real) do
                    comprobante,cambio = CobrarBoletas(Boleta, Monto);
            END ACCEPT;
    END SELECT;
```
## Select con call 
```cpp
    Select 
        otraTarea.E1(); 
    + or delay 
        //aca adentro puedo agregar Select(es otro cuerpo separado e independiente) 
    + else 
        //lo mismo que el or delay
    End Select;
```