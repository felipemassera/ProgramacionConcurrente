# ADA

## Ejercicio 1

Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. 
El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 
unidades  y  cada  camión  3  unidades.  Suponga  que  hay  una  cantidad  innumerable  de 
vehículos  (A  autos,  B  camionetas  y  C  camiones).  Analice  el  problema  y  defina  qué  tareas, 
recursos y sincronizaciones serán necesarios/convenientes para resolverlo. 
a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad. 
```JAva
Procedure Ejercicio1 is

task Admin is
    Entry salir(Peso:IN Integer);
    Entry quieroPasarAuto;
    Entry quieroPasarCamineta;
    Entry quieroPasarCamion;
End Admin;

Task type Auto;
Task type Camioneta;
Task type Camion;

arrClientes: array(1..N) of Auto;
arrClientes: array(1..N) of Camioneta;
arrClientes: array(1..N) of Camion;

TASK BODY Admin is
pesoActual: integer;
Begin
    pesoActual = 0;
    LOOP
        SELECT
            WHEN (pesoActual<5) =>  ACCEPT quieroPasarAuto;                 //EN ESTE CASO NO HAGO LA SUMATORIA DENTRO DE UN DO, ya que no quiero demorar al proceso mientras actualizo mi var local.
            pesoActual = pesoActual + 1;
            
            OR WHEN (pesoActual <4) =>  ACCEPT quieroPasarCamioneta do      //hacer la sumatoria dentro del DO detiene el camioneta hasta que haga END QUIEROPASARCAMIONETA;
                pesoActual = pesoActual + 2;
            end quieroPasarCamioneta;
            
            OR WHEN (pesoActual <3) =>  ACCEPT quieroPasarCamion do
                pesoActual = pesoActual + 3;
            end quieroPasarCamion;
            OR ACCEPT salir(peso: IN Integer) do
                pesoAct = pesoAct - peso;
            end Salir;
        end Select;
    End LOOP;
END BODY Admin;

TASK BODY Auto is
Begin
    Admin.quieroPasarAuto;
    Admin.salir(1);
END BODY Auto;

TASK BODY Camioneta is
Begin
    Admin.quieroPasarCamineta;
    Admin.salir(2);
END BODY Camioneta;

TASK BODY Camion is
Begin
    Admin.quieroPasarCamion;
    Admin.salir(3);
END BODY Camion;

Begin
    null
END Ejercicio1;

```

