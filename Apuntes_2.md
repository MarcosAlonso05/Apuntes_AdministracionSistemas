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

En las empresas se suele usar una combinacion de los diferentes RAIDS para maximizar el espacio pero manteniendo la seguridad.
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
Otra función es tener una red wifi de mi conpañia montada en una dmz del firewall dedicada a los telefonos de empleados. Ya que los portatiles se pueden tener controlados pero cosas como los moviles de los empleados pueden ser mas peligrosos.

Entonces aqui es donde entra el RADIUS, que se encarga de validar hasta donde puede llegar un usuario. Bloquea y no permite el paso a zonas indevidas.
Aunque fisicamente es el mismo cable y la misma antena, pero lógicamente son dos mundos separados.

### -FUNCIONAMIENTO

El Servidor Radius es:
-Sistema de control a red, áctúa como punto de decisión para permitir o denegar el acceso a la red basándose en políticas predefinidas.
-Proxy (RADIUS Proxy), Capacidad de retransmitir las peticiones de autenticación a otros servidores RADIUS externos.
-Autentificador, Verifica las credenciales del usuario contra una base de datos externa o local.

### -VALIDACIONES

1. **Medio De Acceso:**
2. **Condiciones:**
3. **Politicas:**

