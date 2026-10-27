# Diagramas de arquitectura — sin-cajero

Tres niveles del modelo C4, escritos como texto de Mermaid (se versionan igual que el código). GitHub los dibuja directo en esta página; también se pueden pegar en [mermaid.live](https://mermaid.live).

Los módulos y las reglas salen de `docs/especificacion-requisitos.md`; la decisión de fondo (monolito modular) está en `adr-001-monolito-modular.md`.

---

## Nivel 1 · Diagrama de contexto

El sistema completo como una sola caja, quién lo usa y con qué sistema de fuera habla. Sin tecnología.

```mermaid
flowchart TD
cli([Cliente])
bar([Barista])
due([Dueño o administrador])
sis[sin-cajero<br/>Sistema de pedidos sin cajero]
pagos[Proveedor de pagos]

cli -->|ordena y paga| sis
sis -->|avisa que el pedido está listo| cli
bar -->|prepara los pedidos pagados| sis
due -->|administra el catálogo, autoriza cancelaciones y revisa el corte de caja| sis
sis -->|solicita y confirma cobros| pagos

style sis fill:#ff5d00,color:#fff
```

El cliente nunca pasa por un cajero: ordena y paga solo. El barista solo ve pedidos ya pagados (RF-004, RF-006) y el dueño es el único que cancela un pedido pagado (RF-010).

---

## Nivel 2 · Diagrama de contenedores

Las piezas que se despliegan por separado, con su tecnología. El `subgraph` es la frontera: lo de adentro es sin-cajero, lo de afuera no.

```mermaid
flowchart TD
cli([Cliente])
bar([Barista])
due([Dueño o administrador])

subgraph sistema["sin-cajero"]
  apps["Aplicación de pedidos<br/>Web, kiosco y celular"]
  panel["Pantallas del local<br/>Web, barra y estado"]
  adm["Panel del dueño<br/>Web"]
  api["API<br/>Monolito modular"]
  db[("Base de datos relacional<br/>un esquema por cafetería")]
end

pagos[Proveedor de pagos]

cli -->|ordena y paga| apps
cli -->|mira si su pedido está listo| panel
bar -->|ve y marca pedidos| panel
due -->|administra| adm
apps -->|HTTPS/JSON| api
panel -->|HTTPS/JSON y eventos en tiempo real| api
adm -->|HTTPS/JSON| api
api -->|lee y escribe| db
api -->|HTTPS| pagos

style api fill:#ff5d00,color:#fff
```

Hay un solo backend (la API) porque la arquitectura elegida es un monolito modular. Las dos pantallas del local reciben los cambios por eventos para cumplir RNF-REN-001 (menos de 8 segundos entre el pago y la barra). La base usa un esquema por cafetería para aislar el corte de caja de cada una (RNF-SEG-002).

---

## Nivel 3 · Diagrama de componentes

Lo que hay dentro de la API, que es el contenedor donde viven las reglas de negocio.

```mermaid
flowchart TD
front["Aplicaciones y pantallas"]

subgraph api["API (monolito modular)"]
  pedidos["Pedidos<br/>pedido, estrategia de envío a barra y cancelación autorizada"]
  catalogo["Catálogo<br/>productos y precios vigentes"]
  pagos["Pagos<br/>cobro y confirmación"]
  caja["Caja<br/>corte de caja y bitácora de acciones"]
  notif["Notificador de estado<br/>observadores del pedido"]
  adapt["Adaptador de pagos<br/>interfaz hacia el proveedor"]
end

proveedor[Proveedor de pagos]

front --> pedidos
front --> catalogo
pedidos -->|consulta los precios vigentes| catalogo
pedidos -->|pide confirmar el cobro| pagos
pedidos -->|anuncia cada cambio de estado| notif
caja -->|pide los pedidos entregados y cancelados| pedidos
pagos -->|cobra por la interfaz| adapt
adapt -->|HTTPS| proveedor
notif -->|actualiza las pantallas| front

style pedidos fill:#ff5d00,color:#fff
```

Cómo leerlo:

- **Pedidos** es el centro del sistema; Catálogo, Pagos y Caja solo hablan con Pedidos y no entre sí.
- Ningún módulo lee las tablas de otro: cada uno escribe solo en las suyas, dentro de la base de datos del nivel 2, y pide lo demás por la interfaz del módulo dueño (regla de ADR-001).
- El **Notificador** es el Observer de RF-007: Pedidos anuncia el cambio de estado y las pantallas reaccionan sin que Pedidos las conozca.
- El **Adaptador de pagos** está como interfaz, no con el nombre de un proveedor: si el proveedor cambia, solo cambia el adaptador.
