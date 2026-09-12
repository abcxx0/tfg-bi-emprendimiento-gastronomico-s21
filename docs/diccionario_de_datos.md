# Diccionario de datos

Describe la estructura de los 4 archivos CSV que componen el modelo relacional de la solucion (carpeta `data/`), generados por el script `notebooks/tfg_generacion_datos.py`.

## LOTES (`lotes.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_lote (PK) | Alfanumerico | Identificador unico de cada lote de produccion. Formato `L{numero}-{TAM|HUM}`, ej. `L001-TAM`. |
| fecha_produccion | Fecha (Date) | Dia en que se elaboro la produccion. |
| producto | Texto | Producto elaborado: Tamal o Humita. |
| unidades_producidas | Entero | Cantidad total de unidades fabricadas en ese lote. |
| stock_inicial | Entero | Unidades disponibles al inicio del ciclo (0 en el dataset simulado, ya que cada lote parte de produccion nueva). |
| stock_final | Entero | Unidades remanentes luego de las ventas del lote (`unidades_producidas - unidades_vendidas`). |

## VENTAS (`ventas.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_venta (PK) | Alfanumerico | Identificador unico de cada transaccion de venta. Formato `V{numero}`. |
| ID_lote (FK) | Alfanumerico | Vincula la venta con el lote de produccion (`lotes.ID_lote`) del cual salieron las unidades. |
| fecha_venta | Fecha (Date) | Fecha en que se realizo la venta. |
| producto | Texto | Producto comercializado: Tamal o Humita. |
| unidades_vendidas | Entero | Cantidad de unidades comercializadas en la transaccion. |
| precio_unitario | Decimal | Precio de venta por unidad individual. |
| canal | Texto | Canal de venta: Feria o Pedido. |

## INSUMOS (`insumos.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_compra (PK) | Alfanumerico | Identificador unico del registro de consumo/compra de insumo. Formato `C{numero}`. |
| fecha_compra | Fecha (Date) | Fecha de referencia de la compra del insumo (un dia antes del inicio del ciclo de produccion asociado). |
| ingrediente | Texto | Nombre del insumo o materia prima. |
| cantidad | Decimal | Cantidad consumida del ingrediente en ese ciclo, expresada en su unidad de medida. |
| unidad | Texto | Unidad de medida del insumo (kg, L, unidad, m2). |
| precio_unitario | Decimal | Precio de compra por unidad de medida, con variabilidad simulada (+-10% sobre el precio base). |

> Nota de coherencia: INSUMOS no contiene una clave foranea fisica `ID_lote`. La vinculacion entre un insumo y el lote que lo consumio se resuelve de forma analitica en el modelo de Power BI, cruzando `fecha_compra` con `fecha_produccion` del lote y la composicion de RECETAS, no mediante una relacion directa entre tablas. Esto es intencional y esta alineado con el diseno del modelo relacional descrito en la arquitectura de la solucion.

## RECETAS (`recetas.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_receta (PK) | Alfanumerico | Identificador unico de cada item de receta. Formato `R{numero}`. |
| producto | Texto | Producto final al que corresponde la receta: Tamal o Humita. |
| ingrediente | Texto | Insumo componente de la formula. |
| cantidad_por_unidad | Decimal | Cantidad exacta del ingrediente requerida para producir una unidad del producto. |
| unidad | Texto | Unidad de medida de la cantidad (kg, L, unidad, m2). |

## Relaciones entre entidades

| Relacion | Cardinalidad | Descripcion |
|---|---|---|
| LOTES - VENTAS | 1:N | Un lote puede tener multiples ventas asociadas, vinculadas por `ID_lote`. |
| LOTES - INSUMOS | Sin relacion fisica directa | El costo se calcula analiticamente en Power BI cruzando fecha de produccion, receta y precio de compra vigente. |
| RECETAS - INSUMOS | Vinculacion logica por atributo `ingrediente` | Se resuelve mediante medidas DAX que buscan el precio del insumo correspondiente a cada ingrediente de la receta. |
