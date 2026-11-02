---
prd_number: "008"
status: rascunho
revisao: "1ª revisão aprovada pelo casal em 2026-11-03; falta a revisão final"
priority: alta
created: 2026-11-03
issue: ""
depends_on: ["001", "002", "003", "006"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/002-contas-e-cartoes.md
  - docs/prds/003-lancamentos-e-categorias.md
  - docs/prds/006-investimentos-e-previdencia.md
  - docs/prds/007-orcamento.md
---

# PRD 008: Relatórios e patrimônio

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: a planilha da família não tem gráficos, relatórios nem análises; essa foi a queixa que motivou o projeto, junto com a privacidade. O Minhas Economias tinha relatórios, mas com o dinheiro da PJ e dos parentes misturado ao da família.
- **Problema**: o casal pediu quatro análises (rodada 4 do brainstorm): **gastos por categoria no mês**, **evolução ao longo dos meses**, **orçamento planejado x realizado** e **patrimônio total**. O orçamento está no PRD 007; este PRD cobre as outras três, com o histórico migrado desde 2017 (PRD 005).

> Mês de referência e regime de caixa: PRD 003. Contas, donos e câmbio: PRD 002. Valor dos investimentos: PRD 006. Contexto técnico: `docs/trd.md`.

## 2. Solução Proposta

### Visão de produto

- **Gastos por categoria** no mês, do grupo até o lançamento, em gráfico e tabela.
- **Evolução mês a mês** de despesas e receitas, por grupo ou categoria, com comparação com o mesmo mês do ano anterior e com a média.
- **Receitas, despesas e saldo** de cada mês, com a **taxa de poupança**.
- **Patrimônio da família** no fechamento de cada mês: contas, investimentos, previdência, dinheiro e exterior convertido pelo **câmbio digitado** pelo casal, descontado o que se deve nos cartões; e a evolução dele ao longo dos anos.
- **Quanto se gasta com cada filho** (cartão adicional e semanada), e relatórios separados para as contas dos filhos e as administradas.

### Decisões de produto

1. Relatórios de despesas e receitas usam o mês de referência do PRD 003 (compra no cartão no mês do pagamento da fatura). Motivo: regime de caixa decidido pelo casal.
2. Relatórios mostram só o **realizado**; previstos aparecem no orçamento (PRD 007) e na visão de comprometimento (PRD 003). Motivo: análise do que aconteceu.
3. Patrimônio da família exclui contas dos filhos e contas administradas. Motivo: decisões do brainstorm (rodada 5) e PRD 002 (US12).
4. Câmbio do dólar digitado pelo casal no fechamento do mês. Motivo: decisão do casal (rodada 5); sem integração externa.
5. Transferências, ajuste de saldo e valorização de investimentos ficam fora de despesas e receitas. Motivo: PRDs 002, 003 e 006.

### Fora do escopo

- Projeção do patrimônio ou do saldo para meses futuros.
- Relatórios configuráveis pelo usuário (montar o próprio relatório).
- Comparação com índices de inflação ou com outras famílias.
- Relatórios do IR: PRD 010.
- Relatórios da PJ (fora da 1ª versão).

## 3. Funcionalidades

### US01: Ver os gastos por categoria no mês

Como administrador, quero ver quanto gastei em cada grupo e categoria no mês, para saber para onde vai o dinheiro.

**Rules:**
- Gráfico e tabela dos grupos de despesa, com valor e percentual do total do mês; tocar num grupo abre as categorias, e numa categoria, os lançamentos.
- Filtros: mês (ou período), contas e cartões, cartão adicional (quem comprou).
- Também a visão de receitas por grupo e categoria.
- Lançamentos divididos (PRD 003, US08) entram em cada categoria pela sua parte; estornos reduzem a categoria.
- Lançamentos em "A classificar" aparecem como um grupo próprio.

**Edge cases:**
- Mês sem lançamentos → mensagem de mês vazio, não gráfico zerado.
- Categoria com valor líquido negativo no mês (estorno maior que o gasto) → mostrada à parte, sem distorcer o gráfico.
- Mês ainda em curso → indicado como parcial (faturas não pagas ainda não entraram; regime de caixa).

### US02: Ver a evolução ao longo dos meses

Como administrador, quero ver como despesas e receitas mudaram mês a mês, para identificar tendências.

**Rules:**
- Gráfico mês a mês do total de despesas e de receitas, e de qualquer grupo ou categoria escolhida.
- Período escolhido livremente, de 2017 (histórico migrado) até o mês atual; padrão: últimos 12 meses.
- Comparações: com o **mesmo mês do ano anterior** e com a **média dos últimos 12 meses**, em valor e percentual.
- Visão anual: totais por ano, para comparar anos inteiros.
- Os valores são nominais (sem correção pela inflação).

**Edge cases:**
- Categoria criada ou juntada no meio do período → o histórico segue a categoria atual (lançamentos movidos na junção, PRD 003).
- Meses do histórico com diferença de fatura não resolvida (PRD 005, US05) → incluídos, com indicação discreta.
- Mês atual parcial → destacado para não parecer queda.

### US03: Ver receitas, despesas e saldo do mês

Como administrador, quero ver quanto entrou, quanto saiu e quanto sobrou em cada mês, para saber se estamos poupando.

**Rules:**
- Por mês: receitas, despesas, **saldo** (receitas − despesas) e **taxa de poupança** (saldo ÷ receitas).
- Gráfico de barras de receitas e despesas com a linha do saldo, mês a mês.
- Valorização de investimentos não entra (é rendimento, PRD 006); aparece separada como "rendimento dos investimentos no mês" para referência.

**Edge cases:**
- Mês com receita zero → taxa de poupança "não se aplica".
- Mês com saldo negativo → destacado.

### US04: Informar o câmbio do fechamento do mês

Como administrador, quero digitar a cotação do dólar no fechamento de cada mês, para converter as contas do exterior no patrimônio.

**Rules:**
- Uma cotação de dólar para real por mês, digitada pelo casal.
- O app pede a cotação junto com o lembrete de atualização dos investimentos no início do mês (PRD 006, US07).
- A cotação pode ser corrigida depois; o patrimônio daquele mês é recalculado.
- A cotação vale para o patrimônio. A conversão para o IR segue regra própria da Receita (PRD 010).

**Edge cases:**
- Mês sem cotação → usa a do último mês informado e marca o patrimônio daquele mês como "câmbio estimado".
- Meses do histórico migrado (desde 2017) sem cotação → o casal pode digitar as cotações antigas, uma por mês; enquanto isso, vale a regra anterior.

### US05: Ver o patrimônio da família no fechamento do mês

Como administrador, quero ver quanto a família tem no fechamento de cada mês, para acompanhar a situação financeira de forma completa.

**Rules:**
- Patrimônio = contas correntes + dinheiro em espécie + investimentos (valor atual, PRD 006) + previdência + exterior (dólar × câmbio do mês) − **saldo devedor dos cartões** (faturas fechadas não pagas e compras da fatura aberta).
- Só contas cujo dono é a família; contas dos filhos e administradas ficam de fora (decisão 3).
- Composição: por tipo (corrente, investimento, previdência, exterior, dinheiro), por instituição e por conta, em valor e percentual.
- Cada parcela mostra a data da última atualização (saldo conferido, valor de investimento, extrato do exterior).
- Patrimônio "de hoje" além do fechamento dos meses.
- Imóveis, veículos e outros bens fora das contas não entram.

**Edge cases:**
- Investimento sem atualização no mês → usa o último valor, com a data.
- Conta no exterior sem extrato no mês → usa o último saldo, com a data (PRD 002, US03).
- Parcelas futuras de compras já feitas → não descontadas do patrimônio (são compromissos futuros, PRD 003).

### US06: Ver a evolução do patrimônio

Como administrador, quero ver o patrimônio mês a mês ao longo dos anos, para saber se a família está crescendo financeiramente.

**Rules:**
- Gráfico do patrimônio no fechamento de cada mês, total e empilhado por tipo.
- Variação no mês e no ano, separada em **saldo do mês** (receitas − despesas, US03), **rendimento dos investimentos** (PRD 006) e **variação cambial** do exterior.
- Período desde o início dos dados.

**Edge cases:**
- Meses antigos sem valores de investimento (o Minhas Economias tinha só saldo de conta) → usa os saldos migrados das contas de investimento.
- Conta que passa da família para um filho → sai do patrimônio da família em todo o histórico (PRD 002, US04).

### US07: Ver quanto se gasta com cada filho

Como administrador, quero ver os gastos de cada filho (cartão adicional e semanada), para conversar com eles com números na mão.

**Rules:**
- Por filho e por mês: compras no cartão adicional dele (por categoria), semanada e outras despesas lançadas em nome dele.
- Evolução mês a mês por filho.
- Contas próprias dos filhos não entram (o que o filho faz com o dinheiro dele é dele).

**Edge cases:**
- Compras do histórico migrado não identificam o adicional (PRD 005) → os meses antes da migração mostram só a semanada.
- Filho sem cartão adicional → só semanada e despesas em nome dele.

### US08: Ver os relatórios das contas dos filhos e das administradas

Como administrador, quero ver relatórios das contas dos filhos e das contas administradas separados dos da família, para acompanhar sem misturar.

**Rules:**
- Para cada filho e para cada conta administrada: saldo, receitas e despesas por categoria no mês, e evolução — os mesmos relatórios das US01 a US03, filtrados para aquela conta.
- Nunca somados aos da família.
- O dependente vê os relatórios das próprias contas e do próprio cartão adicional (gastos por categoria e evolução) e nada mais (PRD 001).

**Edge cases:**
- Conta administrada encerrada → relatórios do período em que existiu.
- Dependente sem lançamentos → relatórios vazios com mensagem.

### US09: Exportar relatórios

Como administrador, quero exportar um relatório, para guardar ou abrir numa planilha.

**Rules:**
- Exportação em planilha (dados da tabela) e em PDF (gráfico e tabela).
- O arquivo é gerado no aparelho e salvo onde o usuário escolher; o app não envia nada para fora.
- Também exportação dos lançamentos filtrados (período, conta, categoria).

**Edge cases:**
- Relatório com 9 anos de dados → exportação completa, sem cortar linhas.
- Exportação no celular do dependente → só o que ele pode ver.

## 4. Fluxo de Negócio

```
Lançamentos realizados (mês de referência)
   │
   ├──▶ por categoria no mês (US01)
   ├──▶ evolução mês a mês (US02)
   └──▶ receitas − despesas = saldo do mês (US03)
                                   │
Fechamento do mês                  ▼
   saldos das contas ─┐     Patrimônio da família (US05)
   valor dos investimentos ─┤  = contas + dinheiro + investimentos + previdência
   saldo do exterior × câmbio ─┤    + exterior × câmbio − devido nos cartões
   devido nos cartões ─┘     (sem filhos, sem administradas)
                                   │
                                   ▼
             evolução: saldo do mês + rendimento + câmbio (US06)
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Gastos do mês por grupo, categoria e lançamento, com percentual | pedido do casal | abrir março: grupos somam o total de despesas; tocar desce até os lançamentos |
| Evolução mês a mês de qualquer grupo ou categoria, desde 2017 | pedido do casal; histórico migrado | escolher Alimentação de 2017 a hoje: um ponto por mês |
| Comparação com o mesmo mês do ano anterior e com a média de 12 meses | identificar tendências | ver março: mostra março do ano anterior e a média, em valor e % |
| Saldo e taxa de poupança do mês | saber se a família poupa | receitas 10.000, despesas 8.000: saldo 2.000, taxa 20 % |
| Patrimônio converte o exterior pelo câmbio digitado do mês | decisão do casal | saldo de US$ 1.000, câmbio 5,00: R$ 5.000 no patrimônio |
| Patrimônio exclui contas dos filhos e administradas e desconta o devido nos cartões | o que é da família, líquido | movimentar conta do filho e administrada: patrimônio não muda; fatura fechada de 3.000: patrimônio −3.000 |
| Cada parte do patrimônio mostra a data da última atualização | saber o que está desatualizado | investimento atualizado há 2 meses aparece com a data |
| Relatórios com 9 anos de dados abrem no celular em até 3 s | se demorar, o casal não usa | cronometrar a evolução de 2017 a hoje |
| Dependente vê só os relatórios das próprias contas e cartão | privacidade da família | abrir relatórios no celular do filho: nada da família |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Relatórios consultados pelo casal | 0 (planilha sem relatórios) | ao menos uma vez por mês | a partir do 2º mês de uso | a cada fechamento de trimestre | casal |
| Meses com patrimônio completo (todas as partes atualizadas no mês) | 0 | todos | a partir do 2º mês de uso | 9 de 12 no ano | casal |

## 6. Milestones

### Milestone 1: Para onde vai o dinheiro

**Por que é um marco:** a família ganha os gráficos e análises que a planilha nunca teve, com o histórico desde 2017.

**Funcionalidades:** US01, US02, US03, US07

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Gastos do mês por grupo, categoria e lançamento, com percentual
- [ ] Evolução mês a mês de qualquer grupo ou categoria, desde 2017
- [ ] Comparação com o mesmo mês do ano anterior e com a média de 12 meses
- [ ] Saldo e taxa de poupança do mês
- [ ] Relatórios com 9 anos de dados abrem no celular em até 3 s

**Aprovador:** casal

### Milestone 2: Patrimônio da família

**Por que é um marco:** pela primeira vez o casal vê o patrimônio total, inclusive o exterior, e como ele evolui.

**Funcionalidades:** US04, US05, US06

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Patrimônio converte o exterior pelo câmbio digitado do mês
- [ ] Patrimônio exclui contas dos filhos e administradas e desconta o devido nos cartões
- [ ] Cada parte do patrimônio mostra a data da última atualização

**Aprovador:** casal

### Milestone 3: Relatórios separados e exportação

**Por que é um marco:** filhos e contas administradas ganham relatórios próprios, e tudo pode sair do app em planilha ou PDF.

**Funcionalidades:** US08, US09

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Dependente vê só os relatórios das próprias contas e cartão

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Regime de caixa faz o mês em curso parecer menor (faturas ainda não pagas) | Médio | mês atual marcado como parcial (US01, US02) | Pendente |
| Patrimônio desatualizado porque investimentos, exterior ou câmbio não foram atualizados | Médio | data de cada parte; lembrete mensal junto com os investimentos (US04) | Pendente |
| Histórico migrado com distorções (faturas antigas, dinheiro de terceiros não reclassificado) | Médio | conferência na prévia da migração (PRD 005) e indicação nos meses afetados | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 003: mês de referência e categorias | Interna | rascunho (1ª revisão aprovada) | Milestone 1 |
| PRD 002: contas, donos, saldos | Interna | rascunho (1ª revisão; US12 aguardando) | Milestones 2 e 3 |
| PRD 006: valor dos investimentos e rendimento | Interna | rascunho (aguardando 1ª revisão) | Milestone 2 |
| PRD 005: histórico desde 2017 | Interna | rascunho (aguardando 1ª revisão) | nenhum (relatórios funcionam com o que houver) |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — as quatro análises pedidas; câmbio digitado; patrimônio sem filhos.
- `docs/prds/002-contas-e-cartoes.md` — contas, donos, exterior, contas administradas.
- `docs/prds/003-lancamentos-e-categorias.md` — mês de referência, categorias, previstos.
- `docs/prds/006-investimentos-e-previdencia.md` — valor atual e rendimento.
- `docs/prds/007-orcamento.md` — planejado x realizado (a quarta análise pedida).

## 9. Registro de Decisões

- **2026-11-03:** Relatórios pelo mês de referência (regime de caixa) e só com o realizado. Motivo: decisões do brainstorm (rodada 4).
- **2026-11-03:** Patrimônio sem contas dos filhos e administradas; exterior pelo câmbio digitado no fechamento do mês. Motivo: decisões do brainstorm (rodada 5) e PRD 002.
- **2026-11-03:** Orçamento x realizado fica no PRD 007; este PRD cobre as outras três análises pedidas. Motivo: divisão da consolidação.
- **2026-11-03:** `depends_on: ["001", "002", "003", "006"]`. Critério: visibilidade por perfil (001), contas, donos e exterior (002), mês de referência e categorias (003), valor atual e rendimento dos investimentos (006).
- **2026-11-03:** 1ª revisão do casal: as 18 premissas foram aceitas como estão (inclusive: patrimônio desconta o devido nos cartões e não as parcelas futuras; imóveis e veículos fora; variação do patrimônio separada em saldo, rendimento e câmbio; relatórios em até 3 s). Marcas removidas; status mantido em `rascunho` até a revisão final.
