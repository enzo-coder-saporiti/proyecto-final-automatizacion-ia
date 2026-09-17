# Ecosistema de Automatización IA — Calificación de Leads VIP

Sistema autónomo que recibe leads desde un formulario web, los califica con IA, redacta una propuesta comercial personalizada, la somete a aprobación humana y la envía por correo. Sin intervención manual, salvo el punto de control deliberado.

**Autor:** Enzo Gabriel Saporiti · **Curso:** AI Automation (#96920)

---

## Enlaces de la entrega

| Recurso | Enlace |
|---|---|
| **Documentación técnica (PDF)** | [Documentacion-Tecnica.pdf](Documentacion-Tecnica.pdf) |
| **Video demo (3 min)** | [Ver en Drive](https://drive.google.com/file/d/1M1XlcQgCgm9YzicrJ0fG6QQy023DIR_K/view?usp=sharing) |
| **Dashboard de control** | [Panel de KPIs](https://airtable.com/app9gr07C1p0TD5IL/shrNgRpVBnm6wZLP5) |
| **Base de datos (solo lectura)** | [Ver base](https://airtable.com/app9gr07C1p0TD5IL/shrt1sS9I6JBTCcVp) |
| **Diagrama de arquitectura** | [Diagrama General Proyecto Final.pdf](Diagrama%20General%20Proyecto%20Final.pdf) |
| **Workflow principal** | [PROD - Flujo Principal.json](%5BPROD%5D%20Calificacion%20de%20Leads%20IA%20-%20Flujo%20Principal.json) |
| **Workflow de errores** | [SYS - Error Handler Global.json](%5BSYS%5D%20Error%20Handler%20Global.json) |

---

## Mapa de la rúbrica

Los cinco criterios están desarrollados en el PDF, en este orden:

| Criterio | Peso | Dónde |
|---|---|---|
| Mapa de arquitectura | 20% | PDF, sección 1 |
| Estructuras de datos documentadas | 20% | PDF, sección 2 |
| Optimización de costos | 20% | PDF, sección 3 |
| Seguridad y resiliencia | 20% | PDF, sección 4 |
| Dashboard de control | 20% | PDF, sección 5 + enlace arriba |

Las evidencias de los 7 escenarios de prueba están en el Anexo A1 del PDF, con 34 capturas.

---

## Stack

| Capa | Tecnología |
|---|---|
| Orquestador | n8n — 28 nodos + workflow de manejo de errores |
| Base de datos | Airtable — 5 tablas relacionadas |
| Procesamiento IA | Anthropic Claude — Haiku 4.5 (clasificación) + Sonnet 4.6 (redacción) |
| Canales de salida | Slack (aprobación interna) + Gmail (prospecto) |

---

## Qué hace el sistema

1. **Recibe** un lead por webhook (`POST /nuevo-lead`)
2. **Normaliza y valida** — castea tipos y verifica campos obligatorios. Los rechazos se registran con su payload completo
3. **Resuelve la empresa** por upsert, evitando duplicados
4. **Clasifica** con Claude Haiku 4.5: score 0–100, categoría, urgencia y necesidad detectada
5. **Rutea** en tres caminos: `Descartado` se archiva, `Estandar` va a ventas, `VIP` continúa
6. **Redacta** una propuesta con Claude Sonnet 4.6, usando el análisis previo como contexto
7. **Se detiene** y solicita aprobación humana en Slack, con timeout de 24 horas
8. **Envía** por Gmail solo si un humano aprobó, y persiste el `threadId`
9. **Registra** tokens, costo y duración de cada ejecución

---

## Decisiones de diseño

**Un solo proveedor de IA, dos tiers.** Haiku para clasificar, Sonnet para redactar. Reduce la superficie de credenciales y unifica el parseo, conservando la segmentación por costo. Ahorro del 62% frente a usar el tier superior en todo.

**El ruteo condicional es la mayor optimización.** El 80% del volumen nunca alcanza el modelo de redacción: eso ahorra más que la elección de modelo.

**Se persiste antes de procesar.** El lead se guarda antes de la primera llamada a la IA. Un fallo de API es entonces un reintento, no una pérdida de datos.

**Gate humano en el único paso irreversible.** Enviar un correo a un prospecto no se deshace. Todo lo anterior sí.

**Nada hardcodeado.** Prompts, umbrales y canales viven en una tabla de Airtable y se resuelven en tiempo de ejecución.

---

## Evidencias incluidas

| Archivo | Contenido |
|---|---|
| `01-canvas-flujo-principal.png` | Canvas completo, 28 nodos |
| `02-canvas-error-handler.png` | Workflow de manejo de errores |
| `03-ejecucion-exitosa.png` | Ejecución completa en verde |
| `04-ejecucion-con-error.png` | Rama de error activada |
| `05-hitl-slack.png` | Aprobación humana en Slack |
| `06-email-recibido.png` | Correo generado y enviado |
| `07-airtable-leads.png` | Tabla de leads con estados |
| `08-airtable-log-errores.png` | Registro de errores clasificados |
| `dashboard 1.png` a `dashboard 3.png` | Panel de control con KPIs |

---

## Reproducción

1. Importar los dos workflows `.json` en n8n
2. Configurar 4 credenciales: Airtable (PAT con 3 scopes, alcance a una base), Anthropic, Slack (bot token), Gmail (OAuth2)
3. Duplicar la base de Airtable desde el enlace de solo lectura
4. Cargar los 6 registros de la tabla `Config y Prompts`
5. En Settings del flujo principal, asignar el Error Workflow
6. Publicar ambos workflows
7. Emitir un `POST` al webhook con el esquema documentado en el PDF, sección 2.4

**Costo de puesta en marcha:** USD 5 de crédito en Anthropic. El resto del stack opera en planes gratuitos.

---

## Seguridad

Los enlaces publicados son **vistas compartidas de solo lectura** (`airtable.com/app.../shr...`). No se publican enlaces de invitación, que otorgarían acceso de colaborador.

Este repositorio no contiene credenciales: los workflows exportados incluyen referencias a credenciales, no sus valores. Todos los datos de prueba son ficticios.
