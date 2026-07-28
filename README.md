# Caso de Estudio Integrador — Rediseño de la red corporativa de Logística Andina

Proyecto integrador del curso CCNA v7 (ENSA). Rediseño y documentación de la red corporativa de Logística Andina, que incluye diseño lógico, configuraciones, diagramas y procedimientos de troubleshooting.

Descripción
- Proyecto integrador del curso CCNA v7 (ENSA). Rediseño y documentación de la red corporativa de Logística Andina, incluye diseño lógico, configuraciones, diagramas y procedimientos de troubleshooting.

Objetivos
- Diseñar una topología resiliente y escalable.
- Definir direccionamiento IP y segmentación VLAN.
- Implementar políticas de seguridad y QoS básicas.
- Documentar configuraciones y procedimientos de verificación.

Estructura del repositorio
- `automatizacion/`: scripts y plantillas para despliegue automático.
- `configs/`: configuraciones de routers/switches por sitio (ej.: `R-BOG-CORE.txt`, `R-CALI.txt`).
- `diseno/`: documentos de diseño y direccionamiento, y diagramas en `diseno/diagramas/`.
- `gestion/`: políticas y procedimientos de gestión.
- `qos/`: diseño y pruebas de Quality of Service.
- `seguridad/`: políticas y configuraciones de seguridad.
- `topologia/`: archivo de topología de Packet Tracer (`red.pkt`).
- `troubleshooting/`: guías y ejemplos de resolución de problemas.
- `wan/`: diseño y configuraciones de la red WAN.

Archivos clave
- [configs/ISP-1.txt](configs/ISP-1.txt) — configuración ejemplo para proveedor ISP-1.
- [topologia/red.pkt](topologia/red.pkt) — topología en Packet Tracer.
- [diseno/direccionamiento.md](diseno/direccionamiento.md) — esquema de direccionamiento IP.

Cómo usar
- Abrir `topologia/red.pkt` con Packet Tracer para visualizar la topología.
- Consultar `diseno/direccionamiento.md` para la asignación de subredes antes de aplicar configuraciones.
- Cargar configuraciones desde `configs/` en los dispositivos correspondientes (ver convenciones de nombres en `configs/`).

