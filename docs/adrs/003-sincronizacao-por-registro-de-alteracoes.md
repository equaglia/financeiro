---
adr_number: "003"
status: proposto
created: 2026-11-03
supersedes: ""
superseded_by: ""
---

# ADR 003: Sincronização por registro de alterações, junção campo a campo e relógio lógico híbrido

## Contexto

Com dados locais em cada aparelho (ADR 001), edições feitas offline em dois aparelhos precisam se juntar sem perder nada (PRD 011, US03): lançamentos novos nunca conflitam; edições em campos diferentes se juntam; no mesmo campo vale o administrador sobre o dependente e, entre administradores, a mais recente; exclusão contra edição, prevalece a edição e o caso vai para a lista de conflitos. O PRD 011 pede reduzir a dependência do relógio do aparelho. O servidor precisa enviar a cada aparelho só o que o perfil pode ver (PRD 001).

## Alternativas Consideradas

- **Cópia inteira do banco a cada sincronização** — simples; mas não junta edições concorrentes e não filtra por perfil. Descartado.
- **Registro de alterações por campo, ordenado por relógio lógico híbrido (HLC), com o servidor como ordenador** — junta campo a campo, tolera relógios errados, permite filtrar por perfil no servidor. Proposto.
- **CRDTs genéricos (biblioteca de terceiros)** — junção automática robusta; mas modelo de dados mais complexo e regras de prioridade por perfil difíceis de expressar. Alternativa se o proposto se mostrar insuficiente.
- **Serviço de sincronização de terceiros** — contraria o ADR 001. Descartado.

## Decisão

*(proposta — a confirmar na revisão do TRD)*

- Cada registro tem identificador global gerado no aparelho (sem colisão entre aparelhos).
- Cada alteração vira uma entrada no registro local: registro, campo, valor, autor, aparelho e carimbo **HLC**.
- Na sincronização, o aparelho envia as entradas pendentes; o servidor aplica a regra de conflito do PRD 011 por campo, guarda o resultado e devolve ao aparelho as entradas que ele ainda não tem, **filtradas pelo perfil**.
- Exclusão é marca (lápide), não remoção física, para que a regra "edição prevalece" e a lista de conflitos funcionem.
- Envio e recebimento são idempotentes: repetir uma sincronização interrompida não duplica nada.

## Consequências

- **Positivas:** atende as regras do PRD 011; sincronização incremental e retomável; base para reconstruir o servidor a partir de um aparelho de administrador (PRD 011, US07).
- **Negativas:** o registro de alterações cresce e precisa de compactação periódica; mudar o perfil de um membro ou o dono de uma conta exige reenviar ou retirar dados dos aparelhos afetados.
- **Neutras / trade-offs aceitos:** o servidor é a referência de ordem; dois aparelhos não sincronizam entre si sem ele (fora do escopo, PRD 011).
