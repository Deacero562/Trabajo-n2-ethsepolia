# Contrato Inteligente: Subasta

Este contrato implementa una subasta descentralizada con los siguientes mecanismos:

- **Extensión automática del plazo** si se realizan ofertas cerca del cierre.
- **Reembolso parcial de depósitos** para ofertantes no ganadores.
- **Comisión sobre depósitos devueltos**.
- **Restricción para que el propietario no pueda ofertar**.

---

## 📦 Variables Principales

- `propietario`: Dirección del creador del contrato. Tiene permisos especiales (como finalizar la subasta).
- `precioInicial`: Precio mínimo de inicio para la subasta.
- `plazoFinalOriginal`: Tiempo límite inicial de la subasta.
- `plazoActual`: Tiempo límite actualizado (si hubo extensión).
- `subastaEstaActiva`: Indica si la subasta está en curso.
- `fondosRetirados`: Marca si el propietario ya retiró los fondos.
- `constanteMinimaIncremento`: Incremento mínimo (105 = +5%) requerido sobre la oferta ganadora actual.
- `comision`: Porcentaje de comisión aplicado al reembolso del depósito.
- `ofertaGanadora`: Oferta más alta actual (estructura con dirección y monto).

### Mappings

- `depositos`: Monto que cada dirección ha depositado.
- `ultimaOferta`: Última oferta válida realizada por cada dirección.
- `yaRetiraron`: Marca si un ofertante ya retiró su depósito.
- `ofertanteRegistrado`: Para evitar duplicar entradas en la lista de ofertantes.
- `todosLosOfertantes`: Arreglo con todas las direcciones que realizaron al menos una oferta.

---

## ⚙️ Funciones Públicas

### Constructor
```solidity
constructor(uint256 _precioInicial, uint256 _duracionSegundos)

Inicializa la subasta con precio base y duración determinada en segundos.
ofertar

function ofertar(uint256 monto) external

Permite ofertar si:

    La subasta está activa.

    Se cumple el incremento mínimo (5%).

    El ofertante no es el propietario.

    Tiene depósito suficiente.

Extiende el tiempo si la oferta se realiza en los últimos 10 minutos.
devolverDeposito

function devolverDeposito() external

Permite a los ofertantes no ganadores retirar sus depósitos (menos comisión). Solo una vez.
retirarDepositoParcial

function retirarDepositoParcial(uint256 monto) external

Permite retirar una parte del depósito siempre que no quede por debajo de la oferta máxima realizada.
finalizarSubasta

function finalizarSubasta() external

Solo el propietario puede ejecutarla. Marca la subasta como finalizada.
retirarFondosGanador

function retirarFondosGanador() external

Permite al propietario retirar el monto de la oferta ganadora una vez finalizada la subasta.
mostrarGanador

function mostrarGanador() external view returns (address, uint256)

Devuelve la dirección y monto de la oferta ganadora.
mostrarOfertas

function mostrarOfertas() external view returns (address[] memory, uint256[] memory)

Devuelve un arreglo de todos los ofertantes y sus respectivas ofertas.
siguienteOfertaMinima

function siguienteOfertaMinima() public view returns (uint256)

Calcula la siguiente oferta válida mínima (+5% de la actual ganadora).
receive

receive() external payable

Permite a los usuarios enviar directamente ETH al contrato para ser registrados como depósitos.
📢 Eventos

    NuevaOferta(address ofertante, uint256 monto): Emitido cada vez que se recibe una oferta válida.

    SubastaFinalizada(address ganador, uint256 monto): Emitido al finalizar la subasta.

    DepositoDevuelto(address ofertante, uint256 monto): Emitido al devolver depósitos luego de finalizada la subasta.

    ReembolsoParcial(address ofertante, uint256 monto): Emitido al hacer una extracción parcial del depósito durante la subasta.

🔒 Seguridad

    El propietario no puede participar como ofertante (modifier noEsPropietario).

    Se evita que un ofertante retire más de su depósito o por debajo de su mejor oferta.

    Se restringen ciertas funciones a cuando la subasta está activa o finalizada detalladas a continuación:

Restricciones por Estado de la Subasta

Algunas funciones solo pueden ejecutarse cuando la subasta está en un estado específico. Esto se controla mediante modificadores personalizados:
📌 Funciones limitadas a cuando la subasta está activa

Estas funciones usan el modificador requiereSubastaActiva, que impone la condición:

require(subastaEstaActiva && block.timestamp < plazoActual, "La subasta no esta activa");

Esto significa que la subasta:

    Debe estar activa (subastaEstaActiva == true)

    Y no debe haber alcanzado el plazo límite (block.timestamp < plazoActual)

🔒 Funciones restringidas:

    ofertar(uint256 monto)
    → Solo se puede ofertar durante el tiempo válido de la subasta.

    retirarDepositoParcial(uint256 monto)
    → Solo se pueden hacer extracciones parciales mientras la subasta está en curso.

📌 Funciones limitadas a cuando la subasta está finalizada

Estas funciones usan el modificador requiereSubastaFinalizada, que impone la condición:

require(!subastaEstaActiva, "La subasta aun esta activa");

Es decir, solo se ejecutan una vez finalizada la subasta.
🔒 Funciones restringidas:

    devolverDeposito()
    → Permite retirar el depósito (con comisión) solo a ofertantes que no ganaron.

    retirarFondosGanador()
    → El propietario solo puede retirar los fondos del ganador cuando la subasta haya terminado.

🛑 Función que cambia el estado de la subasta:

    finalizarSubasta()
    → Solo el propietario puede ejecutar esta función. Cambia subastaEstaActiva a false, marcando el fin de la subasta.
