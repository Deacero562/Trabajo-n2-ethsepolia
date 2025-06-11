# 🧾 Contrato de Subasta Descentralizada

Este contrato implementa una subasta descentralizada en Solidity con lógica de depósitos, reembolsos parciales, extensión automática de tiempo y control de validez de ofertas. Es parte del trabajo final del **Módulo 2**.

## 🚀 Funcionalidades Principales

- **Inicio de Subasta:** el constructor inicializa la duración y la oferta mínima.
- **Ofertas Válidas:** una oferta es válida si supera la oferta más alta por al menos un 5%.
- **Reembolsos Parciales:** se devuelven automáticamente los fondos de la oferta anterior al nuevo ofertante líder.
- **Extensión del Tiempo:** si una oferta válida se realiza dentro de los últimos 10 minutos, la subasta se extiende 10 minutos más.
- **Depósitos Asociados:** cada oferta queda registrada como depósito a nombre del oferente.
- **Retiros de Fondos:** al finalizar la subasta, los oferentes no ganadores pueden retirar sus depósitos.
- **Cancelación de Subasta:** el propietario puede cancelarla solo si no hubo ofertas.

## ⚙️ Variables

| Nombre | Tipo | Descripción |
|-------|------|-------------|
| `bidsList` | `mapping(address => uint256)` | Última oferta registrada por cada usuario. |
| `deposits` | `mapping(address => uint256)` | Fondos disponibles para retiro por cada usuario. |
| `duration` | `uint256` | Duración en segundos de la subasta. |
| `endTime` | `uint256` | Timestamp de finalización de la subasta. |
| `startingBid` | `uint256` | Monto mínimo de puja. |
| `highestBid` | `uint256` | Monto actual más alto ofertado. |
| `isActive` | `bool` | Indica si la subasta está activa. |
| `isCancelled` | `bool` | Indica si la subasta fue cancelada. |
| `locked` | `bool` | Protección contra ataques de reentrancia. |
| `owner` | `address` | Dirección del propietario (quien despliega el contrato). |
| `highestBidder` | `address` | Dirección del ofertante con la oferta más alta. |
| `bidders` | `address[]` | Lista de todos los participantes que ofertaron. |

## 🧠 Modificadores

- `onlyOwner`: solo el propietario puede ejecutar.
- `noOwner`: impide que el propietario haga ofertas.
- `auctionActive`: solo permite funciones mientras la subasta está activa.
- `auctionFinished`: solo permite funciones cuando ha terminado.
- `noReentrancy`: evita ataques de reentrancia en funciones con `transfer`.

## 🔧 Funciones

### `constructor(uint256 _duration, uint256 _startingBid)`
Inicializa la subasta.  
- `_duration`: duración en segundos desde el momento del despliegue.  
- `_startingBid`: valor mínimo de puja (en wei).

---

### `sendBid() external payable`
Permite realizar una oferta válida.  
- Requiere superar en al menos 5% la mejor oferta.
- Extiende el tiempo si quedan menos de 10 minutos.
- Reembolsa automáticamente al ofertante anterior.

---

### `withdrawDeposit() external`
Permite al usuario retirar su depósito si no ganó.  
- Solo puede ejecutarse si el depósito es mayor a cero.

---

### `finishAuction() external onlyOwner`
Finaliza la subasta cuando el tiempo ha terminado.  
- Emite el evento `AuctionFinished`.

---

### `cancelAuction() external onlyOwner`
Cancela la subasta si aún no hay ofertas.

---

### `withdrawBalance() external onlyOwner`
Permite al propietario retirar el saldo acumulado (sin incluir los depósitos aún no retirados).

---

### `getWinner() external view returns (address)`
Devuelve la dirección del ganador si la subasta ya finalizó.

---

### `getBidders() external view returns (address[] memory)`
Devuelve una lista de todos los ofertantes.

---

### `getHighestBid() external view returns (uint256)`
Devuelve el valor de la mejor oferta actual.

---

## 📢 Eventos

| Evento | Descripción |
|--------|-------------|
| `NewBid(address bidder, uint256 amount)` | Se emite cuando se realiza una nueva oferta. |
| `AuctionFinished(address winner, uint256 amount)` | Se emite al finalizar la subasta. |
| `AuctionCancelled()` | Se emite si el propietario cancela la subasta. |
| `DepositWithdrawn(address bidder, uint256 amount)` | Se emite al retirar un depósito. |

---

## 🧪 Ejemplo de Parámetros para Deploy

- Duración: `600` (10 minutos)
- Oferta mínima: `10000000000000000` (0.01 ETH)
