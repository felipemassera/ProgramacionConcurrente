# Pasaje de mensajes 
1) En  una  oficina  existen  100  empleados  que  envían  documentos  para  imprimir  en  5  impresoras 
compartidas. Los pedidos de impresión son procesados por orden de llegada y se asignan a la primera 
impresora que se encuentre libre: 
a) Implemente un programa que permita resolver el problema anterior usando PMA. 
```cpp
chan solicitud[100](Documento);
chan impresoraLibre[5](int);

process empleado[id 1..100]{
    while(true){
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
    do
        admin!solicitud(new Documento());
    od
}

process Impresora[id: 1..5]{
    Documento documento;
    while(true){
        send ipresoraLibre(id);
        receive impresora(documento);
        imprimir(documento);
    }
}

process admin{                    
    int idImp                               //uso de admin PMS para poder realizar la administracion del orden de peticiones. 
    while(true){
        empleado[*]?solicitud(documento);
        impresora?impresoraLibre(idImp);
        impresora!imprimir[idImp](documento);
    }
}

```
1) Resolver el siguiente problema con PMS. En la estación de trenes hay una terminal de SUBE que 
debe ser usada por P personas de acuerdo con el orden de llegada. Cuando la persona accede a la 
terminal,  la  usa  y  luego  se  retira  para  dejar  al  siguiente.  Nota:  cada  Persona  usa  sólo  una  vez  la 
terminal.  
```cpp
```
1) Resolver el siguiente problema  con PMA. En  un  negocio de cobros digitales hay P personas que 
deben  pasar  por  la  única  caja  de  cobros  para  realizar  el  pago  de  sus  boletas.  Las  personas  son 
atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos 
de 5 boletas de los que pagan más. Adicionalmente, las personas embarazadas tienen prioridad sobre 
los dos casos anteriores. Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les 
devuelve el vuelto y los recibos de pago. 
```cpp
```
# ADA 
1) Resolver el siguiente problema. La página web del Banco Central exhibe las diferentes cotizaciones 
del dólar oficial de 20 bancos del país, tanto para la compra como para la venta. Existe una tarea 
programada que se ocupa de actualizar la página en forma periódica y para ello consulta la cotización 
de cada uno de los 20 bancos. Cada banco dispone de una API, cuya única función es procesar las 
solicitudes de aplicaciones externas. La tarea programada consulta de a una API por vez, esperando 
a lo sumo 5 segundos por su respuesta. Si pasado ese tiempo no respondió, entonces se mostrará 
vacía la información de ese banco. 
```java

```
2) Resolver el siguiente problema. En un negocio de cobros digitales hay P personas que deben pasar 
por la única caja de cobros para realizar el pago de sus boletas. Las personas son atendidas de acuerdo 
con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de 5 boletas de los que 
pagan más. Adicionalmente, las personas ancianas tienen prioridad sobre los dos casos anteriores. 
Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los 
recibos de pago.  
```java

```
3) Resolver el siguiente problema. La oficina central de una empresa de venta de indumentaria debe 
calcular cuántas veces fue vendido cada uno de los artículos de su catálogo. La empresa se compone 
de 100 sucursales y cada una de ellas maneja su propia base de datos de ventas. La oficina central 
cuenta con una herramienta que funciona de la siguiente manera: ante la consulta realizada para un 
artículo determinado, la herramienta envía el identificador del artículo a las sucursales, para que cada 
una  calcule  cuántas  veces  fue  vendido  en  ella.  Al  final  del  procesamiento,  la  herramienta  debe 
conocer cuántas veces fue vendido en total, considerando todas las sucursales. Cuando ha terminado 
de procesar un artículo  comienza con  el siguiente (suponga que la herramienta tiene una función 
generarArtículo()  que  retorna el  siguiente  ID  a  consultar).  Nota:  maximizar  la  concurrencia.  Existe 
una  función  ObtenerVentas(ID)  que  retorna  la  cantidad  de  veces  que  fue  vendido  el  artículo  con 
identificador ID en la base de la sucursal que la llama. 
```java

```
 