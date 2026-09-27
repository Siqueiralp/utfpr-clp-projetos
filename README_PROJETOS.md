# Projetos de CLP - sequencia 1 a 14

Alvo: **SIMATIC Manager / STEP 7 Classic / S7-300**.

| Projeto | Conceito principal | Recursos centrais da revisao minima |
| --- | --- | --- |
| 1 | Semaforo LAD com TOF | **1 TOF**, 1 estado INT, pedido e controle minimo |
| 2 | Semaforo LAD com TON | **1 TON**, 1 estado INT, pedido e controle minimo |
| 3 | Estado inteiro / QB0 | **1 timer**, 4 variaveis persistentes |
| 4 | Timer + contador | **1 timer + 1 contador**, conforme requisito |
| 5 | Fluxo/minuto + noite | **1 timer + 2 contadores**; janela usa RTC |
| 6 | UDT + matriz + TOF | **1 TOF**; tabela temporaria, 4 variaveis persistentes |
| 7 | Shift register 8 bits | **0 timers**; estado = 1 BYTE |
| 8 | 13 lampadas Sao Jose | **0 timers**; 1 FB107 + fase + contador + clock |
| 9 | 13 lampadas Maceio | reutiliza o FB108 do Projeto 8 |
| 10 | Rampas do motor | **0 timers**; M10.0 fornece tick de 100 ms |
| 11 | GRAFCET com ramificacao OU | **1 estado INT + 1 timer** |
| 12 | CTUD IEC | 2 memorias de borda + valor do contador |
| 13 | GRAFCET LAD passo a passo | **1 TON**; bits de passo preservados pelo requisito |
| 14 | Chave reversora | **3 variaveis persistentes + 1 timer** |

## Symbol Table

Os Projetos 1, 2 e 13 possuem SDF com apenas **9 simbolos globais**:

- `I0.0`;
- `Q0.0..Q0.7`.

Estados e recursos internos nao sao cadastrados na Symbol Table.

## Clock Memory

Configure **MB10** como Clock Memory.

- `M10.0` = 10 Hz, borda positiva a cada 100 ms;
- `M10.5` = 1 Hz, 500 ms ligado / 500 ms desligado;
- `M10.7` = 0,5 Hz, usado para reduzir leituras do RTC.

Usos:

- M10.5: Projetos 1, 2, 3, 5, 6, 11 e 13;
- M10.0: Projetos 8/9 e 10;
- M10.7: Projeto 5.

Veja `CONFIGURAR_CLOCK_MEMORY.txt`.

## Projetos 1 e 2

Em vez de um bit para cada etapa, `MW20` guarda o estado:

- 0: via 2 verde;
- 1: via 2 amarela;
- 2: ambas vermelhas;
- 3: via 1 verde;
- 4: via 1 amarela;
- 5: ambas vermelhas;
- 6/7: pedestre e retorno para a via 1;
- 8/9: pedestre e retorno para a via 2.

Assim a direcao de retorno fica codificada no proprio estado e nao requer
`Retoma_Via1`.

`MW22` guarda o preset corrente e **T0 e o unico timer**. No Projeto 1 T0
e TOF; no Projeto 2 T0 e TON.

## Projeto 5

Os contadores C51/C52 medem os pulsos das duas vias. A janela de um minuto nao
usa timer: a troca do campo de minuto obtido por `SFC1` fecha a janela.
Somente `T52` permanece para temporizar a etapa atual.

## Projetos 7, 8 e 9

FB107 guarda todos os oito bits em um unico `BYTE` e usa `SHL/SHR`.
Como as entradas do FB sao pulsos, nao existem memorias internas de borda.

FB108 usa **uma unica instancia de FB107**. Os tempos fixos sao contados a
partir de `M10.0`; nao existe T80 nem quatro parametros de tempo.

## Projeto 10

Nao ha T100 nem OB35. A borda positiva de `M10.0` gera o tick de 100 ms.
O estado da rampa, o contador de ticks e duas memorias de borda sao suficientes.

## Projeto 11

A versao anterior usava oito instancias de um FB de passo. A revisao minima
representa o GRAFCET por um unico `INT etapa`. Os estados 6 e 8 recebem,
respectivamente, as duas entradas alternativas da ramificacao OU. Apenas T110
temporiza o passo ativo.

## Projeto 13

Aqui os bits individuais de passo foram mantidos porque o requisito e
explicitamente **GRAFCET Ladder passo a passo**. Mesmo assim, todos os passos
compartilham um unico TON T0; `MW20` guarda apenas o preset.

## Projeto 14

Nao depende mais do FB111. Um unico `INT estado` representa parado/frente/
reverso, acompanhado apenas por `automatico` e `armaTempo`.

## Importacao e validacao

Cada projeto deve ficar em um S7 Program separado, pois possui seu proprio OB1.
Projetos 8/9 requerem FB107; o Projeto 9 tambem requer FB108.

As fontes foram revisadas estaticamente. Ainda e necessario compilar no
STEP 7 Classic e testar no PLCSIM/CPU antes da entrega.
