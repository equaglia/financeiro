---
prd_number: "009"
status: rascunho
revisao: "aguardando a 1ª revisão do casal"
priority: média
created: 2026-11-03
issue: ""
depends_on: ["001", "002", "003"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/001-usuarios-perfis-e-acesso.md
  - docs/prds/002-contas-e-cartoes.md
  - docs/prds/003-lancamentos-e-categorias.md
---

# PRD 009: Filhos — limite do cartão adicional e semanada

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: cada filho tem um **cartão adicional** no cartão do casal, com **limite mensal**, e uma **conta individual** onde recebe a **semanada por Pix**. No Minhas Economias a semanada era a categoria "Semanada Filhos", lançada nas contas correntes do casal, e o dinheiro em espécie de cada filho tinha conta própria. Os filhos não usam nenhum app de controle e hoje não se interessam.
- **Problema**: os filhos não sabem quanto do limite já usaram, e o casal só descobre quando a fatura chega. A semanada é um gasto fixo da família que se repete toda semana e precisa entrar certo: despesa da família e receita do filho. O casal quer que os filhos passem a usar o app; para isso o app precisa mostrar a eles algo útil logo de cara.

> Perfis e o que o filho vê e faz: PRD 001. Cartão adicional, limite e conta do filho: PRD 002. Lançamentos recorrentes e categoria Semanada: PRD 003.

## 2. Solução Proposta

### Visão de produto

- O casal define o **limite mensal** de cada cartão adicional; o app acompanha o consumo **por ciclo de fatura**.
- O filho vê, ao abrir o app, **quanto ainda pode gastar** no ciclo, as compras recentes, a próxima semanada e o saldo da conta dele.
- **Avisos** ao filho e ao casal quando o consumo chega perto do limite e quando passa. O app avisa, não bloqueia: quem bloqueia é o banco.
- A **semanada** de cada filho é cadastrada uma vez (valor, dia da semana, conta de origem e de destino) e gera os pagamentos previstos de toda semana. Registrar um pagamento cria de uma vez a despesa da família e a receita do filho.

### Decisões de produto

1. Limite acompanhado por ciclo de fatura. Motivo: decidido no brainstorm (é assim que o banco controla).
2. O app avisa e não bloqueia. Motivo: validado na consolidação do brainstorm.
3. Semanada é despesa da família (categoria Semanada) e receita do filho. Motivo: decidido no brainstorm (rodada 4).
4. Compras no cartão adicional são despesas da família, com a categoria da compra (não "Semanada"). Motivo: decidido no brainstorm (rodada 4).
5. Os filhos veem o próprio limite, as próprias compras e a própria conta, e nada da família. Motivo: PRD 001.

### Fora do escopo

- Bloquear o cartão ou mudar o limite no banco (sem integração com bancos).
- Metas de poupança dos filhos, "cofrinhos" ou recompensas *(premissa — confirme ou corrija)*.
- Pedido de dinheiro extra pelo app (filho pede, casal aprova) *(premissa — confirme ou corrija)*.
- Semanada condicionada a tarefas *(premissa — confirme ou corrija)*.

## 3. Funcionalidades

### US01: Definir o limite mensal do cartão adicional

Como administrador, quero definir o limite mensal do cartão adicional de cada filho, para acompanhar quanto ele pode gastar.

**Rules:**
- Valor do limite por cartão adicional, válido a partir de um ciclo de fatura (PRD 002, US05).
- Mudar o limite vale a partir do ciclo escolhido; o histórico de limites fica guardado.
- O limite pode ser o mesmo configurado no banco ou um limite combinado em família, menor que o do banco *(premissa — confirme ou corrija)*.
- Só administradores definem o limite; o filho só vê.

**Edge cases:**
- Cartão adicional sem limite definido → o app mostra só o total gasto no ciclo, sem avisos.
- Limite reduzido no meio do ciclo para menos do que já foi gasto → o ciclo aparece como estourado e o aviso de 100 % é dado.

### US02: Acompanhar o consumo do ciclo

Como filho, quero saber quanto ainda posso gastar no meu cartão neste ciclo, para não passar do limite; como administrador, quero ver o mesmo para cada filho.

**Rules:**
- Consumo do ciclo = compras no cartão adicional com data dentro do ciclo de fatura atual (do fechamento anterior até o próximo) − estornos.
- **Disponível** = limite − consumo; também em percentual.
- Entram as compras **registradas pelo filho** (PRD 001, US06) e as **importadas ou lançadas pelo casal**; uma compra registrada e depois importada conta uma vez só (conciliação, PRD 004).
- Compra parcelada consome o limite pelo **valor total** no ciclo da compra, como o banco faz com o limite *(premissa — confirme ou corrija; alternativa: só a parcela do ciclo)*.
- O ciclo segue o dia de fechamento do cartão do casal; ao fechar, o consumo recomeça.
- O casal vê os cartões adicionais de todos os filhos lado a lado.

**Edge cases:**
- Compra feita e não registrada pelo filho, e fatura ainda não importada → o consumo fica abaixo do real; o app mostra a data da última importação da fatura para o filho saber até quando o número é confiável.
- Compra internacional com IOF → o IOF conta no consumo *(premissa — confirme ou corrija)*.
- Dia de fechamento alterado pelo banco numa fatura (PRD 002, US06) → o ciclo usa a data corrigida.

### US03: Avisar quando o limite aperta

Como filho, quero ser avisado quando estou perto do limite, para segurar os gastos; como administrador, quero saber quando um filho passou do limite.

**Rules:**
- Aviso ao **filho** ao chegar a **80 %** e ao passar de **100 %** do limite do ciclo *(premissa — confirme ou corrija os percentuais)*.
- Aviso ao **casal** ao passar de **100 %** *(premissa — confirme ou corrija; o casal também quer o aviso de 80 %?)*.
- Um aviso por limite por ciclo, no próximo uso do app depois de a compra ser registrada ou sincronizada.
- O casal pode desligar os avisos de um filho.

**Edge cases:**
- Compra registrada pelo filho fora de casa → o aviso para ele sai na hora (o cálculo é no aparelho dele); para o casal, só depois da sincronização em casa (PRD 011).
- Estorno traz o consumo de volta abaixo de 100 % → sem novo aviso; o estado volta a "dentro do limite".

### US04: Cadastrar a semanada de cada filho

Como administrador, quero cadastrar a semanada de cada filho uma vez, para que o app preveja os pagamentos de toda semana.

**Rules:**
- Por filho: valor, **dia da semana**, conta de origem da família (corrente) e destino (conta individual do filho ou a carteira de dinheiro dele, PRD 002).
- Gera os pagamentos **previstos** semanais, como uma conta fixa (PRD 003, US04), na categoria Semanada.
- Mudar o valor vale a partir da data escolhida; o histórico fica.
- A semanada pode ser pausada (férias, castigo) e retomada *(premissa — confirme ou corrija)*.

**Edge cases:**
- Semana com feriado ou viagem em que a semanada é paga adiantada → o pagamento confirma o previsto da semana, com a data real.
- Filho sem conta individual → destino é a carteira de dinheiro dele.

### US05: Registrar o pagamento da semanada

Como administrador, quero registrar o Pix da semanada com um toque, para que entre como despesa da família e receita do filho.

**Rules:**
- Confirmar o previsto da semana cria: **despesa da família** na conta de origem, categoria Semanada; **receita** na conta do filho (ou na carteira dele).
- O Pix que aparece no extrato importado da conta da família (PRD 004) é reconhecido como pagamento da semanada e confirma o previsto.
- Semanada atrasada (previsto vencido e não confirmado) → aparece como atrasada para o casal (PRD 003, US04).
- O filho vê a receita na conta dele e a data da próxima semanada.

**Edge cases:**
- Pix da semanada com valor diferente do cadastrado → confirmado com o valor real, sem mudar o cadastro.
- Dois filhos pagos num Pix só (raro) → dividido em dois lançamentos (PRD 003, US08).
- Semanada paga em dinheiro → origem é a carteira do administrador que pagou, destino é a carteira do filho.

### US06: Tela inicial do filho

Como filho, quero abrir o app e ver logo o que importa para mim, para que valha a pena usá-lo.

**Rules:**
- Mostra: **disponível do limite** no ciclo (valor e barra), data do fechamento, compras recentes do cartão (com as que ele ainda não categorizou destacadas), **saldo da conta** dele, **próxima semanada** (valor e dia) e o saldo da carteira de dinheiro, se tiver.
- Atalho para **registrar uma compra** no cartão adicional e para **categorizar** as pendentes (PRD 001, US06).
- Nada da família aparece (PRD 001, US05).

**Edge cases:**
- Filho sem cartão adicional → a tela mostra só conta, carteira e semanada.
- Dados ainda não sincronizados desde a última importação da fatura → a tela mostra a data da última atualização.

### US07: Dar dinheiro extra aos filhos

Como administrador, quero registrar dinheiro dado aos filhos fora da semanada (presente, adiantamento, despesa escolar), para que entre certo nos números da família.

**Rules:**
- Transferência da família para a conta ou carteira do filho fora da semanada é **despesa da família**, com a categoria escolhida pelo casal (sugestão: "Despesas dos filhos", `docs/categorias-iniciais.md`) e **receita do filho**, como na semanada.
- Adiantamento de semanada: confirma previstos futuros, que deixam de ser pagos nas semanas seguintes *(premissa — confirme ou corrija)*.
- Dinheiro devolvido pelo filho à família → receita da família na mesma categoria (abate a despesa) e despesa na conta do filho.

**Edge cases:**
- Reembolso de compra que o filho pagou pela família (ex.: ele comprou pão para casa) → despesa da família na categoria da compra (Padaria), não em "Despesas dos filhos".
- Valor extra lançado como semanada por engano → reclassificável; o previsto da semana continua pendente.

## 4. Fluxo de Negócio

```
Compra no cartão adicional (registrada pelo filho ou importada)
   │
   ▼
Dentro do ciclo atual? ─não─▶ conta no próximo ciclo
   │ sim
   ▼
consumo do ciclo ÷ limite
   ├── < 80 % ──▶ nada
   ├── ≥ 80 % ──▶ aviso ao filho
   └── > 100 % ─▶ aviso ao filho e ao casal (o app não bloqueia)

Semanada prevista (toda semana)
   │
   ▼
Pix feito? ─sim─▶ despesa da família (Semanada) + receita na conta do filho
   │ não, venceu
   ▼
atrasada (casal vê)
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Disponível do limite = limite − compras do ciclo + estornos | filho sabe quanto pode gastar | limite 500, compras de 200 e 150, estorno de 50: disponível 200 |
| Compra antes e depois do fechamento cai em ciclos diferentes | ciclo de fatura, como o banco | compra no dia do fechamento e no dia seguinte: ciclos diferentes |
| Compra registrada pelo filho e depois importada conta uma vez só | não dar falso alarme de limite | registrar 100 e importar a fatura com a mesma compra: consumo 100 |
| Aviso ao filho a 80 % e a 100 %; ao casal a 100 %; um por ciclo | segurar os gastos sem incomodar | passar de 80 % e 100 % com várias compras: dois avisos ao filho, um ao casal |
| Semanada cadastrada gera os previstos de toda semana | gasto fixo da família | cadastrar semanada de sexta: um previsto por sexta |
| Confirmar a semanada cria despesa da família e receita do filho | regra do brainstorm | confirmar 100: despesa Semanada 100 na família e receita 100 na conta do filho |
| Tela do filho mostra disponível do limite, compras recentes, saldo e próxima semanada | o filho ter motivo para usar o app | abrir o app no celular do filho: os quatro itens visíveis, nada da família |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Filhos que abrem o app ao menos uma vez por semana | 0 (não usam) | 2 | 3 meses após a entrega | 1 | casal |
| Ciclos com o limite estourado sem aviso prévio de 80 % | A levantar | 0 | contínuo | — | casal |
| Semanadas atrasadas no mês | A levantar | 0 | a partir do 2º mês de uso | 1 | casal |

## 6. Milestones

### Milestone 1: Limite do cartão sob controle

**Por que é um marco:** filhos e casal passam a saber, a qualquer momento, quanto do limite já foi usado, e os filhos ganham um motivo para abrir o app.

**Funcionalidades:** US01, US02, US03, US06

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Disponível do limite = limite − compras do ciclo + estornos
- [ ] Compra antes e depois do fechamento cai em ciclos diferentes
- [ ] Compra registrada pelo filho e depois importada conta uma vez só
- [ ] Aviso ao filho a 80 % e a 100 %; ao casal a 100 %; um por ciclo
- [ ] Tela do filho mostra disponível do limite, compras recentes, saldo e próxima semanada

**Aprovador:** casal

### Milestone 2: Semanada automática

**Por que é um marco:** a semanada deixa de ser lançada à mão toda semana e entra certa nos números da família e dos filhos.

**Funcionalidades:** US04, US05, US07

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Semanada cadastrada gera os previstos de toda semana
- [ ] Confirmar a semanada cria despesa da família e receita do filho

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Filhos não registram as compras e o disponível fica errado até a fatura ser importada | Médio | data da última importação na tela do filho; importar a fatura com frequência (PRD 004) | Pendente |
| Filhos continuam sem interesse no app | Médio | tela inicial focada no que é útil para eles (US06); uso dos filhos é desejável, não obrigatório | Monitorando |
| Aviso ao casal atrasado (sincronização só em casa) | Baixo | aceito pela decisão de sincronizar em casa | Mitigado |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 001: perfil dependente, o que o filho vê e faz | Interna | rascunho (1ª revisão aprovada) | todos |
| PRD 002: cartão adicional, limite, conta e carteira do filho | Interna | rascunho (1ª revisão aprovada) | todos |
| PRD 003: recorrentes, previstos, categoria Semanada | Interna | rascunho (1ª revisão aprovada) | Milestone 2 |
| PRD 004: conciliação da compra registrada com a importada; Pix da semanada no extrato | Interna | a escrever (aguarda amostras) | Milestone 1 (consumo exato) |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — limite mensal por ciclo, semanada por Pix, o que os filhos veem.
- `docs/prds/001-usuarios-perfis-e-acesso.md` — perfil dependente.
- `docs/prds/002-contas-e-cartoes.md` — cartão adicional e limite (US05), conta e carteira do filho.
- `docs/prds/003-lancamentos-e-categorias.md` — contas fixas, previstos, categoria Semanada.

## 9. Registro de Decisões

- **2026-11-03:** Limite por ciclo de fatura; o app avisa e não bloqueia. Motivo: decisões do brainstorm (rodada 5 e consolidação).
- **2026-11-03:** Semanada (não mesada), por Pix, como despesa da família e receita do filho. Motivo: decisão do casal (rodada 4; termo corrigido em 2026-11-03).
- **2026-11-03:** Tela inicial do filho focada em limite, compras, saldo e semanada. Motivo: o casal quer que os filhos usem o app, e hoje eles não se interessam.
- **2026-11-03:** `depends_on: ["001", "002", "003"]`. Critério: perfil dependente (001), cartão adicional, limite e contas do filho (002), recorrentes e previstos (003). O 004 melhora a exatidão do consumo, mas não é pré-requisito (há registro manual).
