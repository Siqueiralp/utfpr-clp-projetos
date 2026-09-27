# Tentativa de validação local — Projeto 1

Data: 27/09/2026. Estado: **inconclusivo**. A fonte foi importada, mas não foi possível confirmar a compilação nem executar a lógica no simulador.

## Ambiente usado

- SIMATIC Manager / STEP 7 Classic V5.7, instalado em `C:\Program Files (x86)\Siemens\Step7`.
- S7-PLCSIM instalado e inicializado localmente.
- Projeto de teste criado pelo assistente do STEP 7 em `C:\Program Files (x86)\Siemens\Step7\s7proj\S7_Pro3`. Ele está fora deste repositório.
- CPU escolhida: CPU 314C-2 DP, referência `6ES7 314-6CG03-0AB0`, MPI 2. O assistente criou um OB1 inicial.
- Fonte testada: `Projeto1_TOF.awl` desta pasta. A tabela de símbolos não foi importada, pois a lógica usa endereços absolutos.

## Procedimento e observações

1. Abri o SIMATIC Manager e criei o projeto `S7_Pro3` com a CPU acima e um OB1 inicial em STL.
2. Selecionei `S7 Program(1) > Sources` e usei `Insert > External Source...` para importar `Projeto1_TOF.awl`. A fonte apareceu como `Projeto1_TOF` na lista.
3. Acionei `Compile` no menu de contexto da fonte. O aplicativo `LAD/STL/FBD : Program blocks` abriu, mas permaneceu com a área de edição e a lista de erros vazias. Não apareceu relatório de compilação nem confirmação de que o OB1 foi substituído.
4. Abri a própria fonte e, separadamente, o OB1 criado pelo assistente pelo SIMATIC Manager. Nos dois casos, a mesma janela `LAD/STL/FBD : Program blocks` continuou vazia, sem exibir o conteúdo do objeto. Não foi possível confirmar se a tentativa de compilação havia substituído o OB1.
5. Em `File > Open` do editor, a lista continha apenas `Proj1`, `Projeto1-CLP-LAD`, `S7_Pro1` e `S7_Pro2`; o projeto novo `S7_Pro3` não aparecia. Fechei e reabri o editor pela fonte importada. Ele continuou vazio.
6. Iniciei o S7-PLCSIM. A janela `S7-PLCSIM1` abriu com a CPU em `STOP` e visualizações de `IB 0`, `QB 0`, `MB 0`, `T 0` e `T 1`. Nenhum programa do Projeto 1 foi transferido para o simulador e a CPU não foi colocada em `RUN`.

## Resultado

A importação da fonte e a inicialização do PLCSIM foram confirmadas. **Não há evidência de compilação bem-sucedida nem de funcionamento do semáforo.** O editor vazio impede ler eventuais erros e verificar o OB1 gerado. A ausência do projeto novo na lista do editor sugere um problema de reconhecimento/atualização do projeto, mas a causa não foi confirmada.

Antes de iniciar o simulador, o SIMATIC Manager mostrava `PLCSIM.MPI.1` na barra de status. Depois da abertura do PLCSIM, passou a mostrar `PLCSIM.TCPIP.1`, coerente com `PLCSIM(TCP/IP)` no simulador. Essa interface ainda não foi testada por transferência.

## Para retomar a validação

1. Fazer o editor LAD/STL/FBD abrir o OB1 inicial de `S7_Pro3` e mostrar seu conteúdo. Só então recompilar `Projeto1_TOF` e registrar a lista de erros/avisos.
2. Confirmar que a compilação substituiu o OB1 do projeto de teste e que o bloco abre com as redes esperadas.
3. Alinhar a interface PG/PC do SIMATIC Manager com a interface do PLCSIM, limpar a memória simulada (MRES), transferir o programa e iniciar `RUN`.
4. Observar o ciclo normal `30 s / 4 s / 2 s / 30 s / 4 s / 2 s` em `Q0.0..Q0.7` e testar um pulso em `I0.0` durante cada via, verificando a travessia de `15 s + 5 s` e os intertravamentos.

Até completar esses passos, manter o Projeto 1 como **não validado no STEP 7/PLCSIM**.
