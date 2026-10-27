# Ejercicios de Mermaid: 5 diagramas de la guía

Actividad "Diagramas ADR - Ingeniería de Software" (SIS3407). Los cinco ejemplos salen de la Guía rápida de Mermaid y usan el caso de la clínica veterinaria que vimos en clase. De cada uno dejo el código original, mi versión modificada y qué pasó con el dibujo. En cada modificación hice lo mismo: agregué una caja, cambié una etiqueta y quité una flecha.

Los diagramas están escritos como texto de Mermaid para que se puedan comparar entre versiones. GitHub los dibuja directo en esta página, y también se pueden pegar en [mermaid.live](https://mermaid.live).

---

## Ejemplo 4 · Diagrama de flujo con decisiones

Las validaciones de un abono: primero se revisa que el monto sea mayor a cero y luego que no exceda el saldo.

**Original**

```mermaid
flowchart TD
    ini([Inicio])
    val{Monto mayor a cero?}
    exc{Excede el saldo?}
    apl[Aplicar el abono]
    err1[Monto invalido]
    err2[Excede el saldo]
    fin([Fin])

    ini --> val
    val -->|No| err1
    val -->|Si| exc
    exc -->|Si| err2
    exc -->|No| apl
    err1 --> fin
    err2 --> fin
    apl --> fin

    style apl fill:#ff5d00,color:#fff
```

**Modificado**

```mermaid
flowchart TD
    ini([Inicio])
    val{Monto mayor a cero?}
    exc{Excede el saldo?}
    apl[Aplicar el abono]
    bit[Registrar en bitácora]
    err1[Monto inválido]
    err2[Excede el saldo]
    fin([Fin])

    ini --> val
    val -->|No| err1
    val -->|Sí| exc
    exc -->|Sí| err2
    exc -->|No| apl
    apl --> bit
    bit --> fin
    err1 --> fin

    style apl fill:#ff5d00,color:#fff
```

**Qué cambié y qué pasó**

- **Caja nueva:** `bit`, "Registrar en bitácora", entre aplicar el abono y el fin.
- **Etiqueta cambiada:** "Monto invalido" pasó a "Monto inválido" y las ramas "Si" pasaron a "Sí".
- **Flecha quitada:** `err2 --> fin`. La caja "Excede el saldo" quedó sin salida: Mermaid la dibuja igual, pero el flujo ya no termina en ese camino.

---

## Ejemplo 5 · Diagrama de secuencia

Cómo se pasan los mensajes entre las piezas cuando la recepcionista registra un abono.

**Original**

```mermaid
sequenceDiagram
    actor Rec as Recepcionista
    participant Web as App web
    participant Cob as Cobros
    participant Age as Agenda
    participant BD as Base de datos

    Rec->>Web: Registra un abono de 500
    Web->>Cob: abonar(cuenta, 500)
    Cob->>Age: Que servicio se atendio?
    Age-->>Cob: Consulta general
    Cob->>BD: Guarda el pago
    BD-->>Cob: Listo
    Cob-->>Web: Pago registrado
    Web-->>Rec: Muestra el nuevo saldo
```

**Modificado**

```mermaid
sequenceDiagram
    actor Rec as Recepcionista
    participant Web as App web
    participant Cob as Cobros
    participant Age as Agenda
    participant BD as Base de datos
    participant Not as Notificador

    Rec->>Web: Registra un abono de 500
    Web->>Cob: abonar(cuenta, 500)
    Cob->>Age: Que servicio se atendio?
    Age-->>Cob: Cirugía
    Cob->>BD: Guarda el pago
    Cob->>Not: enviarRecibo(500)
    Cob-->>Web: Pago registrado
    Web-->>Rec: Muestra el nuevo saldo
```

**Qué cambié y qué pasó**

- **Pieza nueva:** `Not`, el Notificador, con un mensaje `enviarRecibo(500)` que le manda Cobros.
- **Etiqueta cambiada:** la respuesta de Agenda pasó de "Consulta general" a "Cirugía".
- **Flecha quitada:** la respuesta `BD-->>Cob: Listo`. Ahora Cobros guarda el pago y avisa que quedó registrado sin esperar confirmación de la base de datos. El diagrama lo dibuja sin problema, pero el flujo ya no es confiable.

---

## Ejemplo 6 · Diagrama de clases

Atributos, métodos, visibilidad y multiplicidad de la cuenta de un paciente.

**Original**

```mermaid
classDiagram
    class CuentaDePaciente {
        -saldo: double
        -pagos: List~Pago~
        +abonar(monto) string
        +consultarSaldo() double
    }

    class Pago {
        -monto: double
        -fecha: Date
    }

    class Paciente {
        -nombre: string
        -especie: string
    }

    Paciente "1" --> "1" CuentaDePaciente : tiene
    CuentaDePaciente "1" --> "*" Pago : registra
```

**Modificado**

```mermaid
classDiagram
    class CuentaDePaciente {
        -saldo: double
        -pagos: List~Pago~
        +abonar(monto) string
        +consultarSaldo() double
    }

    class Pago {
        -monto: double
        -fecha: Date
    }

    class Paciente {
        -nombre: string
        -especie: string
    }

    class Recibo {
        -folio: string
        -total: double
        +imprimir() void
    }

    Paciente "1" --> "1" CuentaDePaciente : posee
    Pago "1" --> "1" Recibo : genera
```

**Qué cambié y qué pasó**

- **Clase nueva:** `Recibo`, con folio, total y el método `imprimir()`, conectada a `Pago`.
- **Etiqueta cambiada:** "tiene" pasó a "posee".
- **Flecha quitada:** `CuentaDePaciente "1" --> "*" Pago : registra`. La cuenta sigue teniendo el atributo `pagos`, pero el dibujo ya no muestra la relación con `Pago`. Ese es el tipo de inconsistencia que se ve en el diagrama antes que en el código.

---

## Ejemplo 7 · Diagrama de estados

Por qué estados pasa una cita y qué evento la mueve de uno a otro.

**Original**

```mermaid
stateDiagram-v2
    [*] --> Agendada
    Agendada --> Confirmada: el dueno confirma
    Agendada --> Cancelada: se cancela a tiempo
    Confirmada --> Atendida: llega a consulta
    Confirmada --> NoAsistio: no se presenta
    Atendida --> [*]
    Cancelada --> [*]
    NoAsistio --> [*]
```

**Modificado**

```mermaid
stateDiagram-v2
    [*] --> Agendada
    Agendada --> Confirmada: el dueño confirma
    Agendada --> Cancelada: se cancela a tiempo
    Confirmada --> Reagendada: el cliente pide otro día
    Reagendada --> Confirmada: el dueño confirma
    Confirmada --> Atendida: es atendida
    Confirmada --> NoAsistio: no se presenta
    Atendida --> [*]
    NoAsistio --> [*]
```

**Qué cambié y qué pasó**

- **Estado nuevo:** `Reagendada`, que sale de `Confirmada` y regresa a `Confirmada` cuando el dueño confirma de nuevo.
- **Etiqueta cambiada:** "llega a consulta" pasó a "es atendida" (y "dueno" a "dueño").
- **Flecha quitada:** `Cancelada --> [*]`. El estado Cancelada ya no termina: parece una cita que sigue viva para siempre, aunque nadie pueda hacer nada con ella.

---

## Ejemplo 8 · Diagrama entidad-relación

Las tablas de la base de datos y cómo se relacionan.

**Original**

```mermaid
erDiagram
    PACIENTE ||--o{ CITA : tiene
    PACIENTE ||--|| CUENTA : tiene
    CITA ||--o| EXPEDIENTE : genera
    CUENTA ||--o{ PAGO : registra

    PACIENTE {
        int id
        string nombre
        string especie
    }
    CITA {
        int id
        date fecha
        string motivo
    }
    PAGO {
        int id
        decimal monto
        date fecha
    }
```

**Modificado**

```mermaid
erDiagram
    PACIENTE ||--o{ CITA : tiene
    VETERINARIO ||--o{ CITA : atiende
    CITA ||--o| EXPEDIENTE : documenta
    CUENTA ||--o{ PAGO : registra

    PACIENTE {
        int id
        string nombre
        string especie
    }
    VETERINARIO {
        int id
        string nombre
    }
    CITA {
        int id
        date fecha
        string motivo
    }
    PAGO {
        int id
        decimal monto
        date fecha
    }
```

**Qué cambié y qué pasó**

- **Entidad nueva:** `VETERINARIO`, que atiende muchas citas.
- **Etiqueta cambiada:** la relación entre cita y expediente pasó de "genera" a "documenta".
- **Flecha quitada:** `PACIENTE ||--|| CUENTA : tiene`. La cuenta quedó desconectada del paciente: solo se relaciona con sus pagos, así que no hay forma de saber de quién es.
