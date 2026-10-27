# Producto 1 – Red empresarial segmentada (Equipo 5)

**Instituto Tecnológico de las Américas (ITLA)** – Conmutación y Enrutamiento (TI-203)
**Profesor:** Onel Luis Pelegrino
**Plataforma:** PNETLab
**Bloque asignado:** `172.24.0.0/15` | `2001:25:24::/48`
| Link Video | `https://www.youtube.com/playlist?list=PLGnmmpX8-gZ8`

Red confiable, segura y escalable para una mediana empresa con segmentación por
departamentos (VLANs), VTP, EtherChannel, STP rápido, HSRP, OSPF multiárea y ACLs.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `README.md` | Documentación del proyecto (este archivo) |
| `Equipo5_Configuraciones.txt` | Configuración de todos los dispositivos, hosts, ACLs y comandos de verificación |

---

## 1. Topología

Dispositivos: **R1-Core**, **R2**, **ISP** (nube), **SWM1/SWM2** (distribución, capa 2),
**SWAC1/SWAC2/SWAC3** (acceso, capa 2), **SWA13** y **SW** (sitio R2, capa 2),
hosts **DOC1, EST1, GES1, VPC11, VPC12** y el servidor **SERVICIOS**.

| Origen | Puerto | Destino | Puerto | Tipo |
|---|---|---|---|---|
| R1-Core | e0/0 | SWM1 | e0/0 | Trunk (VLAN 10, 30) |
| R1-Core | e0/1 | SWM2 | e0/0 | Trunk (VLAN 20, 40) |
| R1-Core | e0/2 | ISP | e0/0 | Routed (salida a Internet) |
| R1-Core | s1/0 | R2 | s1/0 | P2P serial |
| SWM1 | e0/1-3 | SWM2 | e0/1-3 | EtherChannel Po1 (LACP), trunk |
| SWM1 | e1/0 | SWAC1 | e0/0 | Trunk |
| SWM1 | e1/1 | SWAC2 | e0/0 | Trunk |
| SWM1 | e1/2 | SWAC3 | e0/0 | Trunk |
| SWM2 | e1/0 | SWAC1 | e0/1 | Trunk |
| SWM2 | e1/1 | SWAC2 | e0/1 | Trunk |
| SWM2 | e1/2 | SWAC3 | e0/1 | Trunk |
| SWAC1 | e1/0 | DOC1 | eth0 | Acceso VLAN 20 |
| SWAC2 | e2/0 | EST1 | eth0 | Acceso VLAN 10 |
| SWAC3 | e3/0 | GES1 | eth0 | Acceso VLAN 30 |
| R2 | e0/0 | SWA13 | e0/0 | Trunk (VLAN 50, 60) |
| SWA13 | e1/0 | VPC11 | eth0 | Acceso VLAN 50 |
| SWA13 | e2/0 | VPC12 | eth0 | Acceso VLAN 60 |
| R2 | e0/1 | SW | e0/0 | Routed en R2 / acceso VLAN 70 en SW |
| SW | e0/1 | SERVICIOS | eth0 | Acceso VLAN 70 |

---

## 2. Plan de direccionamiento

### 2.1 IPv4 (VLSM, de mayor a menor)

| Red | Dominio | VLAN | Hosts | Máscara | Subred | Rango útil | Broadcast | Gateway (VIP) |
|---|---|---|---|---|---|---|---|---|
| Estudiantes | R1 | 10 | 1500 | /21 | 172.24.0.0 | .0.1 – .7.254 | 172.24.7.255 | 172.24.0.1 |
| Docentes | R1 | 20 | 164 | /24 | 172.24.8.0 | .8.1 – .8.254 | 172.24.8.255 | 172.24.8.1 |
| Finanzas | R2 | 50 | 64 | /25 | 172.24.9.0 | .9.1 – .9.126 | 172.24.9.127 | 172.24.9.1 |
| Académicos | R2 | 60 | 80 | /25 | 172.24.9.128 | .9.129 – .9.254 | 172.24.9.255 | 172.24.9.129 |
| Gestión | R1 | 30 | 29 | /26 | 172.24.10.0 | .10.1 – .10.62 | 172.24.10.63 | 172.24.10.1 |
| Datacenter | R2 | 70 | 44 | /26 | 172.24.10.64 | .10.65 – .10.126 | 172.24.10.127 | 172.24.10.65 |
| Voz | R1 | 40 | 16 | /27 | 172.24.10.128 | .10.129 – .10.158 | 172.24.10.159 | 172.24.10.129 |
| Blackhull (R1) | R1 | 999 | 10 | /28 | 172.24.10.160 | .10.161 – .10.174 | 172.24.10.175 | sin gateway |
| Blackhull (R2) | R2 | 999 | 10 | /28 | 172.24.10.176 | .10.177 – .10.190 | 172.24.10.191 | sin gateway |
| P2P R1–R2 | – | – | 2 | /30 | 172.24.10.192 | .10.193 – .10.194 | 172.24.10.195 | – |
| R1–ISP | – | – | 2 | /30 | 172.24.10.196 | .10.197 – .10.198 | 172.24.10.199 | – |