b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos.
```JAva
Procedure Ejercicio1 is

task Admin is
    Entry salir(Peso:IN Integer);
    Entry quieroPasarAuto;
    Entry quieroPasarCamineta;
    Entry quieroPasarCamion;
End Admin;

Task type Auto;
Task type Camioneta;
Task type Camion;

arrAuto: array(1..A) of Auto;
arrCamioneta: array(1..B) of Camioneta;
arrCamion: array(1..C) of Camion;

TASK BODY Admin is
pesoActual: integer;
Begin
    pesoActual = 0;
    LOOP
        SELECT
            WHEN ((quieroPasarCamion`COUNT==0 or pesoActual>=3) and (pesoActual<5)) =>  ACCEPT quieroPasarAuto;  //en esta caso estoy priorizando el paso de camion, y si puedo no demoro el paso de autos.
            pesoActual = pesoActual + 1;
            
            OR WHEN ((quieroPasarCamion`COUNT==0 or pesoActual=3)and(pesoActual<4)) =>  ACCEPT quieroPasarCamioneta do      //hacer la sumatoria dentro del DO detiene el camioneta hasta que haga END QUIEROPASARCAMIONETA;
                pesoActual = pesoActual + 2;
            end quieroPasarCamioneta;
            
            OR WHEN (pesoActual <3) =>  ACCEPT quieroPasarCamion do
                pesoActual = pesoActual + 3;
            end quieroPasarCamion;
            
            OR ACCEPT salir(peso: IN Integer) do
                pesoAct = pesoAct - peso;
            end Salir;
        end Select;
    End LOOP;
end Admin;

TASK BODY Auto is
Begin
    Admin.quieroPasarAuto;
    Admin.salir(1);
end Auto;

TASK BODY Camioneta is
Begin
    Admin.quieroPasarCamineta;
    Admin.salir(2);
end Camioneta;

TASK BODY Camion is
Begin
    Admin.quieroPasarCamion;
    Admin.salir(3);
end Camion;

Begin
    null
END Ejercicio1;

```
## Ejercicio 2 
Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar 
un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de 
acuerdo con el orden de llegada.  
a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido 
atendidos. 
```java
PROCEDURE ejercicio2 IS
TASK Empleado is
    Entry solicitarAtencion(pago: IN Integer;comprobante: OUT Integer);
end Empleado;

TASK TYPE Cliente;

arrCliente: array (1..C) of Cliente;

TASK BODY Empleado is
begin
    LOOP
        ACCEPT solicitarAtencion(pago: IN Integer;comprobante: OUT Integer) do
            comprobante= Cobrar(pago);
        end solicitarAtencion; 
    END LOOP;
END BODY Empleado;

TASK BODY Cliente is
    pago: integer ; comprobante: integer;
begin
    Empleado.solicitarAtencion(pago,comprobante);
END BODY Cliente;

BEGIN
  null;
END;
```

b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para 
realizar el pago. 
```java
PROCEDURE ejercicio2 IS
TASK Empleado is
    Entry solicitarAtencion(pago: IN Integer;comprobante: OUT Integer);
end Empleado;

TASK TYPE Cliente;

arrCliente: array (1..C) of Cliente;

TASK BODY Empleado is
begin
    LOOP
        ACCEPT solicitarAtencion(pago: IN Integer;comprobante: OUT Integer) do
            comprobante= Cobrar(pago);
        end solicitarAtencion; 
    END LOOP;
end Empleado;

TASK BODY Cliente is
    pago: integer ; comprobante: integer;
begin
    SELECT 
        Empleado.solicitarAtencion(pago,comprobante);
    OR DELAY 600.00
        null;
    end SELECT;
end Cliente;

BEGIN
  null;
END;
```
c) Implemente una solución donde los clientes se retiran si no son atendidos 
inmediatamente. 
```java
PROCEDURE ejercicio2 IS
TASK Empleado is
    Entry solicitarAtencion(pago: IN Integer;comprobante: OUT Integer);
end Empleado;

TASK TYPE Cliente;

arrCliente: array (1..C) of Cliente;

TASK BODY Empleado is
begin
    LOOP
        ACCEPT solicitarAtencion(pago: IN Integer;comprobante: OUT Integer) do
            comprobante= Cobrar(pago);
        end solicitarAtencion; 
    END LOOP;
end Empleado;

TASK BODY Cliente is
    pago: integer ; comprobante: integer;
begin
    SELECT 
        Empleado.solicitarAtencion(pago,comprobante);
    ELSE
        null;
    end SELECT;
end Cliente;

BEGIN
  null;
END;
```
d)  Implemente  una  solución  donde  los  clientes  esperan  a  lo  sumo  10  minutos  para  ser 
atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más 
y se retiran si no son atendidos inmediatamente.
```sh
py
rust
cpp
java

PROCEDURE ejercicio2 IS
TASK Empleado is
    Entry solicitarAtencion(pago: IN Integer;comprobante: OUT Integer);
END BODY Empleado;

TASK TYPE Cliente;

arrCliente: array (1..C) of Cliente;

TASK BODY Empleado is
begin
    LOOP
        ACCEPT solicitarAtencion(pago: IN Integer;comprobante: OUT Integer) do
            comprobante= Cobrar(pago);
        end solicitarAtencion; 
    END LOOP;
END BODY Empleado;

TASK BODY Cliente is
    pago: integer ; comprobante: integer;
begin
    SELECT 
        Empleado.solicitarAtencion(pago,comprobante);
    OR DELAY 600.00
        SELECT 
            Empleado.solicitarAtencion(pago,comprobante);
        ELSE
            null;
        END SELECT;
    END SELECT;
END BODY Cliente;

BEGIN
  null;
END;
```

## Ejercicio 3
Se  dispone  de  un  sistema  compuesto  por  1  central  y  2  procesos  periféricos,  que  se comunican continuamente. Se requiere modelar su funcionamiento considerando las 
siguientes condiciones: 
- La  central  siempre  comienza  su  ejecución  tomando  una  señal  del  proceso  1;  luego 
toma  aleatoriamente  señales  de  cualquiera  de  los  dos  indefinidamente.  Al  recibir  una 
señal de proceso 2, recibe señales del mismo proceso durante 3 minutos. 
- Los  procesos  periféricos  envían  señales  continuamente  a  la  central.  La  señal  del 
proceso  1  será  considerada  vieja  (se  deshecha)  si  en  2  minutos  no  fue  recibida.  Si  la 
señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y 
vuelve a mandarla (no se deshecha).
# CHEQUEAR CON LOS CHICOS
```sh
PROCEDURE Ejercicio3 IS

TASK Central IS
    ENTRY señalPeriferico1(dato:IN DATO);
    ENTRY señalPeriferico2(dato:IN DATO);
    ENTRY finBucle;
end central;
TASK Periferico1;
TASK Periferico2;

TASK Timer is
    ENTRY startTimer;
TASK BODY Timer is
Begin
    LOOP
        ACCEPT startTimer;          //le avisan que tiene que empezar el conteo de 3 min;
        delay 180.0;
        Central.finBucle;
    end LOOP;
END BODY Timer;

TASK BODY Central is
dato: DATO; seguir:boolean;
Begin
    ACCEPT señalPeriferico1(dato:IN DATO);
    LOOP
        Select
            ACCEPT señalPeriferico1(dato:IN DATO);
        ELSE
            ACCEPT señalPeriferico2(dato:IN DATO);
            seguir = true;
            Timer.startTimer;
            while (seguir) LOOP
                Select
                    when(finBucle`count= 0 ) => ACCEPT señalPeriferico2(dato:IN DATO);  //como hacer para recibir siempre mensajes durante 180.0 seg y luego devolver el control PREGUNTAR A LS PIBES??
                or  
                    ACCEPT finBucle;
                    seguir = false;
            end LOOP;
        end Select;
    end LOOP;
