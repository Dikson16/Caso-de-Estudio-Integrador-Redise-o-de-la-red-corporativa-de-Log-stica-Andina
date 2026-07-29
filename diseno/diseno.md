# Documento de Diseño — Red Corporativa Logística Andina S.A.S.

**Fase 0 — Diseño de red**
**Equipo:** [completar nombres e identificación de los integrantes]
**Fecha:** [completar]

## 1. Resumen ejecutivo

Este documento presenta el diseño de la nueva red corporativa de Logística Andina S.A.S., que
reemplaza la red plana actual por una arquitectura jerárquica, segmentada y con redundancia en los
puntos críticos. El diseño da respuesta a los cuatro problemas reportados por la gerencia:

- **Continuidad:** doble salida a Internet en Bogotá (ISP-1 primario / ISP-2 de respaldo) con
  conmutación automática, y eliminación del switch de núcleo como punto único de falla mediante
  doble enlace hacia el router de borde.
- **Seguridad:** segmentación por VLAN con aislamiento de invitados y DMZ, control de acceso por
  ACL y endurecimiento de dispositivos (fases 2 y 3).
- **Confidencialidad entre sedes:** túnel GRE protegido con IPsec entre Bogotá y Medellín (Ley 1581
  de 2012).
- **Gestión y trazabilidad:** NTP, syslog, SNMP y respaldo centralizado (fase 7), más control de
  versiones de toda la configuración en Git.

El diseño soporta el crecimiento proyectado del 30% en tres años **sin necesidad de
re-direccionar** ninguna subred (ver `direccionamiento.md`).

## 2. Principios de diseño

| Principio | Cómo se aplica |
|---|---|
| Jerarquía | Bogotá usa un modelo de dos niveles (núcleo/core L3 + acceso); Medellín y Cali, por su tamaño, usan un único router con sub-interfaces (router-on-a-stick) hacia su propio switch de acceso. |
| Redundancia donde se justifica, no en todas partes | Se invierte en redundancia donde el costo del punto único de falla es alto (salida a Internet, enlace núcleo-borde en Bogotá). Se acepta como riesgo residual documentado lo que el propio cliente ya contrató como enlace único (Medellín con un solo ISP local, Cali con un solo enlace dedicado). |
| Mínimo privilegio | Cada segmento existe porque tiene una necesidad de acceso distinta (ver ACL, fase 3). |
| Crecimiento sin re-direccionamiento | Cada subred se dimensionó redondeando hacia arriba desde el requerimiento +30%, dentro de un espacio /16 que deja además un remanente amplio para crecimiento no previsto. |

## 3. Arquitectura general

```
                         Internet (simulada)
                    ┌───────────┬───────────┬────────────┐
                 ISP-1        ISP-2      ISP local
              (respaldo/      (respaldo   (Medellín,
               primario        de ISP-1)   contratado
               Bogotá)                     aparte)
                    │             │             │
              ┌─────┴─────────────┴───┐         │
              │      R-BOG-01         │    túnel GRE + IPsec
              │  (borde + NAT + VPN)  │◄───(sobre Internet)────┐
              └──────┬───────┬────────┘                        │
               doble │       │ WAN dedicado                    │
               enlace│       │ (privado, sin Internet)          │
                     │       └──────────────► R-CALI-01 ── SW-CALI-01 ── PC Usuarios
              ┌──────┴───────┐                                 │
              │  SW-BOG-L3   │                            R-MED-01 ── SW-MED-01 ── PC Usuarios / Operaciones
              │ (core L3,    │
              │ 6 VLANs SVI) │
              └───┬───┬───┬──┘
          ┌────────┘   │   └─────────┐
     SW-BOG-ACC-01  Server DMZ   SW-BOG-ACC-02 + AP invitados
     (Admin + TI)   Server Gestión (Operaciones + Wi-Fi invitados)
```

OSPFv2 de área única (área 0) corre entre **R-BOG-01, SW-BOG-L3, R-MED-01 (a través del túnel) y
R-CALI-01**. Los enlaces hacia los ISP no participan de OSPF: son externos y su ruta se inyecta al
área con `default-information originate` desde R-BOG-01.

## 4. Diseño por sede

### 4.1 Bogotá (sede principal)

