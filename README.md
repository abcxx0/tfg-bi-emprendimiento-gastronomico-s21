# TFG - BI para Emprendimiento Gastronómico (S21)

## Descripción

Prototipo de inteligencia de negocios descriptivo-diagnóstico desarrollado para un emprendimiento gastronómico artesanal de producción por lotes (tamales y humitas). El sistema permite calcular indicadores de rentabilidad, costos de insumos por lote, análisis de sensibilidad ante variaciones de precios y tiempo de liquidación del lote, a partir de un dataset generado mediante reconstrucción híbrida (parámetros relevados + simulación controlada).

> Descripción provisoria. Se actualizará con el título definitivo del TFG.

## Estructura del repositorio

```
├── notebooks/    # Notebook de Google Colab: generación y validación del dataset sintético (Python, Pandas, NumPy)
├── data/         # Archivos CSV fuente: lotes.csv, ventas.csv, insumos.csv, recetas.csv
├── powerbi/      # Archivo .pbix: modelo relacional, medidas DAX y dashboard
└── docs/         # Diccionario de datos y documentación complementaria
```

## Stack tecnológico

| Capa | Tecnología | Función |
|---|---|---|
| Generación de datos | Python, Pandas, NumPy, Google Colab | Construcción y validación del dataset sintético |
| Almacenamiento | Archivos CSV | Persistencia portable y editable de los datos fuente |
| Modelado y cálculo | Power BI Desktop, DAX | Relaciones entre entidades e indicadores de gestión |
| Visualización | Power BI Desktop | Dashboard interactivo de 4 vistas (Producción, Ventas, Rentabilidad, Sensibilidad) |
| Control de versiones | GitHub | Trazabilidad y reproducibilidad del desarrollo |

## Guía de instalación y ejecución

### Requisitos previos

- Cuenta de Google (para ejecutar el notebook en Google Colab)
- Power BI Desktop instalado (gratuito, disponible para Windows)

### Pasos

1. **Generar o regenerar el dataset**
   - Abrir el notebook ubicado en `notebooks/` desde Google Colab (`Archivo > Abrir notebook > GitHub` y pegar la URL de este repositorio, o subirlo manualmente a Colab).
   - Ejecutar todas las celdas en orden. El notebook valida automáticamente la regla de consistencia `stock_inicial + producción - ventas = stock_final` antes de exportar los archivos.
   - Los archivos `lotes.csv`, `ventas.csv`, `insumos.csv` y `recetas.csv` se generan en la carpeta `data/`.

2. **Abrir el modelo y el dashboard**
   - Descargar el archivo `.pbix` de la carpeta `powerbi/`.
   - Abrirlo con Power BI Desktop.
   - Si se regeneraron los CSV, actualizar el origen de datos desde `Inicio > Transformar datos > Configuración del origen de datos`, apuntando a la nueva ubicación de los archivos, y luego `Actualizar`.

3. **Navegar el dashboard**
   - El dashboard cuenta con 4 vistas accesibles mediante pestañas: **Producción**, **Ventas**, **Rentabilidad** y **Sensibilidad**.
   - La vista de Sensibilidad incluye parámetros *What-If* para simular variaciones en el precio de insumos principales.

## Alcance y límites

Este es un prototipo de nivel descriptivo-diagnóstico, sin analítica predictiva ni prescriptiva. Utiliza un dataset sintético construido a partir de parámetros relevados de un caso real, no registros históricos reales. Para más detalle sobre el alcance, ver `docs/`.

## Autor

Proyecto desarrollado como Trabajo Final de Grado (TFG) en Ciencia de Datos.
