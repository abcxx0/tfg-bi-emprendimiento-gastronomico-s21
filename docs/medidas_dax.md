# Medidas DAX del modelo

Medidas implementadas en el archivo `powerbi/TFG_Dashboard1.pbix`, agrupadas por bloque funcional.

## Gestion de lotes e inventario operativo

**Numero_Lote**
```
Numero_Lote = LEFT(lotes[ID_lote], 4)
```
Extrae los primeros cuatro caracteres del identificador del lote para aislar la codificacion estandarizada de la partida.

**Total_Producido**
```
Total_Producido = SUM(lotes[unidades_producidas])
```
Suma la totalidad de unidades fabricadas en todos los lotes registrados.

**Total_Vendidas**
```
Total_Vendidas = SUM(ventas[unidades_vendidas])
```
Suma el volumen total de unidades comercializadas en todas las transacciones.

**Tamales_Producidos / Humitas_Producidas**
```
Tamales_Producidos = CALCULATE(SUM(lotes[unidades_producidas]), lotes[producto] = "Tamal")
Humitas_Producidas = CALCULATE(SUM(lotes[unidades_producidas]), lotes[producto] = "Humita")
```
Cuantifican el volumen elaborado por linea de producto.

**Tamales_Vendidos / Humitas_Vendidas**
```
Tamales_Vendidos = CALCULATE(SUM(ventas[unidades_vendidas]), ventas[producto] = "Tamal")
Humitas_Vendidas = CALCULATE(SUM(ventas[unidades_vendidas]), ventas[producto] = "Humita")
```
Calculan la cantidad vendida por producto.

## Analisis comercial y ventas

**Ingreso_Total**
```
Ingreso_Total = SUMX(ventas, ventas[unidades_vendidas] * ventas[precio_unitario])
```
Determina la facturacion bruta acumulada.

## Estructura de costos y recetas

**Costo Unitario**
```
Costo Unitario =
VAR FechaLote = IF(HASONEVALUE(LOTES[fecha_produccion]), SELECTEDVALUE(LOTES[fecha_produccion]), MAX(LOTES[fecha_produccion]))
VAR ProductoActual = IF(HASONEVALUE(RECETAS[producto]), SELECTEDVALUE(RECETAS[producto]), BLANK())
RETURN SUMX(
    FILTER(RECETAS, RECETAS[producto] = ProductoActual),
    VAR IngredienteActual = RECETAS[ingrediente]
    VAR Cantidad = RECETAS[cantidad_por_unidad]
    VAR PrecioCompra = CALCULATE(MAX(INSUMOS[precio_unitario]), INSUMOS[ingrediente] = IngredienteActual, INSUMOS[fecha_compra] <= FechaLote)
    RETURN Cantidad * PrecioCompra
)
```
Calcula dinamicamente el costo de fabricacion por unidad, cruzando la fecha de produccion del lote con el precio de compra del insumo vigente en esa fecha.

**% Participacion Costo**
```
% Participacion Costo =
VAR CostoActual = [Costo Unitario]
VAR CostoTotalProducto = CALCULATE([Costo Unitario], ALLSELECTED(RECETAS[ingrediente]), VALUES(RECETAS[producto]))
RETURN DIVIDE(CostoActual, CostoTotalProducto)
```
Representa el peso relativo de un ingrediente sobre el costo unitario total de la receta.

**Costo Total Original**
```
Costo Total Original =
SUMX(VENTAS,
    VAR ProductoActual = VENTAS[producto]
    VAR Unidades = VENTAS[unidades_vendidas]
    VAR CostoUnitario = SUMX(
        FILTER(RECETAS, RECETAS[producto] = ProductoActual),
        VAR IngredienteActual = RECETAS[ingrediente]
        VAR PrecioInsumo = CALCULATE(MAX(INSUMOS[precio_unitario]), INSUMOS[ingrediente] = IngredienteActual)
        RETURN RECETAS[cantidad_por_unidad] * PrecioInsumo
    )
    RETURN Unidades * CostoUnitario
)
```
Calcula el costo historico total de las ventas, valorizado segun receta original y precio de insumos.

**Precio Venta Docena Real**
```
Precio Venta Docena Real =
VAR PrecioUnitario = CALCULATE(AVERAGE(VENTAS[precio_unitario]), VENTAS[producto] = SELECTEDVALUE(RECETAS[producto]), VENTAS[ID_lote] IN VALUES(LOTES[ID_lote]))
RETURN PrecioUnitario * 12
```
Proyecta el precio de venta de una docena a partir del precio unitario promedio del lote.

**Margen por Docena**
```
Margen por Docena = [Precio Venta Docena Real] - [Costo por Docena]
```
Margen de contribucion por docena.

**Margen por Unidad**
```
Margen por Unidad =
VAR PrecioUnitario = CALCULATE(AVERAGE(VENTAS[precio_unitario]), VENTAS[producto] = SELECTEDVALUE(RECETAS[producto]), VENTAS[ID_lote] IN VALUES(LOTES[ID_lote]))
VAR Costo = [Costo Unitario]
RETURN IF(NOT ISBLANK(PrecioUnitario) && NOT ISBLANK(Costo), PrecioUnitario - Costo, BLANK())
```
Rentabilidad unitaria por producto.

