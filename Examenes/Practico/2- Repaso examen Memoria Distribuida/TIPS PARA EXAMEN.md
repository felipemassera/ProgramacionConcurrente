# PMA 
### Usar las propias colas de mensajes , para poder usarlas en las guardas del intermedio.
### Muchos usuarios y muchos admins, usar siempre un intermedio
### Muchos usuarios y un solo admin, no usar intermedio
### En el caso de que un process se tenga que quedar esperando, primero manda sus datos(id) y luego se queda esperando a ser atendido
### Manejar bien las prioridades solicitadas
### Maximizar concurrencia=QUE TODOS TRABAJEN A LA PAR(CASO TIPICO DE INTERMEDIO)

TENER EN CUENTA DECLARAR LOS CANALES "CHAN ..."

# PMS
### Usar bien el concepto de bloqueo , conjuntamente con las guardas.
### Saber lo tipos de estado de las guardas al tener la condición boolean en true o false, y si llego o no llego el mensaje receptor que se va usar.
### Muchos usuarios y muchos admins, usar un intermedio
### Muchos usuarios y un admin, no usar un intermedio.
### Usar bien las prioridades usando colas, conjuntamente con los mensajes solicitados
### Maximizar concurrencia=USAR INTERMEDIOS Y AVECES NO

//cuando hay un solo grupo de procesos que tienen que compartir un recurso, lo que hay que hacer es usar un ADMIN que administre el passing the condition;

# ADA
## tips parcial
### Muchas tareas de distinto tipo, que quieran usar una tarea en especial solo la podran usar cuando una tarea la deje libre.
### Siempre hacer el cuerpo del accept, para tomar variables y poder usarlas en la tarea una vez finalizada el accept
### Los llamados entre tareas pueden traer deadlock
### Caso especial de tarea Timer
### Usar bien rendevouz
### Maximizar concurrencia=(rendevouz+evitar deadlock+identificadores para las tareas+bloquear a los demas, para usar la seccion critica con el accept)
### Para el caso de prioridades se usan los when y muchos or (POR QUE PUEDE HABER MUCHAS TAREAS) ,pero si solo fueran dos tareas, un ELSE funciona tambien

VAR 
    x:int; 
Begin
    For x:1 to N
        personas[x].identificacion(x);
    endfor
end PROCEDURE;

Casos
## Select con accept
    Select 
        when accept E1()do
        end E1;
    Or 
        accept E2()do 
        end E2; 
    Or 
        accept E3()do
        end E3;
    End Select;

*Solo se puede usar 1 de los dos(NO LOS DOS)* 
    + or delay 
        //aca adentro puedo agregar Select(es otro cuerpo separado e independiente) 
    + else 
        //lo mismo que el or delay

## Select con call 
    Select 
        otraTarea.E1(); 
    + or delay 
        //aca adentro puedo agregar Select(es otro cuerpo separado e independiente) 
    + else 
        //lo mismo que el or delay
    End Select;