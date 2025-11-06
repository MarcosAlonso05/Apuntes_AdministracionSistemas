# ADMINISTRACIÓN DE SISTEMAS

---

## ÍNDICE

* [VIRTUALIZACIÓN](#seccion1)
* [DIRECTORIO ACTIVO](#seccion2)
* [DNS](#seccion3)
* [DHCP](#seccion4)
* [PROXY](#seccion5)
* [SERVIDOR WEB](#seccion6)
* [CERTIFICADOS](#seccion7)
* [PREGUNTAS TIPO EXAMEN](#seccion8)

---

<a id="seccion1"></a>
## VIRTUALIZACIÓN

Crear una versión virtual de un recurso tecnológico. Las ventajas claves de esto son:

* Ahorro de costes y eficiencia: Permite que un servidor fisico(Host) ejecute multiples maquinas, ahorrando electricidad espacio y dinero.
* Aislamiento: Cada maquina esta aislada de las otras si una tiene un error no afecta a las otras directamente.
* Agilidad y pruebas: Las maquinas son faciles y rapidas de crear, al crear una puedos probar un software o configuracion y si algo sale mal no afecta al sistema real.

La estructura base de un entorno de virtualizacion es la siguiente:

| Capas |
|:------------------------------------:|
| HW Anfitrión (Host) |
| Sistema Operativo del HW Anfitrión (Host OS) |
| Software de virtualización (Hipervisor) |
| Maquinas Virtuales (Guest OS) |

&#8203;
**Recursos del hardware anfitrión**
A la hora de crear maquinas virtuales es imprtante tener en cuenta los recursos del HW anfitrión (CPU, RAM, almacenamiento, red...) ya que estos se repartiran entre las maquinas virtuales y el propio anfitrión. Por lo tanto el pone las limitaciones.
El rendimiento de las maquinas depende entonces de:

* Cantidad de RAM disponibles.
* Numero de nucleos del procesador.
* La velocidad de disco.
* La carga simultanea de las maquinas.  

Esto lleva a otro punto importante. **Cúantas maquinas puedo crear vs. cuantas puedo tener corriendo**.

1. Maquinas que puedo crear:
    Puedo crear tantas maquinas como el espacion en disco me lo permita. Ya que crear VM solo genera archivos de configuración y un disco virtual por lo que mientras esten apagadas no usan RAM ni CPU.

2. Maquinas que puedo tener corriendo:
    Aqui ya interviene todos los recursos del sistema, por lo que saber cuantas maquinas puedo tener corriendo a la vez depende aparte de los recursos que consumen cada una, tambien de los que consumen el OS anfitrión y el HW de Virtualización.

Ej. Practico:
*¿Cúantas maquinas de 4GB RAM puedo correr con un HW Anfitrion que usa 32GB y un OS Anfitrión 8 GB?*
No se puede saber ya que desconocemos lo que ocupa el sofware de virtualización y no tenerlo encuenta acarrearia serios problemas.
*¿Y con el Hipervisor que usa 2GB RAM?*
32-8-2 = 22 GB
22/4 = 5 Maquinas corriendo a la vez.

&#8203;
**Modos de la targeta de red**
La targeta de red virtual puedo operar en varios modos según el nivel de aislamiento o conexión:

* Bidged: Conecta la VM directamente a la red física. Tiene su propia IP en la red local.
* NAT: La VM comparte la conexión del host, saliendo a Internet a traves de la IP del anfitrión.
* Host-Only: La VM solo se comunica con el host (no acceso a internet).
* Internal / Private: Solo se comunica con otras VMs del mismo entorno virtual.

&#8203;
**Conceptos principales**

**-vMotion:** Permite migrar una máquina virtual en ejecución desde un servidor físico a otro sin apagarla y sin que el usuario o las aplicaciones noten la migración.
¿Como funciona?
Ambos host deben estar conectados a la misma res y compartir el almacenamiento.
Se copia la memoria RAM de la maquina virtual del Host A al Host B mientras sigue funcionando (cualquier cambio se sigue rastreando).
Cuando casi toda la memoria está copiada y solo quedan unos pocos cambios, vMotion pausa la VM por milisegundos, copia los últimos datos y la reanuda en el Host B (Las conexiones de red y las operaciones continúan exactamente donde estaban).
El Host A libera los recursos de la VM, y el Host B pasa a ser su nuevo anfitrión.

**-Observer:** El Observer (o función de alta disponibilidad) vigila las máquinas virtuales. Si detecta que una máquina o servidor físico ha fallado, la reinicia o la migra a otro host automáticamente para evitar interrupciones.

**-Snapshot:** Un snapshot guarda el estado actual de una máquina virtual en un momento dado.
Esto es muy util ya que uno de los usos que se le puede dar es por ejemplo, si el sistema se actualiza hacer una snapshot antes de actualizarlo para en caso de que luego haya problemas podamos volver.
Esto no reemplaza un backup, ya que no es una copia completa ni segura ante fallos físicos.
Importante, la snapshot se debe hacer con la maquina apagada, ya que puede haber procesos ejecutandose lo que puede conllevar errores.

**-Diferencias entre .ovf y .ova:**

* OVF: Es un conjunto de archivos que describen una maquina virtual. Son mas faciles de editar pero ocupan mas espacio.
* OVA: Es un único archivo comprimido que contiene todos los componentes del OVF. Es ideal para importar/exportar una VM completa.

---

<a id="seccion2"></a>
## DIRECTORIO ACTIVO

El Directorio Activo (AD) es un servicio que permite gestionar usuarios, equipos, grupos y políticas de seguridad de una red de forma centralizada.
Cuando instalas un servidor de directorio activo necesitas tambien un servido dns, ya que cuando un cliente quiere asociarse al dominio quiere acceder con nombre no por IP.

**ESTRUCTURA:**
El Directorio Activo está organizado jerárquicamente, esto facilita dividir responsabilidades, delegar permisos y mantener un control centralizado.

Podemos tener un dominio y de este generar subdominios, estas seran las ramas del dominio, tambien podemos generar ramas de los propios subdominios.
Entre dos o mas dominios se pueden establecer relaciones de confianza (bosques). Esto permitiria por ejemplo que los user de un dominio puedan acceder a recursos del otro.
Otra ventaja es que si una persona se mueve entre dominios no sería necesario crear dos cuentas y duplicar usuario, sino que serviria la misma cuenta.
Normalmente en las relaciones de confianza se suelen pactar unos permisos entre los integrantes, ya que no sería logico darse acceso completo el uno con el otro, por lo que se pactan la relacion que se va a tener.

Ramas (Subdominio) --> Arbol (Dominio) --> Bosque (Relación de Confianza)

**USUARIOS Y GRUPOS:**

* Usuarios: Estos en el directorio activo representan una identidad digital. Cada usuario tiene:
  * Nombre de inicio de sesión.
  * Contraseña.
  * Atributos personales.
  * Permisos y políticas aplicadas.

Los usuarios se autentican mediante Kerberos cuando inician sesión en el dominio.

* Maquinas: Funciona igual que una cuenta de usuario pero esta representa a un equipo.
Cada máquina del dominio tiene su propia cuenta en el Directorio Activo. Sirve para que el equipo se autentique contra el dominio y pueda usar recursos compartidos, políticas, etc.

-Diferencias entre usuario y usuario maquina

| Característica | Usuario persona | Usuario máquina |
|:--------------:| :-------------: | :-------------: |
| Representa | Un ser humano | Un equipo o servidor |
| Credenciales | Usuario y contraseña | Nombre de máquina + contraseña |
| Inicio de sesión | Sesión interactiva | Sesión del sistema |
| Propósito | Acceder a recursos, servicios, aplicaciones | Autenticarse en el dominio y aplicar políticas |

* Grupos: Sirven para organizar usuarios o equipos que tienen permisos o funciones similares.
Estos te facilitan la administración de permisos.
Permiten aplicar politicas GPO (herramientas de Active Directory que permiten gestionar y aplicar configuraciones de forma centralizada) a usuarios y equipos dentro de un dominio.
Evita tener que configurar permisos uno por uno.

Tipos de grupos:
-Grupos de seguridad: Se usan para asignar permisos a recursos (por ejemplo, acceso a carpetas, impresoras o servidores).
-Grupos de distribución: Se usan para enviar correos o notificaciones a varios usuarios (no sirven para permisos). Comunmente se pueden dividir en los diferentes departamentos de la empresa.

El alcance de los grupos puede ser local (solo equipo o dominio concreto), global (todo el dominio) o universal (todo el bosque).

Grupos de maquinas y usuarios van por separados aunque se relacionan entre  si.

**GESTION DE CREDENCIALES Y AUTENTIFICACIÓN:**

Proceso General:
Un cliente hace solicitud de acceso al servidor, el servidor valida las credenciales con Kerberos y LDAP, si son correctas este emite un ticket.
El ticket tiene un timeout (configurable por el administrador), si se hace una solicitud de acceso y se acaba el timeaout el usuario no se podra loguear.

-KERBEROS: Es el protocolo de autentificación por defecto del Directorip Activo.

-LDAP: Protocolo que permite consultar, buscar y modificar información dentro del Directorio Activo.

Cada solicitud de acceso va con fecha y hora para ver el tiempo que tarda. Si cada maquina tiene una fecha y hora diferente te dara error en la autentificación. Dos soluciones:

  1. (Poco eficiente y cara) Hacer directorios activos en cada país.
  2. Usa un servidor NTP (Network Time Protocol) que marca una hora oficial para todos los equipos, así todos tienen la misma.

**POLITICAS (GPO – Group Policy Object):**

Las GPO permiten que los administradores controlen, configuren y gestionen de forma centralizada el comportamiento de los usuarios y equipos de un dominio.
Define reglas, configuraciones y restricciones que se aplican automáticamente cuando el usuario o el equipo se conecta al dominio.

* Políticas de usuario:
  Definen qué puede hacer o no un usuario dentro del dominio y cómo se comporta su entorno de trabajo.
  Estas políticas se aplican a las cuentas de usuario, no a las máquinas, y sirven para mantener el control del entorno.
  -Ejemplos comunes:
  * Restringir el acceso al Panel de control o a Configuración de Windows.
  * Bloquear el Administrador de tareas o el registro del sistema.
  * Redirigir carpetas (por ejemplo, que “Documentos” se guarde en un servidor central).
  * Ejecutar scripts de inicio o cierre de sesión.
  * Establecer fondos de pantalla corporativos o configuraciones de escritorio.
  * Ocultar o mostrar unidades de disco (por ejemplo, ocultar C: a los usuarios normales).

* Políticas de contraseña:
  Regulan cómo deben ser las contraseñas de los usuarios del dominio para mejorar la seguridad.
  -Parámetros más comunes:
  * Longitud mínima: número mínimo de caracteres.
  * Complejidad: si debe incluir mayúsculas, minúsculas, números y símbolos.
  * Caducidad: cada cuántos días el usuario debe cambiarla.
  * Historial: evita reutilizar contraseñas anteriores.
  * Bloqueo de cuenta: número de intentos fallidos antes de bloquear temporalmente al usuario.
  * Tiempo de bloqueo: cuánto tiempo debe esperar el usuario si su cuenta se bloquea.

* Políticas de seguridad:
  Afectan tanto a usuarios como a equipos, y regulan aspectos de acceso, permisos y seguridad del sistema operativo. Estas políticas garantizan que todos los equipos cumplan con las normas de seguridad establecidas por la organización.
  -Ejemplos:
  * Qué usuarios pueden apagar o reiniciar el equipo.
  * Quién puede instalar o desinstalar software.
  * Qué recursos o carpetas están protegidos o accesibles.
  * Si se permite conexión por escritorio remoto (RDP).
  * Configuración de firewall, actualizaciones automáticas, bloqueo de puertos, etc.
  * Reglas de auditoría (registrar accesos, intentos de login, etc.).

En el Directorio Activo, los usuarios y equipos se pueden agrupar (unidades organizativas), esto permite ordenar y aplicar políticas específicas sin afectar a todo el dominio.

**CUENTA ADMINISTRADOR LOCAL:**
La cuenta de administración local es una cuenta propia del equipo, no dependiente del dominio. Esta cuenta tiene todos los privilegios sobre ese equipo en concreto, pero no sobre otros equipos de la red.
Esta es extrictamente necesaria ya que si el controlador de dominio falla, el servidor DNS no responde, o no hay conexión de red, el usuario no puede iniciar sesión con su cuenta de dominio.
En ese caso, la única forma de acceder al sistema es con la cuenta de administrador local.

**Pérdida de conexión con el dominio:**
Un PC pertenece al dominio, pero el controlador de dominio no responde o no hay conexión a la red.
Solución:
Desconecta el cable de red o desactiva el adaptador. Inicia sesión con las credenciales en caché del dominio (Windows guarda temporalmente las últimas credenciales usadas).

---

<a id="seccion3"></a>
## DNS

*Puerto 53*
El DNS es un systema que traduce nombres de dominio a sus direcciones IP correspondientes y viceversa. Su función es hacer posible la navegación por nombres en lugar de por numeros.

**Funcionamiento:**
Cuando un equipo quiere acceder a una web o servidor, se sigue un proceso de resolución DNS.

1. El usuario escribe una URL.
2. El sistema operativo revisa su caché local DNS (por si ya resolvió ese nombre antes).
3. Si no está en caché, pregunta al servidor DNS configurado.
4. El servidor DNS, si no tiene la respuesta, va preguntando jerárquicamente a otros servidores DNS.
5. Una vez obtiene la IP, la envía al cliente, que ya puede conectarse.

**Tipos de registros:**

* A Record (Address Record): Asocia un nombre de dominio con una dirección IPv4.
  -Ej: `www.ejemplo.com  ->  192.168.1.10`
* AAAA Record (Quad-A Record): Asocia el nombre de dominio con una dirección IPv6.
  -Ej: `www.ejemplo.com  ->  2001:0db8:85a3::8a2e:0370:7334`
* PTR Record (Pointer Record): Asocia una dirección IP con un nombre de dominio. Se usa para resolución inversa (reverse lookup).
  -Ej: `192.168.1.10 -> www.ejemplo.com`

**Ataques relacionados con DNS:**

Pharming: Consiste en falsear la resolución DNS, redirigiendo al usuario a una página falsa aunque escriba bien la URL.
Se puede hacer modificando el archivo hosts o manipulando un servidor DNS comprometido.
(No confundir con phising, que es un engaño al usuario para que introduzca datos en una web falsa, depende del engaño humano, no de modificar DNS.)

---

<a id="seccion4"></a>
## DHCP

*Puertos utilizados: Servidor -> UDP 67 / Cliente -> UDP 68*
El DHCP es un protocolo que asigna automaticamente direcciones IP y otra información de configuración de red a los dispositivos.
Su función principal es automatizar el asignar direcciones, en lugar de que un administrador tenga que configurar manualmente uno por uno.

**Funcionamiento del proceso DHCP (DORA):**

1. Discovery: El cliente envía un mensaje DHCPDISCOVER por broadcast (a toda la red) para buscar un servidor DHCP disponible.
2. Offer: El servidor responde con una DHCPOFFER, ofreciendo una IP libre y los parámetros de red.
3. Request: El cliente contesta con un DHCPREQUEST, confirmando que acepta la oferta.
4. Acknowledge (ACK): El servidor confirma la cesión con un DHCPACK, y la máquina ya puede usar esa IP.

**Parámetros que puede asignar el DHCP:**

-Dirección IP
-Máscara de red (Subnet Mask)
-Puerta de enlace (Gateway)
-Servidor DNS
-Servidor Proxy
-Tiempo de concesión (Lease Time)
-Nombre de dominio (Domain Name)
-Servidor NTP (hora)
-Rutas estáticas

**Tipos de asignación:**

* Estática: IP configurada directamente en el equipo. No pregunta al servidor DHCP, se autoconfigura.
  -Problema: Si te mueves a otra red con otro rango, no tendrás conexión.
* Dinámica: IP asignada automáticamente del rango DHCP.
  La dirección tiene un tiempo de concesión (Lease Time), cuando el tiempo expira, la IP vuelve a estar disponible.
* Híbrida: Combinación de las anteriores (rango dinámico + reservas estáticas).
* Estatica por MAC: IP se asigna desde el servidor DHCP, pero está reservada para una dirección MAC concreta.
  El cliente siempre recibe la misma IP, porque el servidor la tiene guardada para él.
  Si mueves el equipo a otra red con otro servidor DHCP, seguirá funcionando, solo que recibirá una IP del nuevo rango.
* Dinamica por MAC: El servidor sigue usando DHCP, pero identifica al cliente por su MAC. Si la máquina se conecta dentro de la misma red recibe una IP dinamicamente.
 Asi puede conectarse desde otro lugar sin tener problemas.
  Ventaja: permite tener una IP dentro de la red corporativa para localizar dispositivos en especifico, pero adaptable fuera de ella.

DHCP se usa para los clientes, para los servidores se usan direcciones estaticas configuradas manualmente.

-Lease Time: Es el tiempo que el cliente puede usar la IP asignada antes de que expire. El tiempo es configurable.
Cuando está a punto de expirar, el cliente pide renovar la concesión. Si no lo hace el servidor libera la IP para asignársela a otro equipo.

-Gestión de conflictos:
  El servidor DHCP no asigna IPs ya ocupadas (detecta si están en uso).
  Si varias IPs están ocupadas y el rango es corto, el servidor puede dar la IP con menor tiempo restante de lease.
  Hay casos en los que las maquinas no pueden hacer un discovery, en estos se utilizan asignacion por mac estatica.

**Seguridad DHCP:**

DHCP Spoofing:
Es un ataque en el que alguien instala un servidor DHCP falso en la red.
Este servidor responde antes que el legítimo, asignando IPs y puertas de enlace falsas.
Así puede hacer que el tráfico de los usuarios pase primero por su máquina, permitiendo interceptar o modificar datos (ataque tipo Man-in-the-Middle).

-Solución:
Configurar seguridad DHCP Snooping en los switches.
Solo permitir respuestas DHCP desde puertos confiables.

**Integración DNS + DHCP:**

En entornos de empresa o redes con Directorio Activo, el DHCP y el DNS trabajan juntos:
Cuando el DHCP asigna una IP a un equipo, automáticamente actualiza el DNS, registrando el nombre del cliente con su IP.
Si el lease expira y la IP se reasigna a otro equipo, el DNS se actualiza también, manteniendo coherencia.

Esto se conoce como DNS dinámico (DDNS).

---

<a id="seccion5"></a>
## PROXY

Un servidor proxy es un intermediario entre un cliente y un recurso al que quiero acceder. Su función principal es canalizar, filtrar o gestionar las solicitudes que pasan entre los dos extremos.

**Utilidades principales del servidor proxy:**

1. Ahorro de ancho de banda:
   El proxy puede almacenar en caché páginas o recursos web que los usuarios visitan frecuentemente.
   Cuando otro usuario pide la misma página, el proxy la entrega desde su memoria, sin necesidad de volver a Internet.
   Esto reduce el consumo de ancho de banda y acelera la navegación.
   En empresas grandes se analiza el tráfico y se programan las páginas más usadas para almacenarlas a ciertas horas. Se actualiza cada x tiempo (configurable).
&#8203;
2. Seguridad y aislamiento:
   Se puede colocar un proxy entre servidores sensibles.
   `Internet → Firewall → Proxy → Servidor web → Base de datos`
   Así, el servidor web real no está directamente expuesto a Internet. Si atacan el proxy, solo se cae esa máquina, no el servidor real.
   En este ejemplo, desde fuera solo se pueden acceder a la vista de la web. Entonces a la hora de comunicarse con la base de datos si la dirreccion no es del Proxy lo tira.
   También se puede usar un proxy de base de datos, para que los usuarios o aplicaciones no accedan directamente al servidor de datos, sino a través del proxy.
   En caso de exceso de consultas o ataques, el proxy actúa como “escudo” y evita que se caiga la base de datos real.
&#8203;
3. Control y filtrado de contenido:
   Permite bloquear o permitir el acceso a determinados sitios o servicios.
   Las empresas lo usan para filtrar tráfico, limitar redes sociales o registrar actividad de usuarios.
&#8203;
4. Ocultación de IP pública:
   Cuando navegas a través de un proxy externo, la página web ve la IP del proxy, no la tuya.
   Así puedes ocultar tu dirección IP real o simular estar en otro país (proxies anónimos o públicos).
   Importante: El Proxy solo oculta tu IP, pero no cifra los datos, ademas el si estas navegando con un Proxy externo a ti el internet no tienen tu IP pero si el propietario del Proxy.
   Si se quiere mayor anonimato y seguridad hay que utilizar una VPN.
&#8203;
5. Proxy de correo (Mail Proxy):
   Se usa para filtrar correos entrantes y salientes, detectar spam, virus o phishing, y proteger los servidores de correo reales. Ejemplo: MailMarshal o SpamAssassin.

**Proxy como medida de seguridad:**

Un proxy no es un sistema de seguridad en sí mismo, pero proporciona cierta protección indirecta.
-Oculta los servidores reales.
-Reduce la exposición directa a Internet.
-Permite aplicar políticas de acceso y filtrado.
-Puede absorber picos de carga o ataques básicos

Un Proxy interno es importante por seguridad ante ataques internos.

**Tipos comunes de Proxy:**

| Tipo | Función principal |
|:----:|:-----------------:|
| Proxy HTTP/HTTPS | Filtra tráfico web (navegadores) |
| Proxy de caché | Ahorra ancho de banda almacenando páginas visitadas |
| Proxy inverso | Protege y distribuye carga a servidores internos |
| Proxy de base de datos | Intermediario entre aplicaciones y BD |
| Proxy de correo | Filtra y analiza correos (spam, virus) |
| Proxy transparente | Los usuarios no saben que están pasando por él |
| Proxy anónimo | Oculta IP pública del cliente |

**Tor:**

sa una red de muchos proxys encadenados que van cambiando a cada rato (la base de la Deep Web), lo que hace muy difícil rastrear al usuario.

---

<a id="seccion6"></a>
## SERVIDOR WEB

Un servidor web es un software encargado de recibir peticiones de los clientes (navegadores) y responder con páginas web. Su función principal es almacenar, procesar y entregar contenido web.

Se puede montar en Windows usando IIS (Internet Information Services) o en Linux (Apache, Cherokee, lamp).

**Partes:**

* Código: html, php, perl ...
* Entorno o framework: Herramientas que facilitan la creación y gestión de interfaces o lógica de aplicación. FlutterFlow, Dart, Node...
* Base de datos (opcional): Sistema para almacenar información dinámica. Firebase, MySQL, Oracle...

**Ubicación:**
Dependiendo de las necesidades se encontrara en una parte o en otra.
Si por ejemplo solo va a ser usado por los empleados, y no se requiere el acceso al esterior se monta en la red interna.
O si quiero que sea visible para los clientes lo pongo tras el firewall y que solo se pueda acceder a el atraves de un Proxy.
La ubicación del mismo dependera de la seguridad y los usuarios que tengan acceso a el.

**Servidor Web con DB:**

El servidor web y la base de datos pueden estar alojados tanto en el mismo servidor o diferente.
Si estan en el mismo, la ventaja es que es más rapido, tendra menos latencia pero la desventaja es que si se cae uno cae también el otro.
Si estan separados, tenemos más seguridad pero nos surge algo más de latencia entre web y DB.

Cuando se manejan usuarios o datos sensibles, se recomienda separarlos (mejor aislamiento de funciones y seguridad).

**Protocolos Web:**

| Protocolo | Puerto | Seguridad | Descripción |
|:---------:|:------:|:---------:|:-----------:|
| HTTP | 80 | No cifrado | La información viaja en texto plano |
| HTTPS | 443 | Cifrado (SSL/TLS) | La información viaja encriptada |

-Diferencia entre HTTP y HTTPS:
Con HTTP la comunicación no está cifrada, los datos viajan en texto plano, por lo que pueden ser interceptados.
En HTTPS usa un certificado digital (SSL/TLS) para encriptar los datos, de modo que solo el servidor y el cliente puedan leerlos.

---

<a id="seccion7"></a>
## CERTIFICADOS

Un certificado SSL es un archivo que contiene información que identifica a un servidor web y permite cifrar las comunicaciones.
Estos los emite una CA (Autoridad Certificadora), esta es una entidad de confianza que valida la identidad del servidor o la empresa antes de emitir el certificado.

**Estructura de un certificado:**

Un certificado HTTPS completo está formado por tres niveles encadenados:

| Nivel | Descripción |
|:-----:|:-----------:|
| ROOT | Certificado principal de la CA (máxima autoridad) |
| INTERMEDIATE | Certificados de entidades colaboradoras o filiales de la CA |
| FINAL / MACHINE CERTIFICATE | Certificado propio de tu servidor o máquina |

ROOT e INTERMEDIATE son los comunes y el certificado maquina es individual de cada maquina.

(Si por ejemplo tengo de Root uax.es y de colaborador Ue.es con una maquina con este certificado puedo hacceder a el servidor de la uax o de ue.es).

**Tipos de certificados:**

* **Certificado Autofirmado:**
  Estos son emitidos por uno mismo sin pasar por una CA oficial.
  Se suenlen usar en redes internas, en las que el administrador genera su propio certificado y lo instala manualmente. Estos no tienen Root.
&#8203;
  -Ej: Tengo un Directorio Activo y unos Servidores web internos.
  Entonces, genero un certificado con CA interna y lo instalo en la web interna.
  Los portátiles de la empresa llevan ese certificado raíz, por lo que confían en el servidor y la conexión es cifrada dentro de la red local (las empresas cuando piden portatiles los suele pedir con certificado instalado).
&#8203;
  Esto causaba un problema cualquiera podría generar un certificado falso con el nombre de otra web (“me hago pasar por una CA”). Esto permitiría ataques de suplantación (phishing) o interceptación (MITM).
  Por eso, solo confían en certificados firmados por CAs incluidas en su lista de confianza, entonces en entornos internos aún se pueden usar, si se instala el certificado raíz en todos los clientes.
&#8203;
  Ahora por seguridad los nabegadores modernos no se traga los certificados autofirmados y no te deja acceder.
&#8203;
* **Certificado Oficiales:**
  Estos estan emitidos por unas CA oficiales conocidas a nivel mundial (DigiCert, Sectigo, GlobalSign, Let’s Encrypt, etc.).
  Son verificados y reconocidos automáticamente por los navegadores.
  Para solicitarlos, se genera un CSR, es un documento de texto generado en la maquina.
  -CSR Contiene:
  * Commond Name (dominio o IP).
  * Organization.
  * Organizational Unit.
  * Country.
  * State.
  * Key Size (tamaño de clave, normalmente 2048 bits).
  * Información de contacto.
&#8203;

  La CA me comprueba que los datos son correctos (en especial el common name). Y esta emite el certificado que sera:
  * Root: CA.
  * Intermediates: Resto de certificadoras oficialeas.
  * Final: Certificado para la web que se cetifica
  Ej: Root CA: DigiCert -> Intermediate CA: GlobalSign -> Server Certificate: `www.miempresa.com`.

---

<a id="seccion8"></a>
## *PREGUNTAS TIPO EXAMEN*

**Virtualización:**

*-¿Cúantas maquinas de 4GB RAM puedo correr con un HW Anfitrion que usa 32GB y un OS Anfitrión 8 GB?*
No se puede saber ya que desconocemos lo que ocupa el sofware de virtualización y no tenerlo encuenta acarrearia serios problemas.
*-¿Y con el Hipervisor que usa 2GB RAM?*
32-8-2 = 22 GB
22/4 = 5 Maquinas corriendo a la vez.

&#8203;
**Directorio Activo:**

*-¿Necesario una cuenta de administrador local en los portatiles?*
Si es obligatorio por si pierdo la conexion con el directorio activo

*-¿Que otro servicio tengo que tener levantado a la hora de levantar un servidor Activo?*
DNS

*-¿Si un pc conectado al dominio y con cuenta de dominio, si no es capaz de acceder al mismo, ademas el directorio activo no responde, como accdedes?*
Quitando el cable de red e intentar credenciales locales del dominio

*-¿Por que es necesario levantar dns en directorio activo?*
Cuando un cliente quiere asociarse al dominio se quiere acceder con nombre no por iP

&#8203;
**DNS:**

*-¿Como se llama el ataque en el que modifica la resolucion de DNS?*
Pharming, falsear resolucion de DNS (No confundir con phising)

&#8203;
**DHCP:**

*-¿Que parametros le puedo configurar a una maquina por el servidor DHCP?*
Dirección IP, Máscara de red (Subnet Mask), Puerta de enlace (Gateway), Servidor DNS, Servidor Proxy, Tiempo de concesión (Lease Time), Nombre de dominio (Domain Name), Servidor NTP (hora), Rutas estáticas.

*-¿Porque es mas efectivo dinamicas por mac que estaticas?*
Para que si se conecta desde fuera se pueda asignar la misma ip por lo que si la tubiera estaticas, no podria conectarse desde fuera.

&#8203;
**Proxy:**

*-¿Que me garantiza el anonimato en mis comunicaciones?*
VPN

*-¿Que me garantiza el anonimato de mi ip publica?*
Proxi
Tor
VPN
x Todas las anteriores

&#8203;
**Servidor Web:**

*-¿Diferencia entre http y https?*
http, puerto: 80 no cifrado.
https puerto: 443 y encriptado mediante un certificado SSL.

&#8203;
**Certificados:**

*-¿Si tengo maquina encargada de generar certificados que parte de la estructura sera diferente en mi servidor web y en mis servidor proxi?*
El certificado maquina(final) porque es propio de cada maquina.




