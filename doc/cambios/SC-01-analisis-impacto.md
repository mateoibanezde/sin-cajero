# Análisis de impacto · SC-01 Propina al pagar

**Sistema:** sin-cajero
**Solicitud:** [`SC-01-solicitud-propina.md`](SC-01-solicitud-propina.md)
**Solicitante:** Omar, dupla revisora (entregada el 12/11/2026)
**Analizó:** Mateo Ibañez de la Cueva
**Fecha del análisis:** 13/11/2026
**Recomendación:** aplazar (ver apartado 7)

Los requisitos que se citan están en `docs/especificacion-requisitos.md`, las piezas y nombres de clase en `doc/arch/diagramas.md` (C4) y `doc/arch/diagramas-proyecto.md` (flujo, secuencia, clases, estados y ER), y la regla de módulos en `doc/arch/adr-001-monolito-modular.md`.

---

## 1. Qué pidió, en sus palabras

> "Quiero que, al pagar en el kiosco o en el celular, el cliente pueda dejar propina —10 %, 15 %, 20 % u otra cantidad— y que al cierre del día el dueño vea cuánto se juntó para repartirlo entre los baristas."

Agrega que debe poder pagarse sin propina y que el cliente debe ver por separado el pedido y la propina antes de confirmar. Su motivo es que sin-cajero quita el mostrador de cobro y, con él, el momento en que el cliente deja propina. Si los baristas ganan menos con el sistema, van a resistirse a usarlo.

---

## 2. Traducción a sin-cajero

| Lo que dijo Omar | Dónde vive en sin-cajero |
|---|---|
| "al pagar" | Paso 5 de CU-02 y su equivalente en CU-01. Mensajes `pagar(pedido)` y `cobrar(pedido, total)` del diagrama de secuencia (§2). Método `Pedido.pagar()` del diagrama de clases (§3). |
| "en el kiosco o en el celular" | Contenedor *Aplicación de pedidos* (C4 nivel 2). Pantallas del prototipo "Resumen y total del pedido" y "Confirmación de pago". |
| "dejar propina 10/15/20 % u otra cantidad" | Un dato del **cobro**, no del pedido. Su dueño es el módulo **Pagos**: nueva columna `PAGO.propina` y firma `ProcesadorPago.cobrar(pedido, monto, propina)`. |
| "pueda pagar sin propina" | Opción explícita "sin propina" (`propina = 0`), seleccionada por omisión. |
| "al cierre del día" | RF-011 y CU-07 *Generar corte de caja diario*, en el módulo **Caja**, tabla `CORTE_CAJA`. |
| "el dueño vea cuánto se juntó" | Nueva columna `CORTE_CAJA.total_propinas` y su línea en la pantalla de corte de caja del *Panel del dueño*. |
| "para repartirlo entre los baristas" | **Sin equivalente.** sin-cajero no registra turnos. `ACCION_PEDIDO.actor` solo dice quién marcó un pedido como listo, no quién trabajó el turno. El sistema entrega el total del día y el dueño hace el reparto fuera del sistema. |
| (implícito en "no sé si todas van a querer") | Configuración por cafetería: activar o desactivar la propina y elegir porcentajes. Va en el módulo **Catálogo** (nueva tabla `CONFIG_PROPINA`), en el esquema de cada cafetería (C4 nivel 2). |

**Por qué la propina no es una línea más del pedido.** El atajo obvio sería agregarla como un "producto" en `LINEA_PEDIDO` o sumarla a `Pedido.total`. Las dos opciones rompen RF-003, que exige que el total sea la suma exacta de los precios de catálogo "sin excepción manual posible". Una propina de "otra cantidad" es justo un monto manual. Por eso el pedido no se toca y la propina viaja junto al cobro.

**Por qué la configuración va en Catálogo y no en Pagos.** En el diagrama de componentes (nivel 3), las aplicaciones solo hablan con Pedidos y con Catálogo. El kiosco ya carga el catálogo al abrir, y el dueño ya lo edita en CU-05. Si la configuración viviera en Pagos, habría que agregar una flecha nueva de las aplicaciones a Pagos. En Catálogo no aparece ninguna dependencia nueva.

---

## 3. Impacto directo

Todo lo que hay que tocar para que el cambio exista:

