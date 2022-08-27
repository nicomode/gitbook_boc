# Diseño del algoritmo del protocolo
BOC posibilita generar rendimientos con monedas estables y con ETH.

## Descripción del proceso de monedas estables

![pic-2](../.gitbook/assets/pic-2.png)

1. "Deposit": BOC posibilita al usuario `depositar` las tres principales monedas estables (USDT, USDC, DAI) en cualquier proporcion y monto que desee, y a cambio recibirá el monto correspondiente en USDi.
   "Retirar": Los usuarios pueden `retirar`, en cualquier momento, USDi en las tres principales monedas estables. Por defecto, recibirán las monedas estables en proporción a lo que haya en la [bóveda](https://docs.bankofchain.io/docs/v/spanish/more/appendix#boveda) al momento de hacer el retiro o pueden especificar una determinada moneda a retirar.
2. Luego de que la bóveda recibe las monedas estables,  el proceso de `queryTokenPrice` solicita el valor las [monedas estables](https://docs.bankofchain.io/docs/v/spanish/more/appendix#stablecoin) a través de un oráculo externo. Cuando el valor recibido por el [oráculo](https://docs.bankofchain.io/docs/v/spanish/more/appendix#oraculos) es superior a 1 USD, se calcula a 1 USD, y cuando es inferior a 1 USD, se calcula al precio del oráculo.
3. Basado en el valor calculado, el proceso de [mint/burn](https://docs.bankofchain.io/docs/v/spanish/more/appendix#burn) crea o quema el valor equivalente de USDi.
4. El módulo [Keeper](https://docs.bankofchain.io/docs/v/spanish/more/appendix#keeper) alcanza la condición de activación de `doHardWork` y ejecuta dicha condición asignando los fondos a la estrategia.
5. La bóveda llama al módulo de intercambio y ejecuta la condicion de `swapTokenToWants`.
6. El módulo de intercambio ejecuta la acción de `swapTokens` y completa el intercambio.
7. La bóveda recibe la moneda intercambiada.
8. La bóveda `deposita` las monedas estables en la estrategia que corresponde.
9. La [estrategia](https://docs.bankofchain.io/docs/v/spanish/more/appendix#estrategia) invierte las monedas estables en un tercer protocolo.
10. El módulo Keeper alcanza la condición de activación de `harvest` y activa dicha acción.
11. Se ejecuta la accion de `harvest` en cada estrategia.
12. Cada estrategia ejecuta la accion de `claimRewards` con el fin de recolectar lo generado.
13. Cada estrategia transfiere las monedas recolectadas a la cosechadora a traves del proceso de `transferRewards`.
14. La cosechadora vende lo recolectado a traves del proceso de `sellRewards` a cambio de monedas estables.
15. A traves de la acción `sendProfitToVault` la cosechadora transfiere a la bóveda las monedas estables recibidas.
16. El módulo Keeper alcanza la condición de activar el `rebase` y activa dicho proceso.
17. La bóveda llama a la accion `changeTotalSupply` para emitir USDi adicionales.
18. La bóveda recoge el 20% de los rendicimientos, los cuales son transferidos a tesorería (`Treasury`).
19. La [tesorería](https://docs.bankofchain.io/docs/v/spanish/more/appendix#tesoreria-de-las-daos) beneficiará a los usuarios al utilizar `buyback` para recomprar el token de gobernanza BOC.

### Reglas de creación y quema (mint & burn)

Veremos el proceso de creación y quema de USDi a través de un ejemplo numérico.

Alicia deposita 100 USDT, 100 DAI y 100 USDC.

Según la regla de creación del protocolo, el precio de intercambio es de 1.00 USD cuando el precio obtenido del oráculo Chainlink es igual o superior a 1.00 USD, de lo contrario sería igual al precio obtenido de [Chainlink](https://chain.link/).

Por lo tanto, si el precio obtenido desde Chainlink es:

* 1 USDT = 1,01 USD
* 1 DAI = 0,99 USD
* 1 USDC = 1.00 USD

Alicia obtendrá (mint) 299 USDi:

100 USDT = 100 x 1.00 = 100 USDi (precio de Chainlink > 1 USD，precio final = 1 USD)\
100 DAI = 100 x 0.99 = 99 USDi (precio de Chainlink < 1 USD，precio final = precio de Chainlink)\
100 USDC = 100 x 1.00 = 100 USDi (Chainlink = 1 USD，precio final = 1 USD)

![mint](../.gitbook/assets/mint.png)

Supongamos que ahora Alicia decide retirar sus momendas estables por lo que el protocolo quemará (`burn`) USDi. Tiene 299 USDi y cuando decide retirarlos el contrato inteligente de la bóveda le entregará USDT/USDC/DAI según la proporción que haya de dichas monedas en la bóveda.
Las reglas de quema son opuestas a las reglas de creacion de USDi: cuando el precio obtenido de Chainlink es menor a 1 USD el valor de cambio será de 1 USD, de lo contrario será igual al precio obtenido desde Chainlink.

Suponiendo que los valores obtenidos de Chainlink no cambiaron:

* 1 USDT = 1.01 USD
* 1 DAI = 0.99 USD
* 1 USDC = 1.00 USD

y Alicia decide quemar 299 USDi, obtendría:

* 100 USDi = 100/1.01 = 99 USDT (precio de Chainlink > 1 USD --> precio final = precio de Chainlink)
* 100 USDi = 100/1.00 = 100 DAI (precio de Chainlink < 1 USD --> precio final = 1 USD)
* 100 USDi = 100/1.00 = 100 USDC (precio de Chainlink = 1 USD --> precio final = 1 USD)

![burn](../.gitbook/assets/burn.png)

Los números expuestos en el gráfico son a modo de ejemplo con el objetivo de entender las reglas de creación y quema que utiliza el protocolo BOC. En la realidad, las fluctuaciones de USDi son menores, por lo que el usuario no se encontraría con una pérdida como la ejemplificada. De hecho, posiblemente la pérdida sea menor a 0.01%. El objetivo de estas reglas es eliminar el [arbitraje](https://github.com/Francisco-Rua/boc\_gitbook/blob/es\_version/how-it-works/appendix/README.md#arbitraje) en nuestra plataforma y proteger el protocolo.

## Cosecha

La acción de `harvestTrigger` se activa cada día para determinar si se cumple la condición de `harvest`. Las dos condiciones son:

1. Se supera el intervalo de tiempo máximo.
2. Se cumple la regla para cosechar:

$$
Beneficio \times 20\% > costo de la cosecha.
$$

En caso de que cualquiera de las dos condiciones se cumpla, la estrategia llevará a cabo el trabajo de cosecha (`harvest`):

1. Ejecuta la transferencia del rendimiento cosechado;
2. Informa el activo actual de la estrategia.

| Seteo de parámetros                                                                                                                                                                      | ETH                    | BNB Chain             | Polygon                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- | ---------------------- | ---------------------- |
| Ciclo de activación de tareas programadas                                                                                                                                                | 6:00 am todos los días | 6:00 am todos los días | 6:00 am todos los días |
| Intervalo máximo de tiempo para activar la estrategia de "cosecha". En caso de que el intervalo entre la "cosecha" actual y la anterior sea mayor a este valor, la "cosecha" se deberá hacer. | 3 días               | 1 día               | 1 día               |
| El factor costo-beneficio X de la estrategia de activación de "cosecha" (en caso de que ganancia>=costo\*X, la cosecha deberá hacerse.)                                     | 5                      | 5                      | 5                      |

## Rebalanceo

USDi es un token diseñado de manera que el suministro en circulación se ajusta automáticamente según las fluctuaciones de los precios, este proceso se llama [rebalanceo](https://docs.bankofchain.io/docs/v/spanish/more/appendix#rebase). Cuando el total de activos de la bóveda es mayor al total emitido en USDi, significa que se han generado nuevos ingresos. En este momento, el valor de USDi comparado con el valor de USD será revisado y la cantidad de USDi crecerá, de manera que el total de USDi sea consistente al valor total de los activos de la bóveda, asegurando que 1 USDi se encuentre anlcado a 1 USD. Al mismo tiempo, 20% del USDi adicional será transferido a la tesorería de la DAO en concepto de comisiones de gestión.

## Asignación de fondos

### doHardWork

Las entradas para ajustar la posición del algoritmo son el [APY](https://docs.bankofchain.io/docs/v/spanish/more/appendix#interes-porcentual-anual-apy) oficial del protocolo de terceros, el gas requerido para llevar adelante la inversión en cada estrategia, el límite permitido de [deslizamiento](https://docs.bankofchain.io/docs/v/spanish/more/appendix#slippage) y las [reglas de asignación de fondos](https://github.com/Francisco-Rua/boc\_gitbook/blob/es\_version/how-it-works/introduction-to-boc/README.md#reglas-de-asignaci%C3%B3n-de-fondos).

Los datos de salida son la estrateiga y el monto de los fondos a ser invertidos.

| Parámetros                                                                                               | ETH                                | BNB Chain                  | Polygon                       |
| ------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | -------------------------- | ----------------------------- |
| Ciclo de activación de tareas programadas                                                                           | 7 am todos los dias, excepto Lunes | 7 am todos los dias, excepto Lunes | 7 am todos los dias, excepto Lunes. |
| Costo-Beneficio. Cálculo del Periodo x (si la ganancia X días >= costo, "doHardWork" se llevará a cabo)) | 365 días                           | 365 días                   | 365 días                      |

### Asignación (allocation)

Comparado con `doHardWork`, `allocation` realiza un paso más: retira los fondos de las estrategias de bajo APY, luego usa como entrada el APY oficial del protocolo de terceros, el gas requerido para invertir en cada estrategia, el límite permitido de deslizamiento, la regla de asignación de fondos y el algoritmo de ajuste de posición. 

La salida es la estrategia y el monto esperado de los fondos invertidos.

| Parámetros                                                                                                         | ETH                    | Cadena BNB             | Polygon                |
| ----------------------------------------------------------------------------------------------------------------------------- | ---------------------- | ---------------------- | ---------------------- |
| Ciclo de activación de informes de posición preajustado                                                                       | Cada domingo, luego de hacer doHardWork | Cada domingo, luego de hacer doHardWork | Cada domingo, luego de hacer doHardWork |
| Ciclo de activación de la tarea programada                                                    | 7am todos los lunes.                       | 7am todos los lunes.                       | 7am todos los lunes.                       |
| Cálculo periodo X de costo-beneficio (Si el beneficio del reequilibrio X días >= costo, se puede realizar la accion de "asignación") | 30 días                | 30 días                | 30 días                |

### Algoritmo de asignación de fondos

| Variable       | Significado                                                                                                                                                                                                                                        |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "durationDays" | El ciclo de rebalanceo debe asegurar que en un ciclo después de rebalancear, el beneficio después de rebalanceo - beneficio antes del rebalanceo - el coste del reequilibrio > 0                                                 |
| "yearDays"     | 365 días                                                                                                                                                                                                                                       |
| "asset1"       | Activos originales de la estrategia.                                                                                                                                                                                                        |
| "apr1"         | APR de la estrategia antes de ajustar la posición (el APY necesita ser convertido en APR), el APY del algoritmo de ajuste de la posición es el APY promedio de los últimos 7 días calculado por fuera de la cadena de la estrategia.|
| "deltaAsset"   | Asumir el cambio de capital del rebalanceo de la estrategia.                                                                                                                                                                           |
| "poolAssets1"  | El TVL del pool de la estrategia objetivo es el parámetro para cambiar el APR luego de ajustar la posición                                                                                                            |

Beneficio antes del ajustar la posición

$$
gain1 = \frac{asset1 \times apr1 \times durationDays}{yearDays}
$$

Cambios en los ingresos

$$
gain2 = \frac{ (asset1+deltaAsset-exchangeLoss)\times apr2 \times durationDays}{yearDays}
$$

Cambio de APR

$$
apr2 = \frac{apr1 \times poolAssets1}{(poolAssets1+deltaAsset-exchangeLoss)}
$$

Luego de sustituir apr2 en gain2 utilizando la ecuación anterior:

$$
gain2=\frac{apr1 \times durationDays/yearDays \times (asset1+deltaAsset) \times poolAssets1}{poolAssets1+deltaAsset-exchangeLoss}
$$

Entonces, la relación entre los ingresos cambiados de la estrategia y los activos cambiados es:

$$
deltaGain = gain2-gain1 = \frac{deltaAsset \times (poolAsset1-asset1) \times apr1 \times durationDays}{(poolAsset1+deltaAsset-exchangeLoss) \times yearDays}
$$

Costo del cambio de fondos para una sola estrategia:

| Variable                                 | Siginificado                                                                                                       |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| "withdrawFee"                                         | Comisión por retirar fondos.                                                                      |
| "lendFee"                                             | Comisión por agregar fondos.                                                                        |
| "exchangeLoss"  | Pérdida por desenclaje al momento de hacer un intercambio.                                                                                                               |
| "harvestFee"                                          | Comision por "harvest"                                                                           |
| "profitChangeFee"                                     | Costo por cambio de capital                                                                                     |
| "withdrawGas"                                         | Es el gas consumido por la operación de "retirar". El mismo es estimado previo a hacerse la operación |
| "lendGas"                                             | Es el gas consumido por la operación de "depositar". El mismo es estimado previo a hacerse la operación.                         |
| "exchangeLossRate"                                    | Redeem Slippage                                                                                                |

$$
profitChangeFee=withdrawFee+lendFee+exchangeLoss+harvestFee
$$

$$
withdrawFee = gasPrice \times withdrawGas
$$

$$
lendFee= gasPrice \times lendGas
$$

$$
exchangeLoss=deltaAsset \times exchangeLossRate
$$

$$
harvestFee=harvestGas \times durationDays
$$

Encuentra la suma máxima de deltaGain para todas las estrategias:

$$
profitChange=MAX\sum_{i=1}^m(deltaGain_i -withdrawFee_i-lendFee_i - exchangeLoss_i-harvestFee_i)
$$

$$
profitChange=MAX\sum_{i=1}^m(\frac{deltaAsset_i \times (poolAsset_i-asset_i) \times apr_i * durationDays_i}{(poolAsset_i+deltaAsset_i-exchangeLoss_i) \times yearDays_i}
$$

$$
- operateFee_i - exchangeLoss_i - harvestFee_i)
$$

Cambia beneficio total ProfitChange en esta fórmula, la única variable es deltaAsset para cada estrategia. Al mismo tiempo, la solución debe estar limitada por:

Restricciones

1. Los fondos de la misma estrategia de cada protocolo (restricciones múltiples) no excedan el 30% de los fondos totales
2. La suma de todos los activos que ingresan y salen es 0.

Condiciones límite

1. Los activos estratégicos no pueden exceder el 20% del total de los activos.
2. Los fondos estratégicos no pueden exceder el 50% de los activos del pool objetivo.

Utiliza el programa de python `optimize.minimize` para encontrar el esquema de rebalanceo óptimo.

### Configuración de los parámetros públicos

| Parámetros                                                                                                            | ETH   | BNB Chain | Polygon  |
| -------------------------------------------------------------------------------------------------------------------------------- | ----- | ---------- | -------- |
| Cálculo de la asignación de fondos. Ajuste del desenclaje del intercambio.                                                   | 0,15% | 0,15%      | 0,15%    |
| Configuración del gas (incluye gas de depósito y retiro, el gas por intercambio, costo de cosecha) | 0 (pasa a ser el gas actual cuando los activos > $5 millones)       | Gas real   | Gas real |

### Reglas de cálculo del APY oficial

El APY oficial es necesario como referencia cuando colocamos fondos. Las fuentes para obtener el gas oficial son las siguientes: vfat.tools, coingecko, zapper, Official APY, apy.vision Fee, etc. En caso de que la fuente no incluya el cálculo del APY para la estrategia, directamente BOC copiará la regla de cálculo de APY de la estrategia. En general, el APY oficial de la estrategia consiste en ingresos por creación de mercado y de minado de monedas.
Tomando como ejemplo la estrategia "ConvexLusdStrategy", el APY se obtiene del contenido de la siguiente tabla:

| Marca APY    | APR         | APY              | Interés compuesto | Método de cálculo    | Origen de la información                                                                                             |
| ------------ | ----------- | ---------------- | ----------------- | -------------------- | ----------------------------------------------------------------------------------------------------------- |
| Trx comisión | 0,0024      | Tx comisión       | N                 | API                  | https://www.convexfinance.com/api/curve-apys                                                                |
| crv          | 0,023523458 | cvx mining coins | Y                 | API                  | https://www.convexfinance.com/api/curve-apys                                                                |
| cvx          | 0,024132445 | cvx mining coins | Y                 | Cálculo del contrato | En función del tiempo, el precio de las monedas mineras a un año/activos totales del pool puede obtener APR |

$$
APY=(1+APR)^{periods}-1
$$

El parámetro "periods" es el periodo de pago de intereses.

### Reglas de cálculo del APY real de la estrategia

El APY real de la estrategia es calculado en base al retorno estándar de la moneda de la estrategia.

### ETH Mecanismo de coseecha:

Actualmente, el mecanismo de cosecha en ETH es exactamente igual al mecanismo de cosecha utilizado para las monedas estables. La única diferencia es que el colateral de USD es USDi, mientas que el de ETH es llamado ETHi.
