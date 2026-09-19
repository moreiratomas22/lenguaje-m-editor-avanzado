# script_limpieza.md

Código M completo, escrito directamente en el Editor Avanzado de Power Query (sin usar los botones de la interfaz), para limpiar la tabla `ventas_raw` de TechStore.

```m
let // Paso 1: Fuente de datos original
    Origen = Table.FromRows(
        {
            {1, " Laptop Pro 15 ", "Computación", 1200.00, #date(2024, 1, 5)},
            {2, "Mouse Inalámbrico", "accesorios", 28.00, #date(2024, 1, 8)},
            {3, " Teclado Mecánico", "PRUEBA", 95.00, #date(2024, 1, 12)},
            {4, "Monitor 4K ", "computación", 450.00, #date(2024, 2, 3)},
            {5, " Auriculares BT", "Audio", 120.00, #date(2024, 2, 10)},
            {6, "SSD Externo 1TB ", "PRUEBA", 130.00, #date(2024, 3, 5)},
            {7, "Webcam HD", "Accesorios", 85.00, #date(2024, 3, 12)}
        },
        {"id_venta", "nombre_producto", "categoria", "precio", "fecha_venta"}
    ), // No modificar este paso: tabla de prueba reproducida con Table.FromRows (equivalente a lo que genera "Especificar datos")

    // Paso 2: Eliminar espacios en blanco al inicio y al final
    // de la columna nombre_producto usando Text.Trim. Los espacios sueltos
    // son invisibles pero rompen comparaciones exactas, uniones (merge) y
    // agrupaciones: " Laptop Pro 15 " y "Laptop Pro 15" contarían como
    // productos distintos. Se hace primero para que todo lo que venga
    // después trabaje con nombres ya limpios.
    LimpiarEspacios = Table.TransformColumns(Origen, {{"nombre_producto", Text.Trim, type text}}),

    // Paso 3: Estandarizar la columna categoria a Title Case con Text.Proper,
    // para que variantes como "computación" y "Computación", o "accesorios"
    // y "Accesorios", queden bajo una única grafía (y "PRUEBA" pase a
    // "Prueba"), antes de filtrar por texto.
    EstandarizarCategoria = Table.TransformColumns(LimpiarEspacios, {{"categoria", Text.Proper, type text}}),

    // Paso 4: Filtrar y eliminar registros de prueba.
    // Excluye filas donde categoria sea exactamente "Prueba" (ya en Title Case
    // gracias al paso anterior) usando Table.SelectRows. Se hace DESPUÉS de
    // estandarizar a propósito: ver la justificación en README.md.
    EliminarPruebas = Table.SelectRows(EstandarizarCategoria, each [categoria] <> "Prueba"),

    // Paso 5: Definir tipos de datos correctos.
    // id_venta: Int64.Type (identificador numérico entero)
    // nombre_producto y categoria: type text (datos descriptivos)
    // precio: type number (monto con decimales, se necesita sumar/promediar)
    // fecha_venta: type date (para poder filtrar y comparar por fecha)
    TiparColumnas = Table.TransformColumnTypes(EliminarPruebas, {
        {"id_venta", Int64.Type},
        {"nombre_producto", type text},
        {"categoria", type text},
        {"precio", type number},
        {"fecha_venta", type date}
    })
in
    TiparColumnas
```

## Resultado esperado

5 filas (se eliminan los `id_venta` 3 y 6, categoría "PRUEBA"):

| id_venta | nombre_producto | categoria | precio | fecha_venta |
|---|---|---|---|---|
| 1 | Laptop Pro 15 | Computación | 1200.00 | 2024-01-05 |
| 2 | Mouse Inalámbrico | Accesorios | 28.00 | 2024-01-08 |
| 4 | Monitor 4K | Computación | 450.00 | 2024-02-03 |
| 5 | Auriculares BT | Audio | 120.00 | 2024-02-10 |
| 7 | Webcam HD | Accesorios | 85.00 | 2024-03-12 |

Verificado fuera de Power BI simulando la misma lógica paso a paso (trim → Title Case → filtro → conteo final), antes de pasarlo al Editor Avanzado, para confirmar que da exactamente estas 5 filas.
