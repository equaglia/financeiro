# financeiro: instruções para sessões do Claude

Aplicação de controle financeiro da família. Docs em português.

**Fase atual: especificação.** Nada é implementado ainda.

## Onde paramos (2026-11-03)

- Brainstorm concluído e consolidação validada pelo casal: `docs/brainstorm-controle-financeiro.md` (decisões, estados, pendências e a divisão em 11 PRDs na seção "Consolidação").
- **Processo de revisão dos PRDs:** cada PRD passa por uma 1ª revisão do casal logo depois de escrito (campo `revisao` no frontmatter) e continua `rascunho` ("quase pronto"). Quando todos os 11 estiverem escritos, o casal revisa todos de novo; só então passam a `pronto`, antes da implementação.
- PRD 001 (usuários, perfis e acesso): 1ª revisão aprovada em 2026-11-03.
- PRD 002 (contas e cartões): 1ª revisão aprovada em 2026-11-03 (dinheiro em espécie entrou no escopo).
- PRD 003 (lançamentos e categorias): `rascunho`, **em revisão pelo casal**. Ao retomar: aplicar as correções nas premissas marcadas *(premissa — confirme ou corrija)*, registrar a 1ª revisão no campo `revisao` e seguir para o PRD 004 (importação e conciliação), na ordem da tabela da consolidação (skill `escrever-prd`).
- Pedir as amostras de extratos, faturas e da exportação do Minhas Economias (com dados pessoais apagados) quando chegar o PRD 004/005. Não gravar amostras no repositório; só a descrição do formato.

Fluxo de documentos:

1. `docs/brainstorm-controle-financeiro.md`: decisões, premissas e pendências (base para o resto).
2. `docs/prds/NNN-nome.md`: um PRD por funcionalidade (skill `escrever-prd`).
3. `docs/trd.md` e `docs/adrs/`: arquitetura e decisões técnicas (skill `escrever-trd`).

Estrutura do repositório: `docs/estrutura-do-repositorio.md`.

Requisito central: **privacidade**. Os dados financeiros não passam por serviços de terceiros; sem Open Finance.
