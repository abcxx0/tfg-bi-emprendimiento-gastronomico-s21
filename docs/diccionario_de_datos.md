# Diccionario de datos

Describe la estructura de los 4 archivos CSV fuente (carpeta `data/`) y las tablas calculadas/parametros implementados en Power BI, que componen el modelo relacional completo de la solucion.

## Archivos CSV fuente

### LOTES (`lotes.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_lote (PK) | Alfanumerico | Identificador unico de cada lote de produccion. Formato `L{numero}-{TAM|HUM}`, ej. `L001-TAM`. |
| fecha_produccion | Fecha (Date) | Dia en que se elaboro la produccion. |
| producto | Texto | Producto elaborado: Tamal o Humita. |
| unidades_producidas | Entero | Cantidad total de unidades fabricadas en ese lote. |
| stock_inicial | Entero | Unidades disponibles al inicio del ciclo (0 en el dataset simulado, ya que cada lote parte de produccion nueva). |
| stock_final | Entero | Unidades remanentes luego de las ventas del lote (`unidades_producidas - unidades_vendidas`). |

### VENTAS (`ventas.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_venta (PK) | Alfanumerico | Identificador unico de cada transaccion de venta. Formato `V{numero}`. |
| ID_lote (FK) | Alfanumerico | Vincula la venta con el lote de produccion (`lotes.ID_lote`) del cual salieron las unidades. |
| fecha_venta | Fecha (Date) | Fecha en que se realizo la venta. |
| producto | Texto | Producto comercializado: Tamal o Humita. |
| unidades_vendidas | Entero | Cantidad de unidades comercializadas en la transaccion. |
| precio_unitario | Decimal | Precio de venta por unidad individual. |
| canal | Texto | Canal de venta: Feria o Pedido. |

### INSUMOS (`insumos.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_compra (PK) | Alfanumerico | Identificador unico del registro de consumo/compra de insumo. Formato `C{numero}`. |
| fecha_compra | Fecha (Date) | Fecha de referencia de la compra del insumo (un dia antes del inicio del ciclo de produccion asociado). |
| ingrediente | Texto | Nombre del insumo o materia prima. |
| cantidad | Decimal | Cantidad consumida del ingrediente en ese ciclo, expresada en su unidad de medida. |
| unidad | Texto | Unidad de medida del insumo (kg, L, unidad, m2). |
| precio_unitario | Decimal | Precio de compra por unidad de medida, con variabilidad simulada (+-10% sobre el precio base). |

> Nota de coherencia: INSUMOS no contiene una clave foranea fisica `ID_lote`. La vinculacion entre un insumo y el lote que lo consumio se resuelve de forma analitica en el modelo de Power BI, cruzando `fecha_compra` con `fecha_produccion` del lote y la composicion de RECETAS, no mediante una relacion directa entre tablas. Esto es intencional y esta alineado con el diseno del modelo relacional descrito en la arquitectura de la solucion.

### RECETAS (`recetas.csv`)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ID_receta (PK) | Alfanumerico | Identificador unico de cada item de receta. Formato `R{numero}`. |
| producto | Texto | Producto final al que corresponde la receta: Tamal o Humita. |
| ingrediente | Texto | Insumo componente de la formula. |
| cantidad_por_unidad | Decimal | Cantidad exacta del ingrediente requerida para producir una unidad del producto. |
| unidad | Texto | Unidad de medida de la cantidad (kg, L, unidad, m2). |

## Tablas calculadas y parametros en Power BI

### Dim_Productos

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| producto | Texto | Lista distinta de productos (`Tamal`, `Humita`). Generada con `DISTINCT(lotes[producto])` o `DISTINCT(recetas[producto])`. |

Funcion: Tabla de dimension para filtrado cruzado de lotes, ventas y recetas por producto.

### Dim_Ingredientes

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| ingrediente | Texto | Lista distinta de ingredientes (generada con `DISTINCT(RECETAS[ingrediente])`). |
| Orden Costo Receta | Decimal | Columna calculada que asigna a cada ingrediente su impacto en el costo de la receta (costo real multiplicado por -1 para orden descendente en visualizaciones). |

Funcion: Tabla calculada enriquecida que permite ordenar los ingredientes por su peso en el costo de la receta en graficos y tablas.

### VarChoclo (parametro What-If)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| VarChoclo | Decimal | Valor numerico continuo (porcentaje) seleccionado por el usuario mediante control deslizante. Rango tipico: 0% a 50% (configurable). |

Funcion: Parametro desconectado que alimenta las medidas de simulacion de incremento en el precio del choclo.

### VarCarne (parametro What-If)

| Columna | Tipo de dato | Descripcion |
|---|---|---|
| VarCarne | Decimal | Valor numerico continuo (porcentaje) seleccionado por el usuario mediante control deslizante. Rango tipico: 0% a 50% (configurable). |

Funcion: Parametro desconectado que alimenta las medidas de simulacion de incremento en el precio de la carne.

## Relaciones y tablas auxiliares del modelo de Power BI

| Relacion o tabla auxiliar | Cardinalidad | Campo de vinculacion / configuracion | Funcion en el modelo |
|---|---|---|---|
| lotes → ventas | 1:N | lotes.ID_lote → ventas.ID_lote | Vincula cada registro de venta con el lote de produccion correspondiente. Permite analizar unidades vendidas, stock y liquidacion del lote. |
| Dim_Productos → lotes | 1:N | Dim_Productos.producto → lotes.producto | Permite filtrar y analizar los lotes segun el producto elaborado. |
| Dim_Productos → recetas | 1:N | Dim_Productos.producto → recetas.producto | Permite asociar cada producto con los ingredientes y cantidades definidas en su receta. |
| Dim_Ingredientes → recetas | 1:N | Dim_Ingredientes.ingrediente → recetas.ingrediente | Vincula la tabla calculada de ingredientes con las recetas, permitiendo ordenar y filtrar por ingrediente en visualizaciones y calculos de costo. |
| insumos ↔ recetas | N:N | insumos.ingrediente ↔ recetas.ingrediente; filtro bidireccional | Vincula las compras de ingredientes con su utilizacion en las recetas y permite calcular costos mediante las medidas DAX. |
| VarChoclo | Sin relacion fisica | Tabla de parametro What-If desconectada | Proporciona el valor utilizado para simular variaciones en el precio del choclo. |
| VarCarne | Sin relacion fisica | Tabla de parametro What-If desconectada | Proporciona el valor utilizado para simular variaciones en el precio de la carne. |
