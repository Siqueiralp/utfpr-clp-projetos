# Projetos de Controladores Logicos Programaveis

Implementacoes dos 14 projetos de CLP da disciplina, organizadas em
`Projeto1` a `Projeto14` para STEP 7 Classic / S7-300.

Consulte [a documentacao dos projetos](README_PROJETOS.md) para a ordem,
o mapa de entradas e saidas, as dependencias entre blocos e a importacao
no SIMATIC Manager.

## Revisao de simplificacao

Os Projetos 1, 2 e 13 agora usam tabelas SDF com apenas **9 simbolos de I/O**
e **5 timers de etapa compartilhados por duracao**; o pisca usa Clock Memory. Os enderecos M/T internos ficaram
fora da Symbol Table para evitar cadastro manual desnecessario, mantendo a
estrutura LAD/GRAFCET explicita.

Uma segunda revisao arquitetural aplicou estado compacto em SCL, registradores
em `BYTE`, timers compartilhados, Clock Memory em `MB10` para sinais periodicos
e `OB35` deterministico no Projeto 10.
Antes de simular, veja tambem [CONFIGURAR_CLOCK_MEMORY.txt](CONFIGURAR_CLOCK_MEMORY.txt).
Veja [ARQUITETURA_PERFORMANCE.md](ARQUITETURA_PERFORMANCE.md) para as referencias
e decisoes projeto a projeto.

As fontes ainda precisam ser compiladas no STEP 7 e testadas no PLCSIM
ou no CLP da bancada antes de serem consideradas validadas.

Uma [tentativa de validacao local do Projeto 1](Projeto1/VALIDACAO_LOCAL_2026-09-27.md)
registra a importacao no STEP 7 V5.7 e a abertura do PLCSIM. Ela ocorreu antes
da revisao de simplificacao e terminou inconclusiva porque o editor LAD/STL/FBD
nao exibiu nem a fonte nem o OB1 inicial do projeto de teste.
