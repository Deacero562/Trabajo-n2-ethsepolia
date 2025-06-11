⚙️ Funcionalidades

🔧 Constructor

Inicializa:

Hora de inicio

Duración (segundos)

Oferta mínima inicial

constructor(uint256 _startTime, uint256 _duration, uint256 _startingBid)

🏷️ Función para ofertar

Realiza una oferta válida si:

Supera en al menos 5% la actual mejor oferta.

La subasta está activa.

function sendBid() external payable

Registra el historial de ofertas y extiende la subasta 10 minutos si queda poco tiempo.

💰 Manejo de depósitos

Las ofertas se almacenan en:

mapping(address => uint256) public deposits;

Asociado a cada dirección oferente.

💸 Devolver depósitos

Durante la subasta: permite retirar el exceso de una oferta anterior (reembolso parcial).

Después de la subasta: permite retirar todo el depósito (menos comisión).

function withdrawDeposit() external

Se aplica comisión del 2% sobre el monto devuelto.

🥇 Mostrar ganador

Devuelve el oferente con la oferta más alta.

function getWinner() external view returns (address)

📜 Mostrar ofertas

Lista de todos los oferentes:

function getBidders() external view returns (address[] memory)

Monto de la mejor oferta:

function getHighestBid() external view returns (uint256)

Historial de pujas por usuario:

function getBidHistory(address user) external view returns (uint256[] memory)

❌ Cancelar Subasta

Solo si aún no hay ofertas:

function cancelAuction() external onlyOwner

🏁 Finalizar Subasta

Marca la subasta como finalizada y emite evento:

function finishAuction() external onlyOwner

📤 Retirar Fondos

Solo el owner puede retirar el saldo final:

function withdrawBalance() external onlyOwner

Descuenta depósitos pendientes de devolución.

📢 Eventos Emitidos

event NewBid(address indexed bidder, uint256 amount);
event AuctionFinished(address indexed winner, uint256 amount);
event AuctionCancelled();
event DepositWithdrawn(address indexed bidder, uint256 amount);

🔐 Seguridad y Recomendaciones

Uso de noReentrancy para proteger funciones sensibles.

Solo el owner puede finalizar, cancelar o retirar fondos.

Validaciones estrictas en fechas, valores y estados.

🧪 Otras características

Oferta válida (supera 5%) y rechazo si no lo hace.

Reembolso parcial durante subasta.

Reembolso con comisión después de finalizada.

Protección contra duplicados en lista de oferentes.
