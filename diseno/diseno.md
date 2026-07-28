# Diseño de la red — Logística Andina S.A.S.

## 1. Resumen ejecutivo

Se propone una red jerárquica de dos a tres niveles por sede, con Bogotá como
sede principal (borde + core + distribución + acceso), y Medellín y Cali como
sedes remotas con un único router que combina las funciones de borde y core.
La conectividad a Internet de Bogotá es redundante (doble ISP con
conmutación automática); Cali no tiene salida propia y depende de un enlace
WAN dedicado a Bogotá; Medellín tiene su propio ISP local y además un túnel
cifrado (GRE sobre IPsec) hacia Bogotá para el tráfico de Operaciones. El
direccionamiento interno usa VLSM sobre 172.16.0.0/16 con reserva de
crecimiento del 40 % por segmento.

## 2. Topología lógica

```
                         Internet (ISP-1 / ISP-2)
                                   |
                            R-BOG-EDGE
                        (borde: doble ISP, NAT)
                                   |
                            R-BOG-CORE
                       (core: OSPF, distribución) -------- WAN dedicado -------- R-CALI-01
                                   |                                                  |
                            SW-BOG-DIST                                        SW-CALI-ACC-1
                          (L3, VLANs internas)                                   VLAN Usuarios
                    /       |        |        \
             LAN interna  Red WiFi   DMZ     Gestión
          (Admin·Ops·TI) (invitados) (portal) (NTP·Syslog·SNMP)

   R-BOG-EDGE ===== Túnel GRE + IPsec (sobre Internet) =====> R-MED-01
                                                          (borde+core Medellín)
                                                                  |
                                                            SW-MED-DIST
                                                           /            \
                                                  Usuarios              Operaciones
```

## 3. Jerarquía elegida y justificación

| Nivel | Función | Equipo |
|---|---|---|
| Borde | Salida a Internet, NAT, terminación de VPN | R-BOG-EDGE, R-MED-01 |
| Core | Enrutamiento OSPF corporativo, enlaces WAN internos | R-BOG-CORE, R-CALI-01 |
| Distribución | Inter-VLAN routing, agregación de acceso | SW-BOG-DIST, SW-MED-DIST |
| Acceso | Conexión de hosts, port-security | Switches de 48 y 24 puertos, APs |

**Por qué separar borde y core en Bogotá:** aísla el NAT y la terminación
de la VPN (funciones que cambian con la conmutación de ISP) del
enrutamiento OSPF interno, que debe permanecer estable. También facilita
decidir dónde va cada ACL: las de acceso a Internet y a la VPN se aplican
en el borde; las de control entre segmentos internos, en el core o en la
distribución.

**Por qué Medellín y Cali no replican esa separación:** el volumen de
tráfico y el número de segmentos no lo justifica (2 y 1 VLAN
respectivamente); un router adicional sería un costo sin beneficio de
diseño, y se documenta como decisión consciente, no como omisión.

## 4. Puntos únicos de falla

| Punto único de falla | ¿Se elimina? | Justificación |
|---|---|---|
| Salida a Internet de Bogotá | Sí — doble ISP con ruta flotante | Requisito no negociable del cliente (continuidad) |
| R-BOG-EDGE | No | Un segundo router de borde no está dentro del alcance/presupuesto del caso; se documenta como riesgo aceptado |
| Enlace Bogotá–Cali | No | El cliente contrató un único enlace dedicado; no hay respaldo previsto en el alcance |
| ISP local de Medellín | No | Fuera de los requisitos no negociables (solo se exige continuidad para Bogotá y Cali) |
| SW-BOG-DIST | No | Un solo switch de distribución es aceptable en este tamaño de red; se documenta como candidato a mejora futura (stack o HSRP) |

## 5. Tabla de direccionamiento VLSM (crecimiento 40 %)

Bloque base: `172.16.0.0/16`. Bloques públicos: ISP-1 `209.165.200.224/27`,
ISP-2 `209.165.201.0/27`.