| Rol | Equipo propuesto | Justificación |
|---|---|---|
| Borde / NAT / VPN | `R-BOG-01` — ISR con soporte `securityk9` (p. ej. Cisco 4321, o 2911 con licencia de seguridad) | Debe correr NAT, ACL, doble ISP con ruta flotante y el extremo del túnel IPsec; necesita el conjunto de características de seguridad. |
| Núcleo (core L3) | `SW-BOG-L3` — switch multicapa (p. ej. Catalyst 3560) | Concentra el enrutamiento entre las 6 VLANs internas (Admin, Operaciones, TI, Invitados, Gestión, DMZ) sin sobrecargar al router de borde con interfaces físicas que Packet Tracer no siempre tiene disponibles. |
| Acceso | `SW-BOG-ACC-01` (Admin+TI), `SW-BOG-ACC-02` (Operaciones+Invitados) | Separación física de capas de acceso para aplicar port-security de forma independiente por grupo de usuarios. |
| Inalámbrico invitados | `AP-BOG-GUEST` | Requisito del cliente: Wi-Fi de invitados aislada. |
| DMZ | `Server-BOG-DMZ` conectado directo a `SW-BOG-L3` en VLAN dedicada | Portal web público; ver decisión TDD-F0-05 sobre por qué es una VLAN con ACL y no un tercer enlace físico ("three-legged" clásico). |
| Gestión | `Server-BOG-MGMT` (NTP+Syslog+TFTP) conectado a `SW-BOG-L3` | Centraliza los servicios de la fase 7. |

**VLANs de Bogotá:** 10-Administración/Finanzas, 20-Operaciones, 30-TI, 40-Wi-Fi invitados,
50-Gestión, 60-DMZ.

**Redundancia interna:** `R-BOG-01` se conecta a `SW-BOG-L3` con **dos enlaces físicos**
(primario y respaldo). Esto habilita el requisito de la fase 1 de ajustar costo OSPF y demostrar
conmutación de ruta apagando una interfaz, y elimina ese segmento como punto único de falla.

### 4.2 Medellín

| Rol | Equipo | Justificación |
|---|---|---|
| Router de sede | `R-MED-01` — ISR con `securityk9` | Es el otro extremo del túnel GRE+IPsec hacia Bogotá; necesita cifrado. |
| Acceso | `SW-MED-01` | Un solo switch alcanza para dos VLANs (Usuarios, Operaciones) y ~115 hosts proyectados. |

Medellín sale a Internet por **su propio ISP local** (requisito del cliente, sin redundancia
propia) y además establece el túnel GRE/IPsec hacia Bogotá sobre esa misma salida, para que el
tráfico de Operaciones hacia Bogotá viaje cifrado.

### 4.3 Cali

| Rol | Equipo | Justificación |
|---|---|---|
| Router de sede | `R-CALI-01` — ISR sin licencia de seguridad | No termina VPN ni NAT propio; toda su salida a Internet cursa por Bogotá vía el enlace WAN dedicado, así que no necesita `securityk9` (decisión de costo, ver TDD). |
| Acceso | `SW-CALI-01` | Una sola VLAN de usuarios. |

Cali **no tiene salida propia a Internet** (restricción contratada por el cliente, no una decisión
de diseño): todo su tráfico, incluido el de Internet, cruza el enlace dedicado hacia Bogotá y sale
por R-BOG-01.

## 5. Análisis de puntos únicos de falla (SPOF)

| Punto único de falla | ¿Se elimina o se acepta? | Razón |
|---|---|---|
| Salida a Internet de Bogotá (un solo ISP) | **Se elimina** | Doble ISP con ruta flotante (fase 5). |
| Enlace `R-BOG-01` ↔ `SW-BOG-L3` | **Se elimina** | Doble enlace físico con costo OSPF diferenciado. |
| Router `R-BOG-01` en sí mismo | **Se acepta** | Fuera del alcance del curso (requeriría HSRP/VRRP con un segundo router de borde); se documenta como recomendación futura. |
| Switch `SW-BOG-L3` en sí mismo | **Se acepta** | Mismo motivo; mitigado parcialmente por el doble enlace hacia el borde, no por redundancia de switch. |
| Enlace WAN dedicado Bogotá–Cali | **Se acepta** | Es una restricción contratada por el cliente (un solo enlace dedicado, ver diagrama de contexto), no una omisión del equipo. |
| ISP local de Medellín | **Se acepta** | Restricción contratada explícita del cliente ("un solo ISP local"). |

## 6. Referencias

- Tabla de direccionamiento completa: `direccionamiento.md`
- Justificación detallada de cada decisión: `decisiones.md` (Tabla de Decisiones de Diseño)
- Diagrama editable: `diagramas/` (versionar aquí el .drawio/.svg propio del equipo — usar este
  documento como referencia, no como sustituto)
