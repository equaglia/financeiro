---
prd_number: "010"
status: rascunho
revisao: "aguardando a 1ª revisão do casal"
priority: alta
created: 2026-11-03
issue: ""
depends_on: ["001", "002", "003", "006"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/003-lancamentos-e-categorias.md
  - docs/prds/002-contas-e-cartoes.md
  - docs/prds/006-investimentos-e-previdencia.md
  - https://www.gov.br/receitafederal/pt-br/assuntos/meu-imposto-de-renda/preenchimento/manual-mir/pagamentos-ou-doacoes/despesas-dedutiveis
  - https://www.gov.br/receitafederal/pt-br/assuntos/meu-imposto-de-renda/malha-fiscal/antecipacao/despesas-medicas
  - https://www.gov.br/receitafederal/pt-br/acesso-a-informacao/perguntas-frequentes/imposto-de-renda/dirpf/deducoes/o-que-posso-deduzir-como
  - https://www.gov.br/receitafederal/pt-br/centrais-de-conteudo/publicacoes/manuais/orientacao-tributaria/receita-saude-2.1.pdf
---

# PRD 010: Apoio à declaração do imposto de renda

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: o casal faz **declaração conjunta**, com os **filhos como dependentes**. Na época da declaração, junta recibos, informes e extratos de várias fontes para preencher despesas dedutíveis e bens e direitos.
- **Problema**: o controle financeiro tem quase tudo o que a declaração pede, mas espalhado. O casal decidiu (rodada 6 do brainstorm) que o app deve **ajudar a preencher** duas partes: **despesas dedutíveis** (saúde, educação, previdência) e **bens e direitos** (saldos em 31/12). O app não gera o arquivo oficial nem transmite a declaração.

> Dados de IR no lançamento (CPF/CNPJ do prestador, quem foi atendido): PRD 003, US07. Dados das contas para bens e direitos (agência, número, CNPJ da instituição) e tipo do plano de previdência: PRD 002. Valor das aplicações: PRD 006.

## 2. Solução Proposta

### Visão de produto

- Uma área **Imposto de renda** por ano-calendário, para administradores.
- **Despesas médicas** por prestador e por pessoa atendida, com valor pago, reembolsado e dedutível, prontas para copiar para a declaração e conferir com a pré-preenchida.
- **Educação** por pessoa, com o limite do ano à vista.
- **Previdência PGBL** por participante.
- **Bens e direitos**: o saldo de cada conta e aplicação em 31/12 do ano anterior e do ano, com os dados que a declaração pede, inclusive o exterior convertido pela cotação que o casal informar para o IR.
- **Pendências do IR** ao longo do ano (despesas sem CPF/CNPJ, contas sem dados), para não acumular tudo para março.

### Decisões de produto

1. Só relatórios de apoio; nada de arquivo oficial nem transmissão. Motivo: decisão do casal (rodada 6); as regras e o programa da Receita mudam todo ano.
2. Duas partes da declaração: despesas dedutíveis e bens e direitos. Motivo: decisão do casal (rodada 6).
3. Declaração conjunta com filhos dependentes: as despesas de todos os membros entram na mesma declaração. Motivo: decisão do casal (rodada 6).
4. **Limites e cotações do IR não ficam fixos no app**: o casal informa a cada ano (limite de educação, cotação do dólar em 31/12 para o IR). Motivo: mudam de um ano para outro e as fontes divergem (ver seção 8); errar um valor fixo erraria a declaração em silêncio.
5. Contas dos filhos entram em bens e direitos (são dependentes); contas administradas não. Ver US05.

### Fora do escopo

- Gerar o arquivo da declaração ou transmiti-la (decidido no brainstorm).
- Rendimentos (tributáveis, isentos como lucros e dividendos, exclusivos na fonte), ganhos em renda variável e DARF (não escolhidos na rodada 6). O relatório de receitas por categoria (PRD 008) ajuda a conferir, mas não é relatório de IR.
- Calcular o imposto devido, comparar declaração completa e simplificada, carnê-leão.
- Ler os informes de rendimento dos bancos ou a declaração pré-preenchida *(premissa — confirme ou corrija)*.
- Dívidas e ônus reais *(premissa — confirme ou corrija; a família não citou financiamentos)*.

## 3. Funcionalidades

### US01: Preparar a declaração do ano

Como administrador, quero indicar quem é o titular e quem são os dependentes da declaração do ano, para que os relatórios saiam organizados por pessoa.

**Rules:**
- Por ano-calendário: **titular**, **cônjuge** (dependente na declaração conjunta) e **filhos dependentes**, com CPF e data de nascimento do cadastro (PRD 001, US02).
- Valores do ano informados pelo casal: **limite anual de dedução de educação por pessoa** e **cotação do dólar em 31/12 para o IR** (US05).
- O app sugere os valores do ano anterior e avisa que precisam ser conferidos na Receita.
- Só administradores veem a área de IR (PRD 001).

**Edge cases:**
- Membro sem CPF cadastrado → listado como pendência (US06).
- Filho que deixa de ser dependente num ano → desmarcado naquele ano; os anos anteriores não mudam.
- Titular diferente de um ano para outro → permitido; cada ano tem o seu.

### US02: Relatório de despesas médicas

Como administrador, quero a lista das despesas médicas do ano por prestador e por pessoa atendida, para copiar para a declaração.

**Rules:**
- Entram os lançamentos realizados no ano em categorias dedutíveis de **saúde** (PRD 003, US06 e US07), pela **data do pagamento** *(premissa — confirme ou corrija; para o IR vale o ano em que a despesa foi paga, e no cartão o app usa o pagamento da fatura, como no regime de caixa)*.
- Linha por **prestador** (CPF ou CNPJ, nome) e **pessoa atendida**: valor pago, valor **reembolsado** (plano de saúde, PRD 003, US07) e **valor dedutível** (pago − reembolsado). Só a parte não reembolsada é dedutível (Receita, ver seção 8).
- Totais por pessoa atendida e geral.
- As categorias marcadas como dedutíveis seguem a lista da Receita: médicos, dentistas, psicólogos, fisioterapeutas, terapeutas ocupacionais, fonoaudiólogos, hospitais, clínicas, laboratórios (exames) e planos de saúde. Medicamentos de farmácia, óculos e vacinas fora da conta hospitalar **não** são dedutíveis (Receita, ver seção 8). Em `docs/categorias-iniciais.md`, Farmácia e Óculos já ficam sem a marca de IR.
- Ordenado para conferir com a declaração pré-preenchida: desde 2025, recibos de profissionais de saúde pessoa física são emitidos no **Receita Saúde** e aparecem na pré-preenchida (Receita, ver seção 8). O relatório destaca os prestadores pessoa física para essa conferência.

**Edge cases:**
- Despesa dedutível sem CPF/CNPJ ou sem pessoa atendida → aparece no relatório como **incompleta**, fora do total dedutível, até ser completada.
- Reembolso recebido no ano seguinte ao da despesa → abate no ano do reembolso *(premissa — confirme ou corrija)*.
- Despesa de saúde de conta administrada (parente) → não entra.
- Despesa paga num cartão adicional de filho → entra normalmente (o filho é dependente); o casal completa os dados (PRD 003, US07).

### US03: Relatório de educação

Como administrador, quero as despesas de educação do ano por pessoa, para preencher a declaração respeitando o limite.

**Rules:**
- Entram os lançamentos realizados no ano em categorias dedutíveis de **educação** (ex.: Faculdade, `docs/categorias-iniciais.md`), por prestador (CNPJ, nome) e por aluno.
- Por pessoa: total pago e o **limite do ano** informado na US01, com o excedente destacado (o valor acima do limite não é dedutível).
- Cursos livres e de idiomas não são dedutíveis; ficam fora se a categoria não estiver marcada *(premissa — confirme ou corrija; a confirmar na Receita junto com o limite do ano)*.

**Edge cases:**
- Limite do ano não informado → o relatório mostra os totais e pede o limite.
- Mesma escola para dois filhos num lançamento → dividido por aluno (PRD 003, US08).

### US04: Relatório de previdência

Como administrador, quero as contribuições de previdência do ano por participante, para declarar o PGBL e o VGBL.

**Rules:**
- Por plano (PRD 002 e PRD 006): participante, tipo (PGBL ou VGBL), CNPJ da entidade e total contribuído no ano.
- **PGBL**: contribuições aparecem como dedutíveis; o app lembra que a dedução tem teto proporcional à renda tributável, que o app não calcula *(premissa — confirme ou corrija)*.
- **VGBL**: não é dedutível; aparece em bens e direitos (US05).

**Edge cases:**
- Contribuição sem participante definido no plano → pendência (US06).
- Portabilidade entre planos (PRD 006, US05) → não conta como contribuição.

### US05: Relatório de bens e direitos

Como administrador, quero o saldo de cada conta e aplicação em 31/12 do ano anterior e do ano, com os dados da declaração, para preencher bens e direitos.

**Rules:**
- Uma linha por conta (corrente, investimento com saldo disponível, dinheiro em espécie *(premissa — confirme ou corrija)*) e por aplicação (PRD 006) e plano VGBL: **saldo em 31/12 do ano anterior** e **saldo em 31/12 do ano**, instituição, **CNPJ**, agência e número (PRD 002, US01), e descrição sugerida.
- O app sugere o **grupo e código** do bem na declaração por tipo de conta e de aplicação, com a observação de que os códigos devem ser conferidos no programa do ano *(premissa — confirme ou corrija)*.
- Saldos em 31/12 usam o valor do extrato daquela data: saldo conferido (PRD 002, US08) ou valor atualizado da aplicação (PRD 006, US03). Saldo não conferido em 31/12 aparece como pendência.
- **Contas no exterior**: saldo em dólar em 31/12 e o valor em reais pela **cotação do dólar para o IR** informada pelo casal na US01, separada do câmbio de fechamento do patrimônio (PRD 008). A regra de conversão (cotação de compra ou de venda do Banco Central, e se vale o saldo ou o custo em reais) **não está confirmada**: as fontes consultadas divergem (seção 8). O app mostra o saldo em dólar e a cotação usada, para o casal conferir.
- **Contas dos filhos** entram (dependentes), identificadas pelo filho. **Contas administradas** não entram.

**Edge cases:**
- Conta aberta no ano → saldo do ano anterior zero.
- Conta encerrada no ano → saldo do ano zero, com o do ano anterior.
- Aplicação que venceu e foi reaplicada → as duas aparecem; a vencida com saldo zero no ano.
- Faltam agência, número ou CNPJ → pendência (US06).

### US06: Acompanhar as pendências do IR durante o ano

Como administrador, quero ver o que falta para a declaração ao longo do ano, para não deixar tudo para março.

**Rules:**
- Lista de pendências do ano: despesas dedutíveis incompletas (sem CPF/CNPJ, sem pessoa atendida, CPF/CNPJ inválido), contas e aplicações sem CNPJ, agência ou número, membros sem CPF, planos sem participante, saldos de 31/12 não conferidos, limite de educação e cotação do IR não informados.
- Cada pendência leva direto ao lançamento ou cadastro a completar.
- Lembrete periódico com o número de pendências *(premissa — confirme ou corrija; ex.: no lembrete mensal de atualização, PRD 006)*.

**Edge cases:**
- Pendência de ano já declarado → pode ser dispensada ("não se aplica").
- Lançamento migrado do Minhas Economias sem dados de IR → só vira pendência no ano da migração em diante (anos passados já foram declarados, PRD 005).

### US07: Exportar o pacote do IR

Como administrador, quero exportar os relatórios do ano, para usar no preenchimento ou entregar a quem faz a declaração.

**Rules:**
- Exportação dos relatórios das US02 a US05 em PDF e planilha, gerada no computador ou celular, sem enviar nada para fora (como no PRD 008, US09).
- O pacote mostra a data de geração e as pendências em aberto.

**Edge cases:**
- Exportar com pendências → permitido, com as pendências listadas na primeira página.
- Gerar de novo depois de corrigir → nova versão com nova data.

## 4. Fluxo de Negócio

```
Durante o ano
   lançamento em categoria dedutível ──▶ CPF/CNPJ + pessoa atendida (PRD 003)
   faltou? ──▶ pendência do IR (US06)

Início do ano seguinte
   US01: titular, dependentes, limite de educação, cotação do IR
      │
      ├──▶ US02 saúde (pago − reembolsado, por prestador e pessoa)
      ├──▶ US03 educação (por pessoa, contra o limite)
      ├──▶ US04 previdência (PGBL dedutível; VGBL em bens)
      └──▶ US05 bens e direitos (31/12 do ano anterior e do ano; exterior × cotação do IR)
                 │
                 ▼
          US07 pacote exportado ──▶ preenchimento no programa da Receita (fora do app)
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Despesas médicas por prestador e pessoa, com pago, reembolsado e dedutível | copiar para a declaração | consulta de 500 com reembolso de 300: dedutível 200 |
| Despesas incompletas ficam fora do total e aparecem como pendência | não declarar sem comprovante | lançar consulta sem CPF: fora do total, na lista de pendências |
| Farmácia e óculos não entram nas despesas médicas | regra da Receita | lançar farmácia: não aparece no relatório |
| Educação por pessoa com o limite do ano destacado | não deduzir acima do limite | informar o limite; lançar acima dele para um filho: excedente destacado |
| Bens e direitos com saldos em 31/12 dos dois anos, CNPJ, agência e número | preencher bens e direitos | gerar o ano: uma linha por conta e aplicação com os dois saldos |
| Exterior em dólar e em reais pela cotação do IR informada | conversão para a declaração | saldo US$ 1.000, cotação do IR 5,00: R$ 5.000 |
| Contas dos filhos entram em bens e direitos; administradas não | filhos são dependentes; dinheiro do parente não é da família | gerar com conta de filho e conta administrada: só a do filho aparece |
| Pacote exportado em PDF e planilha, com pendências na primeira página | entregar a quem faz a declaração | exportar com uma pendência: aparece na primeira página |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Tempo para juntar os dados de despesas dedutíveis e bens | A levantar (declaração anterior) | uma noite | primeira declaração com o app | metade do tempo atual | casal |
| Pendências do IR em aberto em 31/01 | não se aplica | 0 | primeira declaração com o app | menos de 5 | casal |
| Diferenças entre o relatório de saúde e a pré-preenchida | A levantar | 0 sem explicação | primeira declaração com o app | todas explicadas | casal |

## 6. Milestones

### Milestone 1: Despesas dedutíveis prontas para a declaração

**Por que é um marco:** saúde, educação e previdência saem do app organizadas por pessoa e prestador, sem garimpar recibos.

**Funcionalidades:** US01, US02, US03, US04, US06

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Despesas médicas por prestador e pessoa, com pago, reembolsado e dedutível
- [ ] Despesas incompletas ficam fora do total e aparecem como pendência
- [ ] Farmácia e óculos não entram nas despesas médicas
- [ ] Educação por pessoa com o limite do ano destacado

**Aprovador:** casal

### Milestone 2: Bens e direitos e pacote do IR

**Por que é um marco:** os saldos de 31/12 de todas as contas e aplicações, inclusive o exterior, saem prontos, e o pacote inteiro pode ser exportado.

**Funcionalidades:** US05, US07

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Bens e direitos com saldos em 31/12 dos dois anos, CNPJ, agência e número
- [ ] Exterior em dólar e em reais pela cotação do IR informada
- [ ] Contas dos filhos entram em bens e direitos; administradas não
- [ ] Pacote exportado em PDF e planilha, com pendências na primeira página

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Regra da Receita mudar (limites, lista de dedutíveis, conversão do exterior, códigos de bens) | Alto | nenhum limite ou cotação fixo no app (decisão 4); lista de categorias dedutíveis editável; conferir no programa do ano | Monitorando |
| Conversão do exterior incerta (compra ou venda; saldo ou custo) | Médio | cotação informada pelo casal; saldo em dólar sempre à vista; confirmar na Receita antes da primeira declaração com o app | Pendente |
| Regime de caixa no cartão desloca despesa médica de dezembro para janeiro | Baixo | para o IR vale o pagamento, e o pagamento da fatura é a data em que a família pagou (a confirmar na revisão) | Pendente |
| Casal não completar os dados de IR durante o ano | Médio | pendências visíveis e lembrete (US06) | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 003: dados de IR no lançamento, categorias dedutíveis, reembolso | Interna | rascunho (1ª revisão aprovada) | Milestone 1 |
| PRD 002: agência, número e CNPJ das contas; PGBL/VGBL e participante | Interna | rascunho (1ª revisão aprovada) | Milestones 1 e 2 |
| PRD 006: valor das aplicações em 31/12; contribuições | Interna | rascunho (aguardando 1ª revisão) | Milestone 2 |
| Confirmação das regras do ano na Receita (limite de educação, conversão do exterior, códigos) | Externa | a verificar antes da primeira declaração | Milestones 1 e 2 |

## 8. Referências

Fontes consultadas em **2026-11-03** (estado: Verificado, quando oficial; A verificar, quando de terceiros ou divergente):

- [Receita Federal: Despesas dedutíveis](https://www.gov.br/receitafederal/pt-br/assuntos/meu-imposto-de-renda/preenchimento/manual-mir/pagamentos-ou-doacoes/despesas-dedutiveis) — **Verificado**: lista de despesas médicas dedutíveis (médicos, dentistas, psicólogos, fisioterapeutas, terapeutas ocupacionais, fonoaudiólogos, hospitais, clínicas, laboratórios, planos de saúde); comprovante com nome, endereço e CPF/CNPJ do prestador e identificação de quem pagou e de quem recebeu o serviço; Receita Saúde desde 1º/1/2025 para profissionais de saúde pessoa física.
- [Receita Federal: Despesas médicas (malha fiscal)](https://www.gov.br/receitafederal/pt-br/assuntos/meu-imposto-de-renda/malha-fiscal/antecipacao/despesas-medicas) — **Verificado**: só a parte não reembolsada é dedutível; nutricionistas, enfermeiros, psicopedagogos, massagistas, vacinas e medicamentos fora da conta hospitalar não são dedutíveis.
- [Receita Federal: O que posso deduzir como despesa médica?](https://www.gov.br/receitafederal/pt-br/acesso-a-informacao/perguntas-frequentes/imposto-de-renda/dirpf/deducoes/o-que-posso-deduzir-como) — **Verificado**: despesas do contribuinte e dos dependentes da declaração.
- [Receita Federal: Receita Saúde](https://www.gov.br/receitafederal/pt-br/centrais-de-conteudo/publicacoes/manuais/orientacao-tributaria/receita-saude-2.1.pdf) — **Verificado**: recibo eletrônico na base da Receita, aparece na pré-preenchida.
- Limite de educação (R$ 3.561,50 por pessoa no IRPF 2026), dependente (R$ 2.275,08) e PGBL até 12 % da renda tributável: **A verificar** — só em fontes de terceiros ([sum.money](https://sum.money/br/deducoes-irpf/), [adrianofreire.com.br](https://www.adrianofreire.com.br/blog/deducoes-ir-2026-o-que-pode-deduzir)). Por isso o limite é informado pelo casal (decisão 4).
- Conversão de saldos no exterior (Lei 14.754/2023, art. 15): **A verificar** — fontes divergem entre cotação de compra e de venda do Banco Central em 31/12 ([Lei 14.754, Câmara](https://www2.camara.leg.br/legin/fed/lei/2023/lei-14754-12-dezembro-2023-795058-normaatualizada-pl.pdf), [investnews](https://investnews.com.br/financas/como-declarar-investimentos-no-exterior/), [brasiltax](https://brasiltax.com/blog/lei-14754-variacao-cambial/)). Confirmar na instrução normativa ou no manual do IRPF do ano.
- `docs/prds/003-lancamentos-e-categorias.md` (US07), `docs/prds/002-contas-e-cartoes.md`, `docs/prds/006-investimentos-e-previdencia.md`, `docs/categorias-iniciais.md`.

## 9. Registro de Decisões

- **2026-11-03:** Só relatórios de apoio, para despesas dedutíveis e bens e direitos; declaração conjunta com filhos dependentes. Motivo: decisões do casal (rodada 6).
- **2026-11-03:** Limites e cotações do IR informados pelo casal a cada ano, não fixos no app. Motivo: mudam todo ano e as fontes consultadas divergem (seção 8).
- **2026-11-03:** Contas dos filhos entram em bens e direitos; contas administradas não. Motivo: filhos são dependentes; o dinheiro do parente não é da família (PRD 002, US12).
- **2026-11-03:** `depends_on: ["001", "002", "003", "006"]`. Critério: membros e CPF (001), dados das contas e planos (002), dados de IR nos lançamentos e categorias dedutíveis (003), valores das aplicações e contribuições (006).
