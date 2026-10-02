# Guion de entrevista — sin-cajero

**Sistema:** sin-cajero
**Entrevistador:** Mateo Ibañez de la Cueva
**Entrevistado (dupla, en rol de dueño de cafetería):** Pablo andrade
**Fecha de la entrevista:** 29/09/2026

---

## Ficha de dominio entregada a la dupla

> Vas a representar al dueño de una cafetería de tamaño mediano (unas 80–120
> ventas diarias), operando desde hace un par de años en una zona con
> tráfico peatonal constante, con un local pequeño (sin mucho espacio para
> filas largas). Tienes 2–3 baristas y normalmente un cajero en turno.
> Conoces bien el negocio del día a día — cómo se mueve la fila, cómo se
> hace el corte de caja, cómo capacitas gente nueva y qué problemas ya has
> tenido con dinero, tiempo o pedidos mal entendidos — pero no sabes nada
> de sistemas ni de tecnología, y no tienes por qué mencionar soluciones
> técnicas en tus respuestas. Responde con lo que tú, como dueño real de
> ese negocio, sabrías y habrías vivido — no intentes adivinar qué es lo
> que Mateo necesita escuchar.

---

## 1. Contexto (preguntas amplias, antes de hablar del sistema)

**P1. Cuéntame cómo es un día normal en tu cafetería, desde que abres hasta que cierras — ¿cómo se reparte la gente y quién hace qué?**

El dueño abre a las 7:00, él mismo hace el corte de caja del día anterior mientras el primer barista prepara la máquina. De 7:30 a 10:00 hay una ola fuerte de gente que va de camino al trabajo; ahí pone a un segundo barista y un cajero. En la tarde baja el ritmo y se queda solo una persona en caja y una en barra.

**P2. ¿Cómo decides qué contratar o cambiar en tu operación cuando algo no está funcionando bien?**

Dice que normalmente reacciona tarde: cambia algo hasta que el problema ya le costó dinero o un cliente se quejó varias veces, porque no lleva números detallados del día a día, solo el corte de caja general.

## 2. Cómo resuelve hoy el problema sin el sistema (ejemplos concretos)

**P3. Piensa en el último viernes o sábado en hora pico: cuéntame exactamente qué pasó desde que entró el primer cliente de la fila hasta que se le entregó su bebida.**

Describe una fila de 12 personas a las 8:40 de la mañana. El cajero tardó casi 10 minutos en cobrarle a todos porque dos clientes pidieron modificaciones (leche de avena, extra shot) que el cajero tuvo que anotar a mano y pasarle al barista en voz alta; uno de esos pedidos se le "traspapeló" y el cliente tuvo que reclamar.

**P4. Cuéntame de una vez reciente en que notaste que faltaba dinero o producto al final del día — ¿qué hiciste para averiguar qué había pasado?**

Cuenta que hace unos meses el corte de caja quedó $340 pesos corto. Revisó el rollo de la caja contra los pedidos que recordaba el barista, pero no pudo reconstruir qué pasó; sospecha que un cajero regaló una bebida a un conocido sin cobrarla, pero nunca lo pudo comprobar y no volvió a pasar (o al menos no se volvió a dar cuenta).

**P5. Describe la última vez que tuviste que capacitar a un cajero nuevo: ¿qué le explicaste primero y qué fue lo que más le costó aprender?**

Lo primero que le explica es el tabulador de precios y combos. Lo que más le cuesta a la gente nueva es manejar los "regalos" y cortesías del dueño: cuándo sí están autorizados y cuándo no, porque no hay una regla escrita, es "a su criterio".

## 3. Excepciones

**P6. Cuéntame de una vez que un cliente pidió algo con anticipación (por ejemplo, llamó antes o dejó encargado un pedido) y luego llegó mucho más tarde de lo esperado — ¿qué pasó con su pedido?**

Dice que esto pasa casi a diario con pedidos telefónicos: el barista prepara el café a la hora que le dijeron, y si el cliente se tarda, el café se queda enfriándose en la barra 10 o 15 minutos. El barista normalmente espera a ver al cliente entrar para rehacerlo si ya se enfrió mucho, pero eso le quita tiempo a los demás pedidos en fila.

**P7. Describe una vez que el sistema de cobro o la terminal fallaron a media operación — ¿qué hizo tu equipo mientras tanto?**

Cuenta que cuando la terminal de tarjeta se cae, simplemente cobran en efectivo y anotan en un papel los pedidos con tarjeta pendientes de capturar después, lo cual casi nunca se termina de registrar bien.

## 4. Verificación de supuestos (requisitos no funcionales marcados como "supuesto propio")

**P8. Si contrataras a alguien completamente nuevo para tu cafetería y lo pusieras frente al kiosco sin explicarle nada, ¿qué tan bien crees que le iría pidiendo un café? ¿Qué parte le costaría más?**

El dueño piensa que un cliente de primera vez tardaría bastante más de un minuto, sobre todo si el menú tiene submenús de personalización (tamaño, leche, extras); calcula que lo normal sería cerca de un minuto y medio para un pedido con una personalización, no para "un producto simple" como se había supuesto.

**P9. Cuando un pedido ya está pagado, ¿cuánto tiempo consideras razonable que tarde en aparecerle al barista en pantalla antes de que empiece a sentirse un retraso?**

Dice que, viendo cómo trabaja hoy su barista (recibe el pedido a viva voz casi al instante), tolerarían hasta 5-8 segundos sin notarlo, pero que si pasara de ahí en hora pico sí lo notarían porque se les acumula la fila de pantalla.

---

## Bitácora de la entrevista

**Supuestos que resultaron falsos o había que ajustar:**

- **RNF-USA-001 (uso del kiosco sin instrucciones):** se había supuesto que un cliente nuevo completa un pedido simple en menos de 60 segundos. La entrevista muestra que ese tiempo es optimista para un pedido con personalización (el caso más común, no la excepción): el dueño calcula más cerca de 90 segundos. Se ajustó la métrica del requisito de 60 a 90 segundos y se aclaró que aplica a un pedido con al menos una personalización, no solo al caso más simple posible.
- **RNF-REN-001 (tiempo de confirmación de pedido):** se había supuesto un límite de 3 segundos por comparación con sistemas comerciales típicos. La entrevista reveló que el barista hoy recibe los pedidos verbales casi instantáneamente y tolera hasta 5-8 segundos sin que se note como demora. Se relajó la métrica de 3 a 8 segundos, que es más realista y sigue siendo exigente en hora pico.

**Algo inesperado que apareció durante la entrevista:**

La entrevista reveló un caso no considerado en el alcance original: cuando la terminal de pago falla, el negocio sigue cobrando en efectivo y apunta a mano los pedidos con tarjeta pendientes de capturar — un modo de "degradación manual" que hoy nunca se termina de registrar bien. Esto no cambia el alcance de esta unidad (sigue fuera de los requisitos actuales, que no contemplan terminales de pago físicas ni su falla), pero queda anotado como una pregunta abierta para la fase de diseño: qué hace el sistema cuando el método de pago falla a medio pedido.

También confirmó, sin cambios, el supuesto más importante del documento: que la pérdida de dinero por ventas no registradas (el corte de caja corto de $340 pesos) es un problema real que ya le ha costado dinero, y que la falta de una regla escrita sobre cortesías es precisamente la brecha que RF-009 (bloqueo de modificación de precio/descuento por el barista) busca cerrar.