END BODY central;

TASK BODY periferico1 is
dato: DATO;
Begin
    LOOP
        dato= new DATO;
        Select
            Central.señalPeriferico1(dato);
        or DELAY 120.0;
            NULL;
    end LOOP;
END BODY Periferico1;

TASK BODY periferico2 is
seguir:boolean; dato: DATO;
Begin
    LOOP
        dato= new DATO();
        seguir = true;
        while (seguir) LOOP
            Select
                central.señalPeriferico2(dato);
                seguir=false;
            else
                delay 60.0;
            end Select;
        end LOOP;
    end LOOP;

END BODY Periferico2;

BEGIN
    null;
END;
```

## Ejercicio 4
 En  una  clínica  existe  un  médico  de  guardia  que  recibe  continuamente  peticiones  de 
atención de las E  enfermeras que trabajan en su piso y de las  P  personas que llegan a la 
clínica ser atendidos.  
*PACIENTE:
Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo 
haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del 
médico. Si no es atendida tres veces, se enoja y se retira de la clínica. 
ENFERMERA:
Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente 
le  hace  una  nota  y  se  la  deja  en  el  consultorio  para  que  esta  resuelva  su  pedido  en  el 
momento  que  pueda  (el  pedido  puede  ser  que  el  médico  le  firme  algún  papel).  Cuando  la 
petición  ha  sido  recibida  por  el  médico  o  la  nota  ha  sido  dejada  en  el  escritorio,  continúa 
trabajando y haciendo más peticiones. 
MEDICO:
El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. 
Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando 
está libre aprovecha a procesar las notas dejadas por las enfermeras. 

```CPP
PROCEDURE Ejercicio4 IS
TASK Medico is
    ENTRY PeticionEnfermeras();
    ENTRY PeticionPaciente();
    ENTRY notaEnEscritorio(Nota:IN TEXT);
end medico;

TASK TYPE Enfermera;
TASK TYPE Paciente;
Enfermeras= array(1..E) of Enfermera;
Pacientes= array(1..P) of Paciente;

TASK Escritorio is
    ENTRY notaEnfermera(Nota:IN TEXT);
end Escritorio;

TASK BODY Escritorio is
mensajes: queue TEXT;
Begin
    Loop
        Select
            ACCEPT notaEnfermera(nota:IN TEXT)do
            insertar(mensajes,nota);
            end notaEnfermera;
        or 
            ACCEPT notaEnEscritorio(Nota: OUT TEXT) do
               if (not mensajes.empty()) then
                nota= mensajes.pop();
                else
                    nota="";
            end notaEnEscritorio;
        end select;
    end Loop;
END BODY Escritorio;


TASK BODY Enfermera is
Nota: Text;
begin
    Loop
        select
            Medico.PeticionEnfermeras;
        else
            nota = New Text();                  
            Escritorio.NotaEnfermera(nota);
        end select;
    end loop;
END BODY Enfermera;

TASK BODY Paciente is
cant:Integer
BEGIN
    cant=0;
    while cant <3 Loop
        SELECT 
            Medico.PeticionPaciente;
            cant=3;
        OR DELAY 300.0;
            cant= cant + 1;
            if (cant <3 ) then
                DELAY 600.0;
            end if;
        END select;
    END LOOP;
END BODY Paciente;

