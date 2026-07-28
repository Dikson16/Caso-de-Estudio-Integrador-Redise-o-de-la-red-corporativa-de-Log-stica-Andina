# Plan de Direccionamiento IP

Este documento consolida el plan de direccionamiento IP para las sedes y segmentos de red definidos, incluyendo la cantidad de hosts actuales, la proyección de crecimiento del 30%, la subred asignada, máscara y rango utilizable.

## Tabla de subredes

| Sede | Segmento | Hosts Actuales | Hosts Proyectados (+30%) | Subred Asignada | Máscara | Rango Utilizable |
|---|---:|---:|---:|---|---|---|
| Bogotá | Administración | 120 | 156 | 172.16.0.0 | /24 | 172.16.0.1 - 172.16.0.254 |
| Bogotá | Invitados | 100 | 130 | 172.16.1.0 | /24 | 172.16.1.1 - 172.16.1.254 |
| Bogotá | Operaciones | 60 | 78 | 172.16.2.0 | /25 | 172.16.2.1 - 172.16.2.126 |
| Medellín | Usuarios | 60 | 78 | 172.16.2.128 | /25 | 172.16.2.129 - 172.16.2.254 |
| Bogotá | TI | 30 | 39 | 172.16.3.0 | /26 | 172.16.3.1 - 172.16.3.62 |
| Medellín | Operaciones | 28 | 37 | 172.16.3.64 | /26 | 172.16.3.65 - 172.16.3.126 |
| Cali | Usuarios | 25 | 33 | 172.16.3.128 | /26 | 172.16.3.129 - 172.16.3.190 |
| Bogotá | Gestión | 10 | 13 | 172.16.3.192 | /28 | 172.16.3.193 - 172.16.3.206 |
| Bogotá | DMZ | 6 | 8 | 172.16.3.208 | /28 | 172.16.3.209 - 172.16.3.222 |
| WAN | Túnel GRE BOG-MED | 2 | 2 | 172.16.255.0 | /30 | 172.16.255.1 - 172.16.255.2 |
| WAN | Enlace Dedicado BOG-CALI | 2 | 2 | 172.16.255.4 | /30 | 172.16.255.5 - 172.16.255.6 |

## Resumen técnico

- La red base utilizada es `172.16.0.0/16`.
- Las subredes fueron asignadas considerando crecimiento proyectado del 30%.
- Los segmentos con mayor demanda de hosts utilizan máscaras `/24` y `/25`.
- Los segmentos pequeños como Gestión y DMZ utilizan `/28`.
- Los enlaces WAN punto a punto utilizan máscaras `/30`, adecuadas para dos direcciones IP utilizables.

## Recomendaciones

1. Mantener este archivo bajo control de versiones en Git.
2. Actualizar la tabla cada vez que se cree, modifique o elimine una subred.
3. Evitar reutilizar rangos IP entre sedes o segmentos.
4. Documentar cambios relevantes mediante commits descriptivos.
5. Validar disponibilidad de IP antes de asignar nuevos equipos o servicios.

## Autor

Área de Tecnología e Información
