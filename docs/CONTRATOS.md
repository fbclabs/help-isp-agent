# Contratos de datos (CAPA 0)

Todos los datos son ficticios. Las IP usan rangos de documentación (RFC 5737) y los DUI/NIT solo respetan el formato, sin dígito verificador real.

## Archivos y sistema que simulan

| Archivo | Sistema simulado | Llave | Lo lee la tool |
|---|---|---|---|
| `data/crm_clientes.json` | CRM | `clienteId`, `dui` o `nit` | `identificar_cliente` |
| `data/bss_facturacion.json` → `cuentas[]` | BSS / facturación | `clienteId`, `facturas[].numero` | `identificar_cliente` (por factura), `facturacion` |
| `data/oss_ont.json` | OSS / gestor de OLT | `clienteId` | `estado_ont`, `reiniciar_ont` |
| `data/noc_incidentes.json` → `incidentes[]`, `pons[]` | NOC | `pon` | `incidentes_pon` |
| `data/acs_wifi.json` → `equipos[]` | ACS TR-069 (solo residencial) | `clienteId` | `wifi_hogar` |
| `data/corp_enlaces.json` → `enlaces[]` | Inventario corporativo y monitoreo | `clienteId` | `enlace_corporativo`, portal (CAPA 3) |
| `data/catalogo_fallas.json` | Base de conocimiento | `fallas[].id`, `palabrasClave` | `buscar_kb` |
| `data/tickets.json` | Mesa de ayuda | `ticketId` | tools de tickets |

## Reglas para las tools (CAPA 1)

1. **Primero identificar.** Ninguna tool devuelve datos si el `clienteId` no salió de `identificar_cliente` en la misma conversación.
2. **Nunca se devuelve el bloque `simulacion`** de `oss_ont.json`. Solo lo usa `reiniciar_ont` para cambiar el estado en memoria después de un reinicio aprobado.
3. **`tests/casos_esperados.*` nunca se carga en el agente.** Ahí están las respuestas.
4. Los tiempos del NOC son relativos (`inicioHaceMin`, `etaEnMin`, `inicioEnMin`). La tool convierte a hora real con `now()`, así la demo no queda desfasada.
5. `"sin_lectura"` significa que no hay medición (ONT en LOS u offline). El agente debe decir "sin lectura" y no inventar un valor. La última lectura conocida está en `rxUltimaLecturaDbm`.

## Áreas

`NOC` · `Soporte` · `Soporte_Hogar` · `Campo` · `Facturacion` · `Soporte_Corporativo`

## Prioridades

`P1` corporativo caído o en riesgo de SLA · `P2` residencial sin servicio · `P3` degradado o intermitente · `P4` WiFi, consulta o facturación

## Ticket (`data/tickets.json`, arreglo)

```json
{
  "ticketId": "TKT-000001",
  "clienteId": "CLI-1003",
  "tipoCliente": "residencial",
  "fallaId": "R03",
  "sintoma": "El internet se va a cada rato",
  "diagnostico": "Rx -27.6 dBm (crítico), OLT Rx -30.2 dBm, 14 flaps en 24 h",
  "evidencia": [
    { "tool": "estado_ont", "dato": "rxDbm", "valor": "-27.6" }
  ],
  "area": "Campo",
  "prioridad": "P3",
  "estado": "abierto",
  "requiereVisita": true,
  "visita": { "ordenId": "OT-000001", "fecha": "2026-10-12 08:00-12:00", "aprobadoPor": "operador" },
  "accionesEjecutadas": [
    { "accion": "agendar_visita", "decisionHumana": "approve", "resultado": "OT-000001 creada" }
  ],
  "historial": [
    { "fecha": "2026-10-11T10:05:00", "autor": "agente", "nota": "Ticket creado" }
  ],
  "creado": "2026-10-11T10:05:00",
  "actualizado": "2026-10-11T10:05:00",
  "cerrado": "",
  "notaCierre": ""
}
```

Estados: `abierto` → `en_progreso` → `pendiente_cliente` → `escalado` → `cerrado`.

## Ficha de escalamiento

La ficha es el mismo ticket con `estado: "escalado"` y estos campos extra: `escaladoA` (área), `resumen` (3 líneas para el humano) y `siguientePaso`.