| # | Pieza | Artefacto | Qué cambia |
|---|---|---|---|
| D1 | *Aplicación de pedidos*, kiosco y celular | Prototipo en Figma, CU-01 y CU-02 | Paso nuevo "Propina" entre el resumen y el pago: 10 %, 15 %, 20 %, otra cantidad o sin propina. El cliente ve el pedido y la propina en dos líneas, y el cargo final como la suma. |
| D2 | Pedidos: `Pedido.pagar()` | Clases §3, secuencia §2 | Recibe la propina elegida y la pasa a Pagos: `cobrar(pedido, total)` pasa a `cobrar(pedido, total, propina)`. `Pedido.total` **no cambia**. |
| D3 | Pagos: `ProcesadorPago` y `AdaptadorProveedor` | Clases §3, componentes nivel 3 | `cobrar(pedido, monto)` pasa a `cobrar(pedido, monto, propina)`. El adaptador hace **un solo cargo** por la suma al proveedor. |
| D4 | Pagos: tabla `PAGO` | ER §5 | Columna nueva `propina`. `monto` sigue siendo solo el importe del pedido. |
| D5 | Catálogo: configuración | ER §5, pantalla de catálogo del *Panel del dueño* | Tabla nueva `CONFIG_PROPINA` (activa, porcentajes) en el esquema de cada cafetería. La lee la *Aplicación de pedidos* junto con el catálogo. |
| D6 | Caja: `CORTE_CAJA` y pantalla de corte | ER §5, RF-011, CU-07, prototipo | Columna `total_propinas` y una línea "Propinas del día" separada de las ventas. |
| D7 | Especificación | `docs/especificacion-requisitos.md` | RF-012 nuevo (propina opcional al pagar). RF-003 se aclara: distingue "total del pedido" de "cargo al cliente". Cambian RF-011 y su criterio, CU-01 y CU-02 (paso nuevo), la tabla de trazabilidad y el registro de cambios. |

**Lo que no cambia, y por qué.**

- **Diagrama de estados (§4).** La propina se elige en `EnCaptura` y se cobra en la misma transición `EsperandoPago → Pagado`. No hace falta un estado nuevo.
- **RF-004 y RNF-SEG-001 (integridad pedido-pago).** Hay un solo cargo por pedido y propina juntos. Si la propina se cobrara aparte, podría quedar un pedido pagado con la propina sin cobrar, que es justo el estado "a medias" que prohíbe RNF-SEG-001.
- **Pantalla de barra (RF-006, RF-009).** Como la propina vive en Pagos, el objeto `Pedido` que recibe `PantallaBarra` (observador) no la trae. El barista no ve quién dejó propina, y el orden de la cola no puede depender de eso. Ubicar el dato en Pagos protege esta regla sin trabajo extra.
- **Topología del ADR-001.** Ningún módulo lee tablas ajenas (ver I1).

---

## 4. Impacto indirecto

Efectos que la solicitud no nombra.

### I1 · La conciliación del corte de caja deja de servir como alarma (el más caro)

**Qué pasa.** Hoy `CORTE_CAJA.diferencia` compara lo cobrado contra el valor de los pedidos entregados. En un día normal debe dar **cero**. Esa diferencia es la alarma del dueño y la razón de ser del sistema: la Visión del producto define el problema como "producto que sale sin quedar registrado". Si la propina entra en el mismo cargo y nadie la separa, se vuelve parte de lo cobrado y la diferencia **nunca vuelve a ser cero**.

**Ejemplo.** Día con 100 pedidos entregados por $6,500 y $650 de propinas. Si las propinas no se separan, el corte marca una diferencia de +$650. Supongamos que ese día además se canceló un pedido de $45 sin reembolso. La diferencia sube a +$695, y nadie la revisa porque "siempre sale algo por las propinas". Una alarma que suena todos los días deja de ser alarma. También se rompe el criterio de aceptación de RF-011, que pide mostrar la diferencia "sin intervención manual": el dueño tendría que restar las propinas a mano.

**Por qué toca la arquitectura.** Según el ADR-001 y el nivel 3, Caja solo habla con Pedidos ("pide los pedidos entregados y cancelados"), y Pagos y Caja no hablan entre sí. Caja no puede leer `PAGO.propina` directamente. La interfaz de Pedidos hacia Caja tiene que devolver la propina de cada pedido, y Pedidos tiene que pedírsela a Pagos. El dato no es de Pedidos, pero Pedidos queda como intermediario. Es la opción que respeta el ADR. La otra, una flecha nueva de Caja a Pagos, cambiaría la topología y necesitaría un ADR-002.

