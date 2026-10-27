# ADR-001: Monolito modular en lugar de microservicios

## Estado
Aceptada, 20 de octubre de 2026 (registrada el 27 de octubre de 2026)

## Contexto
sin-cajero es un sistema de pedidos para cafeterías sin cajero. Hoy atiende un solo local, con unas 80 a 120 ventas al día, pero está pensado como SaaS para sumar más cafeterías, cada una con su catálogo.

El sistema tiene cuatro módulos con fronteras ya trazadas: Pedidos, Catálogo, Pagos y Caja. Pedidos es el centro y los otros tres solo hablan con él.

Lo construye una sola persona, sin equipo de operación para servicios en red. Las fronteras entre módulos acaban de definirse y todavía no se han probado en uso real, así que podrían moverse.

Los requisitos que pesan en la decisión: RNF-REN-001 (menos de 8 segundos entre el pago y la pantalla de barra), RNF-CON-001 (menos de 5 minutos de caída no planeada por día) y RNF-SEG-002 (cada acción sobre un pedido queda registrada y disponible en el corte de caja).

Se evaluaron tres opciones: un monolito sin separación interna, un monolito modular y microservicios. También se evaluó separar Pagos en un servicio aparte desde el inicio, porque depende de un proveedor externo.

## Decisión
Construimos un monolito modular: un solo desplegable, con los cuatro módulos separados por carpetas e interfaces, y con la regla de que ningún módulo lee las tablas de otro. Lo que un módulo necesita de otro lo pide por la interfaz del módulo dueño.

## Consecuencias
Hay un solo despliegue y un solo registro de errores, que es lo que una persona puede operar hoy. Una llamada entre módulos es una llamada dentro del mismo proceso, así que no suma latencia de red al límite de 8 segundos de RNF-REN-001.

No podemos escalar ni desplegar Pagos por separado. Con 80 a 120 ventas al día no hace falta.

Si algún día hay que separarlo, los límites ya están puestos y el corte es por donde ya cortamos. El primer módulo que sacaríamos es Pagos. La señal para hacerlo: que sus cambios se desplieguen mucho más seguido que los de Catálogo o Caja por cambios en el proveedor, o que las llamadas al proveedor empiecen a acercarse al límite de 8 segundos.

El riesgo real es que alguien lea las tablas de otro módulo por atajo y se pierda el límite. Lo revisamos en cada cambio al repositorio.
