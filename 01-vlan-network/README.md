# Documentación de red --- Cisco Packet Tracer

## 1. Descripción general

Red empresarial simulada en Cisco Packet Tracer con un router Cisco
2811, un switch principal Cisco 2960, dos switches departamentales, un
punto de acceso inalámbrico, seis teléfonos IP, computadoras y dos
impresoras.

La red está segmentada mediante VLAN. El router realiza el enrutamiento
entre VLAN (Router-on-a-Stick), entrega direcciones mediante DHCP y
ejecuta Cisco CME para la telefonía IP. Una ACL restringe el acceso de
la red de invitados a las redes internas.

### Estado funcional comprobado

-   DHCP funciona para las redes de RRHH, ADMIN y VOICE.
-   Los equipos de RRHH y ADMIN pueden comunicarse entre sí.
-   GUESTS está aislada de RRHH, ADMIN y VOICE mediante una ACL.
-   Los clientes GUESTS tienen acceso a Internet.
-   El WiFi funciona con el SSID `GUESTS-WIFI`.
-   Los seis teléfonos IP reciben IP, registran sus extensiones y pueden
    realizar llamadas.
-   Las dos impresoras reciben IP por DHCP y responden a ping desde una
    PC de su departamento.

> **Seguridad WiFi:** el SSID `GUESTS-WIFI` está funcionando, pero no se
> configuró autenticación ni contraseña. Por lo tanto, la red
> inalámbrica está abierta en el estado documentado.

------------------------------------------------------------------------

## 2. Topología física y conexiones

  -----------------------------------------------------------------------
  Dispositivo       Interfaz          Conectado a       Función
  ----------------- ----------------- ----------------- -----------------
  RT (Cisco 2811)   Fa0/0             principal-SW      Enlace
                                      Fa0/1             router-switch,
                                                        trunk

  principal-SW      Fa0/1             RT Fa0/0          Trunk hacia el
                                                        router

  principal-SW      Gi0/1             rrhh-sw Gi0/1     Trunk hacia RRHH

  principal-SW      Gi0/2             admin-sw Gi0/1    Trunk hacia ADMIN

  principal-SW      Fa0/2             Access Point      Acceso a VLAN 10

  rrhh-sw           Fa0/1--Fa0/3      Teléfonos IP con  Datos VLAN 20 y
                                      PC conectada      voz VLAN 40
                                      detrás            

  rrhh-sw           Fa0/4             Impresora RRHH    VLAN 20

  rrhh-sw           Fa0/5             PC adicional RRHH VLAN 20

  admin-sw          Fa0/1--Fa0/3      Teléfonos IP con  Datos VLAN 30 y
                                      PC conectada      voz VLAN 40
                                      detrás            

  admin-sw          Fa0/4             Impresora ADMIN   VLAN 30

  admin-sw          Gi0/1             principal-SW      Trunk hacia el
                                      Gi0/2             switch principal
  -----------------------------------------------------------------------

Hay una PC adicional en ADMIN. El número de puerto de esa conexión no
quedó registrado en la información disponible y debe verificarse en la
topología de Packet Tracer.

------------------------------------------------------------------------

## 3. Plan de direccionamiento y VLAN

  --------------------------------------------------------------------------------------------
            VLAN Nombre      Red               Máscara           Gateway        Uso
  -------------- ----------- ----------------- ----------------- -------------- --------------
              10 GUESTS      192.168.10.0/24   255.255.255.0     192.168.10.1   WiFi de
                                                                                invitados

              20 RRHH        192.168.20.0/27   255.255.255.224   192.168.20.1   Computadoras e
                                                                                impresora de
                                                                                RRHH

              30 ADMIN       192.168.30.0/27   255.255.255.224   192.168.30.1   Computadoras e
                                                                                impresora de
                                                                                ADMIN

              40 VOICE       192.168.40.0/27   255.255.255.224   192.168.40.1   Teléfonos IP
  --------------------------------------------------------------------------------------------

### Direcciones reservadas para infraestructura

Se excluyeron del DHCP las siguientes direcciones:

  Pool     Rango excluido
  -------- -----------------------------
  GUESTS   192.168.10.1--192.168.10.10
  RRHH     192.168.20.1--192.168.20.5
  ADMIN    192.168.30.1--192.168.30.5
  VOICE    192.168.40.1--192.168.40.5

Las direcciones excluidas no se entregan automáticamente a clientes
DHCP. Esto deja disponibles direcciones bajas para gateways y posibles
equipos con IP fija o reservas.

------------------------------------------------------------------------

# 4. Configuración por dispositivo

