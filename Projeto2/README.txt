PROJETO 2 - SEMAFORO COM TON

Alvo: STEP 7 Classic / S7-300. Importe Projeto2_TON.awl em Sources,
compile e abra OB1 em LAD. Importe o arquivo SDF em Symbols.

Este projeto substitui o Projeto 1 no mesmo OB1; mantenha cada projeto
em um S7 Program separado para nao sobrescrever o anterior.

O ciclo e 30 / 4 / 2 / 30 / 4 / 2 segundos. Uma borda na botoeira
I0.0 guarda o pedido. Depois do amarelo, as duas vias ficam vermelhas,
o pedestre recebe verde por 15 segundos e vermelho piscante por 5.
Ao final, abre a via que estava fechada antes da chamada.

Teste em PLCSIM com memoria limpa (MRES) antes do primeiro RUN.
Confira os enderecos fisicos de saida antes de baixar em bancada.