### 2.2 IPv6 (`2001:25:24::/48`, un /64 por red en el mismo orden de tamaño que IPv4)

| Red | Prefijo | Gateway | Modo de asignación |
|---|---|---|---|
| Estudiantes | 2001:25:24:0::/64 | ::1 (VIP HSRP) | SLAAC |
| Docentes | 2001:25:24:1::/64 | ::1 (VIP HSRP) | SLAAC |
| Finanzas | 2001:25:24:2::/64 | ::1 (R2) | DHCPv6 stateless |
| Académicos | 2001:25:24:3::/64 | ::1 (R2) | DHCPv6 stateless |
| Gestión | 2001:25:24:4::/64 | ::1 (VIP HSRP) | SLAAC |
| Datacenter | 2001:25:24:5::/64 | ::1 (R2) | Estático (servidor ::10) |
| Voz | 2001:25:24:6::/64 | ::1 (VIP HSRP) | SLAAC |
| Blackhull R1 | 2001:25:24:7::/64 | – | Reservado |
| Blackhull R2 | 2001:25:24:8::/64 | – | Reservado |
| P2P R1–R2 | 2001:25:24:9::/126 | R1 ::1 / R2 ::2 | Estático |
| R1–ISP | 2001:25:24:A::/64 | R1 ::1 / ISP ::2 | Estático |

### 2.3 Direcciones de interfaces y dispositivos

| Dispositivo | Interfaz | IPv4 | IPv6 |
|---|---|---|---|
| R1-Core | e0/0.10 (Estudiantes) | 172.24.0.2/21 (VIP .1) | 2001:25:24:0::2/64 |
| R1-Core | e0/0.30 (Gestión) | 172.24.10.2/26 (VIP .1) | 2001:25:24:4::2/64 |
| R1-Core | e0/1.20 (Docentes) | 172.24.8.2/24 (VIP .1) | 2001:25:24:1::2/64 |
| R1-Core | e0/1.40 (Voz) | 172.24.10.130/27 (VIP .129) | 2001:25:24:6::2/64 |
| R1-Core | e0/2 (ISP) | 172.24.10.197/30 | 2001:25:24:A::1/64 |
| R1-Core | s1/0 (P2P) | 172.24.10.193/30 | 2001:25:24:9::1/126 |
| R2 | e0/0.50 (Finanzas) | 172.24.9.1/25 | 2001:25:24:2::1/64 |
| R2 | e0/0.60 (Académicos) | 172.24.9.129/25 | 2001:25:24:3::1/64 |
| R2 | e0/1 (Datacenter) | 172.24.10.65/26 | 2001:25:24:5::1/64 |
| R2 | s1/0 (P2P) | 172.24.10.194/30 | 2001:25:24:9::2/126 |
| SWM1 | Vlan 30 (gestión) | 172.24.10.4/26 | – |
| SWM2 | Vlan 30 (gestión) | 172.24.10.5/26 | – |
| SWAC1 | Vlan 30 (gestión) | 172.24.10.11/26 | – |
| SWAC2 | Vlan 30 (gestión) | 172.24.10.12/26 | – |
| SWAC3 | Vlan 30 (gestión) | 172.24.10.13/26 | – |
| DOC1 | eth0 | 172.24.8.10/24 | 2001:25:24:1::10/64 |
| EST1 | eth0 | 172.24.0.10/21 | 2001:25:24:0::10/64 |
| GES1 | eth0 | 172.24.10.10/26 | 2001:25:24:4::10/64 |
| VPC11 | eth0 | 172.24.9.10/25 | 2001:25:24:2::10/64 |
| VPC12 | eth0 | 172.24.9.138/25 | 2001:25:24:3::10/64 |
| SERVICIOS | eth0 | 172.24.10.70/26 | 2001:25:24:5::10/64 |