**Margen_Bruto**
```
Margen_Bruto = [Ingresos_Ventas] - [Costo_Insumos_Lote]
```
Resultado financiero bruto de las operaciones.

## Diagnostico de lotes y margenes unitarios de referencia

**Grafico Margen Unitario Ajustado / Original**
```
Grafico Margen Unitario Ajustado =
VAR ProductoActual = SELECTEDVALUE(Dim_Productos[producto])
VAR UltimoLote = MAXX(TOPN(1, FILTER(ALL(LOTES), LOTES[producto] = ProductoActual), LOTES[fecha_produccion], DESC), LOTES[ID_lote])
RETURN DIVIDE(CALCULATE([Margen Ajustado], ALL(LOTES), LOTES[ID_lote] = UltimoLote), CALCULATE(SUM(VENTAS[unidades_vendidas]), ALL(LOTES), LOTES[ID_lote] = UltimoLote), 0)
```
Evalua el margen unitario del ultimo lote de cada producto, en escenario ajustado y en escenario original (misma logica, usando `[Margen Original]`).

**Humita / Tamal Margen Original Lote y Margen Ajustado Lote**

Medidas espejo para cada producto que aislan el ultimo lote producido y calculan el margen unitario historico (original) y simulado (ajustado), dividiendo el margen del lote por las unidades vendidas de ese lote.

## Simulacion de sensibilidad e incrementos

**Valor de VarCarne / Valor de VarChoclo**
```
Valor de VarCarne = SELECTEDVALUE('VarCarne'[VarCarne], 0)
Valor de VarChoclo = SELECTEDVALUE('VarChoclo'[VarChoclo], 0)
```
Capturan el porcentaje de variacion seleccionado por el usuario en los controles deslizantes What-If.

**Precio Carne Ajustado / Precio Choclo Ajustado**
```
Precio Carne Ajustado =
VAR PrecioBase = CALCULATE(MAX(INSUMOS[precio_unitario]), INSUMOS[ingrediente] = "Carne de vaca")
VAR VariacionSeleccionada = SELECTEDVALUE('VarCarne'[VarCarne], 0)
RETURN PrecioBase * (1 + (VariacionSeleccionada / 100))
```
Recalculan el precio unitario del insumo critico aplicando el incremento simulado.

**Costo Ajustado Tamal / Costo Ajustado Humita**

Estiman el costo unitario del producto impactado por la variacion simulada del insumo critico (carne para Tamal, choclo para Humita), sumando el incremento proporcional al costo base.

**Costo Total Ajustado**
```
Costo Total Ajustado =
VAR VarCarneValor = SELECTEDVALUE('VarCarne'[VarCarne], 0)
VAR VarChocloValor = SELECTEDVALUE('VarChoclo'[VarChoclo], 0)
RETURN SUMX(VENTAS,
    VAR ProductoActual = VENTAS[producto]
    VAR Unidades = VENTAS[unidades_vendidas]
    VAR CostoUnitarioAjustado = SUMX(
        FILTER(RECETAS, RECETAS[producto] = ProductoActual),
        VAR IngredienteActual = RECETAS[ingrediente]
        VAR PrecioInsumoOriginal = CALCULATE(MAX(INSUMOS[precio_unitario]), INSUMOS[ingrediente] = IngredienteActual)
        VAR PrecioInsumoAjustado = SWITCH(IngredienteActual,
            "Carne de vaca", PrecioInsumoOriginal * (1 + (VarCarneValor / 100)),
            "Choclo blanco", PrecioInsumoOriginal * (1 + (VarChocloValor / 100)),
            PrecioInsumoOriginal)
        RETURN RECETAS[cantidad_por_unidad] * PrecioInsumoAjustado
    )
    RETURN Unidades * CostoUnitarioAjustado
)
```
Revaloriza el costo total de ventas simulando incrementos en insumos criticos mediante SWITCH.

**Margen Ajustado / Margen Original**
```
Margen Ajustado = [Ingreso Total] - [Costo Total Ajustado]
Margen Original = [Ingreso Total] - [Costo Total Original]
```
Margen de contribucion bajo escenario simulado y bajo escenario de referencia, respectivamente. La diferencia entre ambos es la base del analisis de sensibilidad de la Vista 4.

## Tablas calculadas

**Dim_Ingredientes**
```
Dim_Ingredientes =
ADDCOLUMNS(
    DISTINCT(RECETAS[ingrediente]),
    "Orden Costo Receta",
    VAR IngActual = [ingrediente]
    VAR CostoReal =
        MAXX(
            FILTER(RECETAS, RECETAS[ingrediente] = IngActual),
            VAR PrecioCompra = MAXX(FILTER(INSUMOS, INSUMOS[ingrediente] = IngActual), INSUMOS[precio_unitario])
            RETURN RECETAS[cantidad_por_unidad] * PrecioCompra
        )
    RETURN CostoReal * -1
)
```
Construye una tabla calculada con el listado unico de ingredientes, enriquecida con una columna que ordena los ingredientes por su impacto en el costo de la receta (mayor costo primero, usando valor negativo para orden descendente en visualizaciones).
