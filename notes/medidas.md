# Diccionario de medidas

Medidas DAX del informe. Todas se calculan sobre líneas de producto
(`TipoLinea = "Producto"`), para que envíos, descuentos y ajustes no
distorsionen los resultados.

| Medida | Qué mide | Lógica |
|---|---|---|
| Ventas Brutas | Ingresos por ventas de producto, sin descontar devoluciones | Suma de `Importe` donde `TipoMovimiento = "Venta"` |
| Devoluciones | Importe devuelto por los clientes, en positivo | Suma de `Importe` donde `TipoMovimiento = "Devolución"`, con el signo cambiado |
| Ventas Netas | Ingresos reales tras devoluciones | Ventas Brutas - Devoluciones |
| Pedidos | Número de facturas de venta | Recuento de `Invoice` distintas, no de filas |
| Ticket Medio | Importe medio por pedido | Ventas Brutas / Pedidos |
| Tasa de Devolución | Peso de las devoluciones sobre las ventas | Devoluciones / Ventas Brutas |
| Ingresos por Envío | Dinero cobrado en gastos de envío | Suma de `Importe` con `TipoLinea = "Envío"` |

## Fórmulas

```
Ventas Brutas =
CALCULATE(
    SUM(Ventas[Importe]),
    Ventas[TipoMovimiento] = "Venta",
    Ventas[TipoLinea] = "Producto"
)
```
```
Ventas Netas = [Ventas Brutas] - [Devoluciones]
```
```
Ticket Medio = DIVIDE([Ventas Brutas], [Pedidos])
```
```
Tasa de Devolución = DIVIDE([Devoluciones], [Ventas Brutas])
```
```
Pedidos = 
CALCULATE(
    DISTINCTCOUNT(Ventas[Invoice]),
    Ventas[TipoMovimiento] = "Venta",
    Ventas[TipoLinea] = "Producto"
)
```
```
Ingresos por Envío = 
CALCULATE(
    SUM(Ventas[Importe]),
    Ventas[TipoMovimiento] = "Venta",
    Ventas[TipoLinea] = "Envío"
)
```
```
Devoluciones = 
-CALCULATE(
    SUM(Ventas[Importe]),
    Ventas[TipoMovimiento] = "Devolución",
    Ventas[TipoLinea] = "Producto"
)
```

## Notas

- Estas medidas solo cuentan las líneas que se han separado previamente como producto (`TipoLinea = "Producto"`), dejando fuera los registros de envíos, descuentos y ajustes.
- `Importe` = `Quantity` × `Price`. En las devoluciones es negativo porque la cantidad es negativa.
- `Pedidos` cuenta facturas distintas con al menos una línea de producto vendida, no filas.
- La `Tasa de Devolución` se calcula sobre importes, no sobre unidades, por lo que tienen más peso las devoluciones de artículos caros.
- Las devoluciones se imputan a la fecha de la factura de cancelación, no a la de la venta original. Por eso la tasa de un mes concreto es aproximada: puede incluir devoluciones de ventas de meses anteriores.
- El ticket medio es alto (~497 £). Una posible explicación es que parte de los clientes sean mayoristas, pero es una hipótesis que no he comprobado.
- Las dos páginas del informe comparten el filtro de año (segmentaciones sincronizadas), así que el año elegido en el resumen también afecta a la página de productos y países.

## Hallazgo: la tasa de devolución de 2011

La tasa pasa del 2,54% en 2010 al 4,84% en 2011, pero ese aumento no refleja un cambio en las devoluciones de clientes. Dos operaciones concentran en torno a un tercio del total devuelto (unas 716 mil £):

| Producto | Venta | Cancelación |
|---|---|---|
| Paper Craft, Little Birdie | 168.469,60 £ el 9 de diciembre de 2011 | -168.469,60 £ el mismo día |
| Medium Ceramic Top Storage Jar | 77.183,60 £ el 18 de enero de 2011 | -77.183,60 £ el mismo dia | 

En los dos casos hay una venta y una cancelación por el mismo importe, así que son pedidos anulados y no devoluciones habituales. Lo comprobé en una tabla provisional con `Invoice`, `InvoiceDate`, `Quantity` e `Importe`, filtrada por cada producto.

Efecto en los indicadores:

- Las ventas netas no cambian, porque cada venta y su cancelación se compensan.
- Las ventas brutas, el ticket medio y la tasa de devolución sí se inflan.
- Sin esas dos operaciones (venta y cancelación), la tasa de 2011 sería de aproximadamente el 2,3%, parecida a la de 2010. Es una estimación hecha con cifras redondeadas.

No hay evidencia, con estos datos, de que las devoluciones de clientes hayan aumentado en 2011.

## Limitaciones

- Diciembre de 2011 está incompleto (los datos terminan el 12 de diciembre) y 2009 solo tiene diciembre, así que 2010 es el único año completo. Las comparaciones entre años hay que leerlas con eso en cuenta.
- Alrededor de un 23% de las líneas no tiene `Customer ID`, por lo que no se pueden atribuir a ningún cliente.
- Los datos son de un solo comercio online, así que las conclusiones no se pueden generalizar a otras tiendas.
- No hay una tabla de productos: los rankings usan `Description`, y un mismo código podría aparecer con descripciones distintas.
- Solo se han revisado a fondo las dos mayores cancelaciones. No se ha comprobado si hay otras más pequeñas con el mismo patrón.

## Lecciones del proceso

- Al principio la lectura de decimales con la configuración regional en español hizo que los precios se interpretaran mal (373,57 se leía como 37.357, es decir, multiplicado por 100). Se corrigió cargando `Price` con configuración regional en inglés. Está explicado también en `limpieza.md`.
- Para separar las devoluciones hubo que filtrar también por `TipoLinea`: con `TipoMovimiento` solo, entraban comisiones y envíos devueltos, y no quería contarlos como devoluciones.
- Un filtro de año que quedó activo en una página sin segmentación visible falseó el gráfico de devoluciones. Lo detecté porque las cifras no cuadraban con la tabla de comprobación, y la solución fue hacer visible la segmentación en las dos páginas.
- La subida de la tasa de devolución en 2011 parecía una tendencia, pero al revisar las filas más grandes resultó venir de dos pedidos anulados. Antes de dar un indicador por bueno conviene mirar qué filas lo mueven.