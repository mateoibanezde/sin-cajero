# Especificación de requisitos

> **Plantilla del curso · Ingeniería de Software I · SIS3407**
> Copia este archivo a tu repositorio como `docs/especificacion-requisitos.md`,
> borra las instrucciones en cursiva y los ejemplos, y escribe tu contenido.
> Las reglas de nomenclatura y redacción están en la Guía de redacción de
> requisitos. Se entrega en la **semana 8** junto con el prototipo.

---

**Sistema:**
**Autor:**
**Versión:**
**Fecha de la última actualización:**

---

## 1. Propósito y alcance

**Propósito del documento:** este documento traduce la Visión del
producto de sin-cajero en requisitos concretos y verificables, para
servir de base al prototipo de la semana 8 y a cualquier decisión de
diseño posterior. Va dirigido a quien desarrolle o evalúe el sistema —
en este caso, a mí mismo como desarrollador y a mi dupla como revisor.

**Alcance del sistema:** pedidos desde kiosco en el local y desde el
celular del cliente (en el momento o con anticipación), pantalla de
barra que solo muestra pedidos pagados, pantalla de estado para el
cliente, catálogo de productos y precios administrable por el dueño, y
corte de caja diario que concilia lo cobrado contra lo entregado.

**Fuera del alcance:** pedidos a domicilio, control de inventario de
insumos y materia prima, y facturación fiscal electrónica (CFDI)
automatizada — igual que en la Visión del producto.

---

## 2. Usuarios y su contexto

| Usuario | Qué hace hoy sin el sistema | Qué espera del sistema |
|---|---|---|
| Cliente | Hace fila para ordenar y pagar con un cajero antes de que su pedido empiece a prepararse | Ordenar y pagar sin fila, desde el kiosco o su celular, y saber con certeza cuándo su pedido está listo |
| Barista | Recibe el pedido a viva voz o por escrito del cajero, con margen para ajustes verbales de último momento (cortesías, descuentos) | Ver únicamente pedidos ya pagados, en el orden correcto, sin ambigüedad sobre qué preparar |
| Dueño/administrador | Confía en que el cajero registre cada venta; detecta faltantes hasta el corte de caja o el inventario, cuando ya es tarde para saber la causa | Que ningún producto salga de la barra sin quedar pagado y registrado, y no depender de un cajero de tiempo completo |

**Conflictos identificados entre usuarios:**

1. El barista tiene acceso físico directo al producto (café, leche,
   vasos) y normalmente resuelve situaciones con criterio propio —una
   cortesía, un descuento verbal—. El sistema, para cumplir lo que
   necesita el dueño, tiene que impedir eso: nada se prepara sin un
   pedido pagado asociado. El requisito no puede depender de la buena
   voluntad del barista; tiene que hacerlo estructuralmente imposible
   de saltarse.
2. Un cliente que pide con anticipación y llega tarde deja una bebida
   fría ocupando la barra mientras hay clientes presentes esperando.
   El sistema resuelve esto disparando la preparación cuando el
   cliente confirma su llegada, no a la hora que originalmente eligió.

---

## 3. Requisitos funcionales

### 3.1 Resumen

| ID | Nombre | Prioridad | Origen |
|---|---|---|---|
| RF-001 | Registro de pedido desde kiosco | Imprescindible | Visión del producto |
| RF-002 | Registro de pedido desde celular | Imprescindible | Visión del producto |
| RF-003 | Cálculo de total según catálogo vigente | Imprescindible | Visión del producto |
| RF-004 | Bloqueo de envío a barra sin pago confirmado | Imprescindible | Confirmado por el dueño |
| RF-005 | Disparo de preparación por confirmación de llegada | Importante | Visión del producto |
| RF-006 | Vista de barra solo con pedidos pagados, en orden | Imprescindible | Visión del producto |
| RF-007 | Aviso de pedido listo en pantalla de estado | Importante | Visión del producto |
| RF-008 | Administración de catálogo por el dueño | Imprescindible | Confirmado por el dueño |
| RF-009 | Bloqueo de modificación de precio/descuento por el barista | Imprescindible | Confirmado por el dueño |
| RF-010 | Cancelación de pedido pagado solo con autorización del dueño | Imprescindible | Confirmado por el dueño |
| RF-011 | Corte de caja diario | Importante | Visión del producto |

