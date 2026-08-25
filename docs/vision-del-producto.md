# Visión del producto

> **Plantilla del curso · Ingeniería de Software I · SIS3407**
> Este documento es el primer entregable del semestre y la base de todo lo que viene después.
> Se entrega completo en la **semana 4** y se presenta ante el grupo.
>
> **Cómo usarla:** copia este archivo a tu repositorio como `docs/vision-del-producto.md`, borra las instrucciones en gris de cada apartado y escribe tu contenido en su lugar. Conserva los títulos.

---

**Autor:*Mateo Ibañez de la Cueva*
**Fecha de la última versión:*18/08/2026*
**Repositorio:*main*

---

## 1. Descripción del sistema

**Nombre del sistema:** sin-cajero

**Descripción:** sin-cajero es un sistema de pedidos para una cafetería que
funciona sin cajero ni mesero. El cliente ordena y paga desde una pantalla
en el local (kiosco) o desde su propio celular, ya sea en el momento o con
anticipación. El pedido llega directo a la barra para que el barista lo
prepare, y una pantalla en el local avisa cuándo está listo para recoger.
El cliente nunca hace fila para pagar; solo se acerca a la barra a recoger
su bebida cuando el sistema le avisa que ya está lista.

---

## 2. Problema y usuarios

**El problema:** una cafetería tradicional depende de que el cajero haga
bien dos cosas a la vez: cobrar rápido para no generar fila, y registrar
en el sistema todo lo que sale de la barra. Cuando el negocio confía esa
segunda parte a la honestidad y atención del cajero, hay una ventana
abierta para que se entregue producto sin cobrarlo o sin registrarlo —ya
sea un descuento no autorizado, un "regalo" a un conocido, o una venta que
se cobra en efectivo y no se captura—. El dueño se entera de esa merma
hasta que hace inventario, si es que lo hace, y para entonces ya no puede
saber si fue un error, un desperdicio o un robo.

**Cómo se resuelve hoy sin el sistema:** el dueño paga un cajero, confía en
que registre cada venta, y hace cortes de caja e inventarios periódicos
para tratar de detectar faltantes. Cuando encuentra una diferencia, ya
pasó demasiado tiempo para saber qué la causó.

**Usuarios del sistema:**

| Tipo de usuario | Qué necesita del sistema | Qué le preocupa |
|---|---|---|
| Cliente | Ordenar y pagar rápido, sin fila, y saber cuándo su pedido está listo | Que le cobren mal, que su pedido se pierda o llegue tarde |
| Barista | Ver los pedidos en el orden correcto para prepararlos sin perder tiempo ni confundirse | Que le lleguen pedidos con modificaciones que no puede cumplir a tiempo, o que se le acumulen sin orden claro |
| Dueño/administrador (quien contrata el servicio) | Que ningún producto salga de la barra sin estar pagado y registrado, y no tener que pagar un cajero de tiempo completo | Perder dinero por ventas no registradas o descuentos que nadie autorizó, sin manera de rastrear quién lo hizo |

**Un conflicto entre usuarios:** el barista prepara las bebidas y tiene
acceso físico directo al producto: café, leche, vasos. El sistema, para
cumplir lo que el dueño necesita, tiene que impedir que el barista prepare
o entregue algo que no tenga un pedido pagado asociado en el sistema —ni
siquiera un descuento "de cortesía" para un cliente frecuente o un amigo.
Eso choca con la manera en que un barista humano normalmente opera: usar
su criterio para regalar una muestra, cambiar un precio a mano o ayudar a
un cliente sin efectivo suficiente. Ahí está la decisión de diseño: el
sistema no puede depender de la buena voluntad del barista para que el
control funcione, tiene que hacer que sea imposible (o al menos quede
registrado) saltarse el sistema, sin volver el trabajo del barista más
lento ni más rígido de lo necesario.

Un segundo conflicto, más operativo: un cliente pide con anticipación para
las 8:00 a.m. pero llega a las 8:20; su bebida se preparó a tiempo y lleva
20 minutos fría en la barra, mientras hay fila de clientes presentes. El
barista decide entre rehacerla o entregarla fría. Esto exige definir si el
sistema dispara la preparación a la hora exacta que eligió el cliente, o
espera a que confirme que ya llegó.
## 3. Alcance

- Toma de pedidos desde kiosco físico en el local.
- Toma de pedidos desde el celular del cliente, en el momento o con anticipación, para recogerse en barra.
- Pantalla para barista/cocina que muestra únicamente los pedidos ya pagados, en el orden en que deben prepararse.
- Pantalla de estado visible en el local que avisa al cliente cuándo su pedido está listo para recoger.
- Catálogo de productos y precios administrable por el dueño.
- Corte de caja diario que concilia el total cobrado por el sistema contra el número de pedidos entregados.

