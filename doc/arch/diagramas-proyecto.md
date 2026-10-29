# Diagramas del proyecto — sin-cajero

Cinco diagramas de sin-cajero escritos como texto de Mermaid: flujo, secuencia, clases, estados y entidad-relación. Complementan los tres niveles C4 de `diagramas.md`: aquel dibuja las piezas del sistema y este dibuja cómo se comportan por dentro.

GitHub los dibuja directo en esta página y también se pueden pegar en [mermaid.live](https://mermaid.live). Los requisitos que se mencionan (RF, RNF) están en `docs/especificacion-requisitos.md`, y la decisión de fondo (monolito modular) en `adr-001-monolito-modular.md`.

---

## 1 · Diagrama de flujo: qué pasa cuando se confirma un pago

La regla central del negocio: un pedido solo llega a la barra si está pagado (RF-004), y uno anticipado espera a que el cliente avise que llegó (RF-005).

```mermaid
flowchart TD
    ini([El cliente paga])
    pag{Pago confirmado?}
    rech[Informar que el pago no se completó]
    reg[Registrar el pedido como pagado]
    ant{Es anticipado?}
    esp[Dejar el pedido en espera]
    lleg{El cliente confirma su llegada?}
    bar[Enviar el pedido a la pantalla de barra]
    fin([Fin])

    ini --> pag
    pag -->|No| rech
    pag -->|Sí| reg
    reg --> ant
    ant -->|No| bar
    ant -->|Sí| esp
    esp --> lleg
    lleg -->|No| esp
    lleg -->|Sí| bar
    rech --> fin
    bar --> fin

    style bar fill:#ff5d00,color:#fff
```

Si el pago se rechaza, el pedido no se registra y nunca aparece en la barra (flujo alterno FA-1 del caso de uso CU-02). Un pedido anticipado no sale a la barra por haber pasado su hora, solo por la confirmación de llegada.

---

## 2 · Diagrama de secuencia: un pedido desde el celular, de la orden al aviso de listo

Los mensajes entre las piezas del nivel 3 de los diagramas C4 cuando el cliente ordena desde el celular para ahora (CU-02 y CU-04).

```mermaid
sequenceDiagram
    actor Cli as Cliente
    actor Bar as Barista
    participant App as App de pedidos
    participant Ped as Pedidos
    participant Cat as Catálogo
    participant Pag as Pagos
    participant Pro as Proveedor de pagos
    participant Not as Notificador

    Cli->>App: Elige productos y modificadores
    App->>Ped: crearPedido(productos)
    Ped->>Cat: precios vigentes de los productos
    Cat-->>Ped: precios
    Ped-->>App: total del pedido
    Cli->>App: Confirma y paga
    App->>Ped: pagar(pedido)
    Ped->>Pag: cobrar(pedido, total)
    Pag->>Pro: solicita el cobro
    Pro-->>Pag: cobro confirmado
    Pag-->>Ped: pago confirmado
    Ped->>Not: el pedido está pagado
    Not-->>Bar: aparece en la pantalla de barra
    Bar->>Ped: marca el pedido como listo
    Ped->>Not: el pedido está listo
    Not-->>Cli: aparece en la pantalla de estado
```

Cada flecha entre módulos es una llamada a la interfaz del módulo dueño, no una lectura de sus tablas (regla del ADR-001). El tiempo entre "cobro confirmado" y "aparece en la pantalla de barra" es el que limita RNF-REN-001: menos de 8 segundos.

---

## 3 · Diagrama de clases: el pedido y sus reglas

Las clases del módulo Pedidos y lo que lo rodea. Se usan tres patrones: una estrategia para decidir cuándo sale el pedido a la barra, observadores para avisar los cambios de estado (RF-007) y una interfaz para el cobro, de modo que cambiar de proveedor de pagos solo cambie el adaptador.

```mermaid
classDiagram
    class Pedido {
        -id: string
        -estado: EstadoPedido
        -anticipado: boolean
        -total: double
        +agregarProducto(producto, cantidad) void
        +pagar() boolean
        +confirmarLlegada() void
        +marcarListo() void
        +cancelar(autorizadoPor) void
    }

    class LineaPedido {
        -cantidad: int
        -precioUnitario: double
        -modificadores: List~string~
    }

    class Producto {
        -nombre: string
        -precio: double
        -activo: boolean
    }

    class EstrategiaEnvio {
        <<interface>>
        +cuandoEnviar(pedido) boolean
    }

    class EnvioInmediato {
        +cuandoEnviar(pedido) boolean
    }

    class EnvioPorLlegada {
        +cuandoEnviar(pedido) boolean
    }

    class ObservadorPedido {
        <<interface>>
        +cambioDeEstado(pedido) void
    }

    class PantallaBarra {
        +cambioDeEstado(pedido) void
    }

    class PantallaEstado {
        +cambioDeEstado(pedido) void
    }

    class ProcesadorPago {
        <<interface>>
        +cobrar(monto) boolean
    }

    class AdaptadorProveedor {
        +cobrar(monto) boolean
    }

    class AccionPedido {
        -tipo: string
        -actor: string
        -fechaHora: Date
    }

    Pedido "1" --> "*" LineaPedido : contiene
    LineaPedido "*" --> "1" Producto : se basa en
    Pedido "1" --> "1" EstrategiaEnvio : usa
    EstrategiaEnvio <|.. EnvioInmediato
    EstrategiaEnvio <|.. EnvioPorLlegada
    Pedido "1" --> "*" ObservadorPedido : avisa a
    ObservadorPedido <|.. PantallaBarra
    ObservadorPedido <|.. PantallaEstado
    Pedido ..> ProcesadorPago : cobra con
    ProcesadorPago <|.. AdaptadorProveedor
    Pedido "1" --> "*" AccionPedido : registra
```

`precioUnitario` en la línea guarda el precio del catálogo vigente al momento de la orden, así que un cambio posterior de precio no altera un pedido ya hecho (RF-003). `AccionPedido` es la bitácora que pide RNF-SEG-002 y de donde sale el corte de caja.

---

## 4 · Diagrama de estados: la vida de un pedido

Por qué estados pasa un pedido y qué evento lo mueve de uno a otro.

```mermaid
stateDiagram-v2
    [*] --> EnCaptura
    EnCaptura --> EsperandoPago: el cliente confirma el pedido
    EsperandoPago --> EnCaptura: el pago es rechazado
    EsperandoPago --> Pagado: el pago se confirma
    state anticipado <<choice>>
    Pagado --> anticipado
    anticipado --> EnBarra: es para ahora
    anticipado --> EsperandoLlegada: es anticipado
    EsperandoLlegada --> EnBarra: el cliente confirma su llegada
    EnBarra --> Listo: el barista lo marca como listo
    Listo --> Entregado: el cliente lo recoge
    Pagado --> Cancelado: el dueño autoriza la cancelación
    EsperandoLlegada --> Cancelado: el dueño autoriza la cancelación
    EnBarra --> Cancelado: el dueño autoriza la cancelación
    Entregado --> [*]
    Cancelado --> [*]
```

Un pedido solo llega a `EnBarra` desde `Pagado` o `EsperandoLlegada`, nunca desde `EnCaptura` ni `EsperandoPago` (RF-004, RNF-SEG-001). La única salida de un pedido ya pagado, además de entregarse, es `Cancelado`, y solo con autorización del dueño (RF-010). El barista no tiene ninguna transición que cambie el monto (RF-009).

---

## 5 · Diagrama entidad-relación: la base de datos

Las tablas de una cafetería y cómo se relacionan. No hay una tabla de cafeterías porque el nivel 2 define un esquema de base de datos por cafetería: lo que aísla a una de otra es el esquema, no una columna.

```mermaid
erDiagram
    PRODUCTO ||--o{ LINEA_PEDIDO : "se vende en"
    PEDIDO ||--|{ LINEA_PEDIDO : contiene
    PEDIDO ||--o| PAGO : "se cobra con"
    PEDIDO ||--o{ ACCION_PEDIDO : registra
    CORTE_CAJA ||--o{ PEDIDO : concilia

    PRODUCTO {
        int id PK
        string nombre
        decimal precio
        boolean activo
    }
    PEDIDO {
        int id PK
        string estado
        boolean anticipado
        decimal total
        datetime creado
        int corte_id FK
    }
    LINEA_PEDIDO {
        int id PK
        int pedido_id FK
        int producto_id FK
        int cantidad
        decimal precio_unitario
        string modificadores
    }
    PAGO {
        int id PK
        int pedido_id FK
        decimal monto
        string metodo
        string estado
        datetime confirmado
    }
    ACCION_PEDIDO {
        int id PK
        int pedido_id FK
        string tipo
        string actor
        datetime fecha_hora
    }
    CORTE_CAJA {
        int id PK
        date fecha
        decimal total_cobrado
        int pedidos_entregados
        decimal diferencia
    }
```

Cada tabla tiene un solo módulo dueño, y es el único que escribe en ella: Catálogo (`PRODUCTO`), Pedidos (`PEDIDO`, `LINEA_PEDIDO`), Pagos (`PAGO`) y Caja (`ACCION_PEDIDO`, `CORTE_CAJA`). Los demás módulos piden lo que necesitan por la interfaz del dueño, como dice el ADR-001. `PAGO` es opcional respecto de `PEDIDO` porque un pedido nace antes de cobrarse, y la auditoría de RNF-SEG-001 se hace cruzando `PEDIDO` con `PAGO`.