**Qué hay que hacer.** Hay que definir dos totales en lugar de uno: `total_cobrado` (solo pedidos, sin propina) y `total_propinas`, aparte. La fórmula de `diferencia` usa solo el primero. Lo que deposita el proveedor es la suma de los dos menos su comisión (ver I4), así que conciliar contra el estado de cuenta también cambia.

**Por qué es el más caro.**
1. Toca el control que se vende, no una función secundaria.
2. Si falla, falla **en silencio**: no aparece ningún error, solo un número que deja de ser cero.
3. Concentra el renglón de pruebas más incierto de la estimación. Hay que probar con y sin propina, pedidos cancelados con propina, anticipados cancelados antes de la llegada y días con varias cafeterías (un esquema por cafetería).

### I2 · La cancelación autorizada tiene que reembolsar también la propina

RF-010 y CU-06 dicen que solo el dueño cancela un pedido pagado y que queda registrado quién y cuándo. No dicen cuánto se devuelve, porque hoy se devuelve el pedido completo. Con propina, el reembolso es pedido más propina, `total_propinas` del corte tiene que excluir la de pedidos cancelados, y la acción de cancelación en `ACCION_PEDIDO` (RNF-SEG-002) debería registrar el monto reembolsado. Si no, un día con cancelaciones vuelve a abrir una diferencia.

### I3 · RNF-USA-001: los 90 segundos del kiosco

RNF-USA-001 pide que un cliente de primera vez complete un pedido con una personalización en menos de 90 segundos, probado con 5 usuarios. La propina agrega una pantalla y una decisión al flujo del kiosco. Esa métrica ya se ajustó una vez, de 60 a 90 s, porque el caso real era más largo de lo supuesto. Hay que repetir la prueba. Si no pasa, la propina tiene que ir dentro de la pantalla de pago y no como un paso aparte.

### I4 · La comisión del proveedor de pagos se cobra sobre la propina

El *Adaptador de pagos* hace un solo cargo por pedido más propina. El proveedor cobra su porcentaje sobre todo el cargo, así que el dueño pagaría comisión sobre dinero que no es suyo, a menos que se descuente de la propina. Es una decisión de negocio que el sistema no puede tomar, pero el corte tiene que reflejarla. Queda como pregunta abierta (apartado 8).

### I5 · Cafeterías que no quieren propina (SaaS)

sin-cajero está pensado para varias cafeterías, con un esquema de base de datos por cafetería. Omar mismo duda que todas quieran propina. Por eso la configuración por cafetería (D5) no es opcional, y lo seguro es que la propina venga **desactivada por omisión**.

---

## 5. Resumen de piezas

| Módulo o contenedor | Directo | Indirecto |
|---|---|---|
| Aplicación de pedidos | D1 | I3 |
| Pedidos | D2 | I1 (interfaz hacia Caja), I2 |
| Pagos | D3, D4 | I1, I2 (reembolso), I4 |
| Catálogo | D5 | I5 |
| Caja | D6 | I1, I2 |
| Especificación y diagramas | D7 | RNF-USA-001 (I3), RF-010 (I2) |

---

## 6. Estimación de esfuerzo

**Número: 33 horas** (rango de 20 a 50 h), de una persona.

**Método.** Estimación de tres puntos (PERT) por pieza: E = (O + 4M + P) / 6, donde O es optimista, M más probable y P pesimista.

| # | Pieza | O | M | P | E |
|---|---|---|---|---|---|
| 1 | Especificación: RF-012, RF-003, RF-011, CU-01, CU-02, trazabilidad (D7) | 1 | 1.5 | 2 | 1.5 |
| 2 | Diagramas: secuencia, clases y ER (D2–D6) | 1 | 1.5 | 2.5 | 1.6 |
| 3 | Prototipo en Figma: paso de propina en kiosco y celular, corte con propinas (D1, D6) | 2 | 3 | 4 | 3.0 |
| 4 | *Aplicación de pedidos*: selector de propina en kiosco y celular (D1) | 3 | 5 | 8 | 5.2 |
| 5 | Pagos: `cobrar(pedido, monto, propina)`, adaptador, `PAGO.propina`, reembolso (D3, D4, I2) | 3 | 5 | 8 | 5.2 |
| 6 | Pedidos: pasar la propina y exponerla hacia Caja sin romper el ADR (D2, I1) | 1 | 2 | 3 | 2.0 |
| 7 | Caja: fórmula nueva de conciliación y `total_propinas` (D6, I1) | 3 | 4 | 7 | 4.3 |
| 8 | Catálogo y *Panel del dueño*: `CONFIG_PROPINA` por cafetería (D5, I5) | 2 | 3 | 5 | 3.2 |
| 9 | Migración del esquema de cada cafetería (`PAGO`, `CORTE_CAJA`, `CONFIG_PROPINA`) | 1 | 2 | 3 | 2.0 |
| 10 | Pruebas: conciliación con todas las combinaciones (I1, I2) y nueva prueba de RNF-USA-001 con 5 usuarios (I3) | 3 | 5 | 8 | 5.2 |
| | **Total** | **20** | **32** | **50.5** | **33.1** |

