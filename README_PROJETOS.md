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
| 10 | `Projeto10/Projeto10_Motor.scl` | Quatro rampas e tres patamares em OB35 |
| 11 | `Projeto11/Projeto11_Grafcet.scl` | Passos GRAFCET com ramificacao OU |
| 12 | `Projeto12/Projeto12_CTUD.scl` | FB contador crescente/decrescente IEC |
| 13 | `Projeto13/Projeto13_Grafcet_LAD.awl` | GRAFCET Ladder passo a passo |
| 14 | `Projeto14/Projeto14_ChaveReversora.scl` | Reversao automatica de 5 s e manual |

## Tabela de simbolos minima

Os Projetos 1, 2 e 13 sao os unicos que trazem arquivo `.sdf`. Depois da
revisao de simplificacao, cada SDF possui **somente 9 linhas**:

- `I0.0`: botoeira de pedestre;
- `Q0.0..Q0.7`: oito lampadas.

Bits M e timers T internos nao precisam de simbolos globais para funcionar e,
portanto, ficaram apenas como enderecos absolutos nas redes. Isso evita
preencher dezenas de linhas no Symbol Editor.

Os projetos escritos em SCL mantem seu estado interno nos DBs de instancia e
tambem nao exigem uma tabela global de simbolos para essas variaveis.

## Como importar

1. Crie um **S7 Program diferente para cada projeto**, pois cada fonte contem
   o seu proprio `OB1`. O Projeto 10 contem tambem `OB35` para a base de tempo
   deterministica de 100 ms.
2. Para `.awl`, use `Sources > Insert > External Source File`, compile e abra
   o `OB1` em LAD. Importe o `.sdf` apenas para os projetos 1, 2 e 13.
3. Para `.scl`, adicione a fonte em `Sources` e compile no editor S7-SCL.
   A chamada `FBn.DBn()` no `OB1` cria o DB de instancia automaticamente.
4. Os projetos 8 e 9 reutilizam `FB107`: compile antes o bloco da fonte do
   Projeto 7 (sem o `OB1` desse projeto). O Projeto 9 reutiliza ainda `FB108`
   da fonte do Projeto 8. O Projeto 14 reutiliza `FB111` do Projeto 11.
5. Zere a memoria no PLCSIM antes de cada projeto. Confira a compilacao,
   tempos, intertravamentos e o mapa de I/O antes de testar em bancada.

## Projetos 1, 2 e 13 - semaforo

`I0.0` e o pedido de pedestre. Em `QB0`, bits 0 a 2 sao
verde/amarelo/vermelho da via 1; bits 3 a 5, da via 2; bits 6 e 7,
verde/vermelho de pedestre.

O ciclo normal e 30, 4, 2, 30, 4 e 2 segundos. O pedido fica pendente ate uma
transicao segura; a travessia dura 15 s de verde + 5 s de vermelho piscante.

As revisoes simplificadas compartilham timers por duracao:

- `T0`: 30 s, usado pelas duas fases verdes;
- `T1`: 4 s, usado pelas duas fases amarelas;
- `T2`: 2 s, usado pelas duas fases de vermelho total;
- `T3`: 15 s, verde do pedestre;
- `T4`: 5 s, janela total do piscante;
- `T5`: 500 ms, alternancia do piscante.

Portanto cada um desses projetos usa **6 timers**, em vez de um timer por
etapa/fase do pisca. O Projeto 1 continua usando TOF; os Projetos 2 e 13,
TON. O Projeto 13 preserva um bit por passo para continuar representando o
GRAFCET explicitamente.

## Demais projetos

Projeto 3 escreve os estados diretamente em BYTE. Projetos 4 e 6 continuam
usando exatamente um timer conforme o requisito. O Projeto 4 gera pulsos de
500 ms para o contador; o Projeto 6 usa `S_OFFDT` e tabelas em `UDT106`.

Projeto 5 mede `I0.1` e `I0.2` em janelas de 60 s e escolhe verde de
20 s (<20 veiculos/min), 40 s (20 a 40) ou 60 s (>40). Entre 22:00 e 06:00,
lido pelo relogio da CPU (`SFC1`), as duas amarelas piscam. A revisao atual
usa apenas dois timers: janela de fluxo e etapa; o pisca e derivado de `SFC64`.

Projeto 7: `I0.0` pulso frente, `I0.1` dado frente, `I0.2` pulso tras,
`I0.3` dado tras, `I0.4` reset. O registrador interno e um unico `BYTE`,
deslocado por `SHL/SHR`, e o `OB1` copia `DB107.Data` diretamente para `QB0`.

Projetos 8 e 9: `Q0.0..Q0.5` sao 6 vermelhas, `Q0.6` e amarela,
`Q1.0..Q1.5` sao 6 verdes. Os tempos sao parametros de `FB108.DB108`.
Os grupos sao escritos em `QB0/QB1` e reutilizam o `BYTE` compactado do FB107.

Projeto 10: `I0.0` inicia; `I0.1` para. A rampa roda no `OB35`, que deve estar
configurado para **100 ms** nas propriedades da CPU. Nao ha mais timer T100.
O `OB1` apenas espelha `DB110.Command` em `MW100` e envia a `PQW256`.

Projeto 11 preserva os oito passos GRAFCET e a ramificacao OU, mas todos os
passos mutuamente exclusivos compartilham **um unico `T110`**.

Projeto 12: `I0.0` CU, `I0.1` CD, `I0.2` reset, `I0.3` load, PV=10.
`Q0.0` QU, `Q0.1` QD e `MW100` CV.

Projeto 14: `I0.0` manual frente, `I0.1` manual reverso, `I0.2`
retoma automatico, `I0.3` para. `Q0.0` frente e `Q0.1` reverso.

## Revisao arquitetural

Consulte [`ARQUITETURA_PERFORMANCE.md`](ARQUITETURA_PERFORMANCE.md) para a
comparacao com padroes Siemens e implementacoes publicas, incluindo as decisoes
de performance aplicadas a cada projeto.

## Limite de verificacao

As fontes foram revisadas estaticamente. A compilacao no STEP 7 e a execucao
no PLCSIM/CLP da bancada ainda precisam ser feitas. Em especial, a nova
revisao simplificada dos Projetos 1, 2 e 13 ainda nao foi recompilada no
STEP 7 apos a reducao de timers e simbolos.