### Explícitamente fuera del alcance

- Pedidos a domicilio (con repartidor, dirección y costo de envío).
- Control de inventario de insumos y materia prima (café, leche, vasos) con niveles de stock.
- Facturación fiscal electrónica (CFDI) automatizada.

**Por qué queda fuera:**

Pedidos a domicilio queda fuera porque introduce un problema distinto al
que resuelve este sistema: logística de última milla (repartidor, radio de
cobertura, tiempo estimado de entrega), no la eliminación de la fila y el
control de lo que sale de la barra. Meterlo triplicaría el sistema sin
aportar al problema central del entregable.

Control de inventario de insumos también queda fuera: el control que
necesita el dueño no depende de saber cuánta leche queda, sino de que
ningún producto salga de la barra sin un pedido pagado asociado.

## 4. Tipo de sistema y restricciones

*Instrucción: identifica de qué tipo es tu sistema y qué te obliga a garantizar ese tipo. Un sistema de información y un sistema crítico no se diseñan igual.*

**Tipo de sistema:** Web y SaaS

**Por qué es de ese tipo:** el sistema tiene varias superficies que
comparten información en tiempo real —kiosco, celular del cliente,
pantalla de barra y pantalla de estado— y no depende de un solo
dispositivo ni de hardware especializado. Además, aunque hoy es para un
solo local, está pensado desde el inicio para operar como servicio hacia
otras cafeterías más adelante, con cada una administrando su propio
catálogo.


**Atributos de calidad que impone:**
| Atributo | Por qué importa en mi caso | Qué pasa si no se cumple |
|---|---|---|
| Disponibilidad en horas pico | No existe cajero de respaldo: si el sistema cae, no hay forma alternativa de vender | El negocio se detiene por completo mientras dure la falla |
| Integridad transaccional (pedido y pago como una sola operación) | Un pedido debe registrarse y cobrarse de forma atómica; si el proceso falla a medias, se puede preparar algo sin cobrar o cobrar algo que nunca llega a barra | Reaparece el mismo problema que el sistema busca eliminar: producto que sale sin quedar registrado |
| Trazabilidad de cada acción (pedido, pago, entrega, cancelación) | El dueño necesita poder rastrear quién hizo qué y cuándo para detectar y probar cualquier diferencia | Sin registro, una merma o fuga no se puede atribuir a nadie ni corregir |

**Reglas de negocio que ya identifiqué:**

1. Ningún producto puede prepararse en barra sin que exista antes un
   pedido pagado y registrado en el sistema; no existen cortesías,
   descuentos ni ajustes de precio capturados fuera de él.
2. Un pedido anticipado no dispara su preparación automáticamente a la
   hora que el cliente eligió al ordenar; requiere una confirmación de
   que el cliente ya llegó o está por llegar, para no desperdiciar
   producto si llega tarde.
3. Un pedido ya pagado no puede cancelarse desde la estación de barista;
   la cancelación (y su eventual reembolso) requiere autorización del
   dueño/administrador, porque cancelar sin reembolsar es otra forma de
   que el producto salga sin quedar registrado como venta.

## 5. Ciclo de vida elegido

*Instrucción: este apartado se trabaja en la semana 3, después de ver los modelos de desarrollo. La justificación pesa más que la elección: no hay un modelo correcto, hay uno defendible para tu caso.*

**Modelo elegido:**

**Por qué le conviene a este proyecto:**

*Instrucción: argumenta con las características reales de tu caso. Estabilidad de los requisitos, disponibilidad del cliente, nivel de riesgo, tamaño del equipo, frecuencia de entregas esperada.*

### Alternativas descartadas

**Alternativa 1:**

*Por qué la descarté:*

**Alternativa 2:**

*Por qué la descarté:*

---

## Antes de entregar

Reviso que el documento cumpla lo siguiente:

- [ ] La descripción del apartado 1 se entiende sin ser del área
- [ ] Hay al menos dos tipos de usuario con necesidades distintas
- [ ] Identifiqué un conflicto real entre usuarios
- [ ] El alcance dice qué queda fuera, no solo qué queda dentro
- [ ] Las exclusiones son específicas, no genéricas
- [ ] Identifiqué el tipo de sistema y al menos dos atributos de calidad
- [ ] Anoté al menos tres reglas de negocio no obvias
- [ ] Justifiqué el ciclo de vida contra dos alternativas descartadas
- [ ] El documento está en mi repositorio y se puede leer desde el navegador
- [ ] Borré todas las instrucciones en cursiva de la plantilla
