# financeiro: instruções para sessões do Claude

Aplicação de controle financeiro da família. Docs em português.

**Fase atual: especificação.** Nada é implementado ainda.

## Onde paramos (2026-11-03)

- Brainstorm concluído e consolidação validada pelo casal: `docs/brainstorm-controle-financeiro.md` (decisões, estados, pendências e a divisão em 11 PRDs na seção "Consolidação").
- **Processo de revisão dos PRDs:** cada PRD passa por uma 1ª revisão do casal logo depois de escrito (campo `revisao` no frontmatter) e continua `rascunho` ("quase pronto"). Quando todos os 11 estiverem escritos, o casal revisa todos de novo; só então passam a `pronto`, antes da implementação.
- PRD 001 (usuários, perfis e acesso): 1ª revisão aprovada em 2026-11-03.
- PRD 002 (contas e cartões): 1ª revisão aprovada em 2026-11-03 (dinheiro em espécie entrou no escopo; US12 contas administradas aprovada depois).
- PRD 003 (lançamentos e categorias): 1ª revisão aprovada em 2026-11-03.
- PRD 005 (migração do Minhas Economias): escrito antes do 004 (a exportação já estava em mãos); 1ª revisão aprovada em 2026-11-03. Base: `docs/formatos/minhas-economias-exportacao.md` e `docs/categorias-iniciais.md` (proposta de agrupamento, o casal ajusta).
- PRD 006 (investimentos e previdência): 1ª revisão aprovada em 2026-11-03.
- PRD 007 (orçamento): aguardando a 1ª revisão.
- PRD 008 (relatórios e patrimônio): aguardando a 1ª revisão.
- PRD 009 (filhos: limite e semanada): aguardando a 1ª revisão.
- PRD 010 (apoio ao IR): aguardando a 1ª revisão. Fontes da Receita consultadas em 2026-11-03 (seção 8); limite de educação e conversão do exterior a verificar.
- PRD 011 (sincronização e operação): aguardando a 1ª revisão.
- PRD 004 (importação e conciliação): **próximo**, quando o casal enviar um extrato ou fatura de cada instituição (BB, Santander, XP, BBPrev, exterior). Todos os outros PRDs já estão escritos. Depois do 004: revisão final de todos (coerência entre PRDs), depois TRD e ADRs (skill `escrever-trd`) e glossário.
- Amostras: o casal anexa na conversa; analisar numa pasta isolada do scratchpad; **nunca gravar no repositório** (só o formato, sem nomes nem valores). Nomes de pessoas não entram nos documentos.

Fluxo de documentos:

1. `docs/brainstorm-controle-financeiro.md`: decisões, premissas e pendências (base para o resto).
2. `docs/prds/NNN-nome.md`: um PRD por funcionalidade (skill `escrever-prd`).
3. `docs/trd.md` e `docs/adrs/`: arquitetura e decisões técnicas (skill `escrever-trd`).

Estrutura do repositório: `docs/estrutura-do-repositorio.md`.

Requisito central: **privacidade**. Os dados financeiros não passam por serviços de terceiros; sem Open Finance.
