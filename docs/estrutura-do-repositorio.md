# Estrutura do repositório

```
financeiro/
├── CLAUDE.md                      # instruções para as sessões do Claude
├── README.md
├── docs/
│   ├── brainstorm-controle-financeiro.md
│   ├── estrutura-do-repositorio.md
│   ├── glossario.md               # (a criar) termos: conta, cartão adicional, fatura, categoria…
│   ├── prds/                      # (a criar) 001-lancamentos.md, 002-…
│   ├── trd.md                     # (a criar) arquitetura, stack, não-funcionais
│   └── adrs/                      # (a criar) uma decisão técnica por arquivo
├── .agents/skills/                # skills: brainstorm, escrever-prd, escrever-trd
└── .claude/skills/                # links para .agents/skills
```

As pastas de código serão definidas no TRD, porque dependem da plataforma (web, celular ou ambos) e de onde os dados ficam.