**Contra qué lo comparé.** Apliqué la misma tabla de piezas a **RF-005** (pedido anticipado y pantalla "Ya llegué"), que tiene forma parecida: una pantalla nueva en la *Aplicación de pedidos*, un cambio en `Pedido` (estrategia `EnvioPorLlegada` y estado `EsperandoLlegada`) y pruebas del flujo. Da **unas 16 h**. Propinas sale al doble, y es coherente: RF-005 toca un módulo (Pedidos) y propinas toca cuatro (Pedidos, Pagos, Caja y Catálogo), además de la migración y la conciliación.

**Supuestos.**
- Son horas de una sola persona, porque el ADR-001 parte de que el sistema lo construye una persona.
- El núcleo (RF-001 a RF-011) se construye primero. La estimación mide **cuánto agrega** este cambio, no el núcleo.
- El proveedor de pagos acepta un solo cargo con un desglose (pedido y propina) en los metadatos. Si no lo acepta, el renglón 5 se va al pesimista.
- El reparto entre baristas queda fuera (apartado 2). Si se incluyera, habría que modelar turnos, que hoy no existen, y la cifra crecería bastante más que un renglón.

**Dónde está la incertidumbre.** En los renglones 7 y 10, la conciliación y sus pruebas, por I1. Juntos tienen un rango de 6 a 15 h y explican la mayor parte de la distancia entre el optimista y el pesimista.

---

## 7. Recomendación: aplazar

Lo aplazo, no lo rechazo. El problema que señala Omar es real: sin mostrador se pierde el momento de la propina, y un barista que gana menos tiene motivos para resistirse al sistema. Aun así, hoy no conviene meterlo, por lo que encontré en este análisis:

1. **No ataca el problema central, pero sí toca el mecanismo que lo vigila.** sin-cajero existe para que nada salga de la barra sin pagarse, y la conciliación del corte (I1) es como el dueño lo comprueba. El cambio agrega riesgo justo ahí, a cambio de un beneficio que no es el objetivo del producto.
2. **El corte de caja todavía no se ha probado en uso real.** El ADR-001 reconoce que las fronteras "todavía no se han probado en uso real". Si las propinas entran antes de que el corte dé cero en un día normal, no voy a poder saber si una diferencia viene de un error del corte o de la propina.
3. **Cuesta el doble que RF-005**, un requisito del alcance clasificado como Importante, y esto no está en el alcance.
4. **El problema tiene una solución de $0 mientras tanto.** El cliente sigue recogiendo su bebida en la barra, frente al barista. Un frasco de propinas en el punto de entrega mantiene la propina sin tocar el sistema.

**Cuándo se reactiva.** Cuando se cumpla una de estas condiciones:
- el corte de caja corre dos semanas seguidas en un local con diferencia en cero en días normales, o
- una cafetería cliente la pide como condición para contratar el servicio.

**Lo que sí hago ya (unos 30 minutos).** Dejar escrita en el diagrama ER la regla "`PAGO.monto` es solo el importe del pedido; nada que no sea del pedido se suma ahí". Así, cuando la propina llegue, ya tiene dónde ir sin migrar cortes históricos ni cambiar el significado de una columna con datos reales.

---

## 8. Preguntas abiertas para Omar

1. ¿Quién absorbe la comisión del proveedor sobre la propina, el dueño o la propina misma? (I4)
2. ¿Se puede dejar propina en un pedido anticipado que todavía no llega a la barra? Si se cancela, ¿se reembolsa la propina completa? (I2)
3. ¿Bastaría con el total del día, o el dueño necesita el desglose por barista? Lo segundo exige modelar turnos.

---

## 9. Registro

| Fecha | Qué pasó |
|---|---|
| 12/11/2026 | Omar entrega la solicitud SC-01 en la sesión de revisión |
| 13/11/2026 | Análisis de impacto. Recomendación: aplazar, con condiciones de reactivación |
| 17/11/2026 | Firma de `cobrar` alineada con el diagrama de clases corregido: `cobrar(pedido, monto)` |
