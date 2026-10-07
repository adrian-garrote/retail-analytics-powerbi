# Notas de limpieza

Auditoría de calidad del dataset Online Retail II realizada en Power Query
antes de modificar nada. Primero se midieron los problemas, se apuntaron y después se
decidió que hacer con cada uno.

Total de filas del dataset original: 1.067.371

## 1. Problemas detectados

| Problema | Columna | Filas | % aprox. | Observaciones |
|---|---|---|---|---|
| Nulos | Description | n/d | <1% | |
| Nulos | Customer ID | n/d | 23% | Probables compras sin cliente identificado |
| Facturas canceladas (empiezan por C) | Invoice | 19.494 | 1,8% | 19.493 con Quantity < 0 (devoluciones de clientes) y 1 con Quantity = +1 |
| Ajuste manual con C y Quantity positiva | Invoice | 1 | <0,01% | Descripción "Manual", precio 37.357, sin Customer ID. Valor atípico |
| Quantity <= 0 sin C | Quantity | 3.457 | 0,3% | Price = 0, sin Customer ID, descripciones tipo "damages", "missing", "lost". Ajustes de stock |
| Price <= 0 | Price | 6.207 | 0,6% | Incluye los 3.457 ajustes de stock |
| Price = 0 con Quantity > 0 | Price | ~2.745 | 0,3% | Mezcla de casos, algunos con Customer ID. Sin patrón claro |
| Price < 0 | Price | 5 | <0,01% | Facturas que empiezan por A, "Adjust bad debt", sin Customer ID |
| Códigos que no son producto | StockCode | ~5.790 | 0,5% | Ver detalle abajo |
| Duplicados exactos | Todas | 34.335 | 3,2% | Diferencia entre el recuento inicial y el recuento tras quitar duplicados (1.033.036) |
| Valores no estándar | Country | n/d | | EIRE, RSA, Unspecified, European Community (43 valores distintos en total) |

Nota: algunas categorías se solapan (por ejemplo, los ajustes de stock están
dentro de Price <= 0), así que las filas no se pueden sumar sin más.

### Detalle de códigos que no son producto (StockCode)

| Código | Filas | Tipo probable |
|---|---|---|
| POST | 2.122 | Gastos de envío |
| DOT | 1.446 | Gastos de envío |
| C2 | 282 | Gastos de envío |
| M | 1.421 | Ajuste manual |
| D | 177 | Descuentos |
| S | 104 | Muestras |
| BANK | 102 | Comisiones bancarias |
| ADJUST | 70 | Ajustes |
| AMAZON | 43 | Comisiones |
| TEST | 17 | Pruebas |
| B | 6 | Ajuste de deuda incobrable |

Otros códigos detectados con el filtro por número de dígitos:

- Probables pruebas o ajustes: `TEST001`, `TEST002`, `ADJUST2`, `m`.
- Pendientes de revisar: `PADS`, `C3`, `SP1002`, `GIFT`, `CRUK`.
- Los códigos `DCGS*` tienen formato de producto real (4 dígitos en vez de
  5), por lo que el filtro por número de dígitos los atrapó por error. Se
  tratan como producto salvo que la descripción indique lo contrario.
- `m` y `M` hay que unificarlos pasando el código a mayúsculas.

## 2. Decisiones

Se decide quedarnos con los valores que nos sirven para responder a las preguntas planteadas en el proyecto, y eliminar o excluir aquellos registros que no aporten nada o que puedan alterar los resultados.

| # | Problema | Decisión | Razón |
|---|---|---|---|
| 1 | Duplicados exactos | Eliminar | Probablemente sea doble registro. Se supone que dos líneas idénticas con la misma hora de factura son un error |
| 2 | Facturas C con Quantity < 0 | Mantener y marcar como `Devolución` | Necesarias para analizar devoluciones por producto |
| 3 | Quantity <= 0 sin C | Excluir | Ajustes de stock, sin cliente ni precio |
| 4 | Price < 0 y ajuste "Manual" de 37.357 | Excluir | Ajustes contables, no son ventas. El valor atípico distorsionaría los ingresos |
| 5 | Price = 0 con Quantity > 0 | Excluir de las ventas | Sin importe; no se puede determinar su naturaleza |
| 6 | Códigos que no son producto | Mantener y clasificar en una columna `TipoLinea` (Producto, Envío, Descuento, Ajuste/Otros) | Permite medir envíos y descuentos sin contaminar los rankings de producto |
| 7 | Customer ID nulo | Mantener en el análisis de ventas; excluir solo del análisis de clientes | No afecta a ingresos, pero esas compras no se pueden atribuir a un cliente |
| 8 | EIRE, RSA | Renombrar a Ireland y South Africa | Compatibilidad con los mapas de Power BI |
| 9 | Unspecified, European Community | Mantener, pero excluir de rankings y mapas por país | No son países |


