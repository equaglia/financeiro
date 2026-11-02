# financeiro: instruções para sessões do Claude

Aplicação de controle financeiro da família. Docs em português.

**Fase atual: especificação.** Nada é implementado ainda.

## Onde paramos (2026-11-03)

- Brainstorm concluído e consolidação validada pelo casal: `docs/brainstorm-controle-financeiro.md` (decisões, estados, pendências e a divisão em 11 PRDs na seção "Consolidação").
- PRD 001 (usuários, perfis e acesso): `pronto` (premissas aceitas em 2026-11-03).
- PRD 002 (contas e cartões): `pronto` (revisado em 2026-11-03; dinheiro em espécie entrou no escopo).
- PRD 003 (lançamentos e categorias): `rascunho`, **em revisão pelo casal**. Ao retomar: aplicar as correções nas premissas marcadas *(premissa — confirme ou corrija)*, passar para `pronto` e seguir para o PRD 004 (importação e conciliação), na ordem da tabela da consolidação (skill `escrever-prd`).
- Pedir as amostras de extratos, faturas e da exportação do Minhas Economias (com dados pessoais apagados) quando chegar o PRD 004/005. Não gravar amostras no repositório; só a descrição do formato.

Fluxo de documentos:

1. `docs/brainstorm-controle-financeiro.md`: decisões, premissas e pendências (base para o resto).
2. `docs/prds/NNN-nome.md`: um PRD por funcionalidade (skill `escrever-prd`).
3. `docs/trd.md` e `docs/adrs/`: arquitetura e decisões técnicas (skill `escrever-trd`).

Estrutura do repositório: `docs/estrutura-do-repositorio.md`.

Requisito central: **privacidade**. Os dados financeiros não passam por serviços de terceiros; sem Open Finance.
