---
adr_number: "001"
status: aceito
created: 2026-11-03
supersedes: ""
superseded_by: ""
---

# ADR 001: Dados locais em cada aparelho, com servidor doméstico e sincronização só na rede de casa

## Contexto

O requisito central do projeto é a privacidade: a família saiu do Minhas Economias quando ele passou a exigir Open Finance, e os dados financeiros não podem passar por serviços de terceiros que possam lê-los. Ao mesmo tempo, o casal lança gastos cada um no seu celular e precisa ver os mesmos dados; os filhos têm acesso restrito. O servidor disponível é um computador Windows de casa que **nem sempre fica ligado**, e não pode haver custo adicional. Decidido no brainstorm (rodadas 2 e 3) e validado na consolidação.

## Alternativas Consideradas

- **Cliente-servidor clássico (dados só no servidor)** — simples de consistir; mas nada funciona com o servidor desligado ou fora de casa. Descartado.
- **Servidor alugado (VPS) ou nuvem** — sempre disponível; mas tem custo e põe os dados num terceiro. Descartado.
- **Aparelhos sincronizando por um serviço de nuvem com criptografia ponta a ponta** — privado e sempre disponível; mas o mais difícil de construir e depende de um terceiro para o transporte. Descartado.
- **Local-first com servidor doméstico, sincronização só no Wi-Fi de casa** — cada aparelho guarda sua cópia e funciona offline; o servidor é ponto de encontro e cópia de referência; nada sai de casa. Escolhido.

## Decisão

Cada aparelho mantém um banco de dados local completo para o que o seu perfil pode ver e funciona sem o servidor. O servidor doméstico guarda a cópia de referência, sincroniza os aparelhos quando estão no Wi-Fi de casa e faz as cópias de segurança. Não há acesso ao servidor de fora de casa (sem VPN, túnel ou porta aberta na internet).

## Consequências

- **Positivas:** privacidade máxima (dados nunca saem da rede de casa, exceto a cópia de segurança criptografada); custo zero; o app funciona sempre.
- **Negativas:** o que um lança na rua só chega aos outros depois que os dois aparelhos sincronizarem em casa; é preciso resolver conflitos de edição offline (ADR 003); o sistema depende de cópia de segurança disciplinada (PRD 011).
- **Neutras / trade-offs aceitos:** o servidor filtra os dados por perfil antes de enviar (PRD 001), então o aparelho do dependente não tem cópia da família inteira.