TASK BODY Medico is
Begin
    LOOP
        Select
            when(PeticionPaciente`count=0) => Accept PeticionEnfermeras() do
            //procesar peticion;
            end PeticionEnfermeras;
        or  
            Accept PeticionPaciente() do
            //proceso;
            end PeticionPaciente;
        else
            Escritorio.notaEnEscritorio(nota);
            if (not nota.equals("")) then
                procesar(nota);
            end if;
        end select;
    END LOOP;
END BODY Medico;

BEGIN
    null;
END Ejercicio4;
```
## Ejercicio 4.2
 En  un  sistema  para  acreditar  carreras  universitarias,  hay  UN  Servidor  que  atiende  pedidos 
de  U  Usuarios  de  a  uno  a  la  vez  y  de  acuerdo  con  el  orden  en  que  se  hacen  los  pedidos. 
Cada  usuario  trabaja  en  el  documento  a  presentar,  y  luego  lo  envía  al  servidor;  espera  la 
respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error, 
vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde 
que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 
2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a 
intentarlo (usando el mismo documento).

```sh
Procedure Ejercicio4.2 is
TASK Servidor is
    ENTRY peticionUsuario(doc: IN TEXT , OK : OUT Boolean);
end servidor;
TASK BODY Servidor is
ok: boolean
Begin
    LOOP
        SELECT 
            ACCEPT peticionUsuario(doc: IN TEXT , OK : OUT Boolean) do
                OK = ProcesarDcoumento (doc);
            end peticionUsuario;
        end Select;
    END LOOP;
END BODY servidor;

TASK TYPE Usuario;
TASK BODY Usuario is
Documento : TEXT; OK:  boolean;
Begin
    OK=false;
    Documento = new Documento();
    while (not OK) LOOP
        SELECT
            Servidor.peticionUsuario(Documento, OK);
            If (not OK) then
                Documento = ArreglarDocumento(Documento);
            end IF;
        OR  DELAY 120.0;
            DELAY 60.0;
        end Select;
    END LOOP;
END BODY Usuario;

Begin
    NULL;
end Ejercicio4.2;
```

## Ejercicio 5
 En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada 
una  conoce  previamente  a  que  equipo  pertenece).  Cuando  las  personas  van  llegando 
esperan  con  los  de  su  equipo  hasta  que  el  mismo  esté  completo  (hayan  llegado  los  4 
integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que 
cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser 
de  1,  2  o  5  pesos)  y  se  suman  los  montos  de  las  60  monedas  conseguidas  en  el  grupo.  Al 
finalizar  cada  persona  debe  conocer  el  grupo  que  más  dinero  junto.  Nota:  maximizar  la 
concurrencia.  Suponga  que  para  simular  la  búsqueda  de  una  moneda  por  parte  de  una 
persona existe una función Moneda() que retorna el valor de la moneda encontrada.


 
## Ejercicio 6
 Se  debe  calcular  el  valor  promedio  de  un  vector  de  1  millón  de  números  enteros  que  se 
encuentra  distribuido  entre  10  procesos  Worker  (es  decir,  cada Worker  tiene  un  vector  de 
100  mil  números).  Para  ello,  existe  un  Coordinador  que  determina  el  momento  en  que  se 
debe  realizar  el  cálculo  de  este  promedio  y  que,  además,  se  queda  con  el  resultado.  Nota: 
maximizar la concurrencia; este cálculo se hace una sola vez. 
 
## Ejercicio 7
 Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores 
para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia; 
a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente 
manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores 
para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más 
se  asemeja  a  TEST  en  su  BD;  al  final  del  procesamiento,  el  especialista  debe  conocer  el 
código  de  la  huella  con  mayor  valor  de  similitud  entre  las  devueltas  por  los  8  servidores. 
Cuando  ha  terminado  de  procesar  una  huella  comienza  nuevamente  todo  el  ciclo.  Nota: 
suponga  que  existe  una  función  Buscar(test,  código,  valor)  que  utiliza  cada  Servidor  donde 
recibe  como  parámetro  de  entrada  la  huella  test,  y  devuelve  como  parámetros  de  salida  el 
código  y  el  valor  de  similitud  de  la  huella  más  parecida  a  test  en  la  BD  correspondiente. 
Maximizar la concurrencia y no generar demora innecesaria.

## Ejercicio 8
 Una  empresa  de  limpieza  se  encarga  de  recolectar  residuos  en  una  ciudad  por  medio  de  3 
camiones.  Hay  P  personas  que  hacen  reclamos  continuamente  hasta  que  uno  de  los 
camiones pase por su casa. Cada persona hace un reclamo y espera a lo sumo 15 minutos a 
que  llegue  un  camión;  si  no  pasa,  vuelve  a  hacer  el  reclamo  y  a  esperar  a  lo  sumo  15 
minutos a que llegue un camión; y así sucesivamente hasta que el camión llegue y recolecte 
los  residuos.  Sólo  cuando  un  camión  llega,  es  cuando  deja  de  hacer  reclamos  y  se  retira. 
Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos 
ha hecho sin ser atendido. Nota: maximizar la concurrencia.