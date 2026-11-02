# financeiro: instruções para sessões do Claude

Aplicação de controle financeiro da família. Docs em português.

**Fase atual: especificação.** Nada é implementado ainda. Para continuar, retome o brainstorm a partir de `docs/brainstorm-controle-financeiro.md` (skill `brainstorm`, modo retomada): seções "Estados" e "Pendências".

Fluxo de documentos:

1. `docs/brainstorm-controle-financeiro.md`: decisões, premissas e pendências (base para o resto).
2. `docs/prds/NNN-nome.md`: um PRD por funcionalidade (skill `escrever-prd`).
3. `docs/trd.md` e `docs/adrs/`: arquitetura e decisões técnicas (skill `escrever-trd`).

Estrutura do repositório: `docs/estrutura-do-repositorio.md`.

Requisito central: **privacidade**. Os dados financeiros não passam por serviços de terceiros; sem Open Finance.
