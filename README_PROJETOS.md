# Projetos de CLP - sequencia 1 a 14

Fontes para **SIMATIC Manager / STEP 7 Classic (S7-300)**. A referencia de
hardware usada no Projeto 1 e CPU 314C-2 DP. Cada pasta contem a entrega
correspondente ao item de mesmo numero no enunciado da disciplina.

| Projeto | Arquivo principal | Conceito exigido |
| --- | --- | --- |
| 1 | `Projeto1/Projeto1_TOF.awl` | Semaforo LAD com TOF |
| 2 | `Projeto2/Projeto2_TON.awl` | Semaforo LAD com TON |
| 3 | `Projeto3/Projeto3_Byte.scl` | Estado inteiro e escrita de `QB0` |
| 4 | `Projeto4/Projeto4_Contador.scl` | Um timer, contador e comparacoes |
| 5 | `Projeto5/Projeto5_Inteligente.scl` | Fluxo por minuto e alerta noturno |
| 6 | `Projeto6/Projeto6_Matrizes_TOF.scl` | UDT, matriz de estados/tempos e um TOF |
| 7 | `Projeto7/Projeto7_Shift_Register_BIT.scl` | FB de deslocamento de 8 bits |
| 8 | `Projeto8/Projeto8_SaoJose.scl` | 13 lampadas, bit 1 sequencial |
| 9 | `Projeto9/Projeto9_Maceio.scl` | 13 lampadas, bit 0 sequencial |
| 10 | `Projeto10/Projeto10_Motor.scl` | Quatro rampas e tres patamares |
| 11 | `Projeto11/Projeto11_Grafcet.scl` | Passos GRAFCET com ramificacao OU |
| 12 | `Projeto12/Projeto12_CTUD.scl` | FB contador crescente/decrescente IEC |
| 13 | `Projeto13/Projeto13_Grafcet_LAD.awl` | GRAFCET Ladder passo a passo |
| 14 | `Projeto14/Projeto14_ChaveReversora.scl` | Reversao automatica de 5 s e manual |

## Como importar

1. Crie um **S7 Program diferente para cada projeto**, pois cada fonte contem
   o seu proprio `OB1`.
2. Para `.awl`, use `Sources > Insert > External Source File`, compile e abra
   o `OB1` em LAD. Importe o `.sdf` em `Symbols` para os projetos 1, 2 e 13.
3. Para `.scl`, adicione a fonte em `Sources` e compile no editor S7-SCL.
   A chamada `FBn.DBn()` no `OB1` cria o DB de instancia automaticamente.
4. Os projetos 8 e 9 reutilizam `FB107`: compile antes o bloco da fonte do
   Projeto 7 (sem o `OB1` desse projeto). O Projeto 9 reutiliza ainda `FB108`
   da fonte do Projeto 8. O Projeto 14 reutiliza `FB111` do Projeto 11.
5. Zere a memoria no PLCSIM antes de cada projeto. Confira a compilacao,
   tempos, intertravamentos e o mapa de I/O antes de testar em bancada.

## Enderecos e comportamento

Projetos 1 a 6, 11 e 13: `I0.0` pedido de pedestre. Em `QB0`, bits 0 a 2
sao verde/amarelo/vermelho da via 1; bits 3 a 5, da via 2; bits 6 e 7,
verde/vermelho de pedestre. O ciclo normal e 30, 4, 2, 30, 4 e 2 segundos.
O pedido fica pendente ate terminar o amarelo; a travessia dura 15 + 5
segundos. No Projeto 3 a escrita e feita em BYTE: 140, 148, 164, 161,
162 e 164 no ciclo normal. Projetos 4 e 6 usam exatamente um timer
para a sequencia de etapas; o Projeto 4 gera pulsos de 500 ms para o
contador; o Projeto 6 usa `S_OFFDT` e tabelas em `UDT106`.

Projeto 5: `I0.1` e `I0.2` recebem pulsos dos detectores das vias 1 e 2.
O fluxo e medido em janelas de 60 s e escolhe verde de 20 s (<20
veiculos/min), 40 s (20 a 40) ou 60 s (>40). Entre **22:00 e 06:00**,
horario lido do relogio da CPU (`SFC1`), as duas luzes amarelas piscam.
Esse horario e facilmente alterado na expressao `noite` de `FB105`.

Projeto 7: `I0.0` pulso frente, `I0.1` dado frente, `I0.2` pulso tras,
`I0.3` dado tras, `I0.4` reset; saídas `Q0.0..Q0.7`. Pulsos nas duas
direcoes no mesmo scan se cancelam.

Projetos 8 e 9: `Q0.0..Q0.5` sao 6 vermelhas, `Q0.6` e amarela,
`Q1.0..Q1.5` sao 6 verdes. Os tempos sao os quatro parametros da chamada
`FB108.DB108` no `OB1`; mude somente esses parametros para flexibilizar
as duracoes. Sao Jose acende a primeira lampada e desloca o bit 1.
Maceio acende todas e desloca o bit 0.

Projeto 10: `I0.0` inicia; `I0.1` para. O comando 0..4095 e espelhado em
`MW100` e enviado a `PQW256` (ajuste o endereco conforme o modulo). Tick
de 100 ms: rampa lenta de 32 unidades/tick, rapida de 128 unidades/tick;
patamares de 3 s, 5 s e 3 s. Ao chegar a zero, aguarda novo inicio.

Projeto 12: `I0.0` CU, `I0.1` CD, `I0.2` reset, `I0.3` load, PV=10
no exemplo. `Q0.0` QU, `Q0.1` QD e `MW100` CV. O FB pode ser chamado com
outro PV em outros programas.

Projeto 14: `I0.0` manual frente, `I0.1` manual reverso, `I0.2`
retoma automatico, `I0.3` para. `Q0.0` frente, `Q0.1` reverso.
Na partida, frente liga. No automatico, a direcao alterna a cada 5 s.
O comando manual muda imediatamente e interrompe o automatico; para
retomar, pulse `I0.2`. Comandos manuais simultaneos desligam as duas
saidas. O FB111 representa cada passo do sequenciador.

## Limite de verificacao

As fontes foram revisadas estaticamente neste ambiente. A compilacao no
STEP 7 e a execucao no PLCSIM/CLP da bancada ainda precisam ser feitas.
Em especial, confirme disponibilidade do S7-SCL, dos timers/contadores
numerados, do relogio (`SFC1`/`SFC64`) e o endereco da saida analogica.
