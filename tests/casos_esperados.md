# Casos esperados (CAPA 0)

> Solo para pruebas. **Nunca** se carga en el agente ni en las tools: contiene las respuestas.

| Cliente | Identificar con | Falla | Área | Prioridad | Visita | HITL | Ticket |
|---|---|---|---|---|---|---|---|
| CLI-1001 | DUI 09901001-4 | R01 · Caída masiva en el PON (corte de fibra troncal o feeder) | NOC | P2 | No | — | no |
| CLI-1002 | Factura FAC-2026-100209 | R02 · Corte de energía en casa del cliente (dying gasp) | Soporte | P2 | Sí | agendar_visita | crear |
| CLI-1003 | DUI 09901003-0 | R03 · Señal óptica degradada (conector sucio, curvatura o empalme dañado) | Campo | P3 | Sí | agendar_visita, cerrar_ticket | crear |
| CLI-1004 | DUI 09901004-9 | R06 · WiFi 2.4 GHz saturado o con interferencia | Soporte_Hogar | P4 | No | — | opcional |
| CLI-1005 | Factura FAC-2026-100509 | R05 · Servicio suspendido por mora | Facturacion | P4 | No | — | no |
| CLI-1006 | DUI 09901006-5 | R04 · Pérdida de señal individual (drop cortado o desconectado) | Campo | P2 | Sí | agendar_visita | crear |
| CLI-1007 | DUI 09901007-3 | R08 · ONT sin sesión WAN (PPPoE caído o ONT colgada) | Soporte | P2 | No | reiniciar_ont, cerrar_ticket | crear |
| CLI-1008 | Factura FAC-2026-100810 | R07 · Cobertura WiFi insuficiente en parte de la casa | Soporte_Hogar | P4 | No | — | opcional |
| CORP-2001 | NIT 0614-150515-901-1 | C01 · Enlace dedicado caído por corte de fibra en la última milla | Campo | P1 | Sí | agendar_visita | crear |
| CORP-2002 | NIT 0614-020318-902-3 | C02 · Sesión BGP caída con el enlace físico arriba | Soporte_Corporativo | P1 | No | — | crear |
| CORP-2003 | NIT 0614-110720-903-5 | C04 · Saturación del enlace (latencia y pérdida por uso alto) | Soporte_Corporativo | P3 | No | — | crear |
| CORP-2004 | Factura FAC-2026-200410 | C03 · VLAN de servicio mal configurada en el CPE | Soporte_Corporativo | P1 | No | — | crear |

## Frase del cliente y resultado esperado

### CLI-1001 — R01
- **Cliente dice:** “Hola, no tengo internet desde hace como dos horas y la luz del aparato está roja.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont
- **Resultado:** Informa la masiva INC-2026-0451, la causa y la ETA. No abre ticket individual ni agenda visita.

### CLI-1002 — R02
- **Cliente dice:** “Se fue la luz en mi casa, ya regresó pero el internet no.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, buscar_kb, crear_ticket, agendar_visita
- **Resultado:** Detecta DYING_GASP sin masiva y guía las pruebas de energía. Guion: la ONT no enciende en otro tomacorriente, así que crea ticket y agenda visita para cambiar el adaptador (con aprobación).

### CLI-1003 — R03
- **Cliente dice:** “El internet se me va a cada rato, se desconecta y vuelve solo.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, buscar_kb, crear_ticket, agendar_visita, consultar_ticket, cerrar_ticket
- **Resultado:** Detecta Rx -27.6 dBm (crítico), OLT Rx -30.2 y 14 flaps. Guía la revisión del patchcord. Crea ticket a Campo y agenda visita (con aprobación). Seguimiento y cierre al confirmar la reparación.

### CLI-1004 — R06
- **Cliente dice:** “El internet está bien lento en la noche, sobre todo en los celulares.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, wifi_hogar, buscar_kb
- **Resultado:** La ONT mide 296/148 Mbps (plan 300). La banda 2.4 GHz está al 88%, con 11 redes en el canal 6. Recomienda pasar dispositivos a 5 GHz y cambiar el canal y el ancho. Sin visita.