## 4.1 Router principal --- RT (Cisco 2811)

### Interfaz física

La interfaz `FastEthernet0/0` conecta el router con `principal-SW Fa0/1`
y debe estar habilitada:

``` cisco
enable
configure terminal
interface fastEthernet 0/0
 no shutdown
exit
```

### Router-on-a-Stick: subinterfaces

Cada subinterfaz representa una VLAN y funciona como gateway de esa red:

``` cisco
interface fastEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface fastEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.224
exit

interface fastEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.224
exit

interface fastEthernet 0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.224
exit
```

### DHCP: exclusiones

``` cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.5
ip dhcp excluded-address 192.168.30.1 192.168.30.5
ip dhcp excluded-address 192.168.40.1 192.168.40.5
```

### DHCP: pools

El nombre del pool de invitados quedó escrito como `GUETS` en el router.
Es solamente el nombre del pool; la red y el funcionamiento DHCP
corresponden a GUESTS.

``` cisco
ip dhcp pool GUETS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
exit

ip dhcp pool RRHH
 network 192.168.20.0 255.255.255.224
 default-router 192.168.20.1
 dns-server 8.8.8.8
exit

ip dhcp pool ADMIN
 network 192.168.30.0 255.255.255.224
 default-router 192.168.30.1
 dns-server 8.8.8.8
exit

ip dhcp pool VOICE
 network 192.168.40.0 255.255.255.224
 default-router 192.168.40.1
 dns-server 8.8.8.8
 option 150 ip 192.168.40.1
exit
```

La opción DHCP 150 informa a los teléfonos IP la dirección del servidor
de configuración/telefonía. En este laboratorio apunta a la dirección
del router en la VLAN VOICE.

### ACL: aislamiento de GUESTS

Se creó una ACL extendida llamada `GUETS-FILTER`:

``` cisco
ip access-list extended GUETS-FILTER
 10 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.31
 20 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.31
 30 deny ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.31
 40 permit ip 192.168.10.0 0.0.0.255 any
exit
```

La ACL se aplicó en sentido de entrada en la subinterfaz de GUESTS:

``` cisco
interface fastEthernet 0/0.10
 ip access-group GUETS-FILTER in
exit
```

**Resultado:** los paquetes originados en GUESTS con destino a RRHH,
ADMIN o VOICE se descartan. El resto del tráfico de GUESTS se permite.
El acceso a Internet desde GUESTS fue probado y funciona en el
laboratorio.

Las ACL se procesan de arriba hacia abajo. Por eso los tres `deny` deben
estar antes del `permit` general.

### Telefonía IP: Cisco CME

Se configuró CME en el router:

``` cisco
telephony-service
 max-ephones 6
 max-dn 6
 ip source-address 192.168.40.1 port 2000
 auto assign 1 to 6
```

-   `max-ephones 6`: permite registrar hasta seis teléfonos.
-   `max-dn 6`: permite crear seis números de directorio/extensiones.
-   `ip source-address`: define la dirección y el puerto del servicio
    telefónico.
-   `auto assign 1 to 6`: habilita la asignación automática de los
    números de directorio a los teléfonos.

### Extensiones

``` cisco
ephone-dn 1
 number 201
exit

ephone-dn 2
 number 202
exit

ephone-dn 3
 number 203
exit

ephone-dn 4
 number 301
exit

ephone-dn 5
 number 302
exit

ephone-dn 6
 number 303
exit
```

  Departamento   Extensiones
  -------------- ---------------
  RRHH           201, 202, 203
  ADMIN          301, 302, 303

Los teléfonos necesitaron desconectarse y reconectarse a la corriente
para reiniciarse, volver a solicitar su configuración y recibir las
extensiones. Luego se comprobó que las llamadas funcionaban.

------------------------------------------------------------------------

## 4.2 Switch principal --- principal-SW (Cisco 2960)

### VLAN creadas

``` cisco
enable
configure terminal
vlan 10
 name GUESTS
exit
vlan 20
 name RRHH
exit
vlan 30
 name ADMIN
exit
vlan 40
 name VOICE
exit
```

### Puertos trunk

``` cisco
interface fastEthernet 0/1
 switchport mode trunk
 no shutdown
exit

interface gigabitEthernet 0/1
 switchport mode trunk
 no shutdown
exit

interface gigabitEthernet 0/2
 switchport mode trunk
 no shutdown
exit
```

  Puerto   Destino           Modo
  -------- ----------------- -------
  Fa0/1    Router RT Fa0/0   Trunk
  Gi0/1    rrhh-sw Gi0/1     Trunk
  Gi0/2    admin-sw Gi0/1    Trunk

