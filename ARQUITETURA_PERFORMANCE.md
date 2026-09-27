# Revisao arquitetural e de performance

Esta revisao compara as implementacoes da disciplina com padroes encontrados
em documentacao Siemens e em projetos de automacao publicos. O objetivo nao e
usar a abstracao mais sofisticada possivel, mas reduzir estado persistente,
temporizadores e operacoes repetidas sem esconder o conceito exigido em cada
projeto.

## Principios adotados

1. **Estado compacto + CASE para sequencias em SCL**
   - Um numero de estado representa a fase ativa.
   - CASE concentra as transicoes e evita cadeias extensas de flags.
   - Mantivemos GRAFCET one-hot apenas quando o proprio GRAFCET e requisito.

2. **I/O compactado em BYTE quando os sinais formam um vetor**
   - Registradores de deslocamento e grupos de lampadas sao dados de 8 bits.
   - SHL/SHR e mascaras substituem movimentacoes BOOL repetitivas.
   - A escrita de QB0/QB1 e feita em bloco quando isso nao prejudica o requisito.

3. **Um temporizador por recurso temporal simultaneo**
   - Se apenas uma etapa pode estar ativa, um timer pode ser reutilizado.
   - Nao ha ganho em reservar oito timers para oito passos mutuamente exclusivos.

4. **Clock Memory para sinais periodicos simples**
   - A propria Siemens recomenda Clock Memory para luzes piscantes e atividades
     periodicas simples.
   - Com MB10 configurado, M10.5 fornece periodo de 1 s sem timer de usuario e
     sem chamada de SFC em cada scan.
   - O Projeto 5 usa M10.7 para limitar a releitura do relogio de tempo real.

5. **Base de tempo deterministica quando a taxa fixa e parte do comportamento**
   - O Projeto 10 usa OB35 a 100 ms em vez de construir um tick de 100 ms com
     um timer chamado no OB1.
   - Isso separa a rampa periodica do tempo de ciclo variavel do OB1.

6. **Evitar enderecamento indireto apenas para reduzir linhas**
   - Tabelas e arrays continuam onde sao requisito ou reduzem realmente a logica.
   - No S7-300, enderecamento indireto tem custo adicional de carregar o endereco
     antes de executar a instrucao; portanto, "data-driven" nao e automaticamente
     mais rapido.

7. **Estado persistente somente para informacao que atravessa scans**
   - Valores intermediarios foram movidos para VAR_TEMP quando apropriado.
   - Flags de resultado de timer, mascaras e valores derivados nao precisam ocupar
     o DB de instancia permanentemente.

## Referencias de arquitetura comparadas

### Siemens S7-SCL para S7-300/400

O manual S7-SCL mostra CASE como mecanismo direto de selecao entre alternativas
e fornece SHL/SHR para BYTE, WORD e DWORD. O proprio exemplo Siemens usa
deslocamento e mascara para extrair campos de bits.

Fonte:
https://support.industry.siemens.com/cs/attachments/18735131/GS_SCL_e.pdf

### Siemens S7-300 - custo de enderecamento indireto

A lista de instrucoes do S7-300 define o tempo de uma instrucao com
enderecamento indireto como:

    carregar endereco + executar instrucao

Por isso, uma tabela dinamica pode melhorar manutencao sem necessariamente
melhorar o tempo de scan.

Fonte:
https://support.industry.siemens.com/cs/attachments/8861817/opli312bis318_e.pdf

### Clock Memory para pulsos periodicos

O manual de programacao STEP 7 define o Clock Memory como um byte atualizado
periodicamente pela CPU e cita explicitamente luzes piscantes como aplicacao.
No mapeamento padrao, bit 5 tem periodo de 1,0 s (1 Hz). Essa revisao usa
MB10/M10.5 para o pisca.

Fonte:
https://cache.industry.siemens.com/dl/files/056/18652056/att_70829/v1/S7prv54_e.pdf

### OB35 para tarefas periodicas

A Siemens usa OB35 como cyclic interrupt no S7-300. O intervalo padrao e
100 ms em diversos exemplos e pode ser configurado nas propriedades da CPU.
Esse mecanismo e mais adequado para uma rampa que deve executar em uma base
de tempo fixa do que um tick sintetizado dentro do OB1.

Fontes:
https://support.industry.siemens.com/cs/attachments/109763709/109763709_SEL_STEP7_V15_LIB_V10_en.pdf
https://support.industry.siemens.com/cs/attachments/68624711/68624711_sinamics_s120_pn_at_s7-300400f_docu_v2d0_en.pdf

### Exemplos publicos de sequenciadores

Projetos Siemens/TIA publicos adotam maquinas de estado SCL com passo numerico,
CASE, deteccao de mudanca de passo e blocos modulares. Um exemplo mais generico
usa estruturas de parametros para passos e um sequenciador reutilizavel.

