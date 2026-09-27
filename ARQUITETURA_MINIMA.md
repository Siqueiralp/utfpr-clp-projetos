# Arquitetura minima de recursos

## Regra principal

A arquitetura e escolhida pela menor quantidade de recursos que ainda atende
ao enunciado. Desempenho de CPU e criterio secundario.

A ordem de preferencia e:

1. eliminar estado redundante;
2. codificar contexto no proprio numero do estado;
3. compartilhar timers entre etapas mutuamente exclusivas;
4. usar Clock Memory para pulsos periodicos simples;
5. reutilizar bytes/words em vez de colecoes de BOOL;
6. evitar FBs auxiliares quando um estado simples resolve o problema.

## Padroes aplicados

### Estado numerico

Nos semaforos SCL e nos Projetos 1/2, estados diferentes carregam tambem o
contexto de retorno do pedestre:

- 6/7 retornam para a via 1;
- 8/9 retornam para a via 2.

Isso elimina uma variavel `RetomaVia1`.

### Um timer para etapas exclusivas

Se somente uma etapa pode estar ativa, nao existe motivo funcional para
reservar um timer para cada etapa. Projetos 1, 2, 3, 6, 11, 13 e 14 usam um
unico timer por sequenciador.

Projeto 4 mantem exatamente um timer e um contador porque isso faz parte do
requisito.

### Clock Memory no lugar de infraestrutura adicional

O STEP 7 permite configurar um byte de Clock Memory atualizado pela propria CPU.
Com MB10:

- M10.0: 10 Hz / periodo 100 ms;
- M10.5: 1 Hz / periodo 1 s;
- M10.7: 0,5 Hz / periodo 2 s.

Isso permite:

- piscar lampadas sem timer adicional;
- executar os deslocamentos 8/9 sem T80;
- executar as rampas do Projeto 10 sem T100 nem OB35;
- limitar as leituras do RTC no Projeto 5.

A Siemens documenta Clock Memory justamente para luzes piscantes e atividades
periodicas simples:
https://cache.industry.siemens.com/dl/files/056/18652056/att_70829/v1/S7prv54_e.pdf

### BYTE para registradores e grupos de saida

Projeto 7: oito BOOL de estado foram substituidos por um BYTE.
Projetos 8/9 reutilizam uma unica instancia desse registrador.

### Variaveis temporarias

Valores que nao precisam sobreviver ao scan ficam em VAR_TEMP. No Projeto 6,
inclusive a UDT com as tabelas de estados/tempos e temporaria; somente etapa,
pedido, borda do botao e pulso do TOF persistem.

## Recursos atuais

| Projeto | Timers | Contadores | Estado persistente principal |
| --- | ---: | ---: | --- |
| 1 | 1 | 0 | MW20 + MW22 + 4 bits internos |
| 2 | 1 | 0 | MW20 + MW22 + 4 bits internos |
| 3 | 1 | 0 | 4 variaveis |
| 4 | 1 | 1 | 5 variaveis |
| 5 | 1 | 2 | 10 variaveis |
| 6 | 1 | 0 | 4 variaveis |
| 7 | 0 | 0 | 1 BYTE de saida do FB |
| 8/9 | 0 | 0 | 1 FB107 + 3 variaveis |
| 10 | 0 | 0 | 4 variaveis |
| 11 | 1 | 0 | 4 variaveis |
| 12 | 0 | 0 | CV + 2 memorias de borda |
| 13 | 1 | 0 | bits de passo + MW20 |
| 14 | 1 | 0 | 3 variaveis |

## Excecao deliberada: Projeto 13

Seria possivel substituir os bits de passo por um unico INT, como no Projeto 11.
Isso reduziria ainda mais a memoria, mas deixaria de representar diretamente o
metodo GRAFCET Ladder passo a passo pedido no projeto. Por isso os bits foram
preservados **somente nesse projeto**.

## Comparacao com arquiteturas mais genericas

Sequenciadores parametrizados, arrays de passos e DRUM/DRUM_X sao uteis em
sistemas maiores, mas adicionam estruturas que estes exercicios pequenos nao
precisam. Aqui, "mais industrial" nao significa automaticamente "mais simples".

A regra adotada e: se uma abstracao nao elimina mais recursos do que adiciona,
ela nao entra.

## Validacao

Esta e uma revisao estatica. Antes da entrega:

- configurar MB10 como Clock Memory;
- compilar todas as fontes no STEP 7 Classic;
- verificar a conversao/visualizacao LAD dos Projetos 1, 2 e 13;
- executar os ciclos completos no PLCSIM;
- conferir enderecos fisicos de I/O.
