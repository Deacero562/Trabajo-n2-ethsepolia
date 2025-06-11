# 🧾 Contrato de Subasta Descentralizada

Este contrato implementa una subasta descentralizada en Solidity con lógica de depósitos, reembolsos parciales, extensión automática de tiempo, y control de validez de ofertas. Es parte del trabajo final del **Módulo 2**.

## 🚀 Funcionalidades Principales

- **Inicio de Subasta:** el constructor inicializa duración y oferta mínima.
- **Ofertas Válidas:** una oferta es válida si supera la oferta más alta por al menos un 5%.
- **Reembolsos Parciales Manuales:** los participantes pueden retirar manualmente sus ofertas anteriores a la última válida (con una comisión del 2%).
- **Extensión del Tiempo:** si una oferta válida se realiza dentro de los últimos 10 minutos, la subasta se extiende automáticamente 10 minutos más.
- **Depósitos Asociados:** cada oferta se registra como depósito a nombre del oferente.
- **Retiros de Fondos:** los oferentes no ganadores pueden retirar su depósito (menos el 2% de comisión).
- **Cancelación de Subasta:** el propietario puede cancelar la subasta solo si no hubo ofertas.

---

## ⚙️ Variables

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `bidsList` | `mapping(address => uint256[])` | Historial de ofertas por usuario. |
| `deposits` | `mapping(address => uint256)` | Fondos disponibles para retiro. |
| `previousWithdrawn` | `mapping(address => bool)` | Controla si ya retiró ofertas anteriores. |
| `duration` | `uint256` | Duración de la subasta en segundos. |
| `endTime` | `uint256` | Timestamp del final de la subasta. |
| `startingBid` | `uint256` | Monto mínimo para ofertar. |
| `highestBid` | `uint256` | Oferta más alta actual. |
| `isActive` | `bool` | Indica si la subasta está activa. |
| `isCancelled` | `bool` | Indica si la subasta fue cancelada. |
| `locked` | `bool` | Protección contra reentrancia. |
| `owner` | `address` | Dirección del propietario. |
| `highestBidder` | `address` | Dirección del mejor postor. |
| `bidders` | `address[]` | Lista de participantes únicos. |
| `isBidder` | `mapping(address => bool)` | Control para evitar duplicados. |

---

## 🧠 Modificadores

- `onlyOwner`: solo el propietario puede ejecutar.
- `noOwner`: impide que el propietario oferte.
- `auctionActive`: restringe funciones a cuando la subasta está activa.
- `auctionFinished`: restringe funciones a cuando la subasta ya terminó.
- `noReentrancy`: evita ataques de reentrancia.

---

## 🔧 Funciones

### `constructor(uint256 _duration, uint256 _startingBid)`
Inicializa la subasta.  
- `_duration`: duración en segundos desde el despliegue.  
- `_startingBid`: valor mínimo de puja (en wei).

---

🧠 Consideración sobre el Historial de Ofertas

El contrato guarda el historial completo de ofertas de cada usuario en la variable bidHistory para permitir la funcionalidad avanzada de reembolsos parciales. Esto significa que un ofertante puede retirar las cantidades de sus ofertas anteriores que ya fueron superadas por nuevas pujas suyas, sin afectar su última oferta activa.

Guardar este historial completo es necesario para calcular correctamente cuánto se puede reembolsar en cada retiro parcial, ya que no basta con conocer solo la oferta más alta, sino todas las ofertas previas realizadas por el usuario.


---

### `sendBid() external payable`
Permite realizar una oferta válida.  
- Debe superar la mejor oferta por al menos 5%.  
- Registra la oferta y extiende el tiempo si quedan menos de 10 minutos.  
- La oferta es añadida al historial del usuario.  
- La mejor oferta anterior permanece como depósito y puede ser retirada por el ofertante anterior manualmente.

---

### `withdrawDeposit() external`
Permite retirar el depósito acumulado tras finalizar la subasta.  
- Se aplica una **comisión del 2%**.  
- Solo se puede usar si hay fondos disponibles.

---

### `withdrawPreviousBids() external`
Permite a los participantes retirar **sus ofertas anteriores** (todas menos la última).  
- Solo puede ejecutarse una vez por participante.  
- Se aplica una **comisión del 2%**.  
- Solo es accesible durante la subasta activa.

---

### `finishAuction() external onlyOwner`
Finaliza la subasta si ha transcurrido el tiempo.  
- Emite `AuctionFinished`.

---

### `cancelAuction() external onlyOwner`
Cancela la subasta **si no hubo ofertas previas**.

---

### `withdrawBalance() external onlyOwner`
Permite al propietario retirar el saldo restante en el contrato, **excluyendo depósitos aún no reclamados**.

---

### `getWinner() external view returns (address, uint256)`
Devuelve el ganador y el monto ganador, si la subasta ha finalizado.

---

### `getBidders() external view returns (address[] memory)`
Devuelve la lista de todos los participantes.

---

### `getHighestBid() external view returns (uint256)`
Devuelve el valor de la mejor oferta actual.

---

## 📢 Eventos

| Evento | Descripción |
|--------|-------------|
| `NewBid(address indexed bidder, uint256 amount)` | Se emite al registrar una nueva oferta. |
| `AuctionFinished(address indexed winner, uint256 amount)` | Se emite al finalizar la subasta. |
| `AuctionCancelled()` | Se emite si la subasta es cancelada por el propietario. |
| `DepositWithdrawn(address indexed bidder, uint256 amount)` | Se emite al retirar un depósito. |
| `PartialRefund(address indexed bidder, uint256 amount)` | Se emite al retirar ofertas anteriores (reembolso parcial). |

---

## 🧪 Ejemplo de Parámetros para Despliegue

- **Duración:** `600` (10 minutos)  
- **Oferta mínima:** `10000000000000000` (0.01 ETH)