### 3.2 Fichas

**RF-001 · Registro de pedido desde kiosco**

| Campo | Contenido |
|---|---|
| Descripción | El sistema registra un pedido con los productos, modificadores y método de pago seleccionados en el kiosco del local. |
| Origen | Visión del producto, apartado 3 (alcance). |
| Prioridad | Imprescindible |
| Criterio de aceptación | Al completar un pedido en el kiosco con al menos un producto y un método de pago, el pedido queda registrado con esos datos y un identificador único. Si falta el método de pago, el sistema no lo registra y lo señala. |
| Relacionado con | RF-003, RF-004 |

**RF-002 · Registro de pedido desde celular**

| Campo | Contenido |
|---|---|
| Descripción | El sistema registra un pedido con los productos, modificadores, método de pago y, en su caso, la marca de "anticipado", capturados desde el celular del cliente. |
| Origen | Visión del producto, apartado 3 (alcance). |
| Prioridad | Imprescindible |
| Criterio de aceptación | Al completar un pedido desde el celular, el pedido queda registrado igual que uno de kiosco, con la marca de anticipado activada si el cliente eligió esa opción. |
| Relacionado con | RF-003, RF-005 |

**RF-003 · Cálculo de total según catálogo vigente**

| Campo | Contenido |
|---|---|
| Descripción | El sistema calcula el total del pedido usando los precios del catálogo vigentes en el momento de la orden, antes de solicitar el pago. |
| Origen | Derivado del requisito de que ningún ajuste de precio se haga fuera del sistema (ver conflicto 1, apartado 2). |
| Prioridad | Imprescindible |
| Criterio de aceptación | El total mostrado antes de pagar coincide exactamente con la suma de los precios de catálogo de los productos del pedido, sin excepción manual posible. |
| Relacionado con | RF-001, RF-002, RF-008 |

**RF-004 · Bloqueo de envío a barra sin pago confirmado**

| Campo | Contenido |
|---|---|
| Descripción | El sistema impide que un pedido llegue a la pantalla de barra si su pago no ha sido confirmado. |
| Origen | Confirmado por el dueño — control central del negocio (apartado 2, conflicto 1). |
| Prioridad | Imprescindible |
| Criterio de aceptación | Un pedido sin pago confirmado nunca aparece en la pantalla de barra. Al confirmarse el pago, aparece en un tiempo menor a los definidos en RNF-REN-001. |
| Relacionado con | RF-001, RF-002, RF-006, RNF-SEG-001 |

**RF-005 · Disparo de preparación por confirmación de llegada**

| Campo | Contenido |
|---|---|
| Descripción | El sistema envía un pedido anticipado a la pantalla de barra únicamente cuando el cliente confirma su llegada, no automáticamente a la hora que programó. |
| Origen | Visión del producto, apartado 2 (conflicto entre usuarios). |
| Prioridad | Importante |
| Criterio de aceptación | Un pedido anticipado sin confirmación de llegada no aparece en la pantalla de barra, sin importar que ya haya pasado la hora programada. Al confirmar, aparece de inmediato. |
| Relacionado con | RF-002, RF-006 |

**RF-006 · Vista de barra solo con pedidos pagados, en orden**

| Campo | Contenido |
|---|---|
| Descripción | El sistema muestra en la pantalla de barra únicamente los pedidos pagados, ordenados por el momento en que deben prepararse. |
| Origen | Visión del producto, apartado 3 (alcance). |
| Prioridad | Imprescindible |
| Criterio de aceptación | La pantalla de barra nunca muestra un pedido sin pago confirmado, y el orden mostrado coincide con el orden de disparo de cada pedido (inmediato o por confirmación de llegada). |
| Relacionado con | RF-004, RF-005 |

**RF-007 · Aviso de pedido listo en pantalla de estado**

| Campo | Contenido |
|---|---|
| Descripción | El sistema notifica en la pantalla de estado del local cuándo un pedido queda marcado como listo por el barista. |
| Origen | Visión del producto, apartado 3 (alcance). |
| Prioridad | Importante |
| Criterio de aceptación | Al marcar un pedido como listo desde la estación de barista, el identificador del pedido aparece en la pantalla de estado en un tiempo menor a los definidos en RNF-REN-001. |
| Relacionado con | RF-006 |