---

## 3. VLANs y VTP

| Dominio VTP | VLAN | Nombre | Red |
|---|---|---|---|
| ITLA-E5-R1 | 10 | Estudiantes | 172.24.0.0/21 |
| ITLA-E5-R1 | 20 | Docentes | 172.24.8.0/24 |
| ITLA-E5-R1 | 30 | Gestion | 172.24.10.0/26 |
| ITLA-E5-R1 | 40 | Voz | 172.24.10.128/27 |
| ITLA-E5-R1 | 999 | Blackhull | 172.24.10.160/28 |
| ITLA-E5-R2 | 50 | Finanzas | 172.24.9.0/25 |
| ITLA-E5-R2 | 60 | Academicos | 172.24.9.128/25 |
| ITLA-E5-R2 | 70 | Datacenter | 172.24.10.64/26 (local en SW) |
| ITLA-E5-R2 | 999 | Blackhull | 172.24.10.176/28 |

| Dominio | Switch | Modo VTP |
|---|---|---|
| ITLA-E5-R1 | SWM1 | Server |
| ITLA-E5-R1 | SWM2 | Server (respaldo) |
| ITLA-E5-R1 | SWAC1, SWAC2, SWAC3 | Client |
| ITLA-E5-R2 | SWA13 | Server |
| ITLA-E5-R2 | SW | Transparent (solo conecta con R2, sin trunk hacia SWA13) |

VTP versión 2 con contraseña. Blackhull es la VLAN de los puertos sin uso: apagados y sin gateway.

---

## 4. Procedimientos realizados

### Sprint 1 – Diseño y plan básico (HU1, HU2, HU3)
1. Se tomó el bloque asignado al equipo 5 (`172.24.0.0/15` y `2001:25:24::/48`).
2. Se ordenaron las redes de mayor a menor número de hosts y se aplicó **VLSM**, con la
   máscara mínima que cubre hosts + direcciones reservadas (red, broadcast y, donde aplica, gateways HSRP).
3. Se asignó un `/64` IPv6 por red siguiendo el mismo orden de tamaño que IPv4
   (la red de 1500 hosts es la primera).
4. Se definieron las VLANs por departamento, los dos dominios VTP (R1 y R2) y los modos
   Server/Client/Transparent.
5. Se elaboró el mapa de puertos y la topología para PNETLab (secciones 1 a 3).

### Sprint 2 – Implementación L2/L3 y alta disponibilidad (HU2, HU3, HU4)
1. **Configuración base** en todos los equipos: hostname, usuario administrador, `enable secret`,
   cifrado de contraseñas, SSH y `logging synchronous`.
2. **VLANs y VTP:** SWM1 crea las VLANs como servidor; SWM2 y los SWAC las reciben.
   SWA13 es servidor del dominio R2 y SW usa modo transparente con la VLAN 70 local.
3. **EtherChannel:** SWM1 y SWM2 se unen con `Port-channel1` (3 puertos, LACP `active`) en modo trunk.
4. **Trunks:** SWM1 y SWM2 se enlazan a cada SWAC con VLANs permitidas 10, 20, 30 y 40.
5. **STP:** modo `rapid-pvst`. SWM1 es root de las VLAN 10 y 30 y SWM2 de las VLAN 20 y 40,
   con el otro como root secundario. Los puertos de host llevan `portfast` y `bpduguard`.
6. **Inter-VLAN routing:** como los switches son de capa 2, los gateways están en R1-Core con
   subinterfaces. El enlace e0/0 (hacia SWM1) lleva las VLAN 10 y 30 y el e0/1 (hacia SWM2)
   las VLAN 20 y 40, repartiendo la carga y alineando cada enlace con el root STP.
   R2 usa subinterfaces `e0/0.50` y `e0/0.60` y `e0/1` para el Datacenter.