### CLI-1005 — R05
- **Cliente dice:** “No me abren las páginas, me sale una página del proveedor.”
- **Tools esperadas:** identificar_cliente, facturacion
- **Resultado:** Informa suspensión por mora (47 días, saldo vencido $56.50), los canales de pago y la reconexión automática hasta 30 min después del pago. No crea ticket técnico.

### CLI-1006 — R04
- **Cliente dice:** “No tengo internet y la lucecita que dice LOS está roja.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, buscar_kb, crear_ticket, agendar_visita
- **Resultado:** Detecta LOS individual: 1 de 28 ONT del PON y sin incidente activo. INC-2026-0447 está resuelto y no aplica. Pide revisar el cable y pregunta por obras o poda. Crea ticket P2 y agenda visita (con aprobación).

### CLI-1007 — R08
- **Cliente dice:** “Tengo WiFi pero no hay internet. La luz PON está verde y la de Internet apagada.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, crear_ticket, reiniciar_ont, cerrar_ticket
- **Resultado:** Detecta Rx normal, WAN down y PPPOE_TIMEOUT con facturación al día. Propone reinicio remoto (con aprobación). Después del reinicio, la WAN sube con IP. Cierra el ticket como resuelto en el primer contacto.

### CLI-1008 — R07
- **Cliente dice:** “En el cuarto del segundo piso no me llega el internet, se cortan las videollamadas.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, wifi_hogar, buscar_kb
- **Resultado:** La ONT mide 497/248 Mbps (plan 500) y la utilización es normal. Los dispositivos del piso 2 están entre -77 y -82 dBm. Recomienda sacar el router del mueble y reubicarlo, cablear el cuarto o instalar un mesh.

### CORP-2001 — C01
- **Cliente dice:** “Se nos cayó el enlace, toda la bodega está sin internet.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, enlace_corporativo, crear_ticket, agendar_visita
- **Resultado:** Detecta LOS en la ONT del enlace, sin masiva en el PON y la WAN del CPE en down. Crea ticket P1, escala al NOC y agenda visita urgente (con aprobación). Informa el SLA de 4 h de restauración, con 35 min transcurridos.

### CORP-2002 — C02
- **Cliente dice:** “Tenemos enlace pero no salimos a Internet.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, enlace_corporativo, crear_ticket
- **Resultado:** Detecta capa física y VLAN correctas, ping al gateway correcto y BGP en Active con 0 prefijos. Relaciona la caída con el cambio de ACL del cliente de hace 90 min. Pide revisar TCP/179, el ASN y MD5. Ticket P1 a Soporte_Corporativo. Sin visita.

### CORP-2003 — C04
- **Cliente dice:** “El internet está lentísimo en la mañana y las llamadas se cortan.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, enlace_corporativo, crear_ticket
- **Resultado:** Detecta p95 de 97% sobre 200 Mbps de 08:30 a 12:00, latencia de 182 ms, pérdida de 3.6% y 18452 descartes de salida, con 0 errores CRC. Identifica el servidor de respaldos (46%). Recomienda QoS, mover los respaldos fuera de horario y evaluar una ampliación.

### CORP-2004 — C03
- **Cliente dice:** “Cambiamos el router y desde entonces no hay internet.”
- **Tools esperadas:** identificar_cliente, facturacion, incidentes_pon, estado_ont, enlace_corporativo, crear_ticket
- **Resultado:** Detecta ONT online y WAN up, pero el CPE está sin VLAN (se espera la 310), con 0 MAC aprendidas y ARP incompleto. Guía a TI para configurar la subinterfaz con VLAN 310 e IP 203.0.113.14/30. Ticket P1 a Soporte_Corporativo.

## Fallas sin cliente (reservadas para el caso nuevo no visto, CAPA 4)

R09 (hardware ONT), R10 (corte de energía zonal), R11 (mantenimiento programado), C05 (óptica corporativa), C06 (dying gasp corporativo), C07 (mora corporativa). Para probar una, se agrega un cliente nuevo a los JSON **sin tocar el prompt ni el código**.