Referencias:
https://github.com/sofianlap2/Automated-Sorting-Conveyor-Siemens-TIA-Portal
https://github.com/Raj-11-Bag/Siemens-S7-1500-SCL-Conveyor-Sorting-System
https://github.com/Adam-plc-code/Mawomat/blob/main/Sequence1.scl

## Alteracoes aplicadas

| Projeto | Arquitetura aplicada | Resultado estrutural |
| --- | --- | --- |
| 1 | LAD/TOF explicito preservado | 5 timers + Clock Memory; Symbol Table minima |
| 2 | LAD/TON explicito preservado | 5 timers + Clock Memory; Symbol Table minima |
| 3 | FSM compacta + CASE + QB0 | 2 timers -> 1 + Clock Memory |
| 4 | FSM compacta + temporarios locais | Mantem exatamente 1 timer + 1 contador |
| 5 | FSM compacta + SFC64 para pisca | 3 timers -> 2 + Clock Memory |
| 6 | Sequenciador orientado a tabela | Mantem 1 TOF; Clock Memory para pisca |
| 7 | Registrador em BYTE + SHL/SHR | 8 BOOL de estado -> 1 BYTE |
| 8/9 | FB107 compactado + QB0/QB1 | Grupos de lampadas escritos por byte |
| 10 | FSM executada em OB35 | Remove T100; tick real de 100 ms |
| 11 | GRAFCET one-hot + timer compartilhado | 8 timers -> 1 + Clock Memory |
| 12 | CTUD compacto | Sem mudanca relevante |
| 13 | GRAFCET LAD preservado | 6 timers; Symbol Table minima |
| 14 | FB de passo reutilizavel | Estrutura existente preservada |

## Correcao apos comparar tempos de instrucao

Uma versao intermediaria desta revisao usava `SFC64 TIME_TCK` para gerar o
pisca sem reservar outro timer. A lista de instrucoes do S7-300 mostra que
`SFC64` custa dezenas de microssegundos em CPUs 31x, enquanto iniciar um timer
S7 diretamente fica na faixa de poucos microssegundos. Portanto "eliminar um
timer" via chamada de sistema nao era uma otimizacao de CPU.

A versao final usa **Clock Memory**, que e gerado pela propria CPU e requer
somente a leitura de um bit M no programa. Essa foi uma mudanca motivada por
performance real, nao apenas contagem de recursos.

Fontes:
https://support.industry.siemens.com/cs/attachments/13206730/s7300_instruction_list_en-US.pdf
https://cache.industry.siemens.com/dl/files/056/18652056/att_70829/v1/S7prv54_e.pdf

## Por que nao transformar tudo em uma tabela generica

Uma arquitetura totalmente data-driven poderia guardar saida, tempo,
proximo estado e condicoes em arrays. Ela reduziria codigo-fonte em alguns
projetos, mas teria tres desvantagens aqui:

- esconderia os conceitos LAD, TON, TOF e GRAFCET que fazem parte do exercicio;
- aumentaria o uso de indexacao/endereco indireto no S7-300;
- tornaria a apresentacao e o debug no STEP 7 Classic menos transparentes.

Por isso a arquitetura mais generica ficou restrita ao Projeto 6, onde matriz
e UDT sao parte do proprio requisito.

## Configuracao obrigatoria do Projeto 10

A fonte agora contem OB35. No HW Config da CPU 314C-2 DP:

1. Abra as propriedades da CPU.
2. Localize **Cyclic Interrupts**.
3. Confirme **OB35 = 100 ms**.
4. Compile a configuracao de hardware e o programa.
5. Carregue ambos no PLCSIM/CPU.

O FB110 e chamado somente pelo OB35. O OB1 apenas copia o comando calculado
para MW100 e PQW256.

O botao Start/Stop e amostrado pelo OB35 a cada 100 ms. Isso e suficiente para
botoeiras normais de laboratorio. Se o enunciado exigir captura de pulsos mais
curtos que 100 ms, a entrada deve ser latched no OB1 antes de ser consumida
pelo OB35.

## Validacao

As reducoes acima sao arquiteturais e estaticamente revisadas. Nao representam
benchmark de tempo de scan medido na CPU.

Antes da entrega final ainda e necessario:

- compilar todas as fontes no STEP 7 Classic;
- confirmar a assinatura das funcoes SCL SHL/SHR na instalacao usada;
- habilitar MB10 como Clock Memory nos projetos documentados;
- confirmar OB35 em 100 ms no Projeto 10;
- executar os cenarios no PLCSIM;
- medir cycle time antes/depois se for desejada uma comparacao quantitativa de
  performance.
