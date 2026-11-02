---
prd_number: "006"
status: rascunho
revisao: "1ª revisão aprovada pelo casal em 2026-11-03; falta a revisão final"
priority: alta
created: 2026-11-03
issue: ""
depends_on: ["001", "002", "003"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/002-contas-e-cartoes.md
  - docs/prds/003-lancamentos-e-categorias.md
---

# PRD 006: Investimentos e previdência

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: a família tem investimentos no Banco do Brasil, no Santander (inclusive fundos) e em duas contas da XP, e previdência no BBPrev. No Minhas Economias cada conta de investimento era uma conta com saldo, e os rendimentos entravam como a categoria "Juros de Investimentos". Não havia visão por aplicação (CDB, fundo, ação, Tesouro) nem rendimento de cada uma.
- **Problema**: o casal quer acompanhar **cada aplicação** com aportes, resgates e rendimento (nível "b" escolhido no brainstorm), sem cotação automática, e somar tudo no patrimônio. Sem isso, não se sabe quanto cada aplicação rendeu nem o valor de cada uma em 31/12, que o IR pede em bens e direitos.

> Contas de investimento e previdência (cadastro, dono, tipo de plano): PRD 002. Patrimônio: PRD 008. IR: PRD 010. Importação de extratos da corretora: PRD 004.

## 2. Solução Proposta

### Visão de produto

- Dentro de cada conta de investimento ficam as **aplicações** (CDB, LCI/LCA, Tesouro, fundo, ação, outros) e o **saldo disponível** (dinheiro parado na corretora ou banco).
- Cada aplicação tem **aportes** e **resgates** (de e para o saldo disponível) e um **valor atual**, informado à mão ou vindo da importação do extrato.
- O **rendimento** de um período é a variação do valor atual descontados aportes e resgates.
- A **previdência** é acompanhada por plano: contribuições, saldo do extrato e rendimento, com o tipo do plano (PGBL ou VGBL).
- Uma visão da **carteira**: total, por conta, por tipo de aplicação e evolução mês a mês.

### Decisões de produto

1. Sem cotação automática: o valor atual é digitado ou importado. Motivo: decisão do casal (rodada 2, opção b); nenhuma integração externa.
2. Rendimento = variação do valor atual descontados aportes e resgates. Motivo: dedução validada na consolidação do brainstorm.
3. A valorização dentro da aplicação **não é receita do mês**; aparece no relatório de investimentos e no patrimônio. Já o rendimento **pago em dinheiro numa conta** (juros creditados na conta corrente, dividendos pagos) é receita, categoria Rendimentos. Motivo: senão o orçamento do mês oscila com o mercado.
4. Aplicar e resgatar entre a conta corrente e a conta de investimento é transferência (PRD 002, PRD 003).

### Fora do escopo

- Cotação automática de ações, fundos ou Tesouro (decidido no brainstorm).
- Preço médio, ganho de capital, imposto sobre ações e DARF (IR de renda variável fora do escopo, decidido na rodada 6).
- Recomendação de investimento, comparação com índices (CDI, IPCA, Ibovespa).
- Detalhar aplicações nas contas do exterior: só o saldo total em dólar (PRD 002).
- Simulação de aposentadoria com a previdência.

## 3. Funcionalidades

### US01: Cadastrar uma aplicação

Como administrador, quero cadastrar cada aplicação dentro da conta de investimento, para acompanhá-la separadamente.

**Rules:**
- Dados: conta de investimento, **tipo** (CDB, LCI/LCA, Tesouro, fundo, ação, previdência, outro), **nome** (ex.: "CDB Banco X 2028"), data de início.
- Opcionais: emissor ou administrador e **CNPJ** (usado em bens e direitos, PRD 010), indexador e taxa contratada (só informativos), **vencimento**, quantidade (ações e fundos).
- Toda conta de investimento tem também o **saldo disponível**, que não é aplicação.
- O dono da aplicação é o dono da conta (família, filho ou administrada, PRD 002).

**Edge cases:**
- Aplicação comprada antes do início do app ou da migração → cadastrada com o valor naquela data como aporte inicial.
- Mesmo título comprado em datas diferentes → pode ser uma aplicação só (com vários aportes) ou várias; o casal escolhe.
- Aplicação transferida entre corretoras (portabilidade) → resgate numa conta e aporte na outra, ligados.

### US02: Registrar aportes e resgates

Como administrador, quero registrar quando aplico e resgato, para que o rendimento seja calculado certo.

**Rules:**
- **Aporte**: sai do saldo disponível da conta de investimento e entra na aplicação. **Resgate**: sai da aplicação e volta ao saldo disponível.
- Dinheiro entre a conta corrente e o saldo disponível da conta de investimento é **transferência** (PRD 003, US02).
- Atalho: aplicar direto da conta corrente numa aplicação registra as duas coisas de uma vez.
- Resgate tem valor bruto e, opcionalmente, imposto e taxas retidos; o líquido volta ao disponível.
- Aportes e resgates podem vir da importação do extrato da corretora (PRD 004).

**Edge cases:**
- Resgate maior que o valor atual conhecido → aceito; o app pede atualizar o valor atual (provavelmente estava desatualizado).
- Resgate total → aplicação encerrada, histórico mantido.
- Vencimento de CDB ou Tesouro → o app lembra de registrar o resgate na data (US07).

### US03: Atualizar o valor atual

Como administrador, quero informar o valor atual de cada aplicação, para ver quanto vale e quanto rendeu.

**Rules:**
- Valor atual numa data, por aplicação, digitado ou importado (PRD 004).
- A atualização de referência é a do **fechamento do mês** (último dia útil), alinhada com o patrimônio (PRD 008).
- Tela de atualização em lote: todas as aplicações de uma conta numa tela, com o último valor ao lado, para copiar do extrato.
- O saldo da conta de investimento = saldo disponível + soma dos valores atuais das aplicações.
- A atualização conta como conferência da conta com o extrato (PRD 002, US08) quando o total bate.

**Edge cases:**
- Mês sem atualização → o app usa o último valor e mostra a data dele.
- Valor atual menor que o anterior sem resgate (ação ou fundo caiu) → normal: rendimento negativo.
- Atualização com data anterior a outra já feita → aceita; o rendimento dos meses afetados é recalculado.

### US04: Ver o rendimento

Como administrador, quero ver quanto cada aplicação rendeu no mês, no ano e desde o início, para avaliar meus investimentos.

**Rules:**
- Rendimento no período = valor final − valor inicial − aportes + resgates.
- Mostrado em reais e em **percentual simples** sobre o valor inicial mais os aportes do período.
- Por aplicação, por conta, por tipo e total da família.
- O rendimento da valorização **não entra** nas receitas do mês (decisão 3); juros e dividendos pagos em conta entram como receita.

**Edge cases:**
- Aplicação aberta no meio do período → valor inicial zero; conta só o aporte.
- Período sem nenhuma atualização de valor → rendimento "não calculado", não zero.

### US05: Acompanhar a previdência

Como administrador, quero acompanhar cada plano de previdência, para saber quanto contribuí, quanto vale e quanto é dedutível no IR.

**Rules:**
- Cada plano (PRD 002: tipo PGBL ou VGBL, participante) tem **contribuições**, **valor atual** pelo extrato e **rendimento**, como uma aplicação.
- Contribuição vinda da conta corrente é transferência; se o plano é PGBL, entra também como **despesa dedutível de previdência** no relatório do IR (PRD 010).
- Total contribuído no ano por participante e por tipo de plano.
- Atualização pelo extrato mensal do BBPrev (digitado ou importado, PRD 004).

**Edge cases:**
- Contribuição descontada em folha (sem passar pela conta) → registrada direto no plano (PRD 002).
- Portabilidade entre planos → resgate de um e aporte no outro, sem ser contribuição nova.
- Resgate de previdência → vai para o saldo disponível ou para a conta corrente; imposto retido registrado como no resgate de aplicação.

### US06: Ver a carteira

Como administrador, quero ver todos os investimentos juntos, para saber quanto a família tem investido e como evolui.

**Rules:**
- Total investido da família (contas de investimento e previdência cujo dono é a família), com a data da última atualização.
- Agrupado por conta, por instituição e por tipo de aplicação, com o percentual de cada parte.
- Evolução mês a mês do valor total e do rendimento (gráfico).
- Contas de investimento dos filhos e administradas aparecem separadas, fora do total da família (PRD 002).
- O dependente vê só os investimentos próprios, se tiver (PRD 001).

**Edge cases:**
- Aplicações com valor desatualizado há mais de 2 meses → destacadas.
- Conta de investimento sem aplicações → aparece só com o saldo disponível.

### US07: Vencimentos e lembretes de atualização

Como administrador, quero ser lembrado de vencimentos e de atualizar os valores no fim do mês, para não deixar a carteira desatualizada.

**Rules:**
- Aplicações com vencimento aparecem numa lista dos **próximos vencimentos** (90 dias).
- No início de cada mês, o app lembra de atualizar os valores do fechamento do mês anterior, com a lista do que falta.
- Vencimento passado sem resgate registrado → destacado.

**Edge cases:**
- Aplicação sem vencimento (ação, fundo) → só entra no lembrete de atualização.
- Lembrete ignorado → continua na lista até a atualização.

## 4. Fluxo de Negócio

```
Conta corrente ──transferência──▶ Saldo disponível (conta de investimento)
                                        │  aporte ▲ resgate
                                        ▼        │
                                    Aplicação (CDB, fundo, ação, Tesouro, previdência)
                                        │
                              valor atual no fechamento do mês
                                        │
                                        ▼
               rendimento = valor final − valor inicial − aportes + resgates
                                        │
                     ┌──────────────────┴──────────────────┐
                     ▼                                     ▼
       relatório de investimentos e patrimônio   juros/dividendos pagos em conta
       (não é receita do mês)                    → receita do mês (Rendimentos)
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Todas as aplicações atuais da família podem ser cadastradas nas contas de BB, Santander, XP e BBPrev | substituir o controle por conta do Minhas Economias | cadastrar a carteira real; nada fica de fora |
| Rendimento = valor final − inicial − aportes + resgates | dedução validada no brainstorm | aplicação de 1.000, aporte de 500, valor final 1.530: rendimento 30 |
| Valorização da aplicação não aparece nas receitas do mês | o orçamento do mês não oscila com o mercado | atualizar o valor de um fundo: receitas do mês não mudam |
| Aplicação e resgate entre corrente e investimento não alteram despesas e receitas | não contar duas vezes | aplicar 1.000: totais do mês inalterados |
| Contribuição PGBL aparece como dedutível no ano, por participante | IR | contribuir em PGBL e VGBL: só a PGBL soma no total dedutível |
| Atualizar todas as aplicações de uma conta numa tela só | fechamento do mês rápido | atualizar 10 aplicações sem trocar de tela |
| Carteira mostra total, por conta e por tipo, com a data da última atualização | visão geral dos investimentos | abrir a carteira com valores de datas diferentes: cada um com a sua data |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Aplicações com valor atualizado no fechamento do mês | 0 (não há controle por aplicação) | todas | todo mês, a partir do 2º mês de uso | as de maior valor | casal |
| Diferença entre o total da carteira no app e a soma dos extratos | A levantar | 0 | mensal | explicada | casal |

## 6. Milestones

### Milestone 1: Carteira de investimentos no app

**Por que é um marco:** o casal vê cada aplicação da família, quanto vale e onde está, num só lugar.

**Funcionalidades:** US01, US02, US03, US06

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Todas as aplicações atuais da família podem ser cadastradas nas contas de BB, Santander, XP e BBPrev
- [ ] Aplicação e resgate entre corrente e investimento não alteram despesas e receitas
- [ ] Atualizar todas as aplicações de uma conta numa tela só
- [ ] Carteira mostra total, por conta e por tipo, com a data da última atualização

**Aprovador:** casal

### Milestone 2: Rendimento e previdência

**Por que é um marco:** o casal passa a saber quanto cada investimento rendeu e quanto contribuiu para a previdência no ano, o que hoje não sabe.

**Funcionalidades:** US04, US05, US07

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Rendimento = valor final − inicial − aportes + resgates
- [ ] Valorização da aplicação não aparece nas receitas do mês
- [ ] Contribuição PGBL aparece como dedutível no ano, por participante

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Atualizar valores à mão todo mês dá trabalho e é abandonado | Alto | atualização em lote; importação do extrato da corretora (PRD 004); lembrete mensal | Pendente |
| Extratos da corretora não trazem valor por aplicação no formato importável | Médio | digitação em lote como alternativa; ver amostras no PRD 004 | Pendente |
| Rendimento percentual simples enganar em períodos com muitos aportes | Baixo | mostrar sempre também o valor em reais | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 002: contas de investimento e previdência, tipo de plano | Interna | rascunho (1ª revisão aprovada) | todos |
| PRD 003: transferências | Interna | rascunho (aguardando 1ª revisão) | Milestone 1 |
| PRD 004: importação de extratos da corretora e do BBPrev | Interna | a escrever | nenhum (há digitação) |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — nível (b) dos investimentos, sem cotação; dedução do rendimento.
- `docs/prds/002-contas-e-cartoes.md` — contas de investimento, previdência (PGBL/VGBL, participante), XP só investimento.
- `docs/prds/003-lancamentos-e-categorias.md` — transferências.

## 9. Registro de Decisões

- **2026-11-03:** Investimentos por aplicação, sem cotação automática. Motivo: decisão do casal (rodada 2, opção b).
- **2026-11-03:** Rendimento pela variação do valor descontados aportes e resgates. Motivo: dedução validada na consolidação.
- **2026-11-03:** `depends_on: ["001", "002", "003"]`. Critério: permissões (001), contas de investimento e previdência (002), transferências (003). A importação (004) é alternativa à digitação, não pré-requisito.
- **2026-11-03:** 1ª revisão do casal: as 14 premissas foram aceitas como estão (inclusive: valorização não é receita do mês; rendimento em percentual simples; atualização de referência no fechamento do mês). Marcas removidas; status mantido em `rascunho` até a revisão final.