| Segmento | Hosts hoy | Hosts +40% | Subred | Máscara | Rango utilizable | Gateway |
|---|---|---|---|---|---|---|
| Bogotá – WiFi invitados | 100 | 140 | 172.16.0.0/24 | 255.255.255.0 | .1 – .254 | 172.16.0.1 |
| Bogotá – Admin/Finanzas | 120 | 168 | 172.16.1.0/24 | 255.255.255.0 | .1 – .254 | 172.16.1.1 |
| Bogotá – Operaciones/Rastreo | 60 | 84 | 172.16.2.0/25 | 255.255.255.128 | .1 – .126 | 172.16.2.1 |
| Medellín – Usuarios | 60 | 84 | 172.16.2.128/25 | 255.255.255.128 | .129 – .254 | 172.16.2.129 |
| Bogotá – TI | 30 | 42 | 172.16.3.0/26 | 255.255.255.192 | .1 – .62 | 172.16.3.1 |
| Medellín – Operaciones | 28 | 40 | 172.16.3.64/26 | 255.255.255.192 | .65 – .126 | 172.16.3.65 |
| Cali – Usuarios | 25 | 35 | 172.16.3.128/26 | 255.255.255.192 | .129 – .190 | 172.16.3.129 |
| Bogotá – Gestión | 10 | 14 | 172.16.3.192/27 | 255.255.255.224 | .193 – .222 | 172.16.3.193 |
| Bogotá – DMZ | 6 | 9 | 172.16.3.224/28 | 255.255.255.240 | .225 – .238 | 172.16.3.225 |
| Enlace R-BOG-EDGE ↔ R-BOG-CORE | — | — | 172.16.3.240/30 | 255.255.255.252 | .241 – .242 | — |
| Enlace R-BOG-CORE ↔ R-CALI-01 | — | — | 172.16.3.244/30 | 255.255.255.252 | .245 – .246 | — |
| Túnel GRE R-BOG-EDGE ↔ R-MED-01 | — | — | 172.16.3.248/30 | 255.255.255.252 | .249 – .250 | — |
| Reservado para crecimiento | — | — | 172.16.4.0/22 en adelante | — | — | — |

## 6. Inventario de equipos (Fase 0 )

### Sede Bogotá

| Equipo | Modelo Packet Tracer | Cantidad | Notas |
|---|---|---|---|
| R-BOG-EDGE | 4321 (con módulo/licencia `securityk9`) | 1 | Doble ISP, NAT, terminación de túnel |
| R-BOG-CORE | 2911 | 1 | OSPF, no requiere `securityk9` |
| SW-BOG-DIST | 3560-24PS | 1 | Switch L3, SVIs por VLAN |
| SW-BOG-ACC-ADMIN-1..4 | 2960-48TT | 4 | ~42 hosts c/u, VLAN Admin/Finanzas |
| SW-BOG-ACC-OPS-1..2 | 2960-48TT | 2 | VLAN Operaciones/Rastreo |
| SW-BOG-ACC-TI-1 | 2960-48TT | 1 | VLAN TI |
| SW-BOG-ACC-GUEST-1 | 2960-24TT | 1 | Uplink de los 3 APs |
| AP-BOG-GUEST-1..3 | Access Point-PT-A | 3 | VLAN WiFi invitados |
| SW-BOG-GEST-1 | 2960-24TT | 1 | VLAN Gestión |
| SW-BOG-DMZ-1 | 2960-24TT | 1 | Cuelga de R-BOG-CORE, no de la distribución |
| Servidor NTP/Syslog/TFTP | Server-PT | 1 | Puede combinar los tres servicios |
| Estación SNMP (MIB Browser) | PC-PT | 1 | Segmento Gestión |
| Servidor portal web | Server-PT | 1 | Segmento DMZ, NAT estática |
| PCs de usuario | PC-PT / Laptop-PT | según sustentación | No es necesario poblar cada puerto; con 3–5 PC por VLAN alcanza para verificar |

### Sede Medellín

| Equipo | Modelo Packet Tracer | Cantidad | Notas |
|---|---|---|---|
| R-MED-01 | 4321 (con `securityk9`) | 1 | Borde + core, ISP local + túnel |
| SW-MED-DIST | 2960-24TT | 1 | Agregación hacia el router |
| SW-MED-ACC-USR-1..2 | 2960-48TT | 2 | VLAN Usuarios |
| SW-MED-ACC-OPS-1 | 2960-48TT | 1 | VLAN Operaciones |

### Sede Cali

| Equipo | Modelo Packet Tracer | Cantidad | Notas |
|---|---|---|---|
| R-CALI-01 | 2911 | 1 | Solo enlace WAN dedicado + LAN |
| SW-CALI-ACC-1 | 2960-48TT | 1 | VLAN única de Usuarios |

### Internet simulada

| Elemento | Cómo se simula |
|---|---|
| ISP-1 / ISP-2 | Un router genérico (o `Cloud-PT`) por ISP, |
| Host de prueba en Internet | 1 PC-PT fuera de las nubes ISP, para probar el acceso al portal web publicado |
