# Tabla de Direccionamiento — VLSM

## 1. Criterio de asignación

Se subnetea el bloque privado **172.16.0.0/16** asignado por el cliente. Orden de asignación:
de mayor a menor tamaño de subred (práctica estándar de VLSM para minimizar fragmentación),
agrupando por sede para mantener bloques contiguos y facilitar la lectura del plan.

Cada subred se dimensionó así: **hosts actuales × 1.3 (crecimiento a 3 años)**, redondeado hacia
arriba a la potencia de 2 más cercana que lo soporte. El bloque /16 completo (65 536 direcciones)
es mucho mayor a lo que la organización necesita hoy (~400 hosts); se aprovecha ese margen para
dejar un remanente amplio (172.16.5.0 – 172.16.254.255 completo, más varios /26 y /30 sueltos)
para crecimiento no anticipado, sin sacrificar claridad por una eficiencia de espacio innecesaria
a este tamaño de organización. Este es un punto que se defiende en la TDD (TDD-F0-01).

## 2. Segmentos LAN

| Sede | Segmento | VLAN | Subred | Máscara | Gateway | Rango utilizable | Hosts hoy | Hosts +30% | Hosts soportados |
|---|---|---|---|---|---|---|---|---|---|
| Bogotá | Wi-Fi invitados | 40 | 172.16.0.0/24 | 255.255.255.0 | 172.16.0.1 | .1 – .254 | 100 | 130 | 254 |
| Bogotá | Administración y Finanzas | 10 | 172.16.1.0/24 | 255.255.255.0 | 172.16.1.1 | .1 – .254 | 120 | 156 | 254 |
| Medellín | Usuarios | 10 | 172.16.2.0/25 | 255.255.255.128 | 172.16.2.1 | .1 – .126 | 60 | 78 | 126 |
| Bogotá | Operaciones y rastreo de flota | 20 | 172.16.2.128/25 | 255.255.255.128 | 172.16.2.129 | .129 – .254 | 60 | 78 | 126 |
| Medellín | Operaciones | 20 | 172.16.3.0/26 | 255.255.255.192 | 172.16.3.1 | .1 – .62 | 28 | 37 | 62 |
| Cali | Usuarios | 10 | 172.16.3.64/26 | 255.255.255.192 | 172.16.3.65 | .65 – .126 | 25 | 33 | 62 |
| Bogotá | TI | 30 | 172.16.3.128/26 | 255.255.255.192 | 172.16.3.129 | .129 – .190 | 30 | 39 | 62 |
| — | *(reserva libre)* | — | 172.16.3.192/26 | 255.255.255.192 | — | .193 – .254 | — | — | libre |
| Bogotá | DMZ (portal web) | 60 | 172.16.4.0/28 | 255.255.255.240 | 172.16.4.1 | .1 – .14 | 6 | 8 | 14 |
| Bogotá | Gestión (NTP/syslog/TFTP/SNMP) | 50 | 172.16.4.16/28 | 255.255.255.240 | 172.16.4.17 | .17 – .30 | 10 | 13 | 14 |

## 3. Enlaces punto a punto (/30)

| Enlace | Subred | Extremo A | Extremo B |
|---|---|---|---|
| R-BOG-01 ↔ R-CALI-01 (WAN dedicado) | 172.16.4.32/30 | R-BOG-01: .33 | R-CALI-01: .34 |
| Tunnel0 GRE Bogotá ↔ Medellín | 172.16.4.36/30 | R-BOG-01: .37 | R-MED-01: .38 |
| R-BOG-01 ↔ SW-BOG-L3 (enlace primario) | 172.16.4.40/30 | R-BOG-01: .41 | SW-BOG-L3: .42 |
| R-BOG-01 ↔ SW-BOG-L3 (enlace de respaldo) | 172.16.4.44/30 | R-BOG-01: .45 | SW-BOG-L3: .46 |

*(172.16.4.48 en adelante queda libre dentro de este bloque para futuros enlaces punto a punto.)*

