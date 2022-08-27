# Manual de usuario

Aqui encontraremos una guía rápida para interactuar e invertir en la plataforma de BOC. 
En primer lugar, desde la página principal de Bank of Chain [página de inicio](https://bankofchain.io/#/) hacemos click en el botón `LAUNCH APP`.

![](.gitbook/assets/launchapp.png)

## Conectamos la billetara

Podemos interactuar con BOC a través de nuestra [wallets](more/appendix.md#wallet) de [MetaMask](https://metamask.io/) o [WalletConnect](https://walletconnect.com/), haciendo click en el botón `CONNECT WALLET`.

![](.gitbook/assets/connectwallet.png)

Luego de conectarnos, podemos ver en la página principal el balance de las monedas que tenemos en nuestra billetera.

![](.gitbook/assets/pic-4.png)

## Ajustes de parámetros

### Puente entre redes

BOC provee la opcion de trabajar con puentes [cross-chain bridge](more/appendix.md#puentes-de-blockchain) en caso de que sea necesario hacer operaciones entre diferentes redes.

![](.gitbook/assets/chainbridge.png)

### Cambiar de red

Actualmente BOC trabaja sobre la red de Ethereum, BNB Chain y Polygon. Si necesitamos cambiar de red podremos hacerlo clickeando en `NETWORKS`.

![](.gitbook/assets/networkchange.png)

## Inversión y reembolso

### Depósito

Una vez que conectamos nuestra wallet podremos hacer el depósito seleccionando la bóveda que queremos usar. En la seccion de USDi Vault podremos depositar monedas estables. 
Establecemos la cantidad deseada o seleccionamos el saldo "Max" y hacemos click en `DEPOSIT`.

![](.gitbook/assets/pic-7.png)

En la bóveda de ETHi Vault podremos depositar ETH.

![](.gitbook/assets/deposito%20ethi.jpg)

A diferencia de la bóveda de USDi, la bóveda de ETHi muestra una estimación del gas a utilizar para hacer el depósito.

### Retiro

Luego de conectar nuestra billetera, podremos hacer un retiro seleccionando la bóveda en el que tenemos depósitos haciendo click en el boton `WITHDRAW`. En el caso de USDi Vault tenemos la posibilidad de elegir la moneda estable que queremos recibir o puede ser un mix de ellas.

![](.gitbook/assets/pic-8.png)

En el caso de ETHi Vault tenemos la posibilidad de retirar solamente ETH. Seleccionamos el monto a retirar y hacemos click en el botón `WITHDRAW`.

![](.gitbook/assets/retiro%20ehti.jpg)

Intercambio: habilitación de la función de intercambio. Cada estrategia en BOC utiliza diferentes monedas estables. Cuando retiramos, lo haremos segun el APY de la estrategia de menor a mayor. Por lo tanto, como ejemplo, si se obtiene una estrategia que no sea USDT, se devolverá la moneda estable de la estrategia correspondiente. Si tomamos como ejemplo la estrategia Yearn LUSD, se devolverá LUSD.

### Retiros - Parámetros avanzados

![](.gitbook/assets/advancesetting.png)

Los parámetros de las opciones avanzadas a determinar son:

**Pérdida máxima**: determinamos el porcentaje máximo que podriamos perder al momento de retirar. Cuando especificamos el monto de USDi que queremos retirar podremos ver el valor del activo a retirar, sin embargo este puede no ser el monto que recibiremos por pérdidas o cambios de cotización que pueden ocurrir durante el proceso de retiro.
Por ejemplo, si queremos retirar $1000 y determinamos que el porcentaje máximo a perder es 0.3%, no vamos a recibir menos de $997.

**Slippage**: [Slippage](more/appendix.md#slippage) es el deslizamiento en el valor de la moneda. Debemos especificar el porcentaje de deslizamiento que aceptamos.

### Agregar USDi y ETHi a la wallet

Si no vemos el token USDi en la wallet podremos agregarlo manualmente o de forma muy sencilla haciendo click en `+` que se encuentra al lado del balance de USDi y haciendo click en `Añadir Token` en la billetera. Luego de eso, podremos ver la cantidad de USDi que tenemos en la billetera.

![](.gitbook/assets/addtoken.png)

De la misma forma podremos agregar el token ETHi.

![](.gitbook/assets/agregar%20ethi.jpg)

## Dashboard (Tablero de mando)

Desde la seccion de [dashboard](more/appendix.md#dashboard) podremos ver informacion relevante sobre nuestros activos y los protocolos con el cual interactúan.

[https://dashboard.bankofchain.io](https://dashboard.bankofchain.io)

![](.gitbook/assets/dashboard.jpg)

1. Total de USDi Supply: total de USDi bloqueado en la bóveda. Lee la información desde el subgrafo.
2. Holders: número de usuarios invirtiendo. Lee la información desde el subgrafo.
3. APY (last 30 days): interés anual de los últimos 30 dias.
4. Vault Protocol Allocation: muestra en qué protocolos y en qué proporción están distribuidos los fondos bloqueados.
5. Asset (USD): muestra la cantidad de activos en cada estrategia, medido en dólares.
6. Offical APY: APY oficial, regularmente se actualiza cada mañana.
7. Weekly APY y weekly profit: APY y ganancia semanal.
8. Strategy Adrress: direccion de la estrategia.
9. Vault Operation Record: registro de las operaciones que se realizaron en dicha bóveda, especificando día, método, detalle y link a la transacción.

Subgrafos de BOC:

* ETH: [https://api.thegraph.com/subgraphs/name/bankofchain/boc-subgraph-eth](https://api.thegraph.com/subgraphs/name/bankofchain/boc-subgraph-eth)
* BNB: [https://api.thegraph.com/subgraphs/name/bankofchain/boc-subgraph-bsc](https://api.thegraph.com/subgraphs/name/bankofchain/boc-subgraph-bsc)
* POLYGON: [https://api.thegraph.com/subgraphs/name/bankofchain/boc-subgraph-matic](https://api.thegraph.com/subgraphs/name/bankofchain/boc-subgraph-matic)

## Detalles de la estrategia

Tomando como ejemplo la estrategia SushiUsdcUsdtStrategy, podemos observar la siguiente informacion:

![](.gitbook/assets/detail.jpg)

1. Detalle de la estrategia
   * Nombre de la estrategia
   * Tokens utilizados en la estrategia
   * Total de activos depositados en dicha estrategia (valorados en USD)
   * Estado de la estrategia
2. Rendimiento (APY) de la estrategia
   * APY semanal (línea azul)
   * APY oficial semanal (línea amarilla)
3. Detalle de las operaciones realizadas en la estrategia
   * Total Balance: balance actual de la estrategia
   * Balance Change: cambio que se produjo en la estrategia como consecuencia de la operación realizada
   * Reward Asset: recompensa obtenida (medida en USD)
   * Operation: tipo de operación realizada
      * `lend`: se depositan fondos a la estrategia
      * `harvest`: se retira y reinvierten los fondos en la estrategia
      * `withdraw`: se retiran los fondos de la estrategia

## Análisis de inversiones personales

Al conectar la wallet podremos ver la evolución y posición actual de nuestras inversiones personales:

![](.gitbook/assets/personalpage.jpg)

1. Balance (USDi): muestra los activos depositados en BOC, medidos en USDi.
2. Unrealized profit (USDi): muestra las ganancias generadas y no realizadas, medido en USDi.
3. Realized profit (USDi): muestra las ganancias realizadas.
4. APY (last 7 days): muestra el rendimiento de los últimos 7 días.
5. APY (last 30 days): muestra el rendimiento de los ultimos 30 días.
6. Daily USDi: muestra la evolución diaria de los activos depositados en BOC, medido en USDi.
7. Monthly Profit: muestra la ganancia mensual.
