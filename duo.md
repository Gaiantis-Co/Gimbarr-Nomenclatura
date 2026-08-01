# Variante Duo

## Definición

La variante **Duo** de un movimiento es su versión invertida: la misma figura ejecutada con las **manos invertidas** (la mano dominante hace lo que hacía la duo y viceversa).

No todos los movimientos tienen Duo. Solo existe cuando la figura es **asimétrica respecto a las manos**: si durante la ejecución las manos cambian de posición, la figura puede hacerse de forma inversa y esa inversión produce una ejecución distinta (la variante Duo).

## Regla de detección

Un movimiento tiene variante Duo si cumple **cualquiera** de estas condiciones, evaluadas sobre sus etapas (`inicio` → `ejecución N` → `final`):

| # | Condición | Ejemplo |
|---|---|---|
| A | **Inicia con una sola mano**: la etapa `inicio` tiene alguna mano con agarre `sin_agarre` (la otra mano se incorpora durante la ejecución) | Figuras que arrancan con una mano libre |
| B | **Cambio vs. inicio**: para alguna mano (dominante o duo), una etapa posterior tiene un agarre distinto al que esa misma mano tenía en `inicio` (incluye soltar la barra y retomarla) | `Anclar`, `Entrala de ladrón` |

Si **ninguna** condición se cumple, el movimiento **no tiene Duo**: las manos mantienen su agarre en todas las etapas y arrancan ambas sobre la barra.

### Contraejemplo canónico

**Dominada común**: inicio con dos manos en prono, mismo agarre en todas las etapas → no tiene Duo (su versión invertida sería idéntica a la figura original).

### Fuera del alcance de la regla

- El **ancho de agarre** (cerrado/normal/abierto) es una propiedad de la etapa, no de cada mano, y no participa en la detección.
- Movimientos sin datos de etapas quedan en estado "sin datos": el detector no los marca.

## Construcción del Duo

La variante Duo se construye desde las etapas de la figura base:

1. En **cada etapa**, se intercambian los agarres `dominante ↔ duo`.
2. El resto de la etapa (tipo, nombre, ancho, criterios, reglas, multimedia) se conserva.
3. Nombre del Duo: `<nombre de la base> Duo`.
4. La variante se conecta a la base mediante la tabla `variaciones` con descripción `Duo (manos invertidas)`.

El Duo es **involutivo**: el Duo del Duo restaura la figura original.

## Propagación

La regla y la construcción del Duo aplican a **cada nodo del árbol de variantes**: si un movimiento tiene Duo, cada variante de ese movimiento también tiene el suyo (desde el movimiento hasta cada variante).

Al **activar el Duo de un base B**, se materializa el Duo del árbol:

1. Se crea **B-Duo** y se enlaza `B → B-Duo`.
2. Para **cada variante V de B** (detecte o no Duo por sí misma):
   - Se crea **V-Duo** (o se reutiliza si ya existe, verificado por `es_duo_de`).
   - Se enlazan `V → V-Duo` y `B-Duo → V-Duo`.
3. Al crear una variante V del base B desde el wizard, si V tiene Duo y B ya tiene B-Duo, se enlaza también `B-Duo → V-Duo`.

Las **evoluciones no tienen Duo**: quedan solo en el lado normal.

## Navegación Duo

Los nodos Duo no son figuras independientes:

- **No aparecen en el mapa** de movimientos (el backend excluye de `/nodos/` todo nodo que sea variante o tenga el marcador `es_duo_de`).
- Se navegan desde el **modal del movimiento base**:
  - Botón con el ícono de Duo (SVG) en el header del modal que alterna entre el lado normal y el lado Duo (toggle manual).
  - En la vista Duo, el centro muestra el nombre del Duo (ej. "Escuadra Duo") con estilo Duo (borde punteado e insignia con el ícono de Duo), **sin** los botones "Proponer Variante" / "Forjar Evolución".
  - La sección **Variantes** muestra las variantes del Duo (los Duos materializados de las variantes del lado normal).
  - Clic en una variante Duo desde cualquier vista entra en la vista Duo de esa variante (recursivo); el botón de Duo regresa a la vista anterior.
- El panel de administración del Duo (Activar/Descartar/Eliminar) solo aparece en el lado normal.

## Manejo de falsos positivos (administración)

La detección es automática y puede equivocarse. Por eso el estado del Duo se administra manualmente, **no en la creación** (el wizard solo informa), sino en la vista de administración del movimiento:

| Estado | Significado | Acciones disponibles |
|---|---|---|
| `auto` (detectado) | El detector marcó el movimiento con Duo y aún no se ha decidido | **Activar Duo** (crea la variante Duo y la conecta) / **Descartar** (falso positivo: no vuelve a sugerirse) |
| `descartado` | El administrador descartó el Duo (falso positivo) | **Añadir Duo manualmente** / **Reactivar detección** |
| `activado` | La variante Duo existe y está vinculada | **Eliminar Duo** (desvincula `B → B-Duo` y elimina en cascada el árbol Duo: B-Duo y sus V-Duo) |

**Eliminación en cascada:** al eliminar un movimiento **B** (mapa o admin) se borran también todas sus variantes (V), su **B-Duo** y los **V-Duo** — el subárbol completo de variantes, en cascada recursiva. Las **evoluciones hijas no se borran**.

El estado se persiste en `movimientos.datos_extra.duo` con valores `auto | activado | descartado`, y la variante Duo se identifica por `datos_extra.es_duo_de` (id del movimiento base, formato `m{id}`). Este marcador es la única fuente de verdad para identificar un Duo.

## Implementación

- **Frontend**: `Gimbarr/src/utils/duo.ts` — `detectaDuo(etapas)`, `crearDuo(etapas)`, `getNombreDuo(nombre)`, `encontrarVarianteDuo(variaciones, baseId)` y `esMovimientoDuo(movimiento)` (funciones puras con tests en `src/__tests__/duo.spec.ts`).
- **Auto-creación**: al guardar una figura desde el wizard, si la detección es positiva se crea automáticamente su variante Duo (`MapaDeNodos.vue`), sin agregarla al store del mapa.
- **Administración**: panel "Duo" en `MovementModal.vue` con Activar / Descartar / Eliminar, y toggle con el ícono de Duo (SVG) normal ↔ Duo para navegar la vista Duo.