**RF-008 · Administración de catálogo por el dueño**

| Campo | Contenido |
|---|---|
| Descripción | El sistema permite al dueño/administrador crear, editar y desactivar productos y precios del catálogo. |
| Origen | Confirmado por el dueño. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Un cambio de precio o disponibilidad hecho por el dueño se refleja en el catálogo visible en kiosco y celular antes del siguiente pedido registrado. |
| Relacionado con | RF-003 |

**RF-009 · Bloqueo de modificación de precio/descuento por el barista**

| Campo | Contenido |
|---|---|
| Descripción | El sistema impide que la estación de barista modifique el precio de un pedido o aplique un descuento. |
| Origen | Confirmado por el dueño (apartado 2, conflicto 1). |
| Prioridad | Imprescindible |
| Criterio de aceptación | La estación de barista no ofrece ninguna acción que altere el monto de un pedido; cualquier intento de hacerlo por fuera del sistema no tiene efecto en el registro. |
| Relacionado con | RF-003, RNF-SEG-001 |

**RF-010 · Cancelación de pedido pagado solo con autorización del dueño**

| Campo | Contenido |
|---|---|
| Descripción | El sistema registra la cancelación de un pedido ya pagado únicamente cuando la autoriza el dueño/administrador. |
| Origen | Confirmado por el dueño (apartado 2, conflicto 1). |
| Prioridad | Imprescindible |
| Criterio de aceptación | Un pedido pagado no puede cancelarse desde la estación de barista. Al cancelarse con autorización del dueño, queda registrado quién la autorizó y cuándo. |
| Relacionado con | RF-004, RNF-SEG-002 |

**RF-011 · Corte de caja diario**

| Campo | Contenido |
|---|---|
| Descripción | El sistema genera un corte de caja diario que concilia el total cobrado contra el número de pedidos entregados. |
| Origen | Visión del producto, apartado 3 (alcance). |
| Prioridad | Importante |
| Criterio de aceptación | Al generarse el corte del día, muestra el total cobrado, el número de pedidos entregados y cualquier diferencia entre ambos, sin intervención manual. |
| Relacionado con | RF-004, RNF-SEG-002 |

---

## 4. Requisitos no funcionales

### 4.1 Resumen

| ID | Atributo | Nombre | Prioridad | Origen |
|---|---|---|---|---|
| RNF-CON-001 | Confiabilidad | Disponibilidad en horario de operación | Imprescindible | Visión del producto |
| RNF-SEG-001 | Seguridad | Integridad pedido-pago | Imprescindible | Confirmado por el dueño |
| RNF-SEG-002 | Seguridad | Trazabilidad de acciones | Imprescindible | Visión del producto |
| RNF-REN-001 | Rendimiento | Tiempo de confirmación de pedido | Importante | Supuesto propio |
| RNF-USA-001 | Usabilidad | Uso del kiosco sin instrucciones | Importante | Supuesto propio |

### 4.2 Fichas

**RNF-CON-001 · Disponibilidad en horario de operación**

| Campo | Contenido |
|---|---|
| Atributo de calidad | Confiabilidad |
| Descripción | El sistema está disponible para tomar y procesar pedidos durante todo el horario de operación del local. |
| Métrica | Tiempo de indisponibilidad no planeada menor a 5 minutos por día de operación. |
| Origen | Visión del producto, apartado 4 (no existe cajero de respaldo). |
| Prioridad | Imprescindible |
| Por qué importa | Sin cajero de respaldo, una caída del sistema detiene por completo la venta del local. |
| Afecta a | RF-001, RF-002 |

**RNF-SEG-001 · Integridad pedido-pago**