## 4. Loopbacks (router-id estable)

| Equipo | Loopback0 |
|---|---|
| R-BOG-01 | 172.16.255.1/32 |
| R-MED-01 | 172.16.255.2/32 |
| R-CALI-01 | 172.16.255.3/32 |
| SW-BOG-L3 | 172.16.255.4/32 |

## 5. Direccionamiento público

| Recurso | Bloque / subred | Uso |
|---|---|---|
| Enlace R-BOG-01 ↔ ISP-1 | 209.165.200.224/30 (dentro de 209.165.200.224/27) | ISP-1: .225, R-BOG-01: .226 |
| Resto del bloque ISP-1 | 209.165.200.228 – .254 | PAT por *overload* sobre la interfaz de salida (.226) y NAT estática del portal DMZ en **209.165.200.230** (se configura en fase 4) |
| Enlace R-BOG-01 ↔ ISP-2 | 209.165.201.0/30 (dentro de 209.165.201.0/27) | ISP-2: .1, R-BOG-01: .2 |
| Resto del bloque ISP-2 | 209.165.201.4 – .30 | Reservado para PAT de contingencia si conmuta el ISP primario (decisión a resolver en fase 4/5) |
| Enlace R-MED-01 ↔ ISP local de Medellín | 198.51.100.0/30 | ISP-MED: .1, R-MED-01: .2 — bloque de documentación (RFC 5737 TEST-NET-2), usado porque el cliente no asignó bloque público para esa sede; queda documentado como simulación de laboratorio (TDD-F0-06) |

## 6. Verificación de la restricción de crecimiento

Todas las subredes LAN soportan el requerimiento a 3 años (+30%) sin necesidad de ampliar la
máscara: en cada fila, "Hosts +30%" ≤ "Hosts soportados". Adicionalmente, el bloque /16 conserva
un tramo completo sin asignar (172.16.5.0/24 en adelante) para escenarios de crecimiento no
previstos por el cliente (nueva sede, nuevo servicio).

## 7. Direcciones de gestión de switches de acceso (ajuste de Fase 2)

La Fase 0 solo reservó un segmento de Gestión dedicado en Bogotá (VLAN 50). Al llegar a la Fase 2
—cuando todos los switches necesitan una IP para ser administrados por SSH— se detectó que
Medellín y Cali no tenían un segmento equivalente. Se documenta aquí el ajuste (ver TDD-F2 para la
justificación):

| Equipo | IP de gestión | Subred que la contiene | Gateway |
|---|---|---|---|
| SW-BOG-ACC-01 | 172.16.4.19/28 | VLAN 50 – Gestión (Bogotá), extendida por trunk | 172.16.4.17 |
| SW-BOG-ACC-02 | 172.16.4.20/28 | VLAN 50 – Gestión (Bogotá), extendida por trunk | 172.16.4.17 |
| SW-MED-01 | 172.16.3.2/26 | VLAN 20 – Operaciones (Medellín) | 172.16.3.1 |
| SW-CALI-01 | 172.16.3.66/26 | VLAN 10 – Usuarios (Cali) | 172.16.3.65 |

**Limitación reconocida:** en Medellín y Cali, la IP de gestión del switch comparte el dominio de
broadcast con tráfico de usuario real, en vez de vivir en un segmento de gestión dedicado como en
Bogotá. Se acepta para el alcance del curso; la ACL `TI-ADMIN-ONLY` en las VTY sigue siendo el
control real que decide quién puede administrar, independientemente de en qué VLAN viva la IP.
Mejora recomendada a futuro: una VLAN de gestión dedicada por sede.

## 8. Servidores (Fase 7)

| Servidor | IP | Subred |
|---|---|---|
| Server-BOG-DMZ (portal web) | 172.16.4.2/28 | VLAN 60 – DMZ |
| Server-BOG-MGMT (NTP/Syslog/TFTP/estación SNMP) | 172.16.4.18/28 | VLAN 50 – Gestión |


