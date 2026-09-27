PROJETO 2 - SEMAFORO COM TON MINIMO

A fonte usa somente T0.

MW20 guarda o estado 0..9 e MW22 o preset atual. Os estados 6/7 e 8/9
codificam tambem qual via deve ser retomada depois do pedestre, evitando uma
variavel separada de retorno.

Bits internos:
M0.0 pedido
M0.1 habilitacao do TON
M0.2 memoria FP da botoeira
M0.3 inicializacao

M10.5 fornece o pisca de 500 ms.

A Symbol Table contem apenas I0.0 e Q0.0..Q0.7.

Configure MB10 como Clock Memory, importe o SDF e a fonte AWL, compile e teste
com MRES no PLCSIM.