| Campo | Contenido |
|---|---|
| Atributo de calidad | Seguridad |
| Descripción | Un pedido solo existe en el sistema como enviable a barra si su pago quedó confirmado; no puede haber pedidos "a medias" entre ambos estados. |
| Métrica | 0% de pedidos visibles en la pantalla de barra sin un pago confirmado asociado, verificable auditando la tabla de pedidos contra la de pagos en cualquier momento. |
| Origen | Confirmado por el dueño (apartado 2, conflicto 1). |
| Prioridad | Imprescindible |
| Por qué importa | Es el control central que evita que salga producto sin quedar registrado como venta. |
| Afecta a | RF-004, RF-006, RF-009 |

**RNF-SEG-002 · Trazabilidad de acciones**

| Campo | Contenido |
|---|---|
| Atributo de calidad | Seguridad |
| Descripción | Toda acción sobre un pedido (creación, pago, entrega, cancelación) queda registrada con quién la realizó y cuándo. |
| Métrica | 100% de los pedidos tienen un registro con marca de tiempo y usuario responsable para cada una de sus etapas, disponible en el corte de caja diario. |
| Origen | Visión del producto, apartado 4 (atributo de trazabilidad). |
| Prioridad | Imprescindible |
| Por qué importa | Sin este registro, una diferencia detectada en el corte de caja no se puede atribuir ni corregir. |
| Afecta a | RF-010, RF-011 |

**RNF-REN-001 · Tiempo de confirmación de pedido**

| Campo | Contenido |
|---|---|
| Atributo de calidad | Rendimiento |
| Descripción | Un pedido pagado aparece en la pantalla de barra en un tiempo corto tras confirmarse el pago. |
| Métrica | Menos de 3 segundos entre la confirmación del pago y la aparición del pedido en la pantalla de barra. |
| Origen | Supuesto propio, pendiente de validar con tiempos reales de operación del local. |
| Prioridad | Importante |
| Por qué importa | En hora pico, un retraso aquí genera el mismo cuello de botella que el sistema busca eliminar. |
| Afecta a | RF-004, RF-006 |

**RNF-USA-001 · Uso del kiosco sin instrucciones**

| Campo | Contenido |
|---|---|
| Atributo de calidad | Usabilidad |
| Descripción | Un cliente que nunca ha usado el kiosco puede completar un pedido simple sin ayuda de otra persona. |
| Métrica | Un cliente de primera vez completa un pedido de un solo producto en menos de 60 segundos, sin intervención de personal del local, en pruebas con al menos 5 usuarios distintos. |
| Origen | Supuesto propio, pendiente de validar con pruebas de usuario. |
| Prioridad | Importante |
| Por qué importa | Si el kiosco necesita ayuda para usarse, el negocio termina necesitando otra vez a alguien parado ahí — justo lo que el sistema quiere evitar. |
| Afecta a | RF-001 |

## 5. Casos de uso

*Se trabajan en la semana 7, después de la entrevista. Cada caso de uso
se relaciona con los requisitos funcionales que realiza.*

---

## 6. Trazabilidad

*Esta tabla es la que hace posible el análisis de impacto de la semana
15. Mantenla actualizada conforme cambien los requisitos.*

| Requisito | Origen | Caso de uso | Elemento del prototipo |
|---|---|---|---|
| RF-001 | Entrevista 15 sep | CU-01 Registrar consulta | Pantalla de consulta |

---

## 7. Registro de cambios

*Cada modificación posterior a la primera versión se anota aquí. Un
requisito eliminado se marca como tal, pero su identificador no se
reutiliza.*

| Fecha | Requisito | Qué cambió | Por qué |
|---|---|---|---|
| | | | |

---

## Antes de entregar

- [ ] Todos los requisitos tienen identificador único y ninguno está repetido
- [ ] Cada requisito expresa una sola idea
- [ ] Cada requisito funcional tiene criterio de aceptación comprobable
- [ ] Cada requisito no funcional tiene una métrica, no solo un adjetivo
- [ ] El campo Origen distingue lo confirmado por el cliente de lo que sigo suponiendo
- [ ] Hay al menos un requisito no funcional por cada atributo de calidad que impone mi tipo de sistema
- [ ] Ningún requisito impone una solución técnica
- [ ] Todos los requisitos caben dentro del alcance declarado
- [ ] La tabla de trazabilidad está completa
- [ ] Mi dupla revisó el documento y su revisión está registrada
- [ ] Borré los ejemplos y las instrucciones en cursiva