7. **HSRP (v2):** configurado en cada subinterfaz de R1-Core (VIP `.1`, real `.2`) para IPv4 e IPv6.
   **Limitación:** R1-Core es el único router de su dominio, por lo que no hay un par standby
   y el failover de gateway no se puede demostrar. Se requiere un segundo router (R1-Core-B).
8. **Seguridad de puertos:** los puertos sin uso se apagan y se mueven a la VLAN 999.
9. **Pruebas de redundancia L2:** caída de un puerto del EtherChannel y de un trunk hacia
   un SWAC, midiendo la pérdida de paquetes con un ping continuo.

### Sprint 3 – Enrutamiento, seguridad y pruebas (HU5, HU6, HU7)
1. **OSPFv2:** `router-id` por router y `passive-interface default`.
   - **Área 0:** enlace P2P R1–R2 y las redes del campus (VLAN 10, 20, 30 y 40) en R1-Core.
   - **Área 1 (stub):** LAN de R2 (Finanzas, Académicos y Datacenter). R2 es el ABR.
   - R1-Core origina la ruta por defecto (`default-information originate`) hacia el ISP.
2. **OSPFv3** equivalente para IPv6 (mismas áreas, área 1 stub).
3. **Salida a Internet:** ruta estática por defecto en R1-Core y NAT overload por e0/2.
   La VLAN de Voz queda excluida del NAT (sin acceso a Internet).
4. **ACLs:**

| ACL | Tipo | Dónde | Regla principal |
|---|---|---|---|
| `ACL-EST-IN` | Extendida | R1 e0/0.10 (in) | Estudiantes sin acceso a Gestión, Finanzas, Docentes ni Voz |
| `ACL-DOC-IN` | Extendida | R1 e0/1.20 (in) | Docentes sin acceso a Finanzas ni Gestión |
| `ACL-VOZ-IN` | Extendida | R1 e0/1.40 (in) | Voz solo con Voz y Datacenter |
| `ACL-FIN-IN` | Extendida | R2 e0/0.50 (in) | Finanzas sin acceso a Estudiantes, Docentes ni Gestión |
| `ACL-ACA-IN` | Extendida | R2 e0/0.60 (in) | Académicos sin acceso a Gestión |
| `ACL-DC-OUT` | Extendida | R2 e0/1 (out) | Estudiantes al Datacenter solo con ICMP echo, HTTP y HTTPS |
| `NAT-INET` | Estándar | R1 NAT | Excluye Voz de Internet |
| `10` | Estándar | VTY | Administración remota solo desde Gestión (172.24.10.0/26) |

5. **Pruebas funcionales y de seguridad** (resultado esperado):

| Origen | Destino | Resultado |
|---|---|---|
| GES1 | Todas las redes | Permitido |
| DOC1 | VPC12, SERVICIOS | Permitido |
| DOC1 | VPC11, GES1 | Bloqueado |
| EST1 | SERVICIOS (ping, 80, 443) | Permitido |
| EST1 | GES1, VPC11, DOC1 | Bloqueado |
| VPC11 | SERVICIOS | Permitido |
| VPC11 | EST1, DOC1, GES1 | Bloqueado |
| VPC12 | GES1 | Bloqueado |

---

## 5. Orden de despliegue en PNETLab

1. Aplicar la configuración en **SWM1 y SWM2**, luego en **SWAC1-3, SWA13 y SW**.
2. Aplicar la configuración en **R1-Core y R2**.
3. Esperar la sincronización de VTP (`show vlan brief` en los SWAC).
4. Configurar las IP de los hosts (VPCS).
5. Validar conectividad y OSPF, y **al final** aplicar la sección de ACLs.
6. Guardar con `write memory` en todos los dispositivos.

## 6. Comandos de verificación

```text
show vtp status | show vlan brief | show interfaces trunk
show etherchannel summary | show spanning-tree summary
show standby brief
show ip ospf neighbor | show ip route ospf | show ip ospf database
show ipv6 ospf neighbor | show ipv6 route ospf
show access-lists | show ip interface e0/0.10
ping / traceroute entre hosts según la tabla de pruebas
```

## 7. Notas

- El next-hop del ISP (`172.24.10.198`) es una suposición del plan; debe ajustarse si el
  ISP entrega otra dirección en PNETLab.
- Credenciales de laboratorio: usuario `admin` / contraseña `ITLA-E5`. Cámbialas en un entorno real.

## Equipo 5

Integrantes: _(completar)_
