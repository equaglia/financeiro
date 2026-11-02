---
prd_number: "002"
status: rascunho
revisao: "1ª revisão aprovada pelo casal em 2026-11-03; falta a revisão final"
priority: crítica
created: 2026-11-03
issue: ""
depends_on: ["001"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/001-usuarios-perfis-e-acesso.md
---

# PRD 002: Contas e cartões

## 1. Contexto

- **Produto/área**: controle financeiro da família (casal e dois filhos). Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: a família tem contas correntes e de investimento no Banco do Brasil e no Santander, duas contas na XP Investimentos, previdência no BBPrev e contas em dólar no exterior. Os cartões são Visa e Mastercard do Santander e um Visa em cada conta da XP; os filhos têm cartões adicionais com limite mensal e uma conta individual cada, onde recebem a mesada por Pix. Hoje só uma planilha simples registra receitas e despesas, sem saldos por conta nem faturas.
- **Problema**: lançamentos, importação, orçamento, relatórios, patrimônio e IR precisam saber **onde** o dinheiro está e **de quem** é cada conta. Sem o cadastro de contas e cartões, com faturas por ciclo, não há como aplicar o regime de caixa do cartão nem separar o que é dos filhos.

> Contexto técnico em `docs/trd.md`. Aplicações dentro das contas de investimento e previdência: PRD 006. Câmbio de fechamento e patrimônio: PRD 008. Limite e mesada dos filhos: PRD 009.

## 2. Solução Proposta

### Visão de produto

- Um cadastro único com todos os lugares onde a família tem dinheiro ou dívida: contas correntes, contas de investimento, previdência, contas no exterior, **dinheiro em espécie** e cartões de crédito.
- Cada conta tem **dono** (a família ou um filho) e **moeda** (real ou dólar); isso define quem vê (PRD 001) e o que entra nos relatórios da família.
- Cada cartão tem titular, **cartões adicionais ligados a cada filho** e **faturas por ciclo** (fechamento e vencimento).
- O **pagamento da fatura** é uma transferência da conta de pagamento (corrente ou investimento) para o cartão; é ele que marca o mês em que as compras daquela fatura contam (regime de caixa).
- O saldo de cada conta pode ser **conferido com o extrato**, e diferenças viram um ajuste visível.

### Decisões de produto

1. Tipos de conta fixos: corrente, investimento, previdência, exterior, dinheiro em espécie e cartão de crédito. Motivo: cobre tudo o que a família tem (o dinheiro em espécie existia no Minhas Economias); tipos livres complicariam relatórios e IR.
2. Uma moeda por conta (real ou dólar). Motivo: decidido no brainstorm; conversão só no patrimônio (PRD 008).
3. Compra no cartão conta no mês em que a fatura é paga. Motivo: decisão do casal (rodada 4 do brainstorm).
4. Pagamento da fatura é transferência, não despesa. Motivo: senão o gasto conta duas vezes.
5. Do cartão guardam-se só os **4 últimos dígitos**. Motivo: suficiente para reconhecer o cartão na fatura importada; número completo é dado sensível sem uso.

### Fora do escopo

- Empréstimos, financiamentos e consórcios como tipo de conta.
- Moedas além de real e dólar (decidido no brainstorm: hoje só dólar).
- Cotação automática do dólar (decidido no brainstorm: câmbio digitado, PRD 008).
- Consulta de saldo ou limite direto no banco (sem Open Finance).
- Programa de pontos e milhas do cartão.

## 3. Funcionalidades

### US01: Cadastrar conta corrente

Como administrador, quero cadastrar cada conta corrente da família, para registrar entradas e saídas nela.

**Rules:**
- Dados: nome (ex.: "BB conjunta"), instituição, tipo corrente, moeda real, dono, **saldo inicial** e **data do saldo inicial**.
- Dados opcionais para o IR (PRD 010): agência, número da conta e CNPJ da instituição.
- O saldo atual é o saldo inicial mais os lançamentos a partir da data do saldo inicial.
- Instituições já previstas na lista inicial: Banco do Brasil, Santander, XP Investimentos, BBPrev; o casal pode cadastrar outras.

**Edge cases:**
- Lançamento com data anterior ao saldo inicial (ex.: histórico migrado) → aceito, mas o app avisa que o saldo inicial deveria ser ajustado; a migração (PRD 005) define o saldo inicial na data mais antiga.
- Saldo inicial negativo (cheque especial) → permitido.
- Duas contas com o mesmo nome → recusado; nome deve ser único entre as contas ativas.

### US02: Cadastrar conta de investimento e de previdência

Como administrador, quero cadastrar as contas de investimento (BB, Santander, XP) e a previdência (BBPrev), para acompanhar as aplicações e somá-las no patrimônio.

**Rules:**
- Mesmos dados da conta corrente, com tipo investimento ou previdência.
- A conta de investimento agrupa aplicações (CDB, fundo, ações, Tesouro), detalhadas no PRD 006. Aqui ela existe como lugar de destino dos aportes e origem dos resgates.
- Aporte e resgate entre a conta corrente e a conta de investimento são **transferências** (não são despesa nem receita).
- Previdência: contribuições vindas da conta corrente são transferências para a conta de previdência e, ao mesmo tempo, **despesa dedutível de previdência** para o IR quando o plano for do tipo dedutível (PRD 010). A dedução depende do tipo de plano: só PGBL.
- Para previdência, guardar o **tipo de plano** (PGBL ou VGBL) e o **participante** (qual membro).

**Edge cases:**
- As contas da XP são cadastradas como **contas de investimento** (a XP oferece conta corrente, mas a família não usa esse formato). Os cartões Visa da XP são pagos pela conta de pagamento cadastrada no cartão, que pode ser a própria conta de investimento (US05).
- Contribuição de previdência descontada direto do salário (sem passar pela conta corrente) → lançada como entrada na previdência sem origem na família.

### US03: Cadastrar conta no exterior (dólar)

Como administrador, quero cadastrar as contas no exterior em dólar, para acompanhar o saldo e somá-lo ao patrimônio.

**Rules:**
- Tipo exterior, moeda **dólar**, dono, instituição e país.
- Controle simplificado (decidido no brainstorm): atualizada **uma vez por mês a partir do extrato**. Cada atualização registra o **saldo final do mês em dólar** e, opcionalmente, movimentos resumidos (entradas e saídas do mês).
- Os valores ficam em dólar; a conversão para real acontece só no patrimônio, pelo câmbio de fechamento do mês (PRD 008).
- Transferência entre conta no Brasil e conta no exterior registra os dois valores (reais que saíram e dólares que entraram).

**Edge cases:**
- Mês sem extrato lançado → o patrimônio usa o último saldo conhecido e mostra a data dele.
- Conta no exterior com aplicações (fundos, ações) → tratada como saldo total em dólar, sem detalhar aplicações.

### US04: Cadastrar as contas dos filhos

Como administrador, quero cadastrar a conta individual de cada filho como dele, para que ele a veja e ela fique fora dos relatórios da família.

**Rules:**
- O dono da conta é o filho. O casal cadastra; o filho não cria contas (PRD 001).
- A conta do filho fica **fora** dos relatórios, do orçamento e do patrimônio da família; o casal a vê numa área separada (PRD 001, US05).
- A mesada por Pix sai da conta da família como **despesa da família** e entra na conta do filho como **receita do filho** (decidido no brainstorm). Detalhes no PRD 009.
- Contas correntes e de investimento próprias que os filhos venham a ter usam o mesmo cadastro, com o filho como dono.

**Edge cases:**
- Conta muda de dono (família → filho ou filho → família) → o histórico passa junto e a visibilidade muda na próxima sincronização (PRD 001); os relatórios da família passam a excluir ou incluir **todo o histórico** da conta.
- Transferência entre a conta da família e a do filho que não é mesada (ex.: reembolso) → o casal escolhe se é despesa da família ou não, pela categoria.

### US05: Cadastrar cartão de crédito e cartões adicionais

Como administrador, quero cadastrar cada cartão com seus adicionais, para saber quem fez cada compra.

**Rules:**
- Dados do cartão: nome (ex.: "Santander Visa"), emissor, bandeira, **dia de fechamento**, **dia de vencimento**, limite total (informativo), **conta de pagamento** padrão (corrente ou investimento), 4 últimos dígitos do cartão do titular.
- Cada **cartão adicional** tem: membro da família a quem pertence (um filho ou um dos administradores), 4 últimos dígitos e, para filho, **limite mensal** (usado no PRD 009).
- Toda compra no cartão fica ligada ao **cartão que a fez** (titular ou adicional); é isso que permite ao filho ver só as dele (PRD 001).
- O cartão é sempre da família (o casal paga a fatura). As compras dos adicionais dos filhos são despesas da família (decidido no brainstorm).

**Edge cases:**
- Cartão substituído (perda, vencimento) com novos 4 últimos dígitos → o casal atualiza os dígitos; compras antigas continuam ligadas ao mesmo cartão.
- Compra importada com 4 dígitos que não batem com nenhum cartão cadastrado → fica atribuída ao titular e marcada para o casal revisar (PRD 004).
- Cartão virtual (dígitos diferentes do físico) → cadastrado como mais um conjunto de dígitos do mesmo titular ou adicional.

### US06: Acompanhar a fatura de cada ciclo

Como administrador, quero ver a fatura de cada mês do cartão, com as compras do ciclo, para saber quanto vou pagar e quando.

**Rules:**
- Cada compra cai na fatura do ciclo pela **data da compra** e pelo **dia de fechamento**. Compras parceladas geram uma parcela em cada fatura seguinte (detalhe no PRD 003).
- Situação da fatura: **aberta** (ciclo em curso), **fechada** (aguardando pagamento), **paga**, **paga parcialmente**.
- A fatura mostra: total, compras por cartão (titular e cada adicional), parcelas, estornos e créditos, encargos (anuidade, IOF, juros).
- O casal pode corrigir a data de fechamento e de vencimento de uma fatura específica (o banco às vezes muda) sem alterar o padrão do cartão.
- Total da fatura importada (PRD 004) é comparado com o total calculado; diferença fica visível.

**Edge cases:**
- Dia de fechamento ou vencimento cai em fim de semana ou feriado → vale a data informada pelo casal ou pela fatura importada; o padrão é só uma previsão.
- Estorno de compra de uma fatura anterior → entra como crédito na fatura em que aparece.
- Compra internacional → registrada em reais pelo valor da fatura; IOF como encargo separado.

### US07: Pagar a fatura

Como administrador, quero registrar o pagamento da fatura, para que as compras dela entrem nos relatórios do mês do pagamento.

**Rules:**
- O pagamento é uma **transferência** da conta de pagamento (corrente ou investimento) para o cartão; não é despesa.
- As compras, parcelas e encargos da fatura contam como despesa **no mês do pagamento** (regime de caixa, decidido no brainstorm).
- Antes do pagamento, a fatura aparece como **compromisso a pagar** no vencimento (previsão, PRD 003).
- O pagamento pode vir da importação do extrato da conta corrente (PRD 004), conciliado com a fatura.

**Edge cases:**
- Pagamento parcial (mínimo ou outro valor) → a família sempre paga o total, então é exceção: o app aceita, marca a fatura como "paga parcialmente" e avisa; as despesas contam **proporcionalmente** ao valor pago no mês do pagamento, e o saldo restante passa para a próxima fatura com os juros como encargo.
- Fatura paga em duas vezes no mesmo mês → soma; a fatura vira paga quando o total é atingido.
- Pagamento maior que a fatura → o excedente vira crédito na fatura seguinte.
- Pagamento feito em mês diferente do vencimento (atraso ou adiantamento) → vale o mês do pagamento efetivo.

### US08: Conferir o saldo com o extrato

Como administrador, quero informar o saldo que aparece no extrato do banco, para saber se faltou algum lançamento.

**Rules:**
- O casal informa o saldo do extrato numa data; o app mostra o saldo calculado na mesma data e a diferença.
- Diferença zero → conta marcada como **conferida** até aquela data.
- Diferença diferente de zero → o casal procura o lançamento faltante ou cria um **ajuste de saldo**, visível como tal e fora das categorias de despesa e receita.
- Vale para conta corrente, investimento, previdência, exterior e dinheiro em espécie (por contagem). Para investimento e previdência, a conferência é detalhada no PRD 006.

**Edge cases:**
- Lançamento alterado ou incluído com data anterior a uma conferência → a conferência deixa de valer e o app avisa.
- Importação de extrato que traz o saldo final (PRD 004) → a conferência é feita automaticamente.

### US09: Encerrar conta ou cartão

Como administrador, quero encerrar uma conta ou cartão que não uso mais, para que saia das listas sem perder o histórico.

**Rules:**
- Só pode ser encerrada conta com saldo zero, ou cartão sem fatura em aberto ou a pagar.
- Encerrada, a conta some das listas de lançamento, mas o histórico continua nos relatórios e no IR do período em que existiu.
- Encerramento pode ser desfeito.

**Edge cases:**
- Conta encerrada com saldo diferente de zero → o app pede um ajuste ou uma transferência antes.
- Lançamento importado para uma conta encerrada → o app avisa e permite reabrir.

### US10: Ver todas as contas e cartões

Como administrador, quero ver numa tela todas as contas e cartões com saldo e próxima fatura, para ter a visão geral do dinheiro da família.

**Rules:**
- Lista agrupada por tipo: correntes, investimento, previdência, exterior (em dólar), dinheiro em espécie, cartões (fatura atual e próxima a vencer), contas dos filhos (separadas).
- Cada conta mostra saldo atual e data da última conferência (US08); cada cartão mostra a fatura aberta, a fechada a pagar e o vencimento.
- O dependente vê só a própria conta e o próprio cartão adicional (PRD 001).
- Os valores mostrados são os do aparelho; o app indica a data da última sincronização com o servidor.

**Edge cases:**
- Conta sem nenhum lançamento → mostra o saldo inicial.
- Saldo em dólar → mostrado em dólar; o total em reais aparece só no patrimônio (PRD 008).

### US11: Controlar dinheiro em espécie

Como administrador, quero ter uma conta de dinheiro em espécie (carteira), para registrar saques e gastos em dinheiro, como fazia no Minhas Economias.

**Rules:**
- Tipo dinheiro em espécie, moeda real, dono (família ou filho), nome (ex.: "Carteira"). Pode haver mais de uma.
- **Saque** na conta corrente é **transferência** da corrente para a carteira (não é despesa).
- **Gasto em dinheiro** é despesa lançada na carteira, com categoria, como qualquer outro (PRD 003).
- Dinheiro recebido em espécie é receita lançada na carteira.
- O saldo da carteira é conferido por contagem, como a conferência com o extrato (US08); a diferença vira ajuste de saldo.
- Não há importação para a carteira (não existe extrato); só lançamento manual.
- Entra no patrimônio da família quando o dono é a família (PRD 008).

**Edge cases:**
- Gasto em dinheiro maior que o saldo da carteira → aceito, com aviso de saldo negativo (provavelmente faltou lançar um saque).
- Saque que aparece no extrato importado da conta corrente (PRD 004) → reconhecido como transferência para a carteira quando o casal confirmar; sem carteira cadastrada, o app sugere criar uma.

## 4. Fluxo de Negócio

```
Compra no cartão (data da compra)
   │
   ▼
Antes ou depois do dia de fechamento?
   ├── antes ──▶ fatura do ciclo atual
   └── depois ─▶ fatura do próximo ciclo
                    │
                    ▼
            Fatura fecha ──▶ compromisso a pagar no vencimento
                    │
                    ▼
            Pagamento (transferência da conta corrente)
                    │
                    ▼
            Total pago?
               ├── sim ──▶ todas as compras contam no mês do pagamento
               └── não ──▶ conta a parte paga; saldo + juros vão para a próxima fatura
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Todas as contas atuais da família podem ser cadastradas (BB, Santander, XP x2, BBPrev, exterior, dinheiro em espécie, contas dos filhos) | o app substitui a planilha e o Minhas Economias | cadastrar a lista real; nenhuma fica de fora |
| Compra cai na fatura certa pelo dia de fechamento | o valor a pagar precisa bater com o banco | lançar compras no dia anterior e no dia do fechamento: vão para faturas diferentes |
| Compras da fatura contam no mês em que ela é paga | regime de caixa decidido pelo casal | compra em 25/03, fatura paga em 10/04: aparece no mês de abril |
| Pagamento da fatura não aparece como despesa | não contar o gasto duas vezes | total de despesas do mês não muda ao registrar o pagamento além das compras |
| Cada compra mostra o cartão que a fez (titular ou adicional) | filho vê só as dele; casal vê gastos por filho | fatura com compras do titular e dos dois adicionais: cada uma ligada ao cartão certo |
| Conta do filho fora dos relatórios e do patrimônio da família | "o que o filho faz é dele" | movimentar a conta do filho: relatórios da família não mudam |
| Saldo calculado comparado com o saldo do extrato, com a diferença visível | descobrir lançamento faltante | informar saldo diferente: diferença mostrada; criar ajuste: conta conferida |
| Conta em dólar mostra saldo em dólar | controle simplificado do exterior | lançar extrato mensal: saldo em dólar com a data do extrato |
| Saque é transferência para a carteira e gasto em dinheiro é despesa na carteira | controle do dinheiro em espécie, como no Minhas Economias | sacar R$ 200 e gastar R$ 50: despesas do mês +50, saldo da carteira 150 |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Contas e cartões da família cadastrados no app | 0 (planilha não tem contas) | todos | 1 mês após a entrega | correntes e cartões | casal |
| Faturas do mês com total igual ao do banco | não se aplica | todas | a partir do 2º mês de uso | diferença explicada em todas | casal |
| Contas conferidas com o extrato no mês | não se aplica | todas as correntes e cartões | mensal | as correntes | casal |

## 6. Milestones

### Milestone 1: Contas da família no app

**Por que é um marco:** o casal passa a ver num só lugar onde está o dinheiro da família, com saldos conferidos com o banco.

**Funcionalidades:** US01, US02, US08, US09, US10, US11

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Todas as contas atuais da família podem ser cadastradas
- [ ] Saldo calculado comparado com o saldo do extrato, com a diferença visível
- [ ] Saque é transferência para a carteira e gasto em dinheiro é despesa na carteira

**Aprovador:** casal

### Milestone 2: Cartões e faturas sob controle

**Por que é um marco:** o casal sabe quanto vai pagar em cada cartão, quem gastou, e o regime de caixa passa a valer.

**Funcionalidades:** US05, US06, US07

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Compra cai na fatura certa pelo dia de fechamento
- [ ] Compras da fatura contam no mês em que ela é paga
- [ ] Pagamento da fatura não aparece como despesa
- [ ] Cada compra mostra o cartão que a fez (titular ou adicional)

**Aprovador:** casal

### Milestone 3: Contas dos filhos e do exterior

**Por que é um marco:** o app cobre todas as contas da família, inclusive as dos filhos (separadas) e as em dólar.

**Funcionalidades:** US03, US04

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Conta do filho fora dos relatórios e do patrimônio da família
- [ ] Conta em dólar mostra saldo em dólar

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Banco muda a data de fechamento sem aviso e a compra cai na fatura errada | Médio | correção por fatura (US06); total da fatura importada comparado com o calculado | Pendente |
| Regime de caixa com pagamento parcial confunde os relatórios | Baixo | a família sempre paga o total; regra proporcional explícita para a exceção (US07) | Mitigado |
| Esquecer de lançar o extrato mensal do exterior | Baixo | patrimônio mostra a data do último saldo (US03) | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 001: perfis e dono das contas | Interna | pronto | Milestone 3 |
| PRD 003: lançamentos, parcelas e transferências | Interna | a escrever | Milestones 1 e 2 |
| PRD 006: aplicações dentro das contas de investimento | Interna | a escrever | nenhum (complementa) |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — instituições, regime de caixa, contas dos filhos, exterior em dólar.
- `docs/prds/001-usuarios-perfis-e-acesso.md` — perfis e visibilidade por dono da conta.

## 9. Registro de Decisões

- **2026-11-03:** Tipos de conta fixos: corrente, investimento, previdência, exterior, cartão. Motivo: cobre as contas da família; simplifica relatórios e IR.
- **2026-11-03:** Compra no cartão conta no mês do pagamento da fatura; pagamento é transferência. Motivo: decisões do casal no brainstorm (rodadas 3 e 4).
- **2026-11-03:** Conta do filho tem o filho como dono e fica fora dos relatórios e do patrimônio da família. Motivo: decisão do casal no brainstorm (rodadas 4 e 5).
- **2026-11-03:** `depends_on: ["001"]`. Critério: o dono da conta e a visibilidade por perfil vêm do PRD 001. O PRD 003 (lançamentos) depende deste, não o contrário.
- **2026-11-03:** Revisão do casal: sempre pagam a fatura inteira (pagamento parcial vira exceção); contas da XP são de investimento, sem conta corrente; previdência com tipo de plano (só PGBL é dedutível) e participante; **dinheiro em espécie entra no escopo** como tipo de conta (US11), como no Minhas Economias; demais premissas aceitas. Status mantido em `rascunho` até a revisão final de todos os PRDs (1ª revisão aprovada).
- **2026-11-03:** A conta de pagamento do cartão pode ser corrente ou de investimento. Motivo: os cartões da XP são da conta de investimento, já que a família não usa conta corrente na XP.
