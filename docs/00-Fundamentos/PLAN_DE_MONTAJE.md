# Plan de montaje: entrada obligatoria y su procedencia

> Cierra el hueco #70. Resuelve la pregunta central: **qué hace el sistema cuando el cliente no
> entrega plan de montaje.**
> **2026-08-11** · insumo de C1.1 (#2) y C1.3 (#4), habilita el congelamiento de C1.6 (#7).

## El problema

**Inv. 11:** el plan de montaje del cliente **es entrada, no consecuencia**. `plan_montaje_t_sem` es
campo *required* de `e6_especificaciones`. Lotes, secuencia y memoria se calculan hacia atrás para
cumplirlo **contra la planta cargada**, y el déficit se resuelve explícito. Sin él no hay S9, ni S11,
ni S12 → no hay `capitulo_financiero` → **no hay `precio_licitacion`**.

El conflicto es con la realidad: **el caso real del propio consultor no tiene plan de montaje.**
CNARCCS Domo cotiza *«Montaje: NO INCLUIDO en este alcance»*, y el montaje se cotiza aparte a
solicitud del cliente. Si el campo es obligatorio y ese caso no lo tiene, o el campo no es
obligatorio o hay algo mal planteado.

## La observación que resuelve el conflicto

CNARCCS **no carece de ritmo**. Carece de *plan de montaje*, pero tiene un ritmo de entrega
perfectamente definido y contractual:

- Plazo de fabricación: **24 a 26 semanas**
- Hitos de pago contra avance: **Sem 0 (anticipo 30%) · Sem 10 (50%) · Sem 20 (100%) · Sem 26 (finiquito)**
- Entrega en Puerto de Progreso, al costado del buque

La planta tiene que servir ese ritmo exactamente igual que serviría un plan de montaje. Lo que el
motor necesita de E6 no es «el plan de montaje» como documento: es **el ritmo de demanda en t/semana
que la planta debe satisfacer**. El plan de montaje es su fuente canónica, no la única posible.

## Resolución

**`plan_montaje_t_sem` sigue siendo *required*.** No se relaja el invariante. Lo que se añade es la
**procedencia** de ese ritmo, para que el sistema sepa cuánta confianza merece y lo declare.

```json
"e6_especificaciones": {
  "plan_montaje_t_sem": [22, 38, 24, 21, 10],
  "plan_montaje_meta": {
    "procedencia":     "cliente | despacho | derivado_de_plazo",
    "fuente":          "Programa de montaje CNARCCS rev. 2, 2026-07-18",
    "confirmado_por":  "J. Pérez",
    "fecha":           "2026-08-11"
  }
}
```

| Procedencia | Qué es | Cuándo aplica | Confianza |
|---|---|---|---|
| **`cliente`** | Plan de montaje entregado por el cliente | Caso canónico del inv. 11 | Alta — el invariante se cumple en su forma pura |
| **`despacho`** | No hay montaje contratado, pero sí un calendario de entrega o despacho comprometido | **Caso CNARCCS**: montaje excluido, 26 semanas con hitos e entrega en puerto | Alta — es contractual, solo cambia quién monta |
| **`derivado_de_plazo`** | No hay plan ni calendario; la plataforma deriva el ritmo del plazo pedido | Anteproyecto, licitación con plazo global sin desglose | **Baja — es un supuesto de la plataforma** |

Reglas asociadas:

1. **`confirmado_por` y `fecha` son obligatorios cuando `procedencia != "cliente"`.** Si el ritmo no
   viene del cliente, alguien de la casa se hace responsable de él. Sin ese registro no se emite.
2. **`derivado_de_plazo` entra al RSS** como componente de incertidumbre adicional. El semáforo debe
   reflejar que el ritmo es un supuesto, no un dato — es exactamente el tipo de partida con rango que
   el inv. 7 manda agregar por raíz de suma de cuadrados.
3. **Nota de alcance adicional en la emisión.** Con `procedencia != "cliente"`, la oferta lleva junto
   a la leyenda obligatoria una segunda línea declarando que el ritmo de fabricación asumido no
   proviene del cliente. Misma lógica que la leyenda del inv. 6: lo que es supuesto se declara.
4. **Nunca se deriva en silencio.** Si la plataforma calcula el ritmo, lo dice en el intake, lo
   muestra editable y lo marca en toda salida que lo consuma (S9, S11, S12 y el capítulo financiero).

## Cómo se deriva cuando `procedencia = "derivado_de_plazo"`

La plataforma propone y el usuario edita — nunca al revés. Propuesta por defecto: **rampa
trapezoidal** sobre el plazo pedido (arranque y cierre al 60% del ritmo de meseta), no un reparto
uniforme, porque un reparto plano subestima el pico y el pico es justo lo que S11 tiene que cubrir.

El usuario puede sustituirla por reparto uniforme o por una curva propia. Cualquier edición manual
sube la procedencia a `despacho` si queda respaldada por un calendario, o la mantiene en
`derivado_de_plazo` si sigue siendo un supuesto interno.

## Captura en el intake

Ya especificada en `docs/Diseño de producto/RECONCILIACION_DISENO_Mockup.md` (P3, paso 4) e
implementada en el prototipo. Lo que añade esta resolución a esa spec:

- **Selector de procedencia** de tres opciones al encabezar el bloque, con la de `cliente` por
  defecto.
- Con `despacho`: campos de fuente del calendario y responsable.
- Con `derivado_de_plazo`: campo de plazo pedido, botón **«Derivar ritmo»**, aviso ámbar de que el
  ritmo es un supuesto de la plataforma, y la tabla queda editable.
- La tarjeta de derivados en vivo ya muestra **Pico t/sem** en ámbar cuando supera la velocidad con
  sobretiempo. Ese aviso es el que conecta con `INV-12a`.

## Impacto en el contrato

| Elemento | Cambio |
|---|---|
| `input_schema.json` E6 | **Aditivo**: nuevo objeto `plan_montaje_meta`. `plan_montaje_t_sem` sin cambios |
| `POST /session/{id}/generate` | Acepta `plan_montaje_meta`; sin cambio de firma |
| `emission.blocked` | Nuevo motivo: `ritmo_no_confirmado` (falta `confirmado_por` con procedencia ≠ cliente) |
| `rss.componentes[]` | Puede incluir el componente de incertidumbre del ritmo derivado |
| Ciclo de vida | Sin cambios |

**Todo aditivo. El contrato v0 puede congelarse con esto.**

## Lo que sigue abierto — y por qué no bloquea

Una pregunta, y es de dominio, no de forma:

> **¿Un calendario de despacho satisface el inv. 11 a ojos del consultor?**
> El invariante dice «plan de montaje del cliente». La lectura que sostiene esta resolución es que
> lo que el motor necesita es el **ritmo de demanda**, y que un calendario de entrega contractual lo
> es. Es una interpretación de **su** invariante, así que la confirma él.

No bloquea el congelamiento porque **la forma del payload no depende de la respuesta**: si Daniel
dice que `despacho` no satisface el inv. 11, lo que cambia es la política —esos proyectos pasarían a
`derivado_de_plazo` con su penalización de RSS— no el esquema.

Escalado a J1.3 (#64) como punto para el sync, y listado entre los documentos e insumos pendientes
del consultor.
