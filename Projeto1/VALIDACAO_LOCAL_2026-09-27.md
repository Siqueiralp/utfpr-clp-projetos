# Tentativa de validação local — Projeto 1

> **Nota histórica:** este registro descreve a tentativa de 27/09/2026.
> Desde então o Projeto 1 foi revisado para a versão 0.4. A versão atual usa
> um único timer `T0`, I/O simbólico e foi preparada para a CPU física
> **313C-2 DP 6ES7 313-6CE01-0AB0** observada na bancada.

Data da tentativa: 27/09/2026. Estado: **inconclusivo**. A fonte foi importada,
mas não foi possível confirmar a compilação nem executar a lógica no simulador.

## Ambiente usado na tentativa histórica

- SIMATIC Manager / STEP 7 Classic V5.7.
- S7-PLCSIM.
- Foi criada uma estação de teste com CPU 314C-2 DP; essa CPU **não corresponde**
  ao hardware físico posteriormente identificado.
- A fonte testada era anterior à versão 0.4 e usava endereços absolutos de I/O.

## Situação atual — versão 0.4

Hardware físico identificado:

- CPU: SIMATIC S7-300 CPU 313C-2 DP.
- Referência: `6ES7 313-6CE01-0AB0`.
- I/O digital integrado: 16 DI + 16 DO.
- Endereços padrão do I/O integrado: `I124.0..I125.7` e
  `Q124.0..Q125.7`.

A Symbol Table atual parte desses endereços padrão e associa:

- painel `I0` a `I124.0`;
- painel `O0..O7` a `Q124.0..Q124.7`.

Isso ainda precisa ser confirmado no **HW Config da bancada**, pois os endereços
podem ter sido remapeados. A fonte AWL foi alterada para usar símbolos de I/O;
portanto, se o mapeamento real for diferente, basta alterar a SDF.

## Correções aplicadas na versão 0.4

1. A lógica continua usando apenas um temporizador: `T0`.
2. Foram removidos os atalhos `1 -> 6` e `4 -> 8`.
3. A travessia de pedestre só é iniciada após o estado de 2 s com ambas as
   vias vermelhas.
4. Entradas e saídas externas passaram a ser simbólicas.
5. O procedimento de deploy não recomenda mais MRES automático; primeiro deve
   ser feito backup da estação física.

## Validação necessária na bancada

1. Fazer `Upload Station to PG` e guardar backup.
2. Confirmar a CPU e os endereços em `HW Config`.
3. Confirmar/ajustar `Projeto1_Symbols.sdf`.
4. Configurar `MB10` como Clock Memory.
5. Importar a Symbol Table antes da fonte AWL.
6. Compilar `Projeto1_TOF.awl` e exigir **0 errors**.
7. Transferir OB1 para a CPU em STOP.
8. Colocar a CPU em RUN.
9. Monitorar `MW20`, `T0`, a entrada do botão e as oito saídas.
10. Validar o ciclo `30 / 4 / 2 / 30 / 4 / 2 s`.
11. Validar o pedestre `15 + 5 s` sem eliminar o intervalo de ambas vermelhas.

Até completar esses passos, o Projeto 1 continua **preparado, mas não validado
fisicamente**.
