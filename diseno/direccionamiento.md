# Plan de Direccionamiento IP

Este documento consolida el plan de direccionamiento IP para las sedes y segmentos de red definidos, incluyendo la cantidad de hosts actuales, la proyección de crecimiento del 40%, la subred asignada, máscara, gateway y rango utilizable.

## Tabla de subredes

| Sede     | Segmento                      | Hosts Actuales | Hosts Proyectados (+40%) | Subred Asignada | Máscara         | Gateway        | Rango Utilizable            |
| -------- | ------------------------------ | --------------- | ------------------------- | ----------------- | ---------------- | -------------- | ---------------------------- |
| Bogotá   | Invitados (WiFi)               | 100              | 140                        | 172.16.0.0/24      | 255.255.255.0     | 172.16.0.1      | 172.16.0.1 - 172.16.0.254    |
| Bogotá   | Administración/Finanzas        | 120              | 168                        | 172.16.1.0/24      | 255.255.255.0     | 172.16.1.1      | 172.16.1.1 - 172.16.1.254    |
| Bogotá   | Operaciones/Rastreo             | 60               | 84                         | 172.16.2.0/25      | 255.255.255.128   | 172.16.2.1      | 172.16.2.1 - 172.16.2.126    |
| Medellín | Usuarios                       | 60               | 84                         | 172.16.2.128/25    | 255.255.255.128   | 172.16.2.129    | 172.16.2.129 - 172.16.2.254  |
| Bogotá   | TI                              | 30               | 42                         | 172.16.3.0/26      | 255.255.255.192   | 172.16.3.1      | 172.16.3.1 - 172.16.3.62     |
| Medellín | Operaciones                    | 28               | 40                         | 172.16.3.64/26     | 255.255.255.192   | 172.16.3.65     | 172.16.3.65 - 172.16.3.126   |
| Cali     | Usuarios                       | 25               | 35                         | 172.16.3.128/26    | 255.255.255.192   | 172.16.3.129    | 172.16.3.129 - 172.16.3.190  |
| Bogotá   | Gestión                        | 10               | 14                         | 172.16.3.192/27    | 255.255.255.224   | 172.16.3.193    | 172.16.3.193 - 172.16.3.222  |
| Bogotá   | DMZ                             | 6                | 9                          | 172.16.3.224/28    | 255.255.255.240   | 172.16.3.225    | 172.16.3.225 - 172.16.3.238  |
| WAN      | Enlace R-BOG-EDGE ↔ R-BOG-CORE  | 2                | 2                          | 172.16.3.240/30    | 255.255.255.252   | —               | 172.16.3.241 - 172.16.3.242  |
| WAN      | Enlace Dedicado BOG-CALI        | 2                | 2                          | 172.16.3.244/30    | 255.255.255.252   | —               | 172.16.3.245 - 172.16.3.246  |
| WAN      | Túnel GRE BOG-MED (+ IPsec)     | 2                | 2                          | 172.16.3.248/30    | 255.255.255.252   | —               | 172.16.3.249 - 172.16.3.250  |

Rango reservado sin asignar para crecimiento futuro: `172.16.4.0/22` en adelante.

## Direccionamiento público

| Recurso | Bloque | Uso |
|---|---|---|
| ISP-1 (primario) | 209.165.200.224/27 | Interfaz de R-BOG-EDGE, pool NAT/PAT, NAT estática del portal DMZ |
| ISP-2 (respaldo) | 209.165.201.0/27 | Interfaz de respaldo de R-BOG-EDGE, activa solo tras conmutación de ruta flotante |
| ISP local Medellín | Fuera del control de direccionamiento del cliente | Se simula en Packet Tracer con un bloque arbitrario, documentado como decisión fuera de alcance |

## Resumen técnico

- La red base utilizada es `172.16.0.0/16`.
- Las subredes fueron asignadas considerando crecimiento proyectado del 40%.
- Los segmentos con mayor demanda de hosts utilizan máscaras `/24` y `/25`.
- Gestión pasó de `/28` a `/27`: con 40% de crecimiento el `/28` deja exactamente 14 direcciones utilizables para 14 hosts proyectados, es decir, cero margen real; se prefirió `/27` para dejar holgura genuina.
- Los enlaces WAN punto a punto (`/30`) se ubicaron de forma contigua al resto del direccionamiento interno, en lugar de aislarlos en otro bloque de la /16, para mantener un único argumento de crecimiento: todo lo asignado vive en `172.16.0.0/22`, y el resto de la /16 queda libre.
- El enlace `R-BOG-EDGE ↔ R-BOG-CORE` se incluye porque el diseño de Bogotá usa dos routers (borde y core) separados. Si la topología final usa un solo router en Bogotá, esta fila se elimina y las subredes WAN se renumeran.

## Recomendaciones

1. Mantener este archivo bajo control de versiones en Git.
2. Actualizar la tabla cada vez que se cree, modifique o elimine una subred.
3. Evitar reutilizar rangos IP entre sedes o segmentos.
4. Documentar cambios relevantes mediante commits descriptivos (`fix:`, `feat:`).
5. Validar disponibilidad de IP antes de asignar nuevos equipos o servicios.
6. Si se confirma un solo router en Bogotá, actualizar también `diseno/diseno.md` y el diagrama de topología antes de tocar `configs/`.

## Autor

Área de Tecnología e Información