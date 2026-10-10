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

- Estas medidas solo cuentan las lineas que se han separado previamente como producto en (`TipoLinea = "Producto"`), dejando fuera los registros de envios, descuentos y ajustes.
- `Importe` = `Quantity` × `Price`. En las devoluciones es negativo porque la cantidad es negativa.
- Los datos terminan el 12 de diciembre de 2011, así que diciembre de 2011 está incompleto y 2010 es el único año completo.
- El ticket medio es alto (~497 £), puede ser porque parte de los clientes son mayoristas.
- `Pedidos` cuenta facturas distintas con al menos una línea de producto vendida, no filas.
- La `Tasa de Devolución` se calcula sobre `importes`, no sobre unidades, por lo que tienen más peso las devoluciones de artículos caros.

# Limitaciones

- Tuve problemas al principio transformando los datos con la lectura de decimales con la configuración regional en español, que hizo que todos los precios se multiplicaran por 100.
- Para separar las devoluciones hubo que hacer un filtro por `TipoLinea`, porque el filtro con `TipoMovimiento` incluía comisiones y envíos devueltos, y no quería considerarlo devoluciones. 