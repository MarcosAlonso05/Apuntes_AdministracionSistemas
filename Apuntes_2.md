# ADMINISTRACIÓN DE SISTEMAS

---

## ÍNDICE

- [SERVIDOR DE FICHEROS](#seccion1)
- [SERVIDOR RADIUS](#seccion2)

---

<a id="seccion1"></a>

## SERVIDOR DE FICHEROS (File server)

Se encarga de almacenar y administrar archivos para que otros usuarios puedan acceder a ellos a traves de una red.
Objetivo es almacenar datos no procesarlos.

El file server depende totalmente de sus discos duros, hay diferentes formas en las que se pueden configurar los discos duros de este.

### =RAID=

- **RAID 0:**
    No hay redundancia es decir no se garantiza la perpetuidad de los datos en caso de fallo de un componente.
    Divide la información en bloques y escribe una parte en cada disco simultaneamente.
    Ventaja: Es extremadamente rápido (lectura y escritura).
  - Ej:
        Si tengo dos discos de 1 TB con Raid 0 se combinara y se interpretaran como un solo disco de 2 TB.
        Problema, al no aber redundancia si falla un solo disco, pierdes toda la información del arreglo, ya que los archivos están partidos por la mitad entre los discos.
&#8203;

- **RAID 1:**
    Tengo redundancia pero a cambio pierdo la mitad del espacio.
    Raid 1 escribe exactamente la misma información en ambos discos al mismo tiempo.
    Ventaja: Seguridad de datos simple y lectura rápida (puede leer de ambos a la vez).
  - Ej:
        Si tengo dos discos de 1 TB solo tendre un 1 TB de almacenamiento pero la información se duplica en ambos. Por lo que si cae uno no perderas la información.
&#8203;

- **RAID 5:**
    Minimo es necesario tener tres discos, distribuye los datos entre los discos, pero uno quedara reservado para un codigo matematico llamado paridad.
    Se puede usar paridad par o impar. En cada sicos se guarda o un 1 o un 0 si estamos usando paridad par en caso de que usemos 3 discos (uno esta reservado), si hay un numero impar de unos se guarda un 1 en el disco de paridad si hay un numero par se guarda 0 (la impar es lo mismo pero al rebes).
    Esto nos permite que si uno de los discos se rompe podremos restaurar los datos ya que sabemos que bit va en esa posición (Si se rompe mas de un disco ya no funciona).
  - Ej:
        Si compramos tres discos de 1 TB tendremos 2 TB de espacio ya que uno va para la paridad. Visualización:

    | Disco 1 | Disco 2 | Disco (paridad) |
    |:-------:|:-------:|:-------:|
    |1|0|1|
    |1|1|0|
    |0|0|0|

En las empresas se suele usar una combinacion de los diferentes RAIDS para maximizar el espacio pero manteniendo la seguridad. Todo dependera de los requisitos de la empresa, el dinero dispuesto a gastar y la seguridad que se requiera.

![ej Raids](recursos/raids.png)

---

<a id="seccion2"></a>

## SERVIDOR RADIUS

Un Servidor RADIUS (Remote Authentication Dial-In User Service) es un protocolo y sistema encargado de gestionar el acceso a la red.
Puede servir como un paso previo de acceso a la red.
No es un sistema de control de usuarios, pero si puede validarlos.

### -IMPLEMENTACIÓN

Supongamos que tenemos una organización:

```
RED INTERNA ---- Firewall ---- INTERNET
     |               | 
 Servidores         DMZ
 Maquinas
```

-DMZ (Zona desmilitarizada): Es una zono intermediaria que se suele situar entre Internet y la red interna.
Sale del Firewall, uno de sus usos es poner en ella el frontend de los servidores para que todo el trafico pase por el firewall y cualquiera que acceda desde fuera solo pueda hablar con esa zona.
Así si el servidor que está aislado en la DMZ es atacado, el hacker se queda "encerrado" allí. El Firewall bloquea el tráfico desde la DMZ hacia la Red Interna protegiendola.
Tambien podemos tener en ella una red wifi de invitados. Ya que conectar un "invitado" a la red interna es peligroso, al crear una red wifi aislada en la DMZ gano que su trafico no pase por mi red interna ademas de estar protegido por el firewall.
Otra función es tener una red wifi de mi conpañia montada en una dmz del firewall dedicada a los telefonos de empleados (BYOD). Ya que los portatiles se pueden tener controlados pero cosas como los moviles de los empleados pueden ser mas peligrosos.

Entonces aqui es donde entra el RADIUS, que se encarga de validar hasta donde puede llegar un usuario. Bloquea y no permite el paso a zonas indevidas.
Aunque fisicamente es el mismo cable y la misma antena, pero lógicamente son dos mundos separados.

### -FUNCIONAMIENTO DEL RADIUS

El Servidor Radius es:
-Sistema de control a red, áctúa como punto de decisión para permitir o denegar el acceso a la red basándose en políticas predefinidas.
-Proxy (RADIUS Proxy), Capacidad de retransmitir las peticiones de autenticación a otros servidores RADIUS externos.
-Autentificador, Verifica las credenciales del usuario contra una base de datos externa o local.

**-Radius Externo:**

```
wifi
|
Radius  <--> Relacion de confianza <┐
|                                   |         
Firewall --- Red interna --- Directorio activo 
```
El Radius puede actuar de intermediario entre las maquinas que vengan del wifi y mi servidor de Direcctorio Activo, estos estan conectados entre si mediante una relación de confianza. Funcionamiento:
Supongamos que el Directorio activo (AD) tiene la IP .100 y el Servidor Radius tiene la IP .200, entonces al AD se le indica la IP del Radius y vicebersa ademas que cada uno indica una password.
Pero esto no es suficiente ya que alguien puedo suplantar la identidad de el servidor Radius para engañar al AD por lo que necesito una capa de seguridad. Es por eso que ademas cada uno crea un Certificado y lo instalo en el otro. 

El servidor Radius puede validar usuarios contra diferentes bases de datos para separar tráficos, pudiendo utilizar su base de datos local o el AD. Sigue un orden secuencial para decidir qué hacer con una petición:

1. Base de Datos Local, primero en su propia lista interna de usuarios. Si es un usuario "invitado", Radius autoriza salida directa a Internet, sin acceso a servidores. El proceso termina aquí.
2. Directorio Activo, si el usuario no está en la base local, el Radius actúa como Proxy y consulta al Domain Controller. Si el usuario es un "empleado" El AD verifica las credenciales si son correctas, devuelve "OK" y autoriza el acceso a la red interna.

Por eso el Radius sirve de validador ya que puedo acceder al dominio desde el Radius sin necesidad de acceder al AD directamente. Asi no tengo que preocuparme de las personas que no quiero que accedan a mi red interna pero que tienen que acceder a internet.

**-Radius Interno:**

En una red empresarial se encuentran múltiples tipos de dispositivos de infraestructura (Servicios). Routers, Switches, Firewalls, Cámaras IP, Impresoras, Servidores Multimedia, Bases de Datos...
Para gestionar todos estos dispositivos tendria que crear individualmente cada cuenta en cada dispositivo y cualquier modificación tendria que ir haciendola uno a uno, esto no es nada efectivo.
La solución es el servidor Radius ya que centraliza la gestión. En lugar de loguearse contra la base de datos interna del switch, el switch le pregunta al Radius. 

Ventajas: Solo gestionas las cuentas en un sitio (el Active Directory o la base local del Radius). Si borras al usuario del AD, pierde acceso instantáneamente a todos los routers, switches y cámaras. Permite "hablar" el idioma específico de cada marca. Permite enviar atributos específicos para decirle al dispositivo qué nivel de permisos tiene el usuario y el Radius traduce al comando técnico específico que cada marca necesita entender.
Lo tengo que configurar una primera vez pero luego ya se mantiene. Al igual que con el externo se debe establecer una relación de confianza con el AD.

 - Flujo de Administración:
1. Solicitud: El administrador abre una consola (SSH o Web) contra un Switch e introduce su usuario/contraseña de dominio.
2. Reenvío: El Switch (Cliente Radius) no valida nada. Empaqueta las credenciales y las manda al Servidor Radius.
3. Consulta: El Radius mira en base de datos (local o AD).
4. Respuesta (Authorization): Si las credenciales estan bien se manda un ok, el Radius mira sus políticas y manda el atributo de privilegio.
5. Acceso: El Switch recibe el "Access-Accept" y te deja entrar con los privilegios dados.

### -VALIDACIONES

1. **Medio De Acceso:**
   Una de las formas que tiene el servidor Radius de verificar que el usuario que entra sea correcto es validando el medio por el que accede. Si por ejemplo el usuario esta configurado para entrar con una VPN con una IP 1.1.1.0 si derrepente alguien hace spofin para adivinar la IP y entra por wifi lo puedes configurar para bloquear.
   
3. **Condiciones:**
   Luego tenemos unas condiciones que podemos configurar que se cumplan para permitir el paso. Ejemplos:
   - Rango de Direcciones IP: Se utiliza notación CIDR para agrupar clientes, es decir por ejemplo si el Switch (Cliente Radius) tiene la IP 192.168.1.50, en el servidor configuro el rango 192.168.1.0/24. Esto permite que cualquier switch de esa subred pueda enviar peticiones. Es el protocolo de control de acceso a red basado en puertos.
   - Medios de transmisión: Si es por wifi, cable etc.
   - Protocolo de autentificación: (Estándar 802.1x) Es el protocolo de control de acceso a red basado en puertos. Formas de autenticar:
       - EAP, no requiere de certificado. Usuario + contraseña.
       - PEAP, certificado + usuario + contraseña

        Si tengo usuarios que no acceptan certificados como una impresora, camaras o equipos antiguos utilizo EAP. Si utilizo PEAP obligatoriamente tengo que instalar el certificado.
        Tambien se puede usar la Dirección MAC del dispositivo como si fuera el usuario y la contraseña, el Radius mira una lista de "MACs permitidas".

5. **Politicas:**
   En el servidor radius puedo restringir o permitir la posivilidad de autentificarse por grupos (usuarios o maquinas).
   Puedo restringir el nivel de acceso, por ejemplo puedo hacer que cun usuario del grupo de redes se autentique por medio de un swich a mi servidor radius dandole los permisos que quieras.
   Aquí tienes la última sección de tus apuntes, dedicada a las Políticas y Autorización. Esta es la parte donde se decide "quién entra y qué puede hacer una vez dentro".
   
   El Radius envía instrucciones al Switch/Router mediante atributos:
   Si el usuario pertenece a un grupo permitido, el Radius envía: Access-Accept + Privilege Level.

   Tambien puedo crear grupos de maquinas y elegir desde que maquina puede autenticarse o no.

   Si un usuario o maquina no cumple los requisitos de seguridad y se quiere conectar, puedo bloquearle completamente o puedo por ejemlo dejarle pasar pero metiendole en una red aislada que simule la mia para ver su     comportamiento y aprender de el o sacar evidencias de una falla de seguridad o intento de hakeo.

   En resumen un servidor Radius es un primer filtro de acceso a red.

### RED EDUROAM












