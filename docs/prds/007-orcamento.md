---
prd_number: "007"
status: rascunho
revisao: "aguardando a 1ª revisão do casal"
priority: alta
created: 2026-11-03
issue: ""
depends_on: ["001", "003"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/003-lancamentos-e-categorias.md
  - docs/categorias-iniciais.md
---

# PRD 007: Orçamento

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: a planilha registra receitas e despesas, mas não tem orçamento; não há como comparar o planejado com o realizado.
- **Problema**: o casal pediu "orçamento planejado contra o realizado" (rodada 4 do brainstorm). Sem orçamento por categoria, só se descobre que o mês estourou depois de pagar a fatura.

> Categorias, mês de referência (regime de caixa) e previstos: PRD 003. Relatórios: PRD 008. Contexto técnico: `docs/trd.md`.

## 2. Solução Proposta

### Visão de produto

- **Orçamento mensal** por grupo ou categoria, para despesas e receitas.
- Montado em poucos passos: **copiar o mês anterior**, partir da **média do histórico** (que vem da migração) ou repetir um valor para o ano todo, com exceções por mês (IPTU, matrícula, viagem).
- Acompanhamento do mês com três números por categoria: **planejado**, **realizado** e **previsto** (parcelas, contas fixas e faturas a pagar ainda não realizadas).
- **Avisos** quando uma categoria se aproxima ou passa do planejado.
- Visão do **ano**: planejado x realizado mês a mês.

### Decisões de produto

1. O realizado segue o mês de referência do PRD 003: compra no cartão conta no mês do pagamento da fatura. Motivo: regime de caixa decidido pelo casal.
2. Transferências, ajuste de saldo e valorização de investimentos ficam fora do orçamento. Motivo: não são despesa nem receita (PRDs 002, 003, 006).
3. Contas dos filhos e contas administradas ficam fora; a semanada e as compras nos cartões adicionais entram (são despesas da família). Motivo: decisões do brainstorm (rodadas 4 e 5) e PRD 002.
4. Um orçamento só para a família, feito pelo casal. Motivo: todas as despesas são comuns.

### Fora do escopo

- Orçamento por pessoa ou por filho; o limite do cartão adicional do filho é tratado no PRD 009.
- Sobra de um mês acumulando para o seguinte (orçamento "envelope") *(premissa — confirme ou corrija)*.
- Metas de poupança ou de patrimônio *(premissa — confirme ou corrija; pode virar PRD futuro)*.
- Sugestão automática de cortes *(premissa — confirme ou corrija)*.

## 3. Funcionalidades

### US01: Montar o orçamento do mês

Como administrador, quero definir quanto planejo gastar e receber em cada grupo ou categoria no mês, para ter uma referência para acompanhar.

**Rules:**
- O valor planejado pode ser definido no **grupo** (ex.: Alimentação) ou na **categoria** (ex.: Supermercado).
- Se o grupo tem valor próprio e também categorias com valor, vale o do grupo como teto, e as categorias detalham; a diferença aparece como "resto do grupo" *(premissa — confirme ou corrija; alternativa: o grupo é sempre a soma das categorias)*.
- Receitas também podem ser planejadas (ex.: Lucros e dividendos, Rendimentos); o app mostra o **saldo planejado** do mês (receitas − despesas).
- Categorias sem valor planejado aparecem como **sem orçamento**, com o realizado à vista.
- Só administradores veem e editam o orçamento (PRD 001).

**Edge cases:**
- Categoria desativada ou juntada a outra (PRD 003, US06) → o planejado passa para a categoria que fica, com aviso.
- Valor planejado zero → diferente de "sem orçamento": qualquer gasto aparece como estouro.
- Alterar o planejado de um mês passado → permitido; o histórico registra quem mudou (PRD 001).

### US02: Criar o orçamento a partir do histórico ou do mês anterior

Como administrador, quero montar o orçamento rapidamente a partir do que já gastei, para não digitar tudo do zero todo mês.

**Rules:**
- Opções ao criar o mês: **copiar o mês anterior**; **média dos últimos 3, 6 ou 12 meses** por categoria (usa o histórico migrado, PRD 005); **em branco**.
- **Orçamento do ano**: aplicar um valor a todos os meses do ano e marcar exceções por mês (ex.: IPTU em fevereiro, matrícula em janeiro).
- O mês seguinte é criado automaticamente copiando o anterior, se o casal não fizer nada *(premissa — confirme ou corrija)*.
- Valores sugeridos pela média são arredondados (ex.: para dezenas) *(premissa — confirme ou corrija)*.

**Edge cases:**
- Média de categoria com gasto raro (ex.: IPTU uma vez por ano) → a média mensal confunde; o app destaca categorias com gasto concentrado em poucos meses e sugere a exceção por mês.
- Histórico vazio (antes da migração) → só "copiar" e "em branco".

### US03: Acompanhar planejado, realizado e previsto

Como administrador, quero ver, a qualquer momento do mês, quanto já gastei, quanto ainda vem e quanto planejei em cada categoria, para corrigir o rumo antes de o mês fechar.

**Rules:**
- Por grupo e categoria: **planejado**, **realizado** (lançamentos realizados com mês de referência no mês), **previsto** (previstos do PRD 003 ainda não realizados: parcelas, contas fixas, faturas a pagar, lançamentos futuros) e **disponível** (planejado − realizado − previsto).
- Percentual consumido e destaque visual: dentro do planejado, perto (≥ 80 %) e estourado (> 100 %) *(premissa — confirme ou corrija os limites)*.
- Totais do mês: despesas, receitas e saldo, planejado x realizado.
- Tocar numa categoria mostra os lançamentos que formam o realizado e o previsto.
- Compras no cartão de faturas ainda não pagas aparecem no **previsto** do mês do vencimento, e passam a **realizado** quando a fatura é paga.

**Edge cases:**
- Estorno ou devolução → reduz o realizado da categoria no mês de referência dele (PRD 003, US09).
- Lançamento dividido em várias categorias (PRD 003, US08) → cada parte no orçamento da sua categoria.
- Lançamento em conta em dólar (exterior) → fora do orçamento mensal *(premissa — confirme ou corrija; o exterior é controlado pelo saldo mensal, PRD 002)*.
- Lançamentos em "A classificar" → aparecem numa linha própria, para o casal categorizar.

### US04: Avisar quando o orçamento aperta

Como administrador, quero ser avisado quando uma categoria chega perto ou passa do planejado, para agir no meio do mês.

**Rules:**
- Aviso no app ao chegar a **80 %** e ao passar de **100 %** do planejado da categoria ou do grupo, considerando realizado + previsto *(premissa — confirme ou corrija)*.
- Um aviso por limite por categoria no mês (sem repetir a cada lançamento).
- O casal pode desligar os avisos por categoria.
- Avisos aparecem no próximo uso do app; como a sincronização é em casa, um gasto lançado pelo outro só gera aviso depois da sincronização (PRD 011).
- Avisos de orçamento só para administradores.

**Edge cases:**
- Lançamento previsto grande entra (ex.: parcela anual) e estoura a categoria → aviso como qualquer outro.
- Categoria sem orçamento → sem aviso.

### US05: Ver o orçamento do ano

Como administrador, quero ver o planejado e o realizado de cada mês do ano, para entender onde o ano está desviando.

**Rules:**
- Tabela do ano: categorias nas linhas, meses nas colunas, planejado e realizado; total do ano por categoria e por mês.
- Meses futuros mostram planejado e previsto.
- Gráfico de planejado x realizado acumulado no ano.
- Exportável para planilha *(premissa — confirme ou corrija; a família vem de uma planilha)*.

**Edge cases:**
- Ano com meses sem orçamento definido → aparecem só com o realizado.
- Categoria criada no meio do ano → meses anteriores vazios.

## 4. Fluxo de Negócio

```
Início do mês
   │
   ▼
Orçamento do mês existe? ─não─▶ copiar o anterior / média do histórico / em branco
   │ sim
   ▼
Lançamento realizado (mês de referência = mês)  ──▶ realizado da categoria
Previsto do mês (parcelas, fixas, faturas)      ──▶ previsto da categoria
   │
   ▼
realizado + previsto ≥ 80 % do planejado? ─sim─▶ aviso
   │                         > 100 %?      ─sim─▶ aviso de estouro
   ▼
Fim do mês: planejado x realizado na visão do ano
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Planejado x realizado x previsto por categoria e grupo, no mês | pedido do casal no brainstorm | orçar Supermercado em 2.000, lançar 1.200 e uma conta fixa prevista de 300: realizado 1.200, previsto 300, disponível 500 |
| Compra no cartão entra no realizado do mês do pagamento da fatura | regime de caixa | compra em 25/03, fatura paga em 10/04: realizado de abril |
| Transferências, ajuste de saldo e valorização de investimentos não entram | não distorcer o orçamento | registrar cada um: orçamento do mês inalterado |
| Semanada e compras dos cartões adicionais entram; contas dos filhos e administradas não | decisões do brainstorm | lançar em cada uma: só as primeiras mudam o orçamento |
| Orçamento do mês criado a partir da média de 6 meses em um passo | montar rápido | criar mês com média: todas as categorias com histórico preenchidas |
| Aviso ao chegar a 80 % e ao passar de 100 % | agir no meio do mês | lançar até 80 % e depois 101 %: um aviso em cada ponto, sem repetição |
| Visão do ano com planejado e realizado de cada mês | entender o desvio do ano | abrir o ano com 3 meses fechados: valores batem com os meses |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Meses com orçamento definido | 0 (planilha sem orçamento) | todos | a partir do 2º mês de uso | 9 de 12 no primeiro ano | casal |
| Categorias estouradas no fechamento do mês | A levantar (1º mês de uso) | queda em relação ao 1º trimestre | 6 meses de uso | estável | casal |

## 6. Milestones

### Milestone 1: Orçamento do mês acompanhado

**Por que é um marco:** pela primeira vez o casal compara o que planejou com o que gastou, no meio do mês e não só no fim.

**Funcionalidades:** US01, US02, US03

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Planejado x realizado x previsto por categoria e grupo, no mês
- [ ] Compra no cartão entra no realizado do mês do pagamento da fatura
- [ ] Transferências, ajuste de saldo e valorização de investimentos não entram
- [ ] Semanada e compras dos cartões adicionais entram; contas dos filhos e administradas não
- [ ] Orçamento do mês criado a partir da média de 6 meses em um passo

**Aprovador:** casal

### Milestone 2: Avisos e visão do ano

**Por que é um marco:** o orçamento passa a avisar antes de estourar e mostra o ano inteiro.

**Funcionalidades:** US04, US05

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Aviso ao chegar a 80 % e ao passar de 100 %
- [ ] Visão do ano com planejado e realizado de cada mês

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Regime de caixa atrasa o estouro (compras de março só "aparecem" em abril) | Médio | o previsto mostra as faturas a pagar no mês do vencimento (US03) | Pendente |
| Montar o orçamento dá trabalho e é abandonado | Médio | média do histórico, cópia automática do mês anterior (US02) | Pendente |
| Avisos atrasados porque o outro lançou fora de casa | Baixo | aceito: sincronização em casa (decisão do brainstorm) | Mitigado |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 003: categorias, mês de referência, previstos | Interna | rascunho (1ª revisão aprovada) | todos |
| PRD 005: histórico para a média | Interna | rascunho (aguardando 1ª revisão) | US02 (média) |
| `docs/categorias-iniciais.md` ajustada | Interna | proposta | nenhum (orçamento usa as categorias que existirem) |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — pedido de orçamento planejado x realizado; regime de caixa.
- `docs/prds/003-lancamentos-e-categorias.md` — categorias, mês de referência, previstos, estornos.
- `docs/categorias-iniciais.md` — grupos e categorias.

## 9. Registro de Decisões

- **2026-11-03:** Orçamento mensal por grupo ou categoria, com planejado, realizado e previsto. Motivo: pedido do casal (rodada 4).
- **2026-11-03:** Realizado pelo mês de referência do PRD 003 (regime de caixa no cartão). Motivo: decisão do casal.
- **2026-11-03:** `depends_on: ["001", "003"]`. Critério: só administradores (001); categorias, mês de referência e previstos (003). A média usa o histórico do PRD 005, mas o orçamento funciona sem ele.