### Puerto del Access Point

``` cisco
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 no shutdown
exit
```

El puerto del AP pertenece a VLAN 10, por lo que los clientes
inalámbricos quedan en la red GUESTS.

------------------------------------------------------------------------

## 4.3 Switch de RRHH --- rrhh-sw (Cisco 2960)

### VLAN creadas

``` cisco
enable
configure terminal
vlan 20
 name RRHH
exit
vlan 40
 name VOICE
exit
```

### Puertos para teléfonos IP y computadoras

``` cisco
interface range fastEthernet 0/1 - 3
 switchport mode access
 switchport access vlan 20
 switchport voice vlan 40
 no shutdown
exit
```

Los puertos funcionan con una VLAN de datos (20) para la PC conectada
detrás del teléfono y una VLAN de voz (40) para el teléfono IP.

### Impresora RRHH

``` cisco
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
```

### PC adicional RRHH

``` cisco
interface fastEthernet 0/5
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
```

### Uplink al switch principal

``` cisco
interface gigabitEthernet 0/1
 switchport mode trunk
 no shutdown
exit
```

------------------------------------------------------------------------

## 4.4 Switch de ADMIN --- admin-sw (Cisco 2960)

### VLAN creadas

``` cisco
enable
configure terminal
vlan 30
 name ADMIN
exit
vlan 40
 name VOICE
exit
```

### Puertos para teléfonos IP y computadoras

``` cisco
interface range fastEthernet 0/1 - 3
 switchport mode access
 switchport access vlan 30
 switchport voice vlan 40
 no shutdown
exit
```

### Impresora ADMIN

``` cisco
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 30
 no shutdown
exit
```

### Uplink al switch principal

``` cisco
interface gigabitEthernet 0/1
 switchport mode trunk
 no shutdown
exit
```

Hay una PC adicional conectada en ADMIN. El puerto exacto debe
confirmarse en Packet Tracer antes de documentar su configuración
individual. Si está conectada directamente al switch y no detrás de un
teléfono, el puerto debe configurarse como access VLAN 30.

------------------------------------------------------------------------

## 4.5 Access Point inalámbrico

Configuración realizada desde la interfaz gráfica del dispositivo en
Packet Tracer:

  Parámetro               Configuración
  ----------------------- -----------------------------
  Conexión cableada       principal-SW Fa0/2
  VLAN del puerto         10 (GUESTS)
  SSID                    `GUESTS-WIFI`
  Seguridad inalámbrica   Sin configurar; red abierta

Los clientes WiFi reciben su dirección IP desde el pool DHCP de GUESTS.

------------------------------------------------------------------------

## 4.6 Teléfonos IP

Se instalaron seis teléfonos IP: tres en RRHH y tres en ADMIN.

  --------------------------------------------------------------------------
  Departamento   Puerto del    VLAN de datos     VLAN de voz       Extensión
                 switch              para PC                 
  -------------- ----------- --------------- --------------- ---------------
  RRHH           rrhh-sw                  20              40             201
                 Fa0/1                                       

  RRHH           rrhh-sw                  20              40             202
                 Fa0/2                                       

  RRHH           rrhh-sw                  20              40             203
                 Fa0/3                                       

  ADMIN          admin-sw                 30              40             301
                 Fa0/1                                       

  ADMIN          admin-sw                 30              40             302
                 Fa0/2                                       

  ADMIN          admin-sw                 30              40             303
                 Fa0/3                                       
  --------------------------------------------------------------------------

Los teléfonos obtienen sus direcciones por DHCP desde el pool VOICE
(`192.168.40.0/27`). La opción 150 les informa que el servicio CME está
en `192.168.40.1`.

Se comprobó el registro de las extensiones en los teléfonos y se
realizaron llamadas exitosas.

------------------------------------------------------------------------

## 4.7 Impresoras

Las dos impresoras se configuraron para obtener automáticamente su
dirección IP mediante DHCP:

  Impresora   Puerto             VLAN Método IP
  ----------- ---------------- ------ -----------
  RRHH        rrhh-sw Fa0/4        20 DHCP
  ADMIN       admin-sw Fa0/4       30 DHCP

Se comprobó la conectividad haciendo ping desde una PC hacia la
impresora de su departamento.

Al utilizar DHCP, la dirección puede cambiar si no se configura una
reserva. Para una instalación estable se recomienda crear una reserva
DHCP para cada impresora o asignarles IP fija fuera del rango dinámico.

------------------------------------------------------------------------

