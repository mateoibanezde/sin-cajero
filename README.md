# sin-cajero

Sistema de pedidos digitales para cafetería self-service. El cliente ordena
desde un kiosco en el local o desde su propio celular — sin cajero ni
mesero de por medio — y recoge directamente en barra cuando su pedido está
listo.

## Qué hace

- Toma de pedidos desde kiosco en tienda o desde el celular del cliente
(en local o con anticipación).
- Cobro integrado al pedido (tarjeta o efectivo según el punto).
- Envío de la orden a cocina/barra para preparación.
- Pantalla de estado en el local que avisa cuándo el pedido está listo
para recoger.

## Por qué existe

Elimina la fila de caja y el tiempo del cajero tomando pedidos, sin perder
el trato humano en la preparación: el barista sigue preparando cada bebida,
el sistema solo se encarga de capturar y cobrar el pedido.

## Documentación

- [Visión del producto](docs/vision-del-producto.md)
- [Especificación de requisitos](docs/especificacion-requisitos.md)
- [Guion de entrevista](docs/guion-entrevista.md)
- [Diagrama de casos de uso](docs/diagramas/casos-de-uso.png) ([.drawio editable](docs/diagramas/casos-de-uso.drawio))
- [Prototipo navegable en Figma](https://www.figma.com/design/L5hViUL6EQu4IMeK0YjTly)
- [Arquitectura en C4: contexto, contenedores y componentes](doc/arch/diagramas.md)
- [ADR-001: monolito modular en lugar de microservicios](doc/arch/adr-001-monolito-modular.md)
- [Modelo UML: flujo, secuencia, clases, estados y entidad-relación](doc/arch/diagramas-proyecto.md)
- [Ejercicios de Mermaid (actividad Diagramas ADR)](doc/arch/ejercicios-mermaid.md)
- [Solicitud de cambio SC-01](doc/cambios/SC-01-solicitud-propina.md) y su [análisis de impacto](doc/cambios/SC-01-analisis-impacto.md)

## Estado

Paquete de diseño completo (curso Ingeniería de Software I, Universidad
Anáhuac Querétaro): Visión del producto, Especificación de requisitos
(con casos de uso y trazabilidad), guion de entrevista con la dupla,
diagrama de casos de uso, prototipo navegable en Figma, arquitectura en
C4 con su registro de decisión, modelo UML en Mermaid y análisis de
impacto de la solicitud de cambio SC-01.
