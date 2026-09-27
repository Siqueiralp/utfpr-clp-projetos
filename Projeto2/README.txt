PROJETO 2 - SEMAFORO COM TON

Alvo: STEP 7 Classic / S7-300. Importe Projeto2_TON.awl em Sources,
compile e abra o OB1 em LAD.

SIMBOLOS
--------
Projeto2_Symbols.sdf possui somente 9 entradas na tabela:
I0.0 e Q0.0..Q0.7.

Os estados, flags e timers internos usam enderecos M/T diretamente e nao
precisam ser cadastrados no Symbol Editor.

SIMPLIFICACAO
-------------
Os tempos iguais compartilham o mesmo TON:
T0 = 30 s (etapas 1 e 4)
T1 = 4 s  (etapas 2 e 5)
T2 = 2 s  (etapas 3 e 6)
T3 = 15 s (pedestre verde)
T4 = 5 s  (pedestre piscante)

Total: 5 timers. A alternancia de 500 ms usa M10.5 (Clock Memory 1 Hz).
Configure MB10 como Clock Memory antes de simular.

O ciclo continua 30 / 4 / 2 / 30 / 4 / 2 segundos. Uma borda em I0.0
memoriza o pedido. Depois do amarelo, as duas vias ficam vermelhas,
o pedestre recebe verde por 15 s e vermelho piscante por 5 s. Ao final,
abre a via que estava fechada antes da chamada.

Mantenha cada projeto em um S7 Program separado para nao sobrescrever OB1.
Teste em PLCSIM com MRES antes do primeiro RUN.
