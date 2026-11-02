# Estrutura do repositório

```
financeiro/
├── CLAUDE.md                      # instruções para as sessões do Claude
├── README.md
├── docs/
│   ├── brainstorm-controle-financeiro.md
│   ├── estrutura-do-repositorio.md
│   ├── glossario.md               # (a criar) termos: conta, cartão adicional, fatura, categoria…
│   ├── categorias-iniciais.md     # proposta de grupos e categorias (lista inicial e mapa da migração)
│   ├── formatos/                  # formatos de arquivos de importação e migração (sem dados reais)
│   ├── prds/                      # um PRD por funcionalidade (001-usuarios-perfis-e-acesso.md, …)
│   ├── trd.md                     # arquitetura, stack, não-funcionais
│   └── adrs/                      # uma decisão técnica por arquivo (001–005)
├── .agents/skills/                # skills: brainstorm, escrever-prd, escrever-trd
└── .claude/skills/                # links para .agents/skills
```

As pastas de código (`shared/`, `app-android/`, `app-desktop/`, `servidor/`, `empacotamento/`, `scripts/`, `gradle/`) estão propostas no TRD, seção Arquitetura, e serão criadas na primeira fatia de implementação.