## 4.8 Computadoras y smartphone

  ------------------------------------------------------------------------
  Equipo           Conexión                          VLAN Asignación IP
  ---------------- ---------------- --------------------- ----------------
  PCs de RRHH      Detrás de                           20 DHCP
                   teléfonos IP o                         
                   directamente al                        
                   switch                                 

  PC adicional     rrhh-sw Fa0/5                       20 DHCP
  RRHH                                                    

  PCs de ADMIN     Detrás de                           30 DHCP
                   teléfonos IP o                         
                   directamente al                        
                   switch                                 

  PC adicional     Puerto por                          30 DHCP
  ADMIN            confirmar                              

  Smartphone de    WiFi                                10 DHCP
  invitados        `GUESTS-WIFI`                          
  ------------------------------------------------------------------------

------------------------------------------------------------------------

# 5. Verificaciones realizadas

## DHCP

Se utilizó:

``` cisco
show ip dhcp binding
```

Se observaron concesiones en las redes RRHH, ADMIN y VOICE. En la salida
comprobada aparecieron:

-   RRHH: `192.168.20.6` a `192.168.20.9`.
-   ADMIN: `192.168.30.6` a `192.168.30.8`.
-   VOICE: `192.168.40.6` a `192.168.40.11`.

Las seis direcciones de VOICE confirmaron que los seis teléfonos habían
obtenido una IP.

## VLAN y trunks

Se utilizaron comandos como:

``` cisco
show vlan brief
show interfaces trunk
show interfaces fa0/1 switchport
show cdp neighbors
```

Se verificó que las VLAN estuvieran creadas y que los enlaces
principales funcionaran como trunk.

## ACL

Se verificó con:

``` cisco
show access-lists
```

La ACL `GUETS-FILTER` mostró las tres reglas de bloqueo y la regla final
de permiso general. También se observó coincidencia de paquetes en la
regla que bloquea el acceso de GUESTS a ADMIN.

## Conectividad

Pruebas realizadas:

-   RRHH hacia ADMIN: comunicación exitosa.
-   GUESTS hacia RRHH: bloqueado.
-   GUESTS hacia ADMIN: bloqueado.
-   GUESTS hacia VOICE: bloqueado según la ACL configurada.
-   GUESTS hacia Internet: funciona.
-   PCs hacia sus impresoras departamentales: ping exitoso.
-   Teléfonos IP: extensiones asignadas y llamadas exitosas entre
    departamentos.

------------------------------------------------------------------------

# 6. Consideraciones y tareas futuras

1.  **Configurar seguridad WiFi:** actualmente `GUESTS-WIFI` no tiene
    contraseña. Se recomienda habilitar WPA2-PSK o el método más seguro
    que soporte el dispositivo.
2.  **Guardar las configuraciones:** ejecutar
    `copy running-config startup-config` en cada router y switch para
    guardar la configuración en NVRAM, si el dispositivo/IOS de Packet
    Tracer lo admite.
3.  **Reservar IP para impresoras:** evitar que las direcciones cambien
    mediante reservas DHCP o direcciones estáticas planificadas.
4.  **Confirmar el puerto de la PC adicional de ADMIN:** completar el
    dato en la tabla de conexiones.
5.  **Documentar la salida a Internet:** el acceso a Internet desde
    GUESTS fue probado, pero los comandos de la interfaz WAN, ruta por
    defecto y NAT no están incluidos porque no quedaron registrados en
    la configuración recopilada.
6.  **Revisar alcance de la ACL:** la ACL controla tráfico originado en
    GUESTS hacia las redes internas indicadas. No constituye por sí sola
    una política de aislamiento en sentido inverso para todas las VLAN.

------------------------------------------------------------------------

## 7. Comandos útiles de diagnóstico

  Comando                              Uso
  ------------------------------------ ---------------------------------------
  `show ip interface brief`            Estado e IP de interfaces del router
  `show running-config`                Configuración activa del dispositivo
  `show ip dhcp binding`               Direcciones entregadas por DHCP
  `show ip dhcp pool`                  Estado de los pools DHCP
  `show access-lists`                  Reglas y contadores de ACL
  `show vlan brief`                    VLAN y puertos de acceso
  `show interfaces trunk`              Estado de enlaces trunk
  `show interfaces fa0/5 switchport`   Modo y VLAN de un puerto
  `show cdp neighbors`                 Dispositivos Cisco vecinos detectados

> Los comandos disponibles pueden variar según el modelo y la versión de
> IOS simulada en Packet Tracer. La documentación refleja las
> configuraciones y pruebas que se pudieron confirmar durante el
> laboratorio.
