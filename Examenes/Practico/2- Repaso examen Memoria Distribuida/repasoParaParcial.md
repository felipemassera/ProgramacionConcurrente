# Pasaje de mensajes 
1) En  una  oficina  existen  100  empleados  que  envían  documentos  para  imprimir  en  5  impresoras 
compartidas. Los pedidos de impresión son procesados por orden de llegada y se asignan a la primera 
impresora que se encuentre libre: 
a) Implemente un programa que permita resolver el problema anterior usando PMA. 
```cpp
chan solicitud[100](Documento);
chan impresoraLibre[5](int);
chan impresora(Documento)

process empleado[id 1..100]{
    while(true){
        //TRABAJO
        send solicitud(new Documento(););
    }
}

process Impresora[id: 1..5]{
    Documento documento;
    while(true){
        send ipresoraLibre(id);
        receive impresora(documento);
        imprimir(documento);
    }
}

process admin{                                  //uso de admin para poder realizar la administracion del orden de peticiones. 
    while(true){
        receive solicitud(documento);
        receive impresoraLibre(idImp);
        send impresora[idImp](documento);
    }
}
```
b) Resuelva el mismo problema anterior pero ahora usando PMS. 
```cpp
process empleado[id 1..100]{
    while(true){
        //TRABAJO
        admin!solicitud(new Documento());
    }
}

process Impresora[id: 1..5]{
    Documento documento;
    while(true){
        Admin!ipresoraLibre(id);
        Admin?impresora(documento);
        imprimir(documento);
    }
}

process admin{                    
    queue cola; 
    int idImp;                               //uso de admin PMS para poder realizar la administracion del orden de peticiones usando la cola y preguntando si esta esta vacia luego!
    do
        () empleado[*]?solicitud(documento); -> cola.push(documento);
        [] !cola.empty(); impresora[*]?impresoraLibre(idImp); -> impresora!imprimir[idImp](cola.pop());
    od
}
```
2) Resolver el siguiente problema con PMS. En la estación de trenes hay una terminal de SUBE que 
debe ser usada por P personas de acuerdo con el orden de llegada. Cuando la persona accede a la 
terminal,  la  usa  y  luego  se  retira  para  dejar  al  siguiente.  Nota:  cada  Persona  usa  sólo  una  vez  la 
terminal.  
```cpp
process persona[id:1..P]{
    admin!solicitarUso(id);                                         
    admin?usar();
    //USO LA TERMINAL.
    admin!terminalLibre();
}

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

3) Resolver el siguiente problema  con PMA. En  un  negocio de cobros digitales hay P personas que 
deben  pasar  por  la  única  caja  de  cobros  para  realizar  el  pago  de  sus  boletas.  Las  personas  son 
atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos 
de 5 boletas de los que pagan más. Adicionalmente, las personas embarazadas tienen prioridad sobre 
los dos casos anteriores. Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les 
devuelve el vuelto y los recibos de pago. 
```cpp
    chan cajas[3](int);             //0= embarazadas; 1=<5 boletas ; 2= >=5 boletas
    chan solicitarPago();
    chan fin[P](text,int);

    process Persona[id:1..P]{
        boolean prioridad= X;
        int cantBoletas= X;
        int monto=X, vuelto=X;
        Text comprobante;

        if(prioridad){
            send caja[0](id,monto,boleta);
        }elif(cantBoletas<5){
            send caja[1](id,monto,boleta);
        }else{
            send caja[2](id,monto,boleta);
        }
        send solicitarPago();
        receive fin(comprobante,vuelto);
    }

    process cajero{
        int idP, monto, vuelto;
        text boleta, comprobante;
        while(true){
            receive solicitarPago();
            if(!caja[0].empty()){
                receive caja[0](idP,monto,boleta);    
            }elif (!caja[1].empty()){
                receive caja[1](idP,monto,boleta);
            }else{
                receive caja[2](idP,monto,boleta);
            }
            comprobante, vuelto = cobrar(boleta, monto);
            send fin[idP](comprobante,vuelto);
        }
    }
```
# ADA 
4) Resolver el siguiente problema. La página web del *Banco Central* exhibe las diferentes cotizaciones 
del dólar oficial de *20 bancos* del país, tanto para la compra como para la venta. Existe una tarea 
programada que se ocupa de actualizar la página en forma periódica y para ello consulta la cotización 
de cada uno de los 20 bancos. Cada banco dispone de una API, cuya única función es procesar las 
solicitudes de aplicaciones externas. La tarea programada consulta de a una API por vez, esperando 
a lo sumo 5 segundos por su respuesta. Si pasado ese tiempo no respondió, entonces se mostrará 
vacía la información de ese banco.
```SH
Procedure Banco is

task Central;

task type Banco is
    ENTRY consultaDolar(compra: OUT integer; venta: OUT integer);
end Banco;

arrBanco: array [1..20] of  Banco;

TASK BODY CENTRAL IS
    bancos : arrBanco =([20] Banco);                  //tiene un array de bancos inicializados;
    valorCompra: array [1..20] of Integer;
    valorVenta: array [1..20] of Integer;
