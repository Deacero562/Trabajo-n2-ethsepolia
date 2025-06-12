# 🧾 Contrato de Subasta Descentralizada

Este contrato implementa una subasta descentralizada en Solidity con lógica de depósitos, reembolsos parciales, extensión automática de tiempo y control de validez de ofertas.  
Es parte del trabajo final del Módulo 2.

---

## 🚀 Funcionalidades Principales

- **Inicio de Subasta**: El constructor inicializa la duración y la oferta mínima.  
- **Ofertas Válidas**: Una oferta es válida si supera en al menos un 5% a la oferta más alta actual.  
- **Reembolsos Parciales Manuales**: Los participantes pueden retirar manualmente sus ofertas anteriores a la última válida (con una comisión del 2%).  
- **Extensión del Tiempo**: Si una oferta válida se realiza dentro de los últimos 10 minutos, la subasta se extiende automáticamente por 10 minutos más.  
- **Depósitos Asociados**: Cada oferta se registra como un depósito a nombre del oferente.  
- **Retiros de Fondos**: Los oferentes no ganadores pueden retirar su depósito (menos el 2% de comisión).  
- **Cancelación de Subasta**: El propietario puede cancelar la subasta solo si no hubo ofertas.  

---

## ⚙️ Variables

| Nombre           | Tipo                             | Descripción                                            |
|------------------|---------------------------------|--------------------------------------------------------|
| `bidsList`       | `mapping(address => uint256)`   | Última oferta registrada por cada usuario              |
| `deposits`       | `mapping(address => uint256)`   | Fondos disponibles para retiro                         |
| `bidHistory`     | `mapping(address => uint256[])` | Historial completo de ofertas por usuario              |
| `reimbursed`     | `mapping(address => uint256)`   | Total reembolsado a cada usuario                       |
| `isBidder`       | `mapping(address => bool)`      | Control para evitar duplicados en `bidders`            |
| `duration`       | `uint256`                       | Duración de la subasta en segundos                     |
| `endTime`        | `uint256`                       | Timestamp de finalización de la subasta                |
| `startingBid`    | `uint256`                       | Oferta mínima aceptada                                 |
| `highestBid`     | `uint256`                       | Mejor oferta actual registrada                         |
| `isActive`       | `bool`                          | Indica si la subasta sigue activa                      |
| `isCancelled`    | `bool`                          | Indica si la subasta fue cancelada                     |
| `locked`         | `bool`                          | Protección contra reentrancia                          |
| `owner`          | `address`                      | Dirección del propietario del contrato                 |
| `highestBidder`  | `address`                      | Dirección del mejor postor actual                      |
| `bidders`        | `address[]`                    | Lista de participantes únicos                         |

---

## 🧠 Consideración sobre el Historial de Ofertas

El contrato almacena el historial completo de ofertas de cada usuario en la variable `bidHistory` para habilitar la funcionalidad avanzada de **reembolsos parciales**. Esto permite que un ofertante recupere el monto total de sus **ofertas anteriores que fueron superadas**, sin afectar su **última oferta activa**, que aún compite en la subasta.

Guardar este historial completo es esencial para calcular correctamente cuánto puede reembolsarse en cada retiro parcial. No basta con conocer solo la oferta más alta, ya que cada oferta previa representa un monto independiente que podría ser recuperado si fue superado por una posterior del mismo usuario.

---

## 🔐 Modificadores

- `onlyOwner`: Restringe la ejecución al propietario del contrato.  
- `noOwner`: Impide que el propietario pueda ofertar.  
- `auctionActive`: Requiere que la subasta esté activa y no cancelada.  
- `auctionFinished`: Requiere que la subasta ya haya finalizado.  
- `noReentrancy`: Previene ataques de reentrancia.  

---

## 🔧 Funciones Principales

### `constructor(uint256 _duration, uint256 _startingBid)`

Inicializa la subasta.

- `_duration`: Duración en segundos desde el despliegue.  
- `_startingBid`: Valor mínimo de puja (en wei).  

---

### `sendBid() external payable`

Permite realizar una oferta válida.

- Debe superar en al menos un 5% la mejor oferta actual.  
- Registra la oferta en `bidsList`, `deposits` y `bidHistory`.  
- Si hay una oferta previa, se acumula como depósito para retiro manual.  
- Si quedan menos de 10 minutos, extiende automáticamente la subasta.  

---

### `withdrawDeposit() external`

Permite retirar el depósito total acumulado, aplicando una comisión del 2%.

- Disponible para participantes que no ganaron.  
- Puede ejecutarse en cualquier momento después de la subasta.  

---

### `withdrawPreviousBids() external`

Permite a los participantes retirar todas sus ofertas anteriores a la última.

- Solo si hay más de una oferta realizada.  
- Aplica una comisión del 2%.  
- Evita retirar dos veces las mismas ofertas anteriores.  

---

### `finishAuction() external onlyOwner`

Finaliza la subasta si ya pasó el tiempo de duración.

- Cambia el estado de `isActive` a `false`.  
- Emite el evento `AuctionFinished`.  

---

### `cancelAuction() external onlyOwner`

Cancela la subasta si aún no ha habido ofertas.

- Cambia los estados `isActive` y `isCancelled`.  
- Emite `AuctionCancelled`.  

---

### `withdrawBalance() external onlyOwner`

Permite al propietario retirar el saldo del contrato **excluyendo los depósitos aún no reclamados**.

---

## Funciones de Vista

- `getWinner()`: Devuelve el ganador y la mejor oferta (si no fue cancelada).  
- `getBidders()`: Devuelve la lista de participantes únicos.  
- `getHighestBid()`: Devuelve el valor de la oferta más alta actual.  

---

## 📢 Eventos

| Evento                              | Descripción                                               |
|-----------------------------------|-----------------------------------------------------------|
| `NewBid(address, uint256)`         | Se emite al recibir una nueva oferta válida               |
| `AuctionFinished(address, uint256)`| Se emite al finalizar exitosamente la subasta             |
| `AuctionCancelled()`               | Se emite si la subasta fue cancelada                      |
| `DepositWithdrawn(address, uint256)`| Se emite al retirar el depósito total                     |
| `PartialRefund(address, uint256)` | Se emite al retirar ofertas anteriores (reembolso parcial) |

---

## 🧪 Ejemplo de Parámetros para Despliegue

```solidity
// Constructor
// Duración: 600 segundos (10 minutos)
// Oferta mínima: 0.01 ETH = 10000000000000000 wei

Subasta nuevaSubasta = new Subasta(600, 10000000000000000);
