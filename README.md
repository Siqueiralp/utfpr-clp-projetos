# Projetos de Controladores Logicos Programaveis

Implementacoes dos 14 projetos da disciplina para **STEP 7 Classic / S7-300**.

## Criterio de implementacao

A prioridade deste repositorio e, nesta ordem:

1. **atender ao requisito do projeto**;
2. **usar o menor numero possivel de estados, variaveis, timers, contadores e blocos auxiliares**;
3. manter a logica simples de explicar e testar;
4. somente depois considerar otimizacoes de tempo de scan.

Isso significa preferir um estado numerico a varios bits de etapa, reutilizar
um unico timer quando as etapas sao mutuamente exclusivas e usar Clock Memory
quando ele elimina recursos adicionais sem contrariar o enunciado.

Consulte [README_PROJETOS.md](README_PROJETOS.md) para os projetos e
[ARQUITETURA_MINIMA.md](ARQUITETURA_MINIMA.md) para as decisoes de arquitetura.

Antes de simular os projetos que usam pulsos periodicos, configure
[MB10 como Clock Memory](CONFIGURAR_CLOCK_MEMORY.txt).

As fontes ainda precisam ser compiladas no STEP 7 Classic e testadas no
PLCSIM/CLP da bancada antes de serem consideradas validadas.