Begin
    valorCompra=([20]0);
    valorVenta=([20]0);
    LOOP
        for i in 1..20 LOOP
            Select
                Banco(i).consultaDolar(compra,venta) do
                    valorCompra[i]=compra;
                    valorVenta[i]= venta;
                end;
            OR  delay 5.0;
        end LOOP;
        delay(X);                                   //se espera X tiempo simulando la periodicidad con la que se repite el proceso.
    end LOOP;
END BODY;

TASK BODY BANCO IS
Begin
    LOOP
        comp, vent = ActualizoValoresCompraVenta();
        ACCEPT consultaDolar(compra: OUT integer; venta: OUT integer) do
            compra=comp;
            venta = vent;
        end ACCEPT;
    end LOOP;
END BODY;
Begin
    null;
end Banco;
```

5) Resolver el siguiente problema. En un negocio de cobros digitales hay *P personas* que deben pasar 
por la única caja de cobros para realizar el pago de sus boletas. Las personas son atendidas de acuerdo 
con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de 5 boletas de los que 
pagan más. Adicionalmente, las personas *ancianas* tienen prioridad sobre los dos casos anteriores. 
Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los 
recibos de pago.  
```sh
procedure Negocio

Task cajero is
    ENTRY pagoAnciano(Boleta:In Text; Monto: In Real; Comprobante: Out Text; cambio: OUT real);
    ENTRY pagoConPrioridad(Boleta:In Text; Monto: In Real; Comprobante: Out Text; cambio: OUT real);
    ENTRY pagoSinPrioridad(Boleta:In Text; Monto: In Real; Comprobante: Out Text; cambio: OUT real);
end cajero;

TASK TYPE Persona is
    ENTRY identificar(i:in integer);
end Persona;
arrPersona: array (1..P) of Persona;

TASK BODY Cajero is
begin
    LOOP
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
    end LOOP;    
end BODY;

TASK BODY Persona is
    prioridad: Boolean;
    boletas, comprobante: Text
    monto, vuelto: Real;
begin
    monto =X; pioridad= X; 

    if(prioridad)then
        cajero.pagoAnciano(boletas,monto,comprobante,vuelto);
    else{
        if(boletas<5) then
            cajero.pagoConPrioridad(boletas,monto,comprobante,vuelto);
        else
            cajero.pagoSinPrioridad(boletas,monto,comprobante,vuelto);
        END IF;
    END IF;
end BODY;


Begin
    null;
END Negocio;
```
6) Resolver el siguiente problema. La oficina *central* de una empresa de venta de indumentaria debe 
calcular cuántas veces fue vendido cada uno de los artículos de su catálogo. La empresa se compone 
de *100 sucursales* y cada una de ellas maneja su propia base de datos de ventas. La oficina central 
cuenta con una herramienta que funciona de la siguiente manera:
- Ante la consulta realizada para un artículo determinado, la herramienta envía el identificador del artículo a las sucursales, para que cada 
una  calcule  cuántas  veces  fue  vendido  en  ella.
- Al  final  del  procesamiento,  la  herramienta  debe conocer cuántas veces fue vendido en total, considerando todas las sucursales. 
- Cuando ha terminado de procesar un artículo  comienza con  el siguiente (suponga que la herramienta tiene una función 
generarArtículo()  que  retorna el  siguiente  ID  a  consultar).

  Nota:  maximizar  la  concurrencia.  Existe una  función  ObtenerVentas(ID)  que  retorna  la  cantidad  de  veces  que  fue  vendido  el  artículo  con 
identificador ID en la base de la sucursal que la llama. 

```sh
procedure Oficina is

task central is                                                 //LA CENTRAL ES LA ENCARGADA DE REPARTIR LOS IDS DE LOS PRODUCTOS, PARA ELLO LA ESTRATEGIA DE ESPERAR QUE UN SERVIDOR
    ENTRY servidorLibre(idp: OUT integer);                          SE DECLARE LIBRE ES UNA BUENA OPCION, LUEGO ACUMULAR LOS RESULTADOS Y POR ULTIMO REINICIAR EL CICLO PARA QUE LOS SERVIDORES PUEDAN 
    ENTRY resultadoBusqueda(cant: IN Integer);                      REINICIAR LAS BUSQUEDAS CON OTRO ID
    ENTRY finVuelta();

TASK TYPE sucursal; 

sucursales: Array (1..100) of sucursal;

TASK BODY Central is
    totalProducto, idP: Integer;
    
Begin
    LOOP
        totalProducto = 0;
        idP = generarArtículo();
        for i in 1..200 LOOP                            // tener en cuenta que espera 100 res y hace 100 envios de id.
            Select
                ACCEPT servidorLibre(idProd: OUT integer)do
                    idProd = idP;
                END ACCEPT;
            OR
                ACCEPT resultadoBusqueda(cant: IN Integer) do
                    totalProducto = totalProducto + cant;
                END ACCEPT;
        END LOOP;
        for i in 1..100 LOOP
            ACCEPT finVuelta();
        END LOOP;
        Print ("Producto ID:", idP , " ventas totales: ", totalProducto);       //imprimo en pantalla.
    END LOOP;
END BODY;

TASK BODY Sucursal is
    BD: Base Datos;
Begin
    LOOP
        central.servidorLibre(idP);
        total=ObtenerVentas(idp);
        central.resultadoBusqueda(total);
        central.finVuelta;
    END LOOP;
END BODY;

Begin
    null;
end Oficina;
```
 