---
prd_number: "003"
status: rascunho
priority: crítica
created: 2026-11-03
issue: ""
depends_on: ["001", "002"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/001-usuarios-perfis-e-acesso.md
  - docs/prds/002-contas-e-cartoes.md
---

# PRD 003: Lançamentos e categorias

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: no Minhas Economias o casal digitava todas as despesas; hoje lança receitas e despesas numa planilha simples, sem parcelas, contas fixas nem previsão.
- **Problema**: o lançamento é o coração do app: alimenta saldos, faturas, orçamento, relatórios e IR. Ele precisa ser rápido no celular (o casal lança na hora), respeitar o regime de caixa do cartão, distinguir despesa de transferência, prever parcelas e contas fixas, e guardar no momento certo os dados que o IR vai pedir em fevereiro.

> Contas e cartões: PRD 002. Importação e conciliação com o que foi digitado: PRD 004. Contexto técnico: `docs/trd.md`.

## 2. Solução Proposta

### Visão de produto

- Três tipos de lançamento: **despesa**, **receita** e **transferência** entre contas da família.
- Cada lançamento tem um **mês de referência**, que é o que vale para orçamento e relatórios: o mês da data, nas contas; o mês do pagamento da fatura, no cartão.
- **Categorias com subcategorias**, começando pelas do Minhas Economias, editáveis pelo casal.
- **Parcelas** e **contas fixas** geram lançamentos **previstos**, que mostram quanto dos próximos meses já está comprometido.
- Categorias marcadas como **dedutíveis** (saúde, educação, previdência) pedem, no lançamento, o **CPF ou CNPJ do prestador** e **quem foi atendido**.

### Decisões de produto

1. Transferência entre contas da família não é despesa nem receita. Motivo: decidido no brainstorm; senão o gasto conta duas vezes.
2. Mesada para os filhos é despesa da família (categoria própria), não transferência. Motivo: decidido no brainstorm.
3. Compra no cartão conta no mês do pagamento da fatura. Motivo: decidido no brainstorm (regime de caixa); a data real da compra fica guardada.
4. Dados de IR são pedidos na hora do lançamento, não no fim do ano. Motivo: em fevereiro ninguém lembra o CPF do dentista de abril.
5. Lançamento previsto não conta como realizado: só entra no realizado quando confirmado ou conciliado com a importação. Motivo: orçamento x realizado precisa refletir o que aconteceu.

### Fora do escopo

- Anexar foto do recibo ou da nota ao lançamento *(premissa — confirme ou corrija; útil para o IR, mas pesa no armazenamento e na sincronização; pode vir depois)*.
- Etiquetas (tags) livres além de categoria e subcategoria *(premissa — confirme ou corrija)*.
- Terceiro nível de categoria *(premissa — confirme ou corrija; dois níveis, como no Minhas Economias)*.
- Rateio entre o casal (todas as despesas são comuns, decidido no brainstorm).
- Lançamento por voz ou leitura de nota fiscal por foto *(premissa — confirme ou corrija)*.

## 3. Funcionalidades

### US01: Lançar despesa ou receita

Como membro da família, quero lançar uma despesa ou receita em poucos toques no celular, para registrar na hora em que acontece.

**Rules:**
- Campos: tipo (despesa ou receita), **valor**, **data**, **conta ou cartão** (para cartão: qual cartão, titular ou adicional), **categoria e subcategoria**, descrição, favorecido ou pagador (estabelecimento, empresa, pessoa) e observação.
- Obrigatórios: tipo, valor, data, conta ou cartão e categoria *(premissa — confirme ou corrija; descrição opcional)*.
- O app sugere a categoria a partir do favorecido já usado antes (ex.: "Padaria X" → Alimentação > Padaria) *(premissa — confirme ou corrija)*.
- **Mês de referência**: nas contas (corrente, investimento, dinheiro, exterior), o mês da data; no cartão, o mês em que a fatura daquela compra é paga (PRD 002). Enquanto a fatura não é paga, a compra fica no mês previsto do vencimento.
- Lançamento em conta em dólar fica em dólar (PRD 002).
- O dependente só lança no próprio cartão adicional e na própria conta (PRD 001).
- Lançamento digitado fica marcado como **digitado**, aguardando conciliação com a importação, quando houver (PRD 004).

**Edge cases:**
- Data no futuro → vira lançamento **previsto** (US05), não realizado.
- Valor zero ou negativo → recusado; estorno e devolução são lançados como receita na mesma categoria ou como crédito na fatura (US09).
- Lançamento feito offline → sincroniza depois, em casa (PRD 011); o mês de referência não muda por isso.
- Categoria de receita escolhida para despesa (ou o contrário) → o app não permite; categorias têm tipo (US06).

### US02: Transferir entre contas da família

Como administrador, quero registrar transferências entre contas da família, para que saques, aplicações e pagamentos de fatura não apareçam como gasto.

**Rules:**
- Campos: conta de origem, conta de destino, valor, data, descrição.
- Casos: transferência entre correntes; **saque** (corrente → dinheiro em espécie); **aplicação** e **resgate** (corrente ↔ investimento); **contribuição de previdência** (corrente → previdência); **pagamento de fatura** (conta de pagamento → cartão, PRD 002); remessa ao exterior (reais → dólares, com os dois valores).
- Transferência não tem categoria e não entra em despesas, receitas nem orçamento.
- Exceção: contribuição para previdência **PGBL** também aparece como despesa dedutível de previdência no relatório do IR (PRD 002, PRD 010), sem entrar nas despesas do mês.
- Transferência da família para a conta de um filho: o casal escolhe se é **mesada** (despesa da família, categoria Mesada) ou outro tipo de despesa; nunca é transferência neutra, porque a conta do filho fica fora da família (PRD 002).

**Edge cases:**
- Transferência importada só de um lado (ex.: extrato da corrente mostra a aplicação, mas o extrato da corretora ainda não foi importado) → fica registrada; a importação do outro lado é conciliada com ela (PRD 004).
- Origem igual ao destino → recusado.
- Transferência para conta encerrada → o app avisa (PRD 002).

### US03: Lançar compra parcelada

Como administrador, quero lançar uma compra parcelada no cartão uma única vez, para que cada parcela apareça na fatura certa.

**Rules:**
- Campos adicionais: número de parcelas e valor total (ou valor da parcela).
- O app cria uma parcela em cada fatura a partir da fatura da compra; cada parcela tem o mês de referência do pagamento da fatura dela (regime de caixa).
- Parcelas futuras ficam **previstas** até a fatura correspondente ser paga.
- Diferença de centavos na divisão vai para a primeira parcela *(premissa — confirme ou corrija; alguns bancos usam a última)*.
- Editar a categoria de uma parcela pergunta se vale para todas as parcelas *(premissa — confirme ou corrija)*.
- Parcelamento também vale para compras em conta (ex.: boleto parcelado) como série de lançamentos previstos mensais *(premissa — confirme ou corrija)*.

**Edge cases:**
- Compra parcelada estornada no meio → parcelas futuras canceladas; as já pagas continuam, e o estorno entra como crédito.
- Fatura importada mostra "parcela 3/10" de compra não lançada antes (ex.: anterior ao início do app) → o app cria a série a partir da parcela 3 (PRD 004).
- Antecipação de parcelas → as parcelas antecipadas passam para a fatura em que foram cobradas.

### US04: Cadastrar contas fixas (recorrentes)

Como administrador, quero cadastrar contas fixas (luz, escola, assinaturas, salário), para que o app as preveja todo mês.

**Rules:**
- Campos: descrição, tipo (despesa ou receita), categoria, conta ou cartão, **valor fixo ou estimado**, frequência (mensal, anual, outra), dia previsto, início e fim opcional.
- O app gera os lançamentos **previstos** dos próximos **12 meses** *(premissa — confirme ou corrija)*.
- Quando o valor real chega (digitado ou importado), o previsto é **confirmado** com o valor real e sai da lista de previstos.
- Valor estimado (ex.: luz) usa o valor informado; opcionalmente a média dos últimos 3 meses *(premissa — confirme ou corrija)*.
- Mudar o valor ou o dia de uma conta fixa vale para os previstos futuros, não para os já confirmados.

**Edge cases:**
- Previsto vencido e não confirmado → aparece como **atrasado** para o casal revisar (pagar, confirmar ou descartar).
- Dia previsto 31 em mês de 30 dias → último dia do mês.
- Conta fixa paga no cartão → o previsto fica no mês do pagamento da fatura, como qualquer compra no cartão.

### US05: Ver previstos e comprometimento dos próximos meses

Como administrador, quero ver o que já está comprometido nos próximos meses, para saber quanto da renda está livre.

**Rules:**
- Previstos vêm de: parcelas futuras, contas fixas, faturas fechadas a pagar e lançamentos com data futura.
- Visão por mês: receitas previstas, despesas previstas, comprometimento (despesas previstas sobre receitas previstas) *(premissa — confirme ou corrija a forma de mostrar)*.
- Previsto não conta como realizado nos relatórios (PRD 008), mas aparece no orçamento como "previsto" ao lado do realizado (PRD 007).
- O dependente vê os previstos só do próprio cartão adicional e da própria conta.

**Edge cases:**
- Previsto descartado (assinatura cancelada) → some dos meses seguintes se a conta fixa for encerrada; se descartado só naquele mês, os outros continuam.
- Previsto e lançamento real com valores diferentes → vale o real; a diferença é visível no detalhe.

### US06: Organizar categorias e subcategorias

Como administrador, quero editar, criar e reorganizar categorias e subcategorias, para que reflitam como a família pensa os gastos.

**Rules:**
- Dois níveis: categoria e subcategoria. Lançamento pode usar só a categoria ou a subcategoria.
- Cada categoria tem **tipo**: despesa ou receita.
- Ponto de partida: as categorias da exportação do Minhas Economias (PRD 005). Antes da migração, o app traz uma lista inicial padrão *(premissa — confirme ou corrija)*.
- O casal pode: criar, renomear, mover subcategoria para outra categoria, **juntar** duas categorias (os lançamentos passam para a que fica) e **desativar** (some da escolha, continua no histórico).
- Excluir de vez só categoria sem lançamentos.
- Categoria pode ser marcada como **dedutível no IR**, com o tipo: saúde, educação ou previdência (US07).
- Categoria fixa **Mesada** (despesa) e categoria fixa de **ajuste de saldo** (fora de despesa e receita, PRD 002) *(premissa — confirme ou corrija os nomes)*.
- O dependente usa as categorias da família, sem editar (PRD 001).

**Edge cases:**
- Renomear categoria usada no orçamento → o orçamento acompanha (PRD 007).
- Juntar categorias com orçamentos diferentes → o orçamento da que fica soma os dois, e o app avisa.
- Mover subcategoria de "despesa" para categoria de "receita" → não permitido.

### US07: Registrar dados para o IR no lançamento

Como administrador, quero informar o prestador e quem foi atendido nas despesas dedutíveis, para gerar o relatório do IR sem garimpar recibos.

**Rules:**
- Em categoria dedutível de **saúde** ou **educação**, o app pede: **CPF ou CNPJ do prestador**, **nome do prestador** e **quem foi atendido** (titular, cônjuge ou um dos filhos, declaração conjunta com filhos dependentes).
- O app confere os dígitos verificadores do CPF ou CNPJ.
- O app lembra os dados do prestador: ao escolher o favorecido "Dr. Fulano", preenche CPF/CNPJ e nome *(premissa — confirme ou corrija)*.
- Os dados podem ficar em branco na hora e ser completados depois; o relatório do IR lista as despesas dedutíveis **incompletas** (PRD 010).
- Previdência: o dado é o plano (PRD 002); não pede prestador no lançamento.

**Edge cases:**
- Reembolso do plano de saúde → lançado como receita ligada à despesa reembolsada; o relatório do IR abate o reembolso *(premissa — confirme ou corrija)*.
- Uma consulta para dois atendidos → dividir o lançamento (US08), cada parte com quem foi atendido.
- Despesa dedutível no cartão adicional de um filho → o filho não vê nem preenche os dados de IR; o casal completa *(premissa — confirme ou corrija)*.
- CPF/CNPJ inválido → o app avisa, mas aceita salvar marcado como incompleto.

### US08: Dividir um lançamento em várias categorias

Como administrador, quero dividir um lançamento (ex.: compra de supermercado com remédio), para que cada parte vá para a categoria certa.

**Rules:**
- O lançamento tem um valor total e várias partes, cada uma com categoria, valor e, se dedutível, os dados de IR.
- A soma das partes é igual ao total.
- Na conta ou fatura aparece um lançamento só, com o total; nos relatórios e no orçamento, cada parte na sua categoria.

**Edge cases:**
- Partes que não fecham o total → o app mostra a diferença e não salva até fechar.
- Lançamento dividido conciliado com importação → a conciliação usa o total (PRD 004).

### US09: Encontrar, corrigir e excluir lançamentos

Como administrador, quero buscar, editar e excluir lançamentos, para corrigir erros.

**Rules:**
- Busca por texto (descrição, favorecido, observação), período, conta ou cartão, categoria, valor, tipo e situação (previsto, realizado, digitado não conciliado, importado).
- Qualquer campo pode ser editado; o histórico registra quem alterou (PRD 001).
- Excluir pede confirmação.
- **Estorno ou devolução**: lançado como receita na mesma categoria da despesa (nas contas) ou crédito na fatura (no cartão), abatendo a despesa no mês de referência dele *(premissa — confirme ou corrija)*.
- O dependente só edita e exclui o que é dele, conforme o PRD 001.

**Edge cases:**
- Editar lançamento de mês com conta já conferida com o extrato → a conferência deixa de valer (PRD 002).
- Excluir lançamento importado → o app avisa que uma nova importação do mesmo arquivo não deve recriá-lo (PRD 004).
- Edição simultânea em dois aparelhos fora de casa → regra de conflito do PRD 011.

## 4. Fluxo de Negócio

```
Novo lançamento
   │
   ▼
Entre contas da família?
   ├── sim ──▶ Transferência (sem categoria; fora de despesas e receitas)
   │              └── destino é conta de filho? ──▶ não é transferência: despesa (ex.: Mesada)
   └── não ──▶ Despesa ou receita, com categoria
                  │
                  ▼
           Categoria dedutível? ──sim──▶ pede CPF/CNPJ, prestador, atendido
                  │
                  ▼
           No cartão? ──sim──▶ mês de referência = mês do pagamento da fatura
                  │                (parcelada: uma parcela por fatura)
                  └── não ──▶ mês de referência = mês da data
                  │
                  ▼
           Data futura ou série? ──sim──▶ previsto até confirmar
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Lançar uma despesa no celular leva até 15 s com categoria sugerida | o casal lança na hora; se for lento, volta a esquecer *(premissa — confirme ou corrija o limiar)* | cronometrar um lançamento repetido de favorecido conhecido |
| Transferências (saque, aplicação, pagamento de fatura) não alteram despesas e receitas | não contar duas vezes | registrar cada tipo: totais do mês não mudam |
| Mesada para o filho entra como despesa da família | decidido no brainstorm | transferir para a conta do filho como mesada: despesa na categoria Mesada |
| Compra parcelada em 10x gera 10 parcelas, cada uma na fatura certa | parcelas futuras visíveis | lançar compra 10x: dez faturas com uma parcela cada |
| Compra no cartão aparece no mês do pagamento da fatura | regime de caixa | compra em 25/03, fatura paga em 10/04: aparece em abril |
| Conta fixa gera previstos dos próximos 12 meses e é confirmada pelo valor real | comprometimento futuro | cadastrar luz; lançar o valor real: previsto confirmado com o valor real |
| Despesa em categoria de saúde pede CPF/CNPJ e atendido e confere os dígitos | IR sem garimpar recibos | lançar consulta com CPF inválido: aviso e marcado incompleto |
| Juntar duas categorias move todos os lançamentos | categorias do Minhas Economias precisam de limpeza | juntar A em B: lançamentos de A aparecem em B |
| Lançamento dividido soma o total e cada parte vai para a sua categoria | supermercado com remédio | dividir R$ 300 em 250 + 50: relatório por categoria mostra as duas partes |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Lançamentos do mês sem categoria | A levantar (planilha atual) | 0 | a partir do 2º mês de uso | menos de 5 | casal |
| Despesas dedutíveis com dados de IR completos no fim do ano | 0 (hoje se garimpa recibos) | todas | 31/12 do primeiro ano de uso | 90 % | casal |
| Contas fixas cadastradas | 0 | todas as conhecidas | 1 mês após a entrega | as maiores (escola, moradia, saúde) | casal |

## 6. Milestones

### Milestone 1: Lançar o dia a dia

**Por que é um marco:** o casal passa a registrar todas as despesas, receitas e transferências no app, com categorias próprias, substituindo a planilha.

**Funcionalidades:** US01, US02, US06, US08, US09

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Lançar uma despesa no celular leva até 15 s com categoria sugerida
- [ ] Transferências (saque, aplicação, pagamento de fatura) não alteram despesas e receitas
- [ ] Mesada para o filho entra como despesa da família
- [ ] Compra no cartão aparece no mês do pagamento da fatura
- [ ] Juntar duas categorias move todos os lançamentos
- [ ] Lançamento dividido soma o total e cada parte vai para a sua categoria

**Aprovador:** casal

### Milestone 2: Parcelas, contas fixas e o futuro à vista

**Por que é um marco:** o casal vê quanto dos próximos meses já está comprometido, o que nem o Minhas Economias nem a planilha mostravam de forma clara.

**Funcionalidades:** US03, US04, US05

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Compra parcelada em 10x gera 10 parcelas, cada uma na fatura certa
- [ ] Conta fixa gera previstos dos próximos 12 meses e é confirmada pelo valor real

**Aprovador:** casal

### Milestone 3: Lançamentos prontos para o IR

**Por que é um marco:** as despesas dedutíveis passam a ser registradas com o que a declaração pede, no momento em que acontecem.

**Funcionalidades:** US07

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Despesa em categoria de saúde pede CPF/CNPJ e atendido e confere os dígitos

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Lançar no celular ser lento e o casal desistir de digitar | Alto | sugestão de categoria pelo favorecido; importação como alternativa (PRD 004) | Pendente |
| Pedir dados de IR na hora atrapalhar o lançamento rápido | Médio | campos podem ficar em branco e ser completados depois; relatório de incompletos | Pendente |
| Categorias herdadas do Minhas Economias desorganizadas | Baixo | juntar, mover e desativar (US06) | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 001: perfis e permissões | Interna | pronto | todos |
| PRD 002: contas, cartões, faturas | Interna | pronto | todos |
| PRD 005: categorias do Minhas Economias | Interna | a escrever | nenhum (há lista inicial padrão) |
| PRD 010: uso dos dados de IR | Interna | a escrever | Milestone 3 (forma dos dados) |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — regime de caixa, transferências, mesada, categorias, recorrentes, IR.
- `docs/prds/001-usuarios-perfis-e-acesso.md` — o que o dependente lança e edita.
- `docs/prds/002-contas-e-cartoes.md` — contas, cartões, faturas, pagamento e conta de pagamento.

## 9. Registro de Decisões

- **2026-11-03:** Três tipos de lançamento (despesa, receita, transferência) e mês de referência pelo regime de caixa no cartão. Motivo: decisões do brainstorm (rodadas 3 e 4).
- **2026-11-03:** Mesada é despesa da família, nunca transferência neutra. Motivo: decisão do casal (rodada 4); a conta do filho fica fora da família.
- **2026-11-03:** Dados de IR pedidos no lançamento, com possibilidade de completar depois. Motivo: decisão do casal (rodada 6) de registrar CPF/CNPJ e atendido.
- **2026-11-03:** Previsto não conta como realizado. Motivo: orçamento x realizado precisa refletir o que aconteceu (validado na consolidação).
- **2026-11-03:** `depends_on: ["001", "002"]`. Critério: o lançamento pressupõe perfis e permissões (001) e contas, cartões e faturas (002).
